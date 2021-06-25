---
title: EffectiveJava_Chap5
excerpt: 이펙티브 자바 책 5장 정리
categories: study
---

# 5장. 제네릭

## Item 26. 로 타입은 사용하지 말라

클래스와 인터페이스 선언에 타입 매개변수가 쓰이면, 이를 **제네릭 클래스** 혹은 **제네릭 인터페이스**라 한다. 이를 통틀어 **제네릭 타입**이라 한다. List 인터페이스는 원소의 타입을 나타내는 타입 매개변수 E를 받는다. 완전하게 제네릭 타입으로 표현하면 `List<E>` 이다. 각각의 제네릭 타입은 일련의 매개변수화 타입을 정의한다. `클래스_이름<실제_타입_매개변수>` (ex. `List<String>`원소의 타입이 String인 리스트(매개변수화 타입))

제네릭 타입을 하나 정의하면 그에 딸린 로 타입(raw type)도 함게 정의된다. ex. `List<E>` 의 로 타입은 `List`

```java
// 컬랙션의 로 타입 - 따라 하지 말 것!
// Stamp 인스턴스만 취급한다. - (주석은 컴파일러가 이해하지 못함)
private final Collection stamps = ...;

// 실수로 동전을 넣는다.
stamps.add(new Coin(...));	// "unchecked call" 경고를 내뱉는다. 아무 오류 없이 컴파일되고 실행됨.
  													// 컬렉션에서 동전을 꺼내기 전에는 오류를 알아채지 못함
```

  ```java
  // 반복자의 로 타입 - 따라 하지 말 것!
  for (Iterator i = stamps.iterator(); i.hasNext(); ) {
    Stamp stamp (Stamp) i.next(); // ClassCastException을 던진다. 런타임에 문제가 발생 - 에러 원인을 찾기 어려워진다.
    stamp.cancel();
  }
  ```

  ```java
  // 매개변수화된 컬렉션 타입 - 타입 안전성 확보!
  private final Collection<Stamp> stamps = ...; // Stamp의 인스턴스만 넣어야 함을 컴파일러가 인지
  ```

`List` 같은 로 타입은 사용해서는 안 되나 `List<Object>`처럼 임의 객체를 허용하는 매개변수화 타입은 괜찮다. 이는 모든 타입을 허용한다는 의사를 컴파일러에 명확히 전달한 것이다. 매개변수로 `List`를 받는 메서드에 `List<String>` 을 넘길 수 있지만, `List<Object>`를 받는 메서드에는 제네릭의 하위타입 규칙 때문에 넘길 수 없다. `List<String>`은 `List`의 하위 타입이지만, `List<Object>`의 하위 타입은 아니다. `List<Object>` 같은 매개변수화 타입을 사용할 때와 달리 `List` 같은 로 타입을 사용하면 타입 안전성을 잃게 된다.

```java
// 런타임에 실패
public static void main(String[] args) {
  List<String> strings = new ArrayList<>();
  unsafeAdd(strings, Integer.valueOf(42)); 
  String s = string.get(0); // 컴파일러가 자동으로 형변환 코드를 넣어준다.
  // 런타임 중 Integer를 String으로 변환하려 시도해 ClassCastException이 발생한다.
}
// List 대신 List<Object>를 사용하면 컴파일 에러가 발생한다.
private static void unsafeAdd(List list, Object o) { // unsafeAdd 메서드가 로 타입(List)을 사용
  list.add(o);
}
```

```java
// 잘못된 예 - 모르는 타입의 원소도 받는 로 타입을 사용
// 2개의 집합을 받아 공통 원소를 반환하는 메서드. 동작은 하지만 안전하지 않다.
static int numElementsInCommon(Set s1, Set s2) {
  int result = 0;
  for (Object o1 : s1)
    if (s2.contains(o1))
      result++;
  return result;
}
```

제네릭 타입을 쓰고 싶지만 실제 타입 매개변수가 무엇인지 신경 쓰고 싶지 않다면 비한정적 와일드 카드 타입을 쓰면 된다. 제네릭 타입 `Set<E>`의 비한정적 와일드 카드 타입은 `Set<?>`다 (어떤 타입이라도 담을 수 있는 가장 범용적인 매개변수화 Set 타입).

```java
static int numElementsInCommon(Set<?> s1, Set<?> s2) { ... } // 타입 안전하며 유연함
```

로 타입 컬렉션에는 아무 원소나 넣을 수 있으니 타입 불변식을 훼손하기 쉬우나, `Collection<?>`에는 (null 외에는) 어떤 원소도 넣을 수 없다. 무슨 말인가 싶지만 비한정적 와일드 카드 타입은 **메소드가 실제 타입을 전혀 신경쓰지 않을 때 유용**하다. 이미 원소가 담긴 컬렉션을 인자로 받는다면 무엇이든 꺼내 사용할 수 있고, 그 객체의 타입은 전혀 알 수 없다. 이러한 제약을 받아들일 수 없다면 제네릭 메서드나 한정적 와일드카드 타입을 사용하면 된다.

class 리터럴에는 로 타입을 써야한다. 자바 명세는 class 리터럴에 매개변수화 타입을 사용 하지 못하게 했다(배열과 기본 타입은 허용).

