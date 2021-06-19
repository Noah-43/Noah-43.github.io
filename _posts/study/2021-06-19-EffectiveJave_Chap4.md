---
title: EffectiveJava_Chap4
excerpt: 이펙티브 자바 책 4장 정리
categories: study
---

# 4장. 클래스와 인터페이스

## Item 15. 클래스와 멤버의 접근 권한을 최소화하라

잘 설계된 컴포넌트는 모든 내부 구현을 완벽히 숨겨, 구현과 API를 깔끔히 분리한다. 오직 API를 통해서만 다른 컴포넌트와 소통하며 서로의 내부 동작 방식에는 전혀 개의치 않는다. 이는 정보 은닉, 혹은 캡슐화라고 하는 개념으로 소프트웨어 설계의 근간이 되는 원리다.

### 정보 은닉의 장점

- 시스템 개발 속도를 높인다. 여러 컴포넌트를 병렬로 개발할 수 있기 때문이다.
- 시스템 관리 비용을 낮춘다. 각 컴포넌트를 더 빨리 파악하여 디버깅할 수 있고, 다른 컴포넌트로 교체하는 부담도 적기 때문이다.
- 정보 은닉 자체가 성능을 높여주지는 않지만, 성능 최적화에 도움을 준다. 완성된 시스템을 프로파일링해 최적화할 컴포넌트를 정한 다음, 다른 컴포넌트에 영향을 주지 않고 해당 컴포넌트만 최적화할 수 있기 때문이다.
- 소프트웨어 재사용성을 높인다. 외부에 거의 의존하지 않고 독자적으로 동작할 수 있는 컴포넌트라면 그 컴포넌트와 함께 개발되지 않은 낯선 환경에서도 유용하게 쓰일 가능성이 크기 때문이다.
- 큰 시스템을 제작하는 난이도를 낮춰준다. 시스템 전체가 아직 완성되지 않은 상태에서도 개별 컴포넌트의 동작을 검증할 수 있기 때문이다.

### 접근 제한자 활용

모든 클래스와 멤버의 접근성을 가능한 한 좁혀야 한다. 소프트웨어가 올바르게 동작하는 한 항상 가장 낮은 접근 수준을 부여해야 한다. 톱레벨 클래스와 인터페이스에 부여할 수 있는 접근 수준은 package-private과 public 두 가지다. 톱레벨 클래스나 인터페이스를 public으로 선언하면 공개 API가 되고, package-private으로 선언하면 해당 패키지 안에서만 이용할 수 있다.

멤버(필드, 메서드, 중첩 클래스, 중첩 인터페이스)에 부여할 수 있는 접근 수준은 네 가지다.

- private: 멤버를 선언한 톱레벨 클래스에서만 접근할 수 있다.
- package-private: 멤버가 소속된 패키지 안의 모든 클래스에서 접근할 수 있다. 접근 제한자를 명시하지 않았을 때 적용되는 패키지 접근 수준이다(단, 인터페이스의 멤버는 기본적으로 public이 적용된다).
- protected: package-private의 접근 범위를 포함하며, 이 멤버를 선언한 클래스의 하위 클래스에서도 접근할 수 있다.
- public: 모든 곳에서 접근할 수 있다.

### public 사용시 주의사항

public 클래스의 인스턴스 필드는 되도록 public이 아니어야 한다. public 가변 필드를 갖는 클래스는 일반적으로 Thread Safe 하지 않다. 해당 클래스가 표현하는 추상 개념을 완성하는 데 꼭 필요한 구성요소로써의 상수라면 public static final 필드로 공개해도 된다. 이런 필드는 반드시 기본 타입 값이나 불변 객체를 참조해야한다. 클래스에서 public static final 배열 필드를 두거나 이 필드를 반환하는 접근자 메서드를 제공해서는 안된다.

```java
// 보안 허점 존재
public static final Thing[] VALUES = { ... };
```

```JAVA
// 해결책 1. public 배열을 private으로 만들고 public 불변 리스트를 추가
private static final Thing[] PRIVATE_VALUES = { ... };
public static final List<Thing> VALUES = Collections.unmodifiableList(Arrays.asList(PRIVATE_VALUES));
```

```java
// 해결책 2. public 배열을 private으로 만들고 그 복사본을 반환하는 public 메서드 추가(방어적 복사)
private static final Thing[] PRIVATE_VALUES = { ... };
public static final Thing[] values() {
    return PRIVATE_VALUES.clone();
}
```

### 모듈 시스템

패키지가 클래스들의 묶음이듯, 모듈은 패키지들의 묶음이다. 모듈은 자신에 속하는 패키지 중 공개(export)할 것들을 선언한다. protected 혹은 public 멤버라도 해당 패키지를 공개하지 않았다면 모듈 외부에서는 접근할 수 없다. 모듈 시스템을 활용하면 클래스를 외부에 공개하지 않으면서도 같은 모듈을 이루는 패키지 사이에서는 자유롭게 공유할 수 있다. 대표적인 예는 JDK 자체다. 자바 라이브러리에서 공개하지 않은 패키지들은 해당 모듈 밖에서는 절대로 접근할 수 없다. 모듈의 장점을 누리려면 해야 할 일이 많고 JDK 외에도 모듈 개념이 널리 받아들여질지 예측하기 어려우니 꼭 필요한 경우가 아니라면 사용하지 않는 것이 좋을지도...



