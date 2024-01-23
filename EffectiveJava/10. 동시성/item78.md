# 공유 중인 가변 데이터는 동기화해 사용하라
> synchronized: 해당 메서드나 블록을 한번에 한 스레드씩 수행하도록 보장

## 동기화의 주요 역할
1. 배타적 실행
    - 일관성이 깨진 상태를 볼 수 없게 한다 (= 접근한 값은 수정이 완전히 끝남)
    - 객체에 접근할 때 lock 을 검 (= 변경 중인 객체에 대한 접근을 막음)
2. **스레드 사이의 안정적 통신**
    - 한 스레드가 저장한 값이 다른 스레드에 보이게 한다 (= 접근한 값은 시스템 통틀어 최신임)
    - lock의 보호 하에 수행된 최종 결과를 보여줌
    - 자바 언어 명세상 long과 double 외의 변수를 읽고 쓰는 동작이 원자적인데도 동기화가 필요한 이유
        * 왜 long, double은 안 됨? 32비트 자바에 대한 설명임. 32-bit memory에 값을 할당하는 연산은 원자적인데, 64-bit 연산을 수행하려면 값을 분할해야 하기 때문임. 만약 64비트 자바를 쓰면 모두 원자적임.


## 동기화에 실패한 예들
- `Thread.stop` 메서드
    * deprecated 지정, 자바11~ Throwable을 인수로 받는 stop()은 제거됨
    * 데이터가 훼손될 수 있으니 사용하면 안 됨
    * 왜? 해당 Thread를 바라보던 다른 Thread들에 ThreadDeathException가 전파되어 모든 lock이 해제됨
