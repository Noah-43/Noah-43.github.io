---
title: EffectiveJava_Chap9
excerpt: 이펙티브 자바 책 9장 정리
categories: study
---

# 9장. 일반적인 프로그래밍 원칙

## Item 57. 지역변수의 범위를 최소화하라

지역변수의 유효 범위를 최소로 줄이면 코드 가독성과 유지보수성이 높아지고 오류 가능성은 낮아진다. 자바에서는 문장을 선언할 수 있는 곳이면 어디서든 변수를 선언할 수 있다. **지역변수의 범위를 줄이는 가장 강력한 기법은 가장 처음 쓰일 때 선언하는 것이다.** 또한 **거의 모든 지역변수는 선언과 동시에 초기화해야 한다.** 단, try-catch문은 예외다. 변수를 초기화하는 표현식에서 검사 예외를 던질 가능성이 있다면 try 블록 안에서 초기화하고, 변수 값을 try 블록 바깥에서도 사용해야 한다면 try 블록 앞에서 선언해야 한다.

반복문에서는 반복 변수의 값을 반복문이 종료된 뒤에도 써야 하는 상황이 아니라면 while 문보다는 for 문을 쓰는 편이 낫다. 컬렉션 순회는 다음과 같이 하는 것을 권장한다.

```java
// 컬렉션이나 배열을 순회하는 권장 관용구
for (Element e : c) {
  ... // e로 무언가를 한다.
}
```

반복자를 사용해야 하는 상황이면 for-each 문 대신 전통적인 for 문을 쓰는 것이 낫다.

```java
// 반복자가 필요할 때의 관용구
for (Iterator<Element> i = c.iterator(); i.hasNext(); ) {
  Element e = i.next();
  ... // e와 i로 무언가를 한다.
}
```

while 문은 for 문과 달리 복사해 붙여넣기 오류가 발생할 수 있다.

```java
Iterator<Element> i = c.iterator();
while (i.hasNext()) {
  doSomething(i.next());
}
...
  
Iterator<Element> i2 = cw.iterator();
while (i.hasNext()) {				// 버그!
  doSomethingElse(i2.next());
}
```

for 문을 사용하면 이런 복사해 붙여넣기 오류를 컴파일 타임에 잡아준다.

```java
for (Iterator<Element> i = c.iterator(); i.hasNext(); ) {
  Element e = i.next();
  ... // e와 i로 무언가를 한다.
}
...
  
// "i를 찾을 수 없다"는 컴파일 오류 발생
for (Iterator<Element> i2 = c2.iterator(); i.hasNext(); ) {
  Element e2 = i2.next();
  ... // e2와 i2로 무언가를 한다.
}
```

for 문은 변수 유효 범위가 for 문 범위와 일치하여 똑같은 이름의 변수를 여러 반복문에 써도 서로 아무런 영향을 주지 않는다. 그리고 while 문보다 짧아서 가독성이 좋다.

```java
// 지역변수의 범위를 최소화하는 반복문 관용구
for (int i = 0, n = expensiveComputation(); i < n; i++) {
  ... // i로 무언가를 한다.
}
```

마지막으로 지역변수 범위를 최소하하기 위한 방법은 **메서드를 작게 유지하고 한 가지 기능에 집중하는 것이다.** 메서드를 기능별로 쪼개도록 하자.



## Item 58. 전통적인 for 문보다는 for-each 문을 사용하라

다음은 전통적인 for 문으로 컬렉션과 배열을 순회하는 코드다.

```java
// 컬렉션 순회
for (Iterator<Element> i = c.iterator(); i.hasNext(); ) {
  Element e = i.next();
  ... // e로 무언가를 한다.
}

// 배열 순회
for (int i = 0; i < a.length; i++) {
  ... // a[i]로 무언가를 한다.
}
```

이 관용구들은 while 문보다는 낫지만 가장 좋은 방법은 아니다. 반복자와 인덱스 변수는 코드를 지저분하게 한다. 그리고 쓰이는 요소 종류가 늘어나면 오류가 생길 가능성이 높아진다. 혹시라도 잘못된 변수를 사용했을 때 컴파일러가 잡아주리라는 보장도 없고, 컬렉션이냐 배열이냐에 따라 코드 형태가 달라지므로 주의해야 한다. 이는 for-each문을 사용해 해결할 수 있다.

```java
// 컬렉션과 배열을 순회하는 올바른 관용구
for (Element e : elements) {
  ... // e로 무언가를 한다.
}
```

컬렉션을 중첩 순회해야 할 때 for-each 문의 이점이 더욱 커진다.

