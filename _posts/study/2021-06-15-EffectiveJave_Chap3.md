---
title: EffectiveJava_Chap3
excerpt: 이펙티브 자바 책 3장 정리
categories: study
---

# 3장. 모든 객체의 공통 메서드

## Item 10. equals는 일반 규약을 지켜 재정의하라

equals 메서드는 잘못 재정의 하면 문제가 발생할 수 있다.

### equals를 재정의하지 않는 것이 좋은 상황

- 각 인스턴스가 본질적으로 고유하다. (값을 표현하는게 아니라 동작하는 개체를 표현하는 클래스인 경우. ex. Thread)

- 인스턴스의 '논리적 동치성(logical equality)'을 검사할 일이 없다. (논리적 동치: 표현 방식은 다르지만 결국 같은 의미)

- 상위 클래스에서 재정의한 equals가 하위 클래스에도 딱 들어맞는다.

- 클래스가 private이거나 package-private이고 equals 메서드를 호출할 일이 없다.

  ```java
  @Override public boolean equals(Object o) {
    throw new AssertionError(); // 호출 금지! 실수로라도 호출되는 것을 막는 방법
  }
  ```

### equals를 재정의해야 하는 상황

객체 식별성(object identity; 두 객체가 물리적으로 같은가)이 아니라 논리적 동치성을 확인해야 하는데, 상위 클래스의 equals가 논리적 동치성을 비교하도록 재정의되지 않은 경우에 equals를 재정의해야 한다. ex. 값 클래스(Integer와 String처럼 값을 표현하는 클래스)

- equals가 논리적 동치성을 확인하도록 재정의해두면, 그 인스턴스는 Map의 키와 Set의 원소로 사용할 수 있게 된다.
- 값 클래스라 해도, 값이 같은 인스턴스가 둘 이상 만들어지지 않음이 보장되는 인스턴스 통제 클래스 이거나 Enum이라면 equals를 재정의하지 않아도 된다. (논리적으로 같은 인스턴스가 2개 이상 만들어지지 않아 논리적 동치성과 객체 식별성의 의미가 같아짐)

### equals 메서드 재정의 시 따라야하는 규약

> equals 메서드는 동치관계(equivalence relation)를 구현하며, 다음을 만족한다.
>
> - 반사성(reflexivity): null이 아닌 모든 참조 값 x에 대해, x.equals(x)는 true다.
> - 대칭성(symmetry): null이 아닌 모든 참조 값 x, y에 대해, x.equals(y)가 true면 y.equals(x) 도 true다.
> - 추이성(transitivity): null이 아닌 모든 참조 값 x, y, z에 대해, x.equals(y)가 true이고 y.equals(z)도 true면 x.equals(z)도 true다.
> - 일관성(consistency): null이 아닌 모든 참조 값 x, y에 대해, x.equals(y)를 반복해서 호출하면 항상 true를 반환하거나 항상 false를 반환한다.
> - null-아님: null이 아닌 모든 참조 값 x에 대해, x.equals(null)은 false다.

### equals 메서드 구현 방법

1. == 연산자를 사용해 입력이 자기 자신의 참조인지 확인한다.
2. instanceof 연산자로 입력이 올바른 타입인지 확인한다. (인터페이스를 구현한 클래스끼리 비교할 수 있도록 한다면, 클래스 대신 인터페이스로 확인)
3. 입력을 올바른 타입으로 형변환한다. (instance 검사를 했기 때문에 100% 성공)
4. 입력 객체와 자기 자신의 대응되는 '핵심' 필드들이 모두 일치하는지 하나씩 검사한다. (인터페이스를 사용해 인스턴스 타입을 확인했다면, 필드 값 비교시에도 인터페이스의 메서드를 사용)

### 각 필드의 비교 방법

