---
title: EffectiveJava_Chap7
excerpt: 이펙티브 자바 책 7장 정리
categories: study
---

# 7장. 람다와 스트림

## Item 42. 익명 클래스보다는 람다를 사용하라

### 람다

예전에는 자바에서 함수 타입을 표현할 때 추상 메서드를 하나만 담은 인터페이스를 사용했는데, 이런 인터페이스의 인스턴스를 함수 객체라고 했다. 함수 객체를 만드는 주요 수단은 익명 클래스이다.

```java
// 익명 클래스의 인스턴스를 함수 객체로 사용 - 낡은 기법!
Collections.sort(words, new Comparator<String>() { // Comparator 인터페이스는 정렬을 담당하는 추상 전략
  public int compare(String s1, String s2) {
    return Integer.compare(s1.length(), s2.length());
  }
});
```

익명 클래스 방식은 코드가 너무 길기 때문에 자바는 함수형 프로그래밍에 적합하지 않다. 자바 8에 와서 함수형 인터페이스라 부르는 이 인터페이스들의 인스턴스를 람다식을 사용해 만들 수 있게 되었다. 람다는 함수나 익명 클래스와 개념은 비슷하지만 코드는 훨씬 간결하다.

```java
// 람다식을 함수 객체로 사용 - 익명 클래스 대체. 간결! 명확!
Collections.sort(words, (s1, s2) -> Integer.compare(s1.length(), s2.length()));
```

여기서 람다, 매개변수(s1, s2), 반환값의 타입은 각각 (`Comparator<String>`), String, int지만 코드에서는 언급이 없다. 컴파일러가 문맥을 살펴 타입을 추론해주기 때문이다. **타입을 명시해야 코드가 더 명확할 때만 제외하고는, 람다의 모든 매개변수 타입은 생략하자.** 컴파일러가 "타입을 알 수 없다"는 오류를 낼 때만 해당 타입을 명시하면 된다.

더욱 간결하게 만들 수도 있다.

```java
// 람다 자리에 비교자 생성 메서드를 사용
Collections.sort(words, comparingInt(String::length));
// 자바 8에서 List 인터페이스에 추가된 sort 메서드 이용
words.sort(comparingInt(String::length));
```

### 람다의 활용

```java
// 상수별 클래스 몸체와 데이터를 사용한 열거 타입
public enum Operation {
  PLUS("+") {
    public double apply(double x, double y)	{ return x + y; }
  },
  MINUS("-") {
    public double apply(double x, double y)	{ return x - y; }
  },
  TIMES("*") {
    public double apply(double x, double y)	{ return x * y; }
  },
  DIVIDE("/") {
    public double apply(double x, double y)	{ return x / y; }
  };
  
  private final String symbol;
  
  Operation(String symbol) { this.symbol = symbol; }
  
  @Override public String toString() { return symbol; }
  public abstract double apply(double x, double y);
}
```

상수별 클래스 몸체를 구현하는 방식보다는 열거 타입에 인스턴스 필드를 두는 것이 좋은데, 람다를 이용하면 쉽게 구현할 수 있다. 단순히 각 열거 타입 상수의 동작을 람다로 구현해 생성자에 넘겨, 이 람다를 인스턴스 필드로 저장해두고 apply 메서드에서 필드에 저장된 람다를 호출하기만 하면 된다.

```java
// 함수 객체(람다)를 인스턴스 필드에 저장해 상수별 동작을 구현한 열거 타입
public enum Operation {
  PLUS ("+", (x, y) -> x + y),
  MINUS ("-", (x, y) -> x - y),
  TIMES ("*", (x, y) -> x * y),
  DIVIDE ("/", (x, y) -> x / y);
  
  private final String symbol;
  private final DoubleBinaryOperator op; // DoubleBinaryOperator 인터페이스 변수
  // DoubleBinaryOperator는 java.util.function 패키지가 제공하는 다양한 함수 인터페이스 중 하나
  // double 타입 인수 2개를 받아 double 타입 결과를 돌려준다.
  
  Operation(String symbol, DoubleBinaryOperator op) {
    this.symbol = symbol;
    this.op = op;
  }
  
  @Override public String toString() { return symbol; }
  
  public double apply(double x, double y) {
    return op.applyAsDouble(x, y);
  }
}
```

### 람다 사용 시 주의할 점

다음과 같은 점 때문에 람다가 아니라 상수별 클래스 몸체나 익명 클래스 등을 사용해야 하는 경우가 생긴다. (람다 사용 불가능)

- 람다는 이름이 없고 문서화도 할 수 없다. 따라서 코드 자체로 동작이 명확히 설명되지 않거나 코드 줄 수가 많아지면 람다를 쓰지 말아야 한다.
- 열거 타입 생성자에 넘겨지는 인수들의 타입도 컴파일타임에 추론되기 때문에 열거 타입 생성자 안의 람다는 열거 타입의 인스턴스 멤버에 접근할 수 없다.
- 람다는 추상 클래스의 인스턴스를 만들 수 없고, 추상 메서드가 여러 개인 인터페이스의 인스턴스를 만들 수 없다.
- 람다는 자신을 참조할 수 없다. (람다에서의 this 키워드는 바깥 인스턴스를 가리킨다.)
- 람다도 익명 클래스철머 직렬화 형태가 구현별로 다를 수 있다. (대신에 private 정접 중첩 클래스의 인스턴스를 사용하자.)



## Item 43. 람다보다는 메서드 참조를 사용하라

람다가 익명 클래스보다 나은 점 중 가장 큰 특징은 간결함인데, 메서드 참조를 사용하면 함수 객체를 람다보다 더 간결하게 만들 수 있다.

