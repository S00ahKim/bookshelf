# null이 아닌, 빈 컬렉션이나 배열을 반환하라

## 성능의 문제는 아니다
- null이 성능 저하의 주범이라고 확인되지 않는 한(item67) 빈 컬렉션이나 배열을 반환하는 것과 null을 반환하는 것 간에는 큰 성능차가 없음
    * **굳이 null을 쓸 이유가 없다**는 의미
- 드물게 빈 컬렉션 할당이 성능을 눈에 띄게 떨어뜨릴 때가 있을 수 있는데, 이러면 **빈 컬렉션을 불변으로 두고 그걸 반환**하면 됨
    ```java
    return Collections.emptyList();

    return new Cheese[0];

    private static final Cheese[] EMPTY_CHEESE_ARRAY = new Cheese[0];
    return EMPTY_CHEESE_ARRAY;

    return new ArrayList<>(cheeseInStock);
    ```

## 그럼 왜 null을 피하나?
- 항상 방어 코드를 넣어주어야 함
- 반환하는 쪽에서도 null발생 상황을 특별 취급해야 함
- 위와 같은 이유로 코드가 복잡해지고, 오류 발생 가능성이 높아짐