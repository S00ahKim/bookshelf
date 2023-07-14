# clone 재정의는 주의해서 진행하라
> 복잡한 문제가 발생할 수 있으니 새로운 인터페이스를 만들 때에는 Cloneable을 구현하지 말 것!
> 복제 기능은 생성자와 팩터리를 이용하는 게 최고다.

## Cloneable이란
- 복제해도 되는 클래스임을 명시하는 용도의 믹스인 인터페이스
    * 믹스인? OOP에서 상속되지 않고 타 클래스에서 사용 가능한 메서드를 포함하는 클래스. 자바에서는 '인터페이스'. '포함'이라고도 표현함. (item20)
- 한계
    * 믹스인이라기에는 너무 예외적인 용법
    * Cloneable 구현만으로 외부에서 clone()을 호출할 수 없고, 꼭 오버라이드를 해야 한다.
        + clone()은 Cloneable 인터페이스가 아니라 Object에 protected로 선언되어 패키지가 다르면 접근이 안된다.
        + public한 메서드를 따로 제공하지 않는 한 리플렉션(클래스 정보를 알아내는 기법, item65)을 사용하기도 어렵다.
    * 오버라이드할 때 복잡한 구조를 이해하고 있어야 한다.
    * 생성자를 호출하지 않고 객체를 만들 수 있게 되어 불안정하다.
- 역할
    * clone이 동작 가능한지 불가능한지를 판별해줌.
    * Cloneable을 구현 안 했으면 CloneNotSupportedException을 던짐.
      ```java
      // Object.java
      @IntrinsicCandidate // cf. 저수준 구현을 사용한다는 표시
      protected native Object clone() throws CloneNotSupportedException;
      ``` 
      ```cpp
      // JVM C++ 코드. Cloneable 의 서브타입인지 검사하고, 아닐 경우 에러를 던진다.
      bool cloneable = klass->is_subtype_of(SystemDictionary::Cloneable_klass());
      guarantee(cloneable == klass->is_cloneable(), "incorrect cloneable flag");
      ...{중략}...
      if (!klass->is_cloneable()) {
        ResourceMark rm(THREAD);
        THROW_MSG_0(vmSymbols::java_lang_CloneNotSupportedException(), klass->external_name());
      }
      ``` 
    * Cloneable을 구현했으면 해당 객체의 필드를 하나하나 복사해서 리턴함
        + **shallow copy**를 지원함
          ```java
          // Thus, this method performs a "shallow copy" of this object,
          // not a "deep copy" operation.
          ```
        + 얕은 복사? 원본 객체와 복사본 객체가 같은 메모리 주소를 참조함. 참조 복사.
        + 깊은 복사? 원본 객체와 복사본 객체가 다른 메모리 주소를 참조함. 값 복사.
- Cloneable 같은 인터페이스 사용법은 **아주 특이한 경우**니까 따라하지 말자
    * 일반적: 인터페이스가 정의한 기능을 사용하려고 인터페이스를 구현
    * 이례적: 상위 클래스에서 정의된 메서드 동작 방식을 변경하려고 인터페이스를 구현


## Cloneable을 구현하는 법
### clone()의 규약
- 복잡한데 허술하다
- 아래의 수식을 일반적으로 만족하지만, 필수적인 것은 아니다!!
  ```
  x.clone() != x
  x.clone().getClass() == x.getClass()
  x.clone().equals(x)
  ```

### 구현 순서
1. Cloneable 인터페이스를 구현한다.
    ```java
    public class Foo implements Cloneable {...};
    ```
2. clone 메서드를 public으로 선언한다.
3. try-catch 구조를 만든다.
4. try 안에서 super.clone 을 호출한다.
    * clone은 강제성이 없는 생성자 연쇄(this 또는 super 키워드를 사용해서 생성자에서 다른 생성자를 호출하는 기술)와 유사하다. 따라서 clone 안에서 super.clone 이 아니라 생성자를 호출해도 컴파일 에러는 나지 않는다. 그러나 A <- B <- C 의 관계에서, B의 clone()이 new B() 를 호출한다면, C의 clone()은 잘못된 동작을 하게 된다.
    * super.clone을 해줘야 재귀적으로 부모 클래스의 super.clone도 호출해주면서 의도한 복제를 모두 수행할 수 있다.
    * 상속할 일이 없는 경우(final 클래스 등)라면 무시해도 되고, 굳이 Cloneable을 구현할 필요도 없다.