## Item 16. public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라

```java
// 접근자와 변경자 메서드를 활용해 데이터를 캡슐화
class Point {
    private double x;
    private double y;
    
    public Point(double x, double y) {
        this.x = x;
        this.y = y;
    }
    
    public double getX() { return x; }
    public double getY() { return y; }
    
    public void setX(double x) { this.x = x; }
    public void setY(double y) { this.y = y; }
}
```

public 클래스는 절대 가변 필드를 직접 노출해서는 안 된다. 패키지 바깥에서 접근할 수 있는 클래스라면 접근자를 제공함으로써 클래스 내부 표현 방식을 언제든 바꿀 수 있는 유연성을 얻을 수 있다. package-private 클래스 혹은 private 중첩 클래스라면 데이터 필드를 노출한다 해도 문제가 없다. 그 클래스가 표현하려는 추상 개념만 올바르게 표현해주면 된다. 이 방식은 접근자 방식보다 훨씬 깔끔하다.



## Item 17. 변경 가능성을 최소화하라

불변 클래스는 그 인스턴스의 내부 값을 수정할 수 없는 클래스다. 불변 클래스는 가변 클래스보다 설계하고 구현하고 사용하기 쉬우며, 오류가 생길 여지도 적고 안전하다.

### 불변 클래스 규칙

- **객체의 상태를 변경하는 메서드(변경자)를 제공하지 않는다.**
- **클래스를 확장할 수 없도록 한다.** 하위 클래스에서 부주의하게 혹은 나쁜 의도로 객체의 상태를 변하게 만드는 사태를 막아준다. 상속을 막는 대표적인 방법은 클래스를 final로 선언하는 것이지만, 다른 방법도 존재한다.
- **모든 필드를 final로 선언한다.** 시스템이 강제하는 수단을 이용해 설계자의 의도를 명확히 드러내는 방법이다. 새로 생성된 인스턴스를 동기화 없이 다른 스레드로 건네도 문제없이 동작하게끔 보장하는 데도 필요하다.
- **모든 필드를 private으로 선언한다.** 필드가 참조하는 가변 객체를 클라이언트에서 직접 접근해 수정하는 일을 막아준다. 기술적으로는 기본 타입 필드나 불변 객체를 참조하는 필드를 public final로만 선언해도 불변 객체가 되지만, 이렇게 하면 다음 릴리스에서 내부 표현을 바꾸지 못하므로 권하지는 않는다.
- **자신 외에는 내부의 가변 컴포넌트에 접근할 수 없도록 한다.** 클래스에 가변 객체를 참조하는 필드가 하나라도 있다면 클라이언트에서 그 객체의 참조를 얻을 수 없도록 해야 한다. 이런 필드는 절대 클라이언트가 제공한 객체 참조를 가리키게 해서는 안 되며, 접근자 메서드가 그 필드를 그대로 반환해서도 안 된다. 생성자, 접근자, readObject 메서드 모두에서 방어적 복사를 수행하라.

### 불변 클래스 예시

```java
// 불변 복소수 클래스
public final class Complex {
    private final double re;
    private final double im;
    
    public Complex(double re, double im) {
        this.re = re;
        this.im = im;
    }
    
    public double realPart()	  { return re; }
    public double imaginaryPart() { return im; }
    
    // 사칙연산 메서드. 새로운 Complex 인스턴스를 만들어 반환. 메서드 이름을 동사 대진 전치사를 사용(객체의 값을 변경하지 않는다는 사실 강조)
    // 피연산자에 함수를 적용해 그 결과를 반환하지만, 피연산자 자체는 그대로인 프로그래밍 패턴(함수형 프로그래밍) <-> 절차적 or 명령형 프로그래밍
    public Complex plus(Complex c) {
        return new Complex(re + c.re, im - c.im);
    }
    
    public Complex minus(Complex c) {
        return new Complex(re - c.re, im - c.im);
    }
    
    public Complex times(Complex c) {
        return new Complex(re * c.re - im * c.im,
                           re * c.im + im * c.re);
    }
    
    public Complex dividedBy(Complec c) {
        double tmp = c.re * c.re + c.im * c.im;
        return new Complex((re * c.re + im * c.im) / tmp,
                           (im * c.re - re * c.im) / tmp);
    }
    
    @Override public boolean equals(Object o) {
        if (o == this)
            return true;
        if (!(o instanceof Complex))
            return false;
        Complex c = (Complex) o;
        
        return Double.compare(c.re, re) == 0
            && Double.compare(c.im, im) == 0;
    }
    
    @Override public int hashCode() {
        return 31 * Double.hashCode(re) + Double.hashCode(im);
    }
    
    @Override public String toString() {
        return "(" + re + " + " + im + "i)";
    }
}
```



### 불변 객체 특징 (장단점)

- 불변 객체는 근본적으로 Thread Safe하여 따로 동기화 할 필요 없다. 따라서 불변 객체는 안심하고 공유할 수 있다. 불변 클래스는 한 번 만든 인스턴스를 최대한 재활용하기를 권한다. 자주 쓰이는 값들을 상수(public static final)로 제공 하는 것은 가장 쉬운 재활용 방법이다.

