---
title: EffectiveJava_Chap6
excerpt: 이펙티브 자바 책 6장 정리
categories: study
---

# 6장. 열거 타입과 애너테이션

## Item 34. int 상수 대신 열거 타입을 사용하라

열거 타입은 일정 개수의 상수 값을 정의한 다음, 그 외의 값은 허용하지 않는 타입이다. 자바에서 열거 타입을 지원하기 전에는 정수 상수를 한 묶음 선언해서 사용하곤 했다. 정수 열거 패턴(int enum pattern) 기법에는 단점이 많다. 타입 안전을 보장할 방법이 없으며 표현력도 좋지 않다. 정수 열거 패턴은 평범한 상수를 나열한 것뿐이라 이를 사용한 프로그램은 깨지기 쉽다. 또한 문자열로 출력하기도 다소 까다롭다. 정수 대신 문자열 상수를 사용하는 문자열 열거 패턴(string enum pattern)도 있는데, 이 변형은 더 나쁘다. 상수의 의미를 출력할 수 있다는 점은 좋지만, 문자열 상수의 이름 대신 문자열 값을 그대로 하드코딩 해야하기 때문이다.

### 열거 타입

```java
// 가장 단순한 열거 타입
public enum Apple { FUJI PIPPIN, GRANNY_SMITH }
public enum Orange { NAVEL, TEMPLE, BLOOD }
```

자바의 열거 타입은 완전한 형태의 클래스라서 다른 언어의 열거 타입보다 훨씬 강력하다. 열거 타입 자체는 클래스이며, 상수 하나당 자신의 인스턴스를 하나씩 만들어 public static final 필드로 공개한다. 클라이언트가 인스턴스를 직접 생성하거나 확장할 수 없어 열거 타입 선언으로 만들어진 인스턴스들은 딱 하나씩만 존재함이 보장된다.

### 열거 타입의 장점

- 컴파일타임 타입 안전성을 제공한다.
- 각자의 이름공간이 있어서 이름이 같은 상수도 평화롭게 공존한다.
- 열거 타입의 toString 메서드는 출력하기에 적합한 문자열을 내어준다.
- 임의의 메서드나 필드를 추가할 수 있고 임의의 인터페이스를 구현하게 할 수도 있다.
  - Object 메서드들을 높은 품질로 구현해 둠
  - Comparable과 Serializable을 구현했으며, 그 직렬화 형태도 웬만큼 변형을 가해도 문제없이 동작하게끔 구현해 둠

### 열거 타입 사용 예시

```java
// 데이터와 메서드를 갖는 열거 타입
public enum Planet {
  MERCURY(3.302e+23, 2.439e6),
  VENUS	 (4.869e+24, 6.052e6),
  EARTH	 (5.975e+24, 6.378e6),
  MARS	 (6.419e+23, 3.393e6),
  JUPITER(1.899e+27, 7.149e7),
  SATURN (5.685e+26, 6.027e7),
  URANUS (8.683e+25, 2.556e7),
  NEPTUNE(1.024e+26, 2.477e7);
  
  private final double mass;						// 질량(단위: 킬로그램)
  private final double radius;					// 반지름(단위: 미터)
  private final double surfaceGravity;	// 표면중력(단위: m / s^2)
  
  // 중력상수(단위: m^3 / kg s^2)
  private static final double G = 6.67300E-11;
  
  // 생성자
  Planet(double mass, double radius) {
    this.mass = mass;
    this.radius = radius;
    surfaceGravity = G * mass / (radius * radius);
  }
  
  public double mass()						{ return mass; }
 	public double radius()					{ return radius; }
  public double surfaceGravity()	{ return surfaceGravity; }
  
  public double surfaceWeight(double mass) {
    return mass * surfaceGravity;	// F = ma
  }
}
```

열거 타입 상수 각각을 특정 데이터와 연결지으려면 생성자에서 데이터를 받아 인스턴스 필드에 저장하면 된다. 열거 타입은 근본적으로 불변이라 모든 필드는 final이어야 한다. 필드를 public으로 선언해도 되지만, private으로 두고 별도의 public 접근자 메서드를 두는 게 낫다.

```java
public class WeightTable {
  public static void main(String[] args) {
    double earthWeight = Double.parseDouble(args[0]);
    double mass = earthWeight / Planet.EARTH.surfaceGravity();
    for (Planet p : Planet.values()) // values: 열거 타입 안에 정의된 상수들의 값을 배열에 담아 반환하는 정적 메서드
      System.out.printf("%s에서의 무게는 %f이다.%n", p, p.surfaceWeight(mass));
  }
}
```

열거 타입도 일반 클래스와 마찬가지로, 그 기능을 클라이언트에 노출해야 할 합당한 이유가 없다면 private으로, 혹은 (필요하다면) package-private으로 선언해야 한다.

상수마다 동작이 달라져야 하는 상황에서도 열거 타입을 활용할 수 있다.

```java
// 값에 따라 분기하는 열거 타입 - switch 문 사용
public enum Operation {
  PLUS, MINUS, TIMES, DIVIDE;
  
  // 상수가 뜻하는 연산을 수행한다.
  public double apply(double x, double y) {
    switch(this) {
      case PLUS:		return x + y;
      case MINUS: 	return x - y;
      case TIMES:		return x * y;
      case DIVIDE:	return x / y;
    }
    throw new AssertionError("알 수 없는 연산: " + this); // 실제로 도달할 일이 없는 부분 (하지만 생략시 컴파일 에러)
  }
}
```

위 방법은 새로운 상수를 추가하면 해당 case 문도 추가해야한다. 그렇지 않으면 "알 수 없는 연산"이라는 런타임 오류가 발생한다. 열거 타입에 추상 메서드를 선언하고 각 상수에서 자신에 맞게 재정의하는 방법이 있다. 이를 상수별 메서드 구현(constant-specific method implementation)이라 한다.

```java
// 상수별 메서드 구현을 활용한 열거 타입
public enum Operation {
  PLUS		{public double apply(double x, double y){return x + y;}},
  MINUS		{public double apply(double x, double y){return x - y;}},
  TIMES		{public double apply(double x, double y){return x * y;}},
  DIVIDE	{public double apply(double x, double y){return x / y;}};
  
  public abstract double apply(double x, double y); // apply가 추상 메서드이므로 재정의 하지 않으면 컴파일 오류 발생
}
```

