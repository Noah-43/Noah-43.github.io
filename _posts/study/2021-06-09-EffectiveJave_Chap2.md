---
title: EffectiveJava_Chap2
excerpt: 이펙티브 자바 책 2장 정리
categories: study
---

# 2장. 객체 생성과 파괴

## Item 1. 생성자 대신 정적 팩터리 메서드를 고려하라

클래스는 생성자와 별도로 정적 팩터리 메서드(static factory method)를 제공할 수 있다. (그 클래스의 인스턴스를 반환하는 정적 메서드)

### 정적 팩터리 메서드가 생성자 보다 좋은 점

1. 이름을 가질 수 있다.

  - 이름을 지어 반환될 객체의 특성 묘사 가능하다.
    ex. 값이 소수인 BigInteger 반환 → BigInteger(int, int, Random) vs. BigInteger.probablePrime
  - 생성자는 하나의 시그니처로 하나만 생성 가능하고 각 생성자가 어떤 역할을 하는지 명확하게 알기 어렵다.

2. 호출될 때마다 인스턴스를 새로 생성하지 않다도 된다.

  - 불변 클래스는 인스턴스를 미리 만들어 놓거나 캐싱하여 재활용하는 식으로 불필요한 객체 생성을 피할 수 있다.
  - Boolean.valueOf(boolean) 메서드와 같이 객체를 생성하지 않을 수도 있다.
  - 인스턴스 통제를 통해 클래스를 싱글턴(singleton), 인스턴스화 불가(noninstantiable)하도록 만들 수 있다.

3. 반환 타입의 하위 타입 객체를 반환할 수 있다.

  - API를 만들 때 구현 클래스를 공개하지 않고도 객체를 반환할 수 있다. (`java.util.Collections`를 생각해보자)
  - 정적 팩터리 메서드를 사용하는 클라이언트는 얻은 객체를 (구현체가 아닌) 인터페이스만으로 다룰 수 있다.

4. 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.

  - 반환 타입의 하위 타입이기만 하면 어떤 클래스의 객체를 반환하든 상관없다.

    ex. `EnumSet` 클래스는 원소가 64개 이하면 RegularEnumSet 인스턴스를, 65개 이상이면 JumboEnumSet의 인스턴스를 반환

5. 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.

  - 이는 서비스 제공자 프레임워크(service provider framework)를 만드는 근간이 된다. ex. JDBC(Java Database Connectivity)

    > 서비스 제공자 프레임워크의 세 가지 핵심 컴포넌트
    >
    > - 서비스 인터페이스(service interface): 구현체의 동작 정의
    > - 제공자 등록 API(provider registration API): 제공자가 구현체를 등록 할 때 사용
    > - 서비스 접근 API(service access API): 클라이언트가 서비스의 인스턴스를 얻을 때 사용. '유연한 정적 팩터리'
    > - \+ 제공자 인터페이스(service provider interface): 서비스 인터페이스의 인스텉스를 생성하는 팩터리 객체를 설명

### 정적 팩터리 메서드의 단점

1. 상속을 하려면 public 이나 protected 생성자가 필요해 정적 팩터리 메서드만 제공하면 하위 클래스를 만들 수 없다.
2. 정적 팩터리 메서드는 프로그래머가 찾기 어렵다.
  - API 문서를 잘 써놓고 메서드 이름도 널리 알려진 규약을 따라 짓는 것이 좋다.



## Item 2. 생성자에 매개변수가 많다면 빌더를 고려하라

정적 팩터리와 생성자는 선택적 매개변수가 많을 때 적절히 대응하기가 어렵다.

### 선택적 매개변수가 많을 때 사용한 패턴

1. 점층적 생성자 패턴(telescoping constructor pattern): 필수 매개변수만 받는 생성자, 필수 매개변수와 선택 매개변수 1개를 받는 생성자, ...
  - 매개변수 개수가 많아지면 클라이언트 코드를 작성하거나 읽기 어려운 문제가 있다.