- List.class / String[].class / int.class (O)
- List\<String>.class / List<?>.class (X)

런타임에는 제네릭 타입 정보가 지워지므로 instanceof 연산자는 비한정적 와일드카드 타입 이외의 매개변수화 타입에는 적용할 수 없다. 그리고 로 타입과 비한정적 와일드카드 타입은 instanceof가 똑같이 동작한다(꺾쇠와 물음표 없이, 로 타입을 쓰는 것이 깔끔).

```java
// 로 타입을 써도 좋은 예 - instanceof 연산자
if (o instanceof Set) { // 로 타입
  Set<?> s = (Set<?>) o; // 와일드카드 타입.
  ...
}
```



## Item 27. 비검사 경고를 제거하라

제네릭을 사용하면 비검사 형변환 경고, 비검사 메서드 호출 경고, 비검사 매개변수화 가변인수 타입 경고, 비검사 변환 경고 등을 볼 수 있다. 대부분의 비검사 경고는 쉽게 제거할 수 있다.

```java
Set<Lark> exaltation = new HashSet(); // 컴파일러가 잘못과 수정 방햐야을 알려 줌
Set<Lark> exaltation = new HashSet<>(); // 자바 7부터 지원하는 다이아몬드 연산자를 사용. 컴파일러가 올바른 타입을 추론해 줌
```

할 수 있는 한 모든 비검사 경고를 제거하는 것이 좋다(타입 안전성 보장). 그러면 런타임에 ClassCastException이 발생할 일이 없고, 의도한 대로 잘 동작한다. 경고를 제거할 수는 없지만 타입 안전하다고 확신할 수 있다면 `@SuppressWarnings("unchecked")` 애너테이션을 달아 경고를 숨길 수 있다(단, 가능한 한 좁은 범위에 적용. 변수 선언, 아주 짧은 메서드, 생성자).

```java
// ArrayList에서 가져온 toArray 메서드
public <T> T[] toArray(T[] a) {
  if (a.length < size) {
    //return (T[]) Arrays.copyOf(elements, size, a.getClass()); // 경고 발생
    // 생성한 배열과 매개변수로 받은 배열의 타입이 모두 T[]로 같으므로 올바른 형변환이다.
    @SuppressWarnings("unchecked") T[] result = (T[]) Arrays.copyOf(elements, size, a.getClass());
    return result;
  }
  System.arraycopy(elements, 0, a, 0, siez);
  if (a.length > size)
    a[size] = null;
  return a;
}
```

이 코드는 깔끔하게 컴파일되고 비검사 경고를 숨기는 범위도 최소로 좁혔다. `@SuppressWarnings("unchecked")` 애너테이션을 사용할 때면 그 경고를 무시해도 안전한 이유를 항상 주석으로 남겨야 한다.



## Item 28. 배열보다는 리스트를 사용하라

### 배열과 제네릭 타입의 차이

배열은 공변이고 제네릭은 불공변이다. (서로 다른 타입 Type1과 Type2. `List<Type1>`은 `List<Type2>`의 하위 타입도 상위 타입도 아니다.)

```java
// 런타임에 실패
Object[] objectArray = new Long[1];
objectArray[0] = "타입이 달라 넣을 수 없다."; // ArrayStoreException을 던진다.
```

```java
// 컴파일되지 않음
List<Object> ol = new ArrayList<Long>(); // 호환되지 않는 타입이다.
ol.add("타입이 달라 넣을 수 없다.");
```

배열은 실체화된다. 배열은 런타임에도 자신이 담기로 한 원소의 타입을 인지하고 확인한다. 반면, 제네릭은 타입 정보가 런타입에는 소거된다. 원소 타입을 컴파일타임에만 검사하며 런타임에는 알수조차 없다. 이러한 차이들로 인해 배열과 제네릭은 잘 어우러지지 못한다. 배열은 제네릭 타입, 매개변수화 타입, 타입 매개변수로 사용할 수 없다(`new List<E>[]`, `new List<String>`, `new E[]`식으로 작성하면 컴파일할 때 제네릭 배열 생성 오류 발생). 제네릭 배열을 만들지 못하게 막은 이유는 타입 안전하지 않기 때문이다(컴파일러가 자동 생성한 형변환 코드에서 런타임에 `ClassCastException`발생 가능).

```java
// 컴파일되지 않는다.
List<String>[] stringLists = new List<String>[1]; 	// 허용된다고 가정
List<Integer> intList = List.of(42);								// 원소가 하나인 List<Integer> 생성
Object[] objects = stringLists;											// Object 배열에 처음의 배열을 할당. 배열은 공변이라 문제없다.
objects[0] = intList;																// 런타임에는 제네릭 타입 정보가 소거되서 ArrayStoreException 발생x
String s = stringLists[0].get(0);										// 꺼낸 Integer를 String으로 변환 시도. ClassCastException 발생
```

