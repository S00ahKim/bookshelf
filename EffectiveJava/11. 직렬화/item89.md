# 인스턴스 수를 통제해야 한다면 readResolve보다는 열거 타입을 사용하라

## 싱글턴 패턴의 직렬화
```java
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() { }

    public void leaveTheBuilding() { ... }
}
```
- 이 클래스는 바깥에서 생성자를 호출하지 못하게 막는 방식으로, 인스턴스가 오직 하나만 만들어짐을 보장
- 이 클래스는 선언에 implements Serializable을 추가하는 순간 더이상 싱글턴이 아니게된다.(item3)
    * 기본 직렬화(item87)을 쓰지 않고, 명시적인 readObject(item88)을 사용해도 소용없다.
    * writeObject()를 제공하더라도 소용이 없다.


## readResolve
```java
private Object readResolve() {
    // 기존에 생성된 인스턴스를 반환한다.
    return INSTANCE;
}
```
- readObject가 만들어낸 인스턴스를 받아 호출되는 함수
    * 이때 readObject에서 어떤 객체를 만들었는지와 관계없이 readResolve가 만든 객체로 대체되어 반환.
    * 이때 readObject 가 만들어낸 인스턴스는 가비지 컬렉션의 대상이 된다.
- Elvis 인스턴스의 직렬화 형태는 아무런 실 데이터를 가질 필요가 없으니 모든 인스턴스 필드는 transient 로 선언해야 함
    * readResolve 메서드를 인스턴스의 통제 목적으로 이용한다면 모든 필드는 transient로 선언해야 함
    * 그렇지 않으면 역직렬화 과정에서 역직렬화된 인스턴스를 가져올 수 있어 싱글턴이 깨짐


## 해결책은 enum
```java
public enum Elvis {
    INSTANCE;
    
    ...필요한 데이터들
}
```
- 자바가 선언한 상수 외에 다른 객체가 없음을 보장
- AccessibleObject.setAccessible같은 특권(privileged) 메서드를 악용하면 뚫릴 수는 있는데, 애초에 네이티브 코드 수행 특권을 가로챈 공격자에겐 모든 방어가 무력화되니 논외


## 그래도 readResolve를 쓴다면
- 직렬화 가능 인스턴스 통제 클래스를 작성해야 할 때, 컴파일 타임에는 어떤 인스턴스들이 있는지 모를 수 있으므로 사용해야만 함
- 반드시 private여야 한다.
- 만약 public/protected면 반드시 하위 클래스에서 재정의해야 한다.
    * 안 그러면 하위 클래스 인스턴스를 역직렬화할 때 상위 클래스의 인스턴스를 생성해 ClassCastException을 일으킬 수 있다.