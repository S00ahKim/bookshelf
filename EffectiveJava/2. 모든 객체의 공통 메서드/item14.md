# Comparable을 구현할지 고민하라

## Comparable 인터페이스
- 인스턴스에 순서를 부여할 수 있을 때 구현한다.
- `compareTo()` 를 갖고 있음
- `equals()`와 차이
    * 순서 비교 가능
    * 제네릭함
        + generic? 클래스나 메서드를 작성할 때 타입을 일반화하는 기능
        + 실제로 사용되는 타입이 아니라 추상화된 타입 매개변수를 사용한다.
        + ex `ArrayList<E>`
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
3. 1번을 부호함수 fn 으로 정의한다면...
    * [반사성] 두 객체 참조의 순서를 바꿔서 비교해도 예상한 결과가 나와야 한다.
        + fn(x.compareTo(y)) == -fn(y.compareTo(x))
    * [추이성] A가 B보다 크고 B가 C보다 크면 A는 C보다 커야 한다.
        + (x.compareTo(y) > 0) && (y.compareTo(z) > 0) 이면 x.compareTo(z) > 0
    * [대칭성] 크기가 같은 객체들끼리는 어떤 객체와 비교하더라도 항상 같아야 한다.
        + fn(x.compareTo(y)) == 0 이면 fn(x.compareTo(z)) == fn(y.compareTo(z))
    * compareTo의 동치성 테스트는 equals의 동치성 테스트와 결과가 같을 것을 권한다.
        + (x.compareTo(y) == 0) == (x.equals(y)) 
        + **필수는 아니지만 지키지 않을 경우 별도 표기 권고**. 
        + 왜? 정렬된 컬렉션에서 사용시 의도와 다르게 작동할 수 있음. (**정렬된 클래스는 equals가 아니라 compareTo를 사용**함)
        + 지켜지지 않은 예?
            - `var a = new BigDecimal("1.0")`, `var b = new BigDecimal("1.00")`
            - `a.equals(b) == false`
            - `a.compareTo(b) == 0`


## 주의사항
### equals와 유사한 만큼, equals의 주의사항도 따르자.
- 기존 클래스를 확장(extends)하는 경우, 규약을 제대로 지킬 수 없다.
- 대신 독립된 클래스 안에 기존 클래스를 타입으로 갖는 필드를 두라.
- compareTo는 새로 정의된 클래스를 위해 새로 구현하라.

### equals와의 차이점을 주의하자.
- compareTo 메서드의 파라미터 타입은 컴파일타임에 정해지므로, 타입체크/형변환이 불필요하다.
    * 이유? Comparable은 타입을 인수로 받는 제네릭 인터페이스이기 때문
    * ```java
      // 런타임에 ClassCastException이 발생
      Object obj = "Hello";
      Integer num = (Integer) obj;
      System.out.println(num);

      // 컴파일 에러가 발생
      Box<String> box = new Box<>();
      box.set("Hello");
      Integer num = box.get();
      System.out.println(num);
      ```
- null을 파라미터로 받는 경우 NPE를 던져야 한다.
- Comparable을 구현하지 않은 필드나 표준이 아닌 순서로 비교해야 한다면 Comparator를 사용한다.
    * 비교자(Comparator)는 직접 만들거나 자바가 제공하는 것을 사용
    * ex. `String.CASE_INSENSITIVE_ORDER.compare(비교하려는거, 비교하려는거)`
    * 자바8부터 메서드 체이닝 방식으로 비교자를 만들 수 있게 됨 (성능 저하 있음)
      ```java
      private static final Comparator<PhoneNumber> COMPARATOR =
        comparingInt((PhoneNumber pn) -> pn.areaCode)
        .thenComparingInt(pn -> pn.prefix)
        .thenComparingInt(pn -> pn.lineNum);
      ```
    * 수많은 보조 생성 메서드를 제공함
        + 기본 타입 모두 커버 ex. `comparingInt`, `thenComparingInt` 등
        + 객체용 ex. `comparing` `thenComparing`
          ```java
          List<Student> students = Arrays.asList(
                  new Student(2, "Kim", Student.ClassName.A),
                  new Student(3, "Shin", Student.ClassName.B),
                  new Student(3, "Lee", Student.ClassName.C),
                  new Student(2, "Kang", Student.ClassName.C),
                  new Student(1, "Chul", Student.ClassName.A),
                  new Student(1, "Jang", Student.ClassName.B),
                  new Student(3, "Ahn", Student.ClassName.A),
                  new Student(1, "Hawng", Student.ClassName.C),
                  new Student(2, "Lim", Student.ClassName.B)
          );

          // 반 이름 정렬(오름차순)
          Comparator<Student> comparingClassName = Comparator.comparing(Student::getClassName, Comparator.naturalOrder());

          // 나이 정렬(내림차순)
          Comparator<Student> comparingAge = Comparator.comparing(Student::getAge, Comparator.reverseOrder());

          students.stream()
                  .sorted(comparingClassName.thenComparing(comparingAge))
                  .forEach(o -> {
                      System.out.println(o.getAge() + " : " + o.getName() + " : " + o.getClassName());
                  });
        ``` 


### 비교하는 방법에 관한 조언
- 박싱된 기본 타입 클래스의 경우 정적 메서드 compare로 비교하라 (>, < 등 비추, 자바7~)
- 가장 핵심적인 필드부터 비교하라 (0이 아니면 바로 반환)
- 두 값의 차로 비교하지 마라
    * 정수 오버플로를 일으킬 수 있다. ex. `두 값의 차가 int 최대범위보다 큰 경우`
    * 부동소수점 계산 방식에 따른 오류를 일으킬 수 있다.
      ```java
      double a = 0.3;
      double b = 0.1;
      double c = a - b;
      System.out.println(c); // 0.19999999999999998 로 나올 수 있음
      // 부동소수점 수를 근사적으로 표현하는 이진수 형식을 계산에 이용하기 때문

      // cf. 정확한 결과를 보장하기 위해서는
      // (1) 부동소수점 수를 다른 형식으로 변환하거나, 
      // (2) 오차 범위를 고려하는 방법을 사용해야 함

      // (1) BigDecimal은 십진수 연산을 수행하므로 정확한 결과를 보장할 수 있음
      BigDecimal a = new BigDecimal("0.3");
      BigDecimal b = new BigDecimal("0.1");
      BigDecimal c = a.subtract(b);
      System.out.println(c); // 출력: 0.2

      // (2) 오차 범위(epsilon)를 정의하여 실제 결과와 비교
      double a = 0.3;
      double b = 0.1;
      double c = a - b;
      double epsilon = 0.0001; // 오차 범위
      boolean isEqual = Math.abs(c - 0.2) <= epsilon;
      System.out.println(isEqual); // 출력: true
      ```
    * 정석 비교보다 빠른 것도 아니다.