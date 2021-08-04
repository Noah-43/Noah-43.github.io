---
title: EffectiveJava_Chap11
excerpt: 이펙티브 자바 책 11장 정리
categories: study
---

# 11장. 동시성

## Item 78. 공유 중인 가변 데이터는 동기화해 사용하라

synchronized 키워드는 해당 메서드나 블록을 한번에 한 스레드씩 수행하도록 보장한다. 동기화는 배타적 실행, 즉 한 스레드가 변경하는 중이라서 상태가 일관되지 않은 순간의 다른 스레드가 보지 못하게 막는 용도 외에 중요한 기능이 또 있다. 동기화는 일관성이 깨진 상태를 볼 수 없게 하는 것은 물론, 동기화된 메서드나 블록에 들어간 스레드가 같은 락의 보호하에 수행된 모든 이전 수정의 최종 결과를 보게 해준다.

자바 언어 명세는 스레드가 필드를 읽을 때 항상 '수정이 완전히 반영된' 값을 얻는다고 보장하지만, 한 스레드가 저장한 값이 다른 스레드에게 '보이는가'는 보장하지 않는다. **동기화는 배타적 실행뿐 아니라 스레드 사이의 안정적인 통신에 꼭 필요하다.** 예를 들어 다른 스레드를 멈추는 올바른 방법은 다음과 같다(**Thread.stop은 사용하지 말자!**). 첫 번째 스레드는 자신의 boolean 필드를 폴링하면서 그 값이 true 가 되면 멈춘다. 이 필드를 false로 초기화해놓고, 다른 스레드에서 이 스레드를 멈추고자 할 때 true로 변경하는 식이다.

```java
// 잘못된 코드 - 동기화하지 않아 무한히 수행될 수 있다.
public class StopThread {
  private static boolean stopRequested;
  
  public static void main(String[] args) throws InterruptedException {
    Thread backgroundThread = new Thread(() -> {
      int i = 0;
      while (!stopRequested) i++;
    });
    backgroundThread.start();
    
    TimeUnit.SECONDS.sleep(1);
    stopRequested = true;
  }
}
```

동기화가 빠지면 가상머신이 다음과 같은 최적화를 수행할 수도 있는 것이다.

```java
// 원래 코드
while (!stopRequested) i++;

// 최적화한 코드
if (!stopRequested)
  while (true) i++;
```

이 결과 프로그램은 응답 불가 상태가 되어 더 이상 진전이 없다. stopRequested 필드를 동기화해 접근하면 이 문제를 해결할 수 있다.

```java
// 적절히 동기화해 스레드가 정상 종료한다.
public class StopThread {
  private static boolean stopRequested;
  
  private static synchronized void requestStop() { // 쓰기 메서드 동기화
    stopRequested = true;
  }
  
  private static synchronized boolean stopRequested() { // 읽기 메서드 동기화
    return stomRequested;
  }
  
  public static void main(String[] args) throws InterruptedException {
    Thread backgroundThread = new Thread(() -> {
      int i = 0;
      while (!stopRequested()) i++;
    });
    backgroundThread.start();
    
    TimeUnit.SECONDS.sleep(1);
    requestStop();
  }
}
```

**쓰기와 읽기 모두가 동기화되지 않으면 동작을 보장하지 않는다.**  사실 이 두 메서드는 단순해서 동기화 없이도 원자적으로 동작하지만, 이 코드에서는 통신 목적으로만 사용된 것이다.

volatile 한정자는 배타적 수행과는 상관없지만 항상 가장 최근에 기록된 값을 읽게 됨을 보장한다.

```java
// volatile 필드를 사용해 스레드가 정상 종료
public class StopThread {
  private static volatile boolean stopRequested; // volatile으로 선언하면 동기화를 생략해도 된다.
  
  public static void main(String[] args) throws InterruptedException {
    Thread backgroundThread = new Thread(() -> {
      int i = 0;
      while (!stopRequested) i++;
    });
    backgroundThread.start();
    
    TimeUnit.SECONDS.sleep(1);
    stopRequested = true;
  }
}
```

volatile은 주의해서 사용해야 한다. 다음은 일련번호를 생성할 의도로 작성한 메서드다.

```java
// 잘못된 코드 - 동기화가 필요하다!
private static volatile int nextSerialNumber = 0;

public static int generateSerialNumber() {
  return nextSerialNumber++; // 증가 연산자(++)가 실제로는 nextSerialNumber 필드에 두 번 접근해 문제 발생. 
}
```

프로그램이 잘못된 결과를 계산해내는 이런 오류를 안전 실패(safety failure)라고 한다. generateSerialNumber 메서드에 synchronized 한정자를 붙이면 이 문제가 해결된다. 동시에 호출해도 서로 간섭하지 않으며 이전 호출이 변경한 값을 읽게 된다. 메서드에 synchronized를 붙였다면 volatile은 제거해야 한다.

`java.util.concurrent.atomic` 패키지에는 락 없이도 스레드 안전한 프로그래밍을 지원하는 클래스들이 담겨 있다. volatile은 동기화의 두 효과 중 통신 쪽만 지원하지만 이 패키지는 원자성까지 지원한다. 성능도 동기화 버전보다 우수하다.

```java
// java.util.concurrent.atomic을 이용한 락-프리 동기화
private static final AtomicLong nextSerialNum = new AtomicLong();

public static long generateSerialNumber() {
  return nextSerialNum.getAndIncrement();
}
```