소거 매커니즘 때문에 매개변수화 타입 가운데 실체화될 수 있는 타입은 비한정적 와일드카드 타입뿐이다. 배열을 제네릭으로 만들 수 없어 제네릭 컬렉션에서는 자신의 원소 타입을 담은 배열을 반환하는게 보통은 불가능하다. 또한 제네릭 타입과 가변인수 메서드를 함께 쓰면, 가변인수 메서드를 호출할 때마다 가변인수 매개변수를 담을 배열이 하나 만들어지는데 그 배열의 원소가 실체화 불가 타입이라면 경고가 발생한다. 이 문제는 `@SafeVarargs` 애너테이션으로 대처할 수 있다. 배열로 형변환할 때 제네릭 배열 생성 오류나 비검사 형변환 경고가 뜨는 경우 대부분은 배열인 E[] 대신 컬렉션인 List\<E>를 사용하면 해결된다.

```java
// 제네릭을 쓰지 않고 구현한 버전
public class Chooser { 
  private final Object[] choiceArray;
  
  public Chooser(Collection choices) {
    choiceArray = choices.toArray();
  }
  
  public Object choose() { // 호출할 때마다 반환된 Object를 원하는 타입으로 형변환해야 한다. 
    Random rnd = ThreadLocalRandom.current();
    return choiceArray[rnd.nextInt(choiceArray.length)];
  }
}
```

```java
// 컴파일되지 않는다.
public class Chooser<T> {
  private final T[] choiceArray;
  
  public Chooser(Collection<T> choices) {
    //choiceArray = choices.toArray(); // Object 배열을 T 배열에 대입하려해 컴파일 에러 발생
    choiceArray = (T[]) choices.toArray(); // 형변환이 런타임에도 안전한지 보장할 수 없다는 경고가 뜬다.
  }
  // choose 메서드는 그대로
}
```

비검사 형변환 경고를 제거하려면 배열 대신 리스트를 쓰면 된다.

```java
// 리스트 기반 Chooser - 타입 안전성 확보!
public class Chooser<T> {
  private final List<T> choiceList;
  
  public Chooser(Collection<T> choices) {
    choiceList = new ArrayList<>(choices);
  }
  
  public T choose() {
    Random rnd = ThreadLocalRandom.current();
    return choiceList.get(rnd.nextInt(choiceList.size()));  
  }
}
```



## Item 29. 이왕이면 제네릭 타입으로 만들라

스택 코드를 제네릭 타입으로 작성해보자.

```java
public class Stack<E> {
  private E[] elements;
  private int size = 0;
  private static final int DEFAULT_INITIAL_CAPACITY = 16;
  
  public Stack() {
    //elements = new E[DEFAULT_INITIAL_CAPACITY]; // E와 같은 실체화 불가 타입으로는 배열을 만들 수 없다.
    elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY]; // 컴파일러가 타입 안전성 경고를 내보낸다.
  }
  
  public void push(E e) {
    ensureCapacity();
    elements[size++] = e;
  }
  
  public E pop() {
    if (size == 0)
      throw new EmptyStackException();
    E result = elements[--size];
    elements[size] = null; // 다 쓴 참조 해제
    return result;
  }
  
  public boolean isEmpty() {
    return size == 0;
  }
  
  public void ensureCapacity() {
    if (elements.length == size)
      elements = Array.copyOf(elements, 2 * size + 1);
  }
}
```

배열 elements는 private 필드에 저장되고, 클라이언트로 반환되거나 다른 메서드네 전달되는 일이 전혀 없다. push 메서드를 통해 배열에 저장되는 원소의 타입은 항상 E이기 때문에 이 비검사 형변환은 확실히 안전하다. 그렇다면 범위를 최소로 좁혀 `@SuppressWarnings` 애너테이션으로 경고를 숨기면 된다. 이 예에서는 생성자가 비검사 배열 생성 외에 하는 일이 없어 생성자 전체에서 경고를 숨겨도 무관하다. 애너테이션을 사용하면 Stack은 깔끔히 컴파일되고, 명시적으로 형변환하지 않아도 `ClassCastException` 걱정 없이 사용할 수 있다.

```java
// 배열을 사용한 코드를 제네릭으로 만드는 방법 1

// 배열 elements는 push(E)로 넘어온 E 인스턴스만 담는다.
// 따라서 타입 안전성을 보장하지만, 이 배열의 런타임 타입은 E[]가 아닌 Object[]다!
@SuppressWarnings("unchecked")
public Stack() {
  elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];
}
```

elements 필드의 타입을 E[]에서 Object[]로 바꾸는 것도 제네릭 배열 생성 오류를 해결하는 또 다른 방법이다. 이 경우에는 pop 메서드에서 E에 Object를 대입하려해서 오류가 발생한다. 마찬가지로 비검사 형변환을 수행한 뒤, 안전하지를 증명하고 경고를 숨길 수 있다.

```java
// 배열을 사용한 코드를 제네릭으로 만드는 방법 2
// 비검사 경고를 적절히 숨긴다.
public E pop() {
  if (size == 0)
    throw new EmptyStackException();
  
  // push에서 E 타입만 허용하므로 이 형변환은 안전하다.
  @SupperessWarnings("unchecked") E result = (E) elements[--size]; // 비검사 형변환을 수행하는 할당문에서만 경고 숨김.
  
  elements[size] = null; // 다 쓴 참조 해제
  return result;
}
```

첫 번째 방법은 가독성이 더 좋고, 형변환을 배열 생성 시 단 한 번만 해주면되지만 배열의 런타임 타입이 컴파일타임 타입과 달라 힙 오염을 일으킬 수 있다. 두 번째 방식에서는 배열에서 원소를 읽을 때마다 형변환을 해줘야 한다. 두 방법 각자의 장단점이 존재한다.