- float와 double을 제외한 기본 타입 필드는 == 연산자로 비교
- 참조 타입 필드는 각각의 equals 메서드로 비교
- float와 double는 각각 정적 메서드인 Float.compare(float, float)와 Double.compare(double, double)로 비교 (Float.NaN, -0.0f, 특수한 부동소수 값 등을 다뤄야 하기 때문 / Float.equals와 Double.equals 메서드를 대신 사용할 수도 있지만, 오토박싱을 수반할 수 있어 성능상 좋지 않다.)
- 배열 필드는 원소 각각의 위의 방법들로 비교(배열의 모든 원소가 핵심 필드라면 Arrays.equals 메서드들 중 하나 사용)
  - array1.equals(array2)는 arrya1==array2와 같다. (두 배열이 같은 객체인지를 비교)
  - Arrays.equals(array1, array2)는 두 배열의 내용들이 같은지 비교
- null도 정상 값으로 취급하는 참조 타입 필드인 경우, 정적 메서드인 Object.equals(Object, Object)로 비교 (NullPointerException 발생 예방)
- 비교가 복잡한 필드는 그 필드의 표준형(canonical form)을 저장해둔 후 표준형끼리 비교(불변 클래스에 적합. 가변 객체는 값이 바뀔 때마다 표준형을 최신 상태로 갱신 필요)
- 성능을 고려해 다를 가능성이 더 크거나 비교 비용이 싼 필드를 먼저 비교
- 객체의 논리적 상태와 관련 없는 필드는 비교 x

### equals 메서드 예

equals를 구현할 때 대칭성, 추이성, 일관성을 잘 지켰는지 확인하자! 반사성과 null-아님이 문제되는 경우는 별로 없다.

```java
public final class PhoneNumber {
  private final short areaCode, prefix, lineNum;
  
  public PhoneNumber(int areaCode, int prefix, int lineNum) {
    this.areaCode = rangeCheck(areaCode, 999, "지역코드");
    this.prefix = rangeCheck(prifix, 999, "프리픽스");
    this.lineNum = rangeCheck(lineNum, 9999, "가입자 번호");
  }
  
  private static short rangeCheck(int val, int max, String arg) {
    if (val < 0 || val > max)
      throw new IllegalArgumentException(arg + ": " + val);
    return (short) val;
  }
  
  @Override public boolean equals(Object o) {
    if (o == this)
      return true;
    if (!(o instanceof PhoneNumber))
      return false;
    PhoneNumber pn = (PhoneNumber)o;
    return pn.lineNum == lineNum && pn.prefix == prefix && pn.areaCode == areaCode;
  }
  ... // 나머지 코드 생략
}
```

### equals 재정의 주의사항

- equals를 재정의할 때는 hashCode도 반드시 재정의하자.
- 필드들의 동치성만 검사해도 equals 규약을 지킬 수 있으니 너무 복잡하게 해결하려 들지 말자.
- Object 외의 타입을 매개변수로 받는 equals 메서드는 선언하지 말자. (Object.equals를 재정의한 것이 아니다! @Override를 사용해 실수 예방!)

\+ @AutoValue를 사용하면 equals와 hashCode와 같은 메서드들을 알아서 작성해준다.



## Item 11. equals를 재정의하려거든 hashCode도 재정의하라

equals를 재정의한 클래스 모두에서 hashCode도 재정의해야 한다.

> - equals 비교에 사용되는 정보가 변경되지 않았다면, 애플리케이션이 실행되는 동안 그 객체의 hashCode 메서드는 몇 번을 호출해도 일관되게 항상 같은 값을 반환해야 한다. 단, 애플리케이션을 다시 실행한다면 이 값이 달라져도 상관없다.
> - equals(Object)가 두 객체를 같다고 판단했더라도, 두 객체의 hashCode는 똑같은 값을 반환해야 한다. (논리적으로 같은 객체는 같은 해시코드를 반환해야 한다.)
> - equals(Object)가 두 객체를 다르다고 판단했더라도, 두 객체의 hashCode가 서로 다른 값을 반환할 필요는 없다. 단, 다른 객체에 대해서는 다른 값을 반환해야 해시테이블의 성능이 좋아진다.