사실 이런 문제들을 피하는 가장 좋은 방법은 가변 데이터를 공유하지 않는 것이다. 불변 데이터만 공유하거나 아무것도 공유하지 말자. **가변 데이터는 단일 스레드에서만 쓰도록 하자.**

한 스레드가 데이터를 다 수정한 후 다른 스레드에 공유할 때는 해당 객체에서 공유하는 부분만 동기화해도 된다. 그러면 그 객체를 다시 수정할 일이 생기기 전까지 다른 스레드들은 동기화 없이 자유롭게 값을 읽어갈 수 있다. 이런 객체를 사실상 불변(effectively immutable)이라 하고 다른 스레드에 이런 객체를 건네는 행위를 안전 발행(safe publication)이라 한다. 객체를 안전하게 발행하는 방법에는 클래스 초기화 과정에서 객체를 정적필드, volatile 필드, final 필드, 혹은 보통의 락을 통해 접근 하는 필드에 저장하는 방법, 동시성 컬렉션에 저장하는 방법 등이 있다.



## Item 79. 과도한 동기화는 피하라

**응답 불가와 안전 실패를 피하려면 동기화 메서드나 동기화 블록 안에서는 제어를 절대로 클라이언트에 양도하면 안 된다.** 동기화된 영역 안에서는 재정의할 수 있는 메서드는 호출하면 안 되며, 클라이언트가 넘겨준 함수 객체를 호출해서도 안 된다. 외계인 메서드가 하는 일에 따라 동기화된 영역은 예외를 일으키거나, 교착상태에 빠지거나, 데이터를 훼손할 수도 있다.

```java
// 잘못된 코드. 동기화 블록 안에서 외계인 메서드를 호출한다.
public class ObservableSet<E> extends ForwardingSet<E> {
  public ObservableSet(Set<E> set) { super(set); }
  
  private final List<SetObserver<E>> observers = new ArrayList<>();
  
  public void addObserver(SetObserver<E> observer) {
    synchronized(observers) {
      observers.add(observer);
    }
  }
  
  public boolean removeObserver(SetObserver<E> observer) {
    synchronized(observers) {
      return observers.remove(observer);
    }
  }
  
  private void nofityElementAdded(E element) {
    synchronized(observers) {
      for (SetObserver<E> observer : observers)
        observer.added(this, element);
    }
  }
  
  @Override public boolean add(E element) {
    boolean added = super.add(element);
    if (added)
      notifyElementAdded(element);
    return added;
  }
  
  @Override public boolean addAll(Collection<? extends E> c) {
    boolean result = false;
    for (E element : c)
      result |= add(element); // notifyElementAdded를 호출한다.
    return result;
  }
}
```

관찰자들은 addObserver와 removeObserver 메서드를 호출해 구독을 신청하거나 해지한다. 두 경우 모두 다음 콜백 인터페이스의 인스턴스를 메서드에 건넨다.

```java
@FunctionalInterface public interface SetObserver<E> {
  // ObservableSet에 원소가 더해지면 호출된다.
  void added(ObservableSet<E> set, E element);
}
```

다음 프로그램은 0부터 99까지를 출력한다.

```java
public static void main(String[] args) {
  ObservableSet<Integer> set = new ObservableSet<>(new HashSet<>());
  
  set.addObserver((s, e) -> System.out.println(e));
  
  for (int i = 0; i < 100; i++)
    set.add(i);
}
```

값이 23이면 자기 자신을 제거하는 관찰자를 추가해보자.

```java
set.addObserver(new SetObserver<>() {
  public void added(ObservableSet<Integer> s, Integer e) {
    System.out.println(e);
    if (e == 23)
      s.removeObserver(this);
  }
});
```

이 프로그램은 0부터 23까지 출력한 후 관찰자 자신을 제거하고 종료할 것 같지만, 23까지 출력한 다음 CuncurrentModificationException을 던진다. 관찰자의 added 메서드 호출이 일어난 시점이 notifyElementAdded가 관찰자들의 리스트를 순회하는 도중이기 때문이다. nofityElementAdded 메서드에서 수행하는 순회는 동기화 블록 안에 있으므로 동시수정이 일어나지 않도록 보장하지만, 정작 자신이 콜백을 거쳐 되돌아와 수정하는 것까지 막지는 못한다.

```java
// 쓸데없이 백그라운드 스레드를 사용하는 관찰자
set.addObserver(new SetObserver<> () {
  public void added(ObservableSet<Integer> s, Integer e) {
    System.out.println(e);
    if (e == 23) {
      ExecutorService exec = Executors.newSingleThreadExecutor(); // 실행자 서비스를 사용해 다른 스레드에게 구독해지를 부탁함.
      try {
        exec.submit(() -> s.removeObserver(this)).get();
      } catch (ExecutionException | InterruptedException ex) {
        throw new AssertionError(ex);
      } finally {
        exec.shutdown();
      }
    }
  }
});
```

이 프로그램을 실행하면 예외는 나지 않지만 교착상태에 빠진다. 백그라운드 스레드가 s.removeObserver를 호출하면 관찰자를 잠그려 시도하지만 메인 스레드가 이미 락을 쥐고 있기 때문에 락을 얻을 수 없다. 그와 동시에 메인 스레드는 백그라운드 스레드가 관찰자를 제거하기만을 기다린다. 실제 시스템(특히 GUI 툴킷)에서도 동기화된 영역 안에서 외계인 메서드를 호출하여 교착상태에 빠지는 사례가 자주 있다.

