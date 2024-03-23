# 프로그램의 동작을 스레드 스케줄러에 기대지 말라

## 이식성을 높이는 방법
> 정확성이나 성능이 스레드 스케줄러에 따라 달라지는 프로그램이라면 다른 플랫폼에 이식하기 어렵다.
1. **실행 가능한 쓰레드 평균 개수를 Processor 개수보다 지나치게 많아지지 않도록 하라.**
    - 그래야 Scheduler가 고민할 거리가 줄어든다.
    - 실행 준비가 된 쓰레드들은 맡은 작업을 완료할 때까지 계속 실행되도록 만들어라
    - 전체 쓰레드 수는 훨씬 많을 수 있고, 대기 중인 쓰레드는 실행 가능하지 않다. (실행 가능한 쓰레드 수를 구분하라는 의미)
2. **각 쓰레드가 어떤 유용한 작업을 완료한 후에 다음 일거리가 생길 때까지 대기하도록 만들어라.**
    - 실행 가능한 쓰레드 수를 적게 유지하는 주요 기법
    - 쓰레드는 당장 처리해야 할 작업이 없다면 실행돼서는 안 된다.
    - Executor Framework(item80)의 경우, Thread Pool 크기를 적절히 설정하고 작업을 짧게 유지하면 된다.
        * 단, 너무 짧으면 작업을 분배하는 부담이 성능을 저하시킬 수 있다.


## 피해야 할 상황
### 바쁜 대기 Busy waiting
```java
// 바쁜 대기 버전 CountDownLatch 구현
public class SlowCountDownLatch {
    private int count;
	
    public SlowCountDownLatch(int count) {
        if (count < 0)
            throw new IllegalArgumentException(count + " < 0");
        this.count = count;
     }
    
    public void await() { // 바쁜 대기 중 (하지 마라)
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
- 쓰레드 변덕에 취약하고, Processor에 큰 부담을 주어 다른 유용한 작업이 실행될 기회를 박탈한다.
- 위 예시는 자바의 CountDownLatch와 쓰레드를 1,000개 만들었을 때, 성능이 약 10배 정도 느렸다. (저자 기준)

### Thread.yield에 의존해선 안 된다
```java
class MyThread extends Thread {
    @Override
    public void run() {
        for (int i = 0; i < 5; i++)
            System.out.println(Thread.currentThread().getName() + " in control");
    }
}

public class Main {
    public static void main(String[] args) {
        MyThread t = new MyThread();
        t.start();
        
        for (int i = 0; i < 5; i++) {
            // Control passes to child thread
            Thread.yield();
        
            // After execution of child Thread
            // main thread takes over
            System.out.println(Thread.currentThread().getName() + " in control");
        }
    }
}
```
- 다른 쓰레드에게 실행을 양보하는 메서드
- 증상이 어느정도 호전될 수 있겠으나, 이식성은 그렇지 않다.
- 테스트할 수단도 없다.
- 차라리 Application 구조를 바꾸어 동시에 실행 가능한 쓰레드 수가 적어지도록 조치하라.

### 쓰레드 우선 순위에 의존해서도 안 된다.
- 쓰레드 우선 순위는 Java에서 이식성이 가장 나쁜 특성에 속한다.
- 쓰레드 몇 개의 우선순위를 조율해서 Application 반응 속도를 높이는 상황은 드물고 이식성이 떨어진다.
- '잘 동작하는' 프로그램 서비스 품질을 높이기 위해 드물게 쓰일 수는 있으나, '고치는 용도'로는 절대 사용하지 마라
- 심각한 응답 불가 문제를 쓰레드 우선 순위로 해결하려는 시도는 절대 합리적이지 않다.