```java
// 주어진 키가 맵 안에 아직 없다면 주어진 {키, 값} 쌍을 저장
// 키가 이미 있다면 함수를 현재 값과 주어진 값에 적용한 다음 {키, 함수의 결과} 쌍을 저장
map.merge(key, 1, (count, incr) -> count + incr);
```

람다 대신 메서드 참조를 전달하면 더욱 간결하게 만들 수 있다.

```java
map.merge(key, 1, Integer::sum);
```

매개변수 수가 늘어날수록 메서드 참조로 제거할 수 있는 코드양도 늘어나지만, 매개변수의 이름 자체가 가이드가 되는 경우에는 람다를 사용하는 것이 읽기 쉽고 유지보수에 좋을 수 있다.

람다로 할 수 없는 일이라면 메서드 참조로도 할 수 없지만(예외적으로 제네릭 함수 타입의 경우, 메서드 참조 표현식으로 구현이 가능하나 람다식으로는 불가능), 메서드 참조를 사용하는 편이 보통 더 짧고 간결하다. 람다로 작성할 코드를 새로운 메서드에 담은 다음, 람다 대신 그 메서드 참조를 사용할 수 있다. 메서드 참조에는 기능을 잘 드러내는 이름을 지어줄 수 있고, 문서도 남길 수 있다.

때론 람다가 메서드 참조보다 간결한 경우도 있다. 다음의 코드가 GoshThisClassNameIsHumongous 클래스 안에 있다고 가정해보자.

```java
service.execute(GoshThisClassNameIsHumongous::action);

// 람다로 대체
service.execute(() -> action()); // 짧고 명확함
```

비슷한 예로 `java.util.function` 패키지가 제공하는 제네릭 정적 팩터리 메서드인 `Function.identity()`를 사용하기보다는 똑같은 기능의 람다`(x -> x)`를 직접 사용하는 편이 코드도 짧고 명확하다.

메서드 참조의 유형은 다음과 같이 다섯 가지가 있다.

| 메서드 참조 유형    | 예                     | 같은 기능을 하는 람다                                   |
| ------------------- | ---------------------- | ------------------------------------------------------- |
| 정적                | Integer::parseInt      | str -> Integer.parseInt(str)                            |
| 한정적 (인스턴스)   | Instant.now()::isAfter | Instant then = Instant.now();<br />t -> then.isAfter(t) |
| 비한정적 (인스턴스) | String::toLoserCase    | str - str.toLowerCase()                                 |
| 클래스 생성자       | TreeMap<K,V>::new      | () -> new TreeMap<K,V>()                                |
| 배열 생성자         | int[]::new             | len -> new int[len]                                     |

인스턴스 메서드를 참조하는 유형 두 가지는 수신 객체를 특정하는 한정적 인스턴스 메서드 참조와 수신 객체를 특정하지 않는 비한정적 인스턴스 메서드 참조가 있다. 한정적 참조는 근본적으로 정적 참조와 비슷하다. 즉, 함수 객체가 받는 인수와 참조되는 메서드가 받는 인수가 똑간다. 반면 비한정적 참조에서는 함수 객체를 적용하는 시점에 수신 객체를 알려준다. 이를 위해 수신 객체 전달용 매개변수가 매개변수 목록의 첫 번째로 추가되고, 그 뒤로는 참조되는 메서드 선언에 정의된 매개변수들이 뒤따른다. 이는 주로 스트림 파이프라인에서의 매핑과 필터 함수에 쓰인다.



## Item 44. 표준 함수형 인터페이스를 사용하라.

자바가 람다를 지원하면서 상위 클래스의 기본 메서드를 재정의해 원하는 동작을 구현하는 템플릿 메서드 패턴 대신 함수 객체를 받는 정적 팩터리나 생성자를 제공할 수 있게 되었다. 이때 함수형 매개변수 타입을 올바르게 선택해야 한다.

LinkedHashMap의 removeEldestEntry 메서드를 재정의하면 캐시로 사용할 수 있다. 맵에 새로운 키를 추가하는 put 메서드는 이 메서드를 호출하여 true가 반환되면 맵에서 가장 오래된 원소를 제거한다.

```java
// 맵에 원소가 100개가 될 때까지 커지다가, 그 이상이 되면 새로운 키가 더해질 때마다 가장 오래된 원소를 하나씩 제거
protected boolean removeEldestEntry(Map.Entry<K,V> eldest) {
  return size() > 100; // 인스턴스 메서드라 size()로 원소수 를 알아 낼 수 있음.
}
```

팩터리나 생성자를 호출할 때는 맵의 인스턴스가 존재하지 않기 때문에 생성자에 넘기는 함수 객체는 이 맵의 인스턴스 메서드가 아니다. 따라서 맵은 자기 자신도 함수 객체에 건네줘야 한다. 이를 반영한 함수형 인터페이스는 다음처럼 선언할 수 있다.

```java
// 불필요한 함수형 인터페이스 - 대신 표준 함수형 인터페이스를 사용하라.
@FunctionalInterface interface EldestEntryRemovalRunction<K,V> {
  boolean remove(Map<K,V> map, Map.Entry<K,V> eldest);
}
```

자바 표준 라이브러리에 이미 같은 모양의 인터페이스(`BiPredicate<Map<K,V>, Map.Entry<K,V>>`)가 준비되어 있어 굳이 쓸 필요는 없다. `java.util.function` 패키지를 보면 다양한 용도의 표준 함수형 인터페이스가 담겨 있다. **필요한 용도에 맞는 게 있다면, 직접 구현하지 말고 표준 함수형 인터페이스를 활용하라.** 그러면 API가 다루는 개념의 수가 줄어들고, 유용한 디폴트 메서드가 많아 상호운용성이 좋아질 것이다.

`java.utril.function` 패키지에는 총 43개의 인터페이스가 담겨있다. 기본 인터페이스 6개를 알면 나머지는 충분히 유추해 낼 수 있다. 이 기본 인터페이스들은 모두 참조 타입용이다.

