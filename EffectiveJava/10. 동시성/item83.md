# 지연 초기화는 신중히 사용하라
> 양날의 검!

## 지연 초기화
- 소프트웨어 시스템은 Application 객체를 제작하고 의존성을 서로 연결하는 `준비 과정`과 준비 과정 이후에 이어지는 `Runtime 로직`을 **분리**해야 한다.
- 기본적으로 설정 논리는 일반 실행 논리와 분리해야 모듈성이 높아진다.
- 다만, 지연 초기화는 이 관심사 분리가 제대로 되지 못한 경우다.

### 장점
1. 실제 필요한 시점에 객체를 생성하므로 불필요한 부하를 줄인다.
    - Class와 Instance 초기화 때 발생하는 순환 문제를 해결하는 효과도 있다.
2. Application 실행 속도가 빨라진다.
3. 어떠한 경우에도 null 포인터를 반환하지 않는다.

### 단점
1. 실행 메서드가 생성자 클래스와 인수에 의존하게 된다.
2. 테스트 관점에서 비효율적이다. 테스트용 객체를 만들어야 한다.
3. 일반 실행 로직에 객체 생성 로직을 섞어놔서 모든 실행 경로를 테스트해야 한다.
4. 나중에 생성되는 클래스가 모든 상황에 적합한지 보장하기 어렵다.
5. 지연 초기화하는 필드에 접근하는 비용이 커진다.
    - 초기화가 이루어지는 비율, 실제 초기화에 드는 비용, 초기화된 필드를 호출하는 빈도수에 따라 성능 저하 원인이 될 수 있다.


## 초기화 전략
> 기본 타입 필드와 객체 참조 필드에 모두 적용할 수 있으면서, Thread-safe 한 전략들!

### 대부분의 상황에서 일반적인 초기화가 지연 초기화보다 낫다
```java
class Foo {
    private final FieldType field = computeFieldValue();  
    
    private FieldType computeFieldValue() {
        return new FieldType();
    }
}
```
- 인스턴스 필드를 선언할 때 수행하는 일반적인 초기화 모습
- 불변식을 지킴 item17

### 지연 초기화로 초기화 순환성을 깨뜨리고 싶다면 synchronized를 사용
```java
private FieldType field;

public synchronized FieldType getField() {
    if (field == null)
        field = computeFieldValue();
    return field;
}
```
- 위 관용구는 static 필드에도 똑같이 적용된다.
- 물론 필드와 접근자 메서드에 static 한정자를 추가해야 한다.
- 초기화 순환성?
    * 클래스 A → B → C → A 구조로 초기화 순환성이 존재할 경우, A객체 하나만 생성해도 StackOverFlow가 발생
    * 각 클래스가 지연 초기화되면 사용될 때만 초기화되므로 순환을 깰 수 있음
    * 다만, 로딩 순서에 따라 초기화 순서도, 클래스 변수 참조 순서도 불분명함

### 성능 때문에 정적 필드를 지연 초기화 할 때는 지연 초기화 홀더 클래스 관용구를 사용하라
```java
private static class FieldHolder {
    static final FieldType field = computeFieldValue();
}

public static FieldType getField() { return FieldHolder.field; }
```
- getField()가 처음 호출되는 순간 FieldHolder.field가 처음 읽히면서, FieldHolder 클래스 초기화를 촉발
- getField()메서드가 필드에 접근하면서 동기화를 전혀 하지 않으므로 성능이 느려지지 않음
- 일반적인 VM은 오직 클래스 초기화 단계에서만 필드 접근 동기화를 걸고, 그 이후에는 동기화 코드를 제거하여 아무런 검사나 동기화 없이 필드에 접근하게 됨

### 성능 때문에 인스턴스 필드를 지연 초기화 할 때는 이중검사 관용구를 사용하라
```java
private volatile FieldType field;

public FieldType getField() {
    FieldType result = field;
    if (result != null) // 첫 번째 검사 (Lock 사용 안 함)
        return result;
    
    synchronized(this) {
        if (field == null) // 두 번째 검사 (Lock 사용)
            field = computeFieldValue();
        return field;
    }
}
```
- 필드 값을 두 번 검사한다.
    1. 동기화 없이 검사
        * 속도 향상 목적
        * 매번 초기화하기 보다, 처음 필드에 접근하는 시점에만 초기화하는 것이 좋다.
        * 따라서 이미 초기화된 경우 추가적인 동기화 없이 필드에 접근할 수 있다.
    2. (필드가 아직 초기화되지 않았다면) 동기화하여 검사
        * Thread-safe 목적
        * Multi-thread 환경에서 동시 초기화하는 상황을 방지한다. 
- 지연 초기화하려는 인스턴스 필드는 반드시 volatile로 선언하라
    * 동기화 없이 검사 단계 때문에, 필드가 한 번 초기화되면 동기화 과정이 없다.
    * 따라서 volatile로 선언하여 값의 최신 상태를 보장할 수 있다.
- 반드시 필요하진 않지만 성능을 높여주고, 저수준 동시성 프로그래밍에 표준적으로 적용된다.
- 정적 필드에도 적용할 수 있지만 그럴 필요가 없고, 지연 초기화 홀더 클래스 방식이 더 낫다.

### cf. 단일검사 관용구
```java
private volatile FieldType field;

public FieldType getField() {
    FieldType result = field;
    if (result == null)
        field = result = computeFieldValue();
    retrn result;
}
```
- 반복해서 초기화를 해도 상관없는 인스턴스 필드를 지연 초기화하는 경우, 두 번째 검사를 생략할 수 있다.
- 기본 타입 필드에 이중검사/단일검사 관용구 적용 시, 필드의 값을 null대신 0과 비교하면 된다.
- 특정한 조건에서 단일 검사 필드의 volatile 한정자를 제거해도 된다.
    * 필드 타입이 long과 double을 제외한 다른 기본 타입인 경우
    * 모든 Thread가 field의 값을 다시 계산해도 상관 없는 경우
    * 짜릿한 단일검사(racy single-check) 관용구라 불린다.
        + 특정 환경에선 필드 접근 속도를 높여주지만, 초기화가 Thread 당 최대 한 번 더 수행될 수 있다.
        + 보통은 거의 쓰지 않는 기법이다.