상수별 메서드 구현을 상수별 데이터와 결합할 수도 있다.

```java
// 상수별 클래스 몸체(class body)와 데이터를 사용한 열거 타입
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

다음과 같이 간편하게 계산식 출력이 가능하다.

```java
public static void main(String[] args) {
  double x = Double.parseDouble(args[0]);
  double y = Double.parseDouble(args[1]);
  for (Operation op : Operation.values())
    System.out.printf("%f %s %f = %f%n", x, op, y, op.apply(x, y));
  // 결과
  // 2.000000 + 4.000000 = 6.000000
  // 2.000000 - 4.000000 = -2.000000
  // 2.000000 * 4.000000 = 8.000000
  // 2.000000 / 4.000000 = 0.500000
}
```

열거 타입의 toString 메서드를 재정의하려거든 toString이 반환하는 문자열을 해당 열거 타입 상수로 변환해주는 fromString 메서드도 함께 제공하는 걸 고려해보자.

```java
// 열거 타입용 fromString 메서드 구현
private static final Map<String, Operation> stringToEnum = // 정적 필드가 초기화될 때 맵에 Operation 상수가 추가됨
						  Stream.of(values()).collect(toMap(Object::toString, e -> e));

// 지정한 문자열에 해당하는 Operation을 (존재한다면) 반환한다.
public static Optional<Operation> fromString(String symbol) { // 주어진 문자열이 가리키는 연산이 존재하지 않을 수 있음
  return Optional.ofNullable(stringToEnum.get(symbol));
}
```

상수별 메서드 구현에는 열거 타입 상수끼리 코드를 공유하기 어렵다는 단점이 있다.

```java
// 값에 따라 분기하여 코드를 공유하는 열거 타입
enum PayrollDay {
  MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY;
  
  private static final int MINS_PER_SHIFT = 8 * 60;
  
  int pay(int minutesWorked, int payRate) {
    int basePay = minutesWorked * payRate;
    
    int overtimePay;
    switch(this) {
      case SATURDAY: case SUNDAY: // 주말
        overtimePay = basePay / 2;
        break;
      default: // 주중
        overtimePay = minutesWorked <= MINS_PER_SHIFT ? 0 : (minutesWorked - MIN_PER_SHIFT) * payRate / 2;
    }
    return basePay + overtimePay;
  }
} // 간결하지만, 관리 관점에서 위험한 코드. ex.휴가 상황
```

상수별 메서드 구현으로 급여를 정확히 계산하는 방법은 두가지다.

1. 잔업수당을 계산하는 코드를 모든 상수에 중복해서  넣는다.
2. 계산 코드를 평일용과 주말용으로 나눠 각각을 도우미 메서드로 작성한 다음 각 상수가 자신에게 필요한 메서드를 적절히 호출한다.

두 방식 모두 코드가 가독성이 크게 떨어지고 오류 발생 가능성이 높아진다.

가장 깔끔한 방법은 새로운 상수를 추가할 때 잔업수당 '전략'을 선택하도록 하는 것이다.

```java
// 전략 열거 타입 패턴
enum PayrollDay {
  MONDAY(WEEKDAY), TUESDAY(WEEKDAY), WEDNESDAY(WEEKDAY), THURDAY(WEEKDAY), 
  FRIDAY(WEEKDAY), SATURDAY(WEEKEND), SUNDAY(WEEKEND);
  
  private final PayType payType; // 잔업수당 계산을 private 중첩 열거 타입으로 옮김.
  
  PayrollDay(PayType payType) { this.payType = payType; } // 생산자에서 적당한 것을 선택
  
  int pay(int minutesWorked, int payRate) {
    return payType.pay(minutesWorked, payRate);
  }
  
  // 전략 열거 타입
  enum PayType {
    WEEKDAY {
      int overtimePay(int minsWorked, int payRate) {
        return minsWorked <= MINS_PER_SHIFT ? 0 : (minsWorked - MINS_PER_SHIFT) * payRate / 2;
      }
    },
    WEEKEND {
      int overtimePay(int minsWorked, int payRate) {
        return minsWorked * payRate / 2;
      }
    };
    
    abstract int overtimePay(int mins, int payRate);
    private static final int MINS_PER_SHIFT = 8 * 60;
    
    int pay(int minsWorked, int payRate) {
      int basePay = minsWorked * payRate;
      return basePay + overtimePay(minsWorked, payRate);
    }
  }
}
```

이 패턴은 switch 문보다 복잡하지만 더 안전하고 유연하다. 하지만 기존 열거 타입에 상수별 동작을 혼합해 넣을 때는 switch 문이 좋은 선택이 될 수 있다.

```java
// switch 문을 이용해 원래 열거 타입에 없는 기능을 수행한다.
public static Operation inverse(Operation op) {
  switch(op) {
    case PLUS:		return Operation.MINUS;
    case MINUS:		return Operation.PLUS;
    case TIMES:		return Operation.DIVIDE;
    case DIVIDE:	return Operation.TIMES;
      
    default:	throw new AssertionError("알 수 없는 연산: " + op);
  }
}
```

추가하려는 메서드가 의미상 열거 타입에 속하지 않거나, 종종 쓰이지만 열거 타입안에 포함할 만큼 유용하지는 않은 경우에는 이 방식을 적용하는게 좋다.

필요한 원소를 컴파일타임에 다 알 수 있는 상수 집합이라면 항상 열거 타입을 사용하자. 열거 타입에 정의된 상수 개수가 영원히 고정 불변일 필요는 없다.



## Item 35. ordinal 메서드 대신 인스턴스 필드를 사용하라

대부분의 열거 타입 상수는 자연스럽게 하나의 정수값에 대응된다. 모든 열거 타입은 해당 상수가 그 열거 타입에서 몇 번째 위치인지를 반환하는 ordinal이라는 메서드를 제공한다.

```java
// 합주단의 종류를 연주자가 1명인 솔로(solo)부터 10명인 디텍트(detect)까지 정의한 열거 타입 
// ordinal을 잘못 사용한 예 - 따라 하지 말 것! 유지보수가 어렵다.
public enum Ensemble {
  SOLE, DUET, TRIO, QUARTET, QUINTET, SEXTET, SEPTET, OCTET, NONET, DECTET;
  
