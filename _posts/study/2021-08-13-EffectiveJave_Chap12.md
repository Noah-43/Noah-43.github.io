---
title: EffectiveJava_Chap12
excerpt: 이펙티브 자바 책 12장 정리
categories: study
---

# 12장. 직렬화

## Item 85. 자바 직렬화의 대안을 찾으라

직렬화의 근본적인 문제는 공격 범위가 너무 넓고 지속적으로 더 넓어져 방어하기 어렵다는 점이다. ObjectInputStream의 readObject 메서드는 (Serializable 인터페이스를 구현했다면) 클래스패스 안의 거의 모든 타입의 객체를 만들어 낼 수 있다. 바이트 스트림을 역직렬화하는 과정에서 이 메서드는 그 타입들 안의 모든 코드를 수행할 수 있다. 즉, 그 타입들의 코드 전체가 공격 범위에 들어간다. 자바의 표준 라이브러리나 서드파티 라이브러리는 물론 애플리케이션 자신의 클래스들도 공격 범위에 포함된다.

역직렬화 과정에서 호출되어 잠재적으로 위험한 동작을 수행하는 메서드를 가젯(gadget)이라 부른다. 여러 가젯을 함께 사용하여 가젯 체인을 구성할 수도 있는데, 가끔씩 공격자가 기반 하드웨어의 네이티브 코드를 마음대로 실행할 수 있는 아주 강력한 가젯 체인도 발견되곤 한다. 그래서 아주 신중하게 제작한 바이트 스트림만 역직렬화해야 한다. 가젯까지 갈 것도 없이, 역직렬화에 시간이 오래 걸리는 짧은 스트림을 역직렬화하는 것만으로도 서비스 거부 공격에 쉽게 노출될 수 있다. 이런 스트림을 역직렬화 폭한(deserialization bomb)이라고 한다.

```java
// 역직렬화 폭탄 - 이 스트림의 역직렬화는 영원히 계속된다.
static byte[] bomb() {
  Set<Object> root = new HashSet<>();
  Set<Object> s1 = root;
  Set<Object> s2 = new HashSet<>();
  for (int i = 0; i < 100; i++) {
    Set<Object> t1 = new HashSet<>();
    Set<Object> t2 = new HashSet<>();
    t1.add("foo"); // t1을 t2와 다르게 만든다.
    s1.add(t1); s1.add(t2);
    s2.add(t1); s2.add(t2);
    s1 = t1;
    s2 = t2;
  }
  return serialize(root); // 간결하게 하기 위해 이 메서드의 코드는 생략함
}
```

HashSet 인스턴스를 역직렬화하려면 그 원소들의 해시코드를 계산해야 한다. 이 HashSet을 역직렬화하려면 hashCode 메서드를 2^100번 넘게 호출해야 한다. 역직렬화가 영원히 계속된다는 것도 문제지만, 무언가 잘못되었다는 신호조차 주지 않는다는 것도 큰 문제다.

**직렬화 위험을 회피하는 가장 좋은 방법은 아무것도 역직렬화하지 않는 것이다.** 객체와 바이트 시퀀스를 변환해주는 다른 메커니즘이 많이 있다. 이 방식들은 자바 직렬화의 여러 위험을 회피하면서 다양한 플랫폼 지원, 우수한 성능, 풍부한 지원 도구, 활발한 커뮤니티와 전문가 집단 등 수많은 이점까지 제공한다. 이들의 공통점은 자바 직렬화보다 훨씬 간단하고, 임의 객체 그래프를 자동으로 직렬화/역직렬화하지 않는다는 것이다. 대신 속성-값 쌍의 집합으로 구성된 간단하고 구조화된 데이터 객체를 사용하며, 기본 타입 몇 개와 배열 타입만 지원한다.

이러한 표현의 선두주자는 JSON과 프로토콜 버퍼다. JSON은 브라우저와 서버의 통신용으로, 프로토콜 버퍼는 구글이 서버 사이에 데이터를 교환하고 저장하기 위해 설계했다. 효율은 프로토콜 버퍼가 훨씬 좋지만 텍스트 기반 표현에는 JSON이 아주 효과적이다. 프로토콜 버터는 이진 표현뿐 아니라 사람이 읽을 수 있는 텍스트 표현(pbtxt)도 지원한다.

