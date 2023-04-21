# Decorator pattern

## OCP(Open-Closed Principle)

클래스는 확장에 열려 있어야 하지만 변경에는 닫혀 있어야 한다.

## 데코레이터 패턴 정의

![image](https://user-images.githubusercontent.com/66015002/193959548-94253862-ad80-4f55-a8bc-808fd87e0bec.png)

- 데코레이터 패턴으로 객체에 추가 요소를 동적으로 더할 수 있다.
  - 데코레이터는 자신이 장식하고 있는 객체에게 어떤 행동을 위임하는 일 말고도 추가 작업을 수행할 수 있다.
- 데코레이터를 사용하면 서브클래스를 만들 때보다 훨씬 유연하게 기능을 확장할 수 있다.
  - 한 객체를 여러 개의 데코레이터로 감쌀 수 있다.
  - 객체는 언제든지 감쌀 수 있으므로 실행 중에 필요한 데코레이터를 마음대로 적용할 수 있다.

|   장점   | OCP에 충실하여 기능 확장에 유연하며 코드 수정이 불필요하다                                                                             |
| :------: | :------------------------------------------------------------------------------------------------------------------------------------- |
| **단점** | **클래스가 많아진다.(남들이 처음 볼 경우 이해하기가 힘든 복잡한 구조가 나온다.) <br> 구성 요소를 초기화할 때 코드가 훨씬 복잡해진다**. |


#### 방법

1. 비즈니스 도메인이 그 위에 여러 선택적 레이어가 있는 기본 구성 요소(primary component)로 표시될 수 있는지 확인
2. 기본 구성 요소와 선택적 레이어(데코레이터) 모두에 공통적인 방법을 파악
   - 구성 요소 인터페이스(component interface)를 만들고 거기에서 해당 메서드를 선언
3. 구체적인 구성 요소 클래스를 만들고 기본 동작을 정의
4. 기본 데코레이터 클래스를 생성
   - 래핑된 개체에 대한 참조를 저장하기 위한 필드가 있어야 함
   - 필드는 데코레이터 뿐만 아니라 구체적인 구성 요소에 연결할 수 있도록 구성 요소 인터페이스 유형으로 선언되어야
   - 기본 데코레이터는 모든 작업을 래핑된 개체에 위임
5. 모든 클래스가 구성 요소 인터페이스를 구현하는지 확인
6. 기본 데코레이터에서 확장하여 구체적인 데코레이터를 생성
   - 구체적인 데코레이터는 부모 메서드(항상 래핑된 개체에 위임) 호출 전후에 동작을 실행
7. 클라이언트 코드는 데코레이터를 만들고 클라이언트가 필요로 하는 방식으로 구성

## 데코레이터 적용 예: 자바I/O

![image](https://user-images.githubusercontent.com/66015002/193959499-5ce4e39d-f8e2-4768-a2d4-6386e3750c43.png)

### FilterInputStream의 소스 구조

```java
public class FilterInputStream extends InputStream {
    protected volatile InputStream in;
    // 접근제어자가 proteced이므로 생성자를 사용할 수 없습니다.
    protected FilterInputStream(InputStream in){
        this.in = in;
    }
    public int read() throws IOException{
        return in.read();
    }
    ...
}
```

#### 데코레이터 패턴과 프록시 패턴

| | Decorator	| Proxy |
|:--:|:--:|:--:|
|공통점|	Interface 구현 구조로 이루어짐<br>Wrapper Class와 real class의 관계가 aggregation ( has A )관계|
|차이점	|Wrapper Class와 Real Class의 관계가 런타임에 정해짐	| Wrapper Class와 Real Class의 관계가 컴파일타임에 정해짐|
|사용 목적|runtime에 real Object에 기능을 확장하고 싶을 때 사용<br>기능 확장 관점|Real Object에 대해서 실제로 필요할 때 instance가 생성되고 작업이 진행될 수 있도록 하기 위해 적용<br>기능 제어, 접근 제어(controll) 관점|



#### 프로젝트의 데코레이터 (RESTDOC)
데코레이터 패턴을 직접적으로 사용하는 곳이 거의 없음
RestDoc 문서를 만들기 위해 기존 파라미터에 어노테이션, 기타 환경 변수 등에 설정된 값을 더함


`Abstract Class`
```java
public abstract class FieldsBase {
 
   protected List<FieldDescriptor> fieldsList = Lists.newArrayList();
 
   public List<FieldDescriptor> getFieldsDescriptor(String... prefix) {
      return fieldsList;
   }
}
Impl Class
public class EmptyFields extends FieldsBase {
 
  public EmptyFields() {}
 
  @Override
  public List<FieldDescriptor> getFieldsDescriptor(String... prefix) {
    return fieldsList;
  }
}
```
`Abstract Decorator Class`
```java
public abstract class FieldsDecorator extends FieldsBase {
 
  protected FieldsBase fieldsBase;
 
  protected String description;
 
  public abstract List<FieldDescriptor> getFieldsDescriptor(String... prefix);
}
Decorator Class
public class ProductNoField extends FieldsDecorator {
 
   public ProductNoField(FieldsBase fieldsBase) {
      this.fieldsBase = fieldsBase;
   }
 
   @Override
   public List<FieldDescriptor> getFieldsDescriptor(String... prefix) {
      List<FieldDescriptor> list = fieldsBase.getFieldsDescriptor(prefix);
      list.add(
         fieldWithPath(prefix[0] + "product_no").description(RestdocsDescriptions.PRODUCT_NO)
      );
 
      return list;
   }
}
```

## 구현 관련

```
go
├── component
│      ├── beverage.go     // Component interface
│      ├── dark_roast.go   // Concrete component
│      ├── decaf.go        // Concrete component
│      ├── espresso.go     // Concrete component
│      └── house_blend.go  // Concrete component
│
├── decorator
│      ├── milk.go         // Concrete decorator
│      ├── mocha.go        // Concrete decorator
│      ├── soy.go          // Concrete decorator
│      └── whip.go         // Concrete decorator
│
├── go.mod
└── main.go
```

## Reference

- https://refactoring.guru/design-patterns/decorator
- https://github.com/alex-leonhardt/go-decorator-pattern
- https://levelup.gitconnected.com/the-decorator-pattern-in-go-66ed951b0f7c
