---
title: EffectiveJava_Chap8
excerpt: 이펙티브 자바 책 8장 정리
categories: study
---

# 8장. 메서드

## Item 49. 매개변수가 유효한지 검사하라

메서드와 생성자 대부분은 입력 매개변수의 값이 특정 조건을 만족하기를 바란다. 이런 제약은 반드시 문서화해야 하면 메서드 몸체가 시작되기 전에 검사해야 한다. 그렇지 않으면 메서드가 수행되는 중간에 모호한 예외를 던지며 실패하거나, 잘못된 결과를 반환할 수 있다. 그러면 매개변수 검사에 실패해 실패 원자성을 어기는 결과를 낳을 수 있다.

public과 protected 메서드는 매개변수 값이 잘못됐을 때 던지는 예외를 문서화해야 한다(@throws 자바독 태그 사용, 보통 IllegalArgumentException, IndexOutofBoundsException, NullPointerException 중 하나). 다음은 전형적인 예다.

```java
/**
 * (현재 값 mod m) 값을 반환한다. 이 메서드는 항상 음이 아닌 BigInteger를 반환한다는 점에서 remainder 메서드와 다르다.
 *
 * @param m 계수(양수여야 한다.)
 * @return 현재 값 mod m
 * @throws ArithmeticException m이 0보다 작거나 같으면 발생한다.
 */
public BigInteger mod(BigInteger m) {
  if (m.signum() <= 0)
    throw new ArithmeticException("계수(m)는 양수여야 합니다. " + m);
  ... // 계산 수행
}
```

NullPointerException에 대한 기술이 없는 이유는 이 설명을 BigInteger 클래스 수준에서 기술했기 때문이다. 클래스 수준 주석은 그 클래스의 모든 public 메서드에 적용되므로 각 메서드에 일일이 기술하는 것보다 훨씬 깔끔하다. `@Nullable` 이나 이와 비슷한 애너테이션을 사용해 특정 매개변수는 null이 될 수 있다고 알려줄 수도 있지만, 표준적인 방법은 아니다.

자바 7에 추가된 `java.util.Objects.requireNonNull` 메서드는 유연하고 사용하기도 편하니, 더 이상 null 검사를 수동으로 하지 않아도 된다.

```java
this.strategy = Objects.requireNonNull(strategy, "전략");
```

반환 값은 무시하고 순수하게 null 검사 목적으로 사용해도 된다. 자바 9에서는 Objects에 checkFromIndexSize, checkFromToIndex, checkIndex라는 범위 검사 기능을 하는 메서드들도 더해졌다.

public이 아닌 메서드라면 단언문(assert)을 사용해 매개변수 유효성을 검증할 수 있다.

```java
// 재귀 정렬용 private 도우미 함수
private static void sort(long a[], int offset, int length) {
  assert a != null;
  assert offset >= 0 && offset <= a.length;
  assert length >= 0 && length <= a.length - offset;
  ... // 계산 수행
}
```

단언문들은 자신이 단언한 조건이 무조건 참이라고 선언한다. 단언문은 몇 가지 면에서 일반적인 유효성 검사와 다르다. 첫 번째, 실패하면 AssertionError를 던진다. 두 번째, 런타임에 아무런 효과도, 아무런 성능 저하도 없다(단, java를 실행할 때 명령줄에서 -ea 혹은 --enableassertions 플래그 설정하면 런타임에 영향을 준다).

메서드가 직접 사용하지는 않으나 나중에 쓰기 위해 저장하는 매개변수는 특히 더 신경 써서 검사해야 한다. 생성자는 이러한 원칙의 특수한 사례다. 생성자 매개변수의 유효성 검사는 클래스 불변식을 어기는 객체가 만들어지지 않게 하는데 꼭 필요하다.

유효성 검사 비용이 지나치게 높거나 실용적이지 않을 때, 혹은 계산 과정에서 암묵적으로 검사가 수행될 때는 메서드 몸체 실행 전에 매개변수 유효성 검사를 하지 않아도 된다. 하지만 암묵적 유효성 검사에 너무 의존했다가는 실패 원자성을 해칠 수 있으니 주의해야 한다.

때로는 계산 중 잘못된 매개변수 값을 사용해 발생한 예외와 API 문서에서 던지기로 한 예외가 다를 수 있다. 이런 경우에는 예외 번역(exception translate) 관용구를 사용하여 API 문서에 기재된 예외로 번역해줘야한다.



## Item 50. 적시에 방어적 복사본을 만들라

자바는 안전한 언어이지만, 클라이언트가 불변식을 깨뜨리지 못하도록 방어적으로 프로그래밍해야 한다.

```java
// 기간을 표현하는 클래스 - 불변식을 지키지 못했다.
public final class Period {
  private final Date start;
  private final Date end;
  
  /**
   * @param start 시작 시각
   * @param end 종료 시각; 시작 시각보다 뒤여야 한다.
   * @throws IllegalArgumentException 시작 시각이 종료 시각보다 늦을 때 발생한다.
   * @thorws NullPointerException start나 end가 null이면 발생한다.
   */
  public Period(Date start, Date end) {
    if (start.compareTo(end) > 0)
      throw new IllegalArgumentException(start + "가 " + end + "보다 늦다.");
    this.start = start;
    this.end = end;
  }
  
  public Date start() {
    return start;
  }
  
  public Date end() {
    return end;
  }
  ... // 나머지 코드 생략
}
```

Date가 가변이라는 사실을 이용하면 어렵지 않게 그 불변식을 깨뜨릴 수 있다.

```java
// Period 인스턴스의 내부를 공격해보자.
Date start = new Date();
Date end = new Date();
Period p = new Period(start, end);
end.setYear(78); // p의 내부를 수정했다!
```

