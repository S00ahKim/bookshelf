# 다 쓴 객체 참조를 해제하라

## 자바는 메모리 관리를 하지 않아도 된다는 착각
- `가비지 콜렉터`(GC)가 있기 때문에 C, C++처럼 직접 관리할 필요는 없다.
    * GC는 주기적으로 JVM의 heap 메모리를 점검하여 스택 영역이 참조하지 않는 객체를 메모리에서 해제한다.
    * Young/Old Generation으로 나뉘고, Young은 Eden/Survivor로 나뉜다.
    * YG에서 일어나는 GC는 마이너GC라고 한다.
    * GC 알고리즘은 여러가지가 있는데, mark and sweep 도 그중 하나다.
- 하지만 신경은 써야 한다! 메모리 누수가 발생하는 코드가 있다면...
    * GC 활동과 메모리 사용량이 늘어나 **성능이 저하**될 것
    * 심하면 디스크 페이징 / OutOfMemoryError 발생 등으로 **프로그램이 종료**될 것
        + 디스크 페이징? 주기억장치(RAM)보다 큰 용량의 메모리 공간을 프로세스에게 제공하기 위해 가상 메모리를 사용. 페이지로 나눠서 필요한 만큼만 RAM에 로드함. 프로세스가 페이지에 접근하려고 할 때 Page Fault가 발생하면 종료될 수 있음.
- 메모리 누수는 잘 드러나지 않는다.
    * 철저한 코드 리뷰 / 힙 프로파일러 등 디버깅 도구를 동원해야 발견 가능
        + 인텔리제이의 경우, 상용 버전에서 [프로파일링 기능](https://www.jetbrains.com/help/idea/read-the-profiling-report.html)을 제공한다. 애플리케이션의 실행 방식과 메모리, CPU 리소스가 할당되는 방식에 대한 분석을 제공하는 `Async Profiler`, 애플리케이션이 실행되는 동안 JVM에서 발생한 이벤트에 대한 정보를 수집하는 모니터링 도구인 `Java Flight Recorder` 등을 지원하고, 이를 [그래프 등으로 볼 수 있다](https://www.youtube.com/watch?v=OQcyAtukps4).
    * 애초에 예방하는 것이 가장 좋은 방법

## 메모리 누수(leak)는 왜 발생하는가
- GC는 참조가 살아 있으면 그걸 사용하거나 거기에 사용되는 모든 객체를 회수하지 못함
    * 활성 영역? 점유 중인 메모리 공간 중 실제로 프로그램 내에서 사용되는 영역
    * GC는 활성 영역과 비활성 영역을 구분하지 못한다. 쓸모 없는지를 판단할 수 없다.
- **주범(1) 클래스가 자기 메모리를 직접 관리할 때, 다 쓴 참조를 갖고 있는 경우**
    * ex. 스택을 구현할 때, 실제로 저장하는 변수에서 제거하는 게 아니라 슬라이싱해서 리턴하는 식으로 pop()를 구현
        + 이 경우, 활성 영역은 정의된 size 까지의 영역이고, 나머지는 비활성 영역이 됨
    * 해결: 해당 참조를 다 썼으면 **null 처리(참조 해제)**하기
        + 메모리 누수 뿐 아니라 실수로 참조하는 것도 방지
        + null 처리는 **극히 예외적인 경우**에만 사용해야 한다
- **주범(2) 캐시를 사용할 때, 객체를 다 쓰고도 놔두는 경우**
    * 해결: 외부에서 키를 참조하는 동안만 캐시의 엔트리가 살아있으면 되는 경우에 한해 WeakHashMap을 사용하라
        + 그런 경우에 한해 WeakHashMap이 유용하다.
        + 그런 경우가 언제가 될 수 있나?
          ```java
          // 상품별로 자신을 판매했던 방송을 조회할 수 있는 HashMap이 있다고 가정한다. (상품키:방송리스트)
          // 하고자 하는 작업은 성인용 제품에 대한 필터링이라면, 상품키로 성인용 제품 여부를 확인하고, 성인용 제품이 아닌 경우에는 굳이 저장하지 않아도 된다.
          private static Map<ProductKey, List<Broadcast>> adultProductCache = new WeakHashMap<>();
          ...
          ProductKey productKey = new ProductKey(123L);
          // 단, Integer productKey = 123; 과 같이 리터럴 방식으로 선언될 경우에는 JVM이 따로 풀을 사용해서 메모리 관리를 하므로 이 경우에 해당하지 않는다.
          adultProductCache.put(productKey, getBroadcastByProductKey(productKey));
          // 단, 값에서 키를 참조하고 있을 경우에는 아래와 같이 처리해도 삭제되지 않는다.
          if (isNotForAdult(productKey)) {
            productKey = null; // 이러면 productKey의 강한 참조가 제거되어 GC 대상이 됨 
          }
          ...
          ```
        + 참조의 종류
            1. 강한 참조 (Strong Reference)
                  - 가장 일반적인 참조 유형
                  - new 연산을 통해 객체를 생성하는 경우
                  - 객체에 대한 강한 참조가 있으면 GC 대상이 되지 않음
                  - null 처리는 Unreachable Object로 만들어 강제로 GC되게 하는 것
            2. 약한 참조 (Weak Reference)
                  - `WeakReference<Integer> soft = new WeakReference<Integer>(prime);` (java.lang.ref.WeakReference)
                  - 해당 객체를 가리키는 강한 참조가 없는 경우 GC의 대상이 됨
            3. 소프트 참조 (Soft Reference)
                  - `SoftReference<Integer> soft = new SoftReference<Integer>(prime);` (java.lang.ref.SoftReference)
                  - 약한 참조와 유사하지만, 메모리 부족 상황일 때에만 GC 대상이 됨
            4. 팬텀 참조 (Phantom Reference)
                  - `java.lang.ref.PhantomReference`
                  - 객체의 생명주기 추적을 위해 사용됨
                  - 실제로 참조하는 객체를 사용할 수는 없으며, 오직 GC되기 직전에 알림만 받을 수 있음
    * 해결: 시간이 지날수록 엔트리의 가치를 떨어뜨린다.
        + 대개 엔트리의 유효기간을 정하기 어렵기 때문임
        + 캐시에 세팅된지 오래되었거나(TTL) 캐시에 접근하지 않은지 오래된(LRU) 경우 해제하는 식
        + 백그라운드 스레드를 활용하거나, 새 값을 추가할 때 부가 작업으로 수행하곤 함
- **주범(3) 리스너/콜백을 사용할 때, 클라이언트가 등록만 하고 해지하지 않는 경우**
    * 해결: 콜백을 약한 참조(weak reference)로 지정하기

## 가장 좋은 다 쓴 참조 해제법은 그 참조를 담은 변수를 유효 범위 밖으로 밀어내는 것이다
- 유효 범위? Scoping. 변수가 프로그램에서 접근 가능한 범위
    * 변수의 유효 범위는 중괄호 {}로 정의된 코드 블록에 의해 제한됨
    * ex.메소드 안에서 선언된 건 메소드 안에서만 유효
      ```java
      for (int i = 0; i < 5; i++) {
            // 변수의 유효 범위가 for 반복문에 해당
            // 변수 i는 반복문이 종료되면 유효 범위를 벗어나므로 메모리에서 해제됨
            System.out.println(i);
        }
      ```
- 변수의 범위를 최소가 되게 정의했다면(item57) 자연스럽게 이뤄짐