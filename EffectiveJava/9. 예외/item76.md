# 가능한 한 실패 원자적으로 만들라
> 호출된 메서드가 실패하더라도 해당 객체는 메서드 호출 전 상태를 유지하는 특성

## 1. 불변 객체로 설계하는 방법 (item17)
```java
public class String {
    ...
    public String substring(int beginIndex, int endIndex) {
        int length = length();
        checkBoundsBeginEnd(beginIndex, endIndex, length);
        int subLen = endIndex - beginIndex;
        if (beginIndex == 0 && endIndex == length) {
            return this;
        }
        return isLatin1() ? StringLatin1.newString(value, beginIndex, subLen)
                          : StringUTF16.newString(value, beginIndex, subLen);
    }
    ...
}
```
- 불변 객체의 상태는 생성 시점에 고정되어 절대 변하지 않으므로 **태생적으로 실패 원자적**
- 메서드가 실패하면 새로운 객체가 만들어지지 않을 수는 있으나, 기존 객체 상태를 바꾸진 않음


## 2. 가변 객체 메서드라면 작업 수행에 앞서 매개변수 유효성을 검사하는 방법 (item49)
```java
public Object pop() {
    if (size == 0)
        throw new EmptyStackException();
    Object result = elements[--size];
    elements[size] = null; // 다 쓴 참조 해제
    return result;
}
```
- **가장 흔한 방법**
- 객체 내부 상태를 변경하기 전에 **잠재적 예외 가능성 대부분을 걸러**낼 수 있음
- if 문이 없어도 예외를 던지긴 하지만, 다음 예외에서 ArrayIndexOutOfBoundsException을 던지므로 추상화 수준에 맞지 않음 (item73) 즉, 스택에 없어서 문제인 건데 음수 인덱스 문제로 나타나면 안됨.

```java
public void add(String key, Object value) {
    if (!(value instanceof Integer))
        throw new ClassCastException(value.toString());
    map.put(key, (Integer) value);
}
```
- 실패할 가능성이 있는 모든 코드를, 객체의 **상태를 바꾸는 코드보다 앞에 배치**하는 방법
- TreeMap에 인자를 넣기 전에 정렬을 위한 '어떤 기준'에 적합한지 검사하여, 엉뚱한 타입의 원소라면 ClassCastException을 던지게 함


## 3. 객체의 임시 복사본에서 작업을 수행한 후, 성공적으로 완료되면 원래 객체와 교체하는 방법
```java
default void sort(Comparator<? super E> c) {
    Object[] a = this.toArray(); // 배열로 변환 
    Arrays.sort(a, (Comparator) c);
    ListIterator<E> i = this.listIterator();
    for (Object e : a) {
        i.next();
        i.set((E) e);
    } // 성공적으로 완료되면 원래 객체와 교체한다
}
```
- 데이터를 **임시 자료구조에 저장**해 작업하는 게 빠른 경우 적용하기 좋음
- 정렬 시 배열을 사용하면 참조 지역성이 높으므로 반복문에서 원소 접근이 훨씬 빠르게 가능함
- 정렬에 실패하더라도 입력 스트림은 변하지 않는 효과도 얻을 수 있음


## 4. 작업 도중 발생하는 실패를 가로체는 복구 코드를 작성하여 롤백(Rollback)하는 방법
- 주로 디스크 기반의 내구성(durability)을 보장해야 하는 자료구조에 쓰임
- 자주 쓰이는 방법은 아님


## 실패 원자성 보장이 힘든 경우
- 실패 원자성은 항상 달성할 수 있는 것은 아니다.
    + `ConcurrentModificationException`
        * 다중 스레드 환경에서 한 스레드가 컬렉션을 수정하는 도중 다른 스레드가 동일한 컬렉션을 수정하려고 할 때 발생
        * 에러가 발생하더라도 객체가 유지된다고 가정할 수 없음
        * 락이나 동기화 같은 추가적인 도구가 필요
    + `Error`, `AssertionError`
        * 시스템 레벨의 문제는 일반적인 방법으로 복구가 불가능
        * 이를 위해 실패 원자성을 보장하려는 시도 역시 의미가 없음
- 만들 수 있더라도 달성을 위한 비용이나 복잡도가 큰 연산도 존재하므로, 할 수 있다고 항상 해야하는 것도 아니다.
- 메서드 명세에 기술한 예외임에도 실패 원자성을 지키지 못한다면, **실패 시의 객체 상태를 API 설명에 명시**하라.