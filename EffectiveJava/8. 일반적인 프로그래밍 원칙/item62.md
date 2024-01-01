# 다른 타입이 적절하다면 문자열 사용을 피하라
> 문자열은 의도하지 않은 용도로 자주 사용되는데, 그러면 안되는 경우에 대한 장

## 다른 값 타입을 대신하기에 적합하지 않음
- 의미에 적합한 타입을 사용하라
- ex. 예/아니오 이면 boolean


## 열거 타입을 대신하기에 적합하지 않음
- cf. item34
- 열거 타입은 더 읽기 쉽고 안전하고 강력함


## 혼합 타입을 대신하기에 적합하지 않음
```java
String compoundKey = className + "#" + i.next();
```
- 혼합 타입을 구성하는 요소 각각을 개별로 접근하고, 파싱해야 하는 비용이 들고, 귀찮고, 오류 가능성이 커짐


## 권한을 표현하기에 적합하지 않음
### 문자열로 권한을 표현한 예
```java
// 문자열로 권한을 표현했을 때
// ThreadLocal 에서 쓰는 변수를 문자열 키로 구분하는 예
public class ThreadLocal {
	private ThreadLocal() { } // 객체 생성 불가

	// 현 스레드의 값을 키로 구분해 저장
	public static void set(String key, Object value);

	// (키가 가리키는) 현 스레드의 값을 반환
	public static Object get(String key);
}
```
- 스레드 구분용 문자열 키가 전역 네임스페이스에서 공유됨
- 고유한 키를 제공받지 못할 위험이 있음
- 악의적으로 의도적으로 같은 키를 사용할 위험이 있음

### 클래스로 권한을 표현한 예
```java
public final class ThreadLocal<T> {
	public ThreadLocal() { }
  public void set(T value);
  public T get();
}
```
- 고유한 키를 반드시 갖게 됨
- 타입 안전함
- 빠름
