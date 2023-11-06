# @Override 애너테이션을 일관되게 사용하라

## 언제 사용하는가?
### 1. 상위 클래스의 메서드를 재정의할 때
```java
@Override
public boolean equals(Object o) {
    if (this == o) return true;
    if (o == null || getClass() != o.getClass()) return false;
    Bigram bigram = (Bigram) o;
    return first == bigram.first &&
            second == bigram.second;
}
```
### 2. 인터페이스의 메서드를 재정의할 때
```java
public interface itemInterface {
    public void testItem();
}

public class ItemClass implements itemInterface {
    @Override
    public void testItem() {
        System.out.println("인터페이스 재정의");
    }
}
```


## 장점
```java
@Override
public boolean equals(Bigram b){
    return b.first == first && b.second==second;
}
----
=> Method does not override method from its superclass
```
- 실수로 오버로딩하는 것을 컴파일러 단에서 막아준다.
- 구체 클래스에서 상위 클래스의 추상 메서드를 재정의할 때에는 안 달아도 상관없지만, 그냥 웬만해서는 다 달자.