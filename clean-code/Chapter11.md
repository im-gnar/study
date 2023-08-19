# Chapter11. 시스템

> 깨끗한 코드를 구현하면 낮은 추상화 수준에서 관심사를 분리하기 쉬워진다.
> 시스템(높은 추상화) 수준에서도 깨끗함을 유지하는 방법

### 시스템 제작과 시스템 사용을 분리하라

`제작`과 `사용`은 다르다.<br>
소프트웨어 시스템은 `준비 과정`과 `런타임 로직`을 분리해야 한다.

- 준비 과정 : 애플리케이션 객체를 제작하고 의존성을 연결하는 과정
- 런타임 로직 : 준비 과정 이후에 이어지는 작업

관심사 분리
```java
// 초기화 지연(Lazy Initialization)
public Service getService() {    
    if (service == null) {
        service = new MyServiceImpl(...); // 객체를 생성하는 것은 모든 상황에 적합한 기본값일까?       
    } 
    return service;
}      
```

장점
- 실제 필요시까지 객체를 생성하지 않으므로 불필요한 부하가 없다. 따라서 앱 시작시간이 빨라진다.
- 어떤 경우에도 null을 반환하지 않는다.

단점
- getService() 메서드는 MyServiceImpl 과 생성자 인수에 명시적으로 의존한다.
  - 때문에 런타임 로직에서 MyServiceImpl 를 사용하지 않더라도 의존성을 해결하지 않으면 컴파일이 되지 않는다.
- My ServiceImpl이 무거운 객체라면 단위 테스트에서 적절한 테스트 전용 객체(TEST DOUBLE이나 MOCK OBJECT)를 service 필드에 할당해야 한다.
  - 일반 런타임 로직에다 객체 생성 로직을 섞어놓은 탓에 (service가 null인 경우/null이 아닌 경우 등) 모든 실행 경로도 테스트해야 한다.
  - 객체를 의존하면서 관리해야 하는 책임, 객체를 직접 생성하는 책임 2가지를 따르기에 SRP 단일 책임 원칙을 깨진다



### Main 분리

생성과 관련한 코드는 Main 혹은 Main 이 호출하는 모듈로 옮기고, 나머지 시스템은 모든 객체가 생성되고 모든 의존성이 연결되었다고 가정한다.