  public int numberOfMusicians() { return ordinal() + 1; }
}
```

이 코드는 상수 선언 순서를 바꾸는 순간 오동작 하며, 이미 사용 중인 정수와 값이 같은 상수는 추가할 방법이 없고, 값을 중간에 비워둘 수도 없다. 따라서 열거 타입 상수에 연결된 값은 ordinal 메서드로 얻지 말고 인스턴스 필드에 저장하자.

```java
public enum Ensemble {
	SOLE(1), DUET(2), TRIO(3), QUARTET(4), QUINTET(5), SEXTET(6), SEPTET(7), 
  OCTET(8), DOUBLE_QUARTET(8), NONET(9), DECTET(10), TRIPLE_QUARTET(12);
  
  private final int numberOfMusicians;
  Ensemble(int size) { this.numberOfMusicians = size; }
  public int numberOfMusicians() { return numberOfMusicians; }
}
```

EnumSet과 EnumMap 같이 열거 타입 기반의 범용 자료구조에 쓸 용도가 아니라면 ordinal 메서드는 절대 사용하지 말자.



## Item 36. 비트 필드 대신 EnumSet을 사용하라

열거한 값들이 주로 집합으로 사용될 경우, 예전에는 각 상수에 서로 다른 2의 거듭제곱 값을 할당한 정수 열거 패턴을 사용해왔다.

```java
// 비트 필드 열거 상수 - 구닥다리 기법!
public class Text {
  public static final int STYLE_BOLD					= 1 << 0; // 1
  public static final int STYLE_ITALIC				= 1 << 1; // 2
  public static final int STYLE_UNDERLINE			= 1 << 2; // 4
  public static final int STYLE_STRIKETHROUGH	= 1 << 3; // 8
  
  // 매개변수 styles는 0개 이상의 STYLE_ 상수를 비트별 OR한 값 (비트 필드)
  public void applyStyles(int styles) { ... } 
}
```

```java
text.applyStyles(STYLE_BOLD | STYLE_ITALIC);
```

### 비트 필드의 단점

- 정수 열거 상수의 단점을 그대로 지님
- 비트 필드 값이 그대로 출력되면 단순한 정수 열거 상수를 출력할 때보다 해석이 어려움
- 비트 필드 하나에 녹아 있는 모든 원소를 순회하기가 까다로움
- 최대 몇 비트가 필요한지를 미리 예측하여 적절한 타입(보통 int나 long)을 선택해야 함

### java.util.EnumSet

EnumSet 클래스는 열거 타입 상수의 값으로 구성된 집합을 효과적으로 표현해준다. Set 인터페이스를 완벽히 구현하며, 타입 안전하고, 다른 어떤 Set 구현체와도 함께 사용할 수 있다. EnumSet의 내부는 비트 벡터로 구현되어 대부분의 경우 EnumSet 전체를 long 변수 하나로 표현하여 비트 필드에 비견되는 성능을 보여주고, 대량 작업도 비트를 효율적으로 처리할 수 있는 산술 연산을 써서 구현했다.

```java
// EnumSet - 비트 필드를 대체하는 현대적 기법
public class Text {
  public enum Style { BOLD, ITALIC, UNDERLINE, STRIKETHROUGH }
  
  // 어떤 Set을 넘겨도 되나, EnumSet이 가장 좋다.
  public void applyStyles(Set<Style> styles) { ... }
}
```

EnumSet은 집합 생성 등 다양한 기능의 정적 팩터리를 제공한다.

```java
text.applyStyles(EnumSet.of(Style.BOLD, Style.ITALIC));
```

applyStyles 메서드가  EnumSet\<Style>이 아닌 Set\<Style>을 받은 이유는 , 모든 클라이언트가  EnumSet을 건네리라 짐작되는 상황이라도 인터페이스로 받는 것이 좋다. 이렇게 하면 다른 Set 구현체를 넘기더라도 처리할 수 있다.

EnumSet의 유일한 단점이라면(자바 11까지도) 불변 EnumSet을 만들 수 없다는 것이다. 이러한 단점을 극복하기위해서 (명확성과 성능이 조금 희생되지만) `Collections.unmodifiableSet`으로  EnumSet을 감싸 사용할 수 있다.



## Item 37. ordinal 인덱싱 대신 EnumMap을 사용하라

배열이나 리스트에서 원소를 꺼낼 때 ordinal 메서드로 인덱스를 얻을 수도 있다. (나쁜 방법)

```java
class Plant {
  enum LifeCycle { ANNUAL, PERENNIAL, BIENNIAL }
  
  final String name;
  final LifeCycle lifeCycle;
  
  Plant(String name, LifeCycle lifeCycle) {
    this.name = name;
    this.lifeCycle = lifeCycle;
  }
  
  @Override public String toString() {
    return name;
  }
}
```

식물들을 배열 하나로 관리하고, 이들을 생애주기별(한해살이, 여러해살이, 두해살이)로 묶어보자.

```java
// ordinal()을 배열 인덱스로 사용 - 따라 하지 말 것!
Set<Plant> [] plantByLifeCycle = 
  (Set<Plant>[]) new Set[Plant.LifeCycle.values().length]; // 배열은 제네릭과 호환되지 않음(비검사 형변환 수행)
for (int i = 0; i < plantsByLifeCycle.length; i++)
  plantsByLifeCycle[i] = new HashSet<>();

for (Plant p : garden)
  plantsByLifeCycle[p.lifeCycle.ordinal()].add(p); // 정수는 타입 안전하지 않아 잘못도니 결과를 초래할 수 있다.