자바가 리스트를 기본 타입으로 제공하지 않으므로 ArrayList 같은 제네릭 타입도 결국은 기본 타입인 배열을 사용해 구현해야 하고, HashMap 같은 제네릭 타입은 성능을 높일 목적으로 배열을 사용하기도 한다.

Stack 예처럼 대다수의 제네릭 타입은 타입 매개변수에 아무런 제약을 두지 않는다. `Stack<Object>`, `Stack<int[]>`, `Stack<List<String>>`, `Stack`등 어떤 참조 타입으로도 Stack을 만들 수 있다. 단, `Stack<int>`, `Stack<double>`처럼 기본 타입은 사용할 수 없다.

`java.util.concurrent.DelayQueue`처럼 타입 매개변수에 제약을 두는 제네릭 타입도 있다.

```java
class DelayQueue<E extends Delayed> implements BlockingQueue<E> // 한정적 타입 매개변수 사용
```

`java.util.concurrent.Delayed`의 하위 타입만 받는다. DelayQueue의 원소에서 (형변환 없이) Delayed 클래스의 메서드를 호출할 수 있으며, `ClassCastException` 걱정을 할 필요가 없다. 그리고 모든 타입은 자기 자신의 하위 타입이므로 `DelayQueue<Delayed>`로도 사용할 수 있다.



## Item 30. 이왕이면 제네릭 메서드로 만들라

클래스와 마찬가지로, 메서드도 제네릭으로 만들 수 있다. 매개변수화 타입을 받는 정적 유틸리티 메서드는 보통 제네릭이다. Collections의 '알고리즘' 메서드(binarySearch, sort 등)는 모두 제네릭이다. (타입 매개변수들을 선언하는) 타입 매개변수 목록은 메서드의 제한자와 반환 타입 사이에 온다.

```java
// 제네릭 메서드
// 입력 2개, 반환 1개. 집합 3개의 타입이 모두 같아야 한다. 한정적 와일드카드 타입을 사용하면 더 유연하게 개선할 수 있다.
public static <E> Set<E> union(Set<E> s1, Set<E> s2) { // <E>는 타입 매개변수 목록. 반환 타입은 Set<E>
  Set<E> result = new HashSet<>(s1);
  result.addAll(s2);
  return result;
}
```

```java
// 제네릭 메서드를 활용하는 간단한 프로그램
public static void main(String[] args) {
  Set<String> guys = Set.of("톰", "딕", "해리");
  Set<String> stooges = Set.of("래리", "모에", "컬리");
  Set<String> aflCio = union(guys, stooges);
  System.out.println(aflCio);
}
```

제네릭은 런타임에 타입 정보가 소거되므로 하나의 객체를 어떤 타입으로는 매개변수화할 수 있다. 이렇게 하려면 요청한 타입 매개변수에 맞게 매번 그 객체의 타입을 바꿔주는 정적 팩터리를 만들어야 한다. 이 패턴을 제네릭 싱글턴 팩터리라 하며, `Collections.reverseOrder` 같은 함수 객체나 `Colletions.emptySet` 같은 컬렉션용으로 사용한다.

```java
// 제네릭 싱글턴 팩터리 패턴
private static UnaryOperator<Object> IDENTITY_FN = (t) -> t;

@SuppressWarnings("unchecked")
public static <T> UnaryOperator<T> identityFunction() {
  return (UnaryOperator<T>) IDENTITY_FN;
}
```

```java
// 제네릭 싱글턴을 사용하는 예. 형변환을 하지 않아도 컴파일 오류나 경고가 발생하지 않는다.
public static void main(String[] args) {
  String[] strings = {"삼베", "대마", "나일론"};
  UnaryOperator<String> sameString = identityFunction();
  for (String s: strings)
    System.out.println(sameString.apply(s));
  Number[] numbers = {1, 2.0 3L};
  UnaryOperator<Number> sameNumber = identityFunction();
  for (Number n: numbers)
    System.out.println(sameNumber.apply(n));
}
```

자기 자신이 들어간 표현식을 사용하여 타입 매개변수의 허용 범위를 한정할 수 있다(재귀적 타입 한정). 주로 타입의 자연적 순서를 정하는 Comparable 인터페이스와 함께 쓰인다.

```java
public interface Comparable<T> {
  int compareTo(T o);
}
```

여기서 타입 매개변수  T는 Comparable\<T>를 구현한 타입이 비교할 수 있는 원소의 타입을 정의한다. 실제로 거의 모든 타입은 자신과 같은 타입의 원소와만 비교할 수 있다.

```java
// 재귀적 타입 한정을 이용해 상호 비교할 수 있음을 표현
public static <E extends Comparable<E>> E max(Collection<E> c);
```

타입 한정인 `<E extends Comparable<E>>`는 "모든 타입 E는 자신과 비교할 수 있다"라고 읽을 수 있다(상호 비교 가능하다).