```java
// 반복문 중첩으로 인한 버그 발생
enum Suit { CLUB, DIAMOND, HEART, SPADE }
enum Rank { ACE, DEUCE, THREE, FOUR, FIVE, SIX, SEVEN, EIGHT, NINE, TEN, JACK, QUEEN, KING }
...
  
static Collection<Suit> suits = Arrays.asList(Suit.values());
static Collection<Rank> ranks = Arrays.asList(Rank.values());

List<Card> deck = new ArrayList<>();
for (Iterator<Suit> i = suits.iterator(); i.hasNext(); )
  for (Iterator<Rank) j = ranks.iterator(); j.hasNext(); )
    deck.add(new Card(i.next(), j.next())); // i.next()를 Suit 하나당 한 번이 아닌 Rank 하나당 한 번씩 부른다.
// 숫자가 바닥나면 반복문에서 NoSuchElementException을 던진다.
```

바깥 컬렉션의 크기가 안쪽 컬렉션 크기의 배수라면 이 반복문은 예외를 던지지 않고 정상 수행 되지 않은 채 종료된다.

```java
// 버그가 발생하는 다른 예
enum Face { ONE, TWO, THREE, FOUR, FIVE, SIX }
...
Collection<Face> faces = EnumSet.allOf(Face.class);

for (Iterator<Face> i = faces.iterator(); i.hasNext(); )
  for (Iterator<Face> j = faces.iterator(); j.hasNext(); )
    System.out.println(i.next() + " " + j.next()); // 예외를 던지진 않지만 가능한 조합을 여섯짱만 출력하고 끝나버린다.
```

바깥 반복문에 바깐 원소를 저장하는 변수를 하나 추가해서 해결할 수는 있다.

```java
for (Iterator<Suit> i = suits.iterator(); i.hasNext(); ) {
  Suit suit = i.next();
  for (Iterator<Rank> j = ranks.iterator(); j.hasNext(); )
    deck.add(new Card(suit, j.next()));
}
```

for-each 중첩으로 간단하고 간결하게 해결할 수 있다.

```java
// 컬렉션이나 배열의 중첩 반복을 위한 권장 관용구
for (Suit suit : suits)
  for (Rank rank : ranks)
    deck.add(new Cart(suit, rank));
```

하지만 for-each문을 사용할 수 없는 상황이 세 가지 존재한다.

- 파괴적인 필터링(destructive filtering) - 컬렉션을 순회하면서 선택된 원소를 제거해야 한다면 반복자의 remove 메서드를 호출해야 한다. 자바 8부터는 Collection의 removeIf 메서드를 사용해 컬렉션을 명시적으로 순회하는 일을 피할 수 있다.
- 변형(transforming) - 리스트나 배열을 순회하면서 그 원소의 값 일부 혹은 전체를 교체해야 한다면 리스트의 반복자나 배열의 인덱스를 사용해야 한다.
- 병렬 반복(parallel iteration) - 여러 컬렉션을 병렬로 순회해야 한다면 각각의 반복자와 인덱스 변수를 사용해 엄격하고 명시적으로 제어해야 한다.

for-each 문은 컬렉션과 배열은 물론 Iterable 인터페이스를 구현한 객체라면 무엇이든 순회할 수 있다. Iterable 인터페이스는 메서드가 단 하나다.

```java
public interface Iterable<E> {
  // 이 객체의 원소들을 순회하는 반복자를 반환한다.
  Iterator<E> iterator();
}
```



## Item 59. 라이브러리를 익히고 사용하라

```java
// 무작위 정수를 생성하는 코드 - 흔하지만 문제가 심각!
static Random rnd = new Random();

static int random(int n) {
  return Math.abs(rnd.nextInt()) % n;
}
```

위 코드는 세 가지 문제를 내포하고 있다. 첫 번째, n이 그리 크지 않은 2의 제곱수라면 얼마 지나지 않아 같은 수열이 반복된다. 두 번째, n이 2의 제곱수가 아니라면 몇몇 숫자가 평균적으로 더 자주 반환된다. n 값이 크면 이 현상은 더 두드러진다. 마지막으로 지정한 범위 외의 수가 종종 튀어나올 수 있다. 이 결함을 해결하려면 `Random.nextInt(int)`를 사용하면 된다. **표준 라이브러리를 사용하면 그 코드를 작성한 전문가의 지식과 앞서 크 코드를 사용한 다른 프로그래머들의 경험을 활용할 수 있다.**

자바 7부터는 Random을 더 이상 사용하지 않는게 좋다. **ThreadLocalRandom으로 대체하면 대부분 잘 작동한다.** 포크-조인 풀이나 병렬 스트림에서는 SplittableRandom을 사용하자.

