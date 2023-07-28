# 상속보다는 컴포지션을 사용하라
> 여기서 말하는 '상속'은 오직 구체 클래스 상속임. 인터페이스 구현과 무관함.


## 컴포지션이 나은 이유
- 상속은 캡슐화를 깨뜨린다
    1. 상위 클래스 때문에 **하위 클래스에 이상이 생길 수 있다**
        * ex. 다음 릴리스에 갑자기 내부 구현이 변경된다면?
        * ex. 상위 클래스가 예상한 방식대로 구현되어 있지 않다면?
          ```java
          // HashSet이 처음 생성된 이후 원소가 몇 개 더해졌는지 알고 싶어서 작성
          public class InstrumentedHashSet<E> extends HashSet<E> {
              //추가된 원소의 수
              private int addCount = 0;
              
              public InstrumentedHashSet(int initCap, float loadFactor){
                  super(initCap, loadFactor);
              }
              
              @Override 
              public boolean add(E e) {
                  addCount++;
                  return super.add(e);
              }
              
              @Override
              public boolean addAll(Collection<? extends E> c) {
                  addCount += c.size();
                  return super.addAll(c);
              }
              
              public int getAddCount() {
                  return addCount;
              }
              
          }

          InstrumentedHashSet<Stirng> s = new InsetrumentedHashset<>();
          s.addAll(List.asList("a", "b", "c"));
          // expected: 3
          // actual: 6
          // 이유: addAll()이 add()를 사용해서 구현되어 있기 때문
          ```
    2. 하위 클래스가 **메서드 재정의를 하면서 오류 발생 & 성능 저하 가능성**이 높아진다
        * ex. 상위 클래스 변경에 영향을 받지 않기 위해 하위 클래스에서 구현을 다시 하는 경우
          ```java
          // ex. 위의 예시에서 addAll()을 자체구현하는 경우
          ``` 
        * ex. 다음 릴리스에 새로 추가된 메서드에 대해 하위 클래스가 재정의를 빠뜨려서 예상치 못한 구멍이 생기는 경우
          ```java
          // 보안 때문에 컬렉션에 추가하기 전 검증하는 부분을 추가한 예시
          public class SuperSafeCollection<E> extends Collection<E> {
              ...
            
              @Override
              public boolean add(E e) {
                  검증(e);
                  super.add(e);
              }

              @Override
              public boolean addAll(Collection<? extends E> c) {
                  검증(c);
                  super.addAll(c);
              }

              // 다음 릴리스에는 add, addAll 외에 addAllStrings가 추가되었다고 가정하면,
              // SuperSafeCollection은 addAllStrings에 대한 검증을 진행하지 못할 수 있다.
              // cf. 실제로도 컬렉션 프레임워크에 HashTable, Vector의 추가로 보안 구멍을 수정해야 했다
          } 
          ``` 
    3. **하위 클래스가 추가한 메서드를 상위 클래스에서 사용할 경우 오류**가 발생한다
        * ex. 시그니처는 같고 반환 타입은 다르다: 컴파일 불가능
          ```java
          // 상위 클래스
          public boolean isCheck(String foo) {...}

          // 하위 클래스
          public void isCheck(String foo) {...}
          ``` 
        * ex. 시그니처와 반환 타입이 모두 같다: 재정의한 셈 (위의 문제 반복 + 오히려 상위 클래스 위반)
          ```java
          // 상위 클래스
          public boolean isCheck(String foo) {...}

          // 하위 클래스
          public boolean isCheck(String foo) {...}
          ```
