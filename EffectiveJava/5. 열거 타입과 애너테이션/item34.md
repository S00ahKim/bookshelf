# 상수 대신 열거 타입을 사용하라
> 열거 타입: enum, 자바5~


## (이전의 예) 정수 열거 패턴
```java
// 사과용 상수 이름은 모두 APPLE_로 시작, 오렌지용 상수는 ORANGE_로 시작
public static final int APPLE_FUJI = 0;
public static final int APPLE_PIPPIN = 1;
public static final int APPLE_SMITH = 2;

public static final int ORANGE_NAVEL = 0;
public static final int ORANGE_TEMPLE = 1;
public static final int ORANGE_BLOOD = 2;
```
- 단점(1): 타입 안전을 보장할 방법이 없으며 표현력이 좋지 않다.
    * 자바가 정수 열거 패턴을 위한 별도 이름 공간을 지원하지 않기 때문에 어쩔 수 없이 접두어(APPLE_, ORANGE_)를 사용해 구분한다. 그러나 **컴파일러 입장에서는 둘다 정수 0**이기 때문에 APPLE_가 사용되는 메소드에 ORANGE_가 전달되어도 오류가 발생하지 않는다.
- 단점(2): 정수 열거 패턴을 사용한 프로그램은 깨지기 쉽다.
    * 평범한 상수를 나열한 것 뿐이라 컴파일하면 해당 값이 클라이언트 파일에 그대로 새겨진다. 따라서 상수의 값이 바뀌면 반드시 다시 컴파일 해야 한다.
- 단점(3): 문자열로 출력하기 까다롭다.
    ```java
    final int APPLE_PIE = 0;
    System.out.println(APPLE_PIE); // APPLIE_PIE 가 아닌 0이 출력!
    ```


## (이전의 예) 문자열 열거 패턴
```java
public static final String APPLE_FUJI           = "apple_fuji";
public static final String APPLE_PIPPIN         = "apple_pippin";
public static final String APPLE_GRANNY_SMITH   = "apple_granny_smith";
```
- 장점: 상수의 의미를 출력할 수 있음
- 단점: 문자값을 그대로 하드코딩 하게 만들어 **그 문자열에 오타**가 있어도 컴파일러가 확인 할 길이 없음 (런타임 버그)


## <대안!!> 열거 타입
```java
public enum Apple {FUJI, PIPPIN, GRANNY_SMITH}
public enum Orange {NAVEL, TEMPLE, BLOOD}
```

### 장점
1. `열거타입`은 컴파일타임 타입 안정성을 제공한다.
    - 만약 **다른 타입의 값을 전달하려면 컴파일 오류**가 발생한다. 
2. `열거타입`에는 각자의 이름공간이 있어서 이름이 같은 상수도 평화롭게 공존한다.
    - 열거 타입에 **새로운 상수를 추가하거나 순서를 바꿔도 다시 컴파일 하지 않아도 된다**.
3. `열거타입`의 toString 메소드는 출력하기에 적합한 문자열을 내어준다.
4. Object 메소드들과 Comparable, Serializable을 구현해두었다.
    - `public abstract class Enum<E extends Enum<E>> implements Constable, Comparable<E>, Serializable`
5. `열거타입`은 근본적으로 불변이라 모든 필드는 final이어야 한다. (item17)
6. `열거 타입`에는 임의의 메소드나 필드를 추가할 수 있고 임의의 인터페이스를 구현하게 할 수도 있다.
    - 열거형에 멤버 추가하기
    ```java
    public enum Apple {
        FUJI, 
        PIPPIN, 
        GRANNY_SMITH;
        
        //정수를 저장할 임의의 필드(인스턴스 변수) 추가
        private final int color;

        Apple(int color) {
            this.color = color;
        }

        // 임의의 메서드 추가
        public void printHi() {
            System.out.println("Color : " + this.name());
        }
    };
    ```
    - 열거형에 추상메서드 추가하기
      ```java
      enum Transportation { //운송수단 종류별 상수정의
          BUS(100)      { int fare(int distance) { return distance*BASIC_FARE;}},
          TRAIN(150)    { int fare(int distance) { return distance*BASIC_FARE;}},
          SHIP(100)     { int fare(int distance) { return distance*BASIC_FARE;}},
          AIRPLANE(300) { int fare(int distance) { return distance*BASIC_FARE;}};

          protected final int BASIC_FARE; // protected로 해야 각 상수에서 접근가능
        
          Transportation(int basicFare) { // private Transportation(int basicFare) {
          BASIC_FARE = basicFare;
        }

          public int getBasicFare() { return BASIC_FARE; }

          abstract int fare(int distance); // 거리에 따른 요금 계산
      }

      class EnumEx3 {
          public static void main(String[] args) {
              System.out.println("bus fare="     +Transportation.BUS.fare(100));
              System.out.println("train fare="   +Transportation.TRAIN.fare(100));
              System.out.println("ship fare="    +Transportation.SHIP.fare(100));
              System.out.println("airplane fare="+Transportation.AIRPLANE.fare(100));
          }
      }
      ```

### 특징
- 열거 타입은 클래스다!
- 상수 하나당 자신의 인스턴스를 하나씩 만들어 public static final 필드로 공개한다
- 밖에서 접근할 수 있는 생성자를 제공하지 않는다 (한개의 인스턴스가 보장)
- 자신 안에 정의된 상수들의 값을 배열에 담아 반환하는 정적 메서드인 values를 제공한다
- 정의된 상수를 하나 제거하면 참조하지 않는 곳이면 문제가 없고, 참조하면 컴파일 오류가 발생한다

### 구현
- 널리 쓰이는 열거타입은 톱레벨 클래스로 생성하고 특정 톱레벨 클래스에서만 쓰인다면 해당 클래스의 멤버클래스화(item24)
- 상수별 메서드 제공: case 문으로 구분하는 식은 누락이 될 수 있다

### 단점
- 열거타입 상수끼리 코드 공유가 어렵다. 예를 들면, 타입이 A, B, C가 있다면 A-B는 (가)로 묶을 수 있고 C는 (나)로 묶일 수 있을 때, (가)를 위한 코드를 A, B가 나눠 쓰기 어렵다는 것.
    * [나쁜 해결] 모든 상수들에 중복해서 넣을 수 있다. 그러면 가독성이 나쁘고 오류가 발생하기 쉽다.
    * [나쁜 해결] 기본 메서드를 (나)를 위해 만들고 (가)를 위한 재정의 메소드를 만들 수 있다. 그러면 새로운 경우에 대해 재정의가 누락될 수 있다.
    * [좋은 해결] 전략 패턴을 사용한다. A의 멤버로 (가)를 넣고, (가)에 메서드를 작성한다.