자바 8 이후로는 Date 대신 불변인 Instant를 사용하면 된다. **Date는 새로운 코드를 작성할 때는 더 이상 사용하면 안 된다.**

외부 공격으로부터 Peroid 인스턴스의 내부를 보호하려면 **생성자에서 받은 가변 매개변수 각각을 방어적으로 복사해야 한다.** 그런 다음 Period 인스턴스 안에서는 원본이 아닌 복사본을 사용한다.

```java
// 수정한 생성자 - 매개변수의 방어적 복사본을 만든다.
public Peroid(Date start, Date end) {
  this.start = new Date(start.getTime());
  this.end = new Date(end.getTime());
  
  if (this.start.compareTo(this.end) > 0)
    throw new IllegalArgumentException(this.start + "가 " + this.end + "보다 늦다.");
}
```

**매개변수의 유효성을 검사하기 전에 방어적 복사본을 만들고, 이 복사본으로 유효성을 검사한 점에 주목하자.** 멀티스레딩 환경에서의 취약점을 보완할 수 있다. 방어적 복사에 Date의 clone 메서드를 사용하지 않은 점에도 주목하자. Date는 final이 아니므로 clone이 Date가 정의한 게 아닐 수 있다. **매개변수가 제3자에 의해 확장이될 수 있는 타입이라면 방어적 복사본을 만들 때 clone을 사용해서는 안 된다.**

Period 인스턴스는 접근자 메서드가 내부의 가변 정보를 직접 드러내기 때문에 아직도 변경 가능하다.

```java
// Period 인스턴스를 향한 두 번째 공격
Date start = new Date();
Date end = new Date();
Period p = new Period(start, end);
p.end().setYear(78); // p의 내부를 변경했다!
```

두 번째 공격을 막아내려면 단순히 **접근자가 가변 필드의 방어적 복사본을 반환하면 된다.**

```java
// 수정한 접근자 - 필드의 방어적 복사본을 반환한다.
public Date start() {
  return new Date(start.getTime());
}

public Date end() {
  return new Date(end.getTime());
}
```

이렇게 하면 Peroid 자신 말고는 가변 필드에 접근할 방법이 없으니 불변식을 위배할 방법은 없다. 모든 필드가 객체 안에 완벽하게 캡슐화되었다. Peroid가 가지고 있는 Date 객체는 `java.util.Date`임이 확실하기 때문에 접근자 메서드에서는 방어적 복사에 clone를 사용해도 된다. 하지만 인스턴스를 복사하는 데는 일반적으로 생성자나 정적 팩터리르 쓰는게 좋다.

매개변수를 방어적으로 복사하는 목적이 불변 객체를 만들기 위해서만은 아니다. 메서드든 생성자든 클라이언트가 제공한 객체의 참조를 내부의 자료구조에 보관해야 할 때면 항상 그 객체가 잠재적으로 변경될 수 있는지를 생각해야 한다. 변경될 수 있는 객체라면 임의로 변경되었을 때 문제가 없는지 확신할 수 없다면 복사본을 만들어 저장해야 한다. 내부 객체를 클라이언트에 건네주기 전에 방어적 복사본을 만드는 이유도 마찬가지다.

방어적 복사에는 성능 저하가 따르고, 항상 쓸 수 있는 것도 아니다. 호출자가 컴포넌트 내부를 수정하지 않으리라 확신하면 방어적 복사를 생략할 수 있다. 다른 패키지에서 사용한다고 해서 넘겨받은 가변 매개변수를 항상 방어적으로 복사해 저장해야 하는 것도 아니다. 대신 해당 구성요소를 수정했을 때의 책임이 클라이언트에 있음을 문서화하는 것이 좋다. **방어적 복사를 생략해도 되는 상황은 해당 클래스와 그 클라이언트가 상호 신뢰할 수 있을 때, 혹은 불변식이 깨지더라도 그 영향이 오직 호출한 클라이언트로 국한될 때로 한정해야 한다.** 후자의 예로는 래퍼 클래스는 특성상 클라이언트가 래퍼에 넘긴 객체에 여전히 직접 접근할 수 있어 래퍼의 불변식을 쉽게 파괴할 수는 있지만 그 영향을 클라이언트 자신만 받는다.



## Item 51. 메서드 시그니처를 신중히 설계하라

다음의 요령들을 잘 활용하면 배우기 쉽고, 쓰기 쉬우며, 오류 가능성이 적은 API를 만들 수 있다.

#### 메서드 이름을 신중히 짓자.

항상 표준 명명 규칙을 따라야 한다. 이해할 수 있고, 같은 패키지에 속한 다른 이름들과 일관되게 짓는게 최우선 목표다. 개발자들에게 널리 받아들여지는 이름을 사용하자. 긴 이름은 피하자. 애매하면 자바 라이브러리의 API 가이드를 참조하자.

#### 편의 메서드를 너무 많이 만들지 말자.

모든 메서드는 각자 자신의 소임을 다해야 한다. 메서드가 너무 많은 클래스와 인터페이스는 익히고, 사용하고, 문서화하고, 테스트하고, 유지보수하기 어렵다. 클래스나 인터페이스는 자신의 각 기능을 완벽히 수행하는 메서드로 제공해야 한다. 아주 자주 쓰일 경우에만 별도의 약칭 메서드를 만들자. **확신이 서지 않으면 만들지 말자.**

#### 매개변수 목록은 짧게 유지하자