표준 라이브러리를 쓰면 핵심적인 일과 크게 관련 없는 문제를 해결하느라 시간을 허비하지 않아도 된다. 그리고 따로 노력하지 않아도 성능이 지속해서 개선되고 기능이 점점 많아진다. 또, 자연스럽게 다른 개발자들이 더 읽기 좋고, 유지보수하기 좋고, 재활용하기 쉬운 코드가 된다. 하지만 **메이저 릴리스마다 주목할 만한 수많은 기능이 라이브러리에 추가**되기 때문에 사람들은 라이브러리에 그런 기능이 있는지 모르고 직접 구현하는 경우가 많다.

**자바 프로그래머라면 적어도 `java.lang`, `java.util`, `java.io`와 그 하위 패키지들에는 익숙해져야 한다.** 컬렉션 프레임워크와 스트림 라이브러리, 그리고 `java.util.concurrent`의 동시성 기능도 알아두면 큰 도움이 된다.

우선은 라이브러리를 사용하려 시도해보고, 원하는 기능이 아니라 판단되면 대안을 사용하자. 자바 표준 라이브러리에서 원하는 기능을 찾지 못하면, 서드파티 라이브러리(대표적인 예로 구글의 구아바 라이브러리가 있다.)를 찾아보고 거기에서도 찾지 못했다면, 다른 선택이 없으니 직접 구현하도록 하자.



## Item 60. 정확한 답이 필요하다면 float와 double은 피하라

float과 doubl 타입은 과학과 공학 계산용으로 설계되어 **금융 관련 계산과는 맞지 않다.**

```java
// 오류 발생! 금융 계산에 부동소수 타입을 사용했다.
public static void main(String[] args) {
  double funds = 1.00;
  int itemsBought = 0;
  for (double price = 0.10; funds >= price; price += 0.10) {
    funds -= price;
    itemsBought++;
  }
  System.out.println(itemsBought + "개 구입");
  System.out.println("잔돈(달러):" + funds); // 0.3999999999999999 달러
}
```

**금융 계산에는 BigDecimal, int 혹은 long을 사용해야 한다.**

```java
// BigDecimal을 사용한 해법. 속도가 느리고 쓰기 불편하다.
public static void main(Stringp[] args) {
  final BigDecimal TEN_CENTS = new BigDecimal(".10"); // 계산시 부정확한 값이 사용되는 걸 막기 위해 문자열을 받는 생성자 사용
  
  int itemsBought = 0;
  BigDecimal funds = new BigDecimal("1.00");
  for (BigDecimal price = TEN_CENTS; funds.compareTo(price) >= 0; price = price.add(TEN_CENTS)) {
    funds = funds.subtract(price);
    itemsBought++;
  }
  System.out.println(itemsBought + "개 구입");
  System.out.println("잔돈(달러):" + funds); 
}
```

BigDecimal은 기본 타입보다 쓰기가 훨씬 불편하고 훨씬 느리다는 단점이 있다. 대신 int 혹은 long 타입을 쓸 수도 있다. 그럴 경우 다룰 수 있는 값의 크기가 제한되고, 소수점을 직접 관리해야 한다.

```java
// 정수 타입을 사용한 해법 - 계산을 달러 대신 센트로 수행
public static void main(String[] args) {
  int itemsBought = 0;
  int funds = 100;
  for (int price - 10; funds >= price; price += 10) {
    funds -= price;
    itemsBought++;
  }
  System.out.println(itemsBought + "개 구입");
  System.out.println("잔돈(센트): " + funds);
}
```

BigDecimal을 사용하면 반올림을 완벽히 제어할 수 있지만 성능이 중요하고 소수점을 직접 추적할 수 있고 숫자가 너무 크지 않다면, 숫자를 아홉자리 십진수로 표현할 수 있다면 int를 사용하고, 열여덟 자리 십진수로 표현할 수 있다면 long을 사용하자. 열여덟 자리를 넘어가면 BigDecimal을 사용해야 한다.



## Item 61. 박싱된 기본 타입보다는 기본 타입을 사용하라

자바의 데이터 타입은 크게 int, double, boolean 같은 기본 타입과 String, List 같은 참조 타입으로 나눌 수 있다. 그리고 각각의 기본 타입에 대응하는 참조 타입이 하나씩 있으며, 이를 박싱된 기본 타입이라고 한다(Integer, Double, Boolean). 그런데 기본 타입과 박싱된 기본 타입에는 크게 세 가지 차이점이 있다.

1. 기본 타입은 값만 가지고 있으나, 박싱된 기본 타입은 값에 더해 식별성이란 속성을 갖는다.
2. 기본 타입의 값은 언제나 유효하나, 박싱된 기본 타입은 유효하지 않은 값, 즉 null을 가질 수 있다.
3. 기본 타입이 박싱된 기본 타입보다 시간과 메모리 사용면에서 더 효율적이다.

```java
// 잘못 구현된 비교자
Comparator<Integer> naturalOrder = (i, j) -> (i < j) ? -1 : (i == j ? 0 : 1);
```

