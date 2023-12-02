# 지역변수의 범위를 최소화하라
> cf. 클래스와 멤버의 접근 권한을 최소화하라 (item15)


## 장점
- 코드 가독성 up
- 유지보수성 up
- 오류 가능성 down


## 팁
- 가장 처음 쓰일 때 선언하기 (= 미리 선언해두지 말기)
- 선언과 동시에 초기화하기
    * 필요한 값이 충분치 않으면 준비될 때까지 선언을 미루기
    * 예외: try-catch (예외가 블록이 아니라 메서드 단위에서 발생하므로) / try 문 앞에 선언하기
- while보다 for를 사용하는 게 낫다
    ```java
    for (Iterator<Integer> i = c.iterator(); i.hasNext();) {
      Integer number = i.next();
    }
    i.hasNext(); // 컴파일 에러 발생!

    Iterator<Integer> i = c.iterator();
    while (i.hasNext()) {
      doSomething(i.next());
    }

    Iterator<Integer> i2 = c2.iterator();
    while (i.hasNext()) { // 버그 발생!
      doSomething(i2.next());
    }
    ``` 
    * for 문은 순회용 변수를 블록 안에서 선언하고 종료할 수 있다
    * while 문은 순회용 변수를 블록 밖에서 선언해야 하고, 프로그래머가 신경써서 사용해야 한다
    * for 문이 while 문보다 짧다
- 메서드를 기능별로 잘 쪼개서 작게 유지하고 한 가지 기능에 집중한다