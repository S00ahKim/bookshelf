# 자바 직렬화의 대안을 찾으라

## Concept
### 직렬화
- 넓은 의미로는 어떤 데이터를 다른 데이터의 형태로 변환
- 좁은 의미로는 객체의 상태를 byte stream으로의 변환
    * 반대로 byte stream을 객체의 상태로 변환하는 것을 역직렬화(Deserialization)라고 함
- 직렬화하는 이유
    * 데이터 전송을 했는데 상대 측이 Java 객체라고 알 방법이 없으므로 하나의 약속을 정하고, 그 형태로 변환하는 것이다.
    * 즉, **송수신측 양 쪽에서 모두 이해할 수 있는 형태로 바꾸는 것**이다.

### 바이트 스트림
- 데이터의 흐름. 데이터 통로.
    * Byte : Java에서 I/O Stream 기본 단위를 byte로 둔다. (InputStream, OutputStream)
    * Stream : Client와 Server 같이 출발지와 목적지로 입출력하기 위한 통로
- 바이트 스트림으로 직렬화하는 이유
    * 컴퓨터에서 기본으로 처리되는 최소 단위가 Byte라서
    * bit를 최소 단위로 잡으면 표현 방법이 너무 적어 byte를 하나의 단위로 잡고 있다.
    * Byte Stream으로 변환해야 Network, DB에서도 수신한 데이터를 이해할 수 있게 된다.


## 구현
### 직렬화하는 법
```java
@Test
void writeObjectTest() throws IOException {
    Person person = new Person("jayang", 24);

    byte[] serializedPerson;
    try (ByteArrayOutputStream baos = new ByteArrayOutputStream()) {
        try (ObjectOutputStream oos = new ObjectOutputStream(baos)) {
            oos.writeObject(person);
            // 직렬화된 Person 객체
            serializedPerson = baos.toByteArray();
        }
    }
    assertThat(serializedPerson).isNotEmpty();
}

static class Person implements Serializable {
    private String name;
    private int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
}
```
- Marker Interface인 Serializable 인터페이스를 구현

### 역직렬화하는 법
```java
void writeObjectTest2() throws IOException {
    Person person = new Person("jayang", 24);

    ...
    // 직렬화 생성 코드 생략
    ...

    Person deSerializedPerson = null;

    try (ByteArrayInputStream bais = new ByteArrayInputStream(serializedPerson)) {
        try (ObjectInputStream ois = new ObjectInputStream(bais)) {
            // 역직렬화된 Person 객체
            deSerializedPerson = (Person) ois.readObject();
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
    }

    assertThat(deSerializedPerson).isNotNull();
    assertThat(deSerializedPerson.name).isEqualTo("jayang");
    assertThat(deSerializedPerson.age).isEqualTo(24);
}
```


## 직렬화의 보안 문제
- 공격 범위가 너무 넓고 지속적으로 더 넓어져 **방어하기 힘들다**.
    * ObjectInputStream의 readObject 메서드는 Serializable 인터페이스를 구현한 class path 안의 거의 모든 타입 객체를 만들어 낼 수 있는 생성자다. (반환 타입 Object)
    * Deserialization 과정에서 해당 타입들 안의 모든 코드를 수행할 수 있다. (객체를 그대로 불러오므로 모든 코드를 구행할 수 있게 된다.)
    * 따라서, 타입들의 코드 전체가 공격 범위에 들어간다.
