# 변경 가능성을 최소화하라

## 불변 클래스
- 인스턴스의 내부 값을 수정할 수 없는 클래스
- 불변 인스턴스에 간직된 정보는 고정되어 객체가 파괴되는 순간까지 절대 달라지지 않음
- 가변 클래스보다 설계하고 구현하고 사용하기 쉬우며, 오류가 생길 여지도 적고 훨씬 안전함
- ex. String, 기본타입의 박싱된 클래스들, BigInteger, BigDemical


## 클래스를 불변으로 만드는 5개 규칙
```java
public final class Person{//2.클래스를 확장할 수 없도록 한다.
  private final Address address; //3.모든 필드를 final로 선언한다. // 4. 모든 필드를 private으로 선언한다.
  
  public Person(Address address){
    this.address = address;
  }
  
  /* 5. 자신 외에는 내부의 가변 컴포넌트에 접근할 수 없도록 한다. */
  public Address getAddress(){
    /* 내부 가변 컴포넌트에 접근하게 한 경우 */
    //return address;
    /* 접근할 수 없도록 방어적 복사 */
    Address copyOfAddress = new Address();
    copyOfAddress.setStreet(address.getStreet());
    copyOfAddress.setAipCode(address.getZipCode());
    return new Address();
  }
  
  public static void main(String[] args){
    /* 내부 가변 컴포넌트에 접근하게 한 경우 */
    Adress seattle = new Address();
    seattle.setCity("Seattle");
    
    Person person = new Person();
    
    Address redmond = person.getAddress();
    redmond.setCity("Redmond");
    system.out.println(person.address.getCity());
    // 접근하게 한경우 : Redmond / 접근할 수 없도록 한 경우 : Seatle
  }
}

public class Address{
  private String zipCode;
  private String street;
  
  public String getZupCode(){
    return zipCode;
  }
  
  public setZipCode(String zipCode){
    this.zipCode = zipCode;
  }
  
  public String getStreet(){
    return street;
  }
  
  public setStreet(String street){
    this.street = street;
  }
}
```
1. 객체의 상태를 변경하는 메서드(변경자)를 제공하지 않는다.
2. 클래스를 확장할 수 없도록 한다.
    - ex. 클래스를 final로 선언
3. 모든 필드를 final로 선언한다.
    - 동기화 없이 다른 스레드에 인스턴스를 보내도 문제 없이 동작할 수 있게 해줌
4. 모든 필드를 private으로 선언한다.
    - public final도 불변이긴 한데, 내부 표현 변경의 자유로움에 private이 더 좋음
5. 자신 외에는 내부의 가변 컴포넌트에 접근할 수 없도록 한다.
    - 생성자, 접근자, readObject 메서드(item88) 모두에서 방어적 복사를 수행할 것
        * 방어적 복사 = 객체의 주소값이 아니라 내부 값을 참조하여 복사하는 방법
        * readObject = 매개변수로 바이트 스트림을 받는 생성자

### +a
- private 또는 package-private 생성자 + 정적 팩터리
    ```java
    public class Complex{
      private final double re;
      private final double im;
      
      private Complex(double re, double im){
        this.re = re;
        this.im = im;
      }
      
      public static Complex valueOf(double re, double im){
        return new Complex(re, im);
      }
      
      ... 
    }

    public class ComplexExample{
      Complex complex = Complex.valueOf(1, 0.222);
    }
    ``` 
    * 이런 구조의 이유: **상속하지 못하게 하기 위해**
    * 확장 가능
    * 정적 팩터리를 통해 여러 구현 클래스 중 하나를 활용할 수 있는 유연성을 제공
    * 객체 캐싱 기능으로 성능을 향상
- 오버라이드 가능한 클래스는 방어적인 복사를 사용해야함
    * BigInteger, BigDecimal의 설계 당시에는 불변 객체는 final이어야 한다는 인식이 없어서 오버라이드 가능했음. 그래서 보안이 중요하다면 신뢰할 수 없는 클라이언트로부터의 이 타입은 실제로 그 값인지 확인해야 함.
    * 신뢰할 수 없다면 가변이라 가정하고 방어적으로 복사해야 (item50)
- **다른 합당한 이유가 없다면 모든 필드를 private final로 두라**
- **생성자는 불변식 설정이 모두 완료된 초기화가 완벽하게 끝난 객체를 생성해야** 한다!!
    * 객체 재활용 목적으로 초기화하는 메서드 등을 제공하지 마라. 복잡해지기만 함.


## 불변 클래스의 장점
> 불변으로 만들 수 없는 클래스라도 변경 가능한 부분을 최소화하자
- **함수형 프로그래밍**에 적합함
    * = 피연산자에 함수를 적용해 그 결과를 반환하지만, 피연산자 자체는 그대로인 프로그래밍 패턴
    * 즉, y = f(x) 에서 x가 변경되지는 않는 것
- 불변 객체는 **단순함**
- 불변 객체는 근본적으로 **스레드 안전**하여 따로 동기화할 필요 없음
- 불변 객체는 안심하고 **공유**할 수 있음
    * 가장 쉬운 재활용 방법 : 자주 쓰이는 값들을 상수(public static final)로 제공하는 것
    * 아무리 복사해봐야 원본과 똑같으니 복사 자체가 의미가 없음 (String의 복사 생성자는 초창기의 실수로, 사용 지양)
- 불변 객체 끼리는 내부 데이터를 공유할 수 있음
- 객체를 만들 때 불변 객체로 구성하면 이점이 많음 (**구조가 복잡해도 불변식 유지**)
- **실패 원자성을 제공**(item76)하여 잠깐의 불일치 상태도 발생하지 않음
    * = 메서드에서 예외가 발생한 후에도 그 객체는 여전히 메서드 호출 전과 같이 유효한 상태여야 한다


## 불변 클래스의 단점
- 값이 다르다면 반드시 별도의 객체로 만들어야 한다
    * 해결책(1) 다단계 연산 제공
    * 해결책(2) 가변 동반 클래스 제공
- 특정 상황에서 잠재적 성능 저하