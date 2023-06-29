# private 생성자나 열거 타입으로 싱글턴임을 보장하라

## 싱글턴
- 인스턴스를 오직 하나만 생성할 수 있는 클래스
- 장점
    * 함수 같은 stateless 객체(item24)나, 설계상 유일해야 하는 시스템 컴포넌트 등을 만들 수 있음
    * 최초 1회의 생성자 호출 이후 고정 메모리 영역을 사용하여 추후 해당 객체 접근시 메모리 낭비를 막고, 속도 이득을 얻음
- 단점: 인터페이스의 구현체가 아닌 싱글턴을 사용할 경우, Mock 구현을 사용할 수 없어 사용처 코드를 테스트하기 어려움

## 싱글턴 만들기
1. **public static 멤버가 final 필드인 방식**
    - ex. 
      ```java
      public static final Poppy POPPY = new Poppy(); // 유일한 인스턴스에 접근할 수 있는 public static 멤버를 마련
      private Poppy() { ... } // 생성자는 private으로 두고
      ```
    - 해당 클래스가 싱글턴임이 API에 드러나고(ex. `Poppy.POPPY`), 간결하다.
    - 두번째 방식이 제공하는 장점이 불필요할 경우 이 방식을 사용하는 것이 낫다.
2. **정적 팩터리 메서드를 public static 멤버로 제공하는 방식**
    - ex. 
      ```java
      private static final Poppy POPPY = new Poppy()`;
      private Poppy() { ... }     // 생성자는 private으로 두고
      public static Poppy getInstance() { return POPPY; } // 유일한 인스턴스에 접근할 수 있는 public static 멤버를 마련
      ```
    - 장점
        * 필요할 경우, API를 변경하지 않아도 싱글턴이 아니게 할 수 있다. (ex. getInstance 로직 수정)
        * 필요할 경우, 정적 팩터리를 제네릭 싱글턴 팩터리로 만들 수 있다. (item30)
            + `제네릭 싱글턴 팩터리`? 제네릭으로 타입설정 가능한 인스턴스를 만들어두고, 제네릭으로 받은 타입으로 리턴 타입을 결정하는 방식
            + ex.
              ```java
              public class GenericFactoryMethod {
                  public static final Set EMPTY_SET = new HashSet();

                  public static final <T> Set<T> emptySet() {
                      return (Set<T>) EMPTY_SET;
                  }
              }
              ```
        * 정적 팩터리의 메서드 참조를 공급자로 사용할 수 있다. (item43, item44)
            + `메서드 참조`? 람다식에서 메서드 하나만 호출할 때 불필요한 파라미터 명시를 하지 않는 것. ex. 클래스이름::메소드이름
            + `공급자`? 함수형 인터페이스의 일종으로 파라미터 없이 값을 리턴하는 메서드. 자세한 것은 [여기](#자바-공급자)에
            + ex. `Poppy::getInstance`를 `Supplier<Poppy>`로 사용
3. **원소가 하나인 enum 타입을 선언한다.**
    - ex.
      ```java
      public enum Poppy {
        POPPY;
      }
      ```
    - 장점: 첫번째, 두번째 방법보다 간결하고, 리플렉션 공격을 막아주며, 직렬화도 간단하다.
    - 단점: 조금 부자연스러워 보일 수 있고, 다른 클래스를 상속하지는 못한다.

## private 생성자로 만드는 싱글턴의 단점
- 방법 1, 2
- 테스트하기 위해 리플렉션 API로 직접 생성자 호출할 수 있는데, 이를 막기 위해 예외처리를 하기도 한다.
- 직렬화할 때 단순히 Sirializable 을 구현한다고 선언하는 것만으로는 부족하다. 아래와 같이 하지 않으면 직렬화된 인스턴스를 역직렬화할때마다 새로운 인스턴스가 만들어진다.
    * 모든 인스턴스 필드를 transient(일시적) 이라고 선언
    * readResolve 메서드를 제공해야 함 (싱글턴 객체를 리턴하도록 하고, 생성된 객체는 가비지 컬렉터에 맡김) (item89)

## 자바 공급자
> Supplier<T>
- Supplier 인터페이스
  ```java
  @FunctionalInterface
  public interface Supplier<T> {
      T get();
  }
  ```
- 사용예1. 값 생성
  ```java
  Supplier<Integer> randomNumberSupplier = () -> {
      Random random = new Random();
      return random.nextInt(100);
  };

  int randomNumber = randomNumberSupplier.get();
  ```
- 사용예2. 지연된 계산 (값을 계산하는 작업을 나중에 수행해야 하는 경우)
  ```java
  Supplier<Double> expensiveCalculation = () -> {
      // 복잡한 계산 작업 수행
      return result;
  };

  // 필요한 시점에서 계산을 수행
  Double result = expensiveCalculation.get();
  ```
- 사용예3. 외부에서 값을 제공하는 경우 (외부 리소스나 메서드 등)
  ```java
  Supplier<InputStream> fileStreamSupplier = () -> {
      try {
          return new FileInputStream("myfile.txt");
      } catch (FileNotFoundException e) {
          throw new RuntimeException("File not found", e);
      }
  };

  InputStream fileStream = fileStreamSupplier.get();
  ```
- 왜 사용하는가?
    * 코드의 유연성과 재사용성을 높일 수 있음
    * 람다식과 함께 사용하면 간결하고 읽기 쉬운 코드를 작성할 수 있음