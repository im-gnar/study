## 트랜잭션(transation)

- 완전성 보장 ( 작업 셋이 100% 적용되거나, 아무것도 적용되지 않아야 함을 보장)
  - COMMIT
  - ROLLBACK
- 정합성을 보장하기 위한 기능

잠금

- 동시성을 제어하기 위한 기능
- 하나의 자원에 여러 커넥션이 요청할 경우 순서대로 처리하게 하는 것
- 격리수준 : 여러 트랜잭션 간의 작업 내용을 어디까지 공유하고 차단할 지

---

## 5.1 트랜잭션

`MyISAM`과 `MEMORY` 스토리지 엔진은 트랜잭션을 지원하지 않음. (`InnoDB`만 지원)

### 5.1.1 Mysql 에서의 트랜잭션

`InnoDB`는 트랜잭션의 원칙대로 작업 셋 중 하나라도 실패하면 이전 작업을 롤백하여 원래 상태로 되돌리지만,

`MyISAM` 은 이미 실행된 쿼리를 되돌리지 않고 중지한다.(partial update) → 정합성 문제를 야기

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F7218ca65-e83d-43b2-b8c1-65e51296687a%2FUntitled.png?id=f8d3844e-1c5e-40c6-a9eb-df0ec17a9481&table=block&spaceId=677f44d9-860d-4b3d-8d3a-fa4784c6b778&width=1920&userId=782bb73a-d501-4085-abaa-f4f4b8e9b2f7&cache=v2)

- 트랜잭션은 어플리케이션 개발에서 고민해야할 문제를 줄여준다. 없으면 수동으로 롤백 등의 처리를 해야함

### 5.1.2 주의사항

- 꼭 필요한 최소의 코드에만 적용하는 것이 좋다.
- 데이터베이스 커넥션은 개수가 제한적이어서 작업이 커넥션을 소유하는 시간이 길어질수록 사용가능한 커넥션의 개수는 줄어들 것이다.
- 메일전송, FTP 파일전송, 네트워크를 통해 원격 서버와 통신하는 등의 작업은 트랜잭션 밖에서 처리
- 단순 조회는 트랜잭션에서 제외해도 됨

> MSA 에서의 트랜잭션 관리

- 모놀리식 서비스는 트랜잭션을 하나로 관리가 가능하지만, MSA 는 하나의 트랜잭션으로 묶기 어렵다.
- msa 구성할 때 피드 좋아요에서 문제를 느꼈음
  - 유저(따봉한 글), 피드(따봉수), 따봉(유저+피드번호)
  - 일단 누르면 유저로 보내서 update 치고
  - 따봉정보 insert하고
  - 피드 따봉수도 update
  - 하나라도 삑나면 롤백해야하는데 다른 서비스에서 처리된거라 모놀리식만큼 쉽지않음
- SAGA 패턴과 보상 트랜잭션의 개념