```JAVA
// Complex 클래스 상수 예시
public static final Complex ZERO = new Complex(0, 0);
public static final Complex ONE  = new Complex(1, 0);
public static final Complex I    = new Complex(0, 1);
```

- 불변 클래스는 자주 사용되는 인스턴스를 캐싱하여 같은 인스턴스를 중복 생성하지 않게 해주는 정적 팩터리를 제공할 수 있다.
- 불변 객체를 자유롭게 공유할 수 있다는 점은 방어적 복사도 필요 없다는 뜻이다. clone 메서드나 복사 생성자가 필요없다.
- 불변 객체는 자유롭게 공유할 수 있음은 물론, 불변 객체끼리는 내부 데이터를 공유할 수 있다.
- 객체를 만들 때 다른 불변 객체들을 구성요소로 사용하면 그 구조가 아무리 복잡하더라도 불변식을 유지하기 훨씬 수월하다.
- 불변 객체는 그 자체로 실패 원자성('메서드에서 예외가 발생한 후에도 그 객체는 여전히 (메서드 호출 전과 똑같은) 유효한 상태여야 한다'는 성질)을 제공한다.
- 값이 다르면 반드시 독립된 객체로 만들어야한다.

### 불변 클래스를 만드는 방법

```JAVA
public class Complex { // 클라이언트에서 바라본 이 객체는 사실상 final이다. public이나 protected 생성자가 없어 다른 패키지에서 확장이 불가능.
    private final double re;
    private final double im;
    
    // 생성자를 private으로!
    private Complex(double re, double im) {
        this.re = re;
        this.im = im;
    }
    
    // 생성자 대신 정적 팩터리를 제공
    public static Complex valueOf(double re, double im) {
        return new Complex(re, im);
    }
    ...
}
```

```java
// 인수로 받은 객체가 '진짜' BigInteger인지 확인.
public static BigInteger safeInstance(BigInteger val) {
    return val.getClass() == BigInteger.class ?
        	val : new BigInteger(val.toByteArray()); // 신뢰할 수 없는 하위 클래스의 인스턴스라고 확인되면, 가변이라고 가정하고 방어적으로 복사.
}
```

### 불변 클래스 주의사항

- 클래스는 꼭 필요한 경우가 아니라면 불변이어야 한다. 그러니 getter가 있다고 해서 무조건 setter를 만들지는 말자.
- 불변으로 만들 수 없는 클래스라도 변경할 수 있는 부분을 최소한으로 줄이자. 합당한 이유가 없다면 모든 필드는 private final이어야 한다.
- 생성자는 불변식 설정이 모두 완료된, 초기화가 완벽히 끝난 상태의 객체를 생성해야 한다. 확실한 이유가 없다면 생성자와 정적 팩터리 외에는 그 어떤 초기화 메서드도 public으로 제공해서는 안 된다.



## Item 18. 상속보다는 컴포지션을 사용하라

메서드 호출과 달리 상속은 캡슐화를 깨뜨린다. 상위 클래스가 어떻게 구현되느냐에 따라 하위 클래스의 동작에 이상이 생길 수 있다.

- 상위 클래스의 내부 구현이 달라지면 그 여파로 하위 클래스가 오동작 할 수 있다.
- 상위 클래스에 새로운 메서드를 추가하면 보안이 깨지는 경우가 발생할 수 있다.
- 하위 클래스에서 추가한 메서드를 후에 상위 클래스에서 같은 시그니처로 메서드를 구현하면 컴파일 오류 혹은 위의 문제들과 같은 상황이 발생한다.

### 컴포지션 구성

기존 클래스를 확장하는 대신, 새로운 클래스를 만들고 private 필드로 기존 클래스의 인스턴스를 참조한다. 새 클래스의 인스턴스 메서드들은 기존 클래스의 대응하는 메서드를 호출해 그 결과를 반환한다.(forwarding)

```java
// 래퍼 클래스 - 컴포지션 사용.
public class InstrumentedSet<E> extends ForwardingSet<E> {
    private int addCount = 0;
    
    public InstrumentedSet(Set<E> s) {
        super(s);
    }
    
    @Override public boolean add(E e) {
        addCount++;
        return super.add(e);
    }
    
    @Override public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }
    
    public int getAddCount() {
        return addCount;
    }
}
```

```java
// 재사용할 수 있는 전달 클래스
public class ForwardingSet<E> implements Set<E> { // Set 인터페이스를 구현
    private final Set<E> s;
    public ForwardingSet(Set<E> s) { this.s = s; } // Set의 인스턴스를 인수로 받는 생성자
    
    public void clear() { s.clear(); }
    public boolean contains(Object o) { return s.contains(o); }
    public boolean isEmpty() { return s.isEmpty(); }
    public int size() { return s.size(); }
    public Iterator<E> iterator() { return s.iterator(); }
    public boolean add(E e) { return s.add(e); }
    public boolean remove(Object o) { return s.remove(o); }
    public boolean containsAll(Collection<?> c) { return s.containsAll(c); }
    public boolean addAll(Collection<?> c) { return s.addAll(c); }
    public boolean removeAll(Collection<?> c) { return s.removeAll(c); }
    public boolean retainAll(Collection<?> c) { return s.retainAll(c); }
    public Object[] toArray() { return s.toArray(); }
    public <T> T[] toArray(T[] a) { return s.toArray(a); }
    @Override public boolean equals(Object o) { return s.equals(o); }
    @Override public int hashCode() { return s.hashCode(); }
    @Override public String toString() { return s.toString(); }
}
```

