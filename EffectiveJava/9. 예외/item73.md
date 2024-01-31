# 추상화 수준에 맞는 예외를 던져라

## 수행하려는 일과 무관한 예외를 발생시키지 말라
- 저수준 예외를 처리하지 않고 전파할 경우 발생
- 당황스럽기도 하지만, 내부 구현을 드러내어 API가 오염됨
- 상위 계층에서 저수준 예외를 잡아서 **자기의 추상화 수준에 맞는** 예외로 바꿔(`예외 번역`) 던질 것


## 예외 번역 방법
```java
// 예외 번역 예시
// AbstractSequentialList에서 수행하는 예외 번역의 예
// List의 추상 골격 클래스(아이템 20 참조)
//이 예에서 수행한 예외 번역은 List<E> 인터페이스의 get() 메서드 명세에 명시된 필수 사항임

public abstract class AbstractSequentialList<E> extends AbstractList<E> {


    /**
     * Returns the element at the specified position in this list.
     *
     * @param index index of the element to return
     * @return the element at the specified position in this list
     * @throws IndexOutOfBoundsException if the index is out of range
     *         (index < 0 || index >= size())
     */
    public E get(int index) {
        try {
            return listIterator(index).next();
					// 저수준 추상화를 이용해서
        } catch (NoSuchElementException exc) {
					// 상위 계층에서 추상화 수준에 맞는 예외로 던진다
            throw new IndexOutOfBoundsException("Index: " + index);
        }
    }
}
```


## 예외 연쇄(exception chaining)
```java
try {
  // 저수준 추상화를 이용
} catch (LowerLevelException cause) {
  // 저수준 예외를 고수준 예외에 실어 보낸다(cause)
  throw new HigherLevelException(cause);
}

// 예외 연쇄용 생성자
class HigherLevelException extends Exception {
  HigherLevelException(Throwable cause) {
    super(cause);
  }
}
// 최종적으로 Throwable(Throwable) 생성자까지 cause가 전달됨
```
- 예외 번역 시 저수준 예외가 디버깅에 도움이 되면 **예외 연쇄**를 사용하는 것이 좋다
- 문제의 **근본 원인(cause)**인 **저수준 예외**를 **고수준 예외**에 실어 보내는 방식
- 별도의 접근자 메서드(Throwable의 `getCause` 메서드)를 통해 언제든 저수준 예외를 꺼내 볼 수 있다
- 대부분 표준 예외는 예외 연쇄용 생성자를 갖추고 있다
- 그렇지 않은 예외라고 해도 Throwable의 `initCause` 메서드를 이용해 원인을 직접 못박을 수 있다


## 예외 번역 남용은 금물
- **가능하면 저수준 메서드가 반드시 성공하도록 해서 아래 계층에서는 예외가 발생하지 않도록 하는 것이 최선**
    * ex. 상위 계층 메서드의 매개변수 값을 아래 계층 메서드로 건네기 전에 미리 검사
- 피할 수 없다면 상위에서 조용히 처리해서 문제를 호출자에게 전파하지 말고, 로그만 남기는 방식도 좋음
    * 프로그래머는 로그를 분석해 추가 조치를 취할 것