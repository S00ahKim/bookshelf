# 적시에 방어적 복사본을 만들라

## 자바는 안전한 언어지만, 방어적 코딩 습관은 필요하다
- C, C++ 등 안전하지 않은 언어는 종종 메모리 충돌 오류를 일으킨다
- 자바는 시스템의 다른 부분의 실행 방식과 무관하게 불변식이 지켜진다
- 그럼에도 실수/해킹 등에 사전에 대비하여 코드를 작성하는 편이 좋다


## 불변식이 깨지는 예 1: 생성자의 파라미터로 넘겨주는 객체를 조정
```java
public final class Period {
    private final Date start;
    private final Date end;
}

Date start = new Date();
Date end = new Date();
Period period = new Period(start, end);
end.setYear(78); // period의 내부를 수정했다!
```
- final 필드여도 변경될 수 있다!
- 자바8~ 불변인 Instant를 사용하거나 LocalDateTime이나 ZonedDateTime을 사용하여 방지


## 불변식을 지켜주는 방어적 복사
```java
public Period(Date start, Date end) {
    this.start = new Date(start.getTime());
    this.end = new Date(end.getTime());
    if (start.compareTo(end) > 0) {
        throw new IllegalArgumentException(start + "가 " + end + "보다 늦다.");
    }
}
```
- Date를 사용한 레거시들에 대응은 어쨌든 필요하다
- 방어적 복사?
    * 생성자의 인자로 받은 객체의 복사본을 만들어 내부 필드를 초기화 & 인스턴스에서 복사본 사용
    * getter 메서드에서 내부의 객체를 반환할 때, 객체의 복사본을 만들어 반환
- 유효성 검사는 방어적 복사본을 만든 후 그것으로 검사한다
    * 멀티쓰레딩 환경에서는 검사하는 찰나에 다른 쓰레드가 원본 객체를 수정할 위험(검사시점/사용시점(time-of-check/time-of-use) 공격)이 있으므로
- 매개변수가 제3자에 의해 확장될 수 있는 타입이라면 방어적 복사본을 만들 때 clone을 사용해서는 안 된다
    * ex. Date는 final이 아니므로 Date의 clone()을 사용하지 않음


## 불변식이 깨지는 예 2: 내부 필드에 접근해서 조정
```java
Date start = new Date();
Date end = new Date();
Period period = new Period(start, end);
period.getEnd().setYear(78); // period의 내부를 변경했다!
```


## 불변식을 지켜주는 가변 필드의 방어적 복사본을 반환하는 접근자
```java
public final class Period {

    public Date getStart() {
        return new Date(start.getTime());
    }

    public Date getEnd() {
        return new Date(end.getTime());
    }
}
```
- 생성자와 달리 접근자 메서드에서는 객체를 신뢰할 수 있으므로 방어적 복사에 clone을 사용해도 된다
- 그럼에도 생성자나 정적 팩터리를 권장한다


## 가변 객체를 다룰 때에는 방어적 복사를 고려해보자
- 클라이언트가 제공한 객체의 참조를 내부의 자료구조에 보관해야 할 때 임의로 변경되면 안 되는 경우
- 가변인 내부 객체를 클라이언트에 반환했을 때 안심할 수 없을 경우
    * cf. 길이가 1 이상인 배열은 무조건 가변이다
- 단, 방어적 복사에는 성능 저하가 따르고 항상 쓸 수 있는 것이 아니기 때문에 호출자가 컴포넌트 내부를 수정하지 않는다는 확신이 있다면 방어적 복사를 생략할 수 있다 (불변하다는 것을 문서화하자)