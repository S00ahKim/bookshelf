# 비트 필드 대신 EnumSet을 사용하라
> 열거한 값들이 단독값이 아니라 집합으로 사용될 경우

## 비트 필드
```java
public class Text {
    public static final int STYLE_BOLD = 1 << 0; // 1 0001
    public static final int STYLE_ITALIC = 1 << 1; // 2 0010
    public static final int STYLE_UNDERLINE = 1 << 2; // 4 0100
    public static final int STYLE_STRIKETHROUGH = 1 << 3; // 8 1000

    public void applyStyles(int styles) { ... }
}

text.applyStyles(STYLE_BOLD | STYLE_ITALIC); //  "굵게"와 "기울임꼴" 스타일이 적용됨 0011
```
- 서로 다른 2의 거듭제곱 값을 할당한 정수 열거 패턴(item34)을 사용해왔음
- 장점: 비트별 연산 (합집합, 교집합) 을 효율적으로 수행할 수 있음
- 단점: 정수 열거 상수의 단점 + 해석하기 어려움 + 순회하기 어려움 + 최대 필요 비트를 API 작성시 예측 필요


## EnumSet
```java
public class Text{
    public enum Style {
        BOLD, ITALIC, UNDERLINE, STRIKETHROUGH
    }

    public void applyStyles(Set<Style> styles){ ... }
    // 어떤 Set을 넘겨도 되지만, EnumSet이 최선이다.
    // 선언을 Set으로 한 것은 이왕이면 인터페이스로 받는 게 좋은 습관이기 때문 (item64)
}

text.applyStyles(EnumSet.of(Style.BOLD, Style.ITALIC));
```
- java.util
- 열거 타입 집합을 효율적으로 표현함
- 장점: Set 인터페이스 구현 & 타입 안전 & 비트 벡터로 내부 구현되어 좋은 성능 & 난해한 작업을 EnumSet이 해결
- 단점: 불변을 만들 수 없음. `Collections.unmodifiableSet`으로 감싸주거나 `Guava` 라이브러리를 사용해야 함.