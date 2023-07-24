## Chapter 2. 의미 있는 이름

### 의도를 분명히 밝혀라

- 좋은 이름을 지으려면 시간이 걸리지만 좋은 이름으로 절약하는 시간이 훨씬 더 많다.
- 따로 주석이 필요하다면 의도를 분명히 드러내지 못했다는 말이다.

```java
public List<int []> getThem() {
	List<int []> list1 = new ArrayList<int []>();
	for (int[] x : theList)
		if (x[0] ==4)
			list1.add(x);
		return list1;
}
```

- 코드 맥락이 명시적으로 드러나지 않는다.

```java
public List<int []> getFlaggedCells() {
	List<int []> flaggedCells = new ArrayList<int []>();
	for (int [] cell : gameBoard)
		if(cell[STATUS_VALUE] == FLAGGED)
			flaggedCells.add(cell);
	return flaggedCells;
}
```

- 각 개념에 이름만 붙여도 코드가 상당히 나아진다.

```java
public List<Cell> getFlaggedCells() {
	List<Cell> flaggedCells = new ArrayList<Cell>();
	for (Cell cell : gameBoard)
		if(cell.isFlagged())
			flaggedCells.add(cell);
	return flaggedCells;
}
```

- 클래스와 상수를 이용해 단순히 이름만 고쳤는데도 함수가 하는 일을 이해하기 쉬워졌다.

### 그릇된 정보를 피하라

- 널리 쓰이는 의미가 있는 단어를 다른 의미로 사용해도 안된다. (hp, aix, sco...)
- 서로 흡사한 이름을 사용하지 않도록 주의한다.
- 소문자 `l`과 대문자 `O`, `1` 과 `0` 같이 비슷한 모양으로 혼란을 주는 변수는 피하자.

### 의미 있게 구분하라

- 연속된 숫자를 덧붙이거나 불용어(의미가 없는 단어)를 추가하는 방식은 적절하지 못하다.

```java
public static void copyChars(char a1[], char a2[]) {
	for (int i = 0; i< a1.lenght; i++) {
		a2[i] = a1[i];
}
```

- a1과 a2가 무엇을 의미하는지 알 수 없다.

  ```java
  public static void copyChars(char source[], char destination[]){
      for (int i = 0; i < source.length; i++){
          destination[i] = source[i];
      }
  }
  ```

  - 함수 인수 이름을 source와 destination으로 변경한다면 훨씬 더 코드 읽기가 쉬울 것이다.

### 발음, 검색하기 쉬운 이름을 사용하라

- 발음하기 어려운 이름은 토론하기도 어렵다.
- 이름 길이는 범위크기에 비례해야 한다.

```java
// Bad
for (int j=0; j<34; j++) {
     S += (t[j]*4)/5;
}

// Good
int realDaysPerldealDay = 4;
const int WORK_DAYS_PER_WEEK = 5;
int sum = 0;
for (int j=0; j < NUMBER_OF_TASKSi j++) {
    int realTaskDays = taskEstimate[j] * realDaysPerldealDaY;
    int realTaskWeeks = (realTaskDays / WORK_DAYS_PER_WEEK);
    sum += realTaskWeeks;
}
```

- sum 이라는 변수가 별로 유용하징는 않으나 검색이 가능하다.
- 이름을 의미있게 지으면 함수가 길어지지만 검색이 편해진다.

### 자신의 기억력을 자랑하지 마라

- 단순 반복 횟수가 아니라면 루프에서 i,j,k 등을 사용하지 마라.

### 클래스 이름

- `명사`나 `명사구`가 적합하다.
- Manager, Processor, Data, Info 등과 같은 단어는 피하고, 동사는 사용하지 않는다.

### 메서드 이름

- `동사`나 `동사구`가 적합하다.
- `접근자, 변경자, 조건자`는 javabean 표준에 따라 값 앞에 `get, set, is`를 붙인다.
- 생성자를 오버로딩할때는 인수를 설명하는 이름을 넣은 정적 팩토리 메서드를 사용한다.

### 한 개념에 한 단어를 사용하라

- 똑같은 메서드를 클래스마다 제각각 부르면 혼란스럽다.
- 하지만 일관성을 고려한다고 다른 개념에 같은 단어를 사용하면 안된다.

### 마치면서

- 코드를 개선하려는 노력을 중단해서는 안된다.
- 다른 사람이 짠 코드를 손본다면 리팩터링 도구를 사용해 문제 해결 목적으로 이름을 개선하라.

## Ref.

[Google Java Style Guide](https://google.github.io/styleguide/javaguide.html)
[캠퍼스 핵데이 Java 코딩 컨벤션](https://naver.github.io/hackday-conventions-java/)