앞선 두 경우와 똑같은 상황이지만 불변식이 임시로 깨진 경우를 생각해보자. 자바 언어의 락은 재진입을 허용하므로 교착상태에 빠지지는 않는다. 예외를 발생시킨 첫 번째 예에서는 외계인 메서드를 호출하는 스레드는 이미 락을 쥐고 있으므로 다음번 락 획득도 성공한다. 이렇게 락이 제 구실을 하지 못해 문제가 발생한다. 재진입 가능 락은 객체 지향 멀티스레드 프로그램을 쉽게 구현할 수 있도록 해주지만, 응답 불가(교착상태)가 될 상황을 안전 실패(데이터 훼손)로 변모시킬 수도 있다. 이런 문제는 외계인 메서드 호출을 동기화 블록 바깥으로 옮기면 해결된다.

```java
// 외계인 메서드를 동기화 블록 바깥으로 옮긴다.
private void notifyElementAdded(E element) {
  List<SetObserver<E>> snapshot = null;
  synchronized(observers) {
    snapshot = new ArrayList<>(observers); // 관찰자 리스트를 복사해 사용
  }
  for (SetObserver<E> observer : snapshot)
    observer.added(this, element);
}
```

자바의 동시성 컬렉션 라이브러리의 CopyOnWriteArrayList는 정확히 위 문제를 해결할 목적으로 특별히 설계된 것이다. ArrayList를 구현한 클래스로, 내부를 변경하는 작업은 항상 깨끗한 복사본을 만들어 수행하도록 구현했다. 내부의 배열은 절대 수정되지 않으니 순회할 때 락이 필요 없어 매우 빠르다.

```java
// CopyOnWriteArrayList를 사용해 구현한 스레드 안전하고 관찰 가능한 집합
private final List<SetObserver<E>> observers = new CopyOnWriteArrayList<>();

public void addObserver(SetObserver<E> observer) {
  observers.add(observer);
}

public boolean removeObserver(Setobserver<E> observer) {
  return observers.remove(observer);
}

private void notifyElementAdded(E element) {
  for (SetObserver<E> observer : observers)
    observer.added(this, element);
}
```

명시적으로 동기화한 곳이 사라졌다는 것에 주목하자!

동기화 영역 바깥에서 호출되는 외계인 메서드를 열린 호출(open call)이라 한다. 열린 호출은 실패 방지 효과 외에도 동시성 효율을 크게 개선해준다. **기본 규칙은 동기화 영역에서는 가능한 한 일을 적게 하는 것이다.**

과도한 동기화가 초래하는 진짜 비용은 락을 얻는 데 드는 CPU 시간이 아닌, 경쟁하느라 낭비하는 시간, 즉 병렬로 실행할 기회를 잃고, 모든 코어가 메모리를 일관되게 보기 위한 지연시간이 진짜 비용이다. 가상머신의 코드 최적화를 제한한다는 점도 과도한 동기화의 또 다른 숨은 비용이다.

가변 클래스를 작성할 때는 동기화를 전혀 하지 말고 그 클래스를 동시에 사용해야 하는 클래스가 외부에서 알아서 동기화하게 하거나 동기화를 내부에서 수행해 스레드 안전한 클래스로 만들자. 단, 클라이언트가 외부에서 객체 전체에 락을 거는 것보다 동시성을 월등히 개선할 수 있을 때만 두 번째 방법을 선택해야 한다. `java.util` 은 첫 번째 방식을 취했고, `java, util.concurrent` 는 두 번째 방식을 취했다.

StringBuffer 인스턴스는 거의 항상 단일 스레드였음에도 내부적으로 동기화를 수행했다. 뒤늦게 StringBuilder가 등장한 이유이기도 하다. 비슷한 이유로 스레드 안전한 의사 난수 발생기인 `java.util.Random` 은 동기화하지 않는 버전인 `java.util.concurrent.ThreadLocalRandom` 으로 대체되었다. 선택하기 어렵다면 동기화하지 말고, 문서에 "스레드 안전하지 않다"고 명기하자.

클래스를 내부에서 동기화하기로 했다면, 락 분할(lock splitting), 락 스트라이핑(lock striping), 비차단 동시성 제어(nonblocking concurrency control) 등 다양한 기법을 동원해 동시성을 높여줄 수 있다.

여러 스레드가 호출할 가능성이 있는 메서드가 정적 필드를 수정한다면 그 필드를 사용하기 전에 반드시 동기해야 한다. 그런데 클라이언트가 여러 스레드로 복제돼 구동되는 상황이라면 다른 클라이언트에서 이 메서드를 호출하는 걸 막을 수 없어 외부에서 동기화할 방법이 없다. 결과적으로, 이 정적 필드가 심지어 private이라도 서로 관련 없는 스레드들이 동시에 읽고 수정할 수 있게 되고, 사실상 전역 변수와 같아진다.



## Item 80. 스레드보다는 실행자, 태스크, 스트림을 애용하라

`java.util.concurrent` 패키지는 실행자 프레임워크라고 하는 인터페이스 기반의 유연한 태스크 실행 기능을 담고 있다. 이를 이용해 작업 큐를 다음의 단 한 줄로 생성할 수 있다.

```java
ExcutorService exec = Executors.newSingleThreadExecutro();
```

다음은 이 실행자에 실행할 태스크를 넘기는 방법이다.

```java
exec.execute(runnable);
```

다음은 실행자를 종료시키는 방법이다(이 작업이 실패하면 VM 자체가 종료되지 않을 것이다).

```java
exec.shutdown();
```

실행자 서비스의 주요 기능은 다음과 같다.