레거시 시스템 때문에 자바 직렬화를 완전히 배제할 수 없을 때의 차선책은 **신뢰할 수 없는 데이터는 절대 역직렬화하지 않는 것이다.** 특히, 신뢰할 수 없는 발신원으로부터의 RMI는 절대 수용해서는 안 된다. 직렬화를 피할 수 없고 역직렬화한 데이터가 안전한지 완전히 확신할 수 없다면 객체 역직렬화 필터링(`java.io.ObjectInputFilter`)을 사용하자. 객체 역직렬화 필터링은 데이터 스트림이 역직렬화되기 전에 필터를 설치하는 기능이다. 클래스 단위로, 특정 클래스를 받아들이거나 거부할 수 있는데, '기본 수용' 모드에서는 블랙리스트에 기록된 잠재적으로 위험한 클래스들을 거부하고, '기본 거부' 모드에서는 화이트리스트에 기록된 안전하다고 알려진 클래스들만 수용한다. **블랙리스트 방식보다는 화이트리스트 방식을 추천한다.** 필터링 기능은 메모리를 과하게 사용하거나 객체 그래프가 너무 깊어지는 사태로부터도 보호해주지만, 직렬화 폭탄은 걸러내지 못한다.



## Item 86. Serializable을 구현할지는 신중히 결정하라

어떤 클래스의 인스턴스를 직렬화할 수 있게 하려면 클래스 선언에 implements Serializable만 덧붙이면 된다. 직렬화를 지원하는 것은 이처럼 손쉬워 보이지만, 아주 복잡한 일이다.

### Serializable의 문제점

#### Serializable을 구현하면 릴리스한 뒤에는 수정하기 어렵다.

클래스가 Serializable을 구현하면 직렬화된 바이트 스트림 인코딩(직렬화 형태)도 하나의 공개 API가 된다. 기본 직렬화 형태에서는 클래스의 private와 package-private 인스턴스 필드들마저 API로 공개되는 꼴이 된다(캡슐화가 깨진다). 뒤늦게 클래스 내부 구현을 손보면 원래의 직렬화 형태와 달라지게 된다. 직렬화 형태를 잘 설계하더라도 클래스를 개선하는 데 제약이 될 수 있다. 그러니 직렬화 가능 클래스를 만들고자 한다면, 직렬화 형태도 주의해서 함께 설계해야 한다.

직렬화가 클래스 개선을 방해하는 대표적인 예로는 스트림 고유 식별자, 즉 직렬 버전 UID(serial version UID)이 있다. serialVersionUID라는 이름의 static final long 필드로, 이 번호를 명시하지 않으면 시스템이 런타임에 암호 해시 함수(SHA-1)를 적용해 자동으로 클래스 안에 생성해 넣는다. 편의 메서드를 추가하는 식으로 클래스의 멤버 중 하나라도 수정한다면 직렬 버전 UID 값도 변하는데, 자동 생성되는 값에 의존하면 쉽게 호환성이 깨져버려 런타임에 InvalidClassException이 발생할 수 있다.

#### Serializable을 구현하면 버그와 보안 구멍이 생길 위험이 높아진다.

직렬화는 언어의 기본 매커니즘을 우회하는 객체 생성 기법이다. 기본 방식을 따르든 재정의해 사용하든, 역직렬화는 일반 생성자의 문제가 그대로 적용되는 '숨은 생성자'다. 따라서 기본 역직렬화를 사용하면 불변식 깨짐과 허가되지 않은 접근에 쉽게 노출되게 된다.

#### Serializable을 구현하면 해당 클래스의 신버전을 릴리스할 때 테스트할 것이 늘어난다.

직렬화 가능 클래스가 수정되면 신버전 인스턴스를 직렬화한 후 구버전으로 역직렬화할 수 있는지, 그리고 그 반대도 가능한지를 검사해야 한다. 양방향 직렬화/역직렬화가 모두 성공하고, 원래의 객체를 충실히 복제해내는지를 반드시 확인해야 한다. 클래스를 처음 제작할 때 커스텀 직렬화 형태를 잘 설계해놨다면 이러한 부담을 줄일 수 있다.

### Serializable 구현 주의사항

#### Serializable 구현 여부는 가볍게 결정할 사안이 아니다.

