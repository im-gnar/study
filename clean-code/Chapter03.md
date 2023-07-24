# Chapter3. 함수

## 작게 만들어라!

- 작은 함수, 코드가 짧은 함수를 만들어라
- `if`/`else`/`while`문 등에 들어가는 블록은 한 줄이어야 한다.
  - 중첩 구조가 생길만큼 함수가 커져서는 안된다.

## 한 가지만 해라!

- 함수는 `한 가지`를 해야 한다. 그 `한 가지`를 잘 해야 한다. 그 `한 가지`만을 해야 한다.
- 함수 내에서 의미 있는 이름으로 다른 함수를 추출할 수 있다면 그 함수는 여러 작업을 하는 셈이다.
- 각 함수는 일정한 추상화 수준을 유지한다.

## Switch문

- 각 switch 문을 저차원 클래스에 숨기고 반복하지 말아야한다.

```java
public Money calculatePay(Employee e) throws InvalidEmployeeType {
    switch (e.type){
        case COMMISSIONED:
            return calculateCommissionedPay(e);
        case HOURLY:
            return calculateHourlyPay(e);
        case SALARIED:
            return calculateSalariedPay(e);
        default:
            throw new InvalidEmployeeType(e.type);
    }
}
```

위 함수에서 몇 가지 문제가 있다.

1. 함수가 길다.
   - 새 직원 유형을 추가하면 더 길어지기 때문
2. `한 가지` 작업만 수행하지 않는다.
3. SRP(Single Responsibility Principle)를 위반한다.
   - 코드를 변경할 이유가 여럿이기 때문
4. OCP(Open Closed Principle)를 위반한다.
   - 새 직원 유형을 추가할 때마다 코드를 변경하기 때문
5. 위 함수와 구조가 동일한 함수가 무한정 존재한다
   ```java
   isPayday(Employee e, Date date);
   deliverPay(Employee e, Money pay);
   ...
   ```
   가능성은 무한하며, 모두가 똑같이 유해한 구조다.

```java
public abstract class Employee {
    public abstract boolean isPayDay();
    public abstract Money calculatePay();
    public abstract void deliverPay(Money pay);
}

public interface EmployeeFactory {
    public Employee makeEmployee(EmployeeRecord r) throws InvalidEmployeeType;
}

public class EmployeeFactoryImpl implements EmployeeFactory {
    public Employee makeEmployee(EmployeeRecord r) throws InvalidEmployeeType {
        switch (r.type) {
            case COMMISSIONED:
                return new CommissionedEmployee(r);
            case HOURLY:
                return new HourlyEmployee(r);
            case SALARIED:
                return new SalariedEmployee(r);
            default:
                throw new InvalidEmployeeType(r.type);
        }
    }
}
```

이렇게 상속 관계로 숨긴 후에는 절대로 다른 코드에 노출하지 않는다.

## 서술적인 이름을 사용하라

> "코드를 읽으면서 짐작했던 기능을 각 루틴이 그대로 수행한다면 깨끗한 코드라 불러도 된다"

- 이름이 길어도 괜찮다.
- 길고 서술적인 이름이, 짧고 어려운 이름보다 좋다.
- 길고 서술적인 이름이, 길고 서술적인 주석보다 좋다.
- 함수 이름을 정할 때는 여러 단어가 쉽게 읽히는 명명법을 사용한다.
- 여러 단어를 사용해 함수 기능을 잘 표현하는 이름을 선택한다.

## 함수 인수

- 최선은 인수가 없는 것이고, 인수가 4개 이상(다항) 인 경우는 피하는게 좋다.
- 인수가 많아질수록 각 인수의 조합으로 함수를 검증해야 하므로 테스트가 굉장히 번거로워진다.
- 플래그 인수는 함수가 한꺼번에 여러 가지를 처리한다고 공표하는 셈이다. (분기처리)
- 인수가 2-3개 필요하다면 클래스 변수로 선언할 것을 고려한다
- 가변 인수 (Object... args) 는 인수 하나로 취급한다.
- 함수명을 명확하게 지으면 인수의 순서와 의도를 알 수 있다

```java
assertEquals(expected, actual);
assertExpectedEqualsActual(expected, actual); // 인수 순서를 기억할 필요가 없다
```

## 부수 효과를 일으키지 마라

