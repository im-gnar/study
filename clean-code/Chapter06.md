# Chapter6. 객체와 자료구조

- 변수를 비공개(private)로 정의하는 이유
  - 남들이 변수에 의존하지 않게 만들고 싶어서
  - 변수 타입이나 구현을 맘대로 바꾸고 싶어서
 
### 자료 추상화

- 구현을 감추려면 추상화가 필요하다.
- 변수 사이에 함수라는 계층을 넣는다고 구현이 저절로 감춰지지는 않는다. (@Getter, @Setter..)
- 추상 인터페이스를 제공해 사용자가 구현을 모른 채 자료의 핵심을 조작할 수 있어야 진정한 의미의 클래스다.

```java
// concrete
// 명확하게 직교 좌표계를 쓴다는 것을 알 수 있고, 개별적으로 좌표값을 읽고 설정
public class Point {
	public double x;
	public double y;
}

// abstract
// 클래스 메서드가 접근 정책을 강제한다
public interface Point {
	double getX();
	double getY(); // 조회는 각각 가능하지만
	void setCartesian(double x, double y); // 2개의 값을 동시에 설정하도록 강제한다.
	double getR();
	double getTheta();
	void setPolar(double r, double theta);
```

- 자료를 세세하게 공개하기보다는 추상적인 개념으로 표현하는 편이 좋다.
- 아무 생각 없이 조회/설정 함수를 추가하는 방법이 가장 나쁘다.

```java
// concrete
// 변수 값을 읽어 그대로 반환
public interface Vehicle {
	public getFuelThankCapacityInGallons();
	public getGallonsOfGasoline();
}

// abstract
// 변수값은 노출하지 않고 백분율이라는 추상적 개념으로 반환
public interface Vehicle {
	double getPercentFuelRemaining();
}
```

> Q. 일반적으로 개발할 때 실제 값을 가져와 함수에 맞게 사용하는데, 추상으로 돌려주면 인터페이스에 클래스별로 필요한 모든 함수를 지정? 인터페이스도 잘 쪼개야한다는?

### 자료/객체 비대칭

- 객체 : 추상화 뒤로 자료를 숨긴채 자료를 다루는 함수만 공개
- 자료구조 : 자료를 그대로 공개하며 별다른 함수는 제공하지 않음

**절차지향적 클래스**
```java
public class Square { 
	public Point topLeft; 
	public double side;
}

public class Rectangle { 
	public Point topLeft; 
	public double height; 
	public double width;
}

public class Circle { 
	public Point center; 
	public double radius;
}

public class Geometry {
	public final double PI = 3.141592653589793;

	public double area(Object shape) throws NoSuchShapeException {
		if (shape instanceof Square) { 
			Square s = (Square)shape; 
			return s.side * s.side;
		} else if (shape instanceof Rectangle) { 
			Rectangle r = (Rectangle)shape; 
			return r.height * r.width;
		} else if (shape instanceof Circle) {
			Circle c = (Circle)shape;
			return PI * c.radius * c.radius; 
		}
		throw new NoSuchShapeException(); 
	}
}
```
- 각 도형 클래스는 간단한 자료구조이며 아무 메서드도 제공하지 않는다.
- 실제 도형이 동작하는 방식은 Geometry 클래스에서 구현한다.


**객체지향적 클래스**
```java
public class Square implements Shape { 
	private Point topLeft;
	private double side;

	public double area() { // polymorphic method
		return side * side;
	} 
}

public class Rectangle implements Shape { 
	private Point topLeft;
	private double height;
	private double width;

	public double area() { 
		return height * width;
	} 
}

public class Circle implements Shape { 
	private Point center;
	private double radius;
	public final double PI = 3.141592653589793;

	public double area() {
		return PI * radius * radius;
	} 
}
```
- 각 도형은 도형에 맞는 계산 메서드가 있다.
- 즉, 실제 도형이 동작하는 방식은 본인의 클래스에서 구현한다.


|  | 도형 추가  |  함수 추가|
| :--: | :-- |:-- |
|객체지향| 기존 함수 영향X, 새 클래스 추가| 도형 클래스 전부를 수정|
|절차지향| 도형 클래스 수정X | 모든 함수를 수정|