| 인터페이스         | 함수 시그니처       | 예                  | 설명                                |
| ------------------ | ------------------- | ------------------- | ----------------------------------- |
| UnaryOperator\<T>  | T apply(T t)        | String::toLowerCase | 인수 1개, 반환값과 인수 타입이 같음 |
| BinaryOperator\<T> | T apply(T t1, T t2) | BigInteger::add     | 인수 2개, 반환값과 인수 타입이 같음 |
| Predicate\<T>      | boolean test(T t)   | Collection::isEmpty | 인수 하나를 받아 boolean 반환       |
| Function<T,R>      | R apply(T t)        | Arrays::asList      | 인수와 반환 타입이 다름             |
| Supplier\<T>       | T get()             | Instant::now        | 인수를 받지 않고 값을 반환          |
| Consumer\<T>       | void accept(T t)    | System.out::println | 인수를 하나 받고 반환값은 없음      |

기본 인터페이스는 기본 타입인 int, long, double용으로 각 3개씩 변형이 생겨난다(ex. IntPredicate). 유일하게 Function의 변형은 반환 타입이 매개변수화 됐다(ex. `LongFunction<int[]>`). Function 인터페이스(입력 타입과 결과 타입이 항상 다르다)의 변형은 총 9개가 더 있다. 타입이 모두 기본 타입이면 접두어로 SrcToResult를 사용한다. `LongToIntFunction`(long을 받아 int 반환)과 같이 사용하면 된다(총 6개). 입력이 객체 참조이고 결과가 기본형인 경우는 입력을 매개변수화하고 접두어로 ToResult를 사용한다. `ToLongFunction<int[]>`(int[] 인수를 받아 long을 반환)과 같이 사용하면 된다(총 3개).

기본 인터페이스의 인수 2개짜리 변형은 총 9개가 있다.

- 인수를 2개씩 받는 기본 함수형 인터페이스: `BiPredicate<T,U>`, `BiFunction<T,U,R>`, `BiConsumer<T,U>`
- 기본 타입을 반환하는 변형:  `ToIntBiFunction<T,U>`, `ToLongBiFunction<T,U>`, `ToDoubleBiFunction<T,U>`
- 객체 참조와 기본 타입하나를 받는 Consumer: `ObjIntConsumer<T>`, `ObjLongConsumer<T>`, `ObjDoubleConsumer<T>`

BooleanSupplier 인터페이스는 boolean을 반환하도록 한 Supplier의 변형이다. Predicate와 그 변형 4개도 boolean 값을 반환할 수 있다. 표준 함수형 인터페이스 대부분은 기본 타입만 지원한다. 그렇다고 기본 함수형 인터페이스에 박싱된 기본 타입을 넣어 사용하지는 말자.

대부분의 상황에서는 직접 작성하는 것보다 표준 함수형 인터페이스를 사용하는 편이 낫다. 그렇다면 코드를 직접 작성해야 할 때는 언제인가? 다음 세 가지 조건중 하나 이상을 만족한다면 전용 함수형 인터페이스를 구현을 고민해야 한다.

- 자주 쓰이며, 이름 자체가 용도를 명확히 설명해준다.
- 반드시 따라야 하는 규약이 있다.
- 유용한 디폴트 메서드를 제공할 수 있다.

`@FunctionalInterface` 애너테이션을 사용하는 이유는 `@Override` 를 사용하는 이유와 비슷하다. 프로그래머의 의도를 명시하는 것으로, 크게 세 가지 목적이 있다.

1. 해당 클래스의 코드나 설명 문서를 읽을 이에게 그 인터페이스가 람다용으로 설계된 것임을 알려준다.
2. 해당 인터페이스가 추상 메서드를 오직 하나만 가지고 있어야 컴파일되게 해준다.
3. 유지보수 과정에서 누군가 실수로 메서드를 추가하지 못하게 막아준다.

**직접 만든 함수형 인터페이스에는 항상 `@FunctionalInterface` 애너테이션을 사용하라.**

서로 다른 함수형 인터페이스를 같은 위치의 인수로 받는 메서드들을 다중정의해서는 안 된다. 이 경우 올바른 메서드를 알려주기 위해 형변환을 해야하는 경우가 자주 발생한다.



## Item 45. 스트림은 주의해서 사용하라

자바 8에 추가된 스트림 API는 다량의 데이터 처리 작업에 유용하다. 스트림은 데이터 원소의 유한 혹은 무한 시퀀스를 뜻한다. 스트림 파이프라인은 이 원소들로 수행하는 연산 단계를 표현하는 계념이다. 스트림 안의 데이터 원소들은 객체 참조나 기본 타입 값(int, long, double)이다.

스트림 파이프라인은 소스 스트림에서 시작해 종단 연산으로 끝나며, 그 사이에 하나 이상의 중간 연산이 있을 수 있다. 각 중간 연산은 스트림을 어떠한 방식으로 변환하는데, 각 원소에 함수를 적용하거나 필터링 할 수 있다. 종단 연산은 마지막 중간 연산이 내놓은 스트림의 원소를 정렬해 컬렉션에 담거나, 특정 원소를 선택하거나, 모든 원소를 출력하는 작업을 할 수 있다.

스트림 파이프라인은 지연 평가된다. 평가는 종단 연산 호출시 이루어지며, 종단 연산에 쓰이지 않는 데이터 원소는 계산에 쓰이지 않아 무한 스트림을 다룰 수 있게 한다. 그리고 스트림 API는 메서드 연쇄를 지원해 파이프라인 여러 개를 연결해 표현식 하나로 만들 수 있다.

기본적으로 스트림 파이프라인은 순차적으로 수행된다. 파이프라인을 병렬로 실행하려면 파이프라인을 구성하는 스트림 중 하나에서 parallel 메서드를 호출해주기만 하면 되나, 효과를 볼 수 있는 상황은 많지 않다.