- 시간적 결합이나 순서 종속성을 초래한다.

```java
public class UserValidator {
    private Cryptographer cryptographer;

    public boolean checkPassword(String userName, String password) {
        User user = UserGateway.findByName(userName);
        if (user != User.NULL) {
            String codedPhrase = user.getPhraseEncodedByPassword();
            String phrase = cryptographer.decrypt(codedPhrase, password);
            if ("Valid Password".equals(phrase)) {
                Session.initialize();
                return true;
            }
        }
        return false;
    }
}
```

- `userName`과 `password`를 확인하고 두 인수가 올바르면 true를 아니면 false를 반환하는 함수다.
- `Session.initialize()` 호출은 부수 효과를 일으킨다.
  - **함수 이름만 봐서는 세션을 초기화한다는 사실이 드러나지 않는다**.
  - 함수 이름만 보고 함수를 호출하면 **사용자를 인증하면서 기존 세션 정보를 지워버릴 위험에 처한다**.

```java
// Good
public boolean checkPasswordAndInitializeSession(String userName, String password);
```

### 출력 인수

- 일반적으로 출력 인수는 피해야한다.
- 함수에서 상태를 변경해야 한다면 **함수가 속한 객체 상태를 변경하는 방식**을 택한다.

```java
// bad
public void appendFooter(StringBuffer report, String s) {}

// good
report.appendFooter(s);
```

## 명령과 조회를 분리하라

- 함수는 뭔가를 수행하거나 뭔가에 답하거나 둘 중 하나만 해야 한다.

```java
// 이름이 attribute인 속성을 찾아 값을 value로 설정하고 성공하면 true, 실패하면 false를 반환
public boolean set(String attribute, String value)...

// 설정된 이름을 확인하는 코드인지, 설정하는 코드인지 모호함
if(set("username","koiil"))...

if(setAndCheckIfExists("username", "koiil"))...
```

- 명령과 조회를 분리해 혼란을 애초에 뿌리뽑아야 한다.

```java
if(attributeExists("username")){
    setAttribute("username", "imksh");
    ...
}
```

## 오류 코드보다 예외를 사용하라

- 오류 코드 반환은 자칫하면 if 문에서 명령을 표현식으로 사용하기 쉽다.
  > 정상 동작과 오류 처리 동작을 분리하면 코드를 이해하고 수정하기 쉬워진다.

```java
// Bad
if (deletePage(page) == E_OK) {
    if (registry.deleteReference(page.name) == E_OK) {
        if (configKeys.deleteKey(page.name.makeKey()) == E_OK){
            logger.log("page deleted");
        } else {
            logger.log("configKey not deleted");
        }
    } else {
        logger.log("deleteReference from registry fai1ed"
    };
} else {
    logger.log("delete fai1ed");
     return E_ERROR;
}

// Try/Catch
try{
    deletePage(page);
    // DO SOMETHING
}catch(Exception e){
    logger.log(e.getMessage());
}

// 오류 처리용 함수
public void delete(Page page) {
    try {
        deletePageAndAllReferences(page);
    }catch (Exception e){
        logError(e);
    }
}

private void deletePageAndAllReferences(Page page) throws Exception {
    deletePage(page);
    registry.deleteReference(page.name);
    configKeys.deleteKey(page.name.makeKey());
}
```

- 정상 동작과 오류 처리 동작을 분리하면 코드를 이해하고 수정하기 쉬워진다.
- 오류처리도 하나의 작업이다.
- Error ENUM 대신 Exception을 사용하자

## 반복하지 마라

- AOP(Aspect Oriented Programming), COP(Component Oriented Programming)처럼 중복 제거 전략을 사용하자

## 구조적 프로그래밍

- 함수는 입구/출구가 하나씩만 있어야한다. 루프 안에서 break, continue 는 피하자.
- goto 문은 큰 함수에서만 의미가 있으므로 피하자

## 함수를 짜는 과정

- 처음에는 길고 복잡하다
- 단위 테스트 케이스를 만든다
- 다음을 수행한다
  - 코드를 다듬는다
  - 함수를 만든다
  - 이름을 바꾼다
  - 중복을 제거한다
  - 메서드를 줄이고 순서를 바꾼다
  - 때로는 전체 클래스를 쪼개기도 한다
- 위 단계를 거치는 와중에도 항상 단위 테스트를 통과한다
