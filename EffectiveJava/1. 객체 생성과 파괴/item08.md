# finalizer와 cleaner 사용을 피하라

## 추천하는 대안: AutoCloseable
```java
// cleaner를 안전망으로 활용하는 AutoCloseable 클래스
public class Room implements AutoCloseable {
	private static final Cleaner cleaner = Cleaner.create();
	
	// State가 바로 청소가 필요한 자원
  // GC가 자동으로 수거하려면 절대 Room을 참조(순환참조가 생김)해서는 안 됨.
  // 정적이 아닌 중첩 클래스는 자동으로 바깥 객체의 참조를 갖게 되므로(item24, 람다도 마찬가지) 정적 클래스로 선언함.
	private static class State implements Runnable {
		int numJunkPiles; // 방(Room) 안의 쓰레기 수
		
		State(int numJunkPiles) {
			this.numJunkPiles = numJunkPiles;
		}

		// close 메소드나 cleaner가 호출됨
		@Override public void run() {
			System.out.println("방 청소");
			numJunkPiles = 0;
		}
	}

	// 방의 상태. cleanable과 공유
	private final State state;
	
	// cleanable 객체. 수거 대상이 되면 방을 청소함
	private final Cleaner.Cleanable cleanable;
	
	public Room(int numJunkPiles) {
		state = new State(numJunkPiles);
		cleanable = cleaner.register(this, state);
	}

	@Override public void close() {
		cleanable.clean();
	}
}

// 자동 회수가 되도록 잘 만들어진 클라이언트 코드
// try-with 패턴 활용
// 안녕! -> 방 청소
public class Adult {
  public static void main(String[] args) {
    try (Room myRoom = new Room(7)) {
      System.out.println("안녕!");
    }
  }
}

// 클리너를 사용하게 된 클라이언트 코드
// 안녕!
public class Adult {
  public static void main(String[] args) {
    new Room(99);
    System.out.println("안녕!");
  }
}
```
- 파일/스레드 등 종료해야 할 자원을 가진 클래스가 자원 회수를 지원하는 방법
- 클라이언트에서 인스턴스를 다 쓰고 나면 close 메소드를 호출
- 구현할 때 각 인스턴스는 자신이 닫혔는지를 추적하는 것이 좋음 (필드로 유효성 여부를 관리하고, 이를 검사)


## 객체 소멸자는 일반적으로 불필요하다
### 자바의 객체 소멸자들
1. **finalizer**
    - 자바9~ `@Deprecated`
    - 실행 시점 예측 불가
        * 다른 애플리케이션 스레드보다 우선순위가 낮기 때문
        * 즉시 수행된다는 보장이 없음, GC알고리즘에 달림
        * 클래스에 달려 있으면 인스턴스 자원 회수가 멋대로 지연될 수 있음
        * 이식성 문제의 원인
    - 실행 여부 예측 불가
        * 이식성 문제의 원인
    - 오동작 가능성
        * finalize 도중 발생한 에러 무시됨 & 처리할 작업이 남았어도 에러 발생 직후 종료됨
        * 제대로 회수되지 못한 객체에 접근하려고 시도할 경우 예측 불가능한 오동작 가능성 있음
    - 낮은 성능
    - finalizer 공격 가능
        * ex. https://yangbongsoo.tistory.com/8
        * final이 아닌 클래스를 이 공격에서 방어하려면 아무 일을 하지 않는 finalizer를 만들고 이를 final로 선언하기
2. **cleaner**
    - 자바9~ 대안으로 등장, 덜 위험함
        * 자신을 수행할 스레드를 제어할 수 있음
        * ㄴ 스레드 우선순위를 설정할 수 있음
        * ㄴ 에러 발생시 스레드 중단됨 & 추적 내용 출력
    - 여전히 실행 시점 예측 불가
        * 백그라운드에서 수행
        * GC가 통제함
    - 여전히 실행 여부 예측 불가
    - 낮은 성능

### 왜 자바는 객체 소멸자가 불필요한가
- 명시적으로 할당 해제를 하지 않으면 메모리 누수가 발생하는 C++과 달리 자바는 **가비지 컬렉터**가 접근 불가 객체를 회수해준다.
- 비메모리 자원을 회수할 때에도 사용하는 C++의 파괴자와는 달리 자바는 **try-with, try-finally로 해결**(item09)한다.
- 실행 여부를 보장하지 않으므로 영구적인 상태 변경 작업에서는 절대 사용하면 안 된다. (공유 자원의 영구 Lock 해제 작업 등)
- `System.jc()`, `System.runFinalization()` 같은 메서드는 실행 가능성을 높여주지만, 여전히 보장하지 않는다.
    * 실행을 보장해주는 메서드들에는 심각한 결함이 있으므로 결코 사용하지 말 것

### 불필요하다면서 왜 지원하는가
1. close()를 호출하지 않는 상황에 대비하기 위한 안전망 역할
    - 없는 것보단 나으니까
    - 만들 가치가 있을지 숙고하자
    - ex. `FileInputStream`, `FileOutputStream`, `ThreadPoolExecutor`
2. 네이티브 피어와 연결된 객체를 다룰 때
    - 네이티브 피어? 일반 자바 객체가 네이티브 메서드를 통해 기능을 위임한 네이티브 객체. 자바 객체가 아님!
    - GC가 몰라서 회수도 못하는 객체이기 때문에 소멸자가 쓰이기 좋은 경우
    - 단, 성능 저하를 감당 가능 & 네이티브 피어가 심각한 자원을 갖고 있지는 않음
    - 만약, 성능 저하를 감당할 수 없거나 || 자원을 즉시 회수해야 한다면 -> close를 써라