### hashCode 작성 요령

1. int 변수 result를 선언한 수 값 c로 초기화한다. 이때 c는 해당 객체의 첫 번째 핵심 필드(equals 비교에 사용되는 필드)를 단계 2.1 방식으로 계산한 해시코드다.
2. 해당 객체의 나머지 핵심 필드 f 각각에 대해 다음 작업을 수행한다.
  1. 해당 필드의 해시코드 c를 계산한다.
    1. 기본 타입 필드라면, Type.hashCode(f)를 수행한다. 여기서 Type은 해당 기본 타입의 박싱 클래스다.
    2. 참조 타입 필드면서 이 클래스의 equals 메서드가 이 필드의 equals를 재귀적으로 호출해 비교한다면, 이 필드의 hashCode를 재귀적으로 호출한다. 계산이 더 복잡해질 것 같으면, 이 필드의 표준형을 만들어 그 표준형의 hashCode를 호출한다. 필드의 값이 null이면 0을 사용한다.
    3. 필드가 배열이라면, 핵심 원소 각각을 별도 필드처럼 다룬다. 이상의 규칙을 재귀적으로 적용해 각 핵심 원소의 해시코드를 계산한 다음, 단계 2.2의 방식으로 갱신한다. 배열에 핵심 원소가 하나도 없다면 단순히 상수(0 추천)를 사용한다. 모든 원소가 핵심 원소라면 Arrays.hashCode를 사용한다.
  2. 단계 2.1에서 계산한 해시코드 c로 result를 갱신한다. 코드로는 다음과 같다. `result = 31 * result + c;`
     (31은 홀수이면서 소수, `31 * i`는 `(i << 5) - i`와 같다.)
3. result를 반환한다.

- 파생 필드는 해시코드 계산에서 제외해도 된다. (다른 필드로부터 계산해낼 수 있는 필드는 무시)
- equals 비교에 사용되지 않은 필드는 '반드시' 제외해야 한다. (hashCode 규약 두 번째)

### hashCode 작성 예시

```java
@Override public int hashCode() { // 핵심 필드를 사용한 계산
  int result = Short.hashCode(areaCode);
  result = 31 * result + Short.hashCode(prefix);
  result = 31 * result + Short.hashCode(lineNum);
  return result;
}
```

```java
@Override public int hashCode() { // Objects 클래스의 hash 메서드 이용, 성능이 직접 계산보다 좋지 않다.
  return Objects.hash(lineNum, prefix, areaCode);
}
```

```java
private int hashCode; // 자동으로 0으로 초기화. 초기값은 흔히 생성되는 해시코드와는 달라야 한다.

@Override public int hashCode() {
  int result = hashCode;
  if (result == 0) { // hashCode가 처음 불릴 때 계산(지연 초기화 전략. thread safe하게 만들도록 주의)
    result = Short.hashCode(areaCode);
    result = 31 * result + Short.hashCode(prefix);
    result = 31 * result + Short.hashhCode(lineNum);
    hashCode = result;
  }
  return result;
}
```

- 성능을 높이겠다고 해시코드를 계산할 때 핵심 필드를 생략해서는 안 된다. (속도가 빨라지더라도 해시 품질이 나빠져 해시테이블의 성능을 심각하게 낮출 위험이 있다.)
- hashCode가 반환하는 값의 생성 규칙을 API 사용자에게 자세히 공표하지 말자. 그래야 클라이언트가 이 값에 의지하지 않게 되고, 추후에 계산 방식을 바꿀 수도 있다.



## Item 12. toString을 항상 재정의하라

