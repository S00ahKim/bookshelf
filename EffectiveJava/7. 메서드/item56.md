# 공개된 API 요소에는 항상 문서화 주석을 작성하라

## JavaDoc?
- Java에서 API 문서화 작업을 돕는 유틸리티
- 소스 코드 파일에서 문서화 주석을 추려 API 문서로 변환해줌
- javadoc guide는 Java 4 이후로 갱신되지 않았지만, API 문서 작성법의 정석이라 참고하기 좋음


## 주의 사항
- 공개된 모든 클래스, 인터페이스, 메서드, 필드 선언에 문서화 주석을 달아라
    * 직렬화할 수 있는 클래스라면 직렬화 형태(item87)에 관해서도 적어라.
    * 기본 생성자에는 문서화 주석을 달 방법이 없으니, 공개 클래스는 절대 기본 생성자를 사용하지 마라.
    * 유지보수까지 고려한다면 대다수의 private 클래스, 인터페이스 등에서 문서화 주석을 달아야 할 것이다.
- 메서드용 문서화 주석은 해당 메서드와 클라이언트 사이 규약을 명료화하라
    * 어떻게(how) 동작하는지가 아니라 무엇(what)을 하는지 기술
    * 클라이언트가 해당 메서드를 호출하기 위한 전제조건(precondition)을 모두 나열
        + 일반적으로 @throws 태그로 비검사 예외를 선언하여 암시적으로 기술
        + 또는 @param 태그로 그 조건에 영향을 받는 매개변수에 기술
    * 메서드가 성공적으로 수행된 후에 만족해야 하는 사후조건(postcondition)도 모두 나열
    * 부작용
        + ex. 백그라운드 thread를 시작시키는 메서드
- 제네릭 타입이나 제네릭 메서드는 모든 타입 매개변수에 주석을 달아라
    ```java
    /**
     * An object that maps keys to values.  A map cannot contain duplicate keys; 
      * ...
      * @param <K> the type of keys maintained by this map
      * @param <V> the type of mapped values
      */
    public interface Map<K,V> { ... }
    ```
- 열거 타입은 상수들에도 주석을 달아라
    ```java
    /**
     * An instrument section of a symphony orchestra.
     */
    public enum OrchestraSection {
        /** Woodwinds, such as flute, clarinet, and oboe. */
        WOODWIND,

        /** Brass instruments, such as french horn and trumpet. */
        BRASS,

        /** Percussion instruments, such as timpani and cymbals. */
        PERCUSSION,

        /** Stringed instruments, such as violin and cello. */
        STRING;
    }
    ```
- 애너테이션 타입의 멤버들에도 모두 주석을 달아라
    ```java
    /**
     * 이 애너테이션이 달린 메서드는 명시한 예외를 던져야만 성공하는
    * 테스트 메서드임을 나타낸다.
    */
    @Retention(RetentionPolicy.RUNTIME)
    @Target(ElementType.METHOD)
    public @interface ExceptionTest {
        /**
         * 이 애너테이션을 단 테스트 메서드가 성공하려면 던져야 하는 예외.
        * (이 클래스의 하위 타입 예외는 모두 허용된다.)
        */
        Class<? extends Throwable> value();
    }
    ```
- 패키지와 모듈 시스템 주석
    * package-info.java : 패키지 설명하는 문서화 주석을 작성하는 파일
    * module-info.java : 모듈 시스템 사용 시, 모듈 관련 설명 주석을 작성하는 파일
- Thread 안전 수준
    * 클래스 혹은 정적 메서드가 Thread-safe 여부와 상관 없이!
    * Thread 안전성과 직렬화 가능성에 대한 설명 역시


## 문법
- 메서드 계약(contract) : `@param`, `@return`, `@throw`
    ```java
    /**
     * Returns the element at the specified position in this list.
    *
    * <p>This method is <i>not</i> guaranteed to run in constant
    * time. In some implementations it may run in time proportional
    * to the element position
    *
    * @param index index of element to return; must be
    *        non-negative and less than the size of this list
    * @return the element at the specified position in this list
    * @throws IndexOutOfBoundsException if the index is out of range
    *         ({@code index < 0 || index >= this.size()})
    */
    E get(int index);
    ``` 
    * 모든 매개 변수에 @param
    * 반환 타입이 void가 아니라면 @return
    * 발생할 가능성이 있는 (비검사/검사) 모든 예외에 @throws
    * 인스턴스 메서드 문서화 주석에 쓰인 "this"는 호출된 메서드가 자리하는 객체를 가리킴
- 메타 문자 무시 : `@code`, `@literal`
    * Javadoc은 문서화 주석의 HTML 태그를 최종 문서에 반영한다.
    * {@code ... }
        + 태그로 감싼 내용을 코드용 폰트로 렌더링한다.
        + 태그로 감싼 내용에 포함된 HTML 요소나 다른 자바독 태그를 무시한다.
        + 여러 줄로 된 코드 예시를 넣고 싶다면 <pre></pre> 태그로 전체를 감싸면 된다.
    * {@literal ... }
        + {@code} 태그와 비슷하지만 코드 폰트로 렌더링하지는 않는다.
- 자기사용 패턴(self-use pattern) : `@implSpec`
    ```java
    /**
     * Returns true if this collection is empty.
    *
    * @implSpec
    * This implementations returns {@code this.size() == 0}.
    *
    * @return true if this collection is empty
    */
    public boolean isEmpty() { ... }
    ``` 
    * 해당 메서드와 하위 클래스 사이의 계약을 설명
    * 하위 클래스들이 그 메서드를 상속하거나 super 키워드로 호출할 때, 메서드가 어떻게 동작하는지 명확히 인지하고 사용하도록 도움
- 요약 설명(summary description) : `@summary`
    * 요약 설명은 반드시 대상의 기능을 고유하게 기술해야 함
    * 각 문서화 주석의 첫 번째 문장은 해당 요소의 요약 설명으로 간주됨
        + 첫 번째 . 기호가 나타나기 전까지의 문장을 첫 번째 문장으로 간주 (어쩌구 Mr.도넛은 저쩌구 하면 어쩌구 Mr. 까지)
        + 의도한 바대로 나타나게 하기 위해 @literal을 사용 (ex. 어쩌구  {@literal Mr.} 저쩌구)
        + Java 10~ {@summary}라는 요약 설명 전용 태그가 추가되어 한번에 사용 가능하다. -{@summary 어쩌구 Mr.도넛은 저쩌구}
    * 해당 메서드와 생성자 동작을 설명하는 (주어가 없는) 동사구여야 한다 (영문의 경우 문장이 아니라는 뜻)
    * 클래스, 인터페이스, 필드의 요약 설명은 대상을 설명하는 명사절이어야 한다
- 색인 기능 : `@index`
    ```java
    This method compiles with the {@index IEEE 754} standard.
    ``` 
    * 자바9~
    * 클래스, 메서드, 필드 같은 API 요소의 색인은 자동으로 만들어짐
    * 중요한 단어에 대해 색인을 걸 수 있음
- 메서드 주석 상속 : `@inheritDoc`
    * 메서드 주석을 상속시켜 중복 내용을 줄임
    * 문서화 주석이 없는 API 요소를 발견하면 가장 가까운 문서화 주석을 찾음
        + 상위 '클래스'보다, 해당 클래스가 구현한 '인터페이스'를 먼저 찾음
    * 사용이 까다롭고 제약도 조금 있음