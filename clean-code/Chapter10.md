# Chapter10. 클래스

### 클래스 체계 

자바의 클래스 정의 표준은 추상화 단계가 순차적으로 내려간다.

- 정적 공개 상수 `public static` -> 정적 비공개 상수 `private static` -> 비공개 인스턴스 변수 `private`
- 공개 함수
- 비공개 함수 ( 자신을 호출하는 공개 함수 직후에 넣음)

#### 캡슐화

- 변수와 유틸리티 함수는 공개하지 않는 편이 낫지만, 반드시 숨겨야 한다는 법칙도 없다.
- 때로는 변수나 유틸리티 함수를 protected로 선언해 테스트 코드에 접근을 허용하기도 한다.
- 그럼에도 불구하고 비공개 상태를 유지할 온갖 방법을 강구해야 하고, 캡슐화를 풀어주는 결정은 언제나 최후의 수단이다.


### 클래스는 작아야 한다!

- 클래스가 맡은 책임으로 크기를 측정한다.
- 메서드 수가 적다고 책임이 적은 것은 아니다.
- 클래스 이름은 해당 클래스 책임을 기술해야 한다.
  - 클래스 이름이 모호하거나 간결한 이름이 떠오르지 않는다면 책임이 너무 많아서이다. (ex. Processor, Manager, Super..)
  - 클래스 설명은 if, and, or, but을 사용하지 않고 25단어 내외로 가능해야 한다.

#### 단일 책임 원칙 `SRP(Single Responsibility Principle)`

```java
// 책임(변경할 이유) 가 두개인 클래스
public class SuperDashboard extends JFrame implements MetaDataUser {
    public Component getLastFocusedComponent()  // 자바 스윙 컴포넌트 관리
    public void setLastFocused(Component lastFocused)
    public int getMajorVersionNumber()  // 소프트웨어 버전 정보 추적
    public int getMinorVersionNumber()
    public int getBuildNumber() 
}
```

- 클래스나 모듈을 변경할 이유(책임)가 하나뿐이어야 한다.
- 객체 지향 설계에서 중요한 개념이다.
- 변경할 이유를 파악하려 애쓰다 보면 코드를 추상화하기도 쉬워진다.
- 큰 클래스 몇 개가 아니라 작은 클래스 여럿으로 이뤄진 시스템이 더 바람직하다.

#### 응집도

- 메서드가 변수를 많이 사용할수록 메스드와 클래스는 응집도가 높다.
- 응집도가 높다는 말은 클래스에 속한 메서드와 변수가 서로 의존하며 논리적인 단위로 묶인다는 의미이다.


‘함수를 작게, 매개변수 목록을 짧게'라는 전략을 따르다 보면 때때로 몇몇 메서드만이 사용하는 인스턴스 변수가 아주 많아진다.
- 새로운 클래스로 쪼개야 한다는 신호이다
- 응집도가 높아질수록 변수와 메서드를 적절히 분리해 새로운 클래스 두세 개로 쪼개준다.

#### 응집도를 유지하면 작은 클래스 여럿이 나온다 

- 변수가 많은 큰 함수 일부를 작은 함수 하나로 빼내고 싶은데, 빼내려는 코드가 큰 함수에 정의된 변수 넷을 사용한다
- 네 변수를 클래스 인스턴스 변수로 승격하면 새 함수는 인수가 필요 없다 (함수를 쪼개기 쉬워진다)
- 하지만 클래스가 응집력을 잃게된다.
- 몇몇 함수가 몇몇 변수만 사용한다면 독자적인 클래스로 분리하면 된다.

### 변경하기 쉬운 클래스

대다수 시스템은 지속적인 변경이 가해진다.

- 변경할 때마다 시스템이 의도대로 동작하지 않을 위험이 따른다.
- 깨끗한 시스템은 클래스를 체계적으로 정리해 변경에 수반하는 위험을 낮춘다.

