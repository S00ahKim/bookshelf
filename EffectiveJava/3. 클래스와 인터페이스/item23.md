# 태그 달린 클래스보다는 클래스 계층구조를 활용하라

## 태그 달린 클래스란
```java
class Figure {
  enum Shape { RECTANGLE, CIRCLE };

  final Shape shape;
  double length;
  double width;
  double radius;

  Figure(double radius) {
      this.shape = Shape.CIRCLE;
      this.radius = radius;
  }

  Figure(double length, double width) {
      this.shape = Shape.RECTANGLE;
      this.length = length;
      this.width = width;
  }

  double area() {
      switch (shape) {
          case RECTANGLE:
              return length * width;
          case CIRCLE:
              return Math.PI * (radius * radius);
          default:
              throw new AssertionError(shape);
      }
  }
}
```
- 두 가지 이상의 의미를 표현할 수 있으며, 현재 표현하는 의미는 특정 필드의 값으로 알려주는 클래스
- 태그 달린 클래스는 장황하고 오류 내기 쉽고 비효율적이다
    1. 쓸데없는 코드가 많음 (enum 타입 선언, 태그 필드, switch 문)
    2. 가독성 나쁨 (여러 구현이 한 클래스에 있음)
    3. 메모리를 많이 사용함 (다른 의미를 위한 데이터도 언제나 포함됨)
    4. final로 필드를 선언하려면 의미 없는 필드도 초기화해야 함
    5. 다른 의미를 추가하려면 변경점이 많고, 누락될 경우 런타임 에러 발생
    6. 인스턴스의 타입만으로 나타내려는 바를 명확히 표현 불가


## 클래스 계층구조로 변경하는 방법
1. 계층구조의 루트가 될 추상 클래스 정의
2. 태그 값에 따라 동작이 달라지는 메서드를 루트 클래스 안에 추상 메서드로 선언 (ex. `area()`)
3. 태그 값에 상관없이 동작이 일정한 메서드들을 루트 클래스에 일반 메서드로 추가
4. 모든 하위 클래스에 사용하는 필드 루트 클래스에 추가
5. 루트 클래스를 확장한 구체 클래스를 의미별로 하나씩 정의함 (ex. `class Circle`, `class Rectangle`)