`i == j`는 두 객체 참조의 식별성을 검사하기 때문에 i와 j의 값이 같더라도 서로 다른 Integer 인스턴스라면 비교 결과는 false가 된다. 이처럼 **박싱된 기본 타입에 == 연산자를 사용하면 오류가 일어난다.** 기본 타입을 다루는 비교자가 필요하다면 `Comparator.naturalOrder()`을 사용하자. 모든 비교를 기본 타입 변수로 수행하면 식별성 검사가 이루어지지 않는다.

```java
// 문제를 수정한 비교자
Comparator<Integer> naturalOrder = (iBoxed, jBoxed) -> {
  int i = iBoxed, j = jBoxed; // 오토박싱
  return i < j ? -1 : (i == j ? 0 : 1);
};
```

**기본 타입과 박싱된 기본 타입을 혼용한 연산에서는 박싱된 기본 타입의 박싱이 자동으로 풀린다.** null 참조를 언박싱하면 `NullPointerException`이 발생한다.

```java
public class Unbelievable {
  static Integer i;	// int로 선언해야 한다.
  
  public static void main(String[] args) {
    if (i == 42)	// NullPointerException 발생
      System.out.println("믿을 수 없군!");
  }
}
```

박싱과 언박싱이 반복해어 일어나면 성능이 매우 느려지게 된다.

```java
public static void main(String[] args) {
  Long sum = 0L; // 나쁜 성능의 원인이 되는 박싱 타입 선언
  for (long i = 0; i<= Integer.MAX_VALUE; i++) {
    sum += i;
  }
  System.out.println(sum);
}
```

박싱된 기본 타입의 쓰임은 다음과 같다.

1. 컬렉션의 원소, 키, 값으로 쓴다. 컬렉션은 기본 타입을 담을 수 없으므로 박싱된 기본 타입을 써야만 한다.
2. 매개변수화 타입이나 매개변수화 메서드의 타입 매개변수로는 박싱된 기본 타입을 써야 한다. 마찬가지로 기본 타입을 지원하지 않기 때문이다.
3. 리플렉션을 통해 메서드를 호출할 때도 박싱된 기본 타입을 사용해야 한다.



## Item 62. 다른 타입이 적절하다면 문자열 사용을 피하라

**문자열은 다른 값 타입을 대신하기에 적합하지 않다.** 기본 타입이든 참조 타입이든 적절한 값 타입이 있다면 그것을 사용하고, 없다면 새로 하나 작성하는 것이 더 좋다. **문자열은 열거 타입과 혼합 타입을 대신하기에 적합하지 않다.**

```java
// 혼합 타입을 문자열로 처리한 부적절한 예
String compoundKey = className + "#" + i.next();
```

두 요소를 구분해주는 문자 #이 두 요소 중 하나에서 쓰였다면 문제가 생길 수 있고, 문자열을 파싱해야해서 느리고, 오류 가능성도 커진다. 적절한 equals, toString, compareTo 메서드를 제공할 수 없으며, String이 제공하는 기능에만 의존해야 한다. 차라리 private 정적 멤버 클래스로 전용 클래스를 선언해서 사용하는 편이 낫다.

**문자열은 권한을 표현하기에 적합하지 않다.** 각 스레드가 자신만의 변수를 갖게 하는 스레드별 지역변수 기능을 문자열로 사용한다고 생각해보자.

```java
// 잘못된 예 - 문자열을 사용해 권한을 구분
public class ThreadLocal {
  private ThreadLocal() { } // 객체 생성 불가
  
  // 현 스레드의 값을 키로 구분해 저장한다.
  public static void set(String key, Object value);
  
  // (키가 가리키는) 현 스레드의 값을 반환한다.
  public static Object get(String key);
}
```

이 방식의 문제는 스레드 구분용 문자열 키가 전역 namespace에서 공유된다는 점이다. 만약 두 클라이언트가 같은 키를 사용하게되면 의도치 않게 같은 변수를 공유하게 되고, 두 클라이언트 모두 제대로 기능하지 못할 것이다. 보안도 취약하다. 악의적인 클라이언트라면 의도적으로 같은 키를 사용하여 다른 클라이언트의 값을 가져올 수도 있다. 이 API는 문자열 대신 위조할 수 없는 키를 사용하면 해결된다. 이 키를 권한(capacity)이라고도 한다.

```java
// Key 클래스로 권한을 구분
public class ThreadLocal {
  private ThreadLocal() { } // 객체 생성 불가
  
  public static class Key { // (권한)
    Key() { }
  }
  
  // 위조 불가능한 고유 키를 생성한다.
  public static Key getKey() {
    return new Key();
  }
  
  public static void set(Key key, Object value);
  public static Object get(Key key);
}
```

