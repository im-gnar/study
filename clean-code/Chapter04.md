# Chapter4. 주석

 - 코드로 의도를 잘 표현할 수 있다면 주석은 필요하지 않다. 주석이 필요한 때는 실패했다는걸 의미한다.
 - 코드는 변화하고 진화하지만, 주석이 언제나 코드를 따라가지는 못한다.

### 주석은 나쁜 코드를 보완하지 못한다.

코드에 주석을 추가하는 일반적인 이유는 코드 품질이 나쁘기 때문이다.

코드가 이해하기 힘들다면 주석을 다는게 아니라 코드를 정리해라.

### 코드로 의도를 표현하라!

```java
// 직원이 복지 혜택을 받을 자격이 있는지 검사
if ((employee.flags & HOURLY_FLAG) && (employee.age > 65)){
  // DO SOMETHING..
}

if (employee.isEligibleForFullBenenfits()){
  // DO SOMETHING..
}
```

### 좋은 주석

법적인 주석
```java
// Copyright (C) 2003,2004,2005 by Object Metor, Inc. All rights reserved.
// GNU General Public License 버전 2 이상을 따르는 조건으로 배포한다.   
```
- 저작권, 소유권, 계약조건, 표준라이선스 등 법적인 정보

정보를 제공하는 주석
```java
// 테스트 중인 Responder 인스턴스를 반환한다.
protected abstract Responder responderInstance();

// kk:mm:ss EEE, MMM dd, yyyy 형식이다.   
pattern timeMatcher = Pattern.compile("\\d*:\\d*:\\d* \\w*, \\w* \\d*, \\d*"); 
```
- 메서드의 대한 설명은 주석보다는 메서드 명으로 정보를 전달하는 편이 더 좋다.
- 시간, 날짜를 변환하는 클래스를 만들어 코드를 옮겨주는게 더 좋다.

의도를 설명하는 주석
```java
public int compareTo(Object o)
{
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
