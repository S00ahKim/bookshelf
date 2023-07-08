# try-finally 보다는 try-with-resources를 사용하라

## 자원이란
- resource
- 제한된 물리적 부품 및 가상 구성 요소를 아울러 이르는 말. CPU, 파일, 네트워크 소켓, 메모리 등.
- 자바의 경우, 외부 자원을 편리하게 사용하도록 각종 객체를 지원한다. `InputStream`, `OutputStream`, `java.sql.Connection` 등.
- 자원을 제때 해제하지 않으면 `메모리 누수` 등 성능 문제 발생 가능
- 자바의 경우, `Garbage Collector`가 있기 때문에 사용하지 않는 자원을 자동으로 해제해 주기도 한다.
    * 그러나, GC가 인식하지 못하는 경우가 있고
    * 언제 해제되는지 예측하기 어렵다.
- 자원 해제의 안전망으로 finalizer를 사용하지만, 이는 믿음직하지 못함 (item08)


## 전통적인 방법, try-finally
```java
try {
    // 여기 또는 try 블록 바깥에 자원 선언
    // 여기부터 자원 사용 가능
}
catch(예외 e) {
    // 예외 핸들링
}
finally {
    // 자원 close
}
```
- try에서 예외가 발생해도 finally가 실행되므로 정상적으로 자원 해제 가능
- 단점(1) 가독성: 사용하는 자원이 하나 이상이면 코드가 복잡해짐 
- 단점(2) 어려운 디버깅: 예외가 finally 블록에서 발생한다면 디버깅이 어려움 (최초 발생 에러가 아니라, 최종 발생 에러를 출력)
- 단점(3) 실수 발생 가능: 사용하는 자원이 하나 이상이면 실수로 close를 누락 가능


## 더 나은 방법, try-with-resources
```java
try(여기에 자원 선언) {
    // 여기 안에서 자원 사용 가능
}
catch(예외 e) {
    // 예외 핸들링
}
```
- 자바 7 부터 지원
- 블록의 코드를 모두 실행하고 나면 사용한 자원을 자동으로 해제함
- 선언할 자원은 반드시 `java.lang.AutoCloseable` 를 구현해야 한다.
    * cf. void 를 리턴하는 close 메서드만 갖고 있음
    * 이미 상당수 자바 & 서드파티 라이브러리에서 구현함
    * 닫아야 하는 자원을 뜻하는 클래스를 만들 때 반드시 구현할 것
- 여러 자원을 넘겨줄 때에는 세미콜론(;)으로 구분한다.
    * ```java
      private static void printFile() throws IOException {
          try(  FileInputStream     input         = new FileInputStream("file.txt");
                BufferedInputStream bufferedInput = new BufferedInputStream(input)
          ) {
              int data = bufferedInput.read();
              while(data != -1){
                  System.out.print((char) data);
                  data = bufferedInput.read();
              }
          }
      }
      ```
- 장점(1) 가독성: 짧고 읽기 수월함
- 장점(2) 쉬운 디버깅: 원래는 씹혀서 안보이던 에러도 스택 트레이스에 suppressed(숨겨짐)를 달고 출력됨
    * 자바7에서 Throwable에 추가된 getSuppressed()를 사용하면 프로그램 코드에서 가져올 수 있음
- 장점(3) 실수 방지: 빠뜨리지 않고 자원을 해제할 수 있음