set과 get은 이제 정적 메서드일 이유가 없으니 Key 클래스의 인스턴스 메서드로 바꾸면 Key는 그 자체가 스레드 지역변수가 된다. 결과적으로 지금의 톱레벨 클래스인 ThreadLocal을 없애고 중첩 클래스 Key의 이름을 ThreadLocal로 바꿔버리는 방식으로 리팩터링이 가능하다.

```java
// 리팩터링
public final class ThreadLocal {
  public ThreadLocal();
  public void set(Object value);
  public Object get();
}
```

이 API에서는 get으로 얻은 Object를 실제 타입으로 형변환해 써야 해서 타입 안전하지 않다. 이는 ThreadLocal을 매개변수화 타입으로 선언하면 해결된다.

```java
// 매개변수화하여 타입안전성 확보
public final class ThreadLocal<T> {
  public ThreadLocal();
  public void set(T value);
  public T get();
}
```



## Item 63. 문자열 연결은 느리니 주의하라

문자열 연결 연산자(+)는 여러 문자열을 하나로 합쳐주는 편리한 수단이지만, **문자열 연결 연산자로 문자열 n개를 잇는 시간은 n제곱에 비례한다.**

```java
// 문자열 연결을 잘못 사용한 에 - 느리다!
public String statement() {
  String result = "";
  for (int i = 0; i < numItems(); i++)
    result += lineForItem(i); // 문자열 연결
  return result;
}
```

**성능을 포기하고 싶지 않다면 String 대신 StringBuilder를 사용하자.**

```java
// StringBuilder를 사용하면 문자열 연결 성능이 크게 개선된다.
public String statement2() {
  StringBuilder b = new StringBuilder(numItems() * LINE_WIDTH);
  for (int i = 0; i < numItems(); i++)
    b.append(lineForItem(i));
  return b.toString();
}
```



## Item 64. 객체는 인터페이스를 사용해 참조하라

**적합한 인터페이스만 있다면 매개변수뿐 아니라 반환값, 변수, 필드를 전부 인터페이스 타입으로 선언하라.**

```java
// 좋은 예. 인터페이스를 타입으로 사용
Set<Son> sonSet = new LinkedHashSet<>();
// 나쁜 예. 클래스를 타입으로 사용
LinkedHashSet<Son> sonSet = new LinkedHashSet<>();
```

**인터페이스를 타입으로 사용하는 습관을 길러두면 프로그램이 훨씬 유연해질 것이다.** 그러면 나중에 구현 클래스를 교체하고자 할 때 새 클래스의 생성자(혹은 다른 정적 팩터리)를 호출해주기만 하면 된다.

```java
Set<Son> sonSet = new HashSet<>();
```

단, 원래의 클래스가 인터페이스의 일반 규약 이외의 특별한 기능을 제공하며, 주변 코드가 이 기능에 기대어 동작한다면 새로운 클래스도 반드시 같은 기능을 제공해야 한다.

선언 타입과 구현 타입을 동시에 바꿀 수 있으니 변수를 구현 타입으로 선언해도 괜찮을 거라 생각할 수도 있지만 자칫마녀 프로그램이 컴파일되지 않는다. 예컨대 클라이언트에서 기존 타입에서만 제공하는 메서드를 사용했거나, 기존 타입을 사용해야 하는 다른 메서드에 그 인스턴스를 넘긴다면 컴파일되지 않을 것이다. 변수를 인터페이스 타입으로 선언하면 이런 일이 발생하지 않는다.

**적합한 인터페이스가 없다면 당연히 클래스로 참조해야 한다.**

- String과 BigInteger 같은 값 클래스. 값 클래스를 여러 가지로 구현도리 수 있다고 생각하고 설계하는 일은 거의 없어 final인 경우가 많고 상응하는 인터페이스가 별도로 존재하는 경우가 드문다. 이런 값 클래스는 매개변수, 변수, 필드, 반환 타입으로 사용해도 무방하다.

- 클래스 기반으로 작성된 프레임워크가 제공하는 객체. 이런 경우라도 특정 구현 클래스보다는 (보통은 추상 클래스인) 기반 클래스를 사용해 참조하는게 좋다.

  ex. OutputStream 등 `java.io` 패키지의 여러 클래스

- 인터페이스에는 없는 특별한 메서드를 제공하는 클래스. 클래스 타입을 직접 사용하는 경우는 추가 메서드를 꼭 사용해야 하는 경우로 최소하해야 하며, 절대 남발해서는 안된다.

  ex. PriorityQueue 클래스는 Queue 인터페이스에는 없는 comparator 메서드를 제공

**적합한 인터페이스가 없다면 클래스의 계층구조 중 필요한 기능을 만족하는 가장 덜 구체적인(상위의) 클래스를 타입으로 사용하자.**



## Item 65. 리플렉션보다는 인터페이스를 사용하라