```java
// 컬렉션에서 최댓값을 반환 - 재귀적 타입 한정 사용
// 이 메서드에 빈 컬렉션을 건네면 IllegalArgumentException을 던지니, Optional<E>를 반환하도록 고치는 것이 더 낫다.
public static <E extends Comparable<E>> E max(Collection<E> c) {
  if (c.isEmpty())
    throw new IllegalArgumentException("컬렉션이 비어 있습니다.");
  E result = null;
  for (E e : c)
    if (result == null || e.compareTo(result) > 0)
      result = Objects.requireNonNull(e);
  return result;
}
```



## Item 31. 한정적 와일드카드를 사용해 API 유연성을 높이라

매개변수화 타입은 불공변이다. 불공변 방식보다 유연한 무언가가 필요할 때 한정적 와일드카드를 사용하면 된다.

스택에서 일련의 원소를 스택에 넣는 메서드를 추가해야 한다고 생각해보자.

```java
// 와일드카드 타입을 사용하지 않은 pushAll 메서드
public void pushAll(Iterable<E> src) {
  for (E e : src)
    push(e);
}
```

이 메서드는 Iterable src의 원소 타입이 스택의 원소 타입과 일치하면 잘 작동하지만 그렇지 않다면 매개변수화 타입이 불공변이기 때문에 오류가 발생한다. 이런 상황에서 한정적 와일드카드 타입이라는 특별한 매개변수화 타입을 사용하면 된다.

```java
// 생산자(producer) 매개변수에 와일드카드 타입 적용 
public void pushAll(Iterable<? extends E> src) { // 'E의 하위 타입의 Iterable'
  for (E e : src)
    push(e);
}
```

이렇게 작성하면 타입 안전하다.

pushAll과 짝을 이루어 Stack 안의 모든 원소를 주어진 컬렉션으로 옮겨 담는 popAll 메서드도 추가해야 한다고 생각해보자.

```java
public void popAll(Collection<E> dst) {
  while (!isEmpty())
    dst.add(pop());
}
```

이 메서드도 주어진 컬렉션의 원소 타입이 스택의 원소 타입과 일치한다면 문제 없이 작동하지만 그렇지 않다면 비슷한 오류가 발생한다.

```java
// 소비자(consumer) 매개변수에 와일드카드 타입 적용
public void popAll(Collection<? super E> dst) { // "E의 상위 타입의 Collection"
  while (!isEmpty())
    dst.add(pop());
}
```

유연성을 극대화 하려면 원소의 생산자나 소비자용 입력 매개변수에 와일드카드 타입을 사용해라. 하지만 입력 매개변수가 생산자와 소비자 역할을 동시에 한다면 와일드카드 타입을 쓸 필요가 없다. 타입을 정확히 지정해야 하는 상황으로, 와일드카드 타입을 쓰지 말아야 한다.

> 펙스(PECS): producer-extends, consumer-super

매개변수화 타입 Trk 생산자라면 <? extends T>를 사용하고, 소비자라면  <? super T>를 사용해라. 다른 예제에도 적용해보자.

```java
public Chooser(Collection<? extends T> choices) // choices 컬렉션은 T 타입의 값을 '생산'하기만 한다.
```

```java
public static <E> Set<E> union(Set<? extends E> s1, Set<? extends E> s2) // s1과 s2 모두 E의 생산자이다.
// 반환 타입에는 한정적 와일드카드 타입을 사용하면 안 된다(클라이언트 코드에서도 와일드카드 타입을 써야하게 만든다). 여전히 Set<E> 사용
```

앞의 코드는 자바 8부터는 제대로 컴파일되지만 자바 7까지는 타입 추론 능력이 충분히 강력하지 못해 문맥에 맞는 반환 타입을 명시해야 한다.

```java
// 자바 7까지는 명시적 타입 인수를 사용
Set<Number> numbers = Union.<Number>union(integers, doubles);
```

앞의 max 메서드를 와일드카드 타입을 사용해 다듬으면 다음과 같은 모습이다.

```java
public static <E extends Comparable<E>> E max(List<E> list) // 원래 형태
public static <E extends Comparable<? super E>> E max(List<? extends E> list) // PECS 공식 두 번 적용
```

- 입력 매개변수에서는 E 인스턴스를 생산하므로 `List<? extends E>` 사용
- `Comparable<E>`는 인스턴스를 소비하기 때문에 `Comparable<? super E>` 사용 (Comparable은 언제나 소비자!)
  `Comparator<E>`도 마찬가지로 `Comparator<? super E>`를 사용하는 것이 좋다.

타입 매개변수와 와일드카드에는 공통되는 부분이 있어서, 메서드를 정의할 때 둘 중 어느 것을 사용해도 괜찮을 때가 많다.

```java
// swap 메서드의 두 가지 선언
public static <E> void swap(List<E> list, int i, int j); // 비한정적 타입 매개변수 사용
public static void swap(List<?> list, int i, int j); // 비한정적 와일드카드 사용
```

public API라면 간단한 두 번째 선언이 낫다.  어떤 리스트든 이 메서드에 넘기면 명시한 인덱스의 원소들을 교환해 줄 것이고, 신경 써야 할 타입 매개변수도 없다. 기본 규칙은 메서드 선언에 타입 매개변수가 한 번만 나오면 와일드 카드로 대체하는 것이다(비한적정 타입 매개변수는 비한정적 와일드카드로, 한정적 타입 매개변수는 한정적 와일드카드로).

