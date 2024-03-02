# 과도한 동기화는 피하라
> 락을 얻는 데 드는 시간 자체가 문제라기보다 그 과정을 수행하면서 **병렬처리 못하고**, 모든 코어가 메모리를 보게 하는 **지연 시간**이 발생하고, **JVM의 코드 최적화를 제한**하는 점이 비용이다

## 외계인 메서드를 호출하지 말라
> DeadLock과 Safety Failure를 피하려면 **동기화 메서드 혹은 블럭 안에서는 제어를 클라이언트에게 넘기지 마라**
- 동기화된 영역 안에서는 재정의할 수 있는 메서드를 호출하지 마라
- 동기화된 영역 안에서는 클라이언트가 넘겨준 함수 객체(item24)를 호출하지 마라
- 위와 같은 이른바 `외계인 메서드`는 예외, 혹은 교착 상태에 빠트리거나, 안전 실패를 유발할 수 있다


## 외계인 메서드를 호출하는 예
### 자기 자신을 제거하는 관찰자(옵저버)
```java
set.addObserver(new SetObserver<Integer>() {
    @Override
    public void added(ObservableSet<Integer> set, Integer element) {
        System.out.println(element);
        if (element == 23)
            set.removeObserver(this);
    }
});
```
- 결과: 동시성 오류
    * 23에서 조용히 종료되지 않고 `ConcurrentModificationException`을 던짐
- synchronized는 동시성 보장을 해주지만, 정작 자신이 콜백을 거쳐 되돌아와 수정하는 것을 막지 못함
    * 왜? 자바의 락은 재진입(reentrant)을 허용
        + 자바는 각자 고유한 monitor의 결합으로 동기화 작업을 수행하도록 구현됨
    * 자신을 제거하는 외계인 메서드 added가 synchronized 블럭 내에서 실행되면서 순회 도중인 리스트의 원소를 제거하려 시도하는 것. 이것은 허용되지 않는 동작임에도 막을 수 없음.

### 자기 자신을 제거하는 실행자
```java
set.addObserver(new SetObserver<>() {
    @Override
    public void added(ObservableSet<Integer> set, Integer element) {
        ExecutorService exec = Executors.newSingleThreadExecutor();
        try {
            exec.submit(() -> set.removeObserver(this)).get();
        } catch (ExecutionException | InterruptedException ex) { // 다중 catch
            throw new AssertionError(ex);
        } finally {
            exec.shutdown();
        }
    }
});
```
- 결과: 교착 상태
    * 메인 Thread에서 add()를 호출하고, notifyElementAdded()를 호출하여 synchronized 블럭에 진입해 락 획득
    * notifyElementAdded() 내에서 외계인 메서드 added를 호출하면 instance의 removeObserver() 메서드에 접근
    * 이미 메인 Thread에서 락을 쥐고 있으므로, 메인은 함수의 작업이 끝나길 기다리고 함수는 unlock을 기다리며 교착 상태

### 락의 재진입
- 결과: 안전 실패
- 재진입 가능한 락은 멀티쓰레드 프로그래밍을 쉽게 하지만, 교착 실패 상황을 안전 실패로 만드는 위험이 있음


## 대안
### Open Call
> 외계인 메서드를 synchronized 블럭 바깥으로 옮기기
```java
private void notifyElementAdded(E element) {
    List<SetObserver<E>> snapshot = null;
    synchronized (observers) {
        snapshot = new ArrayList<>(observers);
    }
    for (SetObserver<E> observer : snapshot) {
        observer.added(this, element); // 콜백 메서드
    }
}
```
- synchronized 블럭 내에서는 값을 읽어오기만 하고, 외계인 메서드가 읽어온 값을 출력하면 안전
- 락 없이 안전한 순회, 예외 회피, 교착 상태 해소, 다른 스레드의 대기 시간 감소

### 동시성 컬렉션 사용
- **내부를 변경하는 작업은 항상 깨끗한 복사본을 만들어 수행**하도록 구현됨
- 내부 배열이 불변이라 락이 쓰이지 않아 성능이 우수함
- 늘 복사본을 '생성' 하므로 읽기가 많고 수정이 드문 경우에 적합 (ex. 관찰자 리스트)


## 기억하세요
- **동기화 영역에서는 가능한 한 일을 적게** 하라!
- item78을 준수하면서도 open call할 수 있는 방법을 찾아라

### 가변 클래스가 선택할 수 있는 좋은 옵션
1. **동기화를 전혀 하지 말고, 해당 클래스를 동시에 사용해야 하는 클래스가 외부에서 알아서 동기화하게 하라**
2. 동기화를 내부에서 수행해 스레드 안전한 클래스로 다시 만들라 (item82)
    * 클라이언트가 외부에서 클래스 전체에 락을 거는 것보다 동시성 성능을 월등하게 개선할 수 있을 경우에만!
        + 내부적으로 동기화를 수행하던 `StringBuffer`, `Random`은 스레드 안전하지만 느리다.
        + `StringBuilder`와 `ThreadLocalRandom`은 동기화 하지 않은 버전의 클래스다.
    * **둘 중 뭘 할지 모르겠다면 1번을 선택**하라 (그리고 문서에 스레드 안전하지 않다고 써라)
    * 2번을 선택하기로 했다면 아래 방법들을 고려해보라
        1. 락 분할(Lock Splitting)
            + `하나의 큰 락을 여러 개의 작은 락으로 분할`하여 각각 동시에 접근할 수 있도록 함
            + 리스트 앞 뒤에 원소를 추가하는 작업의 락을 분할하여 동시에 여러 스레드가 접근할 수 있게 한다.
        2. 락 스트라이핑(Lock Striping)
            + `여러 개의 락을 생성하여 서로 다른 락을 동시에 사용`하는 기술
            + 각 Entry가 서로 독립적으로 락을 가지는 경우 유용하다.
            + ex. HashTable 버킷마다 별도의 락을 두어 동시에 다양한 버킷에 접근하도록 한다.
        3. 비차단 동시성 제어(Non-blocking Concurrency Control)
            + 스레드가 `락을 획득하지 않고도 동시에 공유 자원에 접근`할 수 있는 기술
            + 하나의 스레드가 락을 획득하지 않아도 다른 스레드는 블로킹되지 않고 작업을 수행할 수 있다.
            + `원자적(Atomic) 연산`이나 `CAS(Compare-and-Swap)`와 같은 연산을 사용하여 구현된다.
            + 교착 상태를 피할 수 있으며, 스레드 간의 경합 상태가 발생하지 않는다.
            + ex. Queue, Stack에 원소를 추가/제거하는 작업을 비차단으로 구현하여 다수의 스레드가 동시에 접근할 수 있도록 한다. 