- 이상의 문제를 피하기 위해 **컴포지션**을 사용하라
    * 뜻? 기존 클래스가 새로운 클래스의 구성요소(composition)로 사용되는 데서 비롯함
    * 어떻게? 새로운 클래스를 만들고 private 필드로 기존 클래스의 인스턴스를 참조하라
      ```java
      // 임의의 Set에 계측 기능을 추가해서 새로운 Set으로 만드는 것이 이 클래스의 핵심
      public class InstrumentedHashSet<E> extends ForwardingSet<E> {
          // 추가된 원소의 수
          private int addCount = 0;
          
          public InstrumentedHashSet(Set<E> s){
            super(s);
          }
          
          @Override 
          public boolean add(E e) {
              addCount++;
              return super.add(e);
          }
          
          @Override
          public boolean addAll(Collection<? extends E> c) {
              addCount += c.size();
              return super.addAll(c);
          }
          
          public int getAddCount() {
              return addCount;
          }
      }

      // 전달 메서드만으로 이루어진 전달 클래스
      // Set 인터페이스를 구현해서 견고하면서 유연하다
      public class ForwardingSet<E> implements Set<E> {
          private final Set<E> s; 
          public ForwardingSet(Set<E> s) { this.s= s;}
          // ㄴ Set의 인스턴스를 인자로 받는 생성자 제공
          
          public void clear() {s.clear();}
          public boolean contains(Object o) { return s.contains(o);}
          public boolean isEmpty() { return s.isEmpty();}
          public int size() { return s.size();}
          public Iterator<E> iterator() { return s.iterator(); }
          public boolean add(E e) { return s.add(e); }
          public boolean addAll(Collection<? extends E> c) { return s.addAll(c); }
          ...
      }
      ``` 
    * 기존 클래스의 메서드를 사용하는 방법? 인스턴스 메서드(=전달 메서드)가 기존 클래스의 메서드를 호출(=전달)하게 하기
    * 장점?
        + 내부 구현을 불필요하게 노출하지 않음
            - 노출시, API는 내부 구현에 묶이고 성능도 제한되며 클라이언트가 접근 가능해져 혼란 유발
        + 특정 구체 클래스가 아니라 지원하고 싶은 인터페이스의 모든 구체 클래스를 지원 가능
          ```java
          // Set의 구현이면 모두 커버 가능할 만큼 유연함
          Set<Instant> times = new InstrumentedSet<>(new TreeSet<>(cmp));
          Set<E> s = new Instrumented<>(new HashSet<>(INIT_CAPACITY));
          ```
    * 더 알아보기
        + 다른 Set인스턴스를 감싸고 있어서 이런 클래스를 Wrapper Class 라고도 함
        + 다른 Set에 계측 기능을 덧씌운다는 의미에서 Decorator Pattern이 적용되었다고도 함
        + 컴포지션 + 전달 조합에서 래퍼 객체가 내부 객체에 자기 자신의 참조를 넘기는 경우 위임(Delegation)이라고도 함


## 그래도 상속하고 싶을 때 고려할 것들
1. 상위 클래스와 하위 클래스가 is-a 관계인가?
    * A가 B를 상속하려고 한다면, A가 정말 B인지 자문하라
    * is-a 관계가 아님에도 상속했다가 곤란해진 자바 플랫폼 라이브러리의 예
        + ex. Stack is NOT Vector => `Stack extends Vector`
            - [Vector의 동기화 문제로 인한 성능 이슈를 전혀 상관도 없는 스택이 같이 지고 있다](https://jaehee329.tistory.com/27)
        + ex. Properties is NOT HashTable => `Properties extends HashTable`
            - Properties는 키와 값으로 문자열만 받으려고 했다. (`p.getProperty(key)`)
            - 그러나 HashTable은 다른 타입도 허용한다. (`p.getKey(key)`)
            - 사용자는 문자열 외의 타입을 Properties의 키와 값으로 사용한다.
              ```java
              Properties p = new Properties();
              p.put(123L, 1234L);
              // 이 코드에는 이상이 없다!
              ``` 
            - 한참 후에야 문제가 밝혀서 이 상황을 수습하기 어렵게 되었다.
2. 상속하려는 클래스가 확장할 목적으로 설계되었고, 문서화도 잘 된 경우 (item19) 인가?
3. 상속하려는 클래스의 API에는 아무런 결함이 없는가?
4. 결함이 있다면 그 결함이 작성하려는 클래스의 API에 전파되어도 괜찮은가?
    * 컴포지션은 결함을 숨길 수 있다.
    * 상속은 결함을 숨길 수 없다. 다른 패키지의 클래스라면 수정도 어렵다!