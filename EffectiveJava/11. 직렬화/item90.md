# 직렬화된 인스턴스 대신 직렬화 프록시 사용을 검토하라

## 왜?
- Serializable을 구현하는 순간...
    * 생성자 외 방법으로 인스턴스 생성이 가능해짐
    * 따라서 버그와 보안 문제 가능성이 커짐
- 직렬화 프록시를 사용하면...
    * 

## 어떻게?
### 바깥 클래스의 직렬화 프록시를 중첩 클래스로 만든다
```java
private static class SerializationProxy implements Serializable {
    private static final long serialVersionUID = 2123123123L; // 아무 값이나 상관없음
    private final Date start;
    private final Date end;

    public SerializationProxy(Period p) {
        this.start = p.start;
        this.end = p.end;
    }
}
```
- 중첩 클래스를 `private static`으로 선언한다.
- **인스턴스의 데이터를 복사하는 역할**만 수행하게 만든다.
- 바깥 클래스를 불변 클래스로 관리할 수 있게 되어 일관성 검사나 방어적 복사도 필요 없다.
- 중첩 클래스의 **생성자는 단 하나**여야 한다.
- **바깥 클래스를 매개변수로 받아야** 한다.
- 바깥 클래스와 중첩 클래스 모두 Serializable을 구현한다.
- **바깥 클래스와 완전히 같은 필드**로 구성한다.

### 바깥 클래스에 writeReplace 메서드를 추가한다
```java
class Period implements Serializable {
    ...

    // Serialize -> 프록시 인스턴스 반환
    // 결코 바깥 클래스의 직렬화된 인스턴스를 생성해낼 수 없다
    private Object writeReplace() {
        return new SerializationProxy(this);
    }
    
    ...
}
```
- 자바의 직렬화 시스템이 바깥 클래스 인스턴스 대신 SerializationProxy 인스턴스를 반환하게 함
- 직렬화 시스템은 결코 바깥 클래스의 직렬화된 인스턴스 생성이 불가능하다. (직렬화 방지)

### 바깥 클래스에 readObject 메서드를 추가한다
```java
class Period implements Serializable {
    ...

    // Period 자체의 역직렬화를 방지 -> 역직렬화 시도시, 에러 반환
    private void readObject(ObjectInputStream stream) throws InvalidObjectException {
        throw new InvalidObjectException("프록시가 필요합니다.");
    }
}
```
- 바깥 클래스의 불변식을 훼손하려는 공격을 막아낼 수 있음 (역직렬화 방지)

### 프록시 클래스에 readResolve 메서드를 추가한다
```java
private static class SerializationProxy implements Serializable {
    ...
    
    // Deserialize -> Object 생성 (바깥 클래스와 논리적으로 동일한 인스턴스 반환)
    private Object readResolve() {
        return new Period(start, end);
    }
}
```
- 역직렬화 시, 직렬화 프록시를 다시 바깥 클래스 인스턴스로 변환
- 숨은 생성자 기능을 가진 직렬화의 특성을 상당 부분 제거함
    * 일반 인스턴스 만들 때와 똑같이 생성자, 정적 팩터리, 혹은 다른 메서드로 역직렬화 인스턴스를 생성하면 된다.
- 역직렬화된 인스턴스가 해당 클래스 불변식을 만족하는지 검사할 수단을 강구할 필요가 없음
    * 바깥 클래스의 생성자(혹은 정적 팩터리)가 불변식을 확인하고, 인스턴스 메서드들이 불변식을 잘 지켜준다면, 따로 더 할 일이 없다.

### 최종 구현
```java
class Period implements Serializable {
    // 불변 가능
    private final Date start;
    private final Date end;

    public Period(Date start, Date end) {
        this.start = start;
        this.end = end;
    }

    // 직렬화 시, Period 인스턴스를 직렬화하지 않고, SerializationProxy 인스턴스를 직렬화한다.
    private static class SerializationProxy implements Serializable {
        private static final long serialVersionUID = 2123123123L; // 아무 값이나 상관없음
        private final Date start;
        private final Date end;

        public SerializationProxy(Period p) {
            this.start = p.start;
            this.end = p.end;
        }

        // Deserialize -> Object 생성 (바깥 클래스와 논리적으로 동일한 인스턴스 반환)
        private Object readResolve() {
            return new Period(start, end);
        }
    }

    // Serialize -> 프록시 인스턴스 반환
    // 결코 바깥 클래스의 직렬화된 인스턴스를 생성해낼 수 없다
    private Object writeReplace() {
        return new SerializationProxy(this);
    }

    // Period 자체의 역직렬화를 방지 -> 역직렬화 시도시, 에러 반환
    private void readObject(ObjectInputStream stream) throws InvalidObjectException {
        throw new InvalidObjectException("프록시가 필요합니다.");
    }
}
```


## 장점
- 가짜 바이트 스트림 공격을 프록시 단에서 차단함 (item88)
- 내부 필드 탈취 공격을 프록시 단에서 차단함 (item88)
- 바깥 클래스 필드를 final로 선언 가능해지므로 바깥 클래스를 진짜 불변으로 만들 수 있음 (item17)
- 어떤 필드가 기만적 직렬화 공격 목표가 될지 고민하지 않아도 되며, 역직렬화 시 유효성 검사가 필요 없음
- 역직렬화할 인스턴스와 원래 직렬화된 인스턴스의 클래스가 달라도 정상 작동함 (ex. EnumSet)
    * EnumSet은 기본적으로 원소가 64개 이하면 RegularEnumSet, 그보다 크면 JumboEnumSet을 반환하도록 내부적으로 구현되어 있음
    * 원소 64개짜리 EnumSet(RegularEnumSet) → 직렬화 → 원소 5개 추가 → 역직렬화 → EnumSet(JumboEnumSet)


## 단점
- 클라이언트가 확장 가능한 클래스는 불변이 아니므로 적용할 수 없다.
- 객체 그래프에 순환이 있는 클래스에서 사용할 수 없다.
    * readResolve 호출 시, 이런 객체 메서드로 인해 ClassCastException이 발생할 것
    * 아직 Serialization Proxy만 가졌을 뿐, 실제 객체가 만들어진 상태가 아니기 때문
- 실행 속도가 저하된다. (프록시 패턴 자체의 단점)