2. 자바빈즈 패턴(JavaBeans pattern): 매개변수가 없는 생성자로 객체를 만든 후, Setter를 이용해 원하는 매개변수 값을 설정
  - 객체 하나를 위해 여러 메서드를 호출하고, 객체의 일관성(consistency)가 무너지는 문제가 있다.
  - 클래스를 불변으로 만들 수 없다.
3. **빌더 패턴(Builder pattern)**: 필수 매개변수만으로 생성자를 호출해 빌더 객체를 얻고, 원하는 선택 매개변수들을 설정한뒤 build 메서드로 최종적인 객체를 얻어 옴. (생성자나 정적 팩터리가 처리해야 할 매개변수가 많다면 빌더 패턴을 선택!)
  - 빌더의 세터 메서드들은 빌더 자신을 반환하기 때문에 연쇄적으로 호출이 가능하다.
  - 빌더 패턴은 계층적으로 설계된 클래스와 함께 쓰기에 좋다. (추상 클래스는 추상 빌더를, 구체 클래스는 쿠체 필더를 갖도록 함)
  - 객체를 만들려면 빌더부터 만들어야 하는데, 빌더 생성 비용이 크지는 않지만 성능에 민감한 상황에서는 문제가 될 수 있다.
  - **점층적 생성자보다 클라이언트 코드를 읽고 쓰기 간결하고, 자바빈즈보다 훨씬 안전하다.**



## Item 3. private 생성자나 열거 타입으로 싱글턴임을 보증하라

> 싱글턴(singleton): 인스턴스를 오직 하나만 생성할 수 있는 클래스
>
> 클래스를 싱글턴으로 만들면 이를 사용하는 클라이언트를 테스트하기가 어려워질 수 있다.
>
> ex. 무상태(stateless) 객체, 시스템 컴포넌트

### 싱글턴을 만드는 방식

보통 생성자는 private 으로 감춰두고, 유일한 인스턴스에 접근할 수 있는 수단으로 public static 멤버를 하나 두는 방법을 사용한다.

1. public static 멤버가 final 필드인 방식

   ```java
   public class Elvis {
     public static final Elvis INSTANCE = new Elvis();
     private Elvis() { ... }
     
     public void leaveTheBuilding() { ... }
   }
   ```

  - private 생성자는 public static final 필드인 Elvis.INSTANCE를 초기화할 때 딱 한 번만 호출된다. (클래스가 초기화될 때 만들어진 인스턴스가 전체 시스템에 하나뿐임이 보장됨)
  - 예외적으로 권한이 있는 클라이언트의 공격(리플렉션 API를 사용한 생성자 호출)이 있을 수 있으니 생성자에서 두 번째 객체가 생성되려 할 때 예외를 던지게 하는 것이 좋다.
  - 해당 클래스가 싱글턴임이 API에 명백히 드러나고, 간결하다는 장점이 있다.

2. 정적 팩터리 메서드를 public static 멤버로 제공

   ```java
   public class Elvis {
     private static final Elvis INSTANCE = new Elvis();
     private Elvis() { ... }
     public static Elvis getInstance() { return INSTANCE; }
     
     public void leaveTheBuilding() { ... }
   }
   ```

  - Elvis.getInstance는 항상 같은 객체의 참조를 반환. (리플렉션을 통한 예외는 동일 적용)
  - 세 가지 장점이 있다. (이런 장점들이 굳이 필요하지 않다면 public 필드 방식이 좋다.)
    - API를 바꾸지 않고도 싱글턴이 아니게 변경할 수 있다. (ex. 팩터리 메서드가 호출하는 스레드별로 다른 인스턴스를 넘겨주게 할 수도 있다.)
    - 정적 팩터리를 제네릭 싱글턴 팩터리로 만들 수 있다.
    - 정적 팩터리의 메서드 참조를 공급자(supplier)로 사용할 수 있다. (ex. Elvis::getInstance를 Supplier\<Elvis>로 사용한다.)