Object의 기본 toString 메서드가 우리가 작성한 클래스에 적함한 문자열을 반환하는 경우는 거의 없다. `클래스_이름@16진수로_표시한_해시코드`를 반환할 뿐이다. toString의 일반 규약에 따르면 '간결하면서 사람이 읽기 쉬운 형태의 유익한 정보'를 반환해야 하고 '모든 하위 클래스에서 이 메서드를 재정의'해야 한다. toString을 잘 구현한 클래스는 사용하기 좋고, 그 클래스를 사용한 시스템은 디버깅하기 쉽다. toString 메서드는 객체를 println, printf, 문자열 연결 연산자(+), assert 구문에 넘길 때, 디버거가 객체를 출력할 때 자동으로 불린다.

### toString 구현시 주의사항

- toString은 그 객체가 가진 주요 정보 모두를 반환하는게 좋다. 이상적으로는 스스로를 완벽히 설명하는 문자열이어야 한다.
- toString을 구현할 때면 반환값의 포맷을 문서화할지 정해야한다. 포맷을 명시하면 그 객체는 표준적이고, 명확하고, 가독성이 좋아진다. 포맷을 명시한다면, 명시한 포맷에 맞는 문자열과 객체를 상호 전환할 수 있는 정적 팩터리나 생성자를 함께 제공해주면 좋다. 단, 포맷을 명시할 경우 향후 릴리스에서 정보를 더 넣거나 포맷을 개선할 수 있는 유연성을 잃게 된다.
- 포맷 명시 여부와 상관없이 toString이 반환한 값에 포함된 정보를 얻어올 수 있는 API를 제공하자. (+ 반환 값의 의도를 명확히 밝혀야 한다.)

\+ @AutoValue는 toString도 자동 생성해 준다. (하지만 전화번호와 같은 정보는 직접 생성해 주는 것이 좋다.)



## Item 13. clone 재정의는 주의해서 진행하라

Cloneable는 복제해도 되는 클래스임을 명시하는 용도의 믹스인 인터페이스지만, clone 메서드는 Cloneable이 아닌 Object에 protected로 선언되어있다. 그래서 Cloneable을 구현하는 것만으로는 외부 객체에서 clone 메서드를 호출할 수 없다. Cloneable을 구현한 클래스의 인스턴스에서 clone을 호출하면 그 객체의 필드들을 하나하나 복사한 객체를 반환하며, 그렇지 않은 클래스의 인스턴스에서 호출하면 CloneNotSupportedException을 던진다.

### clone 메서드의 일반 규약

> - 이 객체의 복사본을 생성해 반환한다. '복사'의 정확한 뜻은 그 객체를 구현한 클래스에 따라 다를 수 있다. 일반적인 의도는 다음과 같다. 어떤 객체 x에 대해 다음 식은 참이다.
    >   - `x.clone() != x`
>   - `x.clone.getClass() == x.getClass()`
> - 하지만 이상의 요구를 반드시 만족해야 하는 것은 아니다. 다음 식도 일반적으로 참이지만, 역시 필수는 아니다.
    >   - `x.clone().equals(x)`
> - 관례상, 이 메서드가 반환하는 객체는 super.clone을 호출해 얻어야 한다. 이 클래스와 (Object를 제외한) 모든 상위 클래스가 이 관례를 따른다면 다음 식은 참이다.
    >   - `x.clone().getClass() == x.getClass()`
> - 관례상, 반환된 객체와 원본 객체는 독립적이어야 한다. 이를 만족하려면 super.clone으로 얻은 객체의 필드 중 하나 이상을 반환 전에 수정해야 할 수도 있다.

### 불변 클래스의 clone 메서드 재정의

제대로 동작하는 clone 메서드를 가진 상위 클래스를 상속해 Cloneable을 구현한다고 가정했을 때 super.clone을 호출해 얻은 객체는 원본의 완벽한 복제본이다. 모든 필드가 기본 타입이거나 불변 객체를 참조한다면 더 이상 손볼 것이 없다. 하지만 쓸데없는 복사를 지양한다는 관점에서 보면 불변 클래스는 굳이 clone 메서드를 제공하지 않는 것이 좋다.

