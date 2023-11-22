# 스트림에서는 부작용 없는 함수를 사용하라
> 부작용 = side effect

## 스트림 패러다임의 핵심
- 계산을 일련의 transformation(변환)으로 재구성
- 각 변환 단계는 가능한 한 이전 단계의 결과를 받아 처리하는 순수 함수
    * 순수 함수? 입력만 결과에 영향을 주는 함수 (다른 가변 상태 참조 X, 스스로 상태 변경 X)
    * **부작용이 없는 함수**여야 함


## DON'T!!
```java
//텍스트 파일에서 단어별 수를 세어 빈도표 만들기 (나쁜 예시)
Map<String, Long> freq = new HashMap<>();
try(Stream<String> words = new Scanner(file).tokens()) {
    words.forEach(word -> {
        freq.merge(word.toLowerCase(), 1L, Long::sum);
    });
}
```
- 스트림 코드를 가장한 반복적 코드
- 외부 상태(freq)를 수정하는 코드


## DO!!
```java
//텍스트 파일에서 단어별 수를 세어 빈도표 만들기 (좋은 예시)
Map<String, Long> freq;
try (Stream<String> words = new Scanner(file).tokens()) {
		freq = words.collect(groupingBy(String::toLowerCase, counting()));
}
```
- forEach 연산은 스트림 계산 결과를 보고할 때만 사용하고, 계산하는 데는 쓰지 말자
    * 기능이 가장 적음
    * 대놓고 반복적이라 병렬화할 수 없음
    * ex. 스트림 계산 결과를 기존 컬렉션에 추가할 때 사용 가능
- 수집기(collector)를 사용하여 스트림 원소를 쉽게 컬렉션에 모을 수 있다
    * 수집기? reduction 전략을 캡슐화한 블랙박스 객체
    * ex. 수집 관련: `toList()`, `toSet()`, `toMap()`, `toCollection(collectionFactory)`
    * ex. 수집과 관련 없음: `maxBy()`, `minBy()`
    * ex. 기타 유용한 메서드: `groupingBy`, `partitioningBy`, `joining`
    * cf. `.collect(toList());` 처럼 정적 임포트해서 사용하면 가독성이 좋다