리플렉션 기능(`java.lang.reflect`)을 이용하면 프로그램에서 임의의 클래스에 접근할 수 있다. Class 객체가 주어지면 그 클래스의 생성자, 메서드, 필드에 해당하는 Constructor, Method, Field 인스턴스를 가져올 수 있고, 이 인스턴스들로는 그 클래스의 멤버 이름, 필드 타입, 메서드 시그니처 등을 가져올 수 있고 각각에 연결된 실제 생성자, 메서드, 필드를 조작할 수도 있다. 리플렉션을 이용하면 컴파일 당시에 존재하지 않던 클래스도 이용할 수 있지만, 단점도 존재한다.

- 컴파일타임 타입 검사가 주는 이점을 누릴 수 없다. 예외 검사도 마찬가지다.
- 리플렉션을 이용하면 코드가 지저분하고 장황해진다.
- 성능이 떨어진다. 리플렉션을 통한 메서드 호출은 일반 메서드 호출보다 훨씬 느리다.

**리플렉션은 아주 제한된 형태로만 사용해야 그 단점을 피하고 이점만 취할 수 있다. 리플렉션은 인스턴스 생성에만 쓰고, 이렇게 만든 인스턴스는 인터페이스나 상위 클래스로 참조해 사용하자.**

```java
// 리플렉션으로 생성하고 인터페이스로 참조해 활용
public static void main(String[] args) {
  // 클래스 이름을 Class 객체로 변환
  Class<? extends Set<String>> cl = null;
  try {
    cl = (Class<? extends Set<String>>) // 비검사 형변환!
      Class.forName(args[0]);
  } catch (ClassNotFoundException e) {
    fatalError("클래스를 찾을 수 없습니다.");
  }
  
  // 생성자를 얻는다.
  Constructor<? extends Set<String>> cons = null;
  try {
    cons = cl.getDeclaredConstructor();
  } catch (NoSuchMethodException e) {
    fatalError("매개변수 없는 생성자를 찾을 수 없습니다.");
  }
  
  // 집합의 인스턴스를 만든다.
  Set<String> s = null;
  try {
    s = cons.newInstance();
  } catch (IllegalAccessException e) {
    fatalError("생성자에 접근할 수 없습니다.");
  } catch (InstantiationException e) {
    fatalError("클래스를 인스턴스화할 수 없습니다.");
  } catch (InvocationTargetException e) {
    fatalError("생성자가 예외를 던졌습니다: " + e.getCause());
  } catch (ClassCastException e) {
    fatalError("Set을 구현하지 않은 클래스입니다.");
  }
  
  // 생성한 집합을 사용한다.
  s.addAll(Arrays.asList(args).subList(1, args.length));
  System.out.println(s); // 첫 번째 인수로 지정한 클래스가 무엇이냐에 따라 출력 순서가 달라진다.
}

private static void fatalError(String msg) {
  System.err.println(msg);
  System.exit(1);
}
```

이 예는 리플렉션의 두 가지 단점을 보여준다. 첫 번째, 런타임에 총 여섯 가지나 되는 예외를 던질 수 있다. 인스턴스를 리플렉션 없이 생성했다면 컴파일 타임에 다 잡아낼 수 있는 예외들이다. 두 번째, 클래스 이름만으로 인스턴스를 생성해내기 위해 25줄이나 되는 코드를 작성했다. 리플렉션이 아니라면 생성자 호출만 해주면 된다. 리플렉션 예외 각각을 잡는 대신 모든 리플렉션 예외의 상위 클래스인 ReflectiveOperationException을 잡도록 할 수도 있다.

리플렉션은 런타임에 존재하지 않을 수도 있는 다른 클래스, 메서드, 필드와의 의존성을 관리할 때 적합하다. 이 기법은 버전이 여러 개 존재하는 외부 패키지를 다룰 때 유용하다. 단, 같은 목적을 이룰 수 있는 대체 수단을 이용하거나 기능을 줄여 동작하는 등의 적절한 조치가 필요하다. 리플렉션을 사용해야 하는 경우에는 되도록 객체 생성에만 사용하고, 생성한 객체를 이용할 때는 적절한 인터페이스나 컴파일타임에 알 수 있는 상위 클래스로 형변환해 사용해야 한다.



## Item 66. 네이티브 메서드는 신중히 사용하라

자바 네이티브 인터페이스는 자바 프로그램이 네이티브 메서드를 호출하는 기술이다. 여기서 네이티브 메서드란 C나 C++ 같은 네이티브 프로그래밍 언어로 작성한 메서드를 말한다. 전통적으로 네이티브 메서드의 주요 쓰임은 다음 세 가지다.

