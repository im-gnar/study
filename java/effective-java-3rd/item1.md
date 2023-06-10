## [ITEM 01] 생성자 대신 정적 팩터리 메서드를 고려하라
POJO 객체 생성 방법
- Public 생성자 : XArgsConstructer(access = AccessLevel.PUBLIC) 등
- 정적 팩토리 메서드 : 해당 클래스의 인스턴스를 반환하는 static method (of, from...)



### 장점 1.이름을 가질 수 있다

생성될 객체의 상태나 특성을 묘사하는 이름을 가질 수 있다.<br>
또한 public 생성자의 경우 파라미터가 동일한 생성자를 중복생성할 수 없으나, static method는 가능하다.

```java
public class Person {
    String name;
    String address;
    
    public Person(String name) {
        this.name = name;
    }
    
    // error
    public Person(String address) {
        this.address = address;
    }
    
    // works
    public static Person withName(String name) {
        return new Person(name);
    }
    
    public static Person withAddress(String address) {
      Person person = new Person();
    	person.address = address;
      return person;
    }
}
```

### 장점 2.호출될 때마다 인스턴스를 새로 생성하지 않아도 된다.

Immutable class를 미리 만들어 반환하거나 캐싱하여 재사용하여 빌필요한 객체 생성을 줄일 수 있다. (Flyweight pattern)

Boolean.TRUE & FALSE 가 예로 나왓는데, 고정적인 값을 사용하기 때문이면 enum 도 인가?


```java
@IntrinsicCandidate
public static Boolean valueOf(boolean b) {
    return b ? Boolean.TRUE : Boolean.FALSE;
}
```

### 장점 3. 리턴 타입의 하위 타입 인스턴스를 만들 수도 있다

리턴 타입은 인터페이스로 지정하고, 실제로는 인터페이스의 구현체를 리턴함으로서 구현체의 API 는 노출시키지 않고 객체를 생성할 수 있다.<br>
인터페이스의 뒤에 감추어져 있으므로, public으로 제공해야할 API의 갯수는 줄어들고 프로그래머가 API를 제공하기 위해 알아야하는 개념과 난이도 또한 줄어든다.

> 자바 8 이전에는 인터페이스 내에 정적 메소드를 가질 수 없었지만자바 8 버전 이후로는 인터페이스에서 public static 메소드를 추가할 수 있기 때문에 인터페이스로 만들 수 있다. 
private static 메서드는 자바 9 부터 이용할 수 있다.

```java
public class Collections {
    private Collections() {
    }

    // 정적 펙토리 메서드
    public static <T> List<T> unmodifiableList(List<? extends T> list) {
        return (list instanceof RandomAccess ?
                new UnmodifiableRandomAccessList<>(list) :
                new UnmodifiableList<>(list));
    }

    // 구현체(non-public)
    static class UnmodifiableList<E> extends UnmodifiableCollection<E> implements List<E> {
        ...
    }
}
```
return 타입으로는 인터페이스를, 실제 return 되는 값은 구현체로 지정할 수 있다.<br>
이렇게 하면 밖에서 보았을 때는 실제 구현체가 무엇인지는 모르는 채로 인터페이스만 가지고 코딩을 하게 된다.<br>

> 인터페이스의 뒤에 감추어져 있으므로 public 으로 제공해야할 API의 갯수는 줄어들고, 프로그래머가 API를 제공하기 위해 알아야하는 개념과 난이도 또한 줄어든다.

#### 장점 4. 리턴하는 객체의 클래스가 입력 매개변수에 따라 매번 다를 수 있다

반환 타입의 하위 타입이기만 하면 어떤 클래스의 객체를 반환하든 상관없다. (심지어 다른 클래스가 객체를 반환해도 된다.)<br>
예시로 나온 EnumSet 클래스 같은 경우 클라이언트는 내부에 두 클래스의 존재를 모르지만 원소의 수에 따라 원소가 64개 이하이면 RegularEnumSet을 반환하고, 64개를 초과하면 JumboEnumSet 을 반환한다.

```java
public static <E extends Enum<E>> EnumSet<E> noneOf(Class<E> elementType) {
        Enum<?>[] universe = getUniverse(elementType);
        if (universe == null)
            throw new ClassCastException(elementType + " not an enum");
        if (universe.length <= 64)
            return new RegularEnumSet<>(elementType, universe);
        else
            return new JumboEnumSet<>(elementType, universe);
}
   
```


#### 장점 5. 리턴하는 객체의 클래스가 public static 팩토리 메소드를 작성할 시점에 반드시 존재하지 않아도 된다

이러한 유연함 덕에 서비스 제공자 프레임워크를 만들 수 있다.<br>
인터페이스나 클래스가 만들어지는 시점에서 하위 타입의 클래스가 존재하지 않아도 나중에 만들 클래스가 기존의 인터페이스나 클래스를 상속 받으면 언제든지 의존성을 주입 받아서 사용가능하다. <br>
반환값이 인터페이스가 되며 정적 팩터리 메서드이 변경없이 구현체를 바꿔 끼울 수 있다.