4개 이하가 좋다. 너무 많으면 매개변수를 전부 기억하기 어렵다. **같은 타입의 매개변수 여러 개가 연달아 나오는 경우가 특히 해롭다.** 실수로 순서를 바꿔 입력하면 실행은 되나 의도와 다르게 동작한다. 갠 매개변수 목록을 짧게 줄여주는 방법이 있다.

- 여러 메서드로 쪼갠다. 잘못하면 메서드가 너무 많아질 수 있지만, 직교성을 높여 오히려 메서드 수를 줄여주는 효과도 있다. `java.util.List` 인터페이스가 좋은 예다.
- 매개변수 여러 개를 묶어주는 도우미 클래스를 만든다. 일반적으로 이런 도우미 클래스는 정적 멤버 클래스로 둔다. 잇따른 매개변수 몇 개를 독립된 하나의 개념으로 볼 수 있을 때 좋은 방법이다.
- 빌더 패턴을 메서드 호출에 응용한다(위 두 방법을 혼합). 매개변수가 많을 때, 특히 그중 일부는 생략해도 괜찮을 때 좋다.
  - 모든 매개변수를 하나로 추상화한 객체를 정의하고, 클라이언트에서 이 객체의 세터 메서드를 호출해 필요한 값을 설정하게 한다.
  - 이때 각 세터 메서드는 매개변수 하나 혹은 서로 연관된 몇 개만 설정하게 한다.
  - 클라이언트는 먼저 필요한 매개변수를 다 설정한 다음, execute 메서드를 호출해 앞서 설정한 매개변수들의 유효성을 검사한다.
  - 설정이 완료된 객체를 넘겨 원하는 계산을 수행한다.

#### 매개변수의 타입으로는 클래스보다는 인터페이스가 더 낫다.

매개변수로 적합한 인터페이스가 있다면 그 인터페이스를 직접 사용하자. 인터페이스 대신 클래스를 사용하면 클라이언트에게 특정 구현체만 사용하도록 제한하는 꼴이며, 혹시라도 입력 데이터가 다른 형태로 존재한다면 명시한 특정 구현체의 객체로 옮겨 담느라 비싼 복사 비용을 치러야 한다.

#### boolean보다는 원소 2개짜리 열거 타입이 낫다.

메서드 이름장 boolean을 받아야 의미가 더 명확할 때를 제외하고는 열거 타입을 사용하면 코드를 읽고 쓰기가 더 쉬워진다. 나중에 선택지를 추가하기도 쉽다.

```java
public enum TemperaturScale { FAHRENHEIT, CELSIUS } // 화씨온도, 섭씨온도를 원소로 정의한 열거 타입.
```

`Thermometer.newInstance(true)` 보다는 `Thermometer.newInstance(TemperatureScale.CELSIUS)` 가 더 명확하다. 캘빈온도도 지원해야 한다면 Thermometer에 또 다른 정적 메서드를 추가할 필요 없이 TemperatureScale 열거 타입에 캘빈온도(KELVIN)를 추가하면 된다. 온도 단위에 대한 의존성을 개별 열거 타입 상수의 메서드 안으로 리팩터링해 넣을 수도 있다(ex. double 값을 받아 섭씨온도로 변환해주는 메서드를 열거 타입 상수 각각에 정의).



## Item 52. 다중정의는 신중히 사용하라

재정의한 메서드는 동적으로 선택되고, 다중정의한 메서드는 정적으로 선택된다.

```java
// 컬렉션 분류기 - 오류 발생!
public class CollectionClassifier {
  public static String classify(Set<?> s) {
    return "집합";
  }
  
  public static String classify(List<?> lst) {
    return "리스트";
  }
  
  public static String classify(Collection<?> c) {
    return "그 외";
  }
  
  public static void main(String[] args) {
    Collection<?>[] collections = {
      new HashSet<String>(),
      new ArrayList<BigInteger>(),
      new HashMap<String, String>().values()
    };
    
    for (Collection<?> c : collections)
      System.out.println(classify(c)); // "그 외"만 세 번 연달아 출력
  }
}
```

다중정의된 세 classify 중 어느 메서드를 호출할지가 컴파일타임에 정해져 직관과 어긋날 결과가 나온다. 런타임에는 타입이 매번 달라지지만, 호출할 메서드를 선택하는 데는 영향을 주지 못 한다.

```java
// 재정의된 메서드 호출 메커니즘
class Wine {
  String name() { return "포도주"; }
}

class SparklingWine extends Wine {
  @Override String name() { return "발포성 포도주"; }
}

class Champagne extends SparklingWine {
  @Override String name() { return "샴페인"; }
}

public class Overriding {
  public static void main(String[] args) {
    List<Wine> windList = List.of(
    	new Wine(), new SparklingWine(), new Chanpagne());
    
    for (Wine wine : wineList)
      System.out.println(wine.name()); // "포도주", "발포성 포도주", "샴페인"을 차례로 출력
  }
}
```

컴파일타임 타입이 모두 Wine인 것에 무관하게 항상 '가장 하위에서 정의한' 재정의 메서드가 실행된다. 재정의 처럼 런타임 타입에 기초해 메소드로 적절히 자동 분배하기 위해서는 CollectionClassifier의 모든 classify 메서드를 하나로 합친 후 instanceof로 명시적으로 검사하면 된다.

```java
public static String classify(Collection<?> c) {
  return c instanceof Set	 ? "집합" :
  			 c instanceof List ? "리스트" : "그 외";
}
```

