# Chapter9. 단위 테스트

### TDD 법칙 세 가지
```
- 실패하는 단위 테스트를 작성할 때까지 실제 코드를 작성하지 않는다.
- 컴파일은 실패하지 않으면서 실행이 실패하는 정도로만 단위 테스트를 작성한다.
- 현재 실패하는 테스트를 통과할 정도로만 실제 코드를 작성한다.
```

위 법칙을 따르면 테스트 코드와 실제 코드를 거의 동시에 개발할 수 있다.
실제 코드를 전부 테스트하는 테스트 케이스를 만들 수 있지만, 방대한 테스트 코드는 심각한 관리 문제를 유발하기도 한다.

### 깨끗한 테스트 코드 유지하기

- 실제 코드가 진화하면 테스트 코드도 변해야한다
- 복잡한 테스트 코드는 테스트 케이스를 추가하기 어렵고, 오히려 테스트 케이스를 유지보수하는 비용이 늘어난다.
- 테스트 슈트가 없으면 수정한 코드가 제대로 도는지 확인하기 어렵고, 결함율이 높아지기 시작한다.

테스트 코드는 실제 코드 못지 않게 중요하며 깨끗하게 짜야한다.

#### 테스트는 유연성, 유지보수성, 재사용성을 제공한다

코드에 유연성, 유지보수성, 재사용성을 제공하는 버팀목은 단위 테스트다.

- 테스트 케이스가 있으면 변경이 쉬워진다.
- 테스트 케이스가 없다면 모든 변경이 잠정적인 버그다.
- 자동화된 단위 테스트 슈트는 설계와 아키텍처를 최대한 깨끗하게 보존하는 열쇠다.

테스트 코드가 지저분할수록 코드를 변경하는 능력과 코드 구조를 개선하는 능력이 떨어진다.


### 깨끗한 테스트 코드

깨끗한 테스트 코드를 만들려면 가독성이 중요하다.

- 가독성을 높이려면 명료성, 단순성, 풍부한 표현력이 필요하다.
- 테스트 코드는 최소의 표현으로 많은 것을 나타내야 한다.

```java
public void testGetPageHierarchyAsXml() throws Exception {
   // 중복
   // 웹로봇이 사용하는 객체로, 테스트코드와 무관해 테스트코드의 의도만 흐림
  crawler.addPage(root, PathParser.parse("PageOne")); 
  crawler.addPage(root, PathParser.parse("PageOne.ChildOne"));
  crawler.addPage(root, PathParser.parse("PageTwo"));

  request.setResource("root");
  request.addInput("type", "pages");
  // 테스트와 무관
  Responder responder = new SerializedPageResponder();
  SimpleResponse response =
    (SimpleResponse) responder.makeResponse(new FitNesseContext(root), request);
  String xml = response.getContent();

  assertEquals("text/xml", response.getContentType());
  // 중복
  assertSubString("<name>PageOne</name>", xml);
  assertSubString("<name>PageTwo</name>", xml);
  assertSubString("<name>ChildOne</name>", xml);
}
...
```

테스트 코드에서는 좀 더 테스트에 관련된 중요한 내용이 보이도록 작성하는 것이 좋다.

위와 같은 테스트 구조에는 `BUILD-OPERATE-CHECK` 패턴이 적합하다.

- `BUILD` : 테스트 자료를 만든다.
- `OPERATE` : 테스트 자료를 조작한다.
- `CHECK` : 조작한 결과가 올바른지 확인한다.


```java
public void testGetPageHierarchyAsXml() throws Exception {
  // 잡다하고 세세한 코드를 함수로 묶어서 처리
  makePages("PageOne", "PageOne.ChildOne", "PageTwo");

  submitRequest("root", "type:pages");

  assertResponseIsXML();
  assertResponseContains(
    "<name>PageOne</name>", "<name>PageTwo</name>", "<name>ChildOne</name>");
}
...
```

#### 도메인에 특화된 테스트 언어 `DSL(Domain-specific language)`

- 시스템 조작 API를 사용하는 대신 API 위에다 함수와 유틸리티를 구현한 후 그 함수와 유틸리티를 사용해 테스트 코드를 짜기도 읽기도 쉬워진다
- 테스트 코드에서 사용하는 특수 API(테스트 언어)가 된다
- 숙련된 개발자라면 자기 코드를 좀 더 간결하고 표현력이 풍부한 코드로 리팩터링해야한다.

#### 이중 표준

테스트 API 코드에 적용하는 표준은 실제 코드에 적용하는 표준과 다르다.

- 실제 환경이 아니라 테스트 환경에서 돌아가는 코드이기 때문에 실제 코드만큼 효율적일 필요는 없다.
- 실제 환경에서는 절대로 안 되지만 테스트 환경에서는 전혀 문제 없는 방식이 있다.
  - 메모리, CPU 효율과 관련 있는 경우 등