```java
public class Sql {
    public Sql(String table, Column[] columns)
    public String create()
    public String insert(Object[] fields)
    public String selectAll()
    public String findByKey(String keyColumn, String keyValue)
    public String select(Column column, String pattern)
    public String select(Criteria criteria)
    public String preparedInsert()
    private String columnList(Column[] columns)
    private String valuesList(Object[] fields, final Column[] columns) 
    private String selectWithCriteria(String criteria)
    private String placeholderList(Column[] columns)
}
```
Sql 클래스는 SRP를 위반한 클래스다.

- 새로운 SQL문을 지원하려면 반드시 Sql 클래스를 손대야 한다. (ex. `update`)
- 기존 SQL문을 수정할 때도 반드시 Sql 클래스를 손대야 한다.

```java
abstract public class Sql {
	public Sql(String table, Column[] columns)
	abstract public String generate();    
}

public class CreateSql extends Sql { // Sql 클래스에서 파생하는 클래스
	public CreateSql(String table, Column[] columns)
	@Override public String generate()
}

public class InsertSql extends Sql {
	public InsertSql(String table, Column[] columns, Object[] fields)
	@Override public String generate()
  // valueList와 같은 비공개 메서드는 해당하는 파생 클래스로 이관
	private String valuesList(Object[] fields, final Colunmn[] columns)
}

public class SelectSql extends Sql {
	public SelectSql(String table, Column[] columns)
	@Override public String generate()
}
...
// 모든 파생 클래스가 공통으로 사용하는 비공개 메서드는 유틸클래스로 이관
public class Where {
    public Where(String criteria)
    public String generate()
}

public class ColumnList {
    public ColumnList(Column[] columns)
    public String generate()
```

- `SRP` 지원
- `OCP` 지원: 클래스 확장에 개방적이고 수정에 폐쇄적이어야 한다는 원칙
  - 파생 클래스(Sql)를 생성하는 방식 → 새 기능에 개방적, 다른 클래스를 닫아놓는 방식으로 수정에 폐쇄적
  - 그저 UpdateSql 클래스를 끼워 넣으면 됨, 기존 클래스를 변경할 필요가 없음
 
#### 변경으로부터 격리

- 요구사항은 변한다. 따라서 코드도 변한다.
- 객체 지향 프로그래밍: concrete 클래스(상세 구현), abstract 클래스(개념)
  - 상세한 구현에 의존하는 클라이언트 클래스는 구현이 바뀌면 위험에 빠진다.
  - 인터페이스와 추상 클래스를 사용해 구현이 미치는 영향을 격리한다.
- 상세한 구현에 의존하는 코드는 테스트가 어렵다.

```java
// TokyoStockExchange API를 직접 호출하지 않고 StockExchange 인터페이스 생성
public interface StockExchange {
    Money currentPrice(String symbol);
}

public Portfolio {
    private StockExchange exchange;
    public Portfolio(StockExchange exchange) {  // 생성자에서 StockExchange를 인수로 받음
        this.exchange = exchange;
    }
    // ...
}
```

```java
public class PortfolioTest {
    private FixedStockExchangeStub exchange;
    private Portfolio portfolio;

    @Before
    protected void setUp() throws Exception {
        // 테스트용 클래스는 StockExchange 인터페이스를 구현하며 고정된 주가를 반환
        exchange = new FixedStockExchangeStub(); 
        exchange.fix("MSFT", 100);
        portfolio = new Portfolio(exchange);
    }

    @Test
    public void GivenFiveMSFTTotalShouldBe500() throws Exception {
        portfolio.add(5, "MSFT");
        Assert.assertEquals(500, portfolio.value()); 
    }
}
```

- 시스템 결합도를 낮추면 유연성과 재사용성이 높아지고 변경으로부터 격리된다.
- 결합도를 줄이면 `DIP(Dependency Inversion Principle)`를 따르는 클래스가 나온다.
  - `DIP`: 클래스가 상세한 구현이 아니라 추상화에 의존해야 한다는 클래스 설계 원칙
