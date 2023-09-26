# ordinal 인덱싱 대신 EnumMap을 사용하라

## 배열의 인덱스로 ordinal을 사용하지 마라
```java
Set<Plant>[] plantsByLifeCycle = (Set<Plant>[]) new Set[LifeCycle.values().length];
for (int i = 0; i < plantsByLifeCycle.length; i++) {
    plantsByLifeCycle[i] = new HashSet<>();
}

List<Plant> garden = new ArrayList<>();
for (Plant plant : garden) {
    plantsByLifeCycle[plant.lifeCycle.ordinal()].add(plant);
}

for (int i = 0; i < plantsByLifeCycle.length; i++) {
    System.out.printf("%s : %s%n", Plant.LifeCycle.values()[i], plantsByLifeCycle[i]);
}
```
- 동작은 할 것이다
- 배열은 제네릭과 호환되지 않으니(item28) 비검사 형변환을 수행해야 하고 깔끔히 컴파일되지 않을 것이다
- 배열은 각 인덱스의 의미를 모르니 출력 결과에 직접 레이블을 달아야 한다
- 정확한 정숫값을 사용한다는 것을 우리가 직접 보증해야 한다 (잘못되어도 발견도 못할 수도!)
- 컴파일러는 ordinal과 배열 인덱스의 관계를 알 방법이 없다
    * 하나하나 값을 입력해주어야 함
    * 열거 타입을 수정하면서 배열 인덱스도 함께 수정하지 않거나 잘못 수정하면 런타임 오류가 날 것


## EnumMap을 사용해 데이터와 열거 타입을 매핑하자
```java
Map<Plant.LifeCycle, Set<Plant>> plantsByLifeCycle = new EnumMap<>(Plant.LifeCycle.class);
for (Plant.LifeCycle lifeCycle : Plant.LifeCycle.values()) {
    plantsByLifeCycle.put(lifeCycle, new HashSet<>());
}

List<Plant> garden = new ArrayList<>();
for (Plant plant : garden) {
    plantsByLifeCycle.get(plant.lifeCycle).add(plant);
}
System.out.println(plantsByLifeCycle);
```
- EnumMap은 열거 타입을 키로 사용하도록 설계한 아주 빠른 Map 구현체
- 더 짧고 명료하고 안전하고 성능도 원래 버전과 비등하다.
    * 내부에서 배열을 사용함
    * Map의 타입 안전성 + 배열의 성능
- 안전하지 않은 형변환은 쓰지 않는다.
- 맵의 키인 열거 타입이 그 자체로 출력용 문자열을 제공하기 때문에 출력 결과에 직접 레이블을 달 일도 없다.
- 배열 인덱스를 계산하는 과정에서 오류가 날 가능성도 원천봉쇄된다.
- 다차원 관계는 EnumMap<..., EnumMap<...>>으로 표현한다.


## 스트림을 사용할 수 있다
```java
List<Plant> garden = new ArrayList<>();
System.out.println(garden.stream().collect(
        groupingBy(plant -> plant.lifeCycle,
                () -> new EnumMap<>(LifeCycle.class), toSet())));
```
- groupingBy에 원하는 맵 구현체를 명시해 호출할 수 있음
- 스트림에서는 해당 키값에 속하는 원소가 있을 때만 만든다
    * EnumMap은 언제나 모든 키값(LifeCycle)에 대해서 하나씩의 중첩 맵을 만든다