단, 객체를 전송하거나 저장할 때 자바 직렬화를 이용하는 프레임워크용으로 만든 클래스, Serializable을 반드시 구현해야 하는 다른 클래스의 컴포넌트로 쓰일 클래스의 경우 선택의 여지가 없다. 하지만 구현 비용이 적지 않아, 클래스를 설계할 때 이득과 비용을 잘 고려해야 한다. 주로 BigInteger와 Instant 같은 '값' 클래스와 컬렉션 클래스들은 Serializable을 구현하고, 스레드 풀처럼 '동작'하는 객체를 표현하는 클래스들은 대부분 Serializable을 구현하지 않았다.

#### 상속용으로 설계된 클래스는 대부분 Serializable을 구현하면 안 되며, 인터페이스도 대부분 Serializable을 확장해서는 안 된다.

그렇지 않으면 그 클래스를 확장하거나 그 인터페이스를 구현하는 이에게 커다란 부담을 지우게 된다. 하지만 Serializable을 구현한 클래스만 지원하는 프레임워크를 사용하는 상황이라면 규칙을 어길수 밖에 없다. 상속용으로 설계된 클래스 중 Serializable을 구현한 예로는 Throwable과 Component가 있다. 클래스의 인스턴스 필드가 직렬화와 확장이 모두 가능하다면 주의할 점이 있다. 인스턴스 필드 값 중 불변식을 보장해야 할 게 있다면 반드시 하위 클래스에서 finalize 메서드를 재정의하지 못하게 해야 한다. 그리고 인스턴스 필드 중 기본값으로 초기화되면 위배되는 불변식이 있다면 클래스에 다음의 readObjectNoData 메서드를 반드시 추가해야 한다.

```java
// 상태가 있고, 확장 가능하고, 직렬화 가능한 클래스용 readObjectNoData 메서드
private void readObjectNoData() throws InvalidObjectException {
  throw new InvalidObjectException("스트림 데이터가 필요합니다");
}
```

상속용 클래스인데 직렬화를 지원하지 않으면 그 하위 클래스에서 직렬화를 지원하기 위해 그 상위 클래스에서 매개변수가 없는 생성자를 제공하거나, 하위 클래스에서 직렬화 프록시 패턴을 사용해야 한다.

#### 내부 클래스는 직렬화를 구현하지 말아야 한다.

내부 클래스에는 바깥 인스턴스의 참조와 유효 범위 안의 지역변수 값들을 저장하기 위해 컴파일러가 생성한 필드들이 자동으로 추가된다. 이 필드들이 클래스 정의에 어떻게 추가되는지는 언어 명세에 정의되지 않았다. 즉, 내부 클래스에 대한 기본 직렬화 형태는 분명하지가 않다. 단, 정적 멤버 클래스는 Serializable을 구현해도 된다.



## Item 87. 커스텀 직렬화 형태를 고려해보라

**먼저 고민해보고 괜찮다고 판단될 때만 기본 직렬화 형태를 사용하라.** 일반적으로 직접 설계하더라도 기본 직렬화 형태와 거의 같은 결과가 나올 경우에만 기본 형태를 써야 한다. 이상적인 직렬화 형태라면 물리적인 모습과 독립된 논리적인 모습만을 표현해야 한다. **객체의 물리적 표현과 논리적 내용이 같다면 기본 직렬화 형태라도 무방하다.**

```java
// 기본 직렬화 형태에 적합한 후보
public class Name implements Serializable {
  /**
   * 성. null이 아니어야 함.
   * @serial
   */
  private final String lastName;
  
  /**
   * 이름. null이 아니어야 함.
   * @serial
   */
  private final String firstName;
  
  /**
   * 중간이름. 중간이름이 없다면 null.
   * @serial
   */
  private final String middleName;
  
  ... // 나머지 코드 생략
}
```

성명은 논리적으로 이름, 성, 중간이름이라는 3개의 문자열로 구성되며, 위 코드의 인스턴스 필드들은 이 논리적 구성요소를 정확히 반영했다. 세 필드는 결국 클래스의 직렬화 형태에 포함되는 공개 API에 속하며 공개 API는 모두 문서화해야 하기 때문에 모두 private임에도 문서화 주석이 달려 있다. `@serial` 태그로 기술한 내용은 API 문서에서 직렬화 형태를 설명하는 특별한 페이지에 기록된다.

**기본 직렬화 형태가 적합하다고 결정했더라도 불변식 보장과 보안을 위해 readObject 메서드를 제공해야 할 때가 많다.** 앞의 Name 클래스의 경우에는 readObject 메서드가 lastName과 firstName 필드가 null이 아님을 보장해야 한다.