- 스레드를 멈추려고 할 때 true 로 변경하는 변수를 두는데 동기화를 안 함
    ```java
    public class StopThread {
        private static boolean stopRequested;

        public static void main(String[] args) throws InterruptedException {
            Thread backgroundThread = new Thread(() -> {
                int i = 0;
                while (!stopRequested)
                    i++;
            });
            backgroundThread.start();

            TimeUnit.SECONDS.sleep(1);
            stopRequested = true; // 여기서 true로 바꿔주어도 backgroundThread는 끝나지 않음
        }
    }
    ```
    * boolean은 원자적이나, 메인 스레드가 수정한 값을 백그라운드 스레드가 언제 보게 될지 보증 불가 (통신 깨짐)
    * 가상 머신의 최적화 과정에서 코드를 아래와 같이 변경시킬 수도 있음
        ```java
        // 원래 코드
        while (!stopRequested)
            i++;
            
        // 가상 머신에 의해 최적화된 코드
        if (!stopRequested)
            while (true)
                i++;
        ```
        + OpenJDK 서버 VM의 끌어올리기(hoisting) 기법
            - [Java synchronized 키워드와 Memory Barrier](https://zbvs.tistory.com/25)
            - 바이트코드 인터프리터가 읽게 할 코드 외에 자주 접근되는 코드는 `JIT 컴파일러`가 컴파일함
            - JIT 컴파일러는 위 코드를 `동기화 한정자가 없으니 싱글 스레드에서 실행될 것이라 간주`함
            - 멀티 스레드 환경이라면 stopRequested를 누가 바꾸나 안 바꾸나 계속 검사하는 부분이 내부적으로 추가됨
            - 그런데 싱글 스레드 환경이라고 간주하는 순간 코드상에서 저 변수에 접근하지 않는다면 그런 검사가 불필요해짐
            - 그러므로 아예 영원히 안 바뀌겠거니 하고 계속 true로 돌도록 내부적으로 처리하는 것 (hoisting)
        + **응답 불가** (liveness failure) 상태가 됨
            - 프로세스가 무한정 기다리는 상황
- 동기화를 쓰기나 읽기 어느 한 곳에만 함
    * StopThread 예시는 사실 배타적 수행이 불필요하므로 어느 한쪽에만 걸어도 되긴 된다
    * 하지만 다른 경우에는 쓰는 쪽은 기다리면서 쓰는데 읽는 건 맘대로 읽어서 결과가 이상하게 나올 수 있다
- 일련번호를 생성하는 코드에 동기화를 잘못함
    ```java
    private static volatile int nextSerialNumber = 0;

    public static int generateSerialNumber() {
        return nextSerialNumber++;
    }
    ```
    * `nextSerialNumber++`은 한 줄이지만, 내부적으로는 값에 두 번 접근함
        1. 값을 **메모리에서 읽어** 캐시로 가져옴
        2. **캐시값**에 1을 더함
        3. 캐시값을 저장함
        4. 캐시에서 메모리로 값을 저장함
    * 아직 3이 되지 않았는데 다른 스레드에서 캐시에 접근하면 **안전 실패**(safety failure)함
        + 프로그램이 잘못된 결과를 계산하는 오류 
    * 배타적 수행이 필요한 경우이므로, `synchronized` 키워드를 붙여주어야 함
        + 붙인 뒤엔 volatile은 제거해야 함
        + cf. int가 다루는 수는 한계가 있으므로 long으로 바꿔주면 더 안정적임. 아니면 최대값 도달 시 오류내게 하거나.


## 동기화에 실패하면 디버깅이 어렵다
- 문제 자체가 간헐적이거나 특정 타이밍에만 발생함
- VM에 따라 현상이 다르게 일어나기도 함 (최적화 방식이 각기 달라서)


## 동기화를 잘한 예들
- **동기화가 필요한 변수에 쓰기/읽기 모두 동기화함**
    ```java
    public class StopThread {
        private static boolean stopRequested;

        private static synchronized void requestStop() {
            stopRequested = true;
        }

        private static synchronized boolean stopRequested() {
            return stopRequested;
        }

        public static void main(String[] args) throws InterruptedException {
            Thread backgroundThread = new Thread(() -> {
                int i = 0;
                while (!stopRequested())
                    i++;
            });
            backgroundThread.start();

            TimeUnit.SECONDS.sleep(1);
            requestStop();
        }
    }
    ```
- 배타적 수행까지는 불필요한 경우 `volatile` 한정자를 사용함
    ```java
    public class StopThread {
        private static volatile boolean stopRequested;

        public static void main(String[] args) throws InterruptedException {
            Thread backgroundThread = new Thread(() -> {
                int i = 0;
                while (stopRequested)
                    i++;
            });
            backgroundThread.start();

            TimeUnit.SECONDS.sleep(1);
            stopRequested = true;
        }
    }
    ```
    * volatile 한정자를 붙여서 항상 가장 최근에 기록된 값을 읽도록 보장함
        + 배타적 수행 X, 안정적 통신 O
        + volatile로 선언된 변수는 CPU에서 연산을 하면 캐시가 아니라 바로 메모리에 접근함
    * 반복을 할 때마다 동기화하는 비용을 줄임
        + synchronized의 lock & unlock 과정의 context switching으로 인한 overhead가 줄기 때문
- 스레드 안전한 프로그래밍을 지원하는 클래스를 사용함 (ex. `AtomicLong`)
    ```java
    private static final AtomicLong nextSerialNum = new AtomicLong();

    public static long generateSerialNumber() {
        return nextSerialNum.getAndIncrement();
    }
    ```
    * `java.util.concurrent.atomic` 패키지
    * 배타적 수행 O, 안정적 통신 O
    * 락 없이도 스레드 안전한 프로그래밍 지원
        + 어떻게? [CAS(Compare And Swap)](https://devfunny.tistory.com/812)
        + 변수의 값을 변경하기 전에 기존에 가지고 있던 값이 내가 예상하던 값과 같을 경우에만 새로운 값으로 할당하는 방법
        + 아예 멈춰두고(lock, mutual exclusion) 처리하는 synchronized 보다 우수한 성능
- 공유하는 부분만 동기화 (`사실상 불변`, `안전 발행`)
    * 사실상 불변: 한 스레드가 데이터를 수정한 후 공유할 땐 공유할 부분만 동기화함
    * 안전 발행: 사실상 불변인 객체를 다른 스레드에 전달하는 행위
        + 클래스 초기화 과정에서 객체를 `static` 필드, `volatile` 필드, `final` 필드, 혹은 lock 등을 통해 접근하는 필드에 저장
        + 값을 동시성 컬렉션(item81)에 저장
- **애초에 가변 데이터를 공유하지 않는 것이 모든 복잡함을 피하는 최고의 방법**
    * 불변 데이터(item17)만 공유함
    * 아예 공유 변수를 만들지 않음
    * 그리고 이 정책을 영원히 지켜지게 함 (문서화 등)
    * 그리고 사용하는 프레임워크에 대한 이해도를 높임 (실수하지 않도록)