### 스트림 사용 예

스트림을 제대로 사용하면 프로그램이 짧고 깔끔해지지만, 잘못 사용하면 읽기 어렵도 유지보수도 힘들어진다.

```java
// 사전 하나를 훑어 원소 수가 많은 아나그램 그룹들을 출력
public class Anagrams {
  public static void main(String[] args) throws IOException {
    File dictionary = new File(args[0]);
    int minGroupSize = Integer.parseInt(args[1]);
    
    Map<String, Set<String>> groups = new HashMap<>();
    try (Scanner s = new Scanner(dictionary)) {
      while (s.hasNext()) {
        String word = s.next();
        groups.computeIfAbsent(alphabetize(word), (unused) -> new TreeSet<>()).add(word);
      }
    }
    
    for (Set<String> group : groups.values())
      if (group.size() >= minGroupSize)
        System.out.println(group.size() + ": " + group);
  }
  
  private static String alphabetize(String s) {
    char[] a = s.toCharArray();
    Arrays.sort(a);
    return new String(a);
  }
}
```

computeIfAbsent 메서드는 맵 안에 키가 있는지 찾은 다음, 있으면 단순히 그 키에 매핑된 값을 반환하고, 키가 없으면 건네진 함수 객체를 키에 적용하여 값을 계산한 다음 그 키와 값을 매핑해놓고, 계산도니 값을 반환한다.

```java
// 스트림을 과하게 사용 - 따라 하지 말 것!
public class Anagrams {
  public static void main(String[] args) throws IOException {
    File dictionary = new File(args[0]);
    int minGroupSize = Integer.parseInt(args[1]);
   
    try (Stream<String> words = Files.lines(dictionary)) {
    	word.collect(
      		groupingBy(word -> word.chars().sorted()
                  	.collect(StringBuilder::new,
                        (sb, c) -> sb.append((char) c),
                        StringBuilder::append).toString()))
      .values().stream()
      .filter(group -> group.size() >= minGroupSize)
      .map(group -> group.size() + ": " + group)
      .forEach(System.out::println);
    }
  }
}
```

스트림을 과용하면 프로그램을 읽거나 유지보수하기 어려워진다. 스트림을 적당히 사용하면 원래 코드보다 짧고 명확하게 만들 수도 있다.

```java
public class Anagrams {
  public static void main(String[] args) throws IOException {
    File dictionary = new File(args[0]);
    int minGroupSize = Integer.parseInt(args[1]);
    
    try (Stream<String> words = Files.lines(dictionary)) { 
      words.collect(groupingBy(word -> alphabetize(word)))
        .values().stream()
        .filter(group -> group.size() >= minGroupSize)
        .forEach(group -> System.out.println(g.size() + ": " + group));
    }
  }
  
  // alphabetize 메서드는 동일
}
```

람다에서는 타입 이름을 자주 생략하므로 매개변수 이름을 잘 지어야 스트림 파이프라인의 가독성이 유지된다. alphabetize 메서드도 스트림을 사용해 구현할 수는 있지만 char 값들을 처리할 때는 스트림을 삼가는 편이 낫다.

### 언제 스트림을 사용할까?

기존 코드는 스트림을 사용하도록 리팩터링하되, 새 코드가 더 나아 보일 때만 반영하는 것이 좋다.

다음과 같은 일을 수행해야 한다면 스트림은 적합하지 않다.

- 코드 블록에서는 범위 안의 지역변수를 읽고 수정할 수 있다. 하지만 람다에서는 final이거나 사실상 final인 변수만 읽을 수 있고, 지역변수를 수정하는 것은 불가능하다.
- 코드 블록에서는 return 문을 사용해 메서드에서 빠져나가거나, break나 continue 문으로 블록 바깥의 반복문을 종료하거나 반복을 한 번 건너뛸 수 있다. 또한 메서드 선언에 명시된 검사 예외를 던질 수 있다. 하지만 람다로는 이 중 어떤 것도 할 수 없다.

반대로 다음 작업들에는 스트림이 안성맞춤이다.

- 원소들의 시퀀스를 일관되게 변환한다.
- 원소들의 시퀀스를 필터링한다.
- 원소들의 시퀀스를 하나의 연산을 사용해 결합한다(더하기, 연결하기, 최소값 구하기 등).
- 원소들의 시퀀스를 컬렉션에 모은다(아마도 공통된 속성을 기준으로 묶어가며).
- 원소들의 시퀀스에서 특정 조건을 만족하는 원소를 찾는다.

한 데이터가 파이프라인의 여러 단계를 통과할 때 이 데이터의 각 단계에서의 값들에 동시에 접근하기는 어렵다. 가능한 경우, 앞 단계의 값이 필요할 때 매핑을 거꾸로 수행할 수는 있다.

```java
// 소수 (무한) 스트림을 반환하는 메서드
static Stream<BigInteger> primes() {
  return Stream.iterate(TWO, BigInteger::nextProbablePrime);
}

// 메르센 소수를 출력하는 프로그램
public static void main(String[] args) {
  primes().map(p -> TWO.pow(p.intValueExact()).subtract(ONE))
    .filter(mersenne -> mersenne.isProbablePrime(50)) // 매직넘버 50은 소수성 검사가 true를 반환할 확률을 제어
    .limit(20)
    .forEach(System.out::println);
}
```

여기서 메르센 소수의 앞에 지수(p)를 출력하길 원한다면, 첫 번째 중간 연산에서 수행한 매핑을 거꾸로 수행해 메르센 수의 지수를 계산해낼 수 있다.

```java
// 지수는 숫자를 이진수로 표현한 다음 몇 비트인지를 세면 된다.
.forEach(mp -> System.out.println(mp.bitLength() + ": " + mp));
```

