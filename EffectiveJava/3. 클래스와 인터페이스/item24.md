# 멤버 클래스는 되도록 static으로 만들라

## 중첩 클래스란
- 다른 클래스 안에 정의된 클래스 (nested class)
- 자기를 감싼 클래스 안에서만 사용되어야 하며, 그 외에 사용되는 곳이 있으면 top level 클래스로 만들어야 함


## 중첩 클래스의 종류
1. (비정적) 멤버 클래스
    * 예시
        ```java
        public class Animal { // 바깥 클래스, 가장 바깥에 있으므로 톱 레벨 클래스
            // ...
            public class Classification { // 중첩 클래스
                // ...
            }
        }
        ```
    * 특징
        - 다른 클래스 안에 선언됨
            + 바깥 클래스가 인스턴스화되지 않았다면 생성 불가능
            + 바깥 클래스의 인스턴스로 생성자 호출 가능
        - 바깥 클래스의 private 멤버에 접근 가능 (바깥 클래스도 마찬가지)
            + ex. `Animal.this.name`
    * 쓰임새
        - 메서드 밖에서도 사용해야 하거나 메서드 안에 정의하기에 너무 긴 경우
        - 인스턴스들이 바깥 인스턴스를 참조함
        - **어댑터** 정의
            + 어떤 클래스의 인스턴스를 감싸서 다른 클래스의 인스턴스처럼 보이게 하는 뷰로 사용
            + ex. Map 인터페이스의 구현체들이 제공하는 `컬렉션 뷰`
            + ex. Set/List 등의 컬렉션 인터페이스의 구현체들에서 반복자를 구현하는 경우
2. 정적 멤버 클래스
    * 예시
        ```java
        public class Animal {
            private String name = "cat";

            // 열거 타입도 암시적 static
            public enum Kinds {
                MAMMALS, BIRDS, FISH, REPTILES, INSECT
            }

            private static class PrivateSample {
                private int temp;

                public void method() {
                    Animal outerClass = new Animal();
                    System.out.println("private" + outerClass.name);
                    // 바깥 클래스인 Animal의 private 멤버 접근
                }
            }

            public static class PublicSample {
                private int temp;

                public void method() {
                    Animal outerClass = new Animal();
                    System.out.println("public" + outerClass.name);
                    // 바깥 클래스인 Animal의 private 멤버 접근
                }
            }
        }
        ```
        - static으로 선언된 멤버 클래스
            + 바깥 클래스가 인스턴스화되지 않았어도 생성 가능
        - 바깥 클래스가 표현하는 객체의 **한 부분**일 때 사용
            + ex. Map의 Entry (모든 엔트리는 맵과 연관되나, 엔트리의 메서드가 맵을 사용하진 않음)
            + ex. 모든 클래스 안의 열거 타입 (item34)
    * 특징
        - **바깥 인스턴스에 접근할 일이 없으면 가능한 static을 붙이자**
            + 바깥 인스턴스 - 멤버 클래스 관계(참조)를 위한 시간과 공간 소모
            + Garbage Collection이 바깥 클래스의 인스턴스 수거 불가 -> 메모리 누수 발생 (item7)
            + 참조가 눈에 보이지 않아 개발이 어려움
    * 쓰임새
        - 메서드 밖에서도 사용해야 하거나 메서드 안에 정의하기에 너무 긴 경우
        - 인스턴스들이 바깥 인스턴스를 참조하지 않음 (독립적으로 존재함)
3. 익명 클래스
    * 예시
        ```java
        Collections.sort(list, new Comparator<Integer>() {
            @Override
            public int compare(Integer n1, Integer n2) {
                return n1 - n2;
            }
        });
        ``` 
    * 특징
        - 이름이 없는 클래스
        - 응용하는 데 제약이 많음
            * 비정적인 문맥에서 사용될 때만 바깥 클래스의 인스턴스 참조 가능
                + static으로 선언된 메소드에서는 static만 접근이 가능
            * 정적 문맥에서 사용될 때에는 static final 외의 정적 멤버 가질 수 없음
                + 그러나 이 정적 멤버를 바깥에서는 사용 불가능
            * 선언한 지점에서만 인스턴스를 만들 수 있음
            * 클래스의 이름이 필요한 작업(ex. instanceof) 불가능
            * 상속 불가능
            * expression 중간에 등장하므로 가독성을 위해 짧게 작성해야 함
    * 쓰임새
        - 한 메서드 안에서만 쓰이면서 그 인스턴스를 생성하는 지점이 단 한 곳인 경우
        - 해당 타입으로 쓰기에 적절한 클래스/인터페이스가 이미 있음
        - 람다를 지원하기 전에는 즉석에서 작은 함수 객체 등이 필요할 경우 사용했음 (item42)
        - 정적 팩터리 메서드를 구현할 때도 사용함 (item20, 코드20-1)
4. 지역 클래스
    * 예시
        ```java
        public class TestClass {
            private int number = 10;

            public TestClass() {
            }

            public void foo() {
                // 지역변수처럼 선언 (유효범위도 지역변수와 같음)
                class LocalClass {
                    private String name;
                    // private static int staticNumber; // 정적 멤버 가질 수 없음

                    public LocalClass(String name) {
                        this.name = name;
                    }

                    public void print() {
                        // 비정적 문맥에선 바깥 인스턴스를 참조 가능
                        // foo()가 static이면 number에서 컴파일 에러
                        System.out.println(number + name);
                    }
                }
                LocalClass localClass1 = new LocalClass("local1"); // 이름이 있고
                LocalClass localClass2 = new LocalClass("local2"); // 반복해서 사용 가능
            }
        }
        ```
        - 드물게 사용됨
        - 변수처럼 사용되는 만큼, 가독성을 위해 짧게 작성할 것
    * 쓰임새
        - 한 메서드 안에서만 쓰이면서 그 인스턴스를 생성하는 지점이 단 한 곳인 경우
        - 해당 타입으로 쓰기에 적절한 클래스/인터페이스가 없음


## 참고자료
- https://yeonyeon.tistory.com/205
- https://stackoverflow.com/questions/18902484/what-is-a-view-of-a-collection