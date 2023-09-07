# 이왕이면 제네릭 메서드로 만들라
> 클래스 뿐 아니라 메서드도 제네릭으로 만들 수 있다
- 클라이언트에서 입력 매개변수/반환값을 명시적으로 형변환하기보다 안전하고 사용하기 쉬움
- 작성법은 제네릭 타입 작성법과 유사함. 로 타입 말고 제네릭으로 명시하는 것.

## 제네릭 메서드의 활용예
- 매개변수화 타입을 받는 정적 유틸리티 메서드
    * ex. `Collections.binarySearch`
- 제네릭 싱글턴 팩터리
    * 불변 객체를 여러 타입으로 활용할 수 있게 만들 때 요청한 타입 매개변수에 맞게 객체 타입을 바꿔주는 정적 팩터리
    * ex. `Collections.reverseOrder`, `Collections.emptySet`
    * 상태가 없어서 재활용 가능한 함수를 사용할 때
      ```java
      private static UnaryOperator<Object> IDENTITY_FN = (t) -> t;

      @SuppressWarnings("unchecked") // 비검사 형변환 경고 방지
      public static <T> unaryOperator<T> identityFunction() {
          return (UnaryOperator<T>) IDENTITY_FN;
      }
      ```
- 재귀적 타입 한정
    * 자기 자신이 들어간 표현식을 사용하여 타입 매개변수의 허용 범위 한정
    * ex. `public static <E extends Comparable<E>> E max(Collection<E> c);`
    * cf. 와일드카드를 사용한 변형 (item31), 시뮬레이트한 셀프 타입 관용구 (item2)