```java
// 기본 직렬화 형태에 적합하지 않은 클래스
public final class StringList implements Serializable {
  private int size = 0;
  private Entry head = null;
  
  private static class Entry implements Serializable {
    String data;
    Entry next;
    Entry previous;
  }
  
  ... // 나머지 코드는 생략
}
```

논리적으로 이 클래스는 일련의 문자열을 표현하지만 물리적으로는 문자열들을 이중 연결 리스트로 연결했다.

**객체의 물리적 표현과 논리적 표현의 차이가 클 때 기본 직렬화 형태를 사용하면 크게 네 가지 면에서 문제가 생긴다.**

1. **공개 API가 현재의 내부 표현 방식에 영구히 묶인다.**
2. **너무 많은 공간을 차지할 수 있다.**
3. **시간이 너무 많이 걸릴 수 있다.**
4. **스택 오버플로를 일으킬 수 있다.**

 ```java
 // 합리적인 커스텀 직렬화 형태를 갖춘 StringList
 public final class StringList implements Serializable {
   private transient int size = 0; 				// transient 한정자는 해당 인스턴스 필드가
   private transient Entry head = null;		// 기본 직렬화 형태에 포함되지 않는다는 표시
   
   // 이제는 직렬화되지 않는다.
   private static class Entry {
     String data;
     Entry next;
     Entry previous;
   }
   
   // 지정한 문자열을 이 리스트에 추가한다.
   public final void add(String s) { ... }
   
   /**
    * 이 {@code StringList} 인스턴스를 직렬화한다.
    *
    * @serialData 이 리스트의 크기(포함된 문자열의 개수)를 기록한 후
    * ({@code int}), 이어서 모든 원소를(각각은 {@code String})
    * 순서대로 기록한다.
    */
   private void writeObject(ObjectOutputStream s) throws IOException {
     s.defaultWriteObject();
     s.writeInt(size);
     
     // 모든 원소를 올바른 순서로 기록한다.
     for (Entry e = head; e != null; e = e.next)
       s.writeObject(e.data);
   }
   
   private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
     s.defaultReadObject();
     int numElements = s.readInt();
     
     // 모든 원소를 읽어 이 리스트에 삽입한다.
     for (int i = 0; i < numElements; i++)
       add((String) s.readObject());
   }
   
   ... // 나머지 코드는 생략
 }
 ```

클래스의 인스턴스 필드 모두가 transient 일지라도 defaultWriteObeject와 defaultReadObject를 호출하자. 이렇게 해야 향후 릴리스에서 transient가 아닌 인스턴스 필드가 추가되더라도 상호 호환된다. 메서드에 달린 `@serialData` 태그는 필드용의 `@serial` 태그처럼 자바독 유틸리티에게 이 내용을 직렬화 형태 페이지에 추가하도록 요청하는 역할을 한다.

defaultWriteObject 메서드를 호출하면 transient로 선언하지 않은 모든 인스턴스 필드가 직렬화된다. **해당 객체의 논리적 상태와 무관한 필드라고 확신할 때만 transient 한정자를 생략해야 한다.** 커스텀 직렬화 형태를 사용한다면 대부분의 (혹은 모든) 인스턴스 필드를 transient로 선언해야 한다.

기본 직렬화를 사용한다면 transient 필드들은 역직렬화될 때 기본값으로 초기화된다. 기본값을 그대로 사용해서는 안 된다면 readObject 메서드에서 defaultReadObject를 호출한 다음, 해당 필드를 원하는 값으로 복원하거나, 그 값을 처음 사용할 때 초기화하자.

기본 직렬화 사용 여부와 상관없이 **객체의 전체 상태를 읽는 메서드에 적용해야 하는 동기화 메커니즘을 직렬화에도 적용해야 한다.** 모든 메서드를 synchronized로 선언하여 스레드 안전하게 만든 객체에서 기본 직렬화를 사용하려면 writeObejct도 synchronized로 선언해야 한다.

```java
// 기본 직렬화를 사용하는 동기화된 클래스를 위한 writeObject 메서드
private synchronized void writeObject(ObjectOutputStream s) throws IOException {
  s.defaultWriteObject();
}
```

writeObject 메서드 안에서 동기화하고 싶다면 클래스의 다른 부분에서 사용하는 락 순서를 똑같이 따라야 한다. 그렇지 않으면 자원 순서 교착상태(resource-ordering deadlock)에 빠질 수 있다.