### 스트림과 반복 중 어느 것이 나은가

두 집합의 원소들로 만들 수 있는 가능한 모든 조합을 계산하는 것을 두 집합의 데카르트 곱이라고 한다.

```java
// 데카르트 곱 계산을 반복 방식으로 구현
private static List<Card> newDeck() {
  List<Card> result = new ArrayList<>();
  for (Suit suit : Suit.values())
    for (Rank rank : Rank.values())
      result.add(new Card(suit, rank));
  return result;
}
```

동일한 내용을 스트림으로 구현할 수 있다. flatMap은 스트림의 원소 각각을 하나의 스트림으로 매핑한 다음 그 스트림들을 다시 하나의 스트림으로 합친다. 이를 평탄화(flattening)라고도 한다.

```java
// 데카르트 곱 계산을 스트림 방식으로 구현 - 중첩된 람다 사용
private static List<Card> newDeck() {
  return Stream.of(Suit.values())
    .flatMap(suit ->
        Stream.of(Rank.values())
            .map(rank -> new Card(suit, rank)))
    .collect(toList());
}
```

스트림과 반복 중 어느 쪽이 나은지 확신하기 어렵다면 둘 다 해보고 더 나은 쪽을 선택하면 된다.



## Item 46. 스트림에서는 부작용 없는 함수를 사용하라

스트림은 그저 또 하나의 API가 아닌, 함수형 프로그래밍에 기초한 패러다임이다. 스트림 패러다임의 핵심은 계산을 일련의 변환으로 재구성하는 부분이다. 이때 각 변환 단계는 가능한 한 이전 단계의 결과를 받아 처리하는 순수 함수여야 한다. 순수 함수는 오직 입력만이 결과에 영향을 주는 함수이다. 이를 위해 스트림 연산에 건네는 함수 객체는 모두 부작용(side effect)이 없어야 한다.

```java
// 스트림 패러다임을 이해하지 못한 채 API만 사용한 예시 - 따라 하지 말 것!
Map<String, Long> freq = new HashMap<>();
try (Stream<String> words = new Scanner(file).tokens()) {
  words.forEach(word -> {
    freq.merge(word.toLowerCase(), 1L, Long::sum);
  });
}
```

이 코드에서 모든 연산은 forEach에서 일어나는데, forEach가 그저 스트림이 수행한 연산 결과를 보여주는 일 이상을 하는 것은 좋지 않다.

```java
// 스트림을 제대로 활용해 빈도표를 초기화
Map<String, Long> freq;
try (Stream<String> words = new Scanner(file).tokens()) {
  freq = words
    .collect(groupingBy(String::toLowerCase, countion()));
}
```

forEach 연산은 종단 연산 중 기증이 가장 적고 가장 '덜' 스트림답다. 병렬화도 불가능하다. **forEach 연산은 스트림 계산 결과를 보고할 때만 사용하고, 계산하는 데는 쓰지 말자.**

`java.util.stream.Collectors` 클래스는 메서드가 39개나 있지만, 복잡한 세부 내용을 잘 몰라도 이 API의 장점을 대부분 활용할 수 있다. 수집기를 사용하면 스트림의 원소를 손쉽게 컬렉션으로 모을 수 있다. 수집기는 세 가지로, `toList()`, `toSet()`, `toCollection(collectionFactory)`가 있다. 이들은 차례로 리스트, 집합, 지정한 컬렉션 타입을 반환한다.

```java
// 빈도표에서 가장 흔한 단어 10개를 뽑아내는 파이프라인
List<String> topTen = freq.keySet().stream()
  .sorted(comparing(freq::get).reversed()) // comparing 메서드는 키 추출 함수를 받는 비교자 생성 메서드
  .limit(10)
  .collect(toList());
```

### toMap 메서드

Collectors의 나머지 36개 메서드들 중 대부분은 스트림을 맵으로 취합하는 기능으로, 진짜 컬렉션에 취합하는 것보다 훨씬 복잡하다. 스트림의 각 원소는 키 하나와 값 하나에 연관되어 있다. 그리고 다수의 스트림 원소가 같은 키에 연관될 수 있다. 가장 간단한 맵 수집기는 스트림 원소를 키에 매핑하는 함수와 값에 매핑하는 함수를 인수로 받는 `toMap(keyMapper, valueMapper)`이다.

```java
// toMap 수집기를 사용하여 문자열을 열거 타입 상수에 매핑
private static final Map<String, Operation> stringToEnum = 
  Stream.of(values()).collect(toMap(Object::toString, e -> e));
```

이 간단한 toMap 형태는 스트림의 각 원소가 고유한 키에 매핑되어 있을 때 적합하다.

toMap에 키 매퍼와 값 매퍼는 물론 병합 함수까지 제공할 수 있다. 병합 함수의 형태는 `BinaryOperator<U>`이며, 여기서 U는 해당 맵의 값 타입이다. 같은 키를 공유하는 값들은 이 병합 함수를 사용해 기존 값에 합쳐진다.

```java
// 각 키와 해당 키의 특정 원소를 연관 짓는 맵을 생성하는 수집기
Map<Artist, Album> topHits = albums.collect(
	toMap(Album::artist, a->a, maxBy(comparing(Album::sales)))); // 음악가와 그 음악과의 베스트 앨범을 짝 지음
```

인수가 3개임 toMap은 충돌이 나면 마지막 값을 취하는 수집기를 만들 때도 유용하다.

```java
toMap(keyMapper, valueMapper, (oldVal, newVal) -> newVal)
```

toMap은 네 번째 인수로 맵 팩터리를 받는데, 이 인수로 EnumMap이나 TreeMap처럼 원하는 특정 맵 구현체를 직접 지정할 수 있다.