// 결과 출력
for (int i = 0; i < plantsByLifeCycle.length; i++) {
  // 배열은 각 인덱스의 의미를 모르니 출력 결과에 직접 레이블을 달아야 한다.
  System.out.printf("%s: %s%n", Plant.LifeCycle.values()[i], plantsByLifeCycle[i]);
}
```

여기서 배열은 실질적으로 열거 타입 상수를 값으로 매핑하는 일을 하기 때문에 Map을 사용할 수도 있다. EnumMap은 열거 타입을 키로 사용하도록 설계한 아주 빠른 Map 구현체이다.

```java
// EnumMap을 사용해 데이터와 열거 타입 매핑
Map<Plant.LifeCycle, Set<Plant>> plantsByLifeCycle = 
  new EnumMap<>(Plant.LifeCycle.class); // 키 타입의 Class 객체는 한정적 타입 토큰, 런타임 제네릭 타입 정보 제공
for (Plant.LifeCycle lc : Plant.LifeCycle.values())
  plantsByLifeCycle.put(lc, new HashSet<>());
for (Plant p : garden)
  plantByLifeCycle.get(p.lifeCycle.add(p));
System.out.println(plantsByLifeCycle);
```

더 짧고 명료하고 안전하면서 성능도 기존 버전과 비등하다.

- 안전하지 않은 형변환은 쓰지 않는다.
- 맵의 키인 열거 타입이 그 자체로 출력용 문자열을 제공한다.
- 배열 인덱스를 계산하는 과정에서 오류가 날 가능성이 없다.

스트림을 사용해 맵을 관리하면 코드를 더 줄일 수 있다. 다음은 가장 단순한 형태의 스트림 기반 코드다.

```java
// 스트림을 사용한 코드 1 - EnumMap을 사용하지 않는다.
System.out.println(Arrays.stream(garden)
                   .collect(groupingBy(p -> p.lifeCycle)));
```

이 코드는 EnumMap이 아닌 고유한 맵 구현체를 사용했기 때문에  EnumMap을 써서 얻은 공간과 성능 이점이 사라진다는 문제가 있다.

매개변수 3개짜리 Collectors.groupingBy 메서드는 mapFactory 매개변수에 원하는 맵 구현체를 명시해 호출할 수 있다.

```java
// 스트림을 사용한 코드 2 - EnumMap을 이용해 데이터와 열거 타입을 매핑했다.
System.out.println(Arrays.stream(garden)
                   .collect(groupingBy(p -> p.lifeCycle,
                   			() -> new EnumMap<>(LifeCycle.class), toSet())));
```

> ```java
> static <T,K,D,A,M extends Map<K,D>> Collector<T,?,M> groupingBy(
>   Function<? super T,? extends K> classifier, 
>   Supplier<M> mapFactory, 
>   Collector<? super T,A,D> downstream)
> ```
>
> - classifiler: groupingBy를 위한 기준 값을 가져오는 function (무엇으로 묶을 것인가, K)
> - mapFactory: 함수의 수행 결과로서 새롭게 만들어지는 Map을 생성하는 function (결과로 나오는 Map은 무엇인가, Map(K, V))
> - downStream: groupingBy의 결과로서 얻어지는 결과 Collector (결과에 해당하는 값은 무엇인가, V)

EnumMap 버전은 언제나 식물의 생애주기당 하나씩의 중첩 맵을 만들지만, 스트림 버전에서는 해당 생애주기에 속하는 식물이 있을 때만 만드는 차이가 있다.

두 열거 타입 값들을 매핑하느라  ordinal을 두번이나 쓴 배열들의 배열을 볼 수 있다. (나쁜 방법)

```java
// 두 가지 상태(Phase)를 전이(Transition)와 매핑한 예시
// 배열들의 배열의 인덱스에 ordinal()을 사용 - 따라 하지 말 것!
public enum Phase {
  SOLID, LIQUID, GAS;
  
  public enum Transition {
    MELT, FREEZE, BOIL, CONDENSE, SUBLIME, DEPOSIT;
    
    // 행은 from의 ordinal을, 열은 to의 ordinal을 인덱스로 쓴다.
    private static final Transition[][] TRANSITIONS = {
      { null, MELT, SUBLIME },
      { FREEZE, null, BOIL },
      { DEPOSIT, CONDENSE, null }
    };
    
    // 한 상태에서 다른 상태로의 전이를 반환한다.
    public static Transition from(Phase from, Phase to) {
      return TRANSITIONS[from.ordinal()][to.ordinal()];
    }
  }
}
```

컴파일러는  ordinal과 배열 인덱스의 관계를 알 수 없다. Phase나 Phase.Transition 열거 타입을 수정하면서 TRANSITIONS를 함께 수정하지 않거나 잘못 수정하면 러나임 오류가 날 것이다. 그리고  TRANSITIONS의 크기는 상태의 가지수가 늘어나면 제곱해서 커지며  null로 채워지는 칸도 늘어날 것이다. EnumMap 2개를 중첩해서 구현하는 것이 훨씬 낫다(안쪽 맵은 이후 상태와 전이를 연결, 바깥 맵은 이전 상태와 안쪽 맵을 연결).

```java
// 중첩 EnumMap으로 데이터와 열거 타입 쌍을 연결
public enum Phase {
  SOLID, LIQUID, GAS;
  
  public enum Transition {
    MELT(SOLID, LIQUID), FREEZE(LIQUID, SOLID),
    BOIL(LIQUID, GAS), CONDENSE(GAS, LIQUID),
    SUBLIME(SOLID, GAS), DEPOSIT(GAS, SOLID);
    
    private final Phase from;
    private final Phase to;
    
    Transition(Phase from, Phase to) {
      this.from = from;
      this.to = to;
    }
    
    // 상전이 맵 초기화
  	// "이전 상태에서 '이후 상태에서 전이로의 맵'에 대응시키는 맵"
  	private static final Map<Phase, Map<Phase, Transition>> m = 
    	Stream.of(values()).collect(groupingBy(t -> t.from, // 첫 번째 수집기. 전이를 이전 상태를 기준으로 묶음
           () -> new EnumMap<>(Phase.class),
           toMap(t -> t.to, t -> t, // 두 번째 수집기. 이후 상태를 전이에 대응시키는 EnumMap 생성
                 (x, y) -> y,() -> new EnumMap<>(Phase.class))));
    // (x, y) -> y는 두 번째 수집기의 병합 함수. 선언만 하고 실제로 쓰이지 않음
		// 이는 단지 EnumMap을 얻기 위해 맵 팩터리가 필요하고 수집기들은 점층적 팩터리를 제공하기 때문
  
