# 제네릭과 가변인수를 함께 쓸 때는 신중하라

## 가변인수 varargs
- item53
- 자바5~
- 메서드에 넘기는 인수의 개수를 클라이언트가 조절할 수 있게 함
    ```java
    // 예: 파라미터로 들어온 모든 문자열을 출력하는 메서드
    void printAll(String...str) {
        for(String a:str)
            System.out.println(a);
    }
    ```
- 구현 방식: 가변 인수를 담기 위한 배열이 자동으로 하나 만들어짐 (예: str[])
    * 단점!! 감춰졌어야 했는데, 클라이언트에 노출함
    * 가변인수에 제네릭/타입이 포함되면 인지하기 어려운 컴파일 경고가 발생함
        + 실체화 불가 타입으로 varargs를 선언하면 경고를 보냄
        + 제네릭은 실체화 불가 타입(런타임에 타입이 소거되는 경우, item28) 
        + Possible heap pollution ... (힙 오염 발생 가능)
        + 타입이 다른 객체를 참조할 경우 자동 형변환에 실패하여 타입 안정성이 깨짐
          ```java
          static void dangerous(List<String>... stringLists) {
              List<Integer> intList = List.of(42);
              Object[] objects = stringLists;
              objects[0] = intList; // 힙 오염 발생
              String s = stringLists[0].get(0); // ClassCastException
              // 형변환하는 곳이 없어 보이지만, 컴파일러가 이 마지막 줄에서 형변환함!
          }
          ```
    * 단점이 있을 걸 알면서 왜 이렇게 구현했나? 경고 내지 말고 에러를 내야 하는 거 아니야?
        + 실무에서 유용하다...
        + 문제 없이 타입 안전하게 잘 구현된 예시들
            1. `Arrays.asList(T... a)`
            2. `Collections.addAll(Collection<? super T> c, T...elements)`
            3. `EnumSet.of(E first, E... rest)`


## 제네릭 가변인수 안전하게 작성하기
- 이 메서드는 타입 안전하다! 라고 보장해준다면 컴파일러는 더 이상 경고를 뱉지 않는다
    * by `@SafeVarargs` (자바7~)
    * 그 전에는 그냥 두거나 @SuppressWarnings를 붙여서 다른 경고도 숨겨버림
- 무슨 메서드가 타입 안전할까? 원래 의도대로 순수하게 파라미터를 **전달**하는 데에만 쓰인다면!
    1. 메서드가 varargs 배열에 아무것도 저장하지 않는다 (덮어쓰지 않는다)
    2. 배열의 참조가 밖으로 노출되지 않는다 (신뢰할 수 없는 코드가 접근할 일이 없다)
    3. cf. @SafeVarargs 는 오버라이드할 수 없는 메서드(static/final/private)에만 붙일 것
- 안전하지 않은 예시
    ```java
    static <T> T[] pickTwo(T a, T b, T c) {
        switch (ThreadLocalRandom.current().nextInt(3)) {
            case 0: return toArray(a, b);
            case 1: return toArray(a, c); 
            case 2: return toArray(b, c);
        }
        throw new AssertionError();
    }

    String[] attrs = pickTwo("좋은", "빠른", "저렴한"); // ClassCastException 발생!!
    ```
    * 제네릭 가변인수를 받는 toArray는 내부적으로 Object[]를 생성함
    * 그런데 attrs에 저장하기 위해 String[]로 형변환하는 코드를 컴파일러가 자동생성하기 때문;;
    * 그러니 제네릭 가변인수 배열에 다른 메서드(toArray 같은)의 접근은 안전하지 않다!
    * 예외
        + @SafeVarargs가 붙은 메서드에 넘기면 안전하다
        + 변환하는 과정이 없는 일반 메서드에 넘기면 안전하다
- 안전하게 하려면?
    * @SafeVarargs 가 붙은 메서드에만 넘겨준다 (안전하지 않은 메서드면 반드시 수정)
    * 배열이지만 List<> 로 넘겨준다
        ```java
        static <T> List<T> pickTwo(T a, T b, T c) {
            switch (ThreadLocalRandom.current().nextInt(3)) {
                case 0: return List.of(a, b); // List.of는 @SafeVarargs가 붙어있음
                case 1: return List.of(a, c); 
                case 2: return List.of(b, c);
            }
            throw new AssertionError();
        }
        ``` 
    * 단순히 경고를 피하기 위해서라기보다 컴파일러가 타입 안정성을 검증할 수 있도록 하자