# 람다보다는 메서드 참조를 사용하라
> 더 간결하니까!

## 메서드 참조
```java
foo.map(f -> {...})

foo.map(Foo::calculateComplexSummation)
```
- 더 짧고 간결한 코드를 생성할 수 있다.
    * 람다로 구현했을때 너무 길고 복잡하다면, **람다로 작성할 코드를 새로운 메서드에 담은 다음, 람다 대신 그 메서드 참조를 사용**하자!
- 기능을 잘 드러내는 이름을 지어줄 수 있고 친절한 설명을 문서로 남길 수 있다.
- 유형
    * 정적: `Integer::parseInt` (람다: `str -> Integer.parseInt(str)`)
    * 한정적: `Instant.now()::isAfter` (람다:`Instant then = Instant.now(); t -> then.isAfter(t)`
        + 이미 존재하는 인스턴스의 메서드를 참조함
        + 예시에서는 Instant.now()
    * 비한정적: `String::toLowerCase` (람다: `str -> str.toLowerCase()`)
        + 참조하는 시점에만 인스턴스가 있으면 됨
        + 예시에서는 String은 클래스지 인스턴스가 아니지만, 해당 코드가 적용될 땐 값을 가진 인스턴스가 들어올 것
    * 클래스 생성자: `TreeMap<K, V>::new` (람다: `() -> new TreeMap<K, V>()`)
    * 배열 생성자: `int[]::new` (람다: `len -> new int[len]`)
- 제네릭 함수 타입
    * 일반적으로 람다로 할 수 없다면 메서드 참조로도 할 수 없으나, 오직 메서드 참조만 가능한 예


## 람다가 더 나은 경우
```java
// GoshThisClassNameIsHumongous 클래스

service.execute(GoshThisClassNameIsHumongous::action);

// 이를 람다로 대체하면

service.execute(() -> action());
```
- 메서드와 람다가 같은 클래스에 있는 경우