어떠한 Set 구현체라도 계측할 수 있으며, 기존 생성자들과도 함께 사용할 수 있다.

```java
Set<Instant> times = new InstrumentedSet<>(new TreeSet<>(cmp));
Set<E> s = new InstrumentedSet<>(new HashSet<>(INIT_CAPACITY));
```

### 래퍼 클래스

다른 Set 인스턴스를 감싸고 있다는 뜻에서 InstrumentedSet 같은 클래스를 래퍼 클래스라 하며, 다른 Set에 계측 기능을 덧씌운다는 뜻에서 데코레이터 패턴이라고 한다. 래퍼 객체가 내부 객체에 자기 자신의 참조를 넘기는 경우는 위임이라고 한다. 래퍼 클래스는 콜백 프레임워크와는 어울리지 않는다는 단점이 있다. 콜백 프레임워크에서는 자기 자신의 참조를 다른 객체에 넘겨서 다음 호출 때 사용하도록 하는데, 내부 객체는 자신을 감싸고 있는 래퍼의 존재를 모르니 자신(this)의 참조를 넘기고, 콜백 때 래퍼가 아닌 내부 객체를 호출하게 된다(SELF 문제). 전달 메서드들을 구현해 둔 좋은 예로, 구아바는 모든 컬렉션 인터페이스용 전달 메서드를 전부 구현해뒀다.

### 상속 사용 주의사항

- 상속은 하위 클래스가 상위 클래스와 is-a 관계일 때만 쓰여야 한다. (필수 구성요소 인지 확인)
- 확장하려는 클래스의 API에 아무런 결함이 없는지 확인해야 한다.
- 결함이 있다면 하위 클래스의 API까지 전파돼도 괜찮은지 확인해야 한다.



## Item 19. 상속을 고려해 설계하고 문서화하라. 그러지 않았다면 상속을 금지하라

- 상속용 클래스는 재정의할 수 있는 메서드들을 내부적으로 어떻게 이용하는지 문서로 남겨야 한다. 재정의 가능 메서드를 호출할 수 있는 모든 상황을 문서로 남겨야 한다. API 문서의 메서드 설명 끝에 "Implementation Requirements"로 시작하는 절은 그 메서드의 내부 동작 방식을 설명하는 곳이다. 메서드 주석에 `@implSpec` 태그를 붙여주면 자바독 도구가 생성해준다. 클래스를 안전하게 상속할 수 있도록 하려면 내부 구현 방식을 설명해야만 한다. `@ImplSpec` 태그를 활성화하려면 명령줄 매개변수로 `-tag "implSpec:a:Implementation Requirements:"`를 지정해주면 된다.
- 효율적인 하위 클래스를 큰 어려움 없이 만들 수 있게 하려면 클래스의 내부 동작 과정 중간에 끼어들 수 있는 훅(hook)을 잘 선별하여 protected 메서드 형태로 공개해야 할 수도 있다.
- 상속용 클래스를 시험하는 방법은 직접 하위 클래스를 만들어보는 방법이 '유일'하다. 상속용으로 설계한 클래스는 배포 전에 반드시 하위 클래스를 만들어 검증해야 한다.
- 상속용 클래스의 생성자는 직접적으로든 간접적으로든 재정의 가능 메서드를 호출해서는 안 된다. (private, final, static 메서드는 재정의가 불가능하니 생성자에서 안심하고 호출해도 된다.)
- clone과 readObject 메서드는 생성자와 비슷한 효과를 내기 때문에 직접적으로든 간접적으로든 재정의 가능 메서드를 호출해서는 안 된다.
- Serializable을 구현한 상속용 클래스가 readResolve나 writeReplace 메서드를 갖는다면 이 메서드들은 private이 아닌 protected로 선언해야 한다.
- 가장 좋은 방법은 상속용으로 설계하지 않은 클래스는 상속을 금지하는 것이다.
  1. 클래스를 final로 선언한다.
  2. 모든 생성자를 private이나 package-private으로 선언하고 public 정적 팩터리를 만들어 준다.



## Item 20. 추상 클래스보다는 인터페이스를 우선하라

자바가 제공하는 다중 구현 메커니즘은 인터페이스와 추상 클래스가 있다. 둘의 가장 큰 차이는 추상 클래스가 정의한 타입을 수현하는 클래스는 반드시 추상 클래스의 하위 클래스가 되어야 한다는 점이다. 인터페이스가 요구하는 메서드를 추가하고, 클래스 선언에 implements 구문만 추가하면 기존 클래스에 손쉽게 새로운 인터페이스를 구현해 넣을 수 있다. 반면 기존 클래스 위에 새로운 추상 클래스를 끼워넣기는 어렵다.