> 최근 친구랑 얘기하며 "자바에서 MVC, 클래스를 쓴다면 그건 객체지향이 아니라 절차지향이다" 라고 한적이 있는데 처음엔 이해 못햇지만 이 장을 읽으며 좀 알것도 같은 느낌


### 디미터 법칙

> 디미터 법칙 : 모듈은 자신이 조작하는 개체의 속사정을 몰라야 한다.

**기차충돌**

```java
// Bad
final Stirng outputDir = ctxt.getOptions().getScratchDir().getAbsolutePath();

// Good
Options opts = ctxt.getOptions();
File scratchDir = opts.getScratchDir();
final String outputDir = scratchDir.getAbsolutePath();
```
- ctxt 객체가 Options를 포함하며, Options가 ScratchDir를, ScratchDir가 AbsolutePath를 포함한다는 사실을 알 수 있다.
- 각각이 객체라면 내부 구조를 숨기지 못햇으므로 디미터 위반
- 자료구조라도 내부 구조를 노출시키므로 위반

```java
final String outputDir = ctxt.options.scratchDir.absolutePath;
```
- 자료 구조라면 위와 같이 공개 변수만 포함하기 때문에 디미터 법칙을 거론할 필요가 없음

> Chaining 과의 차이점
> 위 예제는 "반환받은 객체" 의 메서드를 사용하지만, 체이닝은 "자기 자신의 객체(this)"를 리턴해서 사용하기 때문에 디미터 법칙 준수

**잡종구조**

- 절반은 객체, 절반은 자료 구조인 잡종 구조
- 잡종 구조는 중요한 기능을 수행하는 함수도 있고, 공개 변수나 공개 조회/설정 함수도 있다.
- 잡종 구조는 새로운 함수는 물론이고 새로운 자료 구조도 추가하기 어려운 양쪽 단점만 모아놓은 구조

**구조체 감추기 - 객체 지향적 방법**

```java
ctxt.getAbsolutePathOfScratchDirectoryOption(); // ctxt객체에 공개해야 하는 메서드가 너무 많아진다
   
ctxt.getScratchDirectoryOption().getAbsolutePath(); // getScratchDirectoryOption()이 객체가 아니라 자료 구조를 반환한다고 가정
```

- ctxt가 객체라면 '무엇인가를 하라' 라고 말해야지 조회를 하라고 말하면 안 된다.
- 함수의 목적을 파악해 객체에게 맞는 임무를 맞겨라.

```java
BufferedOutputStream bos = ctxt.createScratchFileStream(classFileName);
```

### 자료 전달 객체(DTO)

- 자료 구조체의 전형적인 형태로 '공개 변수'만 있고 함수가 없는 클래스
- class 내에 함수가 있으면 객체
- DTO에 get/set 은 자료구조체

**활성 레코드**

- DTO의 특수한 형태
- 공개 변수, 비공개 변수 + 조회/설정 함수가 있는 자료구조지만, save나 find와 같은 탐색 함수도 제공
- 활성 레코드에 비즈니스 규칙 메서드를 추가해 자료구조를 객체로 취급하면 자료구조도 아니고 객체도 아닌 잡종 구조가 나오기 때문에 바람직하지 못하다.
  - 활성 레코드는 자료 구조로 취급
  - 비즈니스 규칙을 담으면서 내부 자료를 숨기는 객체는 따로 생성한다.


### 결론

- 객체는 동작을 공개하고 자료를 숨긴다.
  - 기존 동작을 변경하지 않으면서 새 객체 타입을 추가하기는 쉽지만, 기존 객체에 새 동작을 추가하기는 어렵다.
- 자료구조는 별다른 동작 없이 자료를 노출한다.
  - 기존 자료 구조에 새 동작을 추가하기는 쉽지만, 새 자료 구조를 추가하기는 어렵다.
- 새로운 자료 타입을 추가하는 유연성이 필요하다면 객체가 더 적합
- 새로운 동작을 추가하는 유연성이 필요하다면 자료 구조와 절차적인 코드가 더 적합