하지만 두 번째 swap 선언은 다음 코드가 컴파일되지 않는다.

```java
public static void swap(List<?> list, int i, int j) {
  list.set(i, list.set(j, list.get(i))); // 리스트의 타입이 List<?>라 null외에는 어떤 값도 넣을 수 없다.
}
```

이는 와일드카드 타입의 실제 타입을 알려주는 메서드를  private 도우미 메서드로 따로 작성하여 활용하면 해결할 수 있다. 실제 타입을 알아내려면 이 도우미 메서드는 제네릭 메서드여야 한다.

```java
public static void swap(List<?> list, int i, int j) {
  swapHelper(list, i, j);
}

// 와일드카드 타입을 실제 타입으로 바꿔주는 private 도우미 메서드.
// 리스트가 List<E>임을 알고 있어 꺼낸 값의 타입은 항상 E이고, E 타입의 값을 리스트에 넣어도 안전함을 알고 있다.
// 첫번째 swap 메서드의 시그니처와 완전히 똑같다. 대신 외부에서 와일드카드 기반의 선언을 유지할 수 있다.
private static <E> void swapHelper(List<E> list, int i, int j) {
  list.set(i, list.set(j, list.get(i)));
}
```



## Item 32. 제네릭과 가변인수를 함께 쓸 때는 신중하라

가변인수(varargs) 메서드와 제네릭은 궁합이 좋지 않다. 가변인수는 메서드에 넘기는 인수의 개수를 클라이언트가 조절할 수 있게 해주는데, 가변인수 메서드를 호출하면 가변인수를 담기 위한 배열이 자동으로 하나 만들어진다. varargs 매개변수에 제네릭이나 매개변수화 타입이 포함되면 컴파일 경고가 발생한다. 메서드를 선언할 때 실체화 불가 타입으로  varargs 매개변수를 선언하면 컴파일러가 경고를 보내는데, 거의 모든 제네릭과 매개변수화 타입은 실체화되지 않는다. 매개변수화 타입의 변수가 타입이 다른 객체를 참조하면 힙 오염이 발생한다.

```java
// 제네릭과 varargs를 혼용하면 타입 안전성이 깨진다!
static void dangerous(List<String>... stringLists) {
  List<Integer> intList = List.of(42);
  Object[] objects = stringLists;
  objects[0] = intList;							// 힙 오염 발생
  String s = stringLists[0].get(0)	// ClassCastException
}
```

제네릭 배열을 프로그래머가 직접 생성하는 건 허용하지 않지만, 제네릭 varargs 매개변수를 받는 메서드는 선언할 수 있다. 제네릭이나 매개변수화 타입의 varargs 매개변수를 받는 메서드가 실무에서 매우 유용하기 때문이다(ex. `Arrays.asList(T... a)`, `Collections.addAll(Collection<? super T> c, T... elements)`, `EnumSet.of(E first, E... rest)`). 자바 라이브러리에서 제공하는 이 메서드들은 타입 안전하다.

자바 7에서는 `@SafeVarargs` 애너테이션이 추가되어 제네릭 가변인수 메서드 작성자가 클라이언트 측에서 발생하는 경고를 숨길 수 있게 되었다. 이는 메서드 작성자가 그 메서드가 타입 안전함을 보장하는 장치다. 가변인수 메서드를 호출할 때 varargs 매개변수를 담는 제네릭 배열이 만들어지는데, 메서드가 이 배열에 아무것도 저장하지 않고(그 매개변수들을 덮어쓰지 않고) 그 배열의 참조가 밖으로 노출되지 않는다면(신뢰할 수 없는 코드가 배열에 접근할 수 없다면) 타입 안전하다. 즉, 이 varargs 매개변수 배열이 호출자로부터 그 메서드로 순수하게 인수들을 전달하는 일만 한다면 그 메서드는 안전하다.

```java
// 자신의 제네릭 매개변수 배열의 참조를 노출함. - 안전하지 않다!
static <T> T[] toArray(T... args) { // 1. pickTwo에 어떤 타입의 객체를 넘기더라고 담을 수 있도록 Object[]를 만들어 줌
  return args;
}

// T타입 인수 3개를 받아 그중 2개를 무작위로 골라 담은 배열 반환
static <T> T[] pickTwo(T a, T b, T c) {
  switch(ThreadLocalRandom.current().nextInt(3)) { // 2. toArray 메서드가 돌려준 배열들 그대로 클라이언트에 전달
    case 0: return toArray(a, b);
    case 1: return toArray(a, c);
    case 2: return toArray(b, c);
  }
  throw new AssertionError(); // 도달할 수 없다.
}

public static void main(String[] args) {
  String[] attributes = pickTwo("좋은", "빠른", "저렴한"); // 3. ClassCastException 발생
}
```

이 예는 제네릭 varargs 매개변수 배열에 다른 메서드가 접근하도록 허용하면 안전하지 않다는 것을 보여준다. 단, `@SafeVarargs`로 제대로 애노테이트된 또 다른 varargs 메서드에 넘기는 것과 그저 이 배열 내용의 일부 함수를 호출만 하는(varargs를 받지 않는) 일반 메서드에 넘기는 것은 안전하다.