![image](https://github.com/rachel5004/study/assets/75432228/a23b6b39-8a57-4d42-96a2-779f401aaf5e)

- main 함수에서 시스템에 필요한 객체를 생성한 후 이를 애플리케이션에 넘긴다.
- 애플리케이션은 그저 객체를 사용할 뿐이다.
- 애플리케이션은 main이나 객체가 생성되는 과정을 전혀 모른다.


#### 팩토리

![image](https://github.com/rachel5004/study/assets/75432228/2e2221b9-76de-430f-bf12-75f94db81524)

객체가 생성되는 시점 을 애플리케이션이 결정할 필요가 있을 경우 팩토리를 사용한다.
- 추상 팩토리 패턴
- 팩토리 메소드 패턴

#### 의존성 주입
 
- 제어 역전(IoC) 기법을 의존성 관리에 적용한 메커니즘이다.
- `SRP` : 한 객체가 맡은 보조 책임을 새로운 객체에게 전적으로 넘긴다.

```java
// 의존성 주입을 '부분적으로' 구현한 JNDI 검색 기능
MyService myService = (MyService)(jndiContext.lookup("NameOfMyService"));
// 호출하는 객체는 반환되는 객체의 유형을 제어하지 않고, 의존성을 능동적으로 해결
```

진정한 의존성 주입은 여기서 한 걸음 더 나아간다.

- 클래스가 의존성을 해결하려 시도하지 않는다. 클래스는 완전히 수동적이다.
- 대신에 의존성을 주입하는 방법으로 setter 메서드나 생성자 인수를 (혹은 둘 다를) 제공한다.
- DI 컨테이너는 (대개 요청이 들어올 때마다) 필요한 객체의 인스턴스를 만든 후 생성자 인수나 setter 메서드를 사용해 의존성을 설정한다.
- 실제로 생성되는 객체 유형은 설정 파일에서 지정하거나 특수 생성 모듈에서 코드로 명시한다.

스프링 프레임워크에서는 `ApplicationContext`나 `BeanFactory`에 책임을 넘긴다.
- 객체 사이 의존성은 XML 파일에 정의한다.
- 자바 코드에서는 이름으로 특정한 객체를 요청한다.

### 확장

> 소프트웨어 시스템은 물리적인 시스템과 다르다. 관심사를 적절히 분리해 관리한다면 소프트웨어 아키텍처는 점진적으로 발전할 수 있다.

**관심사를 적절하게 분리하지 못한 아키텍쳐 :  EJB1과 EJB2**

```java
package com.example.banking;
import java.util.Collections;
import javax.ejb.*;

public interface BankLocal extends java.ejb.EJBLocalObject {
    String getStreetAddr1() throws EJBException;
    String getStreetAddr2() throws EJBException;
    String getCity() throws EJBException;
    String getState() throws EJBException;
    String getZipCode() throws EJBException;
    void setStreetAddr1(String street1) throws EJBException;
    void setStreetAddr2(String street2) throws EJBException;
    void setCity(String city) throws EJBException;
    void setState(String state) throws EJBException;
    void setZipCode(String zip) throws EJBException;
    Collection getAccounts() throws EJBException;
    void setAccounts(Collection accounts) throws EJBException;
    void addAccount(AccountDTO accountDTO) throws EJBException;
}
```

-열거하는 속성은 Bank 주소, 은행이 소유하는 계좌다.
각 계좌 정보는 Account EJB로 처리한다.

```java
package com.example.banking;
import java.util.Collections;
import javax.ejb.*;

public abstract class Bank implements javax.ejb.EntityBean { 상응하는 EJB2 엔티티 빈 구현
    // 비즈니스 로직...
    public abstract String getStreetAddr1();
    public abstract String getStreetAddr2();
    public abstract String getCity();
    public abstract String getState();
    public abstract String getZipCode();
    public abstract void setStreetAddr1(String street1);
    public abstract void setStreetAddr2(String street2);
    public abstract void setCity(String city);
    public abstract void setState(String state);
    public abstract void setZipCode(String zip);
    public abstract Collection getAccounts();
    public abstract void setAccounts(Collection accounts);

    public void addAccount(AccountDTO accountDTO) {
        InitialContext context = new InitialContext();
        AccountHomeLocal accountHome = context.lookup("AccountHomeLocal");
        AccountLocal account = accountHome.create(accountDTO);
        Collection accounts = getAccounts();
        accounts.add(account);
    }

    // EJB 컨테이너 로직
    public abstract void setId(Integer id);
    public abstract Integer getId();
    public Integer ejbCreate(Integer id) { ... }
    public void ejbPostCreate(Integer id) { ... }

    // 나머지도 구현해야 하지만 일반적으로 비어있다.
    public void setEntityContext(EntityContext ctx) {}
    public void unsetEntityContext() {}
    public void ejbActivate() {}
    public void ejbPassivate() {}
    public void ejbLoad() {}
    public void ejbStore() {}
    public void ejbRemove() {}
}
```

**문제점**

- 비즈니스 로직은 EJB2 컨테이너에 강하게 결합된다.
  - 클래스를 생성할 때는 컨테이너에서 파생해야 하며 컨테이너가 요구하는 다양한 생명주기 메서드도 제공해야 한다.
- 비즈니스 논리가 덩치 큰 컨테이너와 밀접하게 결합된 탓에 독자적인 단위 테스트가 어렵다.
  - 컨테이너를 흉내 내거나 많은 시간을 낭비하며 EJB와 테스트를 실제 서버에 배치해야 한다.
  - 그래서 EJB2 코드는 프레임워크 밖에서 재사용하기란 사실상 불가능하다.
- 상속도 불가능하며 쓸데없는 DTO(Data Transfer Object)를 작성하게 만들고, 결국 객체 지향 프로그래밍이라는 개념조차 흔들린다.

#### 횡단(cross-cutting) 관심사

- 이론적으로는 독립된 형태로 구분될 수 있지만 실제로는 코드에 산재하기 쉬운 부분들을 뜻한다.
  - transaction, authorization, logging 등
- EJB 아키텍처가 영속성, 보안, 트랜잭션을 처리하는 방식은 관점 지향 프로그래밍(AOP)을 예견했다.

#### 자바 프록시(JDK Proxy)

- 자바 프록시는 단순한 상황에 적합하다.
- 개별 객체나 클래스에서 메서드 호출을 감싸는 경우가 좋은 예다.

```java
// Bank.java (suppressing package names...)
import java.utils.*;

// The abstraction of a bank.
public interface Bank {
    Collection<Account> getAccounts();
    void setAccounts(Collection<Account> accounts);
}

// BankImpl.java
import java.utils.*;

// POJO implementing the abstraction.
public class BankImpl implements Bank {
    private List<Account> accounts;

    public Collection<Account> getAccounts() {
        return accounts;
    }
    
    public void setAccounts(Collection<Account> accounts) {
        this.accounts = new ArrayList<Account>();
        for (Account account: accounts) {
            this.accounts.add(account);
        }
    }
}
```
```java
// BankProxyHandler.java
import java.lang.reflect.*;
import java.util.*;

// “InvocationHandler” required by the proxy API.
public class BankProxyHandler implements InvocationHandler {
    private Bank bank;
    
    public BankHandler (Bank bank) {
        this.bank = bank;
    }
    
    // Method defined in InvocationHandler
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        String methodName = method.getName();
        if (methodName.equals("getAccounts")) {
            bank.setAccounts(getAccountsFromDatabase());
            
            return bank.getAccounts();
        } else if (methodName.equals("setAccounts")) {
            bank.setAccounts((Collection<Account>) args[0]);
            setAccountsToDatabase(bank.getAccounts());
            
            return null;
        } else {
            ...
        }
    }
    
    // Lots of details here:
    protected Collection<Account> getAccountsFromDatabase() { ... }
    protected void setAccountsToDatabase(Collection<Account> accounts) { ... }
}
```
```java
Bank bank = (Bank) Proxy.newProxyInstance(
    Bank.class.getClassLoader(),
    new Class[] { Bank.class },
    new BankProxyHandler(new BankImpl())
);
```
- 프록시로 감쌀 인터페이스 Bank와 비즈니스 논리를 구현하는 POJO BankImpl을 정의했다.
- 단순한 예제지만 코드가 상당히 많으며 제법 복잡하다. 바이트 조작 라이브러리를 사용하더라도 만만찮게 어렵다.
- 코드 '양'과 크기는 프록시의 두 가지 단점이다. 다시 말해서, 프록시를 사용하면 깨끗한 코드를 작성하기 어렵다.
- 또한 프록시는 진정한 AOP 해법에 필요한 'advice'를 명시하는 메커니즘도 제공하지 않는다.


#### 순수 자바 AOP 프레임워크

위 Java Proxy API의 단점들은 Spring, JBoss와 같은 순수 자바 AOP 프레임워크를 통해 해결할 수 있다.<br>
Spring에서는 비지니스 로직을 POJO로 작성해 자신이 속한 도메인에 집중하게 한다.<br>
결과적으로 의존성은 줄어들고 테스트 작성에 필요한 고민도 줄어든다.<br> 이러한 심플함은 user story의 구현과 유지보수, 확장 또한 간편하게 만들어 준다.

```java
<beans>
    ...
    <bean id="appDataSource"
        class="org.apache.commons.dbcp.BasicDataSource"
        destroy-method="close"
        p:driverClassName="com.mysql.jdbc.Driver"
        p:url="jdbc:mysql://localhost:3306/mydb"
        p:username="me"/>
    
    <bean id="bankDataAccessObject"
        class="com.example.banking.persistence.BankDataAccessObject"
        p:dataSource-ref="appDataSource"/>
    
    <bean id="bank"
        class="com.example.banking.model.Bank"
        p:dataAccessObject-ref="bankDataAccessObject"/>
    ...
</beans>
```

![image](https://github.com/rachel5004/study/assets/75432228/90b1b633-fd96-416c-9802-bb32cdadc84b)

Bank객체는 BankDataAccessObject가, BankDataAccessObject는 BankDataSource가 감싸 프록시하는 구조로 되어 각각의 bean들이 "러시안 인형"의 한 부분처럼 구성되었다. <br>
클라이언트는 Bank에 접근하고 있다고 생각하지만 사실은 가장 바깥의 BankDataSource에 접근하고 있는 것이다.


애플리케이션에서 DI 컨테이너에게 (XML 파일에 명시된) 시스템 내 최상위 객체를 요청하려면 다음 코드가 필요하다.
```java
XmlBeanFactory bf = new XmlBeanFactory(new ClassPathResource("app.xml", getClass()));
Bank bank = (Bank) bf.getBean("bank");
```

- 스프링 관련 코드가 거의 필요 없으므로 사실상 프레임워크와 독립적인 애플리케이션을 구현할 수 있다.
- XML은 장황하고 읽기 어렵지만, 이런 설정 파일에 명시된 '정책'이 자동으로 생성되는 프록시나 관점 논리보다는 단순하다.
- EJB3은 xml와 Java5 annotation을 사용해 횡단 관심사를 선언적으로 지원하는 스프링 모델을 따른다.


```JAVA
// 목록 11-5 EJB3 Bank EJB
package com.example.banking.model;

import javax.persistence.*;
import java.util.ArrayList;
import java.util.Collection;

@Entity
@Table(name = "BANKS")
public class Bank implements java.io.Serializable {
    @Id @GeneratedValue(strategy=GenerationType.AUTO)
    private int id;

    @Embeddable // Bank의 데이터베이스 행에 인라인으로 포함된 객체
    public class Address {
        protected String streetAddr1;
        protected String streetAddr2;
        protected String city;
        protected String state;
        protected String zipCode;
    }

    @Embedded
    private Address address;
    @OneToMany(cascade = CascadeType.ALL, fetch = FetchType.EAGER, mappedBy="bank")
    private Collection<Account> accounts = new ArrayList<Account>();
    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public void addAccount(Account account) {
        account.setBank(this);
        accounts.add(account);
    }

    public Collection<Account> getAccounts() {
        return accounts;
    }

    public void setAccounts(Collection<Account> accounts) {
        this.accounts = accounts;
    }
}
```


#### AspectJ 관점

- AOP를 실현하기 위한 가장 강력한 도구는 AspectJ 언어다.
- 8~90%의 경우에는 Spring AOP와 JBoss AOP로도 충분하지만 AspectJ는 훨씬 강력한 수준의 AOP를 지원한다.
- 다만 AspectJ 를 사용하기 위해 새로운 툴, 언어 구조, 관습적인 코드를 익혀야 한다는 단점도 존재한다.
- 최근 나온 "annotation-form AspectJ"로 인해 그 부담은 많이 줄어들었다고 한다.

### 테스트 주도 시스템 아키텍처 구축

- 코드 수준에서 아키텍처 관심사를 분리할 수 있다면, 진정한 테스트 주도 아키텍처 구축이 가능해진다.
- 처음에는 작고 간단한 구조에서 시작하지만 필요에 따라 새로운 기술을 추가해 정교한 아키텍처로 확장시킬 수 있다.

> 최선의 시스템 구조는 각기 POJO (또는 다른) 객체로 구현되는 모듈화된 관심사 영역(도메인)으로 구성된다. 이렇게 서로 다른 영역은 해당 영역 코드에 최소한의 영향을 미치는 관점이나 유사한 도구를 사용해 통합한다. 이런 구조 역시 코드와 마찬가지로 테스트 주도 기법을 적용할 수 있다.

### 의사 결정을 최적화하라

- 모듈을 나누고 관심사를 분리하면 지엽적인 관리와 결정이 가능해진다.
- 결정은 최대한 많은 정보가 모일 때까지 미루고 시기가 되었을 때 책임자(여기서는 사람이 아닌 모듈화된 컴포넌트를 뜻한다)에게 맡기는 것이 좋다

> 관심사를 모듈로 분리한 POJO 시스템은 기민함을 제공한다. 이런 기민함 덕택에 최신 정보에 기반해 최선의 시점에 최적의 결정을 내리기가 쉬워진다. 또한 결정의 복잡성도 줄어든다.


### 명백한 가치가 있을 때 표준을 현명하게 사용하라

- 가볍고 간단한 설계로 충분했을 프로젝트에서도 단지 표준이라는 이유로 EJB2를 채택했다.
- 표준에 집착하는 바람에 고객 가치가 뒷전으로 밀려날 수 있다.

> 표준을 사용하면 아이디어와 컴포넌트를 재사용하기 쉽고, 적절한 경험을 가진 사람을 구하기 쉬우며, 좋은 아이디어를 캡슐화하기 쉽고, 컴포넌트를 엮기 쉽다. 하지만 때로는 표준을 만드는 시간이 너무 오래 걸려 업계가 기다리지 못한다. 어떤 표준은 원래 표준을 제정한 목적을 잊어버리기도 한다.


### 시스템은 도메인 특화 언어가 필요하다

- DSL(Domain-Specific Language, 도메인 특화 언어)은 간단한 스크립트 언어나 표준 언어로 구현한 API를 카리킨다.
- 좋은 DSL은 도메인 개념과 그 개념을 구현한 코드 사이에 존재하는 '의사소통 간극'을 줄여준다.
- DSL을 효율적으로 사용하면 코드 덩어리와 디자인 패턴의 추상도를 높여 주며 그에 따라 코드의 의도를 적절한 추상화 레벨에서 표현할 수 있게 해준다.

> 도메인 특화 언어를 사용하면 고차원 정책에서 저차원 세부사항에 이르기까지 모든 추상화 수준과 모든 도메인을 POJO로 표현할 수 있다.


### 결론
- 깨끗하지 못한 아키텍처는 도메인 논리를 흐리며 기민성을 떨어뜨리므로 시스템 역시 깨끗해야 한다.\
- 모든 추상화 단계에서 의도는 명확히 표현해야 한다. 그러려면 POJO를 작성하고 관점 혹은 관점과 유사한 메커니즘을 사용해 각 구현 관심사를 분리해야 한다.
- 시스템을 설계하든 개별 모듈을 설계하든, 실제로 돌아가는 가장 단순한 수단을 사용해야 한다는 사실을 명심하자.