- 특정 태스크가 완료되기를 기다린다.
- 태스크 모음 중 아무것 하나(invokeAny 메서드) 혹은 모든 태스크(invokeAll 메서드)가 완료되기를 기다린다.
- 실행자 서비스가 종료하기를 기다린다(awaitTermination 메서드).
- 완료된 태스크들의 결과를 차례로 받는다(ExecutorCompletionService 이용).
- 태스크를 특정 시간에 혹은 주기적으로 실행하게 한다(ScheduledThreadPoolExecutor 이용).

큐를 둘 이상의 스레드가 처리하게 하고 싶다면 간단히 다른 정적 팩터리를 이용하여 다른 종류의 실행자 서비스(스레드 풀)를 생성하면 된다. 스레드 풀의 스레드 개수는 고정할 수도 있고 필요에 따라 늘어나거나 줄어들게 설정할 수 있다. `java.util.concurrent.Executors` 의 정적 팩터리를 이용하면 실행자 대부분을 생성할 수 있다. ThreadPoolExecutor 클래스를 사용하면 스레드 풀 동작을 결정하는 거의 모든 속성을 설정할 수도 있다.

작은 프로그램이나 가벼운 서버 등의 일반적인 용도에는 `Executors.newCachedThreadPool` 을 사용하면 된다. 하지만 CachedThreadPool에서는 요청받은 태스크들이 큐에 쌓이지 않고 즉시 스레드에 위임돼 실행되기 때문에 무거운 프로덕션 서버에는 좋지 않다. CPU 이용률일 100%로 치닫고, 새로운 태스크가 도착하는 족족 다른 스레드를 생성하는 문제가 생길 수 있다. `Executors.newFixedThreadPool` 을 선택하거나 완전히 통제할 수 있는 ThreadPoolExecutor를 직접 사용하는 편이 낫다.

스레드를 직접 다루면 Thread가 작업 단위와 수행 메커니즘 역할을 모두 수행하게 된다. 반면 실행자 프레임워크에서는 작업 단위와 실행 메커니즘이 분리된다. 작업 단위를 나나태는 핵심 추상 개념이 태스크다. 태스크는 Runnable과 Callable(값을 반환하고 임의의 예외를 던질 수 있다)이 있다. 태스크를 수행하는 일반적인 메커니즘이 실행자 서비스다. 태스크 수행을 실행자 서비스에 맡기면 원하는 태스크 수행 정책을 선택할 수 있고, 언제든 변경할 수 있다. 핵심은 실행자 프레임워크가 작업 수행을 담당해준다는 것이다.

자바 7부터 실행자 프레임워크는 포크-조인 태스크를 지원하도록 확장되었는데, 이 태스크는 포크-조인 풀이라는 특별한 실행자 서비스가 실행해준다. ForkJoinTask의 인스턴스는 작은 하위 태스크로 나뉠 수 있고, ForkJoinPool을 구성하는 스레드들이 이 태스크들을 처리하며, 일을 먼저 끝낸 스레드는 다른 스레드의 남은 태스크를 가져와 대신 처리할 수도 있다. 이는 CPU를 최대한 활용하면서 높은 처리량과 낮은 지연시간을 달성하게 해준다. 포크-조인 태스크를 직접 작성하고 튜닝하기는 어렵지만, 포크-조인 풀을 이용해 만든 병렬 스트림을 이용하면 이점을 얻을 수 있다. 단, 포크-조인에 적합한 형태의 작업이어야 한다.



## Item 81. wait와 notify보다는 동시성 유틸리티를 애용하라

자바 5에서 도입된 고수준의 동시성 유틸리티가 wait와 notify로 하드코딩해야 했던 전형적인 일들을 대신 처리해준다. **wait와 notify는 올바르게 사용하기가 아주 까다로우니 고수준 동시성 유틸리티를 사용하자.** `java.util.concurrent`의 고수준 유틸리티는 세 범주로 나눌 수 있다. 바로 실행자 프레임워크, 동시성 컬렉션(concurrent colledtion), 동기화 장치(synchronizer)다.

### 동시성 컬렉션

동시성 컬렉션은 List, Queue, Map 같은 표준 컬렉션 인터페이스에 동시성을 가미해 구현한 고성능 컬렉션이다. 높은 동시성에 도달하기 위해 동기화를 각자의 내부에서 수행한다. 따라서 **동시성 컬렉션에서 동시성을 무력화하는 건 불가능하며, 외부에서 락을 추가로 사용하면 오히려 속도가 느려진다.** 동시성 컬렉션에는 여러 기본 동작을 하나의 원자적 동작으로 묶는 '상태 의존적 수정' 메서드들이 추가되었다. 예를 들어 Map의 `putIfAbsent(key, value)` 메서드는 주어진 키에 매핑된 값이 아직 없을 때만 새 값을 집어넣는데, 기존 값이 있었다면 그 값을 반환하고, 없었다면 null을 반환한다. 이 메서드 덕에 스레드 안전한 정규화 맵을 쉽게 구현할 수 있다.

```java
// ConcurrentMap으로 구현한 동시성 정규화 맵 - 최적은 아니다.
private static final ConcurrentMap<String, String> map = new ConcurrentHashMap<>();

public static String intern(String s) {
  String previousValue = map.putIfAbsent(s, s);
  return previousValue = null ? s : previousValue;
}
```

ConcurrentHashMap은 get과 같은 검색 기능에 최적화되었다. get을 먼저 호출하여 필요할 때만 putIfAbsent를 호출하면 더 빠르다.