다중정의가 혼동을 일으키는 상황을 피해야한다. **안전하고 보수적으로 가려면 매개변수 수가 같은 다중정의는 만들지 말자.** 가변인수를 사용하는 메서드라면 다중정의를 아예 하지 말아야 한다. **다중정의하는 대신 메서드 이름을 다르게 지어주는 방법도 있다.** ObjectOutputStream/ObjectInputStream 클래스의 write 메서드와 read가 좋은 예다. 이 메서드들은 모든 기본 타입과 일부 참조 타입용 변형을 가지고 있다. `writeBoolean(boolean)`, `writeInt(int)`, `readBoolean(boolean)`, `readInt(int)`와 같은 식이다.

생성자는 이름을 다르게 지을 수 없어 두 번째 생성자부터는 무조건 다중정의가 된다. 하지만 정적 팩터리라는 대안을 활용할 수 있는 경우가 많고, 재정의와 혼용될 걱정이 없다. 하지만 여러 생성자가 같은 수의 매개변수를 받아야 하는 경우를 완전히 피해가기는 힘든데, 다음과 같은 방법이 도움이 된다. 매개 변수 수가 같은 다중정의 메서드가 많더라도, 그중 어느 것이 주어진 매개변수 집합을 처리할지가 명확히 구분되면 된다. 특정 두 타입의 값을 서로 어느쪽으로든 형변환할 수 없으면 된다. 이 조건을 충족하면 어느 다중정의 메서드를 호출할지가 매개변수들의 런타임 타입만으로 결정된다.

자바 4까지는 모든 기본 타입이 모든 참조 타입과 근본적으로 달랐지만, 자바 5에서 오토박싱이 도입되면서 발생하는 문제가 있다.

```java
public class SetList {
  public static void main(String[] args) {
    Set<Integer> set = new TreeSet<>();
    List<Integer> list = new Arraylist<>();
    
    for (int i = -3; i < 3; i++) {
      set.add(i);
      list.add(i);
    }
    
    for (int i = 0; i < 3; i++) {
      set.remove(i);
      list.remove(i);
    }
    System.out.println(set + " " + list); // 결과로 "[-3, -2, -1] [-2, 0, 2]"를 출력한다.
  }
}
```

`set.remove(i)` 의 시그니처는 `remove(Object)` 다. 반면 `list.remove(i)` 는 다중정의된 `remove(int index)` 를 선택한다. 이 문제는 `list.remove`의 인수를 Integer로 형변환하여 올바른 다중정의 메서드를 선택하게 하면 해결된다(`Integer.valueOf` 를 이용해도 된다).

```java
for (int i = 0; i < 3; i++) {
  set.remove(i);
  list.remove((Integer) i); // 혹은 remove(Integer.valueOf(i))
}
```

자바 8에서 도입한 람다와 메서드 참조 역시 다중정의 시의 혼란을 키웠다.

```java
// Thread 생성자 호출
new Thread(System.out::println).start();

// ExecutorService의 submit 메서드 호출 - 컴파일 오류 발생
ExecutorService exec = Executors.newCachedThreadPool();
exec.submit(System.out::println);
```

submit 다중 정의 메서드 중에는 `Callable<T>` 를 받는 메서드도 있다. 만약 println이 다중정의 없이 단 하나만 존재했다면 이 submit 메서드 호출이 제대로 컴파일 됐을 것이다. 지금은 참조된 메서드(println)와 호출한 메서드(submit) 양쪽 다 다중정의되어, 다중정의 해소 알고리즘이 우리의 기대처럼 동작하지 않는다. `System.out::pringln`은 부정확한 메서드 참조다. 다중정의된 메서드(혹은 생성자)들이 함수형 인터페이스를 인수로 받을 때, 비록 서로 다른 함수형 인터페이스라도 인수 위치가 같으면 혼란이 생긴다. **메서드를 다중정의할 때, 서로 다른 함수형 인터페이스라도 같은 위치의 인수로 받아서는 안 된다.**

Object 외의 클래스 타입과 배열 타입은 근본적으로 다르다. Serializable과 Cloneable 외의 인터페이스 타입과 배열 타입도 근본적으로 다르다. String과 Throwable처럼 상위/하위 관계가 아닌 관련 없는 클래스들끼리도 클래스도 근본적으로 다르다. 이 외에도 어떤 방향으로도 형변환할 수 없는 타입 쌍들이 있다. 앞에서 나열한 간단한 예보다 복잡해지면 대부분 프로그래머는 어떤 다중정의 메서드가 선택될지 구분하기가 어렵다. 다중정의된 메서드 중 하나를 선택하는 규칙은 매우 복잡하며, 자바가 버전업될수록 더 복잡해지고 있다.

String은 자바 4부터 `contentEquals(StringBuffer)` 메서드를 가지고 있었다. 그런데 자바 5에서 StringBuffer, StringBuilder, String, CharBuffer 등의 비슷한 부류의 타입을 위한 공통 인터페이스로 CharSequence가 등장하였고, 자연스럽게 String에도 CharSequence를 받은 contentEquals가 다중정의되었다. 다행히 이 두 메서드는 같은 객체를 입력하면 완전히 같은 작업을 수행해주니 문제는 없다. 이처럼 어떤 다중정의 메서드가 불리는지 몰라도 기능이 똑같다면 신경 쓸 것이 없다. 이렇게 하는 가장 일반적인 방법은 상대적으로 더 특수한 다중정의 메서드에서 덜 특수한(더 일반적인) 다중정의 메서드로 일을 넘겨버리는(forward) 것이다.

```java
// 인수를 포워드하여 두 메서드가 동일한 일을 하도록 보장한다.
public boolean contentEquals(StringBuffer sb) {
  return contentEquals((CharSequence) sb);
}
```



## Item 53. 가변인수는 신중히 사용하라

가변인수 메서드는 명시한 타입의 인수를 0개 이상 받을 수 있다. 가변인수 메서드를 호출하면, 가장 먼저 인수의 개수와 길이가 같은 배열을 만들고 인수들을 이 배열에 저장하여 가변인수 메서드에 건네준다.

