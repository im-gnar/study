# Chapter4. 주석

 - 코드로 의도를 잘 표현할 수 있다면 주석은 필요하지 않다. 주석이 필요한 때는 실패했다는걸 의미한다.
 - 코드는 변화하고 진화하지만, 주석이 언제나 코드를 따라가지는 못한다.

### 주석은 나쁜 코드를 보완하지 못한다.

코드에 주석을 추가하는 일반적인 이유는 코드 품질이 나쁘기 때문이다.

코드가 이해하기 힘들다면 주석을 다는게 아니라 코드를 정리해라.

### 코드로 의도를 표현하라!

```java
// 직원이 복지 혜택을 받을 자격이 있는지 검사
if ((employee.flags & HOURLY_FLAG) && (employee.age > 65)) {
  // DO SOMETHING..
}

if (employee.isEligibleForFullBenenfits()){
  // DO SOMETHING..
}
```
위 코드는 아래와 같이 메서드명으로도 충분히 표현이 가능하다.

### 좋은 주석

**법적인 주석**
```java
// Copyright (C) 2003,2004,2005 by Object Metor, Inc. All rights reserved.
// GNU General Public License 버전 2 이상을 따르는 조건으로 배포한다.   
```
- 저작권, 소유권, 계약조건, 표준라이선스 등 법적인 정보

**정보를 제공하는 주석**
```java
// 테스트 중인 Responder 인스턴스를 반환한다.
protected abstract Responder responderInstance();

// kk:mm:ss EEE, MMM dd, yyyy 형식이다.   
pattern timeMatcher = Pattern.compile("\\d*:\\d*:\\d* \\w*, \\w* \\d*, \\d*"); 
```
- 메서드의 대한 설명은 주석보다는 메서드 명으로 정보를 전달하는 편이 더 좋다.
- 시간, 날짜를 변환하는 클래스를 만들어 코드를 옮겨주는게 더 좋다.

**의도를 설명하는 주석**
```java
public int compareTo(Object o) {
    if(o instanceof WikiPagePath)
    {
        WikiPagePath p = (WikiPagePath) o;
        String compressedName = StringUtil.join(names, "");
        String compressedArgumentName = StringUtil.join(p.names, "");
        return compressedName.compareTo(compressedArgumentName);
    }
    return 1; // 오른쪽 유형이므로 정렬 순위가 더 높다.
}
```
```java
public void testConcurrentAddWidgets() throws Exception {
    WidgetBuilder widgetBuilder = new WidgetBuilder(new Class[]{BoldWidget.class});
    String text = "'''bold text'''";
    Parent parent = new BoldWidget(new MockWidgetRoot(), "'''bold text'''");
    AtomicBoolean failFlag = new AtomicBoolean();
    failFlag.set(false);

    // 스레드를 대량 생성하는 방법으로 어떻게든 RaceCondition을 만들려 시도한다.  
    for(int i=0; i < 25000; i++){
         WidgetBuilderThread widgetBuilderThread = new WidgetBuilderThread(widgetBuilderThread, text, parent, failFlag);
         Thread thread = new Thread(widgetBuilderThread);
         thread.start();
    }
    assertEquals(false, failFlag.get());
}
```
- 문제 해결방식에 동의하지 않더라도 저자의 의도는 분명히 드러난다.

**의미를 명료하게 밝히는 주석**
```java
public void tempCompareTo() throws Exception {
   WikikPage a = PathParse.parse("PageA");
   WikikPage ab = PathParse.parse("PageA.PageB");
   WikikPage b = PathParse.parse("PageB");
   WikikPage aa = PathParse.parse("PageA.PageA");
   WikikPage bb= PathParse.parse("PageB.PageB");
   WikikPage ba = PathParse.parse("PageB.PageA");

   assertTrue(a.compareTo(a) == 0)   // a == a
   assertTrue(a.compareTo(b) != 0)   // a != b
   assertTrue(a.compareTo(ab) == 0)  // ab == ab
   assertTrue(a.compareTo(b) == -1)  // a < a
   assertTrue(a.compareTo(ab) == -1) // aa < ab
   assertTrue(a.compareTo(bb) == -1) // ba < bb
   assertTrue(a.compareTo(a) == 1)   // b > a
   assertTrue(a.compareTo(aa) == 1)  // ab > aa
   assertTrue(a.compareTo(ba) == 1)  // bb > ba
}
```