위의 세 가지 toMap에는 변종이있다. 그중 toConcurrentMap은 병렬 실행된 후 결과로 ConcurrentHashMap 인스턴스를 생성한다.

### groupingBy 메서드

Collectors가 제공하는 또 다른 메서드인 groupingBy 메서드는 입력으로 분류 함수를 받고 출력으로는 원소들을 카테고리별로 모아 놓은 맵을 담은 수집기를 반환한다. 그리고 이 카테고리가 해당 원소의 맵 키로 사용된다. 다중 정의 된 groupingBy 중 형태가 가장 간단한 것은 분류 함수 하나를 인수로 받아 맵을 반환한다. 반환된 맵에 담긴 각각의 값은 해당 카테고리에 속하는 원소들을 모두 담은 리스트다.

```java
words.collect(groupingBy(word -> alphabetize(word)))
```

groupingBy가 반환하는 수집기가 리스트 외의 값을 갖는 맵을 생성하게 하려면, 분류 함수와 함께 다운스트림 수집기도 명시해야 한다. 다운스트림 수집기는 해당 카테고리의 모든 원소를 담은 스트림으로부터 값을 생성하는 역할을 한다. `toCollection(collectionFactory)`를 건네면 원하는 컬렉션 타입을 선택해 그 컬렉션을 값으로 갖는 맵을 생성할 수 있다. 다운스트림 수집기로 `counting()` 을 건네는 방법도 있는데, 각 카테고리를 해당 카테고리에 속하는 원소의 개수와 매핑한 맵을 얻을 수 있다.

```java
Map<String, Long> freq = words
  .collect(groupingBy(String::toLowerCase, counting()));
```

groupingBy는 다운스트림 수집기에 더해 맵 팩터리도 지정할 수 있게 해준다. 이 버전은 맵과 그 안에 담긴 컬렉션의 타입을 모두 지정할 수 있다. 예컨대 값이 TreeSet인 TreeMap을 반환하는 수집기를 만들 수 있다.

위 세 가지 groupingBy 각각에 대응하는 groupingByConcurrent 메서드들도 볼 수 있다. 이는 메서드의 동시 수행 버전으로, ConcurrentHashMap 인스턴스를 만들어준다.

### 그 외의 메서드

분류 함수 자리에 프레디키트를 받고 키가 Boolean 맵을 반환하는 partitioningBy도 있다. 프레디키트에 더해 다운스트림 수집기까지 입력받는 버전도 다중정의되어 있다. counting 메서드가 반환하는 수집기는 다움스트림 수집기 전용이다. 그 밖에도 summing, averaging, summarizing으로 시작하며, 각각 int, long, double 스트림용으로 하나씩 존재하는 메서드가 있다.  그리고 다중정의된 reducing 메서드들, filtering, mapping, flatMapping, collectingAndThen 메서드도 있다.

Collectors에 정의되어 있지만 '수집'과는 관련이 없는 메서드들도 존재한다. 그중 minBy와 maxBy는 인수로 받은 비교자를 이용해 스트림에서 값이 가장 작은 혹은 가장 큰 원소를 찾아 반환한다.

마지막으로 CharSequence 인스턴스의 스트림에만 적용할 수 있는 joining 메서드가 있다. 매개변수가 없는 joining은 단순히 원소들을 연결하는 수집기를 반환한다. CharSequence 타입의 구분문자(delimiter) 하나를 매개변수로 받는 joining은 연결 부위에 이 구분 문자를 삽입해준다. 인수 3개짜리 joining은 구분문자에 더해 접두문자(prefix)와 접미문자(suffix)도 받는다.

**가장 중요한 수집기 팩터리는 toList, toSet, toMap, groupingBy, joining이다.**



## Item 47. 반환 타입으로는 스트림보다 컬렉션이 낫다

자바 7까지는 원소 시퀀스(일련의 원소)를 반환하기 위해 메서드 반환 타입으로 Collection, Set, List 같은 컬렉션 인터페이스, 혹은 Iterable이나 배열을 썼다. 기본적으로는 컬렉션 인터페이스를 사용하고 for-each 문에서만 쓰이거나 반한된 원소 시퀀스가 일부 Collection 메서드를 구현할 수 없을 때는 Iterable 인터페이스를, 반환 원소들이 기본 타입이거나 성능에 민감한 상황에는 배열을 썼다.

자바 8부터 생긴 스트림은 반복을 지원하지 않는다. Stream 인터페이스는 Iterable 인터페이스가 정의한 추상 메서드를 전부 포함하지만 Stream이 Iterable을 확장하지 않았기 때문에 for-each를 사용할 수 없다.

```java
// 자바 타입 추론의 한계로 컴파일되지 않는다.
for (ProcessHandle ph : ProcessHandle.allProcesses()::iterator) { // 메서드 참조를 사용
  // 프로세스 처리
}
```

```java
// 메서드 참조를 매개변수화된 Iterable로 적절히 형변환해준다. - 작동은 하지만 난잡하고 직관성이 떨어진다.
for (ProcessHandle ph : (Iterable<ProcessHandle>)
     										ProcessHandle.allProcesses()::iterator) {
  // 프로세스 처리
}
```

어댑터 메서드를 사용하면 어떤 스트림도 for-each문으로 반복할 수 있다. 이 경우 자바의 타입 추론이 문맥을 잘 파악하여 따로 형변환을 할 필요도 없다.

```java
// Stream<E>를 Iterable<E>로 중개해주는 어댑터
public static <E> Iterable<E> iterableOf(Stream<E> stream) {
  return stream::iterator;
}

for (ProcessHandle ph : iterableOf(ProcessHandle.allProcesses())) {
  // 프로세스 처리
}
```

반대로 중개해주는 어댑터도 쉽게 구현할 수 있다.