```java
// 간단한 가변인수 활용 예
static int sum(int... args) {
  int sum = 0;
  for (int arg : args)
    sum += arg;
  return sum;
}
```

가변 인수의 개수는 런타임에 (자동 생성된) 배열의 길이로 알 수 있다.

```java
// 인수가 1개 이상이어야 하는 가변인수 메서드 - 잘못 구현한 예
static int min(int... args) {
  if (args.length == 0)
    throw new IllegalArgumentException("인수가 1개 이상 필요합니다."); // 런타임에 실패
  int min = args[0];
  for (int i = 1; i < args.length; i++)
    if (args[i] < min)
      min = args[i];
  return min;
}
```

위 코드는 args 유효성 검사를 명시적으로 해야 하고, min의 초깃값을 Integer.MAX_VALUE로 설정하지 않고는 (더 명료한) for-each 문도 사용할 수 없다. 하지만 매개변수를 2개 받도록 하면 된다.

```java
// 인수가 1개 이상이어야 할 때 가변인수를 제대로 사용하는 방법
static int min(int firstArg, int... remainingArgs) {
  int min = firstArg;
  for (int arg : remainingArgs)
    if (arg < min)
      min = arg;
  return min;
}
```

가변인수는 인수 개수가 정해지지 않았을 때 아주 유용하다. 그런데 성능에 민감한 상황이라면 가변인수 메서드는 호출될 때마다 배열을 새로 하나 할당하고 초기화하기 때문에 좋지 않을 수 있다. 이 비용을 감당할 수는 없지만 가변인수의 유연성이 필요할 때 선택할 수 있는 패턴이 있다. 예를 들어 해당 메서드 호출의 95%가 인수를 3개 이하로 사용한다면, 인수가 0개인 것부터 4개인 것까지 5개를 다중정의하고, 마지막 다중정의 메서드가 가변인수를 사용하면 된다. EnumSet의 정적 팩터리도 이 기법을 사용해 열거 타입 집합 생성 비용을 최소화한다. EnumSet은 비트 필드를 대체하면서 성능까지 유지해야 하므로 아주 적절하게 활용한 예시이다.



## Item 54. null이 아닌, 빈 컬렉션이나 배열을 반환하라

컬렉션이나 배열 같은 컨테이너가 비었을 때 null을 반환하는 메서드를 사용할 때면 항상 방어코드를 넣어줘야한다. 클라이언트에서 방어 코드를 빼먹으면 오류가 발생할 수 있다.

```java
// 컬렉션이 비었으면 null을 반환 - 따라 하지 말 것!
private final List<Cheese> cheesesInStock = ...;

/**
 * @return 매장 안의 모든 치즈 목록을 반환한다.
 * 		단, 재고가 하나도 없다면 null을 반환한다.
 */
public List<Cheese> getCheeses() {
  return cheesesInStock.isEmpty() ? null : new ArrayList<>(cheesesInstock);
}

// null 상황을 처리하는 코드
List<Cheese> cheeses = shop.getCheeses();
if (cheeses != null && cheeses.contains(Cheese.STILTON))
  System.out.println("좋았어, 바로 그거야.");
```

빈 컨테이너를 할당하는 데도 비용이 드니 null을 반환하는 쪽이 낫다는 주장은 틀렸다. 성능 차이는 신경 쓸 수준이 되지 못하고, 빈 컬렉션과 배열은 굳이 새로 할당하지 않고도 반환할 수 있다.

```java
// 빈 컬렉션을 반환하는 올바른 예
public List<Cheese> getCheese() {
  return new ArrayList<>(cheesesInStock);
}
```

불변 객체는 자유롭게 공유해도 안전하기 때문에 빈 '불변' 컬렉션을 반환하는 것은 최적화 방법이다.

```java
// 최적화 - 빈 컬렉션을 매선 새로 할당하지 않도록 했다.
public List<Cheese> getCheeses() {
  return cheesesInStock.isEmpty() ? Collections.emptyList() : new ArrayList<>(cheesesInStock);
}
```

배열을 쓸 때도 마찬가지로 null 대신 길이가 0인 배열을 반환해라.

```java
// 길이가 0일 수도 있는 배열을 반환하는 올바른 방법.
public Cheese[] getCheeses() {
  return cheesesInStock.toArray(new Cheese[0]); // 건넨 배열은 원하는 반환 타입을 알려주는 역할을 한다. (Cheese[])
}
```

이 방식이 성능을 떨어뜨릴 것 같다면 길이 0짜리 배열(불변)을 미리 선언해두고 매번 그 배열을 반환하면 된다.

```java
// 최적화 - 빈 배열을 매번 새로 할당하지 않도록 했다.
private static final Cheese[] EMPTY_CHEESE_ARRAY = new Cheese[0];

public Cheese[] getCheeses() {
  return cheesesInStock.toArray(EMPTY_CHEESE_ARRAY); // cheesesInStock이 비었을 때면 언제나 EMPTY_CHEESE_ARRAY를 반환
}
```

단순히 성능을 개선할 목적이라면 toArray에 넘기는 배열을 미리 할당하는 것은 좋지 않다. 오히려 성능이 떨어질 수 있다.

```java
// 나쁜 예 - 배열을 미리 할당하면 성능이 나빠진다.
return cheesesInStock.toArray(new Cheese[cheesesInStock.size()]);
```

**null이 아닌, 빈 배열이나 컬렉션을 반환하자!**



## Item 55. 옵셔널 반환은 신중히 하라