**어떤 직렬화 형태를 택하든 직렬화 가능 클래스 모두에 직렬 버전 UID를 명시적으로 부여하자.** 이는 직렬 버전 UID가 일으키는 잠재적인 호환성 문제를 사라지게 만들고, 성능도 향상시킨다.

```java
private static final long serialVersionUID = <무작위로 고른 long 값>;
```

클래스 일련 번호를 생성해주는 serialver 유틸리티를 사용해도 되며, 그냥 아무 값이나 선택해도 된다. 직렬 버전 UID가 꼭 고유할 필요는 없다. 단, 호환성 유지를 위해서는 구버전에서 사용한 자동 생성 값을 그대로 사용해야 한다. 이 값은 직렬화된 인스턴스가 존재하는 구버전 클래스를 serialver 유틸리티에 입력으로 주어 실행하면 얻을 수 있다. 기존 버전 클래스와의 호환성을 끊고 싶다면 단순히 직렬 버전 UID의 값을 바꿔주면 된다. **구버전으로 직렬화된 인스턴스들과의 호환성을 끊으려는 경우를 제외하고는 직렬 버전 UID를 절대 수정하지 말자.**



## Item 88. readObject 메서드는 방어적으로 작성하라

```java
// 방어적 복사를 사용하는 불변 클래스
public final class Period {
  private final Date start;
  private final Date end;
  
  /**
   * @param start 시작 시각
   * @param end 종료 시각; 시작 시각보다 뒤여야 한다.
   * @throws IllegalArgumentException 시작 시각이 종료 시각보다 늦을 때 발생한다.
   * @throws NullPointerException start나 end가 null이면 발생한다.
   */
  public Period(Date start, Date end) {
    this.start = new Date(start.getTime());
    this.end = new Date(end.getTime());
    if (this.start.compareTo(this.end) > 0)
      throw new IllegalArgumentException(start + "가 " + end + "보다 늦다.");
  }
  
  public Date start() { return new Date(start.getTime()); }
  public Date end() { return new Date(end.getTime()); }
  public String toString() { return start + " - " + end; }
  
  ... // 나머지 코드는 생략
}
```

Period 객체의 물리적 표현이 논리적 표현과 부합하므로 기본 직렬화 형태를 사용해도 나쁘지 않다. 하지만 이 클래스의 주요한 불변식을 더는 보장하지 못하게 된다. readObject 메서드가 실질적으로 또 다른 public 생성자이기 때문이다. readObject 메서드에서도 인수가 유효한지 검사해야 하고 필요하다면 매개변수를 방어적으로 복사해야 한다. readObject는 매개변수로 바이트 스트림을 받는 생성자라 할 수 있다. 보통의 경우 바이트 스트림은 정상적으로 생성된 인스턴스를 반들어지지만 불변식을 깨뜨릴 의도로 임의 생성한 바이트 스트림을 건네면 정상적인 생성자로는 만들어낼 수 없는 객체를 생성할 수 있다.

단순히 Period 클래스 선언에 implements Serializable만 추가한다면, 허용되지 않는 Period 인스턴스를 생성할 수 있다. Period를 직렬화할 수 있도록 선언한 것만으로 클래스의 불변식을 깨뜨리는 객체를 만들 수 있게 된 것이다. 이 문제를 고치려면 Period의 readObject 메서드가 defaultReadObject를 호출한 다음 역직렬화된 객체가 유효한지 검사해야 한다.

```java
// 유효성 검사를 수행하는 readObject 메서드 - 아직 부족하다!
private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
  s.defaultReadObject();
  
  // 불변식을 만족하는지 검사한다.
  if (start.compareTo(end) > 0)
    throw new InvalidObjectException(start + "가 " + end + "보다 늦다.");
}
```

위 작업으로 공격자가 허용되지 않는 Period 인스턴스를 생성하는 일을 막을 수 있지만, 정상 Period 인스턴스에서 시작도니 바이트 스트림 끝에 private Date 필드로의 참조를 추가하면 가변 Period 인스턴스를 만들어낼 수 있다. 공격자는 ObjectInputStream에서 Period 인스턴스를 읽은 후 스트림 끝에 추가된 이 '악의적인 객체 참조'를 읽어 Period 객체의 내부 정보를 얻을 수 있다. 이제 이 참조로 얻은 Date 인스턴스들을 수정할 수 있으니, Period 인스턴스는 더는 불변이 아니게 된다. 이처럼 변경할 수 있는 Period 인스턴스를 획득한 공격자는 이 인스턴스가 불변이라고 가정하는 클래스에 넘겨 보안 문제를 일으킬 수 있다.