5. 내부 로직에서 적절히 수정해준다.
    * = 객체의 내부 깊은 구조에 숨은 모든 가변 객체를 복사하고, 복제본이 가진 객체 참조 모두가 **원본이 아닌** 복사된 객체를 가리키게 되도록 변경
    * 기본 타입 또는 불변 객체 참조만 갖는 클래스는 따로 변경할 필요 없음
        + `기본 타입`? primitive type. 값을 저장할 때, 데이터의 실제 값이 저장된다. 기본 타입에는 정수형(byte, short, int, long), 실수형(float, double), 문자형(char), 논리형(boolean) 등이 있다.
        + `불변 객체`? 인스턴스 생성 후 내부 상태 변경이 불가능한 객체 (반: `가변 객체`)
        + 단, 객체에 고유ID/일련번호가 있다면 수정 필요
    * clone은 원본 객체에 아무런 해를 끼치지 않고, 복제된 객체도 정상적으로 작동할 수 있도록 불변식(`invariant`)을 보장해야 한다.
        + `불변식`? 어떤 객체가 정상적으로 작동하기 위해 절대 허물어지면 안 되는 값, 식, 상태의 일관성을 보장하기 위해 항상 참이 되는 조건. 즉 **객체의 상태가 프로그래머의 의도에 맞게 잘 정의되어 있다고 판단할 수 있는 기준**을 제공하는 속성.
        + 불변식이 침해되는 예? 배열 elements를 갖는 클래스 스택을 단순히 super.clone 하여 복제하면 원본 스택과 복제된 스택이 같은 배열에 요소를 넣는 불상사가 발생할 수 있다. 그러면 스택의 동작 방식이 프로그래머가 기대한 대로 돌아가지 않을 것이다.
    * 가변 객체가 포함되어 있을 경우에는 적절하게 변경해주어야 함
        + (1) 가변 객체를 갖는 경우
            - **재귀적으로 clone을 수행**해준다. 
            - ex. `cloneStack.elements = elements.clone()` (p.81)
        + (2-1) 가변 객체 안에 가변 객체를 갖는 경우
            - 예를 들어 참조 타입으로 정의된 배열인 경우에는 따로 deepCopy()를 구현해서 **참조 타입의 모든 요소 또한 clone()** 해주는 등의 작업이 필요하다. (p.83) 
            - deepCopy를 구현하는 방법에 따라 성능 이슈가 있을 수 있다.
        + (2-2) 가변 객체 안에 가변 객체를 갖는 경우
            - **고수준 API를 호출**하여 깊은 복사를 수행한다.
            - 저수준보다는 다소 느리며, 아키텍처의 지향점과는 거리가 있는 방식.
    * 가변 객체를 정확히 복제하기 위해서는 final 필드의 한정자를 제거해야 한다
        + final 필드에는 새로운 값을 할당할 수 없기 때문에 위와 같은 재할당이 불가능하기 때문
        + `가변 객체를 참조하는 필드는 final로 선언하라`는 일반 용법과 충돌함 (cf. item17)
        + ㄴ 멀티스레드 환경에서 동기화 문제 해결 가능 & 변경 가능성을 최소화할 수 있기 때문
    * cf. 배열의 clone은 제대로 사용된 유일한 예
        + 런타임 타입 = 컴파일타임 타입 = 원본 배열과 똑같은 배열
        + TODO
6. 클라이언트가 형변환하지 않아도 되도록 리턴 타입을 해당 객체로 공변 반환 타이핑(covariant return typing)한다.
    * 공변? B가 A의 부분집합일 때 고차함수 T에 대해 T<B> 도 T<A> 의 부분집합이 된다.
    * a.k.a. 부모 클래스 메서드를 오버라이딩할 때, 해당 메서드의 리턴 타입은 자식 클래스 타입으로 변경할 수 있다.
    * `Object clone()` 을 `Foo clone()` 으로 정의할 수 있다는 의미
7. catch 안에서 CloneNotSupportedException 을 잡아준다.
    * 왜? Object의 clone이 checked exception을 던지기 때문
    * Cloneable을 구현했기 때문에 발생할 일이 없음
    * cf. CloneNotSupportedException은 unchecked exception이어야 했다는 신호 (item71)