> 위 두 가지 방식은 싱글턴 클래스를 직렬화하기 위해 모든 인스턴스 필드를 일시적(transient)이라고 선언하고 readResolve 메서드를 제공해야 한다. 그렇지 않으면 직렬화도니 인스턴스를 역직렬화할 때 마다 새로운 인스턴스가 만들어진다.
>
> ```java
> // 싱글턴임을 보장해주는 readResolve 메서드
> private Object readResolve() {
>   // '진짜' Elvis를 반환하고, 가짜 Elvis는 가비지 컬렉터에 맡긴다.
>   return INSTANCE;
> }
> ```

3. 원소가 하나인 열거 타입 선언

   ```java
   public enum Elvis {
     INSTANCE;
     
     public void leaveTheBuilding() { ... }
   }
   ```

  - public 필드 방식과 비슷하지만 더 간결하고 직렬화나 리플렉션 공격으로 인한 문제가 발생하지 않는다.
  - 부자연스러워 보일 수 있으나 **대부분의 상황에서 싱글턴을 만드는 가장 좋은 방법**이다.
  - 싱글턴이 Enum 외의 클래스를 상속해야 한다면 사용할 수 없다. (열거 타입이 다른 인터페이스를 구현하도록 선언할 수는 있다)



## Item 4. 인스턴스화를 막으려거는 private 생성자를 사용하라

### 정적 메서드와 정적 필드만을 담은 클래스

1. `java.lang.Math`와 `java.util.Arrays`: 기본 타입 값이나 배열 관련 메서드들을 모아 놓은 경우
2. `java.util.Collections`: 특정 인터페이스를 구현하는 객체를 생성해주는 정적 메서드(or 팩터리)를 모아 놓은 경우
3. final 클래스와 관련한 메서드들을 모아놓은 경우 (final 클래스를 상속해서 하위 클래스에 메서드를 넣는 것은 불가능)

### 유틸리티 클래스의 생성자

이러한 클래스는 인스턴스로 만들어 사용하지 않지만 생성자를 명시하지 않으면 컴파일러가 자동으로 기본 생성자를 만들어준다.

- 추상 클래스로 만드는 것은 인스턴스화를 막을 수 없다. (하위 클래스를 만들어 인스턴스화 할 수 있음)

- **private 생성자를 추가하면 클래스의 친스턴스화를 막을 수 있다.** (직관적이지 않으니 적절한 주석이 필요. 상속을 불가능하게 하는 효과도 있다.)

  ```java
  public class UtilityClass {
    // 기본 생성자가 만들어지는 것을 막는다(인스턴스화 방지용).
    private UtilityClass() {
      throw new AssertionError(); // 꼭 AssertionError일 필요는 없지만, 클래스 안에서 실수로라도 생성자를 호출하지 않도록 해 줌
    }
    ... // 나머지 코드 생략
  }
  ```



## Item 5. 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라

사용하는 자원에 따라 동작이 달라지는 클래스에는 정적 유틸리티 클래스나 싱글턴 방식이 적합하지 않다. 이 자원들을 클래스가 직접 만들게 해서도 안 된다. 대신에 **인스턴스를 생성할 때 생성자에 필요한 자원을 넘겨주는 방식**을 사용할 수 있다(의존 객체 주입의 한 형태). 의존 객체 주입은 클래스의 유연성, 재사용성, 테스트 용이성을 개선해준다.

```java
public class SpellChecker {
  private final Lexicon dictionary;
  
  public SpellChecker(Lexicon dictionary) { // 생성자에 필요한 자원을 넘겨줌
    this.dictionary = Objects.requireNonNull(dictionary);
  }
  
  public boolean isValid(String word) { ... }
  public List<String> suggestions(String typo) { ... }
}
```

- 위 예시는 dictionary라는 하나의 자원만 사용하지만, 자원이 몇 개든 의존 관계가 어떻든 상관없이 잘 작동한다.
- 불변을 보장하여 여러 클라이언트가 의존 객체들을 안심하고 공유할 수 있다.
- 의존 객체 주입은 생성자, 정적 팩터리, 빌더 모두에 똑같이 응용 가능하다.