- **신뢰할 수 없는 Stream을 함부로 Deserialization하면 매우 위험**하다.
    * 원격 코드 실행(RCE, Remote Code Execution), 서비스 거부(DoS, Denial-of-Service) 등의 공격으로 이어질 수 있다.
    * 가젯(Gadget) : Deserialization 과정에 호출되어 잠재적으로 위험한 동작을 수행하는 메서드
        + 여러 가젯이 모여 가젯 체인이 구성되면, 공격자가 기반 하드웨어 native code를 마음대로 실행할 수 있는 경우도 있다.
    * 역직렬화 폭탄(Deserialization Bomb)
        ```java
        public class DeserializationBomb {
            public static void main(String[] args) throws Exception {
                System.out.println(bomb().length); // 5,744 byte
                deserialize(bomb());
            }
            
            static byte[] bomb() {
                Set<Object> root = new HashSet<>();
                Set<Object> s1 = root;
                Set<Object> s2 = new HashSet<>();
                for (int i = 0; i<100; i++) {
                    Set<Object> t1 = new HashSet<>();
                    Set<Object> t2 = new HashSet<>();
                    t1.add("foo"); // t1을 t2와 다르게 만든다.
                    s1.add(t1); s1.add(t2); // s1: {{"foo"}}, s1: {{}, {"foo"}}
                    s2.add(t1); s2.add(t2); // s2: {{"foo"}}, s2: {{}, {"foo"}}
                    s1 = t1; // s1: {"foo"}
                    s2 = t2; // s2: {}
                }
                return serialize(root); // 간결하게 하기 위해 이 메서드의 코드는 생략함
            }
        }
        ```
        + Deserialization에 시간이 오래 걸리는 짧은 Stream만으로 DoS에 쉽게 노출된다.
        + 위 HashSet을 역직렬화하기 위해 2^100번 넘게 hashCode를 호출해야 한다. (태양이 꺼질 때까지도 돌아갈 것이다.)
- **용량**도 다른 format에 비해 몇 배 이상의 크기를 가진다.


## 크로스-플랫폼 구조화된 데이터 표현(Cross-platform Structured-data Representation)
> 승리하는 유일한 길은 전쟁하지 않는 것이다. 아무것도 역직렬화하지 마라.
- 객체와 byte sequence를 변환해주는 mechanism
    * **Java 직렬화보다 훨씬 간단하고, 임의 객체 그래프를 자동으로 직렬화/역직렬화하지 않는다.**
    * 속성-값 쌍의 집합으로 간단하고 구조화된 데이터 객체를 사용한다.
    * 기본 타입 몇개와 배열 타입만 지원
    * JSON, 프로토콜 버퍼(protobuf)가 여기에 속한다.

### JSON
```json
{
    "userName": "Martin",
    "favouriteNumber": 1337,
    "interests": ["daydreaming", "hacking"]
}
```
- 원래는 Javascript용, 브라우저와 서버 통신용으로 설꼐
- 텍스트 기반 데이터 표현 방식이므로 사람이 읽을 수 있다.

### Protobuf
```
message Person {
    required string user_name        = 1;
    optional int64  favourite_number = 2;
    repeated string interests        = 3;
}
```
- 서버 사이에 데이터를 교환하고 저장하기 위해 설계
- 이진 표현이라 효율이 훨씬 높다. (고성능 직렬화)
- 데이터 표현 뿐 아니라 타입 또한 강제할 수 있고, 사람이 읽을 수 있는 텍스트 표현(pbtxt)도 지원한다.


## 레거시 코드를 다룰 때 할 수 있는 선택
```java
public class SerializationFilter {
    public static void main(String[] args) throws Exception {
        Employee emp = new Employee("jayang", 24);

        // Serialization
        String fileName = "employee.ser";
        FileOutputStream fos = new FileOutputStream(fileName);
        ObjectOutputStream oos = new ObjectOutputStream(fos);
        oos.writeObject(emp);
        oos.close();
        fos.close();

        // Deserialization
        FileInputStream fis = new FileInputStream(fileName);
        ObjectInputStream ois = new ObjectInputStream(fis);

        ois.setObjectInputFilter(
                (info) -> {
                    if (info.serialClass() == Employee.class) {
                        return ObjectInputFilter.Status.ALLOWED;
                    }
                    return ObjectInputFilter.Status.REJECTED;
                }
        );
        Employee empNew = (Employee) ois.readObject();
    }
}
```
- 신뢰할 수 없는 데이터는 절대 Deserialization하지 않는다.
- 직렬화를 피할 수 없고, 안전한 데이터인지 확인할 수 없다면, **객체 역직렬화 필터링**을 사용하라
    * Data Stream이 Deserialization되기 전에 Filter를 설치하는 전략
    * 클래스 단위로 특정 클래스를 수용, 거부할 수 있다. (블랙리스트보단 화이트 리스트 방식을 택하라)