[MSA 핵심 디자인 패턴-saga, event sourcing, CQRS, BFF, Api Gateway](https://doqtqu.tistory.com/337)

[SAGA 패턴 & 보상트랜잭션](https://waspro.tistory.com/603)

---

## 5.2 MySQL 엔진의 잠금

- 스토리지 엔진레벨과 MySQL 엔진 레벨로 나눌 수 있다.
- 스토리지 엔진을 제외한 나머지 부분을 MySQL 엔진이라고 보면된다.
- MySQL 엔진 레벨의 잠금은 모든 스토리지 엔진에 영향을 미치지만, 스토리지 엔진 레벨의 잠금은 스토리지 엔진 간 상호 영향을 미치지 않는다.
- MySQL 엔진에서는 테이블 락, 메타데이터 락, 네임드 락과 같은 잠금을 제공한다.

### 5.2.1 글로벌 락

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F0ef0f90b-fb75-49bd-a77a-0e22e41fc47f%2FUntitled.png?id=9b1f7267-7981-4418-be08-869569e0548f&table=block&spaceId=677f44d9-860d-4b3d-8d3a-fa4784c6b778&width=1920&userId=782bb73a-d501-4085-abaa-f4f4b8e9b2f7&cache=v2)

- MySQL에서 제공하는 잠금 가운데 범위가 가장 큼
  - MySQL 서버 전체
  - 작업 대상 테이블뿐만 아니라 다른 데이터베이스 포함
- 실행 중인 쿼리가 있다면 해당 쿼리 수행 후에 락이 적용된다
  - 잠금을 걸기 전 flush 를 해야하기 때문
- mysqldump 등 백업 프로그램에서 내부적으로 사용됨
  - 옵션을 잘 확인하는게 좋음
  - 지금은 특정 DB 만 덤프를 뜨는 쿼리도 있던데, 이건 그 DB만? 아니면 동일하게 전체 락?
- MySQL 8.0에서는 데이터 변경을 허용하는 백업 락 추가됨
  - innodb 사용으로 가벼운 글로벌락의 필요성
- 백업락 획득 시 모든 세션에서는 스키마, 인증 관련 정보 변경 불가
  - 데이터베이스, 테이블 등 모든 객세 생성,수정,삭제 불가
  - repair table, optimize table 불가
  - 사용자 관리 및 비밀번호 변경 불가

### 5.2.2 테이블 락

- 개별 테이블 단위로 설정되는 잠금
- 명시적 획득
  - `LOCK TABLES table_name [ READ | WRITE ]` 명령
  - 특별한 상황이 아니면 명시적 테이블락도 애플리케이션에서 사용할 필요가 거의 없다.
- 묵시적 획득
  - MyISAM이나 MEMORY 테이블에 데이터를 변경하는 쿼리를 실행하면 발생
  - 데이터가 변경되는 테이블에 잠금을 설정하고 데이터를 변경한 후 즉시 잠금을 해제하는 형태
- InnoDB 테이블의 경우 스토리지 엔진 차원에서 레코드 기반의 잠금을 제공하기 때문에 단순 데이터 변경 쿼리로 인해 묵시적인 테이블 락이 설정되지 않음

### 5.2.3 네임드 락

- `GET_LOCK()` 함수로 임의의 문자열에 대해 잠금 설정
- 동일 데이터를 변경하거나 참조하는 프로그램끼리 분류하여 네임드 락을 걸고 쿼리 실행
- 배치처럼 한꺼번에 많은 레코드를 변경하는 경우 데드락이 발생할 확률이 높음
- MySQL 8.0 부터 네임드 락을 중첩 사용 가능

[신규 서비스 배포 전에 실험과 개선을 반복한 이야기](https://helloworld.kurly.com/blog/vsms-performance-experiment/#%EB%8D%B0%EB%93%9C%EB%9D%BD%EC%9D%98-%EC%9B%90%EC%9D%B8-%EF%B8%8F)

### 5.2.4 메타데이터 락

- 데이터베이스 객체(테이블,뷰 등)의 이름이나 구조 변경 시 자동으로 획득하는 잠금

---

## **5.3 InnoDB 스토리지 엔진 잠금**

- 레코드 기반 잠금
- 이전에는 `lock_monitor` 혹은 `SHOW ENGINE INNODB STATUS` 로만 모니터링 가능
- 최근 버전에서는 트랜잭션 상태와 잠금에 대해서 조회할 수 있는 여러 모니터링 방법 추가

[MySQL :: MySQL 8.0 Reference Manual :: 15.7.1 InnoDB Locking](https://dev.mysql.com/doc/refman/8.0/en/innodb-locking.html#innodb-gap-locks)

### 5.3.1 InnoDB 스토리지 엔진의 잠금

- 레코드 기반의 잠금 기능을 제공하며, 잠금 정보가 상당히 작은 공간으로 관리되기 때문에 락 에스컬레이션 현상은 없음
  - 레코드 락이 페이지 락 또는 테이블 락으로 전파되는 경우
- 레코드와 레코드 사이의 간격을 잠그는 갭(GAP) 락이라는 것이 존재한다.

**5.3.1.1 레코드 락**

- 레코드 자체만을 잠그는 것
- InnoDB 스토리지 엔진은 레코드 자체가 아니라 인덱스의 레코드를 잠근다.
  - 레코드 자체를 잠그느냐, 인덱스를 잠그느냐는 상당히 크고 중요한 차이
- 보조 인덱스의 경우 넥스트키 락 / 갭 락
- PK 또는 유니크 인덱스의 경우 레코드 락

**5.3.1.2 갭 락**

- 레코드와 바로 인접한 레코드 사이의 **간격만을** 잠그는 것
- 레코드와 레코드 사이의 간격에 새로운 레코드가 생성되는 것을 제어
- 갭락 자체로 사용되기보다 넥스트 키 락의 일부로 자주 사용된다.

**5.3.1.3 넥스트 키 락**

- 레코드 락과 갭락을 둘다 활용해서 구현된 개념이 넥스트 키 락
- STATEMENT 포맷의 바이너리 로그를 사용하는 경우 REPEATABLE READ 격리 수준 사용해야 함
- PK Clustered Index 는 Sparse Index 로 구성 가능하고
- Secondary Index 는 dense index 로 구성가능하다.
- 레플리카 서버와 소스 서버에서 만든 쿼리 결과가 동일하도록 보장
- **팬텀 리드** 문제를 해결하기 위해서 넥스트 키 락이 필요
- 하지만 넥스트 키 락과 갭락으로 인해 데드락이나 다른 트랜잭션을 기다리는 일이 은근히 자주 발생한다.
- 가능하다면 바이너리 로그 포맷을 ROW 형태로 바꿔서 넥스트 키 락이나 갭 락을 줄이는 것이 좋다.
  - MySQL 8.0에서는 기본 설정

**5.3.1.4 자동 증가 락**

- `AUTO_INCREMENT` 칼럼이 사용된 테이블에 동시에 여러 레코드가 INSERT되는 경우, 저장되는 각 레코드는 중복되지 않고 저장된 순서대로 증가하는 일련번호 값을 가져야 한다.
- 트랜잭션과 관계 없이 INSERT나 REPLACE 문장에서 AUTO_INCREMENT 값을 가져오는 순간만 락이 걸렸다가 즉시 해제된다.
- 테이블에 단 하나만 존재한다.
- 하나의 쿼리가 `AUTO_INCREMENT`을 걸면 나머지 쿼리는 `AUTO_INCREMENT` 락을 기다려야 한다.
- `innodb_autoinc_lock_mode` 시스템 변수를 이용해 자동 증가 락의 작동방식을 변경할 수 있다.
- 자동 증가 값이 한번 증가하면 절대 줄어들지 않는 이유는 AUTO_INCREMENT 잠금을 최소화하기 위함.

### 5.3.2 인덱스와 잠금

- InnoDB의 잠금은 레코드를 잠그는 것이 아니라 인덱스를 잠그는 방식
- 인덱스를 사용하면서 접근한 레코드 모두를 잠금
- UPDATE 문장을 위해 적절한 인덱스가 준비돼 있지 않다면 각 클라이언트 간의 동시성이 상당히 떨어져서 한 세션에서 UPDATE 작업을 하는 중에는 다른 클라이언트는 그 테이블을 업데이트하지 못하고 기다려야 하는 상황이 발생

### 5.3.3 레코드 수준의 잠금 확인 및 해제

- MySQL 5.1 부터는 각 트랜잭션이 어떤 잠금을 기다리고 있는지, 기다리고 있는 잠금을 어떤 트랜잭션이 가지고 있는지를 information_schema DB에 `INNODB_TRX`,  `INNODB_LOCKS`, `INNODB_LOCK_WAITS` 테이블을 통해 확인 가능
  ![https://github.com/Growing-Up-Together/ReadingRecord/raw/main/RealMySQL/images/beenz/5%EC%9E%A5/5-5.png](https://github.com/Growing-Up-Together/ReadingRecord/raw/main/RealMySQL/images/beenz/5%EC%9E%A5/5-5.png)
- MySQL 8.0 부터는 information_schema의 정보들은 조금씩 제거되고 있으며, 그 대신 performance_schema의 `data_locks`와 `data_lock_waits` 테이블로 대체 되고 있다.
  - `p173` 예제
    [[데이터베이스] MySQL 트랜잭션 격리 수준](https://steady-coding.tistory.com/562)
    [mysqldump 사용법(db backup 및 load 하기)](https://www.lesstif.com/dbms/mysqldump-db-backup-load-17105804.html)