- 인수나 반환값 자체를 명확하게 만들면 좋지만 표준 라이브러리 등 변경하지 못하는 코드라면 주석을 사용하는 것이 좋다.
- 그릇된 주석을 달아놓을 위험은 상당히 높으니 위와 같은 주석을 달 때는 더 나은 방법이 없는지 고민하고 주의하자.


**결과를 경고하는 주석**
```java
// 여유 시간이 충분하지 않다면 실행하지 마십시오.
public void _testWithReallyBigFile() {
    writeLinesToFile(10000000);
    ...
}
```
- 요즘은 @Ignore 속성을 이용해 테스트 케이스를 꺼버리고 구체적인 설명은 @Ignore 속성에 문자열로 넣어준다
- JUnit4가 나오기 전에는 메서드 이름 앞에 _ 기호를 붙이는 방법이 일반적인 관례

```java
@Ignore("실행이 너무 오래 걸린다")
public void _testWithReallyBigFile() {
 ...
}

public static SimpleDateFormat makeStandardHttpDateFormat() {
    // SimpleDateFormat은 스레드에 안전하지 못하다.
    // 따라서 각 인스턴스를 독립적으로 생성해야 한다.
    SimpleDateFormat df = new SimpleDateFormat("EEE, dd MMM yyyy HH:mm:ss z");
    df.setTimeZone(TimeZone.getTimeZone("GMT"));
    return df;
}
```
- 효율을 위해 정적 초기화 함수를 사용하려다 주석 덕분에 실수를 면할 수 있었다.
- `@Immutable`, `@ThreadSafe`, `@NotThreadSafe` 등 병렬 쓰레드에서 안정성을 위한 어노테이션도 있음(Effective Java item 82 스레드안전성)

**TODO 주석**
```java
// TODO-MoM 현재 필요하지 않다. 
// 체크아웃 모델을 도입하면 함수가 필요 없다.  
protected VersionInfo makeVersion() throws exception{
 return null;
}
```
- `TODO` : 필요하지만 당장 구현하기 어려운 업무.
- `FIXME` : 문제가 있지만 당장 수정할 필요는 없을 때

**중요성을 강조하는 주석**
```java
String listItemContent = matchgroup(3).trim();
// 여기서 trim은 정말 중요하다. trim 함수는 문자열에서 시작 공백을 제거한다.
// 문자열에 시작 공백이 있으면 다른 문자열로 인식되기 때문이다.
new ListItemWidget(this, listItemContent, this.level + 1);
return buildList(text.substring(match.end());
```

---

### 나쁜 주석

**주절거리는 주석**

```java
public void loadProperties() {
    try {
        String propertiesPath = propertiesLocation + "/" + PROPERTIES_FILE;
        FileIOnputStream propertiesStream = new FileInputStream(propertiesPath);
        loadedProperties.load(propertiesStream);
	  } catch(IOException e) {
        // 속성 파일이 없다면 기본값을 모두 메모리로 읽어 들였다는 의미다.
	  }
}
```

- 해당 주석은 저자에게야 의미가 있겠지만 다른 사람들에게는 전해지지 않는다.
- 답을 알아내기 위해선 다른 코드를 뒤져야하며, 이 주석은 독자와 제대로 소통하지 못하는 주석이다.

**같은 이야기를 중복하는 주석**
```java
// this.closed가 true일 때 반환되는 유틸리티 메서드다.
// 타임아웃에 도달하면 예외를 던진다.
public synchronized void waitForClose(final long timeoutMillis) throws Exception {
    if(!closded) {
        wait(timeoutMillis);
        if(!closed)
            throw new Exception("MockResponseSender could not be closed");
    }
}
```
이 주석은 주석이 코드보다 더 많은 정보를 제공하지 못한다.

**오해할 여지가 있는 주석**
- 같은 이야기를 중복하는 주석의 코드는 중복이 많으면서도 오해할 여지가 있다.
- 위 예제는 this.closed가 true로 변하는 순간에 메서드를 반환하는 것이 아니라, true 여야 메서드가 반환된다.

**의무적으로 다는 주석**