### 인터페이스가 추상 클래스보다 좋은 점

- 인터페이스는 믹스인(mixin) 정의에 안성맞춤이다. 믹스인이란 클래스가 구현할 수 있는 타입으로, 믹스인을 구현한 클래스에 원래의 '주된 타입' 외에도 특정 선택적 행위를 제공한다고 선언하는 효과를 준다. 대상 타입의 주된 기능에 선택적 기능을 '혼합'한다고 해서 믹스인이라 부른다. 추상 클래스는 믹스인이 불가능하다.

- 인터페이스로는 계층구조가 없는 타입 프레임워크를 만들 수 있다.

```java
// 가수 인터페이스
public interface Singer {
    AudioClip sing(Song s);
}
// 작곡가 인터페이스
public interface Songwriter {
    Song compose(int chartPosition);
}
// 싱어송라이터~ 제3의 인터페이스
public interface SingerSongwriter extends Singer, Songwriter { // 두 개를 확장
	// 새로운 메서드 추가
    AudioClip strum();
    void actSensitive();
}
```

- 래퍼 클래스 관용구와 함께 사용하면 인터페이스는 기능을 향상시키는 안전하고 강력한 수단이 된다.
- 인터페이스의 메서드 중 구현 방법이 명백한 것이 있다면, 그 구현을 디폴트 메서드로 제공할 수 있다.(단, 제약은 있다.)

### 인터페이스 + 추상 클래스

인터페이스와 추상 골격 구현 클래스를 함께 제공하는 식으로 장점을 모두 취할 수도 있다. 인터페이스로는 타입을 정의하고, 필요하면 디폴트 메서드 몇 개도 함께 제공한다. 골격 구현 클래스는 나머지 메서드들까지 구현한다. 이는 템플릿 메서드 패턴이다. 관례상 이름이 Interface라면 그 골격 구현 클래스의 이름은 AbstractInterface로 짓는다. 컬렉션 프레임워크의 AbstractCollection, AbstractSet, AbstractList, AbstractMap 각각이 핵심 컬렉션 인터페이스의 골격 구현이다.

```java
// 골격 구현을 사용해 완성한 구체 클래스
static List<Integer> intArrayAsList(int[] a) {
    Objects.requireNonNull(a);
    
    // 다이아몬드 연산자를 이렇게 사용하는 건 자바 9부터 가능하다.
    // 더 낮은 버전을 사용한다면 <Integer>로 수정하자.
    return new AbstractList<>() { // 골격 구현
        @Override public Integer get(int i) {
            return a[i];	// 오토박싱
        }
        
        @Override public Integer set(int i, Integer val) {
            int oldVal = a[i];
            a[i] = val;		// 오토언박싱
            return oldVal;  // 오토박싱
        }
        
        @Override public int size() {
            return a.length;
        }
    };
}
```

### 골격 구현 클래스

골격 구현 작성은 다음을 따르면 된다.

1. 인터페이스를 잘 살펴 다른 메서드들의 구현에 사용되는 기반 메서드들을 선정한다. 이 기반 메서드들은 골격 구현에서 추상 메서드가 된다.
2. 기반 메서드들을 사용해 직접 구현할 수 있는 메서드를 모두 디폴트 메서드로 제공한다. 단, equals와 hashCode 같은 Object의 메서드는 디폴트 메서드로 제공하면 안 된다.
3. 인터페이스의 메서드 모두가 기반 메서드와 디폴트 메서드가 된다면 골격 구현 클래스를 별도로 만들 이유는 없다.
4. 기반 메서드나 디폴트 메서드로 만들지 못한 메서드가 남아 있다면, 이 인터페이스를 구현하는 골격 구현 클래스를 하나 만들어 남은 메서드들을 작성한다. 골격 구현 클래스에는 필요하면 public이 아닌 필드와 메서드를 추가해도 된다.

```java
public abstract class AbstractMapEntry<K,V> implements Map.Entry<K,V> {
    // getKey, getValue는 기반 메서드
    // 변경 가능한 엔트리는 이 메서드를 반드시 재정의해야 한다.
    @Override public V setValue(V value) {
        throw new UnsupportedOperationException();
    }
    
    // Object 메서드들은 디폴트 메서드로 제공해서는 안되므로 골격 구현 클래스에  구현
    // Map.Entry.equals의 일반 규약을 구현한다.
    @Override public boolean equals(Object o) {
        if (o == this)
            return true;
        if (!(o instanceof Map.Entry))
            return false;
        Map.Entry<?,?> e = (Map.Entry) o;
        return Objects.equals(e.getKey(), getKey()) && Objects.equals(e.getValue(), getValue());
    }
    
    // Map.Entry.hashCode의 일반 규약을 구현한다.
    @Override public int hashCode() {
        return Objects.hashCode(getKey()) ^ Objects.hashCode(getValue());
    }
    
    @Override public String toString() {
        return getKey() + "=" + getValue(); // 기반 메서드를 사용해 구현
    }
}
```

Map.Entry 인터페이스나 그 하위 인터페이스로는 이 골격 구현을 제공할 수 없다. 디폴트 메서드는 equals, hashCode, toString 같은 Object 메서드를 재정의 할 수 없기 때문이다.

