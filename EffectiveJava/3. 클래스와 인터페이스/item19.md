# 상속을 고려해 설계하고 문서화하라. 그러지 않았다면 상속을 금지하라.

## 상속을 고려한 문서화
- 재정의할 수 있는 메서드를 내부에서 어떻게 이용하는지 문서화
    * 호출하는 메서드
    * 호출 순서
    * 호출 결과가 처리에 영향을 주는 바
    * 호출 가능한 상황들
- 메서드 주석에 `@impleSpec` 태그를 붙여 자바독이 생성해주는 절 뒤에 내부 동작 설명 붙이기
    ```java
    /**
      This implementation iterates over the collection looking for the specified element. If it finds the element, it removes the element from the collection using the iterator's remove method.

      Note that this implementation throws an UnsupportedOperationException if the iterator returned by this collection's iterator method does not implement the remove method and this collection contains the specified object.
     * /
    ```
    * 이런 식의 문서화는 `'어떻게'를 설명하지 말고 '무엇'을 설명하라`는 격언과 대치되나, 상속이 캡슐화를 해치기 때문에 어쩔 수 없는 부분
    * 해당 태그는 선택 사항이기 때문에 활성화하려면 명령줄 매개변수로 지정해주자
- 일단 문서화한 것은 클래스가 쓰이는 한 반드시 지켜야 함. 그렇지 않으면 하위 클래스의 오동작 가능성 있음.


## 상속을 고려한 설계
- 효율적인 하위 클래스를 큰 어려움 없이 만들기 위해 **클래스의 내부 동작 과정 중간에 끼어들 수 있는 훅(hook)을 잘 선별하여 protected 메서드 형태로 공개해야 할 수도** 있다
    * ex. `java.util.AbstractList` 의 `removeRange()`
    * ㄴ 하위 클래스에서 부분리스트의 clear 메소드를 고성능으로 만들기 쉽게 하기 위해 제공
    * 무엇을 protected로 노출해야 하는지는 쉽게 결정할 수 없으며, 하위 클래스를 직접 작성해서 필요성을 확인하는 방법 뿐
    * 하위 클래스를 작성하다보면 어떤 메서드가 protected로 필요한지, 어떤 메서드가 protected로 제공되었으나 사실은 private이었어야 하는지 알 수 있게 됨
- 상속용 클래스의 **생성자는 직/간접적으로 재정의 가능 메서드를 호출하면 안 된다**
    * 상위 클래스 생성자가 하위 클래스 생성자보다 먼저 실행되기 때문
    * 하위 클래스의 초기화에 의존하고 있다면 NPE 발생 위험
    * private, final, static 메서드는 재정의가 불가능하니 생성자에서 안심하고 호출해도 됨


## 상속용으로 설계하지 않은 클래스는 상속을 금지하라
- 명백한 경우를 말하는 것이 아님
    * 허용하는 것이 정당함: 추상 클래스나 인터페이스의 골격 구현 (item20)
    * 허용하지 않아야만 함: 불변 클래스 (item17)
- 방법1) 클래스를 final로 선언
- 방법2) 모든 생성자를 private 또는 package-private으로 선언하고 public 정적 팩터리를 만듦 (item17)
- 대안
    * 핵심 기능을 정의한 인터페이스를 구현하는 방법 적극 활용 ex. `Set`, `List`, `Map`
    * 래퍼 클래스 (item18)
- 그래도 허용하고 싶다면
    * 클래스 내부에서 재정의 가능 메서드를 사용하지 않도록 하고 문서화하기