```java
// 제네릭 varargs 매개변수를 안전하게 사용하는 메서드
@SafeVarargs
static <T> List<T> flatten(List<? extends T>... lists) {
  List<T> result = new ArrayList<>();
  for (List<? extends T> list : lists)
    result.addAll(list);
  return result;
}
```

정리하면 다음과 같다. 제네릭이나 매개변수화 타입의 varargs 매개변수를 받는 모든 메서드에  `@SafeVarargs`를 달아야 한다. 다음 두 조건을 모두 만족하는 제네릭 varargs 메서드는 안전하다.

- varargs 매개변수 배열에 아무것도 저장하지 않는다.
- 그 배열(혹은 복제본)을 신뢰할 수 없는 코드에 노출하지 않는다.

그리고 `@SaveVarargs` 애너테이션은 재정의할 수 없는 메서드에만 달아야 한다. 재정의한 메서드도 안전할지는 보장할 수 없기 때문이다. (실체는 배열인) varargs 매개변수를 List 매개변수로 바꾸는 방법도 있다.

```java
// 제네릭 varargs 매개변수를 List로 대체한 예 - 타입 안전하다!
static <T> List<T> flatten(List<List<? extends T>> lists) {
  List<T> result = new ArrayList<>();
  for (List<? extends T> list : lists)
    result.addAll(list);
  return result;
}
```

정적 팩터리 메서드인  `List.of`를 활용하면 다음 코드와 같이 이 메서드에 임의개수의 인수를 넘길 수 있다(`List.of`에도 `@SafeVarargs` 애너테이션이 달려 있기 때문).

```java
audience = flatten(List.of(friends, romans, countrymen));
```

이 방식은 클라이언트 코드가 살짝 지저분해지고 속도가 조금 느려질 수 있지만, 컴파일러가 이 메서드의 타입 안전성을 검증할 수 있다.

```java
// pickTwo에 List.of를 사용하여 타입 안전성 확보
static <T> List<T> pickTwo(T a, T b, T c) {
  switch(ThreadLocalRandom.current().nextInt(3)) {
    case 0: return List.of(a, b);
    case 1: return List.of(a, c);
    case 2: return List.of(b, c);
  }
  throw new AssertionError();
}

public static void main(String[] args) {
  List<String> attributes = pickTwo("좋은", "빠른", "저렴한");
}
```



## Item 33. 타입 안전 이종 컨테이너를 고려하라

제네릭은 `Set<E>`, `Map<K, V>` 등의 컬렉션과 `ThreadLocal<T>`, `AtomicReference<T>` 등의 단일원소 컨테이너에도 흔히 쓰인다. 이런 모든 쓰임에서 매개변수화 되는 대상은 (원소가 아닌) 컨테이너 자신이다. 따라서 하나의 컨테이너에서 매개변수화할 수 있는 타입의 수가 제한된다. 컨테이너 대신 키를 매개변수화한 다음, 컨테이너에 값을 넣거나 뺄 때 매개변수화한 키를 함께 제공하면 조금 더 유연하게 사용할 수 있다. 이렇게 하면 제네릭 타입 시스템이 값의 타입이 키와 같음을 보장해줄 것이다. 이러한 설계 방식을 타입 안정 이종 컨테이너 패턴(type safe heterogeneous container pattern)이라 한다.

### Favorites 클래스 예제

즐겨 찾는 인스턴스를 저장하고 검색할 수 있는 Favorites 클래스를 생각해보자. 각 타입의  Class 객체를 매개변수화한 키 역할로 사용하면 되는데, 이 방식이 동작하는 이유는 class의 클래스가 제네릭이기 때문이다. class 리터럴의 타입은 `Class`가 아닌 `Class<T>`다. 컴파일타임 타입 정보와 런타임 타입 정보를 알아내기 위해 메서드들이 주고받는  class 리터럴을 타입 토큰이라 한다.

```java
// 타입 안전 이종 컨테이너 패턴 - API
public class Favorites {
  public <T> void putFavorite(Class<T> type, T instance); // 키가 매개변수화 됨
  public <T> T getFavorite(Class<T> type);
}
```

```java
// 타입 안전 이종 컨테이너 패턴 - 클라이언트
public static void main(String[] args) {
  Favorites f = new Favorites();
  
  f.putFavorite(String.class, "Java");
  f.putFavorite(Integer.class, 0xcafebabe);
  f.putFavorite(Class.class, Favorites.class);
  
  String favoriteString = f.getFavorite(String.class);
  int favoriteInteger = f.getFavorite(Integer.class);
  Class<?> favoriteClass = f.getFavorite(Class.class);
  
  System.out.printf("%s %x %s%n", favoriteString, favoriteInteger, favoriteClass.getName());
  // Java cafebabe Favorites
}
```

Favorites 인스턴스는 타입 안전하다.  String을 요청했는데 Integer를 반환하는 일은 절대 없다. 또한 모든 키의 타입이 제각각이라, 일반적인 맵과 달리 여러가지 타입의 원소를 담을 수 있다.