이 문제의 근원은 Period의 readObject 메서드가 방어적 복사를 충분히 하지 않은 데 있다. **객체를 역직렬화할 때는 클라이언트가 소유해서는 안 되는 객체 참조를 갖는 필드를 모두 반드시 방어적으로 복사해야 한다.** 따라서 readObject에서는 불변 클래스 안의 모든 private 가변 요소를 방어적으로 복사해야 한다.

```java
// 방어적 복사와 유효성 검사를 수행하는 readObject 메서드
private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
  s.defaultReadObject();
  
  // 가변 요소들을 방어적으로 복사한다.
  start = new Date(start.getTime());
  end = new Date(end.getTime());
  
  // 불변식을 만족하는지 검사한다.
  if (start.compareTo(end) > 0)
    throw new InvalidObjectException(start + "가 " + end + "보다 늦다.");
}
```

방어적 복사를 유효성 검사보다 앞서 수행하며, Date의 clone 메서드는 사용하지 않았다. 두 조치 보두 Period를 공격으로부터 보호하는 데 필요하다. 또한 final 필드는 방어적 복사가 불가능하니 주의하자.

기본 readObject 메서드를 써도 좋을지를 판단하는 간단한 방법이 있다. transient 필드를 제외한 모든 필드의 값을 매개변수로 받아 유효성 검사 없이 필드에 대입하는 public 생성자를 추가할 수 없다면 커스텀 readObject 메서드를 만들어 모든 유효성 검사와 방어적 복사를 수행하거나, 직렬화 프록시 패턴을 사용해야 한다.

final이 아닌 직렬화 가능 클래스라면 생성자처럼 readObject 메서드도 재정의 가능 메서드를 호출해서는 안 된다. 이 규칙을 어겼는데 해당 메서드가 재정의되면, 하위 클래스의 상태가 완전히 역직렬화되기 전에 하위 클래스에서 재정의된 메서드가 실행된다.

#### 안전한 readObject 메서드를 작성하는 지침

- private이어야 하는 객체 참조 필드는 각 필드가 가리키는 객체를 방어적으로 복사하라. 불변 클래스 내의 가변 요소가 여기 속한다.
- 모든 불변식을 검사하여 어긋나는 게 발견되면 InvalidObjectException을 던진다. 방어적 복사 다음에는 반드시 불변식 검사가 뒤따라야 한다.
- 역직렬화 후 객체 그래프 전체의 유효성을 검사해야 한다면 ObjectInputValidation 인터페이스를 사용하라.
- 직접적이든 간접적이든, 재정의할 수 있는 메서드는 호출하지 말자.



## Item 89. 인스턴스 수를 통제해야 한다면 readResolve 보다는 열거 타입을 사용하라

```java
// 싱글턴 패턴
public class Elvis {
  public static final Elvis INSTANCE = new Elvis();
  private Elvis() { ... }
  
  public void leaveTheBuilding() { ... }
}
```

이 클래스는 그 선언에 implements Serializable을 추가하는 순간 더 이상 싱글턴이 아니게 된다. 기본 직렬화를 쓰지 않더라도 그리고 어떤 명시적인 readObject를 사용하더라도 이 클래스가 초기화 될 때 만들어진 인스턴스와는 별개인 인스턴스를 반환하게 된다.

역직렬화한 객체의 클래스가 readResolve 메서드를 적절히 정의해뒀다면, 역직렬화 후 새로 생성된 객체를 인수로 이 메서드가 호출되고, 이 메서드가 반환한 객체 참조가 새로 생성된 객체를 대신해 반환된다. 대부분의 경우 이때 새로 생성된 객체의 참조는 유지하지 않으므로 바로 가비지 컬렉션 대상이 된다.

```java
// 인스턴스 통제를 위한 readResolve - 개선의 여지가 있다!
private Object readResolve() {
  // 진짜 Elvis를 반환하고, 가짜 Elvis는 가비지 컬렉터에 맡긴다.
  return INSTANCE;
}
```