1. 레지스트리 같은 플랫폼 특화 기능을 사용한다.
2. 네이티브 코드로 작성된 기존 라이브러리를 사용한다. 레거시 데이터를 사용하는 레거시 라이브러리가 그 예다.
3. 성능 개선을 목적으로 성능에 결정적인 영향을 주는 영역만 따로 네이티브 언어로 작성한다.

**성능을 개선할 목적으로 네이티브 메서드를 사용하는 것은 거의 권장하지 않는다.** 단, 네이티브 라이브러리 쪽은 GNU 다중 정밀 연산 라이브러리(GMP)를 필두로 개선 작업이 계속되어왔기 때문에 고성능의 다중 정밀 연산이 필요하다면 네이티브 메서드를 통해 GMP를 사용하는 것을 고려해도 좋다.

네이티브 메서드에는 심각한 단점이 있다. 네이티브 메서드는 안전하지 않으므로 네이티브 메서드를 사용하는 애플리케이션도 메모리 훼손 오류로부터 더 이상 안전하지 않다. 네이티브 언어는 자바보다 이식성도 낮고 디버깅도 어렵다. 주의하지 않으면 속도가 오히려 느려질 수도 있고, 가비지 컬렉터가 네이티브 메모리는 자동 회수 및 추적 할 수 없다. 자바 코드와 네이티브 코드의 경계를 넘나들 때마다 비용도 추가되고, 네이티브 메서드와 자바 코드 사이의 '접착 코드'를 작성하는 것은 귀찮고 가독성이 떨어진다.



##  Item 67. 최적화는 신중히 하라

다음의 세 격언은 최적화의 어두운 진실을 이야기해준다.

> (맹목적인 어리석음을 포함해) 그 어떤 핑계보다 효율성이라는 이름 아래 행해진 컴퓨팅 죄악이 더 많다(심지어 효율을 높이지도 못하면서).
>
> \- 윌리엄 울프

> (전체의 97% 정도인) 자그마한 효율성은 모두 잊자. 섣부른 최적화가 만악의 근원이다.
>
> \- 도널드 크누스

> 최적화를 할 때는 다음 두 규칙을 따르라.
>
> 첫 번째, 하지마라.
>
> 두 번째, (전문가 한정) 아직 하지 마라. 다시 말해, 완전히 명백하고 최적화되지 않은 해법을 찾을 때까지는 하지 마라.
>
> \- M. A. 잭슨

**빠른 프로그램보다는 좋은 프로그램을 작성하자.** 좋은 프로그램은 정보 은닉 원칙을 따르므로 개별 구성요소의 내부를 독립적으로 설계할 수 있다. 따라서 시스템의 나머지에 영향을 주지 않고도 각 요소를 다시 설계할 수 있다.

**성능을 제한하는 설계를 피하자.** API, 네트워크 프로토콜, 영구 저장용 데이터 포맷과 같은 설계 요소들은 완성 후에는 변경이 어렵거나 불가능할 수 있으며, 동시에 시스템 성능을 심각하게 제한할 수 있다.

**API를 설계할 때 성능에 주는 영향을 고려하자.** 잘 설계된 API는 성능도 좋은 게 보통이다. 신중하게 설계하여 깨끗하고 명확하고 멋진 구조를 갖춘 프로그램을 완성한 다음에야 최적화를 고려해볼 차례가 된다. 잭슨이 소개한 최적화 규칙 두 가지에 하나를 더 추가한다면 **"각각의 최적화 시도 전후로 성능을 측정하라"** 정도가 되겠다. 시도한 최적화 기법이 성능을 눈에 띄게 높이지 못하는 경우가 많고, 심지어 더 나빠지게 할 때도 있다. 일반적으로 90%의 시간을 단 10%의 코드에서 사용한다는 사실을 기억해두자.

프로파일링 도구는 개별 메서드의 소비 시간과 호출 횟수 같은 런타임 정보를 제공하여, 최적화를 위해 집중할 곳은 물론 알고리즘을 변경해야 한다는 사실을 알려주기도 한다. 시스템 규모가 커질수록 프로파일러가 더 중요해진다. jmh는 프로파일러는 아니지만 자바 코드의 상세한 성능을 알기 쉽게 보여주는 마이크로 벤치마킹 프레임워크다.

자바는 프로그래머가 작성하는 코드와 CPU에서 수행하는 명령 사이의 '추상화 격차'가 커서 최적화로 인한 성능 변화를 일정하게 예측하기가 어렵다. 자바의 성능 모델은 정교하지 않을뿐더러 구현 시스템, 릴리스, 프로세서마다 차이가 있다. 여러 자바 플랫폼이나 여러 하드웨어 플랫폼에서 구동한다면 최적화의 효과를 그 각각에서 측정해야 한다.



## Item 68. 일반적으로 통용되는 명명 규칙을 따르라

