# readObject 메서드는 방어적으로 작성하라
> 그렇지 않으면 불변식을 깨뜨릴 것!

## readObject 메서드를 작성하는 지침
- readObject 메서드를 작성할 때는 언제나 public 생성자를 만든다고 생각하고 만들어야 함
    * 인수가 유효한지 검사해야 (item49)
    * 필요하다면 매개변수를 방어적으로 복사해야 (item50)
- private여야 하는 객체 참조 필드는 각 필드가 가리키는 객체를 방어적으로 복사하자
    * ex. 불변 클래스 내의 가변 요소
- 모든 불변식을 검사하고, 어긋나면 InvalidObjectException을 던진다.
    * 방어적 복사 다음에는 반드시 불변식 검사가 뒤따라야 함 (불변식 복사 → 불변식 검사)
- 역직렬화 후 그래프 전체 유효성 검사가 필요하다면 ObjectInputValidation 인터페이스를 사용하라
- readObject에서 직접적이든 간접적이든, 재정의 가능한 메서드를 호출하지 마라. (초기화 되기 전에 호출된다.)


## 역직렬화 방어적 수행
> 객체의 유효성 검사
```java
private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
    s.defaultReadObject();

    // 불변식을 만족하는지 검사한다.
    if (start.compareTo(end) > 0)
        throw new InvalidObjectException(start + "가 " + end + "보다 늦다.");
}
```
- 역직렬화 직후, 불변식을 만족하는지 확인한다.
    * readObject 메서드가 defaultReadObject를 호출한 다음 역직렬화된 객체가 유효한지 검사
- 이렇게 하면 허용되지 않는 Period 인스턴스 생성을 막을 수 있다.
- 정말 단순하지만, 이것만으로는 아직 안전하다고 할 수 없다.
    * 악의적인 객체 참조를 읽어 객체 내부의 값을 수정할 수 있다는 문제


## 가변 공격 방어
> 객체의 방어적 복사
- cf https://docs.oracle.com/en/java/javase/11/docs/specs/serialization/protocol.html#grammar-for-the-stream-format
```java
public class MutablePeriod {
    // Period 인스턴스
    public final Period period;

    // 시작 시각 필드 - 외부에서 접근할 수 없어야 한다.
    public final Date start;

    // 종료 시각 필드 - 외부에서 접근할 수 없어야 한다.
    public final Date end;

    public MutablePeriod() {
        try {
            ByteArrayOutputStream bos = new ByteArrayOutputStream();
            ObjectOutputStream out = new ObjectOutputStream(bos);

            // 유효한 Period 인스턴스를 직렬화한다.
            out.writeObject(new Period(new Date(), new Date()));

            /*
             * 악의적인 '이중' 인스턴스를 만든다. (이전 객체 참조, 자바 객체 직렬화 명세 6.4절)
             * 직렬화된 바이트 스트림을 역직렬화하여 Period 인스턴스를 만든다.
             * 그리고 내부의 Date 필드를 훔쳐낸다.
             */
            byte[] ref = {0x71, 0, 0x7e, 0, 5}; // 참조 #5
            bos.write(ref); // 시작 필드
            ref[4] = 4; // 참조 #4
            bos.write(ref); // 종료 필드

            // Period의 내부 Date 필드들을 훔쳐낸다.
            ObjectInputStream in = new ObjectInputStream(new ByteArrayInputStream(bos.toByteArray()));
            period = (Period) in.readObject();
            start = (Date) in.readObject();
            end = (Date) in.readObject();
        } catch (IOException | ClassNotFoundException e) {
            throw new AssertionError(e);
        }
    }

    public static void main(String[] args) {
        MutablePeriod mp = new MutablePeriod();
        Period p = mp.period;
        Date pEnd = mp.end;

        // 내부의 Date 필드를 훔쳐낸다.
        pEnd.setYear(78);
        System.out.println(p);

        // 내부의 Date 필드를 훔쳐낸다.
        pEnd.setYear(69);
        System.out.println(p);
    }
}
```
- 정상 Period 인스턴스의 byte 스트림 끝에 private Date 필드로의 참조를 추가해 가변 Period 인스턴스를 만들 수 있다.
- 객체를 역직렬화할 때는 Client가 소유해서는 안 되는 객체 참조를 갖는 필드를 반드시 모두 방어적으로 복사하라
    * readObject에서는 불변 클래스 내부의 모든 private 가변 요소를 방어적으로 복사해야 함

```java
private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
    s.defaultReadObject();
    
    // 가변 요소를 방어적으로 복사한다.
    start = new Date(start.getTime());
    end = new Date(end.getTime());
    
    // 불변식을 만족하는지 검사한다.
    if (start.compareTo(end) > 0)
        throw new InvalidObjectException(start + "가 " + end + "보다 늦다.");
}
```
- 방어적 복사를 유효성 검사보다 앞에 둔다. (item50)
- final 필드는 방어적 복사가 불가능하므로 한정자를 제거해야 한다. 공격 위험에 노출되는 것보다 낫다.

### final이 아닌 직렬화 가능 클래스의 경우
- 생성자와 마찬가지로 readObject도 재정의 가능 메서드를 호출해서는 안 된다. (item19)
- 해당 메서드가 재정의되면, 하위 클래스의 상태가 완전히 역직렬화되기 전에 하위 클래스의 재정의된 메서드가 실행되어 프로그램 오작동으로 이어진다.


## 기본 readObject 메서드를 사용 가능 판단 방법
- (transient 필드를 제외한) 모든 필드 값을 매개변수로 받아 유효성 검사 없이 필드에 대입하는 public 생성자를 추가해도 됨?
    * 그렇다 -> 기본 readObject 메서드
    * 아니다 -> 커스텀 readObject 메서드
- 직렬화 프록시 패턴(item90)을 사용하는 방법도 있음