**사실, readResolve를 인스턴스 통제 목적으로 사용한다면 객체 참조 타입 인스턴스 필드는 모두 transient로 선언해야 한다.** 그렇지 않으면 readResolve 메서드가 수행되기 전에 역직렬화된 객체의 참조를 공격할 여지가 남는다. 싱글턴이 transient가 아닌 참조 필드를 가지고 있다면, 그 필드의 내용은 readResolve 메서드가 실행되기 전에 역직렬화된다. 그렇다면 잘 조작된 스트림을 써서 해당 참조 필드의 내용이 역직렬화되는 시점에 그 역직렬화된 인스턴스의 참조를 훔쳐올 수 있다.

```java
// 잘못된 싱글턴 - transient가 아닌 참조 필드를 가지고 있다!
public class Elvis implements Serializable {
  public static final Elvis INSTANCE = new Elvis();
  private Elvis() { }
  
  private String[] favoriteSongs = { "Hound Dog", "Heartbreak Hotel" }; // transient 아님.
  public void printFavorites() {
    System.out.println(Arrays.toString(favoriteSongs));
  }
  
  private Object readResolve() {
    return INSTANCE;
  }
}
```

```java
// readResolve 메서드와 인스턴스 필드 하나를 포함한 도둑 클래스
public class ElvisStealer implements Serializable {
  static Elvis impersonator;
  private Elvis payload;
  
  private Object readResolve() {
    // resolve되기 전의 Elvis 인스턴스의 참조를 저장한다.
    impersonator = payload;
    
    // favoriteSongs 필드에 맞는 타입의 객체를 반환한다.
    return new String[] { "A Fool Such as I" };
  }
  private static final long serialVersionUID = 0;
}
```

직렬화된 스트림에서 싱글턴의 비휘발성 필드를 이 도둑의 인스턴스로 교체해, 싱글턴과 도둑은 서로 참조하는 순환고리를 만들게 된다. 싱글턴이 도둑을 포함하므로 싱글턴이 역직렬화될 때 도둑의 readResolve 메서드가 먼저 호출되고, 도둑의 readResolve 메서드가 수행될 때 도둑의 인스턴스 필드에는 역직렬화 도중인 싱글턴의 참조가 담겨 있게 된다. 도둑의 readResolve 메서드는 이 인스턴스 필드가 참조한 값을 정적 필드로 복사하여 readResolve가 끝난 후에도 참조할 수 있도록 한다. 그런 다음 이 메서드는 도둑이 숨긴 transient가 아닌 필드의 원래 타입에 맞는 값을 반환한다. 이 과정을 생략하면 직렬화 시스템이 도둑의 참조를 이 필드에 저장하려 할 때 VM이 ClassCastException을 던진다.

favoriteSongs 필드를 transient로 선언하여 이 문제를 고칠 수 있지만 Elvis를 원소 하나짜리 열거 타입으로 바꾸는 편이 더 낫다.

```java
// 열거 타입 싱글턴 - 전통적인 싱글턴보다 우수하다.
public enum Elivs {
  INSTANCE;
  private String[] favoriteSongs = { "Hound Dog", "Heartbreak Hotl" };
  public void printFavorites() { 
    System.out.println(Arrays.toString(favoriteSongs)); 
  }
}
```

단, 직렬화 가능 인스턴스 통제 클래스를 작성해야 하는데, 컴파일타임에는 어떤 인스턴스들이 있는지 알 수 없는 상황이라면 연거 타입으로 표현하는 것이 불가능 해, readResolve를 사용해야 한다.

**readResolve 메서드의 접근성은 매우 중요하다.** final 클래스에서 readResolve 메서드는 private이어야 한다. final이 아닌 클래스에서는 하위 클래스를 잘 고려해야 한다.



## Item 90. 직렬화된 인스턴스 대신 직렬화 프록시 사용을 검토하라

Serializable을 구현하면 버그와 보안 문제가 일어날 가능성이 커진다. 직렬화 프록시 패턴(serialization proxy pattern)을 사용하면 이 위험을 크게 줄일 수 있다. 바깥 클래스의 논리적 상태를 정밀하게 표현하는 중첩 클래스를 설계해 private static으로 선언한다. 이 중첩 클래스가 바로 바깥 클래스의 직렬화 프록시다. 중첩 클래스의 생성자는 단 하나여야 하며, 바깥 클래스를 매개변수로 받아야 한다. 설계상, 직렬화 프록시의 기본 직렬화 형태는 바깐 클래스의 직렬화 형태로 쓰기에 이상적이다. 그리고 바깥 클래스와 직렬화 프록시 모두 Serializable을 구현한다고 선언해야 한다.