```java
// 이 메서드 동작을 위해 PhoneNumber 클래스 선언에 Cloneable을 구현한다고 써줘야한다.
@Override public PhoneNumber clone() {
  try {
    return (PhoneNumber) super.clone();
  } catch (CloneNotSupportedException e) {
    throw new AssertionError(); // 일어날 수 없는 일
  }
}
```

### 가변 클래스의 clone 메서드 재정의

Stack 클래스에서 clone 메서드가 단순히 super.clone의 결과를 그대로 반환한다면 Object 배열인 elements 필드가 원본 Stack 인스턴스와 똑같은 배열을 참조해 프로그램이 이상하게 동작하거나 NullPointerException이 발생할 수 있다. clone 메서드는 사실상 생성자와 같은 효과를 낸다. 즉, clone은 원본 객체에 아무런 해를 끼치지 않는 동시에 복제된 객체의 불변식을 보장해야한다.

```java
// clone을 재귀적으로 호출해 주는 방법 이용
@Override public Stack clone() {
  try {
    Stack result = (Stack) super.clone();
    result.elements = elements.clone(); // Object[]로 형변환할 필요 없다. (배열의 clone 메서드는 제대로 동작함!)
    return result;
  } catch (CloneNotSupportedException e) {
    throw new AssertionError();
  }
}
```

Cloneable 아키텍처는 '가변 객체를 참조하는 필드는 final로 선언하라'는 일반 용법과 충돌한다. 그래서 복제할 수 있는 클래스를 만들기 위해 일부 필드에서 final 한정자를 제거해야 할 수도 있다.

clone을 재귀적으로 호출 하는 것만으로 충분하지 않은 상황도 존재한다.  ex. 해시테이블용 clone 메서드

```java
public class HashTable implements Cloneable {
  private Entry[] buckets = ...; // 해시테이블 내부는 버킷들의 배열. 각 버킷은 키-값 쌍을 담는 연결 리스트의 첫 번째 엔트리 참조
  
  private static class Entry {
    final Object key;
    Object value;
    Entry next;
    
    Entry(Object key, Object, value, Entry next) {
      this.key = key;
      this.value = value;
      this.next = next;
    }
    
    // 이 엔트리가 가리키는 연결 리스트를 재귀적으로 복사
    Entry deepCopy() {
      return new Entry(key, value,
                       next == null ? null : next.deepCopy()); // 연결 리스트 전체 복사를 위해 재귀 호출
    }
  }
  
  @Override public HashTable clone() {
    try {
      HashTable result = (HashTable) super.clone();
      result.buckets = new Entry[buckets.length];
      for (int i = 0; i < buckets.length; i++) // 버킷 배열을 순회하며 비지 않은 각 버킷에 대해 깊은복사 수행
        if (buckets[i] != null)
          result.buckets[i] = buckets[i].deepCopy();
      return result;
    } catch (CloneNotSupportedException e) {
      throw new AssertionError();
    }
  }
  ...
}
```

위 방법은 재귀 호출 때문에 리스트의 원소 수 만큼 스택 프레임을 소비하여, 리스트가 길면 스택 오버플로를 일으킬 위험이 있다. 이 문제를 피하려면 deepCopy를 재귀 호출 대신 반복자를 써서 순회하는 방향으로 수정해야 한다.

```java
Entry deepCopy() {
  Entry result = new Entry(key, value, next);
  for (Entry p = result; p.next != null; p = p.next)
    p.next = new Entry(p.next.key, p.netx.value, p.next.next);
  return result;
}
```

마지막으로 복잡한 가변 객체를 복헤나는 방법은 super.clone을 호출하여 얻은 객체의 모든 필드를 초기 상태로 설정한 다음, 원본 객체의 상태를 다시 생성하는 고수준 메서드들을 호출하는 것이다. HashTable 예에서라면, buckets 필드를 새로운 버킷 배열로 초기화한 다음 원본 테이블에 담긴 모든 키-값 쌍 각각에 대해 복제본 테이블의 put(key, value) 메서드를 호출해 준다. 이처럼 고수준 API를 활용하면 간단해지지만, 저수준에서 바로 처리할 때보다는 느리고 Clonealbe 아키텍처의 기초가 되는 필드 단위 객체 복사와는 거리가 멀다.