생성자에 자원 팩터리를 넘겨주는 방식으로 변형해서 사용할 수 있다. 이 방식을 사용해 클라이언트는 자신이 명시한 타입의 하위 타입이라면 무엇이든 생성할 수 있는 팩터리를 넘길 수 있다.

```java
// 팩터리가 생성한 타일(Tile)들로 구성된 모자이크(Mosaic)를 만드는 메서드
Mosaic create(Supplier<? extends Tile> tileFactory) { ... }
```

의존성이 수 천 개나 되는 큰 프로젝트에서는 코드를 어지럽게 만들기도 하지만 대거(Dagger), 주스(Guice), 스프링(Spring) 같은 의존 객체 주입 프레임워크를 사용하면 이러한 문제를 해소할 수 있다.



## Item 6. 불필요한 객체 생성을 피하라

1. 똑같은 기능의 객체를 매번 생성하는 것보다 객체 하나를 재 사용하는 것이 좋을 때가 많다. 불변 객체는 언제든 재사용 가능하다.

   ```java
   String s = new String("bikini"); // "bikini" 자체가 생성자로 만들어내려는 String과 기능적으로 완전히 똑같다. (X)
   String s = "bikini"; // String 인스턴스를 재사용. 이와 똑같은 문자열 리터럴을 사용하는 모든 코드가 같은 객체 재사용. (O)
   ```

   생성자 대신 정적 팩터리 메서드를 제공하는 불변 클래스에서 정적 팩토리 메서드를 사용해 불필요한 객체 생성을 피할 수 있다. 생성자는 호출할 때마다 새로운 객체를 만들지만, 팩터리 메서드는 그렇지 않다. 가변 객체도 사용 중에 변경되지 않을 것임을 안다면 재사용할 수 있다.





2. 생성 비용이 비싼 객체는 캐싱하여 재사용하는 것이 좋다.

   ```java
   static boolean isRomanNumeral(String s) { // 주어진 문자열이 유효한 로마 숫자인지 확인하는 메서드
     return s.matches("^(?=.)M*(C[MD]|D?C{0,3})(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
   }
   ```

   `String.matches`는 정규표현식으로 문자열 형태를 확인하는 가장 쉬운 방법이지만, 성능이 중요한 상황에서의 반복 사용에 적합하지 않다. 이 메서드가 내부에서 만드는 정규표현식용 Pattern 인스턴스는 생성 비용이 높은 반면 한 번 쓰고 버려져서 곧바로 가비지 컬렉션 대상이 된다.

    ```java
   public class RomanNumerals {
     // Pattern 인스턴스를 클래스 초기화(정적 초기화) 과정에서 직접 생성해 캐싱
     private static final Pattern ROMAN 
       = Pattern.compile("^(?=.)M*(C[MD]|D?C{0,3})(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
   
     // 메서드 호출 시 인스턴스를 재사용
     static boolean isRomanNumeral(String s) {
       return ROMAN.matcher(s).matches();
     }
   }
    ```

3. 어댑터는 실제 작업은 뒷단 객체에 위임하고, 자신은 제2의 인터페이스 역할을 해주는 객체다. 어댑터는 뒷단 객체만 관리하면 되기 때문에 뒷단 객체 하나당 어댑터 하나씩만 만들어지면 충분하다. 여러 개를 만들 필요도 이득도 없다.

4. 오토박싱은 기본 타임과 그에 대응하는 박싱된 기본 타입의 구분을 흐려주지만, 완전히 없애주는 것은 아니다.

   ```java
   private static long sum() {
     Long sum = 0L;
     for (long i = 0; i <= Integer.MAX_VALUE; i++) 
       sum += i; // 불필요한 Long 인스턴스가 약 2^31개 만들어짐 (long 타입 i가 Long 타입 sum에 더해질 때마다.)
     
     return sum;
   }
   ```

   박싱된 기본 타입보다는 기본 타입을 사용하고, 의도치 않은 오토박싱이 숨어들지 않도록 주의하자.