```java
// Iterable<E>를 Stream<E>로 중개해주는 어댑터
public static <E> Stream<E> streamOf(Iterable<E> iterable) {
  return StreamSupport.stream(iterable.spliterator(), false);
}
```

Collection 인터페이스는 Iterable의 하위 타입이도 stream 메서드도 제공하니 반복과 스트림을 동시에 지원한다. 따라서 **원소 시퀀스를 반환하는 공개 API의 반환 타입에는 Collection이나 그 하위 타입을 쓰는 게 일반적으로 최선이다.** 반환하는 시쿼스의 크기가 메모리에 올려도 안전할 만큼 작다면 ArrayList나 HashSet 같은 표준 컬렉션 구현체를 반환하는게 최선일 수 있지만 단지 컬렉션을 반환한다는 이유로 덩치 큰 시퀀스를 메모리에 올려서는 안 된다. 반환할 시퀀스가 크지만 표현을 간결하게 할 수 있다면 전용 컬렉션을 구현하는 방안을 검토해보자. AbstractCollection을 이용하면 전용 컬렉션을 손쉽게 구현할 수 있다.

```java
// 입력 집합의 멱집합을 전용 컬렉션에 담아 반환
public class PowerSet {
  public static final <E> Collection<Set<E>> of(Set<E> s) {
    List<E> src = new ArrayList<>(s);
    if (src.size() > 30)
      throw new IllegalArgumentException("집합에 원소가 너무 많습니다(최대 30개).: " + s);
    return new AbstractList<Set<E>>() {
      @Override public int size() {
        // 멱집합의 크기는 2를 원래 집합의 원소 수 만큼 거듭제곱한 것과 같다.
        return 1 << src.size();
      }
      
      @Override public boolean contains(Object o) {
        return o instanceof Set && src.containsAll((Set)o);
      }
      
      @Override public Set<E> get(int index) {
        Set<E> result = new HashSet<>();
        for (int i = 0; index != 0; i++, index >>= 1)
          if ((index & 1) == 1)
            result.add(src.get(i));
        return result;
      }
    };
  }
}
```

AbstractCollection을 활용해서 Collection 구현체를 작성할 때는 Iterable용 메서드 외에 2개만 더 구현하면 된다. 바로 contains와 size다. contains와 size를 구현하는게 불가능할 때는 컬렉션보다는 스트림이나 Iterable을 반환하는 편이 낫다. 별도의 메서드를 두어 두 방식을 모두 제공해도 된다.

입력 리스트의 부분리스트를 모두 반환하는 메서드를 작성한다고 해보자. 필요한 부분리스트를 만들어 표준 컬렉션에 담는 코드는 간단하지만 이 컬렉션은 입력 리스트 크기의 거듭제곱만큼 메로리를 차지한다. 이는 스트림으로 어렵지 않게 구현할 수 있다. 첫 번째 원소를 포함하는 부분리스트를 그 리스트의 prefix, 마지막 원소를 포함하는 부분리스트를 suffix라 치면, 어떤 리스트의 부분리스트는 단순히 그 리스트의 프리픽스의 서픽스(혹은 서픽스의 프리픽스)에 빈 리스트 하나만 추가하면 된다.

```java
// 입력 리스트의 모든 부분리스트를 스트림으로 반환
public class SubLists {
  public static <E> Stream<List<E>> of(List<E> list) {
    return Stream.concat(Stream.of(Collections.emptyList()), // 반환되는 스트림에 빈 리스트 추가
        prefixes(list).flatMap(SubLists::suffixes)); // 모든 프리픽스의 모든 서픽스로 구성된 하나의 스트림을 만듦
  }
  
  private static <E> Stream<List<E>> prefixes(List<E> list) {
    return IntStream.rangeClosed(1, list.size())
      .mapToObj(end -> list.subList(0, end));
  }
  
  private static <E> Stream<List<E>> suffixes(List<E> list) {
    return IntStream.range(0, list.size())
      .mapToObj(start -> list.subList(start, list.size()));
  }
}
```

for 반복문을 중첩해 만들어 스트림으로 변환할 수 있지만 가독성이 안 좋아질 것이다. 이 방식의 취지는 데카르트 곱 코드와 비슷하다.

```java
// 입력 리스트의 모든 부분리스트를 스트림으로 반환
public static <E> Stream<List<E>> of(List<E> list) {
  return IntStream.range(0, list.size())
    .mapToObj(start ->
         IntStream.rangeClosed(start + 1, list.size())
             .mapToObj(end -> list.subList(start, end)))
    .flatMap(x -> x);
}
```

이 코드는 빈 리스트는 반환하지 않는다. 이 부분을 고치려면 concat을 사용하거나 rangeClosed 호출 코드의 1을 `(int)Math.signum(start)`로 고쳐주면 된다. 이렇게 스트림을 반환할 수가 있는데, 반복을 사용하는게 더 자연스러운 상황에서도 사용자는 스트림을 쓰거나 Stream을 Iterable로 변환해주는 어댑터를 이용해야 한다. 속도적인 면에서 스트림을 활용한 구현보다 Collection을 직접 구현해 사용하는 것이 좋다.



## Item 48. 스트림 병렬화는 주의해서 적용하라

자바로 동시성 프로그램을 작성하기가 점점 쉬워지고는 있지만, 이를 올바르고 빠르게 작성하는 일은 어렵다. 동시성 프로그래밍을 할 때는 안전성(safety)과 응답 가능(liveness) 상태를 유지하기 위해 애써야 하는데, 병렬 스트림 파이프라인 프로그래밍에서도 다를 바 없다.

```java
// 스트림을 사용해 처음 20개의 메르센 소수를 새애성하는 프로그램
public static void main(String[] agrs) {
  primes().map(p -> TWO.pow(p.intValueExact()).subtract(ONE))
  	.filter(mersenne -> mersenne.isProbablePrime(50))
    .limit(20)
    .forEach(System.out::println);
}

static Stream<BigInteger> primes() {
  return Stream.iterate(TWO, BigInteger::nextProbablePrime);
}
```