### clone 메서드 재정의 주의사항

- 생성자와 마찬가지로 clone 메서드도 재정의될 수 있는 메서드를 호출해서는 안된다.

- Object의 clone 메서드는 CloneNotSupportedException을 던진다고 선언했지만 재정의한 public clone 메서드에서는 throws 절을 없애야 한다. (검사 예외를 던지지 않아야 그 메서드를 사용하기 편하다)

- 상속용 클래스는 Cloneable을 구현해서는 안된다. (구현한다면 다음과 같은 방법이 있다)

  - 제대로 작동하는 clone 메서드를 구현해 protected로 두고 CloneNotSupportedException을 던질 수 있다고 선언한다.

  - clone을 동작하지 않게 구현해놓고 하위 클래스에서 재정의하지 못하게 한다.

    ```java
    @Override
    protected final Object clone() thorws CloneNotSupportedException {
      throw new CloneNotSupportedException();
    }
    ```

- Cloneable을 구현하는 모든 클래스는 접근 제한자는 public으로 반환 타입은 클래스 자신인 clone을 재정의해야 한다.

- 불변 객체 참조만 갖는 클래스라면 clone 메서드에서 아무 필드도 수정할 필요가 없지만 일련번호나 고유 ID는 수정해줘야 한다.

### 복사 생성자와 복사 팩터리

Cloneable을 이미 구현한 클래스를 확장한다면 어쩔 수 없이 clone을 잘 작동하도록 구현해야 한다. 그렇지 않은 상황에서는 복사 생성자와 복사 팩터리라는 더 나은 객체 복사 방식을 제공할 수 있다.

```java
public Yum(Yum yum) { ... }; // 복사 생성자. 단순히 자신과 같은 클래스의 인스턴스를 인수로 받는 생성자
```

```java
public static Yum newInstance(Yum yum) { ... }; // 복사 팩터리. 복사 생성자를 모방한 정적 팩터리
```

복상 생성자와 복사 팩터리는 Cloneable/clone 방식보다 장점이 많다.

- 생성자를 쓰지 않는 방식을 사용하지 않는다.

- 엉성하게 문서화된 규약에 기대지 않는다.

- 정상적인 final 필드 용법과 충돌하지 않는다.

- 불필요한 검사 예외를 던지지 않는다.

- 형변환이 필요없다.

- 클라이언트가 원본의 구현 타입에 얽매이지 않고 복제본의 타입을 직접 선택할 수 있다.

  \+ 인터페이스 기반 복사 생성자와 복사 팩터리의 더 정확한 이름은 '변환 생성자'와 '변환 팩터리'다.
  ex. HashSet 객체 s를 TreeSet 타입으로 복제. new TreeSet<>(s)와 같이 처리.



## Item 14. Comparable을 구현할지 고려하라

Comparable 인터페이스에는 compareTo 메서드만 존재한다. compareTo는 Object의 메서드가 아니다. compareTo는 단순 동치성 비교와 더불어 순서까지 비교할 수 고, 제네릭하다. Comparable을 구현했다는 것은 그 클래스의 인스턴스들에는 자연적인 순서가 있음을 의미하고 `Arrays.sort(a);`을 사용해 정렬할 수 있다. 자바 플랫폼 라이브러리의 모든 값 클래스와 열거 타입은 Comparable을 구현했다. 알파벳, 숫자, 연대 같이 순서가 명확한 값 클래스를 작성한다면 반드시 Comparable 인터페이스를 구현하자.

```java
public interface Comparable<T> {
  int compareTo(T t);
} 
```

