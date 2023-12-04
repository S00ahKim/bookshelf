# 반환 타입으로는 스트림보다 컬렉션이 낫다

## 원소 시퀀스, 일련의 원소를 반환하는 메서드 반환 타입
- 자바8 이전
    * 기본적으로 `Collection` 인터페이스
    * for-each문에서만 쓰이거나, 반환된 원소 시퀀스가 일부 Collection 메서드 구현이 불가능한 경우 `Iterable`
    * 반환 원소가 기본 타입이거나 성능에 민감한 경우 `배열`
- 자바8 이후 (Stream 등장)
    * **Stream은 반복(iteration)을 지원하지 않음**
    * 스트림만 반환하는 API는 for each 사용자들의 불만을 산다
    * 스트림은 Iterable을 확장하지 않아서 for each로 반복할 수 없기 때문
        + 단, Iterable 인터페이스가 정의한 방식대로 동작하는 추상 메서드를 모두 포함함


## 스트림을 Iterable하게 하는 방법들
- 명시적 형변환은 작동은 하지만 난잡하고 직관적이지 않다
- 자바의 타입 추론에 의존하는 어댑터 방식
    ```java
    public static <E> Iterable<E> iterableOf(Stream<E> stream) { 
            return stream::iterator;
    }
    ```


## 공개 API 반환 타입은 Collection이나 그 하위 타입을 쓰자
- 여러모로 추가 작업이 필요하므로, 공개 API 작성시엔 양쪽을 배려하자
    * 메서드가 오직 Stream 파이프라인에서만 쓰일 걸 안다면 Stream을 반환
    * 메서드가 오직 반복문에서만 쓰일 걸 안다면 Iterable을 반환
- 반환 시퀀스의 사이즈에 따라 반환 타입을 결정하자
    * 표준 Collection 사용 : 원소 사이즈가 작은 경우
    * 전용 Collection 구현 : 원소 사이즈가 큰 경우