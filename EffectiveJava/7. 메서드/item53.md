# 가변인수는 신중히 사용하라

## 무엇을, 왜, 어떻게
- 가변인수(varargs)란? 명시한 타입의 인수를 0개 이상 받을 수 있음
    * 자바1.5~
    * 가변인수 메서드 호출 시, 인수의 개수와 길이가 같은 배열을 만들고 인수들을 이 배열에 저장하여 가변인수 메서드에 전달함
    * 인수 개수는 런타임에 자동 생성된 배열의 길이 args.length로 알 수 있음
- 왜? 인수 개수가 일정하지 않은 메서드를 정의할 때
- 어떻게? 필수 매개변수는 가변인수 앞에, 성능 문제를 고려해서


## 가변인수 사용의 단점
- 0개를 넘길 수는 있으나, 0개를 넘기면 컴파일이 아닌 런타임에 실패한다
- args 유효성 검사를 명시적으로 해야하고, 코드도 지저분하다
- 가변인수 메서드는 호출될 때마다 배열을 새로 하나 할당하고 초기화하므로 성능에 민감하다면 사용을 지양하라


## 대안?
- 차라리 인수를 2개 이상 받게 하는 것이 좋을 수 있다
- 또는 다중 정의(오버로딩) 메서드를 여럿 제공하고, 적은 케이스에만 가변인수를 사용하게 유도할 수 있다 (ex. `EnumSet`)
```java
public static <E extends Enum<E>> EnumSet<E> of(E e) {
    EnumSet<E> result = noneOf(e.getDeclaringClass());
    result.add(e);
    return result;
}

...

public static <E extends Enum<E>> EnumSet<E> of(E e1, E e2, E e3, E e4, E e5) {
    EnumSet<E> result = noneOf(e1.getDeclaringClass());
    result.add(e1);
    result.add(e2);
    result.add(e3);
    result.add(e4);
    result.add(e5);
    return result;
}

@SafeVarargs
public static <E extends Enum<E>> EnumSet<E> of(E first, E... rest) {
    EnumSet<E> result = noneOf(first.getDeclaringClass());
    result.add(first);
    for (E e : rest)
        result.add(e);
    return result;
}
```