  	public static Transition from(Phase from, Phase to) {
    	return m.get(from).get(to);
  	}
  }
}
```

> ```java
> Collector<T, ?, M> toMap(Function<? super T, ? extends K> keyMapper,
>                          Function<? super T, ? extends U> valueMapper,
>                          BinaryOperator<U> mergeFunction, // 중복일 때 병합을 어떻게 할 것인가(충돌 처리 방법)
>                          Supplier<M> mapSupplier) // 어떤 맵을 사용할 것인가(매개변수 안 쓰면 기본은 HashMap)
> ```
>
>

여기에 새로운 상태인 플라스마(PLASMA)와 연결된 전이 2개를 추가해보자. 앞의 예제의 배열로 만든 코드라면 상수를 추가할 뿐만 아니라 원소 9개짜리 배열을 원소 16개짜리로 교체해야 한다. 반면, EnumMap 버전은 다음과 같이 상태 목록과, 전이 목록만 추가해 주면 된다.

```java
// EnumMap 버전에 새로운 상태 추가
public enum Phase {
  SOLID, LIQUID, GAS, PLASMA;
  
  public enum Transition {
    MELT(SOLID, LIQUID), FREEZE(LIQUID, SOLID),
    BOIL(LIQUID, GAS), CONDENSE(GAS, LIQUID),
    SUBLIME(SOLID, GAS), DEPOSIT(GAS, SOLID),
    IONIZE(GAS, PLASMA), DEIONIZE(PLASMA, GAS);
    
    ... // 나머지 코드는 그대로
  }
}
```



## Item 38. 확장할 수 있는 열거 타입이 필요하면 인터페이스를 사용하라

열거 타입은 거의 모든 상황에서 타입 안전 열거 패턴보다 우수하다. 하지만 타입 안전 열거 패턴은 확장할 수 있으나 열거 타입은 그럴 수 없다. 사실 대부분 상황에서 열거 타입을 확장하는건 좋지 않은 생각이다. 그런데 연산 코드 같은 경우는 확장할 수 있는 열거 타입이 유용하다. 열거 타입이 임의의 인터페이스를 구현할 수 있다는 사실을 이용해, 연산 코드용 인터페이스를 정의하고 열거 타입이 이 인터페이스를 구현하게 하는 방법이 있다.

```java
// 인터페이스를 이용해 확장 가능 열거 타입을 흉내냄
public interface Operation {
  double apply(double x, double y);
}

public enum BasicOperation implements Operation {
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
  
  BasicOperation(String symbol) { this.symbol = symbol; }
  
  @Override public String toString() { return symbol; }
}
```

열거 타입인 BasicOperation은 확장할 수 없지만 인터페이스인  Operation은 확장할 수 있고, 이 인터페이스를 연산의 타입으로 사용하면 된다. Operation을 구현한 또 다른 열거 타입을 정의해 기본 타입인  BasicOperation을 대체할 수 있다.

```java
// 확장 기능 열거 타입 - 지수 연산과 나머지 연산 추가
public enum ExtendedOperation implements Operation {
  EXP("^") {
    public double apply(double x, double y)	{ return Math.pow(x, y); }
  },
  REMAINDER("%") {
    public double apply(double x, double y)	{ return x % y; }
  };
  
  ExtendedOperation(String symbol) { this.symbol = symbol; }
  
  @Override public String toString() { return symbol; }
}
```

새로 작성한 연산은 Operation 인터페이스를 사용하도록 작성되어 있기만 하면 기존 연산을 쓰던 곳 어디든 쓸 수 있다. apply가 인터페이스에 선언되어 있으니 열거 타입에 따로 추상 메서드로 선언하지 않아도  된다.

```java
public static void main(String[] args) {
  double x = Double.parseDouble(args[0]);
  double y = Double.parseDouble(args[1]);
  test(ExtendedOperation.class, x, y);
}

// 클래스 객체가 열거 타입(원소 순회를 위해)인 동시에 Operation의 하위 타입(원소가 뜻하는 연산을 수행하기 위해)이어야 한다.
private static <T extends Enum<T> & Operation> void test(Class<T> opEnumType, double x, double y) {
  for (Operation op : opEnumType.getEnumConstants())
    System.out.printf("%f %s %f = %f%n", x, op, y, op.apply(x, y));
}
```

Class 객체 대신 한정적 와일드카드 타입인 `Collection<? extends Operation>`을 넘기는 방법도 있다.

```java
public static void main(String[] args) {
  double x = Double.parseDouble(args[0]);
  double y = Double.parseDouble(args[1]);
  test(Arrays.asList(ExtendedOperation.values()), x, y);
}

// 클래스 객체가 열거 타입(원소 순회를 위해)인 동시에 Operation의 하위 타입(원소가 뜻하는 연산을 수행하기 위해)이어야 한다.
private static void test(Collection<? extends Operation> opSet, double x, double y) {
  for (Operation op : opSet)
    System.out.printf("%f %s %f = %f%n", x, op, y, op.apply(x, y));
}
```

이 방식은 덜 복잡하고 test 메서드가 조금 더 유연해지는 반면, 특정 연산에서는 EnumSet과 EnumMap을 사용하지 못한다.

인터페이스를 이용해 확장 가능한 열거 타입을 흉내 내는 방식은 열거 타입끼리 구현을 상속할 수 없다는 문제가 있다. 아무 상태에도 의존하지 않는 경우에는 디폴트 구현을 이용해 인터페이스에 추가하는 방법이 있는 반면,  위 예시에서는 연상 기호를 저장하고 찾는 로직이 구현체 모두에 들어가야만 한다. 공유하는 기능이 많다면 그 부분을 별도의 도우미 클래스나 정적 동우미 메서드로 분리하는 방식으로 코드 중복을 없앨 수 있다.



## Item 39. 명명 패턴보다 애너테이션을 사용하라

전통적으로 도구나 프레임 워크가 특별히 다뤄야 할 프로그램 요소에는 딱 구분되는 명명 패턴을 적용해왔다.

### 명명 패턴의 단점

- 오타가 나면 안 된다.
- 올바른 프로그램 요소에서만 사용되리라 보증할 방법이 없다.
- 프로그램 요소를 매개변수로 전달할 마땅한 방법이 없다.

### 애너테이션

애너테이션은 이 모든 문제를 해결해준다. Test라는 이름의 애너테이션을 정의한다고 해보자(자동으로 수행되는 간단한 테스트용 애너테이션, 예외 발생 시 테스트 실패 처리).

```java
// 마커(marker) 애너테이션 타입 선언
import java.lang.annotation.*;

