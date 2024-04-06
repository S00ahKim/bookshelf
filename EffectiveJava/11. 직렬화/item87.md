# 커스텀 직렬화 형태를 고려해보라


## 기본 직렬화를 사용해도 되는 경우
1. 직접 설계하더라도 기본 직렬화 형태와 거의 같은 결과가 나올 경우
    - 어떤 객체의 기본 직렬화 형태는 해당 객체를 root로 하는 객체 그래프의 물리적 모습을 나름 효율적으로 인코딩한다.
        * 객체가 포함한 데이터, 접근 가능한 모든 객체, 객체들이 연결된 위상(topology)까지 기술한다.
    - 그러나, 이상적인 직렬화 형태는 물리적 모습과 독립된 논리적 모습만을 표현해야 한다. (공간 문제) 
2. 객체의 물리적 표현과 논리적 내용이 같은 경우
    ```java
    public class Name implements Serializable {
        /**
         * 성. null이 아니어야 함.
        * @serial
        */
        private final Stirng lastName;

        /**
         * 이름. null이 아니어야 함.
        * @serial
        */
        private final String firstName;

        /**
         * 중간이름. 중간 이름이 없다면 null.
        * @serial
        */
        private final String middleName;
        
        ...
    }
    ```
    - 이름, 성, 중간이름은 논리적으로 물리적으로 동일한 표현을 가지므로 기본 직렬화를 적용해도 괜찮다.
* **기본 직렬화 형태가 적합하더라도, 불변식 보장과 보안을 위해 readObject 메서드를 제공해야 할 때가 많다.**
    - readObject 메서드가 lastName과 firstName 필드가 **null이 아님**을 보장해야 한다. (item88, item90)
    - private임에도 직렬화 형태에 포함되는 공개 API에 속하기 때문에 문서화 주석을 달아주어야 한다.
    - `@serial` 태그는 API 문서에서 직렬화 형태를 설명하는 특별 페이지에 기록된다.


## 기본 직렬화 형태에 적합하지 않은 경우
```java
public final class StringList implements Serializable {
    private int size = 0;
    private Entry head = null;

    private static class Entry implements Serializable {
        String data;
        Entry next;
        Entry previous;
    }
    // ... 생략
}
```
- 논리적 : 일련의 문자열을 표현
- 물리적 : 문자열들을 이중 연결 리스트로 연결
* **객체의 물리적 표현과 논리적 표현의 차이가 클 때, 기본 직렬화 형태는 크게 네 가지 면에서 문제가 생긴다.**
    1. 공개 API가 현재 내부 표현 방식에 영구히 묶인다.
        - private 클래스인 StringList.Entry가 공개 API가 되어버린다.
        - 연결 리스트를 더는 사용하지 않더라도, 연결 리스트로 표현된 입력도 처리할 수 있어야 하므로 관련 코드를 절대 제거할 수 없다.
    2. 너무 많은 공간을 차지할 수 있다.
        - 불필요한 **내부구현(각 노드의 양방향 연결 정보와 모든 Entry 정보)까지 직렬화** 한다.
        - "이상적인 직렬화 형태는 물리적 모습과 독립된 논리적 모습만을 표현한다"
    3. 시간이 너무 많이 걸릴 수 있다.
        - 직렬화 로직은 객체 그래프 위상 정보가 없으므로 직접 순회해볼 수밖에 없다. (StringList는 간단히 참조를 순회하기만 해도 되긴 하다.)
    4. StackOverflow를 일으킬 수 있다.
        - 기본 직렬화 과정은 객체 그래프를 **재귀 순회**한다.
        - 운영 환경에 따라 한계치가 바뀐다.


## StringList 커스텀 직렬화 예시
-  아이디어
    * 합리적인 직렬화 형태
    * 리스트가 포함한 문자열 개수를 적고, 문자열들을 나열하는 수준 (논리적인 구성만 담는다.)
    * writeObejct와 readObject가 직렬화 형태 처리
- transient 한정자
    * 해당 인스턴스 필드가 **기본 직렬화 형태에 포함되지 않는다**는 표시.
    * 해당 객체의 논리적 상태와 무관한 필드라고 확신하는 경우를 제외하고 모두 transient 한정자를 붙인다.
    * 다른 필드에서 유도되는 필드(캐시된 해시값), JVM을 실행할 때마다 값이 달라지는 필드 모두 포함된다.
    * transient 필드는 기본 직렬화로 역직렬화하면 기본값(0, false, null)로 초기화 되므로, 기본값을 그대로 사용하면 안 되는 경우 defaultReadObject() 호출 후(item88), 혹은 처음 사용 시점에 초기화(item83)하라.
