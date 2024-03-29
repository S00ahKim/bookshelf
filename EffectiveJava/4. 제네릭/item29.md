# 이왕이면 제네릭 타입으로 만들라

## 제네릭 타입
- 특정(Specific) 타입을 미리 지정해주는 것이 아닌, 필요에 의해 지정할 수 있도록 하는 일반(Generic) 타입
- 많은 타입을 지원하는 객체를 만들고자 할 때 사용할 수 있음
    * Object로 하면 되지 않아? 그러면 클라이언트는 꺼내어 쓸 때마다 그에 맞게 **형변환을 해줘야** 한다. 클라이언트가 예상하고 있는 것과 다른 타입의 원소가 튀어나오는 경우에는 **런타임 오류**를 발생시킬 수도 있다.
- 암묵적인 네이밍 규칙
    * <T> : Type
    * <E> : Element
    * <K> : Key
    * <V> : Value
    * <N> : Number


## 제네릭과 배열 혼용할 경우 발생하는 에러 해결법
1. (E[]) 로 타입 캐스팅
    * 제네릭 배열 생성을 금지하는 제약을 대놓고 무시하고 우회하는 방법
    * 컴파일 타임에 타입 안전성을 검증할 방법이 없어서 프로그래머가 수동으로 확인해야 함
    * 안전하다고 해도 런타임 타입은 E[]가 아닌 Object[]
2. Object[] 로 변경
    * 값을 돌려주는 메서드에서 (E)로 타입 캐스팅해주는 방법

### 첫번째 방법을 실무에선 선호함
- 가독성이 좋고 코드가 짧으며 형변환이 배열 생성하는 부분 한 군데에만 들어가면 됨
- 그러나 힙 오염이 될 여지가 있음 (item32)

### cf. 한정적 타입 매개변수
- 타입 매개변수에 제약을 두는 제네릭 타입
- 대다수의 제네릭 타입은 기본 타입을 사용할 수 없는 것 외의 여타 제약은 안 두는 편
- 그러나 <E extends Delayed> 이런 식으로 아예 타입 매개변수 자체에 제약을 두어 해당 타입의 하위 타입만 받겠다고 한 뒤에 형변환 없이 Delayed의 메서드들에 접근하게 할 수 있음