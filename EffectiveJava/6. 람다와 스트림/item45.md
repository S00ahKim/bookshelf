# 스트림은 주의해서 사용하라
> 스트림과 반복문 중 어떤 것이 나은지 확신할 수 없다면 둘 다 해보고 나은 쪽을 선택하라

## 스트림 API
- 자바8~
- 목적: 다량의 데이터 처리 작업 지원
- 핵심: 스트림(요소) & 스트림 파이프라인(연산)
- 스트림(stream)
    * 어디서든 스트림이 만들어질 수 있다 ex. 컬렉션, 배열, 파일 등
    * 스트림의 데이터는 객체 참조 또는 기본 타입 값(int, long, double)
- 스트림 파이프라인(stream pipeline)
    ```java
    collection.stream()              // 소스 스트림
        .filter(x -> x.flag == true) // 중간 연산 (transform)
        .count();                    // 종단 연산
    ```
    * lazy evaluation
        + 종단 연산이 호출될 때 평가를 해서 여기에 쓰이지 않으면 계산에 쓰이지 않는다.
        + 즉 종단 연산이 호출되지 않으면 파이프라인은 아무것도 하지 않는다.
    * 스트림 API는 fluent API다. 메서드 체이닝을 지원한다.
    * 기본적으로 순차적 실행
        + 병렬 실행이 필요하면 parallel()을 호출하면 되나, 효과적인 상황은 많지 않음 (item48)


## 주의
- 스트림을 과용하면 프로그램을 읽거나 유지보수하기 어렵다.
    * 가독성 tip
        1. 람다식에서 매개변수 이름을 의미에 맞게 지어준다. ex. `.(word -> word.getFirstLetter())` 
        2. (특히 스트림에서는) 도우미 메서드를 만든다. ex.`.(word -> isAnagram(word))`
- char 값들을 처리할 때에는 스트림을 사용하지 말라.
    * ```java
      "Hello Char".char().forEach(Systeom.out::print); // 72101108108111321191111410810
      ```
    * char 스트림을 지원하지도 않는다. 명시적으로 형변환해주어야 한다.


## 스트림은 언제?
- 스트림을 사용하면 좋은 경우는 아래와 같다.
    * 원소들의 시퀀스를 일관되게 변환
    * 원소들의 시퀀스를 필터링
    * 원소들의 시퀀스를 하나의 연산을 사용해 결합 (더하기, 연결하기, 최솟값 구하기 등)
    * 원소들의 시퀀스를 컬렉션으로 모음
    * 원소들의 시퀀스에서 특정 조건을 만족하는 원소를 찾음
- 스트림을 사용하기 어려운 경우는 아래와 같다.
    * 파이프라인의 여러 단계를 통과할 때, 이 데이터의 각 단계에서의 값들에 동시에 접근해야 하는 경우
        + ex. 메르센 소수(2^p-1 형태의 수인데, p가 소수인 수)를 출력하는 프로그램
        + ```java
          public class MersennePrime {
              private static final BigInteger TWO = BigInteger.valueOf(2);

              public static void main(String[] args) {
                  primes().map(p -> TWO.pow(p.intValueExact()).subtract(BigInteger.ONE))
                          .filter(mersenne -> mersenne.isProbablePrime(50))
                          .limit(20)
                          .forEach(mp -> System.out.println(mp.bitLength() + ": " + mp));
                          // ㄴ 메르센 소수 앞에 지수(p)를 출력하기 위해 mp를 역산하여 얻어냈다.
              }

              static Stream<BigInteger> primes() { // 스트림 반환 메서드는 복수명사가 좋다
                  return Stream.iterate(TWO, BigInteger::nextProbablePrime);
              }
          }

          /*
          2: 3
          3: 7
          5: 31
          7: 127
          13: 8191
          17: 131071
          19: 524287
          ...
          */
          ```
        + **스트림 파이프라인은 일단 한 값을 다른 값에 매핑하고 나면 원래 값을 잃는다**.
        + 우회하는 방법은 있을 수 있다. ex. 원래 값과 새로운 값을 쌍으로 넘기기, 역산하게 하기
        + 그러나 스트림의 존재 의의인 가독성과 단순성을 희생해야 하기 때문에 만족스럽지 못할 것이다.


## 더 나은 쪽을 선택하자
- 기존 코드는 스트림을 사용하여 리팩터링하되, 새 코드가 더 나아 보일 때만 반영하라.
    * 때로는 반복문과 스트림을 조합하는 게 최선일 때도 있다.
    * 스트림에서는 파이널 변수만 접근할 수 있다. (지역 변수 접근 불가 ㅠㅠ)
        + 외부 변수를 lambda 안의 지역 변수로 사용하면, 해당 외부 변수를 복사한 형태로 사용함
        + 지역 변수를 관리하는 쓰레드와 람다가 실행되는 쓰레드가 다를 수 있음
        + 지역 변수는 스택 영역에서 생성되며, 해당 block이 끝나면 스택에서 사라짐
    * 람다에서는 return이나 break나 continue로 빠져나가거나 건너뛰거나 예외를 던질 수 없다.
- 때로는 단순히 취향 차이일 수도 있다.
    * 팀원들이 어느 쪽에 익숙한지를 고려하자.