```java 
/**
 *
 * @Param title CD 제목
 * @Param author CD 저자
 * @Param tracks CD 트랙 숫자
 * @Param durationInMinutes CD 길이(단위 : 분)
 */
public void addCD(String title, String author, int tracks, int durationMinutes) {
    CD cd = new CD();
    cd.title = title;
    cd.author = author;
    cd.tracks = tracks;
    cd.duration = durationMinutes;
    cdList.add(cd);
}
```
- 모든 함수, 변수에 Javadocs, 주석을 달아야 한다는 규칙은 오히려 코드만 헷갈리게 만들 수 있다.

**이력을 기록하는 주석**

```java
/**
 * 변경 이력 (11-Oct-2001부터)
 * _________________________
 * 11-Oct-2001 : 클래스를 다시 정리하고 새로운 패키지인
 *               com.jrefinery.date로 옮겼다.(DG);
 * 05-Nov-2001 : getDescription() 메서드를 추가했으며 
 *               NotableDate class를 제거했다. (DG);
 * 12-Nov-2001 : IBD가 setDescription() 메서드를 요구한다.
 *               Notable 클래스를 없앴다. (DG);
 * ....
 */
```

- 소스 코드 관리 시스템이 없었을 때는 코드에 이력(Changelog)을 표시하는 것이 바람직했다.
- 하지만 이젠 혼란만 가중할 뿐이므로 완전히 제거하자.

**있으나 마나 한 주석**

```java
/**
 * 기본 생성자
 */
protected AnnualDateRule() {    
}

try {
    ...
} catch(Exception e) {
    // 이게 뭐야!  
}
```

  
**함수나 변수로 표현할 수 있다면 주석을 달지 마라**
```java
// 전역 목록 <smodule>에 속하는 모듈이 우리가 속한 하위 시스템에 의존하는가?
if (smodule.getDependSubsystems().contains(subSysMod.getSubSystem()))...

ArrayList moduleDependees = smodule.getDependSubsystems();
String ourSubSystem = subSysMod.getSubSystem();
if (moduleDependees.contains(ourSubSystem))...
```
- 위 코드는 주석을 없애고 아래처럼 표현 가능

**위치를 표시하는 주석**
```java
// Actions //////////////////////////////
```
- 이 기능은 일반적으로 잡음이나, 눈에 띄어야 할 때 등 필요할 때만 드물게 사용하는 편이 좋다.

**닫는 괄호에 다는 주석**
```java
try {
     while (true) {
         
     }	//while
} // try
```
- 중첩이 심하고 장황한 함수라면 의미가 있을지도 모르지만 작고 캡슐화된 함수에는 불필요하다.
- 닫는 괄호에 주석을 달아야겠다는 생각이 든다면 대신에 함수를 줄이자.

**공로를 돌리거나 저자를 표시하는 주석**
```java
/* koiil 이 추가함 */
```
- 이제는 소스 코드 관리 시스템으로 누가 언제 무엇을 추가했는지 알 수 있다.
  
**주석으로 처리한 코드**
```java
InputStreamResponse reponse = new inputStreamREsponse();
...
// InputStream resultsStream = formatter.getResultStream();
```

- 주석으로 처리된 코드는 다른 사람들이 지우기를 주저한다. 이유가 있어서 남겨놓았으리라고 생각하기 때문이다.
- 이젠 모든 코드 기록은 형상 관리에서 처리하기 때문에 과감하게 지우는 것이 좋다.

**전역 정보**

```java
/**
 * 적합성 테스트가 동작하는 포트: 기본값은 <b>8082</b>
 * 
 * @param fitnessPort
 */
public void setFitnessePort(int fitnessePort) {
    this.fitnessePort = fitnessPort;
}
```

- 주석을 달아야 한다면 근처에 있는 코드만 기술하라. 
- 위 코드는 포트값을 통제하지 않기 때문에, 위 주석은 코드를 설명하는 것이 아니라 시스템 어딘가에 있는 다른 함수를 설명한다 뜻

**모호한 관계**
주석과 주석이 설명하는 코드는 둘 사이 관계가 명백해야 한다.
```java
/*
 * 모든 픽셀을 담을 만큼 충분한 배열로 시작한다(여기에 필터 바이트를 더한다).
 * 그리고 헤더 정보를 위해 200바이트를 더한다.
 */
this.pngBytes = new byte[((this.width + 1) * this.height * 3) + 200];
```

- 여기서 필터 바이트란 무엇일까? +1과 관련이 있을까? 200을 추가하는 이유는?
- 주석을 다는 목적은 코드만으로 설명이 부족해서다. 주석 자체가 다시 설명을 요구해선 안된다.