```java
public class ElectricCar extends Car {
    ...
}
public class Car {

  	public static Car getCar(int position) {
        Car car = new Car();
        
        // Car 구현체의 FQCN(Full Qualified Class Name)을 읽어온다
        // FQCN에 해당하는 인스턴스를 생성한다.
        // car 변수를 해당 인스턴스를 가리키도록 수정
        
        return car;
    }
}
```

- 서비스 인터페이스 : 구현체의 동작을 정의
- 서비스 등록 API : 제공자가 구현체를 등록할 때 사용
- 서비스 접근 API : 클라이언트가 서비스의 인스턴스를 얻을 때 사용
- 서비스 제공자 인터페이스 : 서비스 인터페이스의 인스턴스를 생성하는 팩토리 객체


클라이언트는 서비스 접근 API를 이용해서 원하는 구현체를 가져올 수 있다.

조건을 명시하지 않을 경우 기본 구현체 혹은 지원하는 구현체들을 돌아가며 반환한다.
이러한 서비스 접근 API가 서비스 제공자 프레임워크의 근간인 유여한 정적 팩토리 메소드의 실체다.
서비스 제공자 인터페이스가 없다면 리플렉션을 이용해서 구현체를 인스턴스화 역할을 수행하도록 한다.

> Ex. JDBC
```java
Class.forName("oracle.jdbc.driver.OracleDriver"); 
Connection conn = null; 
conn = DriverManager.getConnection("jdbc:oracle:thin:@localhost:1521:ORA92", "scott", "tiger"); 
```
- DriverManager.registerDriver : 제공자 등록 API 역할 (oracle, mysql ...)
- Connection  : 서비스 인터페이스 역할
- DriverManager.getConnection  : 서비스 접근 API 역할
- Driver : 서비스 제공자 인터페이스 역할

서비스 프로바이더 프레임워크는 서비스의 구현체를 대표하는 서비스 인터페이스와 구현체를 등록하는데 사용하는 프로바이더 등록 API 그리고 클라이언트가 해당 서비스의 인스턴스를 가져갈 때 사용하는 서비스 엑세스 API가 필수로 필요하다.

부가적으로, 서비스 인터페이스의 인스턴스를 제공하는 서비스 프로바이더 인터페이스를 만들 수도 있는데, 그게 없는 경우에는 리플랙션을 사용해서 구현체를 만들어 준다.

---

#### 단점 1: public 또는 protected 생성자 없이 static public 메소드만 제공하는 클래스는 상속할 수 없다

상속을 하게되면 super() 를 호출하며 부모 클래스의 함수들을 호출하게된다. 그러나 부모 클래스의 생성자가 private이라면 상속이 불가능하다.<br>
보통 정적 팩토리 메서드만 제공하는 경우 생성자를 통한 인스턴스 생성을 막는 경우가 많다.<br>
ex) Collections는 생성자가 private으로 구현되어 있기 때문에 상속할 수 없다.


#### 단점 2: 프로그래머가 static 팩토리 메소드를 찾는게 어렵다.

생성자는 Javadoc 상단에 모아서 보여주지만 static 팩토리 메소드는 API 문서에서 특별히 다뤄주지 않는다.<br>
따라서 클래스나 인터페이스 문서 상단에 팩토리 메소드에 대한 문서를 제공하는 것이 좋다.

---

#### 정적 팩터리 메서드 명명규칙


- from: 매개변수를 하나 받아서 해당 타입의 인스턴스를 반환하는 형변환 메서드
```java
Date d = Date.from(instant);
```
- of: 여러 매개변수를 받아 적합한 타입의 인스턴스를 반환하는 집계 메서드
```java
Set<Rank> faceCards = EnumSet.of(JACK, QUEEN, KING);
```
- instance / getInstance: 매개변수로 명시한 인스턴스를 반환, 같은 인스턴스임을 보장하진 않는다.
```java
StackWalker luke = StackWalker.getInstance(options);
```
- create / newInstance: instance 혹은 getInstance와 같지만, 매번 새로운 인스턴스 생성을 보장
```java
Object newArray = Array.newInstance(classObject, arrayLen);
```
- getType: getInstance와 같으나, 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할 때 사용
```java
FileStore fs = Files.getFileStore(path); 
```
- newType: newInstance와 같으나, 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할 때 사용
```java
BufferedReader br = Files.newBufferedReader(path);
```
- type: getType과 newType의 간결한 버전
```java
List<Complaint> litany = Collections.list(legacyLitany);
```

---

개인적으로 static 생성자를 자주 사용하는 부분

- Entity <-> DTO 등등
- parameter 에 따라 다른 응답값을 리턴해야할 때(분기로직이 필요할때)
- 객체 생성 시 필드 데이터 조작이 필요한 경우 (객체 생성 관련 역할은 해당 객체에 두려는데 맞는지는?)