자바 8 전에는 메서드가 특정 조건에서 값을 반환할 수 없으면 예외를 던지거나, null을 반환했다. 두 방법 모두 허점이 있다. 예외는 진짜 예외적인 상황에서만 사용해야 하며 예외를 생성할 때 비용이 만만치 않다. null을 반환하면 별도의 null 처리 코드를 추가해야한다. 그렇지 않으면 NullPointerException이 발생할 수 있다. 자바 8 부터 생긴  `Optional<T>`는 null이 아닌 T 타입 참조를 하나 담거나, 아무것도 담지 않을 수 있다. 옵셔널은 원소를 최대 1개 가질 수 있는 불변 컬렉션이다. 옵셔널을 반환하는 메서드는 예외를 던지는 메서드보다 유연하고 사용하기 쉬우며, null을 반환하는 메서드보다 오류 가능성이 작다.

```java
// 컬렉션에서 최댓값을 구한다(컬렉션이 비었으면 예외를 던진다).
public static <E extends Comparable<E>> E max(Collection<E> c) {
  if (c.isEmpty())
    throw new IllegalArgumentException("빈 컬렉션");
  
  E result = null;
  for (E e : c)
    if (result == null || e.compareTo(result) > 0)
      result = Objects.requireNonNull(e);
  
  return result;
}
```

```java
// 컬렉션에서 최댓값을 구해 Optional<E>로 반환한다.
public static <E extends Comparable<E>> Optional<E> max(Collection<E> c) {
  if (c.isEmpty())
    return Optional.empty();
  
  E result = null;
  for (E e : c)
    if (result == null || e.compareTo(result) > 0)
      result = Objects.requireNonNull(e);
  
  return Optional.of(result); // result로 null을 넣으면 NullPointerException 발생
}
```

null 값도 허용하는 옵셔널을 만들려면 `Optional.ofNullable(value)`를 사용하면 된다. **옵셔널을 반환하는 메서드에서는 절대  null을 반환하지 말자.**

비교자를 명시적으로 전달해 앞의 max 메서드를 스트림 버전으로 작성할 수도 있다.

```java
// 컬렉션에 최댓값을 구해 Optional<E>로 반환한다. - 스트림 버전
public static <E extends Comparable<E>> Optional<E> max(Collection<E> c) {
  return c.stream().max(Comparator.naturalOrder());
}
```

옵셔널은 검사 예외와 취지가 비슷하다. 반환 값이 없을 수도 있음을 API 사용자에게 명확히 알려준다.

메서드가 옵셔널을 반환한다면 클라이언트는 값을 받지 못했을 때 취할 행동을 선택해야 한다. 그중 하나는 기본값을 설정하는 방법이다.

```java
// 옵셔널 활용 1 - 기본값을 정해둘 수 있다.
String lastWordInLexicon = max(words).orElse("단어 없음...");
```

또는 상황에 맞는 예외를 던질 수 있다. 시리제 예외가 아니라 예외 팩터리를 건네면 예외가 실제로 발생하지 않는 한 예외 생성 비용은 들지 않는다.

```java
// 옵셔널 활용 2 - 원하는 예외를 던질 수 있다.
Toy myToy = max(toys).orElseThrow(TemperTantrumExcepion::new);
```

옵셔널에 항상 값이 채워져 있다고 확신한다면 그냥 곧바로 값을 꺼내 사용할 수 있다. 단 잘못 판단한 것이라면 NoSuchElementException이 발생한다.

```java
// 옵셔널 활용 3 - 항상 값이 채워져 있다고 가정한다.
Element lastNobleGas = max(Elements.NOBLE_GASES).get();
```

기본값을 설정하는 비용이 아주 커서 부담이 될 때는 `Supplier<T>`를 인수로 받는 orElseGet을 사용하면, 값이 처음 필요할 때 `Supplier<T>`를 사용해 생성하므로 초기 설정 비용을 낮출 수 있다.

isPresent 메서드는 옵셔널이 채워져 있으면 true를, 비어 있으면  false를 반환한다.

```java
// 부모 프로세스의 프로세스 ID를 출력하거나 없다면 "N/A" 출력
Optional<ProcessHandle> parentProcess = ph.parent();
System.out.println("부모 PID: " + (parentProcess.isPresent() ? String.valueOf(parentProcess.get().pid()) : "N/A"));

// Optional의 map을 사용한 방법
System.out.println("부모 PID: " + ph.parent().map(h -> String.valueOf(h.pid())).orElse("N/A"));
```

스트림을 사용하면 옵셔널들을 `Stream<Optional<T>>`로 받아서, 그중 채워진 옵셔널들에서 값을 뽑아 `Stream<T>`에 건네 담아 처리하는 경우가 드물지 않다. 자바 8에서는 다음과 같이 구현할 수 있다.

```java
streamOptionals
  .filter(Optional::isPresent) // 옵셔널에 값이 있다면
  .map(Optional::get)					 // 그 값을 꺼내 매핑
```

자바 9에서는 Optional에 `stream()`메서드가 추가되었다. 이 메서드는 Optional을 Stream으로 변환해주는 어댑터다.

```java
streamOfOptionals
  .flatMap(Optional::stream)
```

**컬렉션, 스트림, 배열, 옵셔널 같은 컨테이너 타입은 옵셔널로 감싸면 안 된다.** 빈 `Optional<List<T>>`보다는 `List<T>`를 반환하는 게 좋다. 빈 컨테이너를 그대로 반환하면 클라이언트에 옵셔널 처리 코드를 넣지 않아도 된다. **결과가 없을 수 있으며, 클라이언트가 이 상황을 특별하게 처리해야 한다면  `Optional<T>`를 반환하자.** 하지만 Optional도 새로 할당하고 초기화해야 하는 객체이고, 그 안에서 값을 꺼내려면 메서드를 호출해야 한다. 그래서 성능이 중요한 상황에서는 옵셔널이 맞지 않을 수 있다. OptionalInt, OptionalLong, OptionalDouble같이 기본 타입을 전용 옵셔널 클래스들이 있으니 **박싱된 기본 타입을 담은 옵셔널을 반환하는 일은 없도록 하자.** 단 Boolean, Byte, Character, Short, Float은 예외일 수 있다.

