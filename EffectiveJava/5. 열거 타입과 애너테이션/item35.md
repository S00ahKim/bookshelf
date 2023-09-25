# ordinal 메서드 대신 인스턴스 필드를 사용하라

## ordinal 메서드란?
> JAVA의 enum 클래스에서 기본적으로 내장되어 있는 메서드
```java
public enum Week {
	Monday,
	Tuesday,
	Wednesday,
	Thursday,
	Friday,
	Saturday,
	Sunday
}
```
- `name()` : String 타입을 리턴하며, 열거 객체의 문자열을 리턴한다
    ```java
    Week today = Week.Sunday;
    String name = today.name(); // “Sunday”
    ```
- `ordinal()` : int 타입을 리턴하며, 열거 객체의 순번(0부터 시작)을 리턴
    ```java
    Week today = Week.Sunday;
    int ordinal = today.ordinal(); // 6
    ```
- `compareTo()` : int 타입을 리턴하며, 열거 객체를 비교해서 순번 차이를 리턴한다
    ```java
    Week day1 = Week.Monday;
    Week day2 = Week.Wednesday;
    int result1 = day1.compareTo(day2); // -2 리턴
    int result2 = day2.compareTo(day1); // 2 리턴
    ```
- `valueOf(String name)` : 주어진 문자열의 열거 객체를 리턴한다
    ```java
    Week weekDay = Week.valueOf("Monday"); // Week.Monday
    ```
- `values()` : 모든 열거 객체들을 배열로 리턴한다
    ```java
    Week[] days = Week.values();
    ```


## 문제점
- 편하니까 쓰고 싶긴 하지만...
- 열거 타입 순서를 지키면서 정의해야 한다
    * 상수 선언 순서를 바꾸는 순간 오동작함
    * 이미 사용 중인 정수와 값이 같은 상수는 추가할 방법이 없음
- 값을 중간에 비워둘 수 없다
    * 따라서 더미를 추가해야 한다


## 해결책
- 열거 타입 상수에 연결된 값은 ordinal 메서드로 얻지 말고, 인스턴스 필드에 값을 지정, 저장하자
- 문서: 대부분의 프로그래머는 쓸 일이 없는 메서드다. 범용 자료구조를 위해 만들어졌다.