```java
// ConcurrentMap으로 구현한 동시성 정규화 맵 - 더 빠르다!
public static String intern(String s) {
  String result = map.get(s);
  if (result == null) {
    result = map.putIfAbsent(s, s);
    if (result == null)
      result = s;
  }
  return result;
}
```

ConcurrentHashMap은 동시성이 뛰어나며 속도도 무척 빠르다. 동시성 컬렉션은 동기화한 컬렉션을 낡은 유산으로 만들었다. 대표적인 예로, **Collections.synchronizedMap 보다는 ConcurrentHashMap을 사용하는게 훨씬 좋다.**

컬렉션 인터페이스 중 일부는 작업이 성공적으로 완료될 때까지 기다리도록 확장되었다. Queue를 확장한 BlockingQueue에 추가된 메서드 중 take는 큐의 첫 원소를 꺼내고, 이때 만약 큐가 비었다면 새로운 원소가 추가될 때까지 기다린다. 이런 특성 덕에 작업 큐(생산자-소비자 큐)로 쓰기에 적합하다. ThreadPoolExcutor를 포함한 대부분의 실행자 서비스 구현체에서 이 BlockingQueue를 사용한다.

### 동기화 장치

동기화 장치는 스레드가 다른 스레드를 기다릴 수 있게 하여, 서로 작업을 조율할 수 있게 해준다. 가장 자주 쓰이는 동기화 장치는 CountDownLatch와 Semaphore다. CyclicBarrier와 Exchanger는 그보다 덜 쓰이며, 가장 강력한 동기화 장치는 바로 Phaser다.

CountDownLatch는 일회성 장벽으로, 하나 이상의 스레드가 또 다른 하나 이상의 스레드 작업이 끝날 때까지 기다리게 한다. CountDownLatch의 유일한 생성자는 int 값을 받으며, 이 값이 래치의 countDown 메서드를 몇 번 호출해야 대기 중인 스레드들을 깨우는지를 결정한다. 이 간단한 장치를 활용하면 유용한 기능들을 쉽게 구현할 수 있다.

```java
// 동시 실행 시간을 재는 간단한 프레임워크
public static long time(Executor executor, int concurrency, Runnable action) throws InterruptedException {
  CountDownLatch ready = new CountDownLatch(concurrency);
  CountDownLatch start = new CountDownLatch(1);
  CountDownLatch done = new CountDownLatch(concurrency);
  
  for (int i = 0; i < concurrency; i++) {
    excutor.execute(() -> {
      // 타이머에게 준비를 마쳤음을 알린다.
      ready.countDown();
      try {
        // 모든 작업자 스레드가 준비될 때까지 기다린다.
        start.await();
        action.run();
      } catch (InterruptedException e) {
        Thread.currentThread().interrupt();
      } finally {
        // 타이머에게 작업을 마쳤음을 알린다.
        done.countDown();
      }
    });
  }
  
  ready.await(); // 모든 작업자가 준비될 때까지 기다린다.
  long startNanos = System.nanoTime();
  start.countDown(); // 작업자들을 깨운다.
  done.await(); // 모든 작업자가 일을 끝마치기를 기다린다.
  return System.nanoTime() - startNonos;
}
```

ready 래치는 작업자 스레드들이 준비가 완료됐음을 타이머 스레드에 통지할 때 사용한다. 통지를 끝낸 작업자 스레드들은 두 번재 래치인 start가 열리기를 기다린다. 마지막 작업자 스레드가 ready.countDown을 호출하면 타이머 스레드가 시작 시각을 기록하고 start.countDown을 호출하여 기다리던 작업자 스레드들을 깨운다. 그 직후 타이머 스레드는 세 번째 래치인 done이 열리기를 기다린다. done 래치는 마지막 남은 작업자 스레드가 동작을 마치고 done.countDown을 호출하면 열린다.

time 메서드에 넘겨진 실행자는 concurrency 매개변수로 지정한 동시성 수준만큼의 스레드를 생성할 수 있다. 그렇지 못하면 스레드 기아 교착 상태(thread starvation deadlock)이 발생할 것이다. InterruptedException을 캐치한 작업자 스레드는 `Thread.currentThread().Interrupt()` 관용구를 사용해 인터럽를 되살리고 자신은 run 메서드에서 빠져나온다. **시간 간격을 잴 때는 상항 System.currentTimeMillis가 아닌 System.nanoTime을 사용하자.** System.nanoTime은 더 정확하고 정밀하며 시스템의 실시간 시계의 시간 보정에 영향받지 않는다.

앞의 예에서 사용한 카운트다운 래치 3개는 CyclicBarrier(혹은 Phaser) 인스턴스 하나로 대체할 수 있다. 이러면 코드가 더 명료해지겠지만 이해하기는 더 어려워 진다.

### wait와 notify

wait 메서드는 스레드가 어떤 조건이 충족되기를 기다리게 할 때 사용한다. 락 객체의 wait 메서드는 반드시 그 객체를 잠근 동기화 영역 안에서 호출해야 한다.

```java
// wait 메서드를 사용하는 표준 방식
synchronized (obj) {
  while (<조건이 충족되지 않았다>)
    obj.wait(); // (락을 놓고, 깨어나면 다시 잡는다.)
  
  ... // 조건이 충족됐을 때의 동작을 수행한다.
}
```

**wait 메서드를 사용할 때는 반드시 대기 반복문(wait loop) 관용구를 사용하라. 반복문 밖에서는 절대로 호출하지 말자.** 이 반복문은 wait 호출 전후로 조건이 만족하는지를 검사하는 역할을 한다.

