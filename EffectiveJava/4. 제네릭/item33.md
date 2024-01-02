# 타입 안전 이종 컨테이너를 고려하라 

## 타입 안전 이종 컨테이너
- type safe heterogeneous container
- `여러 다른 종류들로 이루어진 값`을 저장하는 타입 안전한 객체
- 대개 제네릭을 사용하는 객체(ex. 컬렉션 API)는 지원할 수 있는 **타입 매개변수 개수가 고정**되어 있다.
    * 매개변수화되는 대상은 **컨테이너 자신**이다
    * ex. 학번을 저장하는 `Set<Integer>`: Integer 타입 값만을 저장할 수 있다.
    * ex. 이름과 나이를 저장하는 `Map<String, Integer>`: String 타입 key와 Integer 타입 value만 저장할 수 있다.
- **지원 가능한 타입의 개수가 많아야 하는 경우**에는 타입 안전 이종 컨테이너 패턴을 사용하자.
    * 매개변수화되는 대상은 **키**다
        + 컨테이너에서 값을 넣거나 뺄 때 매개변수화한 키를 함께 제공한다.
        + 제네릭 타입 시스템 값의 타입이 키와 같음을 보장해준다.
    * ex. 데이터 베이스의 타입 종류를 저장하는 `Set`: 모든 Column을 type-safe하게 이용하기 위해


## 예시
```java
/**
 * Favorites: 타입 안전 이종 컨테이너 
 * 타입 안전? String을 요청했는데 Integer를 반환하는 일이 없음
 * 이종 컨테이너? 모든 키의 타입이 제각각이라, 여러 가지 타입의 원소를 담을 수 있다
 */
public class Favorites {
    private Map<Class<?>, Object> favorites = new HashMap<>();

    public <T> void putFavorite(Class<T> type, T instance) {
        favorites.put(Objects.requireNonNull(type), instance);
    }

    public <T> T getFavorite(Class<T> type) {
        return type.cast(favorites.get(type));
    }
}

public static void main(String[] args) {
    Favorites f = new Favorites();

    f.putFavorite(String.class, "Java");
    f.putFavorite(Integer.class, 0xcafebabe);
    f.putFavorite(Class.class, Favorites.class);

    String favoriteString = f.getFavorite(String.class);
    int favoriteInteger = f.getFavorite(Integer.class);
    Class<?> favoriteClass = f.getFavorite(Class.class);

    System.out.printf("%s %x %s%n", favoriteString, favoriteInteger, favoriteClass.getName());
}
```
- 원리
    * **타입 안전 이종 컨테이너는 타입 토큰을 키로 쓴다** (= 각 타입의 Class 객체를 매개변수화한 키 역할로 사용)
    * 타입 토큰? 컴파일 타입 정보와 런타임 타입 정보를 알아내기 위해 메서드들이 주고 받는 class 리터럴
    * Class의 리터럴 타입은 Class가 아닌 Class<T> (ex. String.class는 Class<String>)
- Map<Class<?>, Object>
    * 모든 키가 서로 다른 매개변수화 타입일 수 있다
    * 모든 타입을 커버하기 위해 Object로 선언해뒀지만, 사실 넣어줄 때 타입 검사를 하게 하므로 알아서 잘 들어갈 것이다
- putFavorite
    * 타입 정보를 함께 받아서 키-값 간 타입 일관성은 유지할 수 있게 한다
    * 하지만 키와 값 사이의 타입 링크(type linkage) 정보는 버려진다 (Object로 여겨질 것)
- getFavorite
    * 필요한 타입 정보를 받아서 해당되는 값을 불러온다
    * 불려온 값은 Object이므로 Class의 cast() 메서드를 사용해 객체가 가리키는 타입으로 동적 형변환한다
    * 메소드 시그니처에 리턴 타입을 매개변수 타입과 일치시켜서 타입 안정성을 얻는다


## 제약
1. 악의적인 클라이언트가 Class 객체를 제네릭이 아닌 로타입으로 넘기면 안전성이 깨진다.
    ```java
    // 문제 상황 예시
    // 컴파일은 가능하지만 비검사 경고가 발생 & 런타임에 ClassCastException 발생
    favorites.put((Class) Integer.class, "Invalid Type");
    int value = favorites.getFavorite(Integer.class);

    // 해결 방법
    // type.cast(instance)로 instance의 타입이 type으로 명시한 타입과 같은지 확인
    public <T> void putFavorite(Class<T> type, T instance) {
        favorites.put(Objects.requireNonNull(type), type.cast(instance));
    }
    ```
    * 유사한 해결 방법 적용한 예: Collections.checkedList, Collections.checkedSet, Collections.checkedMap
2. 실체화 불가 타입에는 사용할 수 없다
    * String, String[]은 저장할 수 있지만, List<String>은 저장할 수 없다.
    * List<String>과 List<Integer> 모두 List.class라는 객체를 공유하기 때문이다.
    * 완벽히 만족스러운 우회로는 없다.


## 한정적 타입 토큰
- 만약 타입을 제한하고 싶다면 한정적 타입 토큰을 활용
- 예시: 애너테이션 API
    ```java
    package java.lang.annotation;

    public interface Annotation {
        boolean equals(Object obj);
        int hashCode();
        String toString();
        Class<? extends Annotation> annotationType();
    }
    ```
    * 리플렉션의 대상이 되는 타입들, 즉 클래스(`java.lang.Class<T>`), 메서드(`java.lang.reflect.Method`), 필드(`java.lang.reflect.Field`) 같이 프로그램 요소를 표현하는 타입들에서 `대상 요소에 달려 있는 어노테이션을 런타임에 읽어오는 기능`을 아래 메서드로 구현한다.
      ```java
      public <T extends Annotation> T getAnnotation(Class<T> annotationType);
      ``` 
- 한정적 타입 토큰을 타입 세이프하게 사용하기
    * 여기서 그냥 Class<?>와 같이 비한정적 와일드카드 타입을 한정적 타입 토큰을 받는 메서드에 전달하면 객체를 Class<? extends Annotation>으로 형변환하겠지만 비검사 경고 문구가 뜬다.
    * Class 클래스는 형변환을 안전하게, 동적으로 수행해주는 인스턴스 메서드 asSubClass를 제공한다.
      ```java
      public <U> Class<? extends U> asSubclass(Class<U> clazz) {
          if (clazz.isAssignableFrom(this))
              return (Class<? extends U>) this;
          else
              throw new ClassCastException(this.toString());
      }
      ```
      + asSubclass는 호출된 인스턴스 자신의 Class 객체를 인수가 명시한 클래스로 형변환한다
      + 된다면 자신이 인수로 들어온 타입의 하위 클래스라는 것
      + 성공하면 인수로 들어온 타입 객체를, 실패하면 예외를 발생시킨다
    * 컴파일 시점에서 타입을 알 수 없는 어노테이션을 asSubclass 메서드를 사용해 런타임에 읽어내는 예
      ```java
      static Annotation getAnnotation(AnnotatedElement element, String annotationTypeName) {
          Class<?> annotationType = null; //비한정적 타입 토큰
        
          try {
              annotationType = Class.forName(annotationTypeName);
          } catch (Exception ex) {
              throw new IllegalArgumentException(ex);
          }

          return element.getAnnotation(annotationType.asSubclass(Annotation.class));
      }
      ```