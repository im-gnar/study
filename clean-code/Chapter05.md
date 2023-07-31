# Chapter5. 형식 맞추기

> 프로그래머라면 형식을 깔끔하게 맞춰 코드를 짜야한다.


### 형식을 맞추는 목적

- 오늘 구현한 코드가 다음 버전에서 바뀔 확률은 아주 높다.
- 구현 스타일과 가독성 수준은 시간이 지나도 유지보수 용이성과 확장성에 계속 영향을 미친다.
- 원래 코드는 사라질지라도 개발자 자신의 스타일과 규율은 사라지지 않는다.

### 적절한 행 길이를 유지하라

- 큰 파일보다 작은 파일이 이해하기 쉽다.
- 200줄 미만의 코드로도 충분히 크고 복잡한 시스템을 만들 수 있다.

### 신문 기사처럼 작성하라

- 이름은 간단하면서도 설명이 가능하게 짓는다.
- 이름만 보고도 올바른 모듈을 살펴보고 있는지 아닌지를 판단할 정도로 신경 써서 짓는다.
- 아래로 내려갈수록 의도를 세세하게 묘사한다.
  
### 개념은 빈 행으로 분리하라

- 각 행은 수식이나 절을 나타내고, 일련의 행 묶음은 완결된 생각 하나를 표현한다. 그렇기에 생각 사이는 빈 행을 넣어 분리해야 마땅하다.
- 코드를 읽어 내려가다 보면 빈 행 바로 다음 줄에 눈길이 멈춘다.

```java
package fitnesse.wikitext.widgets;

import java.util.regex.*;

public class BoldWidget extends ParentWidget {
    public static final String REGEXP = "'''.+?'''";
    private static final Pattern pattern = Pattern.compile("'''(.+?)'''",
      Pattern.MULTILINE + Pattern.DOTALL
    );
    
    public BoldWidget(ParentWidget parent, String text) throws Exception {
        super(parent);
        Matcher match = pattern.matcher(text);
        match.find();
        addChildWidgets(match.group(1));
    }
}
```

### 세로 밀집도

- 줄바꿈은 개념 분리, 세로 밀집도는 연관성을 의미한다.
- 밀접한 코드는 세로로 가까이 놓여 한눈에 들어오도록 의미없는 주석 등은 제거해야한다.
  
```java
public class ReporterConfig {
    /**
      * 리포터 리스너의 클래스 이름
      */
    private String m_className;
    
     /**
      * 리포터 리스너의 속성
      */
    private List<Property> m_properties = new ArrayList<Property>();
    public void addProperty(Property property) {
        m_properties.add(property);
    }
}

public class ReporterConfig {
    private String m_className;
    private List<Property> m_properties = new ArrayList<Property>();
    
    public void addProperty(Property property) {
        m_properties.add(property);
    }
}
```

### 수직 거리

- 서로 밀접한 개념은 한 파일에 속해야 마땅하다.
- 굳이 protected를 달면서 같은 패키지 안에서 쓰지 말고, 이미 연관성 있는 것이므로 한 파일내에 쓰는 것이 바람직하다

### 변수 선언

- 변수는 사용하는 위치에 최대한 가까이 선언한다.
- 지역 변수는 각 함수 맨 처음에 선언한다.
- 루프를 제어하는 변수는 흔히 루프 문 내부에 선언한다.

### 인스턴스 변수

```java
public static class test{
    int instanceVar; // 인스턴스 변수
    static int classVar; //클래스 변수
}
```
- Java 는 클래스 맨 처음에 선언한다.
- 맨앞이든 맨뒤든은 중요한게 아니고 변수를 모아서 세로로 거리를 두지 않는다는 것이다.

### 개념적 유사성

```java
public class Assert {
    static public void assertTrue(String message, boolean condition){
        if(!condition)
            fail(message);
    }
    
    static public void assertTrue(boolean condition){
        assertTrue(null, condition);
    }
    
    static public void assertFalse(String message, boolean condition){
        assertTrue(message, !condition);
    }
    
    static public void assertFalse(boolean condition){
        assertFalse(null, condition);
    }
}
```
- 종속 함수, 비슷한 동작을 하는 함수, 명명법이 똑같고 기본 기능이 유사한 함수 등을 개념적인 친화도가 높다고 한다.

### 세로 순서

- 호출되는 함수를 호출하는 함수보다 나중에 배치하면 소스 코드 모듈이 고차원에서 저차원으로 자연스럽게 내려간다.

### 가로 형식 맞추기

- 행 길이는 짧은 편이 바람직하며, 80자까지는 아니라도 100~120자 정도 내로 유지하는게 좋다.

### 가로 공백과 밀집도

```java
private void measureLine(String line) {
    lineCount++;
    
    // 할당 연산자를 강조하기 위해 앞 뒤에 공백을 줬다.
    // 연산자 좌우로 공백을 주어 두 가지 주요 요소가 확실히 나뉜다는 사실이 더욱 분명해졌다.
    int lineSize = line.length();
    totalChars += lineSize;
    
    // 함수 이름과 이어지는 괄호 사이에 공백을 주지 않음으로써 함수와 인수의 밀접함을 보여준다.
    // 괄호 안 인수는 공백으로 분리하고, 쉼표를 강조하여 인수가 별개라는 사실을 보여준다.
    lineWidthHistogram.addLine(lineSize, lineCount);
    recordWidestLine(lineSize);
}

public class Quadratic {
    // 연산자 우선순위를 강조하기 위해서도 공백을 사용한다.
    public static double root1(double a, double b, double c) {
        double determinant = determinant(a, b, c);
        return (-b + Math.sqrt(determinant)) / (2*a); // 공백 사용
    }
    ...
}
```

### 가로 정렬

```java
// bad
public class FitNesseExpediter implements ResponseSender {
    private Socket          socket;
    private InputStream     input;
    private OutputStream    output;
    private long            requestProgress;
  
  	this.context = context;
    socket       = s;
}

// good
public class FitNesseExpediter implements ResponseSender {
    private Socket socket;
    private InputStream input;
    private OutputStream output;
    ...
}
```
- 코드가 엉뚱한 부분을 강조해 진짜 의도가 가려지기 때문에 좋은 구조는 아니다.
- 정렬이 필요한 정도로 목록이 길다면 클래스를 쪼개야 한다는 의미다.
  
### 들여쓰기

- 범위(scope)로 이루어진 계층을 표현하기 위해 코드를 들여쓴다.
- 들여쓰기한 파일은 구조가 한눈에 들어온다.
- 변수, 생성자 함수, 접근자 함수, 메서드가 금방 보인다.
- 간단한 if문, 짧은 함수라도 들여쓰기를 무시하지 말자.

### 가짜 범위

```java
while (dis.read(buf, 0, readBufferSize) != -1)
;
```
- 빈 while문이나 for문을 접했을 경우에, 세미콜론(;)은 새 행에다 넣어줘야 눈에 띈다.

  
### 팀 규칙

- 팀은 한 가지 규칙에 합의해야 하고, 모든 팀원은 그 규칙을 따라야 한다.
- 스타일은 일관적이고 매끄러워 한 소스파일에서 봤던 형식이 다른 소스 파일에도 쓰이리라는 신뢰감을 독자에게 줘야한다.