대기 전에 조건을 검사하여 조건이 이미 충족되었다면 wait를 건너뛰게 한 것은 응답 불가 상태를 예방하는 조치이고, 대기 후에 조건을 검사하여 조건이 충족되지 않았다면 다시 대기하게 하는 것은 안전 실패를 막는 조치다. 조건이 만족되지 않아도 스레드가 깨어날 수 있는 상황이 몇 가지 있다.

- 스레드가 notify를 호출한 다음 대기 중이던 스레드가 깨어나는 사이에 다른 스레드가 락을 얻어 그 락이 보호하는 상태를 변경한다.
- 조건이 만족되지 않았음에도 다른 스레드가 실수로 혹은 악의적으로 notify를 호출한다. 공개된 객체를 락으로 사용해 대기하는 클래스는 이런 위험에 노출된다. 외부에 노출된 객체의 동기화된 메서드 안에서 호출하는 wait는 모두 이 문제에 영향을 받는다.
- 깨우는 스레드는 지나치게 관대해서, 대기 중인 스레드 중 일부만 조건이 충족되어도 notifyAll을 호출해 모든 스레드를 깨울 수도 있다.
- 대기 중인 스레드가 nofity 없이도 깨어나는 경우가 있다. 허위 각성(spurious wakeup)이라는 현상이다.

notify와 notifyAll 중 무엇을 선택하느냐 하는 문제도 있다. 일반적으로 nofityAll을 사용하는게 합리적이고 안전하다. 깨어나야 하는 모든 스레드가 깨어남을 보장하니 항상 정확한 결과를 얻을 것이다. 모든 스레드가 같은 조건을 기다리고, 조건이 한 번 충족될 때마다 단 하나의 스레드만 혜택을 받을 수 있다면 nofityAll 대신 notify를 사용해 최적화할 수 있다. 하지만 외부로 공개된 객체에 대해 실수로 혹은 악의적으로 notify를 호출하는 상황에 대비하기 위해 wait를 반복문 안에서 호출했듯, notify 대신 notifyAll을 사용하면 관련 없는 스레드가 실수로 혹은 악의적으로 wait를 호출하는 공격으로부터 보호할 수 있기 때문에 notifyAll을 사용하는 것이 좋다.



## Item 82. 스레드 안전성 수준을 문서화하라

한 메서드를 여러 스레드가 동시에 호출할 때 그 메서드가 어떻게 동작하느냐는 해당 클래스와 이를 사용하는 클라이언트 사이의 중요한 계약과 같다. API 문서에 아무런 언급이 없으면 사용자는 나름의 가정을 해야하고, 그 가정이 틀리면 심각한 오류로 이어질 수 있다.

**메서드 선언에 synchronized 한정자를 선언할지는 구현 이슈일 뿐 API에 속하지 않는다.** 따라서 이것만으로는 그 메서드가 스레드 안전하다고 믿기 어렵다. **멀티스레드 환경에서도 API를 안전하게 사용하게 하려면 클래스가 지원하는 스레드 안전성 수준을 정확히 명시해야 한다.** 다음은 일반적인 경우에서 스레드 안전성이 높은 순으로 나열한 것이다.

- **불변**(immutable): 이 클래스의 인스턴스는 마치 상수와 같아서 외부 동기화도 필요 없다. String, Long, BigInteger가 대표적이다.
- **무조건적 스레드 안전**(unconditionally thread-safe): 이 클래스의 인스턴스는 수정될 수 있으나, 내부에서 충실히 동기화하여 별도의 외부 동기화 없이 동시에 사용해도 안전하다. AtomicLong, ConcurrentHashMap이 여기에 속한다.
- **조건부 스레드 안전**(conditionally thread-safe): 무조건적 스레드 안전과 같으나, 일부 메서드는 동시에 사용하려면 외부 동기화가 필요하다. Collections.synchronized 래퍼 메서드가 반환한 컬렉션들이 여기 속한다(이 컬렉션들이 반환한 반복자는 외부에서 동기화해야 한다).
- **스레드 안전하지 않음**(not thread-safe): 이 클래스의 인스턴스는 수정될 수 있다. 동시에 사용하려면 각각의(혹은 일련의) 메서드 호출을 클라이언트가 선택한 외부 동기화 메커니즘으로 감싸야 한다. ArrayList, HashMap 같은 기본 컬렉션이 여기 속한다.
- **스레드 적대적**(thread-hostile): 이 클래스는 모든 메서드 호출을 외부 동기화로 감싸더라도 멀티스레드 환경에서 안전하지 않다. 이 수준의 클래스는 일반적으로 정적 데이터를 아무 동기화 없이 수정한다. 동시성을 고려하지 않고 작성하다 보면 우연히 만들어질 수 있다. 스레드 적대적으로 밝혀진 클래스나 메서드는 일반적으로 문제를 고쳐 재배포하거나 사용 자제(deprecated) API로 지정한다.

이 분류는 스레드 안전성 애너테이션(`@Immutable`, `@ThreadSafe`, `@NotThreadSafe`)과 대략 일치한다. 무조건적 스레드 안전과 조건부 스레드 안전은 모두 `@ThreadSafe` 애너테이션 밑에 속한다.

