# public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라

## 가변 필드를 직접 노출하지 말자
```java
// 이러지 말자!
public class Point{
  public double x;
  public double y;
}

// 이렇게 하자!
public Class Point{
  private double x;
  private double y;

  public Point(double x, double y){
    this.x = x;
    this.y = y;
  }

  public double getX() { return x; }
  public double getY() { return y; }

  public double setX() {this.x = x; }
  public double setY() {this.y = y; }
}
```
- 데이터 필드에 직접 접근할 수 있으니 캡슐화의 이점을 제공하지 못함 (item15)
- API를 수정하지 않고는 내부 표현을 바꿀 수 없음
    * getX()를 제공하면 x의 이름을 변경할 수 있음
- 불변식을 보장할 수 없음
    * 외부에서 마음대로 값을 변경할 수 없으므로 데이터 불변을 보장함
- 외부에서 필드에 접근할 때 부수 작업을 수행할 수도 없음
    * getter/setter에서 유효성 검증, 값 보정 등을 해줄 수 있음
- 지켜지지 않은 예: `Point` `Dimension` (item67, 성능 문제가 지속되는 원인)

## 불변 필드는 덜 위험하지만 웬만해서는 직접 노출하지 말자
- 여전히 API를 수정하지 않고는 내부 표현을 바꿀 수 없음
- 불변식은 보장할 수 있음

## package-private 클래스 혹은 private 중첩 클래스라면 필드 노출이 나을 때도 있다
- 해당 접근제어자가 클래스에 붙어 있다면, 작성자가 모르는 사용이 일어날 일이 없다.
- 코드가 깔끔해진다는 장점이 있어서 이 경우에 한해 사용해도 좋다.
- 그러나 최근의 추세는 모든 클래스에 대해 필드 노출을 막는 방향이라고 한다.