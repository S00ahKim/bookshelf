# 톱레벨 클래스는 한 파일에 하나만 담으라

## 톱레벨 클래스
- 중첩 클래스가 아닌 클래스
- 즉, 다른 클래스나 인터페이스 내부에 선언되지 않은 클래스


## 톱레벨 클래스가 여러 개라면 문제가 발생할 수 있다
```java
// Foo 라는 파일에 아래 클래스가 정의된다면
class Utensil {
    static final String NAME="pan";
}

class Dessert {
    static final String NAME="cake";
}

// Utensil, Dessert 클래스 각각이 컴파일된다
```
- 소스 파일 하나에 톱레벨 클래스를 여러 개 선언하는것은 아무런 득도 없고 심각한 위험을 감수해야 한다
    - 한 클래스를 여러 가지로 정의할 수 있게 되고,
    - 그중 어느 것을 사용할지는 어느 소스 파일을 먼저 컴파일하냐에 따라 달라지기 때문 (컴파일러에 어느 소스 파일을 먼저 건네느냐에 따라 동작이 달라짐)


## 그러니 톱레벨 클래스들은 서로 다른 소스 파일로 분리하자
- 굳이 여러 톱레벨 클래스를 한 파일에 담고 싶다면 정적 멤버 클래스(item24)를 사용