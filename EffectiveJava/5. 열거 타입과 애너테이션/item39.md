# 명명 패턴보다 애너테이션을 사용하라

## 명명 패턴
```java
//JUnit3 : 메서드 이름을 test로 시작해야함
public class helloTest extends TestCase {
    public void testHello(){
        String hello = "Hello!"
        giveMeHello(hello);
    }

    public void testHello_FooException() {
        String bye = "Bye!";
        try {
            giveMeHello(bye);
        } catch (FooException e) {
            System.out.println("good!");
        }
    }
}
```
- 전통적으로 특별히 다루는 프로그램 요소를 구분하기 위해 사용한 방법
- ex. `JUnit3`는 테스트 메서드의 경우 메서드 이름을 `test`로 시작하도록 함 & `TestCase` 클래스 상속받도록 함
- 단점
    1. 오타가 나면 안 됨 (`tsetFoo`는 프레임워크가 인식하지 못함)
    2. 잘 사용되리라 보장이 안 됨 (클래스 이름을 `TestFooMapper`로 지어두고 내부 메소드 실행을 기대할수도)
    3. 매개변수 전달이 어려움 (예외 발생을 테스트하는 경우, 예외 타입을 테스트에 명시적으로 넘겨줄 수 없음)
       * 보기도 나쁘고 깨지기도 쉽다 (item62) 


## 애너테이션
```java
//JUnit4 : @Test 애너테이션을 사용해 테스트 
public class helloTest  {
    @Test
    public void testHello(){
        String hello = "Hello!"
        giveMeHello(hello);
    }

    @Test(expected=FooException.class)
    public void testBye() {
        String bye = "Bye!";
        giveMeHello(bye);
    }
}
```


## 예시로 알아보기: 자체제작 테스트 애너테이션 @Test
### 애너테이션 선언
```java
/**
 * 이 에너테이션은 매개변수 없는 정적 메서드에만 달 수 있는 마커 에너테이션이다.
 */
// ㄴ 주석으로 사용 방법을 알려준다
@Retention(RetentionPolicy.RUNTIME) // 보존 정책 명시 (런타임에도 유지하라)
@Target(ElementType.METHOD) // 적용 대상 명시 (메서드에만 달 수 있다)
public @interface Test {
}
```
- 메타 애너테이션: 애너테이션 선언에 다는 애너테이션. ex. @Retention, @Target
- 마커 애너테이션: 별다른 매개변수 없이 단순히 마킹하는 용도로 사용하는 애너테이션 ex. 예시의 @Test

### 사용 예시
```java
public class Sample {
    // 성공
    @Test
    public static void m1(){}

    // 무시됨
    public static void m2(){}

    // 실패 (약속하지 않은 예외 던짐)
    @Test
    public static void m3(){
        throw new RuntimeException("실패");
    }

    // 
    @Test
    public void m4() { }
}
```

### 마커 애너테이션 처리
```java
public class RunTests {
    public static void main(String[] args) throws Exception{
        int tests = 0;
        int passed = 0;
        Class<?> testClass = Class.forName(args[0]);
        for (Method m : testClass.getDeclaredMethods()) {
            if (m.isAnnotationPresent(Test.class)) {    //Test 애너테이션이 선언된 메서드만을 호출한다.
                tests++;
                try {
                    m.invoke(null);
                    passed++;
                } catch (InvocationTargetException wrappedExc) {
                    Throwable exc = wrappedExc.getCause();
                    System.out.println(m + " 실패 : " + exc);
                } catch (Exception e) {
                    System.out.println("잘못 사용한 테스트 @Test : " + m);
                }
            }
        }
        System.out.printf("성공 : %d, 실패 : %d%n", passed, tests - passed);
    }
}
```