# 스레드보다는 실행자, 태스크, 스트림을 애용하라

## 작업 큐
- 과거에는 클라이언트가 요청한 작업을 백그라운드 스레드에 위임해 비동기적으로 처리하기 위해 단순한 작업 큐를 사용함
    * 작업 큐의 동작 방식
        1. Client 요청 작업을 Background Thread에 위임해 비동기적으로 작업을 처리한다.
        2. 작업 큐가 필요 없어지면 Client가 queue에 중단을 요청할 수 있다.
        3. Queue는 남아 있는 작업을 마저 완료한 후 스스로 종료한다.
        4. Safety Failure나 Dead Lock가 될 여지를 없애는 작업을 수반했었다.
- 작업 큐를 사용할 경우 위와 같이 예외 상황에 대처해야 해서 수많은 코드 작업이 발생됨
- 실행자 프레임워크의 등장으로 이제 이러한 수고를 덜 수 있게 되었음


## 실행자 프레임워크 (Executor Framework)
- `java.util.concurrent`
- 인터페이스 기반의 유연한 태스크 실행 기능 지원
- 장점 : 기존보다 모든 면에서 뛰어난 작업 큐를 이제는 단 한 줄로 생성할 수 있다.
    ```java
    // 작업 큐 생성
    ExecutorService exec = Executors.newSingleThreadExecutor();

    // 실행할 태스크를 넘김
    exec.execute(runnable);

    // 실행자 종료
    exec.shutdown();
    ```

### 주요 기능
1. 특정 태스크가 완료되기를 기다림.
     - get() 메서드를 통해 태스크가 완료되기를 기다릴 수 있다.
      ``` java
      ExecutorService exec = Executors.newSingleThreadExecutor();
      exec.submit(()  -> s.removeObserver(this)).get();
      ```
2. 태스크 모음 중 아무것 하나(invokeAny 메서드) 혹은 모든 태스크(invokeAll 메서드)가 완료되기를 기다림.
      ```java
      ExecutorService exec = Executors.newSingleThreadExecutor();
      exec.submit(()  -> s.removeObserver(this)).get();
      ```
3. 실행자 서비스가 종료하기를 기다림(awaitTermination 메서드).
      ``` java
      exec.awaitTermination(10, TimeUnit.SECONDS);
      ```
4. 완료된 태스크들의 결과를 차례로 받는다(ExecutorCompletionService 이용).
      ``` java
      ExecutorService exex = Executors.newFixedThreadPool(2);
      ExecutorCompletionService executorCompletionService = 
          new ExecutorCompletionService(exex);
      ```


## 스레드 풀
- 고정하거나 필요에 따라 늘어나거나 줄어들게 설정 가능
- 방법 1. `java.util.concurrent.Executors` 정적 팩토리들을 이용
    * ex. `newFixedThreadPool()` 인자 개수만큼 고정돼 Thread Pool 생성
        + 무거운 프로덕션 서버에 적합함
    * ex. `newCachedThreadPool()` 필요할 때, 필요한 만큼 Thread Pool 생성
        + 특별히 설정할 게 없고, 작은 프로그램이나 가벼운 서버에 적합함
        + 요청받은 태스크들이 큐에 쌓이지 않고 즉시 스레드에 위임돼 실행됨. 가용한 스레드가 없다면 새로 하나를 생성함.
        + 상기 이유로 서버가 무겁다면 쓰기 좋지 않음
- 방법 2. `ThreadPoolExecutor` 클래스를 직접 사용
    * 스레드 풀 동작을 결정하는 거의 모든 속성을 제어할 수 있음
    * 무거운 프로덕션 서버에 적합함


## 스레드를 직접 다루는 것을 삼갈 것
> 작업 큐를 손수 만드는 일과 스레드를 직접 다루는 일은 삼가야 한다
- 스레드를 직접 다루면 스레드가 작업 단위와 수행 매커니즘 역할을 모두 수행
- 실행자 프레임워크를 사용하면 **작업 단위와 실행 매커니즘이 분리**됨
    * 작업 단위
        + 핵심 추상 개념 `태스크`
            - Runnable
            - Callable (Runnable과 비슷하지만 값을 반환하고 임의의 예외를 던질 수 있음)
    * 실행 매커니즘
        + `실행자 서비스` 
            - 원하는 태스크 수행 정책을 선택할 수 있음 & 언제든 변경할 수 있음
            - Collection Framework가 데이터 모음을 담당하듯, Executor Framework가 작업 수행을 담당함
- ex. 포크-조인(fork-join)
    * 자바7~ 실행자 프레임워크에서 포크-조인(fork-join) 태스크를 지원 (`ForkJoinPool`)
    * Task의 크기에 따라 분할(Fork)하고, 분할된 Task가 처리되면 합쳐(Join)서 리턴