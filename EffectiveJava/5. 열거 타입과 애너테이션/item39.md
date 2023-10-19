# 명명 패턴보다 애너테이션을 사용하라
> 당신이 다른 프로그래머가 소스코드에 추가 정보를 제공하는 도구를 만드는 일을 한다면 애너테이션을 제공하라

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


## 예시로 알아보기(1): 자체제작 테스트 애너테이션 @Test
> 테스트 메서드를 명시해주는 애너테이션

### 선언
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

### 처리
```java
public class RunTests {
    public static void main(String[] args) throws Exception{
        int tests = 0;
        int passed = 0;
        Class<?> testClass = Class.forName(args[0]);
        for (Method m : testClass.getDeclaredMethods()) {
            if (m.isAnnotationPresent(Test.class)) { //Test 애너테이션이 선언된 메서드를 찾음
                tests++;
                try {
                    m.invoke(null);
                    passed++;
                } catch (InvocationTargetException wrappedExc) {
                    // 테스트 메서드가 예외를 던지면 리플렉션 매커니즘이 InvocationTargetException로 싸서 다시 던짐
                    Throwable exc = wrappedExc.getCause(); // 원래 예외의 실패 정보를 추출
                    System.out.println(m + " 실패 : " + exc);
                } catch (Exception e) {
                    /*
                     * 애너테이션에게 매개변수 없는 정적 메서드에만 사용하는 것은
                     * 강제된 사항은 아니기 때문에 따로 구현이 필요하나 생략된 것으로 보임
                     * 예) !element.getModifiers().contains(Modifier.STATIC) 등으로 확인하는 부분
                     **/
                    System.out.println("잘못 사용한 테스트 @Test : " + m);
                }
            }
        }
        System.out.printf("성공 : %d, 실패 : %d%n", passed, tests - passed);
    }
}
```

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

    // 실패 (잘못된 사용)
    @Test
    public void m4() { }
}
```


## 예시로 알아보기(2): 자체제작 테스트 애너테이션 @ExceptionTest
> 매개변수로 받은 특정 예외를 던져야만 성공하는 테스트를 명시해주는 애너테이션

### 선언
```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
    Class<? extends Throwable> value(); //매개변수 선언
}
```
- Throwable을 확장한 클래스의 Class 객체 (=모든 예외 수용) (item33)

### 처리
```java
public class RunExceptionTests {
    public static void main(String[] args) throws Exception{
        int tests = 0;
        int passed = 0;
        Class<?> testClass = Class.forName("example.Sample2");
        for (Method m : testClass.getDeclaredMethods()) {
            if (m.isAnnotationPresent(ExceptionTest.class)) {
                // ㄴExceptionTest 애너테이션을 사용한 메서드 선별
                tests++;
                try {
                    m.invoke(null);
                } catch (InvocationTargetException wrappedExc) {
                    Throwable exc = wrappedExc.getCause();
                    Class<? extends Throwable> excType = m.getAnnotation(ExceptionTest.class).value();    // 애너테이션의 매개변수 타입 확인
                    if (excType.isInstance(exc)) {    // 애너테이션의 매개변수 타입과 같을 경우 통과
                        passed++;
                    }
                } catch (Exception e) {
                    System.out.println("잘못 사용한 테스트 @Test : " + m);
                }
            }
        }
        System.out.printf("성공 : %d, 실패 : %d%n", passed, tests - passed);
    }
}
```

### 사용 예시
```java
public class Sample2 {
    // 성공
    @ExceptionTest(ArithmeticException.class)   
    public static void m1() {
        int i = 1 / 0;
    }

    // 실패 (명시한 예외가 아닌 다른 예외를 던짐)
    @ExceptionTest(ArithmeticException.class)
    public static void m2() {
        int[] arr = new int[0];
        arr[1] = 1;     // outOfIndex 예외 발생
    }

    // 실패 (예외를 던지지 않음)
    @ExceptionTest(ArithmeticException.class)
    public static void m3() {}
}
```

### 매개변수를 여러 개 주고 싶다면
1. 배열로 넘기기
    1. 애너테이션 선언을 배열로 수정한다 ex. `Class<? extends Throwable>[] value();`
    2. 매개변수가 여러개 필요한 애너테이션에 배열로 값을 넘긴다 ex. `@ExceptionTest({FooException.class, BarException.class})`
    3. 이 방법은 단일 매개변수도 유연하게 받아들인다 ex. `@ExceptionTest(FooException.class)`
    4. 애너테이션 처리 코드에 배열을 처리할 수 있도록 대응한다 (p243)
2. 애너테이션을 여러번 선언하기 (자바8~, 가독성 좋으나 조금 복잡하다)
    ```java
    @Retention(RetentionPolicy.RUNTIME)
    @Target(ElementType.METHOD)
    @Repeatable(ExceptionTestContainer.class)
    public @interface ExceptionTest {
        Class<? extends Throwable> value();
    }

    //컨테이너 애너테이션 
    @Retention(RetentionPolicy.RUNTIME)
    @Target(ElementType.METHOD)
    public @interface ExceptionTestContainer {
        ExceptionTest[] value();
    }

    // 사용 예시
    @ExceptionTest(IndexOutOfBoundsException.class)
    @ExceptionTest(NullPointerException.class)
    public static void doublyBad() { }
    ``` 
    1. 애너테이션 선언에 `@Repeatable` 메타애너테이션을 달아준다
    2. 해당 애너테이션을 리턴하는 컨테이너 애너테이션을 정의하고, @Repeatable에 매개변수로 넘긴다
    3. 컨테이너 애너테이션에 내부 애너테이션 타입의 배열을 리턴하는 value 메서드를 작성한다
    4. 컨테이너 애너테이션에도 마찬가지로 @Retention, @Target을 명시한다
    5. 처리 코드에 isAnnotationPresent 가 @해당애너테이션 @컨테이너애너테이션 모두를 검사하게 한다
        * 만약 @해당애너테이션 만 검사하면 여러개 달린 건 컨테이너로 들어와서 못 잡아냄
        * 만약 @컨테이너애너테이션 만 검사하면 하나 달린 건 원본으로 들어와서 못 잡아냄
        * (p244-245)