# wait와 notify보다는 동시성 유틸리티를 애용하라
> wait와 notify를 올바르게 사용하기 까다로우니 고수준 유틸리티를 권함


## 고수준 동시성 유틸리티(java.util.concurrent)
### 1. 실행자 프레임워크(Executor Service)
- item80

### 2. 동시성 컬렉션(Concurrent Collection)
> List, Queue, Map 같은 표준 컬렉션 인터페이스에 동시성을 가미해 구현한 고성능 컬렉션
- 높은 동시성 도달을 위해 synchronize를 각자의 내부에서 수행 (item79)
- Concurrent Collection에서 동시성을 무력화하는 것은 불가능
    * 상태 의존적 수정 메서드: 여러 기본 동작을 하나의 원자적 동작으로 묶는 메서드
        + ex. Map 인터페이스의 `putIfAbsent()`: 키에 매핑된 값이 없을 때에만 새 값을 집어넣고, 없으면 그 값을 반환
        + 동시성 무력화가 불가능하므로 여러 메서드를 원자적으로 묶어 호출할 수 없기 때문
        + 자바8~ 일반 컬렉션 메서드에도 디폴트 메서드 형태로 추가됨 (item21)
- 외부에서 Lock을 추가로 사용하면 오히려 속도가 느려짐
- 동기화한 컬렉션(Collections.synchronizedXXX) 대신 동시성 컬렉션을 사용할 것 (성능 극적 개선)
- 블로킹 컬렉션(Collections.BlockingXXX)은 작업이 성공적으로 수행될 때까지 기다리도록 확장된 인터페이스
    * ex. BlockingQueue의 경우 Queue의 원소를 꺼낼 때, queue가 비어있으면 새로운 원소가 추가될 때까지 대기
    * 위와 같은 특성 때문에 작업 큐(생산자-소비자 큐)로 사용하기 적합함
    * ThreadPoolExecutor를 포함한 대부분의 ExcutorService 구현체에서 사용

### 3. 동기화 장치(synchronizer)
- 스레드가 다른 스레드를 기다릴 수 있게 함
- 종류
    * 가장 강력한 동기화 장치 `Phaser`
    * 가장 자주 쓰이는 장치 `CountDownLatch`, `Semaphore`
    * 그 외에 `CyclicBarrier`와 `Exchanger`
- CountDownLatch
    ```java
    public class CountDownLatch {
        ...
        public CountDownLatch(int count) { // 유일한 생성자
            if (count < 0) throw new IllegalArgumentException("count < 0");
            this.sync = new Sync(count);
        }
        ...
    }
    ```
    * count값이 Latch의 countDown 메서드를 몇 번 호출해야 대기 중인 쓰레드들을 깨우는지 결정
    ```java
    // 어떤 동작들을 동시에 시작해 모두 완료하기까지 시간을 재는 프레임워크
    public class CountDownLatchTest {
        public static void main(String[] args) {
            ExecutorService executorService = Executors.newFixedThreadPool(5);
            try {
                long result = time(executorService, 3, () -> System.out.println("hello"));
                System.out.println("총 걸린 시간 : " + result);
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                executorService.shutdown();
            }
        }

        public static long time(Executor executor, int concurrency, Runnable action) throws InterruptedException {
            CountDownLatch ready = new CountDownLatch(concurrency);
            CountDownLatch start = new CountDownLatch(1);
            CountDownLatch done = new CountDownLatch(concurrency);

            for (int i = 0; i < concurrency; i++) {
                executor.execute(() -> {
                    ready.countDown(); // 타이머에게 준비가 됐음을 알린다.
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
            return System.nanoTime() - startNanos;
        }
    }
    ```
    * Executor와 동시성 수준(concurrency, 동작을 몇 개나 동시에 수행할 수 있는지)을 매개변수로 받음
    * concurrency 개수만큼 Executor Thread를 생성하고 Thread Pool에 할당
        + `ready.countDown()` : 타이머에게 준비가 되었음을 알린다.
        + `start.await()` : 모든 Executor Thread가 시작될 때까지 기다린다
        + `action.run()` : Runnable action을 실핸한다
        + `done.countDown()` : 타이머에게 작업을 마쳤음을 알린다.
        + `ready.await()` : 모든 Executor Thread가 준비될 때까지 대기한다.
    * ready.countDown()이 시작되면 대기하던 쓰레드들이 동작
    * 마지막 Executor 쓰레드가 작업을 마치면 시계가 멈춤
    * cf. 시간 간격을 잴 때는 항상 `System.currentTimeMillis`가 아닌 `System.nanoTime`을 사용
        + 더 정확하고 정밀하며 시스템의 실시간 시계의 시간 보정에 영향을 받지 않음
        + 1초 미만의 시간이 걸리는 작업이라면 jmh같은 특수 프레임워크를 사용


## wait, notify
- 스레드의 상태 제어를 위한 메서드
    * wait() : 가지고 있던 고유 Lock을 해제하고, 스레드를 잠들게 하는 역할
        + 스레드가 어떤 조건이 충족되기를 기다리게 할 때 사용
        + 락 객체의 wait 메서드는 반드시 객체를 잠근 synchronized 영역 안에서 호출해야 함 
    * notify() : 잠들어 있던 스레드 중 임의로 하나를 골라 깨우는 역할
- **wait과 notify를 사용해야 한다면, 동시성 유틸리티를 먼저 고려할 것**
- 그럼에도 사용해야 한다면...
    ```java
    synchronized (obj) {
        while (조건이 충족되지 않았다) {
            obj.wait(); // 락을 놓고, 깨어나면 다시 잡는다.
        }

        ... // 조건이 충족됐을 때의 동작을 수행한다.
    }
    ```
    * wait 메서드를 사용할 때는 반드시 대기 반복문(wait 루프) 관용구를 사용 & 반복문 밖에서는 절대 호출하지 마라!
        + 호출 전 검사: 조건이 충족되었다면 wait을 건너뛰게 하여 **응답 불가 상태를 예방**
        + 호출 후 검사: **안전 실패 상태를 예방**
    * 일반적으로 notify보다 **notifyAll을 사용하는 게 합리적이고 안전**
        + 깨어나야 하는 모든 쓰레드가 깨어남을 보장하므로 **항상 정확한 결과**를 얻음
        + 다른 쓰레드까지 깨어날 수 있으나, 조건이 충족되지 않으면 다시 잠들 것이므로 프로그램 정확성에 큰 문제는 없음
        + notifyAll은 모든 쓰레드를 깨워버리므로, notify로 인해 관련 없는 쓰레드가 실수 혹은 악의적으로 호출되는 경우를 막을 수 있음
        + 모든 쓰레드가 같은 조건을 기다리고, 조건이 한 번 충족될 때 단 하나의 쓰레드만 혜택을 받는 특수한 경우엔 notify로 최적화할 수 있음