```java
// 타입 안전 이종 컨테이너 패턴 - 구현
public class Favorites {
  // 키가 와일드카드 타입. 모든 키가 서로 다른 매개변수화 타입일 수 있다는 의미.
  // 값 타입이 단순히 Object. 키와 값 사이의 타입 관계를 보증하지 않음. 모든 값이 키로 명시한 타입임을 보증하지 않음.
  private Map<Class<?>, Object> favorites = new HashMap<>();
  
  public <T> void putFavorite(Class<T> type, T instance) {
    favorites.put(Objects.requireNonNull(type), instance);
  }
  
  public <T> T getFavorite(Class<T> type) {
    return type.cast(favorites.get(type));	// cast 메서드는 형변환 연산자의 동적 버전
    																				// 주어진 인수가 Class 객체가 알려주는 타입의 인스턴스인지를 검사
    																				// 1. 맞으면 그 인수를 그대로 반환
    																				// 2. 아니면 ClassCastException을 던짐
  }
}
```

cast 메서드의 시그니처가 Class 클래스가 제네릭이라는 이점을 완벽히 활용하기 때문에  cast를 사용한다.

```java
public class Class<T> {
  T cast(Object obj);
}
```

지금의 Favorites 클래스에 악의적인 클라이언트가 Class 객체를 로 타입으로 넘기면  Favorites 인스턴스의 타입 안정성이 쉽게 깨진다. 하지만 이렇게 짜여진 클라이언트 코드에서는 컴파일할 때 비검사 경고가 뜰 것이다. Favorites가 타입 불변식을 어기는 일이 없도록 보장하려면 면  putFavorite 메서드에서 인수로 주어진  instance의 타입이  type으로 명시한 타입과 같은지 확인하면 된다.

```java
// 동적 형변환으로 런타임 타입 안전성 확보
public <T> void putFavorite(Class<T> type, T instance) {
  favorites.put(Objects.requireNonNull(type), type.cast(instance));
}
```

`java.util.Collections`의 checkdSet, checkedList, checkedMap 같은 메서드들이 이 방식을 적용한 컬렌션 래퍼들이다.

Favorites 클래스는 실체화 불가 타입에는 사용할 수 없다.  `String`이나 `String[]`은 가능하지만 `List<String>`을 저장하려하면 컴파일 오류가 발생한다. `List<String>`용 Class 객체를 얻을 수 없기 때문이다. `List<String>.class`라고 쓰면 문법 오류가 난다.  `List<String>`과 `List<Integer>`는 `List.class`라는 같은 Class 객체를 공유한다. 이 제약을 슈퍼 타입 토큰으로 해결하려는 시도도 있지만 완벽하지는 않다.

```java
// 뭐 요론식으로 사용한다....
Favorites f = new Favorites();

List<String> pets = Arrays.asList("개", "고양이", "앵무");

f.putFavorite(new TypeRef<List<String>>(){}, pets);
List<String> listofStrings = f.getFavorite(new TypeRef<List<String>>(){});
```

Favorites가 사용하는 타입 토큰은 비한정적이다(어떤 Class 객체든 받아들인다). 한정적 타입 토근을 활용하면 메서드들이 허용하는 타입을 제한할 수 있다. 한정적 타입 토큰이란 단순히 한정적 타입 매개변수나 한정적 와일드카드를 사용하여 표현 가능한 타입을 제한하는 타입 토큰이다. 애너테이션  API는 한정적 타입 토큰을 적극적으로 사용한다.

```java
// AnnotatedElement 인터페이스에 선언된 메서드. 대상 요소에 달려있는 애너테이션을 런타임에 읽어 오는 기능
// 이 메서드는 리플렉션의 대상이 되는 타입들(프로그램 요소를 표현하는 타입들)에서 구현한다.
// (클래스java.lang.Class<T>, 메서드(java.lang.reflect.Mehod), 필드(java.lang.reflect.Field))
public <T extends Annotation> T getAnnotation(Class<T> annotationType);
```

여기서 annotationType 인수는 애너테이션 타입을 뜻하는 한정적 타입 토큰이다. 이 메서드는 토큰으로 명시한 타입의 애너테이션이 대상 요소에 달려 있다면 그 애너테이션을 반환하고, 없다면 null을 반환한다. 즉, 애너테이션된 요소는 그 키가 애너테이션 타입인, 타입 안정 이종 컨테이너인 것이다.

Class<?> 타입의 객체가 있고, 이를 한정적 타입 토큰을 받는 메서드에 넘기려면 객체를  `Class<? extends Annotation>`으로 형변환할 수도 있지만, 이 형변환은 비검사이므로 컴파일 경고가 뜰 것이다. Class 클래스의  asSubclass 메서드는 이런 형변환을 안전하게 수행해 준다.

```java
// asSubclass를 사용해 한정적 타입 토큰을 안전하게 형변환한다.
static Annotation getAnnotation(AnnotatedElement element, String annotationTypeName) {
  Class<?> annotationType = null; // 비한정적 타입 토큰
  try {
    annotationType = Class.forName(annotationTypeName);
  } catch (Exception ex) {
    throw new IllegalArgumentException(ex);
  }
  // 호출된 인스턴스 자신의 Class 객체를 인수가 명시한 클래스로 형변환한다.
  // 성공하면 인수로 받은 클래스 객체를 반환, 실패하면 ClassCastException을 던짐
  return element.getAnnotation(annotationType.asSubclass(Annotation.class));
}
```