### 예시
```java
// ex. ArrayList
public Object clone() {
    try {
        ArrayList<?> v = (ArrayList<?>) super.clone();
        v.elementData = Arrays.copyOf(elementData, size);
        v.modCount = 0;
        return v;
    } catch (CloneNotSupportedException e) {
        // this shouldn't happen, since we are Cloneable
        throw new InternalError(e);
    }
}
```

### 특히 주의하세요
- clone에서는 재정의될 수 있는 메서드를 호출하면 안 된다
    * Person <- Employee 가 있을 때...
      ```java
      public class GameEntity {
          String nickname;
      }

      public class Person extends GameEntity implements Cloneable {
          public String name;
          public Gender gender;

          public Person(String name, Gender gender) {
              this.name = name;
              this.gender = gender;
          }

          public void setNickname(Gender gender, String name) {
              if (gender.equals(Gender.FEMALE)) {
                  this.nickname = "F_" + name;
              }
          }

          @Override
          public Person clone() {
              try {
                  Person p = (Person) super.clone();
                  p.setNickname(this.gender, this.name);
                  return p;
              } catch (CloneNotSupportedException e) {
                  throw new InternalError(e);
              }
          }
      }

      public class Employee extends Person {

          public Employee(String name, Gender gender) {
              super(name, gender);
          }

          @Override
          public void setNickname(Gender gender, String name) {
              if (gender.equals(Gender.FEMALE)) {
                  this.nickname = "EMP_F_" + name;
              } else {
                  this.nickname = "EMP_" + name;
              }
          }
      }

      public enum Gender {
          MALE, FEMALE
      }

      @Test
      void test() {
          Employee sooah = new Employee("수아", Gender.FEMALE);
          System.out.println(sooah.nickname);
          // ㄴ null
          
          var cloneSooah = sooah.clone();
          System.out.println(cloneSooah.nickname);
          // ㄴ 아직 setNickName을 하지 않았음에도 EMP_F_수아 가 나온다.
          // 이처럼, 복제 과정에서 자신의 상태를 교정할 기회를 잃게 된다.
      }
      ``` 
    * 유사: 생성자에서는 재정의될 수 있는 메서드를 호출하면 안 된다 (item19)
- 오버라이드한 clone()에서 throws 절을 없애라
    * Cloneable을 구현하는 법 > 구현 순서 > try-catch 구조를 만든다.
    * checked exception이 없어야 메서드를 사용하기 편리하다 (item71)
- 상속하기 위한 클래스에서는 Cloneable을 구현하지 마라
    * (1) 구현 여부를 하위 클래스가 선택하도록 함
        + Object의 방식 모방
        + 잘 작동하는 protected clone을 두고, CloneNotSupportedException을 던질 수 있다고 선언
    * (2) 구현하지 못하게 함
        + clone을 아예 퇴화시킴
        + ```java
          @Override
          protected final Object clone() throws CloneNotSupportedException {
              throw new CloneNotSupportedException();
          }
          ```
- Cloneable을 구현한 쓰레드 세이프 클래스를 작성했다면 clone 메서드도 동기화해야한다
    * Object의 clone은 동기화를 고려하지 않았기 때문
    * 쓰레드 세이프 클래스? 자원을 공유해도 연산 결과에 영향이 없는 클래스


## 대안: 복사 생성자, 복사 팩터리
- 복사 생성자
    ```java
    public Foo(Foo foo) {...};
    ```
- 복사 팩터리
    ```java
    public static Foo newInstance(Foo foo) {...};
    ```
- 장점
    * 복잡하지 않음
    * 엉성하지 않음
    * final 필드 사용 가능
    * 불필요한 checked exception을 던지지 않음
    * 불필요한 형변환 없음
    * 파라미터로 클래스의 인터페이스를 구현한 다른 클래스를 받을 수 있음
        + 인터페이스 기반 복사 생성자(conversion constructer, 변환 생성자) & 복사 팩터리(conversion factory, 변환 팩터리)
        + 원본 구현 타입에 얽매이지 않고 복제본 타입을 직접 선택할 수 있음
        ```java
        // 변환 생성자
        HashSet<Integer> hashSet = new HashSet<>();
        TreeSet<Integer> treeSet = new TreeSet<>(hashSet);
        ``` 