골격 구현은 기본적으로 상속해서 사용하는 것을 가정하므로, 인터페이스에 정의한 디폴트 메서드든 별도의 추상 클래스든, 골격 구현은 반드시 그 동작 방식을 잘 정리해 문서로 남겨야 한다.

단순 구현(simple implementation)은 골격 구현의 작은 변종으로, AbstractMap.SimpleEntry가 좋은 예다. 단순 구현도 골격 구현과 같이 상속을 위해 인터페이스를 구현한 것이지만 추상 클래스가 아니다.



## Item 21. 인터페이스는 구현하는 쪽을 생각해 설계하라

자바 8 전에는 인터페이스에 메서드를 추가했을 대 기존 구현체에 존재하지 않으면 컴파일 오류가 났다. 자바 8에 와서 기존 인터페이스에 메서드를 추가할 수 있도록 디폴트 메서드가 생겼다. 디폴트 메서드를 선언하면, 그 인터페이스를 구현한 후 디폴트 메서드를 재정의하지 않은 모든 클래스에서 디폴트 구현이 쓰이게 된다. 자바 8에서는 핵심 컬렉션 인터페이스들에 다수의 디폴트 메서드가 추가되었지만, 생각할 수 있는 모든 상황에서 불변식을 해치지 않는 디폴트 메서드를 작성하는 것은 어렵다. 디폴트 메서드는 (컴파일에 성공하더라도) 기존 구현체에 런타임 오류를 일으킬 수 있다. 기존 인터페이스에 디폴트 메서드로 새 메서드를 추가하는 일은 꼭 필요한 경우가 아니면 피해야 한다(새로운 인터페이스를 만드는 경우라면 표준적인 메소드 구현에 매우 유용하다). 디폴트 메서드는 인터페이스로부터 메서드를 제거하거나 기존 메서드의 시그니처를 수정하는 용도가 아니다. 결론적으로 인터페이스를 설계할 때는 세심한 주의를 기울여야 하며, 새로운 인터페이스 릴리즈 전에 반드시 테스트를 거쳐야한다. 인터페이스를 릴리스한 후라도 결함을 수정이 가능한 경우도 있겠지만, 절대 그 가능성에 기대서는 안 된다.



## Item 22. 인터페이스는 타입을 정의하는 용도로만 사용하라

인터페이스는 자신을 구현한 클래스의 인스턴스를 참조할 수 있는 타입 역할을 한다. 메서드 없이 상수를 뜻하는 static final 필드로만 가득 찬 상수 인터페이스는 그 역활과 맞지 않다.

```java
// 상수 인터페이스 안티패턴. 인터페이스를 잘못 사용한 예 - 사용금지!
public interface PhysicalConstants {
    // 아보가드로 수 (1/몰)
    static final double AVOGADROS_NUMBER	= 6.022_140_857e23;
    // 볼츠만 상수 (J/K)
    static final double BOLTZMANN_CONSTANT	= 1.380_648_52e-23;
    // 전자 질량 (kg)
    static final double ELECTRON_MASS	    = 9.109_383_56e-31;
}
```

클래스 내부에서 사용하는 상수는 외부 인터페이스가 아니라 내부 구현에 해당한다. 특정 클래스나 인터페이스와 강하게 연관된 상수라면 그 클래스나 인터페이스 자체에 추가해야 한다. 상수를 공개할 목적이라면 열거 타입으로 만들거나, 인스턴스화할 수 없는 유틸리티 클래스에 담아 공개하면 된다.

```java
package constantutilityclass; // 대충 유틸 클래스 패키지 어딘가

public class PhusicalConstants {
    private PhysicalConstants() { } // 인스턴스화 방지    
    
    // 아보가드로 수 (1/몰)
    public static final double AVOGADROS_NUMBER		= 6.022_140_857e23; // 자바 7부터 가독성을위해 '_' 사용 가능. 십진수 리터럴도 가능.
    // 볼츠만 상수 (J/K)
    public static final double BOLTZMANN_CONSTANT	= 1.380_648_52e-23;
    // 전자 질량 (kg)
    public static final double ELECTRON_MASS	    = 9.109_383_56e-31;
}
```

유틸리티 클래스에 정의된 상수를 클라이언트에서 사용하려면 `PhysicalConstants.AVOGADROS_NUMBER` 이런식으로 클래스 이름까지 함께 명시해야 한다. 유틸리티 클래스의 상수를 빈번히 사용한다면 정적임포트를하여 클래스 이름을 생략할 수 있다.

```java
import static constantutilityclass.PhysicalConstants.*;

public class Test {
    double atoms(double mols) {
        return AVOGADROS_NUMBER * mols;
    }
    ...
    // PhysicalConstants를 빈번히 사용한다면 정적 임포트가 값어치를 한다.
}
```



## Item 23. 태그 달린 클래스보다는 클래스 계층구조를 활용하라

### 태그 달린 클래스

