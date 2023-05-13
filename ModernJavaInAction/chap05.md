# 스트림 활용

## 필터링
1. Predicate 으로 필터링
    * 프레디케이트? boolean을 반환하는 함수
    * 프레디케이트와 일치하는 모든 요소를 포함하는 스트림 반환
    * ex. `.filter(Dish::isVegan)`
2. 고유 요소 필터링
    * 고유한지 여부는 hashCode, equals로 결정됨
    * ex. `.distinct()`


## 슬라이싱
> 요소를 선택하거나 스킵하기
1. Predicate 으로 슬라이싱
    * filter는 전체 스트림을 반복하며 요소 각각에 프레디케이트를 적용함
    * 만약 정렬된 아주 큰 스트림이라면 특정 값 이상부터는 보지 않도록 하여 성능 이득을 꾀할 수 있음
    * ex. `.takeWhile(dish -> dish.getCalories() < 320)` 320 미만까지를 가져온다
    * ex. `.dropWhile(dish -> dish.getCalories() < 320)` 320 이상부터를 가져온다
2. 스트림 축소
    * ex. `.limit(3)`
    * 결과 스트림의 정렬 여부는 소스의 정렬 여부에 따름
3. 요소 건너뛰기
    * 처음 n개 요소를 제외한 스트림을 반환
    * n이 스트림 사이즈보다 크면 빈 스트림이 반환
    * ex. `.skip(2)`


## 매핑
1. 스트림의 각 요소에 함수 적용하기
    * 함수를 인수로 받아 스트림 각 요소에 적용한 결과가 새로운 요소로 반환
    * 새로운 버전을 만든다는 개념에 가까움 (고침 X, 변환 O)
    * ex. `.map(Dish::getName)`
2. 스트림 평면화
    * 스트림의 각 값을 다른 스트림으로 만든 다음에 모든 스트림을 **하나의 스트림**으로 연결하는 것
    * ex. `.map(word -> word.split("")).flatMap(Arrays::stream)`
        + 이 경우, 각 배열을 스트림이 아니라 스트림의 콘텐츠로 매핑한다.


## 검색과 매칭
> - 이때 사용되는 메서드들은 모두 **최종 연산**이다.
> - anyMatch, allMatch, noneMatch는 하나만 해당되어도 결과를 낼 수 있어서 **쇼트서킷**으로 평가한다.
1. 프레디케이트가 적어도 한 요소와 일치하는지 확인 ex. `.anyMatch(Dish::isVegan)`
2. 프레디케이트가 모든 요소와 일치하는지 확인 ex. `.allMatch(Dish::isVegan)`
3. 프레디케이트와 일치하는 요소가 없는지 확인 ex. `.noneMatch(Dish::isVegan)`
4. 현재 스트림에서 임의의 요소 반환 ex. `.findAny()`
5. 스트림의 첫번째 요소 반환 ex. `.findFirst()`


## 리듀싱
> - 모든 스트림 요소를 처리해서 값으로 도출하는 연산
> - 함수형 프로그래밍에서는 폴드fold 라고 부름
1. 요소의 합
    * ex. `int sum = numbers.stream().reduce(0, (a,b) -> a+b);` 또는 `.reduce(0, Integer::sum)`
    * 인수: 초기값, 두 요소를 조합해서 새로운 값을 만드는 BinaryOperator<T>
    * 초기값이 a 가 되어 시작한다고 보면 됨
    * 초기값을 안 받아도 되게 오버로드된 경우도 있는데, 이 때에는 Optional이 반환됨
2. 최대값과 최소값
    * ex. `Optional<Integer> max = numbers.stream().reduce(Integer::max)`
    * ex. `Optional<Integer> max = numbers.stream().reduce(Integer::min)`
3. 맵 리듀스
    * 스트림의 각 요소를 1로 매핑하고 reduce로 그 1의 합계를 계산하는 방식으로 count를 구현하는 등 map-reduce를 연결하는 기법
    * 구글이 웹 검색에서 사용하면서 유명해짐
- reduce를 사용하면 내부 반복이 추상화되어 병렬로 실행할 수 있게 됨. 기존의 반복 합계에서는 공유 변수가 있어 어려웠음.


## 기타
- 상태 있음 vs 상태 없음
    * 스트림에서 처리하는 요소 수와 관계없이 내부 상태의 크기는 한정됨(`bounded`)
    * 결과를 누적하는 연산은 내부 상태가 필요`stateful`함 (ex. reduce, sum, max 등)
    * 과거의 이력을 알고 있어야 하는 연산은 **모든 요소가 버퍼에 추가되어 있어야** 함 (ex. sorted, distinct 등)
    * ===============
    * 크기가 고정되지 않은 스트림(`unbounded`)을 만들 수 있음 (보통은 limit 등으로 값을 제한해서 결과를 만듦)
    * ex. `Stream.iterate(0, n-> n+2).limit(5).forEach(System.out::println)`
- 숫자형 스트림
    * 숫자에 특화된 IntStream, DoubleStream, LongStream 을 제공함
    * sum, max 등 자주 사용되는 연산 수행 메서드 제공
    * 박싱(int -> Integer) 과정에서 일어나는 효율성과 관련될 뿐, 추가 기능은 없음
    * `intStream.boxed()` 처럼 숫자 스트림을 스트림으로 변환할 수 있음
- 값으로 스트림 만들기 ex. `Stream.of("Hello", "World");`
- nullable한 객체로 스트림 만들기 ex. `Stream.ofNullable(foo.getBar());`
- 배열로 스트림 만들기 ex. `Arrays.stream(numbers).sum()`