조건부 스레드 안전한 클래스는 주의해서 문서화해야 한다. 어떤 순서로 호출할 때 외부 동기화가 필요한지, 그리고 그 순서로 호출하려면 어떤 락 혹은(드물게) 락들을 얻어야 하는지 알려줘야 한다. 일반적으로 인스턴스 자체를 락으로 얻지만 예외도 있다.

> synchronizedMap이 반환한 맵의 컬렉션 뷰를 순회하려면 반드시 그 맵을 락으로 사용해 수동으로 동기화하라.
>
> Map<K, V> m = Collections.synchronizedMap(new HashMap<>());
> Set\<K> s = m.keySet(); // 동기화 블록 밖에 있어도 된다.
> 	...
> synchronized(m) { // s가 아닌 m을 사용해 동기화해야 한다!
> 	for (K key : s)
> 		key.f();
> }
>
> 이대로 따르지 않으면 동작을 예측할 수 없다.

클래스의 스레드 안전성은 보통 클래스의 문서화 주석에 기재하지만, 독특한 특성의 메서드라면 해당 메서드의 주석에 기재하도록 하자. 열거 타입은 굳이 불변이라고 쓰지 않아도 되고, 반환 타입만으로 명확히 알 수 없는 정적 팩터리라면 자신이 반환하는 객체의 스레드 안전성을 반드시 문서화해야 한다.

클래스가 외부에서 사용할 수 있는 락을 제공하면 클라이언트에서 일련의 메서드 호출을 원자적으로 수행할 수 있다. 단, 내부에서 처리하는 고성능 동시성 제어 메커니즘과 혼용할 수 없게 된다. 그래서 ConcurrentHashMap 같은 동시성 컬렉션과는 함꼐 사용하지 못 한다. 또한, 클라이언트가 공개된 락을 오래 쥐고 놓지 않는 서비스 거부 공격을 수행할 수도 있다. 서비스 거부 공격을 막으려면 synchronized 메서드(이 역시 공개된 락이나 마찬가지다) 대신 비공개 락 객체를 사용해야 한다.

```java
// 비공개 락 객체 관용구 - 서비스 거부 공격을 막아준다.
private final Object lock = new Object(); // lock 필드는 항상 final로 선언하자! - 락 객체가 교체되는 일을 예방

public void foo() {
  synchronized(lock) {
    ...
  }
}
```

비공개 락 객체는 클래스 바깥에서는 볼 수 없으니 클라이언트가 그 객체의 동기화에 관여할 수 없다. 이는 락 객체를 동기화 대상 객체 안으로 캡슐화한 것이다. 비공개 락 객체 관용구는 무조건적 스레드 안전 클래스에서만 사용할 수 있다. 조건부 스레드 안전 클래스에서는 특정 호출 순서에 필요한 락이 무엇인지를 클라이언트에게 알려줘야 하므로 이 관용구를 사용할 수 없다.

비공개 락 객체 관용구는 상속용으로 설계한 클래스에 특히 잘 맞는다. 상속용 클래스에서 자신의 인스턴스를 락으로 사용한다면, 하위 클래스는 의도치 않게 기반 클래스의 동작을 방해할 수 있다(그 반대도 마찬가지다). 같은 락을 다른 목적으로 사용하게 되어 하위 클래스와 기반 클래스는 '서로가 서로를 훼방놓는' 상태에 빠진다. 이는 실제고 Thread 클래스에서 나타나는 문제다.



## Item 83. 지연 초기화는 신중히 사용하라

지연 초기화는 필드의 초기화 시점을 그 값이 처음 필요할 때까지 늦추는 기법으로, 정적 필드와 인스턴스 필드 모두에 사용할 수 있다. 주로 최적화 용도로 쓰이지만, 클래스와 인스턴스 초기화 때 발생하는 위험한 순환 문제를 해결하는 효과도 있다. 클래스 혹은 인스턴스 생성 시의 초기화 비용은 줄지만 그 대신 지연 초기화하는 필드에 접근하는 비용은 커진다. 해당 클래스의 인스턴스 중 그 필드를 사용하는 인스턴스의 비율이 낮은 반면, 그 필드를 초기화하는 비용이 크다면 지연 초기화가 제 역할을 해줄 것이다. 멀티스레드 환경에서의 지연 초기화는 까다롭다. 지연 초기화하는 필드를 둘 이상의 스레드가 공유한다면 반드시 동기화해야 버그가 발생하지 않는다.

**대부분의 상황에서 일반적인 초기화가 지연 초기화보다 낫다.** 다음은 인스턴스 필드를 선언할 때 수행하는 일반적인 초기화의 모습이다.

```java
// 인스턴스 필드를 초기화하는 일반적인 방법
private final FieldType field = computeFieldValue();
```

지연 초기화가 초기화 순환성을 깨뜨릴 것 같으면 synchronized를 단 접근자를 사용하자. 이 방법이 가장 간단하고 명확한 대안이다.

```java
// 인스턴스 필드의 지연 초기화 - synchronized 접근자 방식
private FieldType field;

private synchronized FieldType getField() {
  if (field == null)
    field = computeFieldValue();
  return field;
}
```

이상의 두 관용구는 정적 필드에도 똑같이 적용된다. 물론 필드와 접근자 메서드 선언에 static 한정자를 추가해야 한다. **성능 때문에 정적 필드를 지연 초기화해야 한다면 지연 초기화 홀더 클래스(lazy initialization holder class) 관용구를 사용하자.**

```java
// 정적 필드용 지연 초기화 홀더 클래스 관용구
private static class FieldHolder {
  static final FieldType field = computeFieldValue();
}

private static FieldType getField() { return FieldHolder.field; } // 메서드가 처음 호출될 때 FieldHolder 클래스 초기화
```

