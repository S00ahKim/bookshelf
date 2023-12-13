# 다중정의는 신중히 사용하라

## 재정의(Override)한 메서드는 동적으로 선택된다
- 상위 클래스가 정의한 것과 같은 시그니처의 메서드를 하위 클래스에서 다시 정의한 것
- 메서드를 재정의한 다음 '하위 클래스 인스턴스'에서 그 메서드를 호출하면 **재정의한 메서드가 실행**되며, 컴파일 타임의 인스턴스 타입은 신경쓰지 않는다.


## 다중정의(Overloading)한 메서드는 정적으로 선택된다
```java
public class CollectionClassifier {
    public static String classify(Set<?> s) {
        return "집합";
    }

    public static String classify(List<?> lst) {
        return "리스트";
    }

    public static String classify(Collection<?> c) {
        return "그 외";
    }

    public static void main(String[] args) {
        Collection<?>[] collections = {
                new HashSet<String>(),
                new ArrayList<BigInteger>(),
                new HashMap<String, String>().values()
        };

        for (Collection<?> c : collections)
            System.out.println(classify(c)); // "그 외", "그 외", "그 외"
    }
}
```
- 어느 메서드를 사용할지 컴파일 타임에 정해졌기 때문에 c가 Collection으로 인식되어서임
- 의도한 대로 사용하려면 classify 메서드를 하나로 합친 후 instanceof로 명시적으로 검사해야 함


### List의 다중정의
```java
public class SetList {
    public static void main(String[] args) {
        Set<Integer> set = new TreeSet<>();
        List<Integer> list = new ArrayList<>();

        for (int i = -3; i < 3; i++) {
            set.add(i);
            list.add(i);
        }
        for (int i = 0; i < 3; i++) {
            set.remove(i);
            list.remove(i);
        }
        System.out.println(set + " " + list); 
        // [-3, -2, -1] [-2, 0, 2] 
    }
}
```
- List<E> 인터페이스는 remove(Object)와 remove(int)로 다중정의함
- set.remove(i)의 시그니처는 remove(Object)이며, 다중정의된 다른 메서드가 없으니 기대대로 동작함
- 반면, list.remove(i)는 다중정의된 remove(int idex)를 선택하여 원소가 아닌 "지정한 위치"의 원소를 제거함
- 의도한 대로 사용하려면 Integer로 형변환하면 됨


### 람다의 다중정의
```java
// 1. Thread의 생성자 호출
new Thread(System.out::println).start();

// 2. ExecutorService의 submit 메서드 호출
ExecutorService exec = Executors.newCachedThreadPool();
exec.submit(System.out::println); // 컴파일 에러
```
- 다중정의 메서드들(혹은 생성자)들이 함수형 인터페이스를 인수로 받을 때, 서로 다른 함수형 인터페이스라도 인수 위치가 같으면 컴파일 실패할 수 있다. println, submit 모두 다중정의되어 적절한 다중정의 메서드를 찾는 알고리즘이 찾기 어려워하는 경우다.


## 팁
- 매개변수 수가 같은 다중정의는 만들지 마라
- 가변인수(varags)를 사용하는 메서드라면 다중정의를 아예 하지 말자
- 다중정의하는 대신 메서드 이름을 다르게 지어주자
    * ex. writeBoolean(boolean), writeInt(int), writeLong(long)
    * 생성자는 이름을 다르게 지을 수 없으니 두번째 생성자부터는 다중정의가 되지만, 대신 정적팩터리를 사용할 수 있다 (item1)
- 완전히 똑같은 일을 해준다면 다중정의가 되어 있어도 크게 신경쓸 것까지는 없다
    * 다른 일을 할 때가 문제지... ex. String의 `valueOf(char[])`, `valueOf(Object)`