## Item 7. 다 쓴 객체 참조를 해제하라

가비지 컬렉션 언어에서 객체 참조 하나를 살려두면 가비지 컬렉터는 그 객체뿐 아니라 그 객체가 참조하는 모든 객체를 회수해가지 못하기 때문에 메모리 누수가 발생하고 성능에 악영향을 줄 수 있다. 메모리 누수는 철저한 코드리뷰나 힙 프로파일러 같은 디버깅 도구를 동원해야만 발견되기도 하기 때문에 예방법을 익혀두는 것이 중요하다.

### 메모리 누수의 원인

1. Stack에서의 '활성 영역' 밖의 참조

   ```java
   public class Stack {
     private Object[] elements;
     private int size = 0;
     private static final int DEFAULT_INITIAL_CAPACITY = 16;
     
     public Stack() {
       elements = new Object[DEFAULT_INITIAL_CAPACITY];
     }
     
     public void push(Object e) {
       ensureCapacity();
       elements[size++] = e;
     }
     
     public Object pop() {
       if (size == 0)
         throw new EmptyStackException();
       Object result = elements[--size];
       element[size] = null; // 다 쓴 참조 해제. ('활성 영역' 밖의 참조) - 가비지 컬렉터가 회수해 가도록!
       return elements[--size];
     }
     
     private void ensureCapacity() {
       if (elements.length == size)
         elements = Arrays.copyOf(elements, 2 * size + 1);
     }
   }
   ```

2. 객체 참조를 캐시에 넣고 한참을 그냥 놔두는 상황은 메모리 누수를 일으킨다. 이를 해결하는 방법에는 여러가지가 있다.

  - 캐시 외부에서 키(key)를 참조하는 동안만 엔트리가 살아있는 캐시가 필요한 상황에는 WeakHashMap을 사용해 캐시를 만들면 다 쓴 엔트리는 그 즉시 자동으로 제거된다.
  - 캐시를 만들 때 시간이 지날수록 엔트리의 가치를 떨어뜨리고, 쓰지 않는 엔트리들을 백그라운드 스레드를 활용하거나 캐시에 새 엔트리를 추가할 때 부수 작업을 통해 제거한다.

3. 리스너(listner) 혹은 콜백(callback)도 메모리 누수의 원인이 된다. 클라이언트가 콜백을 등록만 하고 명확히 해지하지 않는다면 누수가 발생하는데 이를 막기위해 콜백을 약한 참조로 저장하면 가비지 컬렉터가 즉시 수거해간다. (ex. WeakHashMap에 키로 저장)



## Item 8. finalizer와 cleaner 사용을 피하라

자바는 두 가지 객체 소멸자를 제공한다. 그중 finalizer는 예측할 수 없고, 상황에 따라 위험할 수 있어 일반적으로 불필요하다. cleaner는 finalizer보다는 덜 위험하지만, 여전히 예측할 수 없고, 느리고, 일반적으로 불필요하다.

### 사용하지 말아야 할 이유

- finalizer와 cleaner는 즉시 수행된다는 보장이 없기 때문에 제때 실행되어야 하는 작업은 절대 할 수 없다.
- finalizer와 cleaner는 수행 시점뿐 아니라 수행 여부조차 보장되지 않기 때문에 상태를 영구적으로 수정하는 작업에서는 절대 이 두 소멸자에 의존해서는 안 된다.
- finalizer 동작 중 발생한 예외는 무시되며, 처리할 작업이 남았더라도 그 순간 종료된다. (경고조차 출력하지 않는다. 그나마 cleaner를 사용하는 라이브러리는 자신의 스레드를 통제하기 때문에 같은 문제가 발생하지 않는다.)
- finalizer와 cleaner는 심각한 성는 문제도 동반한다. (느려진다)
- finalizer를 사용한 클래스는 finalizer 공격에 노출되어 심각한 보안 문제를 일으킬 수도 있다. (final이 아닌 클래스를 finalizer 공격으로부터 방어하려면 아무 일도 하지 않는 finalizer 메서드를 만들고 final로 선언)