- 구현
    ```java
    public final class StringList implements Serializable {
        private transient int size = 0;// 직렬화 대상에서 제외한다.
        private transient Entry head = null;

        // 이번에는 직렬화 하지 않는다.
        private static class Entry {
            String data;
            Entry next;
            Entry previous;
        }

        // 문자열을 리스트에 추가한다.
        public final void add(String s) { ... }

        /**
         * StringList 인스턴스를 직렬화한다.
        */
        private void writeObject(ObjectOutputStream stream)
                throws IOException {
            stream.defaultWriteObject();
            stream.writeInt(size);

            // 모든 원소를 순서대로 기록한다.
            for (Entry e = head; e != null; e = e.next) {
                s.writeObject(e.data);
            }
        }

        private void readObject(ObjectInputStream stream)
                throws IOException, ClassNotFoundException {
            stream.defaultReadObject();
            int numElements = stream.readInt();

            for (int i = 0; i < numElements; i++) {
                add((String) stream.readObject());
            }
        }
        // ... 생략
    }
    ```
    * 필드 모두가 transient더라도 writeObject / readObject에서 가장 먼저 defaultWriteObject / defaultReadObject를 호출하라
        + 향후 transient가 아닌 인스턴스 필드가 추가되어도, 구버전과 호환성이 깨지지 않는다.
        + 구버전에서 defaultReadObject를 호출하지 않으면, 신버전을 역직렬화 할 때 StreamCorruptedException이 발생한다.
- 성능
    * 문자열들의 길이가 평균 10이라면, 개선 버전은 원래 버전의 절반 정도 공간을 차지한다.
    * 수행 속도가 빨라지고, StackOverflow가 전혀 발생하지 않는다. (직렬화 크기 제한 없어짐)
- 불변식이 세부 구현에 따라 달라지는 객체에서의 역직렬화 (ex. Hash Table)
    * Hash Table이 Entry를 담은 버킷을 담을 지는 Key에서 구한 hashcode로 결정한다.
    * 해당 hashcode는 구현에 따라, 심지어 계산할 때마다 달라지기도 한다.
    * 애초에 불변식이 세부 구현에 따라 달라지므로, 기본 직렬화 방식을 사용하면 불변식이 심각하게 훼손될 수 있다.


## 동기화
> 어떤 직렬화 형태든 객체 전체 상태를 읽는 메서드에 적용해야 하는 동기화 메커니즘을 직렬화에도 적용하라
```java
private synchronized void writeObject(ObjectOutputStream s) throws IOException {
    s.defaultWriteObject();
}
```
- 모든 메서드가 synchronized로 선언하여 thread-safe한 객체(item82)에서 기본 직렬화를 사용하려면 writeObject도 synchronized로 선언하라.
- writeObject 메서드 안에서 동기화 하려면 클래스의 다른 부분에서 사용하는 Lock 순서를 똑같이 따라라.
- 그러지 않으면 자원 순서 교착상태(resource-ordering deadlock)에 빠질 수 있다.
```java
public class IncorrectLockOrderExample implements Serializable {
    private final Object lock1 = new Object();
    private final Object lock2 = new Object();

    private String data;

    public IncorrectLockOrderExample(String data) {
        this.data = data;
    }

    private void writeObject(ObjectOutputStream out) throws IOException {
        synchronized (lock1) {
            // 락 lock1을 획득한 상태에서 다른 락 lock2를 획득하려고 시도
            synchronized (lock2) {
                out.defaultWriteObject(); // 기본 직렬화 수행
            }
        }
    }

    public void modifyData(String newData) {
        synchronized (lock2) {
            // 락 lock2를 획득한 상태에서 다른 락 lock1을 획득하려고 시도
            synchronized (lock1) {
                this.data = newData;
            }
        }
    }
}
```


## serialVersionUID
> 어떤 직렬화 형태든 직렬화 가능 클래스 모두에 serialVersionUID를 명시적으로 부여하라
```java
private static final long serialVersionUID = <무작위로 고른 long 값>;
```
- 잠재적 호환성 문제가 사라진다.
- Runtime에 값을 생성하느라 복잡한 연산을 수행할 필요가 없으므로 성능도 조금 빨라진다.
- 고유할 필요가 없고, 생각나는 아무 값이나 선택해도 된다.
- 이미 구버전으로 직렬화된 인스턴스가 있다면, 구버전 클래스를 serialver 유틸리티에 입력으로 주어 얻어낸 자동 생성된 값을 그대로 사용해야 한다.
    * `$ serialver MyClass`
- 구버전으로 직렬화된 인스턴스와의 호환성을 끊는 경우를 제외하고, serialVersionUID를 절대 수정하지 마라.