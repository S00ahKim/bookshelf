# 인스턴스화를 막으려거든 private 생성자를 사용하라

## 정적 메서드와 정적 필드만 담은 클래스가 유용한 경우가 있다.
- `java.lang.Math`
    * 기본 타입 값, 수학 계산을 위한 메서드를 모아둠
    * `public static final double PI = 3.14159265358979323846;`, `Math.max()` 등을 제공함
- `java.util.Arrays`
    * 배열 관련 유틸리티 메서드를 모아둠
    * `Arrays.sort()` 등을 제공함
- `java.util.Collections`
    * 특정 인터페이스를 구현하는 객체를 생성해주는 정적 메서드 또는 팩터리를 모아둠
    * `Collections.sort()`, `Collections.emptyList()` 등을 제공함
    * cf. 자바8 부터 인터페이스에 이런 메서드를 넣는 것이 가능하게 되었음
- final 클래스와 관련한 메서드를 모아둠

## private 생성자를 추가하는 것으로 인스턴스화를 막을 수 있다.
- 컴파일러는 생성자가 없을 때만 기본 생성자를 만드는데, 그러면 의도치 않게 인스턴스화할 수 있게 됨
- 직관적이지 않을 수 있으니 주석을 달아두는 것 추천
- 상속을 불가능하게 하는 효과도 있음
- cf. lombok은 `@NoArgsConstructor(access = AccessLevel.PRIVATE)`으로 손쉽게 만들 수 있음

## 추상 클래스로 만드는 것으로는 인스턴스화를 막을 수 없다.
- 하위 클래스를 만들어 인스턴스화할 수 있기 때문
- 사용처에서 상속해서 쓰라고 오해할 가능성이 있음 (item19)