### finalizer와 cleaner의 용도

1. 자원의 소유자가 close 메서드를 호출하지 않는 것에 대비한 안전망 역할.
  - 즉시 (혹은 끝까지) 호출되리라는 보장은 없지만, 클라이언트가 하지 않은 자원 회수를 늦게라도 해줄 수 있다.
  - 자바 라이브러리의 일부 클래스는 안전망 역할의 finalizer를 제공한다. (ex. `FileInputStream`, `FileOutputStream`, `ThreadPoolExecutor`)
2. 네이티브 피어(일반 자가 객체가 네이티브 메서드를 통해 기능을 위임한 네이티브 객체)와 연결된 객체 회수.
  - 네이티브 피어는 자바 객체가 아니니 가비지 컬렉터가 네이티브 객체까지 회수하지 못한다.
  - 성능 저하를 감당할 수 없거나 네이티브 피어가 사용하는 자원을 즉시 회수해야 한다면 close 메서드를 사용해야 한다.



## Item 9. try-finally보다는 try-with-resources를 사용하라

자바 라이브러리에는 close 메서드를 호출해 직접 닫아줘야 하는 자원이 많다. (ex. `InputStream`, `OutputStream`, `java.sql.Connection`

전통적으로 자원이 제대로 닫힘을 보장하는 수단으로 try-finally가 쓰였다. 하지만 자원이 둘 이상이면 너무 지저분해진다. 그리고 try-finally 문은 try 블록과 finally 블록 모두에서 예외가 발생할 수 있는데, 기기의 물리적 문제가 생긴다면 try 블록에서 예외를 던지고 같은 이유로 finally의 close 메서드도 실패할 것이다. 이런 상황에서는 첫 번째 예외에 관한 정보가 남지 않아 시스템 디버깅이 어려워지는 문제가 발생한다.

try-with-resources 구조를 사용하면 이러한 문제를 해결할 수 있다. 이 구조를 사용하려면 해당 자원이 AutoCloseable 인터페이스(단순히 void를 반환하는 close 메서드 하나를 정의해 둔 인터페이스)를 구현해야한다. 자바 라이브러리와 서드파티 라이브러리들의 수많은 클래스와 인터페이스들은 AutoCloseable을 구현하거나 확장해뒀다.

```java
// 복수의 자원을 처리하는 try-with-resources. 짧고 가독성이 좋다.
static void copy(String src, String dst) throws IOException {
  try (InputStream in = new FileInputStream(src);
       OutputStream out = new FileOutputStream(dst)) {
    byte[] buf = new byte[BUFFER_SIZE];
    int n;
    while ((n = in.read(buf)) >= 0)
      out.write(buf, 0, n);
  }
}
```

보통의 try-finally에서처럼 try-with-resources에서도 catch 절을 쓸 수 있다. catch절 덕분에 try 문을 중첩하지 않고도 다수의 예외를 처리할 수 있다.

```java
static String firstLineOfFile(String path, String defaultVal) {
  try (BufferedReader br = new BufferedReader(new FileReader(path))) {
    return br.readLine();
  } catch (IOException e) {
    return defaultVal; // 파일을 열거나 데이터를 읽지 못했을 때 예외를 던지는 대신 기본값을 반환
  }
}
```

꼭 회수해야 하는 자원을 다룰 때는 try-finally 말고, try-with-resources를 사용하자. 코드가 더 짧고 분명해지고, 만들어지는 예외 정보도 훨씬 유용해진다. (정확하고 쉽게 자원 회수가 가능하다.)


###### 참고 도서 : Effective JAVA 3/E (Joshua Bloch 지음, 이복연 옮김)
