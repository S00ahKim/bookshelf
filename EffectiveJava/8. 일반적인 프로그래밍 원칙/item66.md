# 네이티브 메서드는 신중히 사용하라
> 쓸 거면 최소한만 사용하고 신중하게 테스트하라

## 자바 네이티브 인터페이스 (JNI)
- 자바 프로그램이 네이티브 메서드를 호출하는 기술
- 네이티브 메서드?
    * C 또는 C++ 같은 네이티브 프로그래밍 언어로 작성한 메서드
    * 주요 쓰임
        1. `레지스트리` 같은 플랫폼 특화 기능 사용
            * 윈도우에서 사용하는 시스템 구성 정보를 저장한 데이터베이스
        2. 네이티브 코드로 작성된 기존 라이브러리를 사용
            * ex. OpenCV (Open Source Computer Vision) 등
        3. 성능 개선을 목적으로 성능에 결정적인 영향을 주는 영역만 네이티브 언어로 작성


## 자바가 제공하는 것으로 충분
- OS 등 하부 플랫폼의 기능을 흡수하고 있음
    * ex. 자바9~ `process API` (OS 프로세스 접근)
    * ```java
      public class JavaProcess {
          public static void printProcessInfo(){
              ProcessHandle processHandle = ProcessHandle.current();
              ProcessHandle.Info processInfo = processHandle.info();
          
              System.out.println("processHandle.pid(): " + processHandle.pid());
              System.out.println("processInfo.arguments(): " + processInfo.arguments());
              System.out.println("processInfo.command(): " + processInfo.command());
              System.out.println("processInfo.startInstant(): " + processInfo.startInstant());
              System.out.println("processInfo.user(): " + processInfo.user());    
          }
          public static void main(String[] args) {
              printProcessInfo();
          }
      }

      // processHandle.pid(): 847020
      // processInfo.arguments(): Optional.empty
      // processInfo.command(): Optional[C:\Program Files\Java\jdk-17\bin\java.exe]
      // processInfo.startInstant(): Optional[2023-07-25T01:43:55.528Z]
      // processInfo.user(): Optional[DESKTOP-DEGF65N\qud12]
      ```
- 섬세하게 튜닝하며 성능을 계속 개선해옴
    * ex. java.math `BigInteger`
        + 자바3 때 개선 이후 네이티브 구현보다 빨라졌었다
        + 네이티브 라이브러리가 계속 개선하던 것에 비해
        + 자바에서는 이후 대규모 개선이 적었다
        + 그래서 정말로 고성능 다중 정밀 연산이 필요하면 네이티브를 고려해도 좋다


## 네이티브 메서드의 단점
- 네이티브 언어는 **안전하지 않다** (item50)
    * ex. 메모리 훼손 오류, 메모리 오버플로우, 이미 해제된 메모리에 접근, 메모리 누수 등
    * `메모리 훼손 오류`? 메모리를 잘못된 방식으로 읽거나 쓰는 오류
- 플랫폼을 자바 언어보다 더 많이 타서 **이식성이 낮다**
- 디버깅이 더 어렵다
- 주의하지 않으면 속도가 더 느려진다
- 가비지 컬렉터는 네이티브 메모리를 자동 회수하지 못하고, 추적도 못한다 (item8)
- 자바 코드와 네이티브 코드의 경계를 넘나들 때 비용이 추가된다
- 자바 코드와 네이티브 코드 간 접착 코드를 작성하는 것은 귀찮고 가독성도 떨어진다