```java
// Period 클래스용 직렬화 프록시
private static class SerializationProxy implements Serializable {
  private final Date start;
  private final Date end;
  
  SerializationProxy(Period p) {
    this.start = p.start;
    this.end = p.end;
  }
  
  private static final long serialVersionUID = 234098243823485285L; // 아무 값 노 상관.
}
```

바깥 클래스에는 다음 메서드를 추가한다. 이 메서드는 범용적이니 직렬화 프록시를 사용하는 모든 클래스에 그대로 복사해 쓰면 된다.

```java
// 직렬화 프록시 패턴용 writeReplace 메서드
private Object writeReplace() {
  return new SerializationProxy(this);
}
```

이 메서드는 자바의 직렬화 시스템이 바깥 클래스의 인스턴스 대신 SerializationProxy의 인스턴스를 반환하게 하는 역할을 한다. 직렬화가 이뤄지기 전에 바깥 클래스의 인스턴스를 직렬화 프록시로 변환해 주는 것이다. 하지만 공격자는 불변식을 훼손하고자 바깥 클래스의 직렬화된 인스턴스를 생성하려는 시도를 해 볼 수 있는데, 다음의 readObject 메서드를 바깥 클래스에 추가하면 이 공격을 막아낼 수 있다.

```java
// 직렬화 프록시 패턴용 readObject 메서드
private void readObject(ObjectInputStream stream) throws InvalidObjectException {
  throw new InvalidObjectException("프록시가 필요합니다.");
}
```

바깥 클래스와 논리적으로 동일한 인스턴스를 반환하는 readResolve 메서드를 SerializationProxy 클래스에 추가하면 역직렬화 시에 직렬화 시스템이 직렬화 프록시를 다시 바깥 클래스의 인스턴스로 변환하게 해준다. readResolve 메서드는 공개된 API만을 사용해 바깥 클래스의 인스턴스를 생성한다. 즉, 일반 인스턴스를 만들 때와 똑같은 생성자, 정적 팩터리, 혹은 다른 메서드를 사용해 역직렬화된 인스턴스를 생성한다.

```java
// Period.SerializationProxy용 readResolve 메서드
private Object readResolve() {
  return new Period(start, end); // public 생성자를 사용한다.
}
```

방어적 복사처럼, 직렬화 프록시 패턴은 가짜 바이트 스트림 공격과 내부 필드 탈취 공격을 프록시 수준에서 차단해준다. Period의 필드를 final로 선언해도되므로 Period 클래스를 진정한 불변으로 만들 수도 있고, 어떤 필드가 직렬화 공격의 목표가 될지 고민하지 않아도 되며, 역직렬화 때 유효성 검사를 수행하지 않아도 된다. 그리고 직렬화 프록시 패턴은 역직렬화한 인스턴스와 원래의 직렬화된 인스턴스의 클래스가 달라도 정상 작동한다.

```java
// EnumSet의 직렬화 프록시
private static class SerializationProxy <E extends Enum<E>> implements Serializable {
  // 이 EnumSet의 원소 타입
  private final Class<E> elementType;
  
  // 이 EnumSet 안의 원소들
  private final Enum<?>[] elements;
  
  SerializationProxy(EnumSet<E> set) {
    elementType = set.elementType;
    elements = set.toArray(new Enum<?>[0]);
  }
  
  private Object readResolve() {
    EnumSet<E> result = EnumSet.noneOf(elementType);
    for (Enum<?> e : elements)
      result.add((E)e);
    return result;
  }
  
  private static final long serialVersionUID = 362491234563181265L;
}
```

하지만 직렬화 프록시 패턴에는 두 가지 한계가 있다.

1. 클라이언트가 멋대로 확장할 수 있는 클래스에는 적용할 수 없다.
2. 객체 그래프에 순환이 있는 클래스에도 적용할 수 없다. 이런 객체의 메서드를 직렬화 프록시의 readResolve 안에서 호출하려 하면 ClassCaseException이 발생할 것이다.
   (직렬화 프록시만 가졌을 뿐 실제 객체는 아직 만들어진 것이 아니기 때문)

마지막으로, 직렬화 프록시 패턴은 강력하고 안전하지만 방어적 복사보다 성능이 좋지 않다.
