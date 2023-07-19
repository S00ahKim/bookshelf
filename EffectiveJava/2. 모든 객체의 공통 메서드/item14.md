# Comparable을 구현할지 고민하라

## Comparable 인터페이스
- 인스턴스에 순서를 부여할 수 있을 때 구현한다.
- `compareTo()` 를 갖고 있음
- `equals()`와 차이
    * 순서 비교 가능
    * 제네릭함
    * Object 안에 정의된 메서드가 아님
- 활용 예
    * 정렬
        + 정렬된 컬렉션 ex. `TreeSet` `TreeMap`
        + 유틸리티 클래스 ex. `Collections`, `Arrays`
    * 검색
        + 유틸리티 클래스 ex. `Collections`, `Arrays`
    * 극단값 계산
        + ex. `Long.MAX_VALUE`
- 구현 예
    * 자바 플랫폼 라이브러리의 모든 값 클래스
        + ex. `String`, `Integer`
    * 열거 타입 (item34)
    * 순서가 명확한 클래스
        + ex. `LocalDateTime` (시간)


## 규약
1. 이 객체가 주어진 객체보다 작으면 음의 정수, 같으면 0, 크면 양의 정수 반환
2. 비교할 수 없는 타입의 객체가 주어지면 ClassCastException 던짐
3. 1번을 부호함수 fn 으로 정의한다면... TODO
    * [반사성] 두 객체 참조의 순서를 바꿔서 비교해도 예상한 결과가 나와야 한다.
        + fn(x.compareTo(y)) == -fn(y.compareTo(x))
    * [대칭성] A가 B보다 크고 B가 C보다 크면 A는 C보다 커야 한다.
        + (x.compareTo(y) > 0) && (y.compareTo(z) > 0) 이면 x.compareTo(z) > 0
    * [추이성] 크기가 같은 객체들끼리는 어떤 객체와 비교하더라도 항상 같아야 한다.
        + fn(x.compareTo(y)) == 0 이면 fn(x.compareTo(z)) == fn(y.compareTo(z))
    * compareTo의 동치성 테스트는 equals의 동치성 테스트와 결과가 같을 것을 권한다.
        + (x.compareTo(y) == 0) == (x.equals(y)) 
        + `필수는 아니지만 지키지 않을 경우 별도 표기 권고`. 
        + 왜? 정렬된 컬렉션에서 사용시 의도와 다르게 작동할 수 있음. (정렬된 클래스는 equals가 아니라 compareTo를 사용함)
        + 지켜지지 않은 예?
            - `var a = new BigDecimal("1.0")`, `var b = new BigDecimal("1.00")`
            - `a.equals(b) == false`
            - `a.compareTo(b) == 0`


## 주의사항
### equals와 유사한 만큼, equals의 주의사항도 따르자.
- 기존 클래스를 확장(extends)하는 경우, 규약을 제대로 지킬 수 없다.
- 대신 독립된 클래스 안에 기존 클래스를 타입으로 갖는 필드를 두라.
- compareTo는 새로 정의된 클래스를 위해 새로 구현하라.