```java
// Bad
@Test
public void turnOnLoTempAlarmAtThreashold() throws Exception {
  hw.setTemp(WAY_TOO_COLD); 
  controller.tic(); 
  assertTrue(hw.heaterState());   
  assertTrue(hw.blowerState()); 
  assertFalse(hw.coolerState()); 
  assertFalse(hw.hiTempAlarm());       
  assertTrue(hw.loTempAlarm());
}

// Good
@Test
public void turnOnCoolerAndBlowerIfTooHot() throws Exception {
  tooHot();
  assertEquals("hBchl", hw.getState());
}

@Test
public void turnOnHeaterAndBlowerIfTooCold() throws Exception {
  tooCold();
  assertEquals("HBchl", hw.getState());
}

@Test
public void turnOnHiTempAlarmAtThreshold() throws Exception {
  wayTooHot();
  assertEquals("hBCHl", hw.getState());
}

@Test
public void turnOnLoTempAlarmAtThreshold() throws Exception {
  wayTooCold();
  assertEquals("HBchL", hw.getState());
}

public String getState() {
  String state = "";
  state += heater ? "H" : "h";
  state += blower ? "B" : "b";
  state += cooler ? "C" : "c";
  state += hiTempAlarm ? "H" : "h";
  state += loTempAlarm ? "L" : "l";
  return state;
}
```

#### 테스트당 assert 하나

JUnit으로 테스트 코드를 짤 때는 함수마다 assert 문을 단 하나만 사용해야 한다

- `장점`: assert문이 하나인 함수는 결론이 하나라서 코드를 이해하기 쉽고 빠르다.
- `단점`: 테스트를 분리하면 중복되는 코드가 많아진다.

```java
public void testGetPageHierarchyAsXml() throws Exception { 
  // given
  givenPages("PageOne", "PageOne.ChildOne", "PageTwo"); 
  // when
  whenRequestIsIssued("root", "type:pages");
  // then
  thenResponseShouldBeXML(); 
}

public void testGetPageHierarchyHasRightTags() throws Exception { 
  // given
  givenPages("PageOne", "PageOne.ChildOne", "PageTwo"); 
  // when
  whenRequestIsIssued("root", "type:pages");
  // then
  thenResponseShouldContain(
		"<name>PageOne</name>", "<name>PageTwo</name>", "<name>ChildOne</name>"
	); 
}
```

- 기존 코드에서 한 번에 사용되는 'assert 문'을 2가지 메서드로 분리
- `given-when-then` 이라는 관례를 사용하여 이름 변경
- **테스트 메서드는 한 가지 결론(기능)을 가지고 있어 코드가 보다 읽기 쉬워졌다.**

하지만, **테스트를 분리하면서 중복되는 코드가 생겼다.**

이를 해결하기 위한 방법은 2가지가 있다.

1. `TEMPLATE METHOD` 패턴 사용
- given/when 부분을 부모 클래스에 두고 then 부분을 자식 클래스에 두는 방식

2. 완전히 독자적인 테스트 클래스 생성
- `@Before` 함수에서 클래스 값으로 given/when 설정
- `@Test` 함수에서 given/when은 클래스로 사용하고 then 설정


#### 테스트당 개념 하나

- 테스트 함수마다 한 개념만 테스트하라
- 개념 당 assert문 수를 최소로 줄여라

#### F.I.R.S.T

깨끗한 테스트는 다음 다섯 가지 규칙을 따른다.

- `Fast(빠르게)` : 테스트는 빨리 돌아야 한다.
  - 테스트가 느려 자주 돌리지 못하면 초반에 문제를 찾아내 고치지 못한다.
- `Independent(독립적으로)` : 각 테스트는 서로 의존하면 안 된다.
  - 각 테스트는 독립적으로, 어떤 순서로 실행해도 괜찮아야 한다.
  - 하나가 실패할 때 나머지도 잇달아 실패하므로 원인을 진단하기 어려워지며 후반 테스트가 찾아내야 할 결함이 숨겨진다.
- `Repeatable(반복가능하게)` : 테스트는 어떤 환경에서도 반복 가능해야 한다.
  - 실제 환경, QA 환경, 네트워크에 연결되지 않은 노트북 환경에서도 실행할 수 있어야 한다.
- `Self-Validating(자가 검증하는)` : 테스트는 bool 값으로 결과를 내야 한다 (성공 아니면 실패)
- `Timely(적시에)` : 단위 테스트는 테스트하려는 실제 코드를 구현하기 직전에 구현한다.


### 결론

- 테스트 코드는 실제 코드 만큼이나 프로젝트 건강에 중요하다.
- 테스트 코드는 실제 코드의 유연성, 유지 보수성, 재사용성을 보존하고 강화한다.
- 테스트 코드를 지속적으로 깨끗하게 관리하고 표현력을 높이고 간결하게 정리하자.
- 테스트 API를 구현해 도메인 특화 언어(DSL)를 만들자.
- 테스트 코드가 망가지면 실제 코드도 망가진다. 테스트 코드를 깨끗하게 유지하자.
