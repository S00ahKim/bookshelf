# 로 타입은 사용하지 말라

## 제네릭
- 클래스와 인터페이스 선언에 **타입 매개변수**(type parameter)가 쓰이면, 이를 (제네릭 클래스 혹은 제네릭 인터페이스) = `제네릭 타입`이라 한다. 
- 제네릭 타입은 **매개변수화 타입**(parameterized type)을 정의함
    * `List<String>`은 매개변수화 타입
    * `String`은 실제 타입


## 로 타입
- 제네릭 타입을 하나 정의하면 함께 정의되는 타입
    * `List`는 로 타입
- 제네릭 도입 이전 코드와 호환을 위함
    * 로 타입으로 정의된 경우, 에러를 쉽게 찾아낼 수 없음
    * ```java
      //예시 22-1. Generic 도래하기 전
      List numbers = new ArrayList<>();//Row타입으로 선언
      numbers.add("String");
      numbers.add(9);
      for(Object number:numbers){
          System.out.println(number);// 여전히 동작하지만 좋은 예가 아님. 
          System.out.println((Integer)number);//오류 반환
      }

      //예시 22-2. Generic등장이후
      List<Integer> numbers2 = new ArrayList<>();//타입을 정의하여 선언
      numbers2.add(10);
      numbers2.add("오류"); // Integer타입에 String을 add하는 순간 오류 발생
      ```
    * 오류는 가능한 한 발생 즉시 발견하는 것이 좋으므로 타입을 정확하게 정의하여 사용하는 것이 바람직함
    * 위 예시에서는 런타임시점에나 오류를 발견가능하기 때문에 오류가 발생되는 위치를 정확히 알아내기 어려움


## Row Type을 사용한다면 제네릭이 안겨주는 안전성과 표현력을 모두 잃게 되니 쓰지 마라!!
- 여러 타입을 쓰고 싶은데...
    * `List<Object>`는 괜찮다: 모든 타입을 허용한다는 의사를 컴파일러에게 명확히 전달한 것
    * 비한정적 와일드카드 `?`는 괜찮다: 타입을 신경쓰기도 싫고 몰라도 된다고 표현한 것
        + 제네릭과 마찬가지로...
            1. Object에 정의된 기능만 쓸 것이고
            2. 타입이 뭐가 오든 무관하며
            3. List 인터페이스에 정의한 기능을 사용한다는 의미를 전달함
        + 그러나 List<?>는 타입 파라미터와 관련된 기능(add/addAll)을 쓸 수 없다. 따라서 null 외에는 아무것도 추가할 수 없다.
- 예외
    * class 리터럴에는 로 타입을 써야 한다. `User<String>.class; (사용블가)`
    * 어차피 소거되므로 instanceof 연산자 뒤에는 로 타입을 쓰자.