```java
class Figure {
    enum Shape { RECTANGLE, CIRCLE };
    
    // 태그 필드 - 현재 모양을 나타낸다.
    final Shape shape;
    
    // 다음 필드들은 모양이 사각형(RECTANGLE)일 때만 쓰인다.
    double length;
    double width;
    
    // 다음 필드는 모양이 원(CIRCLE)일 때만 쓰인다.
    double radius;
    
    // 원 생성자
    Figure(double radius) {
        shape = Shape.CIRCLE;
        this.radius = radius;
    }
    
    // 사각형 생성자
    Figure(double length, double width) {
        shape = Shape.RECTANGLE;
        this.length = length;
        this.width = width;
    }
    
    double area() {
        switch(shape) {
            case RECTANGLE:
                return length * width;
            case CIRCLE:
                return Math.PI * (radius * radius);
            default:
                throw new AssertionError(shape);
        }
    }
}
```

- 열거 타입 선언, 태그 필드, switch문 등 쓸데없는 코드가 많다.
- 가독성이 나쁘다.
- 다른 의미를 위한 코드가 있어 메모리를 많이 사용한다.
- 필드들을 final로 선언하려면 해당 의미에 쓰이지 않는 필드들까지 생성자에서 초기화해야한다.
- **태그 달린 클래스는 장황하고, 오류를 내기 쉽고, 비효율적이다.**

### 클래스 계층구조

1. 계층구조의 루트가 될 추상 클래스를 정의하고, 태그 값에 따라 동작이 달라지는 메서드들을 루트 클래스의 추상 메서드로 선언한다.
2. 태그 값에 상관없이 동작이 일정한 메서드들을 루트 클래스에 일반 메서드로 추가한다.
3. 모든 하위 클래스에서 곹옹으로 사용하는 데이터 필드들도 전부 루트 클래스에 추가한다.
4. 루트 클래스를 확장한 구체 클래스를 의미별로 정의해 각자의 의미에 해당하는 데이터 필드들을 추가한다.
5. 투르 클래스가 정의한 추상 메서드를 각자의 의미에 맞게 구현한다.

```java
// 루트 클래스
abstract class Figure {
    abstract double area();
}

// 원 클래스
class Circle extends Figure {
    final double radius;
    
    Circle(double radius) { this.radius = radius; }
    
    @Override double area() { return Math.PI * (radius * radius); }
}

// 사각형 클래스
class Rectangle extends Figure {
    final double length;
    final double width;
    
    Rectangle(double length, double width) {
        this.length = length;
        this.width = width;
    }
    
    @Override double area() { return length * width; }
}
```

클래스 계층구조를 사용하면 루트 클래스의 코드를 건드리지 않고도 프로그래머들이 독립적으로 계층구조를 확장하고 함께 사용할 수 있다. 타입이 의미별로 따로 존재해 변수의 의미를 명시하거나 제한할 수 있고, 특정 의미만 매개변수로 받을 수 있다. 타입 사이의 자연스러운 계층 관계를 반영할 수 있어서 유연성은 물론 컴파일타임 타입 검사 능력도 높아진다.

정사각형(사각형의 특별한 형태)도 지원하도록 다음과 같이 수정할 수 있다.

```java
class Square extends Rectangle {
    Square(double side) {
        super(side, side);
    }
}
```



## Item24. 멤버 클래스는 되도록 static으로 만들라

중첩 클래스란 다른 클래스 안에 정의된 클래스를 말한다. 중처버 클래스는 자신을 감싼 바깥 클래스에서만 쓰여야 하며, 그 외의 쓰임새가 있다면 톱레벨 클래스로 만들어야 한다. 중첩 클래스의 종류는 정적 멤버 클래스, (비정적) 멤버 클래스, 익명 클래스, 지역 클래스, 이렇게 네 가지다. 이 중 첫번째를 제외한 나머지는 내부 클래스에 해당한다.

### 정적 멤버 클래스

정적 멤버 클래스는 다른 클래스 안에 선언되고, 바깥 클래스의 private 멤버에도 접근할 수 있다. 정적 멤버 클래스는 private으로 선언하면 바깥 클래스에서만 접근할 수 있다. 정적 멤버 클래스는 흔히 바깥 클래스와 함께 쓰일 때만 유용한 public 도우미 클래스로 쓰인다. 계산기가 지원하는 연산 종류를 정의하는 Operation 열거 타입은 Calculator 클래스의 public 정적 멤버 클래스가 되어야 한다. 그러면 Calculator의 클라이언트에서 `Calculator.Operatoin.PLUS` 같은 형태로 원하는 연산을 참조할 수 있다. private 정적 멤버 클래스는 흔히 바깥 클래스가 표현하는 객체의 한 부분(구성요소)을 나타낼 때 쓴다.

### 비정적 멤버 클래스

비정적 멤버 클래스의 인스턴스는 바깥 클래스의 인스턴스와 암묵적으로 연결된다. 그래서 비정적 멤버 클래스의 인스턴스 메서드에서 정규화된 this(`클래스명.this`)를 사용해 바깥 인스턴스의 메서드를 호출하거나 바깥 인스턴스의 참조를 가져올 수 있다. 개념상 중첩 클래스의 인스턴스가 바깥 인스턴스와 독립적으로 존재할 수 있다면 정적 멤버 클래스로 만들어야 한다(비정적 멤버 클래스는 바깥 인스턴스 없이는 생성할 수 없긴 때문). 비정적 멤버 클래스의 인스턴스와 바깥 인스턴스 사이의 관계는 멤버 클래스가 인스턴스화될 때 확립되며, 더 이상 변경할 수 없다. 이 관계는 보통 바깥 클래스의 인스턴스 메서드에서 비정적 멤버 클래스의 생성자를 호출할 때 자동으로 만들더진다(`바깥 인스턴스의 클래스.new MemberClass(args)`를 호출해 수동으로 만들기도 함). 이 관계 정보는 비정적 멤버 클래스의 인스턴스 안에 만들어진다.