자바 플랫폼은 명명 규칙이 잘 정립되어 있으며, 그중 많은 것이 자바 언어 명세에 기술되어 있다. 자바의 명명 규칙은 크게 철자와 문법, 두 범주로 나뉜다.

### 철자 규칙

- 철자 규칙은 패키지, 클래스, 인터페이스, 메서드, 필드, 타입 변수의 이름을 다룬다. 이 규칙을 어긴 API는 사용하기 어렵고, 유지보수하기 어렵다.
- 패키지와 모듈 이름은 각 요소를 점(.)으로 구분하여 계층적으로 짓는다. 요소들은 모두 소문자 알파벳 혹은 숫자로 이뤄진다. 조직 바깥에서도 사용될 패키지라면 조직의 인터넷 도메인 이름을 역순으로 사용한다. 예외적으로 표준 라이브러리와 선택적 패키지들은 각각 java와 javax로 시작한다.
- 패키지 이름의 나머지는 해당 패키지를 설명하는 하나 이상의 요소로 이뤄진다. 각 요소는 일반적으로 8자 이하의 짧은 단어로 한다. 여러 단어로 구성된 이름이라면 각 단어의 첫 글자만 따서 써도 좋다. 요소의 이름은 보통 한 단어 혹은 약어로 이뤄진다.
- 인터넷 도메인 이름 뒤에 요소 하나만 붙인 패키지가 많지만, 많은 기능을 제공하는 경우엔 계층을 나눠 더 많은 요소로 구성해도 좋다.
- 클래스와 인터페이스의 이름은 하나 이상의 단어로 이뤄지며, 각 단어는 대문자로 시작한다.
- 메서드와 필드 이름은 첫 글자를 소문자로 쓴다는 점만 빼면 클래스 명명 규칙과 간다.
- 상수 필드를 구성하는 단어는 모두 대문자로 쓰며 단어 사이는 밑줄로 구분한다.
- 지역변수에도 다른 멤버와 비슷한 명명 규칙이 적용되나, 약어를 사용할 수 있다.
- 타입 매개변수 이름은 보통 한 문자로 표현한다. 임의의 타입엔 T, 컬렉션 원소의 타입은 E, 맵의 키와 값은 K와 V, 예외는 X, 메서드 반환 타입은 R을 주로 사용한다. 그 외에 임의 타입 시퀀스에는 T, U, V 혹은 T1, T2, T3를 사용한다.

| 식별자 타입         | 예                                               |
| ------------------- | ------------------------------------------------ |
| 패키지와 모듈       | org.junit.jupiter.api, com.google.common.collect |
| 클래스와 인터페이스 | Stream, FutureTask, LinkedHashMap, HttpClient    |
| 메서드와 필드       | remove, groupingBy, getCrc                       |
| 상수 필드           | MIN_VALUE, NEGATIVE_INFINITY                     |
| 지역변수            | i, denom, houseNum                               |
| 타입 매개변수       | T, E, K, V, X, R, U, V, T1, T2                   |

### 문법 규칙

- 패키지에 대한 규칙은 따로 없다.
- 객체를 생성할 수 있는 클래스의 이름은 보통 단수 명사나 명사구를 사용하고, 객체를 생성할 수 없는 클래스의 이름은 보통 복수형 명사로 짓는다.
- 인터페이스의 이름은 클래스와 똑같이 짓거나, able 혹은 ible로 끝나는 형용사로 짓는다.
- 애너테이션은 지배적인 규칙이 없이 명사, 동사, 전치사, 형용사가 두루 쓰인다.
- 어떤 동작을 수행하는 메서드의 이름은 동사나 동사구로 짓는다.
- boolean 값은 반환하는 메서드라면 보통 is나 has로 시작하고 명사나 명사구, 혹은 형용사로 기능하는 단어나 구로 끝나도록 짓는다.
- 반환 타입이 boolean이 아니거나 해당 인스턴스의 속성을 반환하는 메서드의 이름은 보통 명사, 명사구, 혹은 get으로 시작하는 동사구로 짓는다.
- 객체의 타입을 바꿔서, 다른 타입의 또 다른 객체를 반환하는 인스턴스 메서드의 이름은 보통 toType 형태고, 객체의 내용을 다른 뷰로 보여주는 메서드의 이름은 asType 형태로, 객체의 값을 기본 타입 값으로 반환하는 메서드의 이름은 보통 typeValue 형태로 짓는다.
- 정적 팩터리의 이름은 다양하지만 from, of, valueOf, instance, getInstance, newInstance, getType, newType을 흔히 사용한다.

API 설계를 잘 했다면 필드가 직접 노출될 일이 거의 없기 때문에 필드 이름에 관한 문법 규칙은 클래스, 인터페이스, 메서드 이름에 비해 덜 명확하고 덜 중요하다. 지역변수 이름도 필드와 비슷하게 지으면 되나, 조금 더 느슨하다.