이 프로그램의 속도를 높이고 싶어 스트림 파이프라인의 `parallel()` 을 호출한다면, 스트림 라이브러리가 이 파이프라인을 병렬화하는 방법을 찾아내지 못해 아무것도 출력하지 못하면서 종료되지 않는 문제가 발생한다. 환경이 아무리 좋더라고 **데이터 소스가 Stream.iterate거나 중간 연산으로 limit을 쓰면 파이프라인 병렬화로는 성능 개선을 기대할 수 없다.** 따라서 스트림 파이프라인을 마구잡이로 병렬화하면 성능이 오히려 나빠질 수 있다.

대체로 **스트림의 소스가 ArrayList, HashMap, HashSet, ConcurrentHashMap의 인스턴스거나 배열, int 범위, long 범위일 때 병렬화의 효과가 가장 좋다.** 이 자료구조들은 데이터를 원하는 크기로 정확하고 손쉽게 나눌 수 있어서 일을 다수의 스레드에 분배하기에 좋다. 나누는 작업은 Spliterator가 담당하며, Spliterator 객체는 Stream이나 Iterable의 spliterator 메서드로 얻어올 수 있다. 이 자료구조들은 원소들을 순차적으로 실행할 때의 참조 지역성이 뛰어나다(이웃한 원소의 참조들이 메모리에 연속해서 저장되어 있다). 참조 지역성이 낮으면 스레드는 데이터가 주 메모리에서 캐시 메모리로 전송되어 오기까지 공백기를 가지기 때문에 다량의 데이터를 처리하는 벌크 연산을 병렬화할 때 참조 지역성은 중요한 요소로 작용한다. 참조 지역성이 가장 뛰어난 자료구조는 기본 타입의 배열이다.

스트림 파이프라인의 종단 연산의 동작 방식도 병렬 수행 효율에 영향을 주는데, 종단 연산 중 병렬화에 가장 적합한 것은 축소이다. 축소는 파이프라인에서 만들어진 모든 원소를 하나로 합치는 작업으로, Stream의 recude 메서드 중 하나, 혹은 min, max, count, sum 같이 완성된 형태로 제공되는 메서드 중 하나를 선택해 수행한다. anyMatch, allMatch, noneMatch처럼 조건에 맞으면 바로 반환되는 메서드도 병렬화에 적합하다. 반면, 가변 축소를 수행하는 Stream의 collect 메서드는 병렬화에 적합하지 않다(컬렉션을 합치는 비용).

직접 구현한 Stream. Iterable, Collection이 병렬화의 이점을 제대로 누리게 하고 싶다면 spliterator 메서드를 반드시 재정의하고 결과 스트림의 병렬화 성능을 테스트해야 한다. 스트림을 잘못 병렬화하면 (응답 불가를 포함해) 성능이 나빠질 뿐만 아니라 결과 자체가 잘못되거나 예상 못한 동작이 발생할 수 있다. 결과가 잘못되거나 오동작하는 것은 안전 실패라 한다. 안전 실패는 병렬화한 파이프라인이 사용하는 mappers, filters, 혹은 프로그래머가 제공한 다른 함수 객체가 명세대로 동작하지 않을 때 벌어질 수 있다. Stream 명세는 이때 사용되는 함수 객체에 관한 규약을 정의해뒀는데, 이를 지키지 못하는 상태라도 파이프라인을 순차적으로 수행한다면 올바른 결과를 얻을 수도 있다. 하지만 병렬로 수행하면 실패로 이어지기 쉽다. 앞의 병렬화한 메르센 소수 프로그램은 출력 순서가 올바르지 않을 수 있는데, forEach를 forEachOrdered로 바꿔주면 된다.

스트림 병렬화는 성능 최적화 수단이다. 다른 최적화와 마찬가지로 변경 전후로 반드시 성능 테스트를하여 병렬화를 사용할 가치가 있는지 확인해야 한다. 이상적으로는 운영 시스템과 흡사한 환경에서 테스트하는 것이 좋다. **조건이 잘 갖춰지면 parallel 메서드 호출 하나로 거의 프로세서 코어 수에 비례하는 성능 향상을 만끽할 수 있다.**

```java
// n 보다 작거나 같은 소수의 개수를 계산하는 스트림 파이프라인 - 병렬화에 적합
static long pi(long n) {
  return LongStream.rangeClosed(2, n)
    .mapToObj(BigInteger::valueOf)
    .filter(i -> i.isProbablePrime(50))
    .count();
} // ex. 31초
```

```java
// n 보다 작거나 같은 소수의 개수를 계산하는 스트림 파이프라인 - 병렬화 버전
static long pi(long n) {
  return LongStream.rangeClosed(2, n)
    .parallel() // 추가
    .mapToObj(BigInteger::valueOf)
    .filter(i -> i.isProbablePrime(50))
    .count();
} // ex. 9.2초
```

병렬화를 해줌으로써 계산시간이 많이 단축되는 것을 알 수 있다. 하지만 n이 크다면 위 계산 대신 레머의 공식이라는 알고리즘을 사용하는 것이 좋다.

무작위 수들로 이뤄진 스트림 병렬화 시에는 ThreadLocalRandom(혹은 Random) 보다는 SplittableRandom 인스턴스를 이용하자. SplittableRandom을 사용해 병렬화하면 성능이 선형으로 증가한다. ThreadLocalRandom은 단일 스레드에서 쓰고자 만들어졌기 때문에 병렬 스트림용 데이터 소스로 사용 가능하지만 그만큼 성능이 좋지 않다. Random은 모든 연산을 동기화하기 때문에 병렬 처리 시 성능이 매우 좋지 않다.