### compareTo 메서드의 일반 규약

> - 이 객체와 주어진 객체의 순서를 비교한다. 이 객체가 주어진 객체보다 작으면 음의 정수를, 같으면 0을, 크면 양의 정수를 반환한다. 이 객체와 비교할 수 없는 타입의 객체가 주어지면 ClassCastException을 던진다.
>
> - 다음 설명에서 sgn(표현식) 표기는 수학에서 말하는 부호 함수(signum function)를 뜻하며, 표현식의 값이 음수, 0, 양수일 때 -1, 0, 1을 반환하도록 정의했다.
    >   - Comparable을 구현한 클래스는 모든 x, y에 대해 `sgn(x.compareTo(y)) == -sgn(y.compareTo(x))`여야 한다(따라서 x.compareTo(y)는 y.compareTo(x)가 예외를 던질 때에 한해 예외를 던져야 한다).
>   - Comparable을 구현한 클래스는 추이성을 보장해야 한다. 즉, (`x.compareTo(y) > 0 && y.compareTo(z) > 0`)이면 `x.compareTo(z) > 0`이다.
>   - Comparable을 구현한 클래스는 모든 z에 대해 `x.compareTo(y) == 0`이면 `sgn(x.compareTo(z)) == sgn(y.compareTo(z) > 0)`이다.
>   - 이번 권고가 필수는 아니지만 꼭 지키는 게 좋다. `(x.compareTo(y) == 0) == (x.equals(y))`여야 한다. Comparable을 구현하고 이 권고를 지키지 않는 모든 클래스는 그 사실을 명시해야 한다. 다음과 같이 명시하면 적당할 것이다. `"주의: 이 클래스의 순서는 equals 메서드와 일관되지 않다."`

hashCode 규약을 지키지 못하면 해시를 사용하는 클래스와 어울리지 못하듯, compareTo 규약을 지키지 못하면 비교를 활용하는 클래스와 어울리지 못한다. 비교를 활용하는 클래스의 예로는 정렬된 컬렉션인 TreeSet과 TreeMap, 검색과 정렬 알고리즘을 활용하는 유틸리티 클래스인 Collections와 Arrays가 있다. compareTo 메서드로 수행하는 동치성 검사도 equals 규약과 같이 반사성, 대칭성, 추이성 등을 충족해야 한다. Comparable을 구현한 클래스를 화장해 값 컴포넌트를 추가하고 싶다면, 확장하는 대신 독립된 클래스를 만들어 이 클래스에 원래 클래스의 인스턴스를 가리키느 필드를 두고 내부 인스턴스를 반환하는 '뷰' 메서드를 제공하면 된다.

정렬된 컬렉션들은 동치성을 비교할 때 equals 대신 compareTo를 사용한다.

- HashSet은 BigDecimal("1.0")과 BigDecimal("1.00")을 equals로 비교해 두 개의 원소를 가짐
- TreeSet은 BigDecimal("1.0")과 BigDecimal("1.00")을 compareTo 메서드로 비교해 같은 인스턴스로 간주. 하나의 원소를 가짐

### compareTo 메서드 구현 예

```java
public final class CaseInsensitiveString implements Comparable<CaseInsensitiveString> {
    // CaseInsensitiveString 참조는 CaseInsensitiveString 참조와만 비교할 수 있다.
    public int compareTo(CaseInsensitiveString cis) {
        return String.CASE_INSENSITIVE_ORDER.compare(s, cis.s); // 관계 연산자 <와 >를 사용하는 대신 정적 메서드 compare를 이용하자!
    }
    ...
}
```

```java
// 클래스의 핵심 필드가 여러 개라면 가장 핵심적인 필드부터 비교해 나가자!
public int compareTo(PhoneNumber pn) {
    int result = Short.compare(areaCode, pn.areaCode); 		// 가장 중요한 필드
    if (result == 0) {
        result = Short.compare(prefix, pn.prefix);			// 두 번째로 중요한 필드
        if (result == 0)
            result = Short.compare(lineNum, pn.lineNum);	// 세 번째로 중요한 필드
    }
}
```

