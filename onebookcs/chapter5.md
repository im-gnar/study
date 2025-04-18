# 5장: 컴퓨터 아키텍처와 운영체제

## 기본적인 구조 요소들

- 폰 노이먼: 단일 메모리 버스로 인한 병목 현상 발생
- 하버드: 명령어/데이터 메모리 분리로 병렬 처리 가능 → DSP, 임베디드 시스템에 활용
- 현대: L1 캐시에서 하버드 방식을 채택한 하이브리드 구조

### 프로세서 코어

- 멀티프로세서, 병렬화(여러 CPU 활용)만으로는 성능 높이기 부족
- 전력 장벽(Power Wall) 극복을 위한 설계


### 마이크로프로세서와 마이크로 컴퓨터

- 마이크로프로세서: 메모리와 I/O가 프로세서 코어와 별도 패키지 -> 큰 시스템에 부품으로 사용
- 마이크로컴퓨터(마이크로컨트롤러): 모든 요소를 한 칩안에 패키징 -> 식기세척기 등에 사용되는 작은 컴퓨터
  - 칩 내부에 메모리가 차지하는 영역이 크기 때문에 마이크로프로세서보다 덜 강력함
- 단일 칩 시스템(System on Chip, SoC): 더 복잡한 마이크로컴퓨터. I/O 대신 WiFi 회로 등 복잡한 장치 내장.
  - 일부 SoC는 FPGA 제공하여 사용자 커스터마이징 가능

## 프로시저, 서브루틴, 함수

- 함수: 코드를 재사용하는 중요한 방법 (== 프로시저, 서브루틴...)
- 함수 호출 시 컴퓨터 내부에서 일어나는 과정
1. 반환 주소 저장
2. 함수 파라미터를 누산기에 저장
3. 함수 작업 수행
4. 함수 반환(간접 분기) 뒤에 실행 계속
5. 반환 주소 반환


## 스택