**옵셔널을 컬렉션의 키, 값, 원소나 배열의 원소로 사용하는 게 적절한 상황은 거의 없다.** 간혹 인스턴스 필드를 옵셔널로 선언하는 일은 있다.



## Item 56. 공개된 API 요소에는 항상 문서화 주석을 작성하라

API에는 잘 작성된 문서가 필요한데, 자바에서는  자바독이라는 유틸리티가 이를 도와준다. 자바독은 소스코드 파일에서 문서화 주석이라는 특수한 형태로 기술된 설명을 추려 API 문서로 변환해준다. 자바 버전이 올라가며 추가된 중요한 자바독 태그로는 자바 5의 `@literal`과 `@code`, 자바 8의 `@implSpec`, 자바 9의 `@index`를 꼽을 수 있다.

**API를 올바로 문서화하려면 공개된 모든 클래스, 인터페이스, 메서드, 필드 선언에 문서화 주석을 달아야 한다.** 직렬화할 수 있는 클래스라면 직렬화형태에 관해서도 적어야 한다. 기본 생성자에는 문서화 주석을 달 방법이 없으니 공개 클래스는 절대 기본 생성자를 사용하면 안된다.

**메서드용 문서화 주석에는 해당 메서드와 클라이언트 사이의 규양을 명료하게 기술해야 한다.** 상속용으로 설계된 클래스의 메서드가 아니라면 무엇을 하는지를 기술해야 한다. 문서화 주석에는 클라이언트가 해당 메서드를 호출하기 위한 전제조건을 모두 나열해야 한다. 또한 메서드가 성공적으로 수행된 후에 만족해야 하는 사후조건도 모두 나열해야 한다. 일반적으로 전제조건은 `@throw` 태그로 비검사 예외를 선언하여 암시적으로 기술하고 `@param` 태그를 이용해 그 조건에 영향받는 매개변수를 기술할 수도 있다. 전제조건과 사후조건뿐만 아니라 부작용도 문서화해야 한다.

메서드의 계약을 완벽히 기술하려면 모든 매개변수에 `@param` 태그를, 반환 타입이 void가 아니라면 `@return` 태그를, 발생할 가능성이 있는 모든 예외에 `@throws` 태그를 달아야한다.

```java
// BigInteger의 API 문서 예
/**
 * Returns the element at the specified position in this list.
 *
 * <p>This method is <i>not</i> guaranteed to run in constant time. 
 * In some implementations it may run in time proprotional to the element Position.
 *
 * @param		index index of element to return; must be non-negative and less than the size of this list
 * @return	the element at the specified position in this list
 * @throws	IndexOutOfBoundsException if the index is out of range
 *					({@code index < 0 || index >= this.siez()})
 */
E get(int index);
```

자바독 유틸리티는 문서화 주석을  HTML로 변환하므로 문서화 주석 안의  HTML 요소들이 최족  HTML 문서에 반영된다. `@thorws` 절에 사용된 `{@code}` 태그는 태그로 감싼 내용을 코드용 폰트로 렌더링하고, 태그로 감싼 내용에 포함된 HTML 요소나 다른 자바독 태그를 무시한다. 문서화 주석에 여러 줄로 된 코드 예시를 넣으려면 `{@code}` 태그를 다시 `<pre>` 태그로 감싸면 된다. 이렇게 하면 HTML의 탈출 메타문자를 쓰지 않아도 코드의 줄바꿈이 그대로 유지된다. 단, @기호에는 무조건 탈출문자를 붙여야 하니 애너테이션 사용 시에는 주의하자. 클래스를 상속용으로 설계할 때는 자기사용 패턴에 대해서도 문서에 남겨 다른 프로그래머에게 그 메서드를 올바로 재정의 하는 방법을 알려줘야 한다. 자기사용 패턴은 자바 8에 추가된 `@implSpec` 태그로 문서화한다. 이 주석은 해당 메서드와 하위 클래스 사이의 계약을 설명한다.

```java
/**
 * Returns true if this collection is empty.
 *
 * @implSpec
 * This implementation returns {@code this.size() == 0}.
 *
 * @return true if this collection is empty
 */
public boolean isEmpty() { ... }
```

자바독 명령줄에서  `-tag "implSpec:a:Implementation Require ments:"` 스위치를 켜주지 않으면 `@implSpec` 태그를 무시해버린다.

API 설명에 <, > & 등의 HTML 메타분자를 포함시키려면 특별한 처리를 해줘야 한다. 가장 좋은 방법은 `{@literal}` 태그로 감싸는 것이다. `{@code}` 태그와 비슷하지만 코드 폰트로 렌더링하지는 않는다.

```java
/**
 * A geometric series converges if {@literal |r| < 1}.
 */
// "A geometric series converges if |r| < 1."로 변환된다.
```

