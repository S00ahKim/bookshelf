# 확장할 수 있는 열거 타입이 필요하면 인터페이스를 사용하라

## 일반적으로...
- 열거타입은 확장할 수 없다
    * 열거한 값들을 그대로 가져온 다음 값을 더 추가할 수 없다
- 열거타입을 확장하려 하는 것은 권장하지 않는다
    * 확장한 타입의 원소는 기반 타입의 원소로 취급하지만 그 반대는 성립하지 않는 것은 이상하다.
    * 기반 타입과 확장된 타입들의 원소 모두를 순회할 방법이 마땅치 않다.
    * 확장성을 높이려면 고려할 요소가 늘어나 설계와 구현이 더 복잡해진다.


## 필요하다면 인터페이스를 사용하라
- 하지만 어떤 상황에서는 확장이 필요하다 (ex. 연산 코드)
- 인터페이스에 공통 메서드를 정의하고, 열거 타입은 해당 인터페이스의 표준 구현체처럼 대하자
- 추상 메서드를 선언할 필요 없다 (item34와의 차이)
- 장점
    * 인터페이스를 사용하도록 하면 보다 유연한 사용이 가능함
- 한계: 열거 타입끼리 구현을 상속할 수 없음
    * 아무 상태에도 의존하지 않는 경우, 인터페이스에 디폴트 메서드를 추가 (단, 모든 구현에 대응 필요)
    * 의존하는 경우, 공유하는 기능을 별도의 도우미 클래스나 정적 도우미 메서드로 분리하는 방식을 사용


## 예를 들면
1. 열거 타입의 Class 객체를 이용해 확장된 열거 타입의 모든 원소를 사용하는 예
    ```java
    public static void main(String[] args) {
        double x = Double.parseDouble(args[0]);
        double y = Double.parseDouble(args[1]);
        test(ExtendedOperation.class, x, y);
        // ㄴ ExtendedOperation의 class 리터럴을 넘겨 확장된 연산들을 모두 출력함
        //                        ㄴ 한정적 타입 토큰 역할을 함
    }
    private static <T extends Enum<T> & Operation> void test(
            Class<T> opEnumType, double x, double y) {
        for (Operation op : opEnumType.getEnumConstants())
            System.out.printf("%f %s %f = %f%n",
                    x, op, y, op.apply(x, y));
    }
    ```
2. 컬렉션 인스턴스를 이용해 확장된 열거 타입의 모든 원소를 사용하는 예
    ```java
    public static void main(String[] args) {
        double x = Double.parseDouble(args[0]);
        double y = Double.parseDouble(args[1]);
        test(Arrays.asList(ExtendedOperation.values()), x, y);
        // ㄴ Class 객체 대신에 한정적 와일드카드 타입인 Collection<? extends Operation>을 사용
        // 위 예시보다는 조금 더 유연함 (여러 구현 타입의 연산을 조합해 호출할 수 있음)
    }
    private static void test(Collection<? extends Operation> opSet,
                            double x, double y) {
        for (Operation op : opSet)
            System.out.printf("%f %s %f = %f%n",
                    x, op, y, op.apply(x, y));
    }
    ```