> [!NOTE]
> ```java
> class CallStackTest {
>	public static void main(String[] args) {
>		firstMethod();
>	 }
>	static void firstMethod() {
>		secondMethod();
>	 }
>	static void secondMethod() {
>		System.out.println("secondMethod()");
>	 }
> }
> ```
> ![image](https://github.com/user-attachments/assets/3967b4e6-8a20-408c-8ebf-dbc65df431ef)


> [!NOTE]
> 하드웨어의 스택 오버플로/언더플로 방지를 위한 한계 레지스터
> - 앱 크래시 (앱이 응답하지 않습니다)
> - 앱에서 너무 많은 함수 호출이 중첩되면 스택 오버플로 방지 메커니즘이 작동하여 앱을 강제 종료



## 인터럽트

- 인터럽트 동작 과정
1. 현재 실행 중인 명령어 완료
2. PC(Program Counter), 레지스터 상태 스택에 저장
3. IVT(Interrupt Vector Table)에서 핸들러 주소 로드
4. 커널 모드 전환 후 핸들러 실행
5. IRET 명령어로 복구 및 사용자 모드 복귀


> [!NOTE]
> 주변 장치의 인터럽트 종류
> <img width="720" alt="image" src="https://github.com/user-attachments/assets/12787450-355a-420c-b4aa-f7b9380b0080" />


> [!NOTE]
> Context Switching
> - 타이머, 우선순위 등의 이유로 Interrupt 트리거
> - 현재 실행중인 프로세스 처리
> ```
> 현재 명령어 완료 → EIP 레지스터에 다음 명령어 주소 저장  
> EFLAGS, EAX 등 모든 레지스터 상태를 커널 스택에 푸시  
> IDT(Interrupt Descriptor Table)에서 인터럽트 핸들러 주소 로드  
> ```
>  - 커널 모드 전환 및 핸들러 실행
>  ```c
>  // Linux 커널 코드 (kernel/sched/core.c)
> void scheduler_tick(void) {
>    struct rq *rq = cpu_rq(cpu);
>    struct task_struct *curr = rq->curr;
>    curr->sched_class->task_tick(rq, curr, 0);  // CFS 스케줄러 틱 처리
> }
> ```
>  - 컨텍스트 저장/복구
> ```c
> // task_struct 구조체 (Linux)
> struct task_struct {
>     unsigned int state;          // 프로세스 상태
>     struct thread_info *thread;  // CPU별 레지스터 저장 영역
>     mm_segment_t addr_limit;     // 메모리 접근 권한
>     // ... (150개 이상의 필드)
> };
> ```

## 상대 주소 지정

- 시스템 프로그램 (각 프로그램을 전환시켜 줄 수 있는 일종의 관리자 프로그램)
  - OS, OS 커널
- 사용자 프로그램(프로세스)
  - 시스템 프로그램 외 프로그램
- 시분할 방식: 
- 인덱스 레지스터: 들어있는 값과 명령어의 주소와 더해 유효 주소를 계산
- 상대 주소 지정: 명령어의 주소를 기준으로 하는 상대적인 주소로 해석하는 방식


## 메모리 관리 장치

### MMU(memory management unit)

- 주요 역할
  - 주소 변환: 프로그램이 사용하는 가상 주소를 실제 물리 주소로 매핑
  - 메모리 보호: 각 프로그램이 허용된 메모리 영역만 접근하도록 차단
  - 캐시 관리: 자주 사용하는 메모리 영역을 빠르게 접근할 수 있도록 최적화

- 페이지
  - 메모리를 고정 크기 블록으로 나눈 단위 (일반적으로 4KB)
  - 프로그램이 0x1234 주소(가상) 접근 요청 → MMU가 페이지 테이블 확인 → 실제 물리 주소 0x5678로 변환
  - 프로그램이 5KB 메모리 요청 → 4KB 페이지 2개 할당 → 3KB 공간 낭비 발생
- 페이지 테이블
  - 가상 ↔ 물리 주소 매핑 정보를 저장한 데이터베이스
- 페이지 폴트
  - 물리적 메모리에 연관되지 않은(RAM에 없는) 주소에 접근하면 '페이지 폴트' 예외 발생
  - 페이지폴트 발생 시 새로운 페이지를 할당하기 위해 현재 할당된 페이지 중 교체
  - 페이지 교체 알고리즘 (FIFO, Optimal, LRU, NUR, Second-Chance, Random...)
  

## 가상 메모리

- OS는 MMU를 사용해 사용자 프로그램에 가상 메모리 제공
- 요청 받은 메모리가 사용 가능한 메모리의 크기보다 클 경우 현재 필요하지 않는 페이지를 디스크로 옮김(스왑 아웃)
- 다시 불러들이는 건 '스왑 인'
- 필요한 페이지만을 물리 메모리에 로드하는 요구 페이징(demand paging) 방식
- 성능 저하 막기 위한 다양한 기법 존재 (LRU 등)

> [!TIP]
> 리눅스 스왑 메모리
> - 스왑 메모리: 실제 메모리 RAM이 가득 찼지만 더 많은 메모리가 필요할 때 디스크 공간을 이용하여 부족한 메모리를 제공
> - `swapon`, `swappiness` ...


## 시스템 공간과 사용자 공간

- CPU에는 시스템 모드에 있는지 사용자 모드에 있는지 결정하는 비트가 레지스터에 들어 있음
- I/O 처리 명령어 등 일부 명령어는 특권 명령어(시스템 모드에서만 실행 가능)
- 트랩이나 시스템 콜을 사용해 시스템 모드 프로그램(운영체제)에 요청 보낼 수 있음
- 사용자 프로그램을 다른 프로그램과 운영체제로부터 보호하고, 운영체제가 자원 할당을 전적으로 제어할 수 있다는 장점


## 메모리 계층과 성능

![image](https://github.com/user-attachments/assets/a50a0018-bb64-4a52-8a10-9f4568620eed)


| 영역 | 용도 |
|:--:|:--|
| 레지스터 | - 컴퓨터에서 제일 빠른 메모리로, CPU 계산 과정에서 작동한다<br> - 컴퓨터에서 4대 주요 기능(기억, 해석, 연산, 제어)을 관할하는 장치<br> - CPU는 자체적으로 데이터를 저장할 방법이 없으므로 메모리로 직접 데이터를 전송할 수 없음<br> - 연산을 위해서 반드시 레지스터를 거쳐야 하며, 이를 위해 레지스터는 특정 주소를 가리키거나 값을 읽어올 수 있음<br> - CPU 내부 레지스터: 프로그램 카운터, 누산기, 명령어 레지스터, 상태 레지스터 등이 있다.
| 캐시 메모리 | - CPU와의 거리에 따라 L1 ~ L3 캐시로 나뉨<br> - 데이터나 값을 미리 복사해 놓는 임시 장소로 보통 SRAM으로 구성 |
| 메인 메모리(RAM) | - 프로세서보다 느림<br> - RAM(Random Access Memory) : 휘발성 기억 장치 |
| 스토리지 (SSD, HDD) | - 보조 기억장치|


## 코프로세서

- 한 칩에서 넣어서 프로세서 코어에서 특정 연산을 분리하여 처리 (그래픽 처리 등)
- DMA: 장치에 데이터 복사 등 귀찮은 일 떠맡겨서 CPU가 유용한 연산을 더 많이 처리할 수 있음


## 메모리 상의 데이터 배치

| 영역 | 용도 | 비고 |
|:--:|:--:|:--:|
| 정적 데이터 영역|프로그램 작성 시점에 그 크기가 결정되는 데이터| 얼마나 많은 메모리가 필요한지 알고 있음|
| 힙 | 동적 데이터 | 주로 정적 데이터가 차지하는 영역 바로 위 영역|
|스택 | 함수 호출 시 지역 변수와 반환 주소를 저장| - |

- 마이크로컴퓨터에는 MMU가 없어서 스택을 맨 위에, 힙을 그 아래에 배치하는 경우가 많음
- 힙과 스택이 충돌하지 않게 하는 것이 중요

> [!NOTE]
> <img width="983" alt="image" src="https://github.com/user-attachments/assets/644be540-e227-4929-b3e0-52e37968c7d2" />



## 프로그램 실행

- 라이브러리: 여러 프로그램에서 공통으로 사용되는 함수들을 모은 것
- 프로그램 구조: 단일 파일이 아닌 여러 조각으로 나뉘어 관리
  - 여러 사람이 동시에 프로그램의 다른 부분 개발 가능
- 링크: 나뉜 조각들을 하나로 연결하는 과정
  - 링커: 각 프로그램 조각을 중간 파일로 변환하고 하나의 실행 파일로 만듦
  - ELF(Executable and Linkable Format): 현대의 가장 유명한 링크 방식
- 링크 방식:
  - 정적 링크: 라이브러리를 필요한 함수가 들어있는 파일로 간주해 프로그램과 직접 연결
  - 동적 링크: 여러 프로그램이 같은 라이브러리 공유 가능, 메모리 사용 효율화
- 진입점: 프로그램의 첫 번째 명령어 위치
  - 실제 실행 시 런타임 라이브러리가 먼저 실행된 후 진입점 명령어 실행