각 문서화 주석의 첫 번째 문장은 해당 요소의 요약 설명으로 간주된다. 요약 설명은 반드시 대상의 기능을 고유하게 기술해야 한다. 헷갈리지 않으려면 **한 클래스(혹은 인터페이스) 안에서 요약 설명이 똑같은 멤버(혹은 생성자)가 둘 이상이면 안 된다.** 다중정의된 메서드가 있다면 주의하자. 요약 설명에서는 마침표(.)에 주의해야 한다. 요약 설명이 끝나는 판단 기준은 처음 발견되는 {<마침표> <공백> <다음 문장 시작>} 패턴의 <마침표>까지다. 여기서 <공백>은 스페이스, 탭, 줄바꿈(혹은 첫 번째 블록 태그)이며 <다음 문장 시작>은 소문자가 아닌 문자다. 가장 좋은 해결책은 의도치 않은 마침표를 포함한 텍스트를 `{@literal}`로 감싸주는 것이다.

```java
/**
 * A suspect, such as Colonel Mustard or {@literal Mrs. Peacock}.
 */
public class Suspect { ... }
```

자바 10부터는 `{@summary}`라는 요약 설명 전용 태그가 추가되어, 다음처럼 깔끔하게 처리할 수 있다.

```java
/**
 * {@summary A suspect, such as Colonel Mustard or Mrs. Peacock.}
 */
public class Suspect { ... }
```

메서드와 생성자의 요약설명은 해당 메서드와 생성자의 동작을 설명하는 (주어가 없는) 동사구여야 한다.

- ArrayList(int initialCapacity): Constructs an empty list with the spicified initial capacity.
- Collection.size(): Returns the number of elements in this collection.

클래스, 인터페이스, 필드의 요약 설명은 대상을 설명하는 명사절이어야 한다. 클래스와 인터페이스의 대상은 그 인스턴스이고, 필드의 대상은 필드 자신이다.

- Instant: An instantaneous point on the time-line.
- Math.PI: The **double** value that is closer than any other to pi, the ratio of the circumference of a circle to its diameter.

자바 9부터는 자바독이 생성한  HTML 문서에 검색(색인) 기능이 추가되었다. 클래스, 메서드, 필드 같은 API 요소의 색인은 자동으로 만들어지며, 원한다면 `{@index}` 태그를 사용해 API에서 중요한 용어를 추가로 색인화할 수 있다.

```java
/**
 * this method complies with the {@index IEEE 754} standard
 */
```

**제네릭 타입이나 제네릭 메서드를 문서화할 때는 모든 타입 매개변수에 주석을 달아야 한다.**

```java
/**
 * An object that maps keys to values. A map cannot contain duplicate keys;
 * each key can map to at most one value.
 *
 * (Remainder omitted)
 *
 * @param <K> the type of keys maintained by this map
 * @param <V> the type of mapped values
 */
public interface Map<K, V> { ... }
```

**열거 타입을 문서화할 때는 상수들에도 주석을 달아야 한다.**

```java
/**
 * An instrument section of a symphony orchestra.
 */
public enum OrchestraSection {
  /** Woodwinds, such as flute, clarinet, and oboe. */
  WOODWIND,
  
  /** Brass instruments, such as french horn and trumpet. */
  BRASS,
  
  /** Percussion instruments, such as timpani and cymbals. */
  PERCUSSION,
  
  /** Stringed instruments, such as violin and cello. */
  CTRING;
}
```

**애너테이션 타입을 문서화할 때는 멤버들에도 모두 주석을 달아야 한다.** 필드 설명은 명사구로, 애너테이션 타입의 요약 설명은 프로그램 요소에 이 애너테이션을 단다는 것이 어떤 의미인지를 설명하는 동사구로 한다.

```java 
/**
 * Indicates that the annotated method is a test mehotd that must throw the designated exception to pass.
 */
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
  /**
   * The exception that the annotated test method must throw in order to pass.
   * (The Test is Permitted to throw any subtype of the type described by this class object.)
   */
  Class<? extends Throwable> value();
}
```

패키지를 설명하는 문서화 주석은  package-info.java 파일에 작성한다. 이 파일은 패키지 선언을 반드시 포함해야 하며 피키지 선언 관련 애너테이션을 추가로 포함할 수도 있다. 자바 9부터 지원하는 모듈 시스템도 비슷하게 module-info.java 파일에 작성하면 된다.

**클래스 혹은 정적 메서드가 스레드 안전하든 그렇지 않든, 스레드 안전 수준을 반드시 API 설명에 포함해야 한다.** 또한, 직렬화할 수 있는 클래스라면 직렬화 형태도 API 설명에 기술해야 한다.

자바독은 메서드 주석을 상속시킬 수 있다. 문서화 주석이 없는 API 요소를 발견하면 자바독이 가장 가까운 문서화 주석을 찾아준다. 또한 `{@inheritDoc}` 태그를 사용해 상위 타입의 문서화 주석 일부를 상속할 수 있다. 이 기능을 활용하면 거의 똑같은 문서화 주석 여러 개를 유지보수하는 부담을 줄일 수 있지만, 사용하기 까다롭고 제약도 조금 있다.

비록 공개된 모든 API 요소에 문서화 주석을 달았더라도, 이것만으로는 충분하지 않을 때가 있다. 여러 클래스가 상호작용하는 복잡한 API라면 문서화 주석 외에도 전체 아키텍처를 설명하는 별도의 설명이 필요할 때가 있다. 이런 설명 문서가 있다면 관련 클래스나 패키지의 문서화 주석에서 그 문서의 링크를 제공해 주면 좋다.

자바 7에셔는 명령줄에  `-Xdoclint` 스위치를 켜주면 자바독 문서를 올바르게 작성했는지 확인하는 기능이 활성화되고, 자바 8부터는 기본적으로 작동한다. 자바 9와 10의 자바독은 기본적으로 HTML 4.01 문서를 생성하지만, 명령줄에서 -html5 스위치를 켜면  HTML 5 버전으로 만들어준다.