```java
// 비교자 생성 메서드를 활용하여 간결하게 구현가능하다. 단, 성능 저하가 뒤따른다.
private static final Comparator<PhoneNumber> COMPARATOR =
    comparingInt((PhoneNumber pn) -> pn.areaCode) // 인수의 타입이 PhoneNumber임을 명시
    	.thenComparingInt(pn -> pn.prefix)		 // 더 이상 인수의 타입을 명시하지 않음(자바의 타입 추론 능력)
    	.thenComparingInt(pn -> pn.lineNum);

public int compareTo(PhoneNumber pn) {
    return COMPARATOR.compare(this, pn);
}
```

- 위 코드는 클래스를 초기화할 때 비교자 생성 메서드 2개를 이용해 비교자를 생성한다.

  1. comparingInt는 객체 참조를 int 타입 키에 매핑하는 키 추출 함수를 인수로 받아, 그 키를 기준으로 순서를 정하는 비교자를 반환하는 정적 메서드다.

     comparingInt는 람다를 인수로 받으며, 이 람다는 PhoneNumber에서 추출한 지역 코드를 기준으로 전화번호의 순서를 정하는 Comparator\<PhoneNumber>를 반환한다. (이 람다에서 입력 인수의 타입(PhoneNumber pn)을 명시함)

  2. thenComparingInt는 Comparator의 인스턴스 메서드로, int 키 추출자 함수를 입력 받아 다시 비교자를 반환한다. (첫 번째 비교자를 적용한 다음 새로 추출한 키로 추가 비교를 수행) thenComparingInt는 원하는 만큼 연달아 호출이 가능하다.

### Comparator의 보조 생성 메서드

- 숫자 기본 타입용 비교자 생성 메서드
  - long과 double용으로는 comparingInt와 thenComparingInt의 변형 메서드가 존재한다.
  - short처럼 더 작은 정수 타입에는 int용 버전을 사용하면 된다.
  - float은 double용 메서드를 이용한다.
- 객체 참조용 비교자 생성 메서드
  - comparing 정적 메서드
    - 키 추출자를 받아서 그 키의 자연적 순서를 사용
    - 키 추출자 하나와 추출된 키를 비교할 비교자까지 2개의 인수를 받아 사용
  - thenComparing 인스턴스 메서드
    - 비교자 하나만 인수로 받아 그 비교자로 부차(2차, 3차, ...) 순서를 정함
    - 키 추출자를 인수로 받아 그 키의 자연적 순서로 보조 순서를 정함
    - 키 추출자 하나와 추출된 키를 비교할 비교자까지 총 2개의 친수를 받아 보조 순서를 정함

### compareTo 메서드 구현시 주의사항

이따금 '값의 차'를 기준으로 결과를 반환하는 compareTo나 compare 메서드를 볼 수 있다.

```java
static Comparator<Object> hashCodeOrder = new Comparator<>() { // 추이성을 위배한다! 사용해서는 안 된다!
  public int compare(Object o1, Object o2) {
      return o1.hashCode() - o2.hashCode(); // 정수 오버플로를 일으킬 수 있다. (IEEE 754 부동 소수점 계산 방식에 따른 오류를 낼 수도 있다.)
  }  
};
```

정적 compare 메서드나 Comparator 인터페이스가 제공하는 비교자 생성 메서드를 사용하자!

```java
// 정적 compare 메서드를 활용한 비교자
static Comparator<Object> hashCodeOrder = new Comparator<>() {
  public int Compare(Object o1, Object o2) {
      return Integer.compare(o1.hashCode(), o2.hashCode());
  }  
};
```

```java
// 비교자 생성 메서드를 활용한 비교자
static Comparator<Object> hashCodeOrder = Comparator.comparingInt(o -> o.hashCode());
```
