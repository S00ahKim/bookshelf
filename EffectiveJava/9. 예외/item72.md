# 표준 예외를 사용하라

## 기존에 만들어져 있는 예외를 사용하라
- 자주 발생하는 예외는 미리 정의되어 있다 (`주된 예외들`)
- 특수 상황을 위한 예외가 있다면 그걸 사용하라
    * `ArithmeticException`, `NumberFormatException` 복소수/유리수를 다룰 때
- 사용 지양
    * `Exception`, `RuntimeException`, `Throwable`, `Error`
    * 다른 예외들의 상위 클래스라서 안정적으로 테스트하기 어렵다
- API 문서로 확인할 것
    * 예외의 이름이 상황과 부합하나?
    * 예외가 던져지는 맥락이 상황과 부합하나?
- 장점
    * 타인이 사용하기 쉬운 코드가 된다
    * 타인이 읽기 쉬운 코드가 된다
    * 예외 클래스 수가 적어 메모리 사용량, 클래스 적재 시간이 적게 든다
- 필요하다면, 표준 예외를 확장하여 새로운 예외를 정의하자
    * 예외는 직렬화 가능하며, 직렬화에 드는 비용이 있다는 점은 고려할 것

## 주된 예외들
- `IllegalArgumentException`(item49) 부적절한 파라미터 사용시
    * ex. iterate(n) 에 -1을 넘길 때
    * 인수 값에 따라 실패 여부가 갈릴 수 있는 경우
- `IllegalStateException` 상태가 메서드를 수행하기에 부적함
    * ex. doSomething(x) 에 제대로 초기화되지 않은 객체를 넘길 때
    * 인수 값이 뭐가 오든 어차피 실패했을 경우
- `NullPointerException` null이 허용되지 않음
    * ex. getName(x) 에 null이 들어감
- `IndexOutOfBoundsException` 시퀀스의 허용 범위를 넘음
    * ex. length가 10인데 11번째에 접근함
- `ConcurrentModificationException` 허용되지 않은 동시 수정이 발견됨
    * ex. 싱글쓰레드용 객체로 보이는데 여러 쓰레드가 동시에 수정하려고 함
- `UnsupportedOperationException` 호출한 메서드를 지원하지 않음
    * ex. CannotRemoveList extends List 의 인스턴스에 remove를 호출