/**
 * 테스트 메서드임을 선언하는 애너테이션이다.
 * 매개변수 없는 정적 메서드 전용이다. - (이 제약을 컴파일러가 강제하려면 적절한 애너테이션 처리기를 직접 구현해야 함)
 */
@Retention(RetentionPolicy.RUNTIME) // @Test가 런타임에도 유지되어야 한다는 표시. 생략 시 테스트 도구는 @Test 인식 불가!
@Target(ElementType.METHOD) // @Test가 반드시 메서드 선언에서만 사용되어야 한다는 표시. 클래스 선언, 필드 선언 등의 요소 x
public @interface Test {
}
```

`@Rectention`과 `@Target`처럼 애너테이션 선언에 다는 애너테이션을 메타애너테이션이라한다. `@Test`와 같은 애너테이션을  "아무 매개변수 없이 단순히 대상에 마킹한다"는 뜻에서 마커 애너테이션이라 한다. 이 애너테이션을 사용하면 프로그래머가  Test 이름에 오타를 내거나 메서드 선언 외의 프로그램 요소에 달면 컴파일 오류를 내준다.

```java
// 마커 애너테이션을 사용한 프로그램 예
public class Sample {
  @Test public static void m1() { } // 성공해야 한다.
  public static void m2() { }				// 테스트 도구가 무시한다.
  @Test public static void m3() {		// 실패해야 한다.
    throw new RuntimeException("실패");
  }
  public static void m4() { }				// 테스트 도구가 무시한다.
  @Test public void m5() { } 				// 잘못 사용한 예: 정적 메서드가 아니다.
  public static void m6() { }				// 테스트 도구가 무시한다.
  @Test public static void m7() { 	// 실패해야 한다.
    throw new RuntimeException("실패");
  }
  public static void m8() { }				// 테스트 도구가 무시한다.
}
```

`@Test` 애너테이션이 Sample 클래스의 의미에 직접적인 영향을 주지는 않는다. 그저 이 애너테이션에 관심 있는 프로그램에게 추가 정보를 제공할 뿐이다. 더 넓게 이야기하면, 대상 코드의 의미는 그대로 둔 채 그 애너테이션에 관심 있는 도구에서 특별한 처리를 할 기회를 준다.

```java
// 마커 애너테이션을 처리하는 프로그램
import java.lang.reflect.*;

public class RunTests {
  public static void main(String[] args) throw Exception {
    int tests = 0;
    int passed = 0;
    Class<?> testClass = Class.forName(args[0]); // 명령줄로부터 완전 정규화된 클래스 이름을 받아 옴
    for (Method m : testClass.getDeclaredMethods()) { // @Test 애너테이션이 달린 메서드를 차례로 호출
      if (m.isAnnotationPresent(Test.class)) { // isAnnotationPresent는 실행할 메서드를 찾아주는 메서드
        tests++;
        try {
          m.invoke(null);
          passed++;
        } catch (InvocationTargetException wrappedExc) { 	// 테스트 메서드가 예외를 던지면 리플렉션 메커니즘이
          Throwable exc = wrappedExc.getCause();					// InvocationTargetException으로 감싸서 다시 던짐
          System.out.println(m + " 실패: " + exc);				 // 잡아서 원래 예외에 담긴 실패 정보를 추출해 출력
        } catch (Exception exc) {
          System.out.println("잘못 사용한 @Test: " + m);
        }
      }
    }
    System.out.printf("성공: %d, 실패: %d%n", passed, tests - passed);
  }
}
```

`InvocationTargetException` 외의 예외가 발생한다면 `@Test` 애너테이션을 잘못 사용했다는 뜻이다. 아마도 인스턴스 메서드, 매개변수가 있는 메서드, 호출할 수 없는 메서드 등에 달았을 것이다. RunTests로 Sample을 실행했을 때의 출력 메시지는 다음과 같다.

```
public static void Sample.m3() 실패: RuntimeException: Boom
잘못 사용한 @Test: public void Sample.m5()
public static void Sample.m7() 실패: RuntimeException: Crash
성공: 1, 실패: 3
```

### 특정 예외를 던져야만 성공하는 테스트

```java
// 매개변수 하나를 받는 애너테이션 타입
import java.lang.annotation.*;