```java
// 비정적 멤버 클래스의 흔한 쓰임 - 자신의 반복자 구현
public class MySet<E> extends AbstractSet<E> {
    ... // 생략
    
    @Override public Iterator<E> iterator() {
        return new MyIterator(); // 관계가 만들어지는 시점
    }
    
    private class MyIterator implements Iterator<E> { // 비정적 멤버 클래스
        ...
    }
}
```

**멤버 클래스에서 바깥 인스턴스에 접근할 일이 없다면 무조건 static을 붙여서 정적 멤버 클래스로 만들자.** static을 생략하면 바깥 인스턴스로의 숨은 외부 참조를 가지게 되서, 시간과 공간이 소비된다. 더 심각하게는 가비지 컬렉션이 바깥 클래스의 인스턴스를 수거하지 못하는 메모리 누수가 생길 수 있다.

### 익명 클래스

익명 클래스는 바깥 클래스의 멤버도 아니고, 멤버와 달리 쓰이는 시점에 선언과 동시에 인스턴스가 만들어진다. 코드의 어디서든 만들 수 있으며, 비정적인 문맥에서 사용될 때만 바깥 클래스의 인스턴스를 참조할 수 있다. 정적 문맥에서는 상수 표현을 위해 초기화된 final 기본 타입과 문자열 필드(상수 변수) 필드만 가질 수 있다. 익명 클래스는 선언한 지점에서만 인스턴스를 만들 수 있고, instanceof 검사나 클래스의 이름이 필요한 작업은 수핼할 수 없다. 여러 인터페이스를 구현할 수 없고, 인터페이스를 구현하는 동시에 다른 클래스를 상속할 수도 없다. 익명 클래스를 사용하는 클라이언트는 그 익명 클래스가 상위 타입에서 상속한 멤버 외에는 호출할 수 없다. 익명 클래스는 정적 팩터리 메서드를 구현할 때 주로 사용된다.

### 지역 클래스

지역 클래스는 지역변수를 선언할 수 있는 곳이면 실직적으로 어디서든 선언할 수 있고, 유효 범위도 지역변수와 같다. 멤버 클래스처럼 이름이 있고 반복해서 사용할 수 있고, 익명 클래스처럼 비정적 문맥에서 사용될 때만 바깥 인스턴스를 참조할 수 있으며, 정적 멤버는 가질수 없고, 가독성을 위해 짧게 작성해야 한다.



## Item 25. 톱레벨 클래스는 한 파일에 하나만 담으라

소스 파일 하나에 여러 톱레벨 클래스를 선언하면, 한 클래스를 여러가지로 정의할 수 있으며, 그중 어느 것을 사용할지는 어느 소스 파일을 먼저 컴파일하냐에 따라 달라진다.

```java
// Main 클래스는 다른 톱레벨 클래스 2개(Utensil과 Dessert)를 참조
public class Main {
    public static void main(String[] args) {
        System.out.println(Utensil.NAME + Dessert.NAME);
    }
}
```

```java
// Utensil.java
class Utensil {
    static final String NAME = "pan";
}

class Dessert {
    static final String NAME = "cake";
}
```

```java
// Dessert.java
class Utensil {
    static final String NAME = "pot";
}

class Dessert {
    static final String NAME = "pie";
}
```

- `javac Main.java Dessert.java` 명령으로 컴파일한다면 컴파일 오류가 나고 클래스를 중복 정의한 것을 알려줄 것이다.
- `javac Main.java`나 `javac Main.java Utensil.java` 명령으로 컴파일하면 Dessert.java 파일을 작성하기 전처럼 pancake를 출력한다.
- `javac Dessert.java Main.java` 명령으로 컴파일하면 potpie를 출력한다.

컴파일러에 어느 소스 파일을 먼저 건네느냐에 따라 동작이 달라지는 문제가 발생한다.

이를 해결하기 위해서는 톱레벨 클래스들을 서로 다른 소스 파일로 분리하거나 (굳이 여러 톱레벨 클래스를 한 파일에 담고 싶다면)정적 멤버 클래스를 사용해야 한다. 다른 클래스에 딸린 부차적인 클래스라면 정적 멤버 클래스로 만드는 쪽이 일반적으로 더 낫다. 가독성이 좋고, private으로 선언하면 접근 범위도 최소로 관리할 수 있기 때문이다.

```java
// 톱레벨 클래스들을 정적 멤버 클래스로 만든 방법
public class Test {
    public static void main(String[] args) {
        System.out.println(Utensil.NAME + Dessert.NAME);
    }
    
    private static class Utensil {
        static final String NAME = "pan";
    }
    
    private static class Dessert {
        static final String NAME = "cake";
    }
}
```