**성능 때문에 인스턴스 필드를 지연 초기화해야 한다면 이중검사(double-check) 관용구를 사용하라.** 필드의 값을 두 번 검사하는 방식으로, 한 번은 동기화 없이 검사하고, (필드가 아직 초기화되지 않았다면) 두 번째는 동기화하여 검사한다. 두 번째 검사에서도 필드가 초기화되지 않았을 때만 필드를 초기화한다. 필드가 초기화된 후로는 동기화하지 않으므로 해당 필드는 반드시 volatile로 선언해야 한다.

```java
// 인스턴스 필드 지연 초기화용 이중검사 관용구
private volatile FieldType field; // volatile 항상 최근에 기록 된 값을 읽는 것을 보장

private FieldType getField() {
  FieldType result = field; // result 변수는 필드가 이미 초기화된 상황에서는 그 필드를 딱 한 번만 읽도록 보장하는 역할
  if (result != null) // 첫 번째 검사 (락 사용 안 함)
    return result;
	
  synchronized(this) {
    if (field == null) // 두 번째 검사 (락 사용)
      field = computeFieldValue();
    return field;
  }
}
```

정적 필드는 이중검사보다 지연 초기화 홀더 클래그 방식이 낫다.

이중검사에는 주요 변종이 두 가지 있다. 반복해서 초기화해도 상관없는 인스턴스 필드를 지연 초기화해야 할 때가 있는데, 이런 경우라면 이중검사에서 두 번째 검사를 생략할 수 있다. 이 변종의 이름은 단일검사(single-check) 관용구다.

```java
// 단일검사 관용구 - 초기화가 중복해서 일어날 수 있다!
private volatile FieldType field;

private FieldType getField() {
  FieldType result = field;
  if (result == null)
    field = result = computeFieldValue();
  return result;
}
```

앞서 나온 모든 초기화 기법은 기본 타입 필드와 객체 참조 필드 모두에 적용할 수 있다.

모든 스레드가 필드의 값을 다시 계산해도 상관없고 필드의 타입이 long과 double을 제외한 다른 기본 타입이라면, 단일검사의 필드 선언에서 volatile 한정자를 없애도 된다. 이 변종은 짜릿한 단일검사(racy single-check) 관용구라 불린다. 이 관용구는 어떤 환경에서는 필드 접근 속도를 높여주지만, 초기화가 스레드당 최대 한 번 더 이뤄질 수 있다. 보통은 거의 쓰지 않는다.



## Item 84. 프로그램의 동작을 스레드 스케줄러에 기대지 말라

**정확성이나 성능이 스레드 스케줄러에 따라 달라지는 프로그램이라면 다른 플랫폼에 이식하기 어렵다.** 이식성 좋은 프로그램을 작성하는 가장 좋은 방법은 실행 가능한 스레드의 평균적인 수를 프로세서 수보다 지나치게 많아지지 않도록 하는 것이다. 여기서 실행 가능한 스레드의 수와 전체 스레드 수는 구분해야 한다. 전체 스레드 수는 훨씬 많을 수 있고, 대기 중인 스레드는 실행 가능하지 않다. **스레드는 당장 처리해야 할 작업이 없다면 실행돼서는 안 된다.** 실행자 프레임워크를 예로 들면, 스레드 풀 크기를 적절히 설정하고 작업은 짧게 유지하면 된다. 단, 너무 짧으면 작업을 분배하는 부담이 오히려 성능을 떨어뜨릴 수도 있다.

스레드는 절대 바쁜 대기(busy waiting) 상태가 되면 안 된다. 공유 객체의 상태가 바뀔 때까지 쉬지 않고 검사해서는 안 된다.

```java
// CountDownLatch - 바쁜 대기 버전
public class SlowCountDownLatch {
  private int count;
  
  public SlowCountDownLatch(int count) {
    if (count < 0)
      throw new IllegalArgumentException(count + " < 0");
    this.count = count;
  }
  
  public void await() {
    while (true) {
      synchronized(this) {
        if (count == 0)
          return;
      }
    }
  }
  
  public synchronized void countDown() {
    if (count != 0)
      count--;
  }
}
```

하나 이상의 스레드가 필요도 없이 실행 가능한 상태인 시스템은 성능과 이식성이 떨어질 수 있다. 특정 스레드가 다른 스레드들과 비교해 CPU 시간을 충분히 얻지 못해서 간신히 돌아가는 프로그램을 보더라도 **Thread.yield를 써서 문제를 고쳐보려는 유혹을 떨쳐내자.** 이식성이 좋지 않고, 거듭 사용할수록 성능을 떨어뜨릴 수 있다. **Thread.yield는 테스트할 수단도 없다.** 차라리 애플리케이션 구조를 바꿔 동시에 실행 가능한 스레드 수가 적어지도록 조치해주자.

스레드 우선순위를 조절하는 방법도 있지만, 스레드 우선순위는 자바에서 이식성이 가장 나쁜 특성에 속한다. 스레드 몇 개의 우선순위를 조율해서 애플리케이션의 반응 속도를 높일 수도 있지만, 그래야 할 상황은 드물고 이식성이 떨어진다. 심각한 응답 불가 문제의 진짜 원인을 찾아 수정하지 않고 스레드 우선순위로 해결하려는 시도는 합리적이지 않다.

위 두 방법을 프로그램을 간신히 동작하는 프고그램을 '고치는 용도'로 사용해서는 절대 안 된다.