/**
 * 명시한 예외를 던져야만 성공하는 테스트 메서드용 애너테이션
 */
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
  // Class<? extends Throwable> - Throwable을 확장한 클래스의 Class 객체(모든 예외 타입을 다 수용). 한정적 타입 토큰
  Class<? extends Throwable> value();
}
```

이 애너테이션을 실제 활용하는 모습은 다음과 같다.

```java
// 매개변수 하나짜리 애너테이션을 사용하는 프로그램
public class Sample2 {
  @ExceptionTest(ArithmeticException.class) // class 리터럴을 애너테이션 매개변수의 값으로 사용
  public static void m1() {		// 성공해야 한다.
    int i = 0;
    i = i / i;
  }
  @ExceptionTest(ArithmeticException.class)
  public static void m2() {		// 실패해야 한다. (다른 예외 발생)
    int[] a = new int[0];
    int i = a[1];
  }
  @ExceptionTest(ArithmeticException.class)
  public static void m3() { }	// 실패해야 한다. (예외가 발생하지 않음)
}
```

테스트 도구는 다음과 같다. (앞의  테스트 코드에서 if 문 수정)

``` java
if (m.isAnnotationPresent(ExceptionTest.class)) { 
  tests++;
  try {
    m.invoke(null);
    System.out.printf("테스트 %s 실패: 예외를 던지지 않음%n", m);
  } catch (InvocationTargetException wrappedExc) { 	
    Throwable exc = wrappedExc.getCause();
    // 애너테이션 매개변수의 값을 추출하여 테스트 메서드가 올바른 예외를 던지는지 확인
    Class<? extends Throwable> excType = m.getAnnotation(ExceptionTest.class).value();
    if(excType.isInstance(exc)) {
      passed++;
    } else {
      System.out.printf("테스트 %s 실패: 기대한 예외 %s, 발생한 예외 %s%n", m, excType.getName(), exc);
    }
  } catch (Exception exc) {
    System.out.println("잘못 사용한 @ExceptionTest: " + m);
  }
}
```

테스트 프로그램이 문제없이 컴파일되면 애너테이션 매개변수가 가리키는 예외가 올바른 타입이라는 뜻이다. 단, 해당 예외의 클래스 파일이 컴파일타임에는 존재했으나 런타임에는 존재하지 않을 수 있는데, 이 때는 테스트 러너가 TypeNotPresentException을 던진다.

### 예외를 여러개 명시하는 테스트

```java
// 배열 매개변수를 받는 애너테이션 타입
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
  Class<? extends Throwable>[] value();
}
```

단일 원소 배열에 최적화했지만, 앞서의 `@ExceptionTest`들도 모두 수정 없이 수용한다(유연하다). 원소가 여러 개인 배열을 지정할 때는 다음과 같이 원소들을 중괄호로 감싸고 쉼표로 구분해주기만 하면 된다.

```java
// 배열 매개변수를 받는 애너테이션을 사용하는 코드
@ExceptionTest({ IndexOutOfBoundsException.class, NullPointerException.class })
public static void doublyBad() {
  List<String> list = new ArrayList<>();
  
  // 자바 API 명세에 따르면 다음 메서드는 IndexOutOfBoundsException이나 NullPointerException을 던질 수 있다.
  list.addAll(5, null);
}
```

새로운  `@ExceptionTest`를 지원하도록 테스트 러너를 다음과 같이 수정하면 된다. 꽤 직관적이다.

```java
if (m.isAnnotationPresent(ExceptionTest.class)) { 
  tests++;
  try {
    m.invoke(null);
    System.out.printf("테스트 %s 실패: 예외를 던지지 않음%n", m);
  } catch (Throwable wrappedExc) { 	
    Throwable exc = wrappedExc.getCause();
    int oldPassed = passed;
    Class<? extends Throwable>[] excTypes = m.getAnnotation(ExceptionTest.class).value();
    for (Class<? extends Throwable> excType : excTypes) {
    	if(excType.isInstance(exc)) {
      	passed++;
        break;
    	}  
    }
    if (passed == oldPassed)
      System.out.printf("테스트 %s 실패: %s %n", m, exc);
}
```

### 반복 가능한 애너테이션 타입

자바 8에서는 여러 개의 값을 받는 애너테이션을 다른 방식으로도 만들 수 있다. 배열 매개변수 대신 `@Repeatable` 메타애너테이션(`@Repeatable`을 단 애너테이션은 하나의 프로그램 요소에 여러 번 달 수 있다.)을 다는 방식이다.

단, 주의할 점이 있다.

1. `@Repeatable`을 단 애너테이션을 반환하는 '컨테이너 애너테이션'을 하나 더 정의하고, `@Repeatable`에 이 컨테이너 애너테이션의 class 객체를 매개변수로 전달해야 한다.
2. 컨테이너 애너테이션은 내부 애너테이션 타입의 배열을 반환하는  value 메서드를 정의해야 한다.
3. 컨테이너 애너테이션 타입에는 적절한 보존 정책(`@Retention`)과 적용 대상(`@Target`)을 명시해야 한다. (안 그러면 컴파일  x)

```java
// 반복 가능한 애너테이션
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@Repeatable(ExceptionTestContainer.class) // 컨테이너 애너테이션의 class 객체를 매개변수로 전달
public @interface ExceptionTest {
  Class<? extends Throwable> value();
}

// 컨테이너 애너테이션
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTestContainer {
  ExceptionTest[] value(); // 내부 애너테이션 타입의 배열을 반환
}
```

배열 방식 대신 반복 가능 애너테이션을 적용하면 다음과 같다.

```java
// 반복 가능 애너테이션을 두 번 단 코드
@ExceptionTest(IndexOutOfBoundsException.class)
@ExceptionTest(NullPointerException.class)
public static void doublyBad() { ... }
```

반복 가능 애너테이션을 여러 개 달면 하나만 달았을 때와 구분하기 위해 해당 '컨테이너' 애너테이션 타입이 적용된다. getAnnotationsByType 메서드는 이 둘을 구분하지 않아서 반복 가능 애너테이션과 그 컨테이너 애너테이션을 모두 가져오지만, isAnnotationPresent 메서드는 둘을 명확히 구분한다. 따라서 반복 가능 애너테이션을 여러 번 단 다음 isAnnotationPresent로 반복 가능 애너테이션이 달렸는지 검사한다면 "그렇지 않다"라고 알려준다(컨테이너가 달렸기 때문). 그 결과 그 메서드를 무시하고 지나친다. 같은 이유로, 컨테이너 애너테이션이 달렸는지를 검사하면 반복 가능 애너테이션을 한 번만 단 메서드를 무시하고 지나친다. 따라서 모두 검사하려면 따로따로 확인해야 한다.

```java
// 반복 가능 애너테이션 다루기
// 두 애너테이션을 따로따로 확인
if (m.isAnnotationPresent(ExceptionTest.class) || m.isAnnotationPresent(ExceptionTestContainer.class)) {
  tests++;
  try {
    m.invoke(null);
    System.out.printf("테스트 %s 실패: 예외를 던지지 않음%n", m);
  } catch (Throwable wrappedExc) { 	
    Throwable exc = wrappedExc.getCause();
    int oldPassed = passed;
    ExceptionTest[] excTests = m.getAnnotationsByType(ExceptionTest.class); // 구분 없이 애너테이션을 가져 옴
    for (ExceptionTest excTest : excTests) {
    	if(excTest.value().isInstance(exc)) {
      	passed++;
        break;
    	}  
    }
    if (passed == oldPassed)
      System.out.printf("테스트 %s 실패: %s %n", m, exc);
}
```

이처럼 애너테이션이 명명 패턴보다 낫기 때문에 애너테이션으로 할 수 있는 일을 명명 패턴으로 처리할 이유는 없다.  일반 프로그래머가 애너테이션 타입을 직접 정의할 일은 거의 없지만 자바 프로그래머라면 예외 없이 자바가 제공하는 애너테이션 타입들은 사용해야 한다.



## Item 40. @Override 애너테이션을 일관되게 사용하라

상위 타입의 메서드를 재정의했음을 뜻하는 `@Override`는 메서드 선언에만 달 수 있으며, 일관되게 사용하면 여러 가지 악명 높은 버그들을 예방해준다.

```java
// 영어 알파벳 2개로 구성된 문자열을 표현하는 클래스
public class Bigram {
  private final char first;
  private final char second;
  
  public Bigram(char first, char second) {
    this.first = first;
    this.second = second;
  }
  
  public boolean equals(Bigram b) { // 문제: overriding이 아니라 overloading 됨
    return b.first == first && b.second == second;
  }
  public int hashCode() {
    return 31 * first + second;
  }
  
  public static void main(String[] args) {
    Set<Bigram> s = new HashSet<>();
    for (int i = 0; i < 10; i++)
      for (char ch = 'a'; ch <= 'z'; ch++)
        s.add(new Bigram(ch, ch));
    System.out.println(s.size()); // 260이 출력 됨 - 잘못된 결과!
  }
}
```

Object의 equals를 재정의하려면 매개변수 타입을 Object로 해야한다. Object의 equals는 == 연산자와 똑같이 객체 식별성만을 확인한다. 따라서 같은 소문자를 소유한 바이그램 10개 각각이 서로 다른 객체로 인신된다. Object.equals를 재정의한다는 의도를 명시하면 컴파일러가 오류를 찾아낼 수 있다.

```java
@Override public boolean equals(Bigram b) {
  return b.first == first && b.second == second; // 컴파일 오류 발생
}
```

```java
// 올바른 재정의
@Override public boolean equals(Object o) {
  if (!(o instanceof Bigram))
    return false;
  Bigram b = (Bigram) o;
  return b.first == first && b.second == second;
}
```

**상위 클래스의 메서드를 재정의하려는 모든 메서드에 `@Override` 애너테이션을 달자!** 구체 클래스인데 아직 구현하지 않은 추상 메서드가 남아 있다면 컴파일러가 그 사실을 바로 알려주기 때문에 굳이  `@Override`를 달지 않아도 된다. `@Override`를 일관되게 사용한다면 이처럼 실수로 재정의했을 때 경고해줄 것이다. `@Override`는 클래스뿐 아니라 인터페이스의 메서드를 재정의할 때도 사용할 수 있다. 추상 클래스나 인터페이스에서는 상위 클래스나 상위 인터페이스의 메서드를 재정의하는 모든 메서드에 `@Override`를 다는 것이 좋다.  상위 클래스가 구체 클래스든 추상 클래스든 마찬가지다.



##  Item 41. 정의하려는 것이 타입이라면 마커 인터페이스를 사용하라

아무 메서드도 담고 있지 않고, 단지 자신을 구현하는 클래스가 특정 속성을 가짐을 표시해주는 인터페이스를 마커 인터페이스라 한다. Serializable은 자신을 구현한 클래스의 인스턴스는 ObjectOutputStream을 통해 쓸(write) 수 있다고(직렬화 할 수 있다고)  알려준다.

### 마커 애너테이션 vs. 마커 인터페이스

- 마커 인터페이스는 이를 구현한 클래스의 인스턴스들을 구분하는 타입으로 쓸 수 있으나, 마커 애너테이션은 그렇지 않다.
  - 마커 인터페이스는 어엿한 타입이기 때문에, 마커 애너테이션을 사용했다면 런타임에야 발견될 오류를 컴파일타임에 잡을 수 있다.
- 마커 인터페이스는 적용 대상을 더 정밀하게 지정할 수 있다.
  - 적용 대상(`@Target`)을 Element.TYPE으로 선언한 애너테이션은 모든 타입에 달 수 있다. 더 세밀하게 제한하지 못한다.
  - 특정 인터페이스를 구현한 클래스에만 적용하고 싶은 마커가 있다면, 마커를 인터페이스로 정의해 클래스에서 그 인터페이스를 구현하면 된다.
  - Set 인터페이스도 일종의 마커 인터페이스로 볼 수 있다. (Collection의 하위 타입에만 적용할 수 있으며, Collection이 정의한 메서드 외에는 새로 추가한 것이 없다.)
  - 마커 인터페이스는 객체의 특정 부분을 불변식으로 규정하거나, 그 타입의 인스턴스는 다른 클래스의 특정 메서드가 처리할 수 있다는 사실을 명시하는 용도로 사용할 수 있다. (ex. Serializable 인터페이스는  ObjectOutputStream이 처리할 수 있는 인스턴스임을 명시)
- 마커 애너테이션은 마커 인터페이스와 달리 거대한 애너테이션 시스템의 지원을 받는다.
  - 애너테이션을 적극 활용하는 프레임워크에서는 마커 애너테이션을 쓰는 것이 일관성을 지키는 데 유리하다.

### 언제 어떤 것을 사용해야 하는가?

- 클래스와 인터페이스 외의 프로그램 요소(모듈, 패키지, 필드, 지역변수 등)에 마킹해야 할 때는 애너테이션을 쓸 수밖에 없다.
  - 클래스와 인터페이스만이 인터페이스를 구현하거나 확장할 수 있기 때문이다.
- 마킹이 된 객체를 매개변수로 받는 메서드를 작성할 일이 있다면 마커 인터페이스를 써야 한다.
  - 마커 인터페이스를 해당 메서드의 매개변수 타입으로 사용하여 컴파일타임에 오류를 잡아낼 수 있다.
  - 이런 메서드를 작성할 일이 절대 없다고 확인한다면 마커 애너테이션이 더 낫다.
- 애너테이션을 활발히 활용하는 프레임워크에서 사용하려는 마커라면 마커 애너테이션을 사용하는 편이 좋다.
