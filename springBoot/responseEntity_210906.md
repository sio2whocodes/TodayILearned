### SpringBoot에서 ResponseEntity사용하기

ResponseEntity는 Spring Framework에서 제공하는 **HttpEntity** 클래스를 상속받아 HttpStatus, HttpHeaders, HttpBody을 변수로 갖는 클래스이다.

이 클래스를 우리는 서버에서 클라이언트로 응답을 전송할 때 사용한다.

HttpStatus는 흔히 우리가 알고 있는 200, 404와 같은 HTTP상태코드를 담는 변수이다.

HttpBody에는 서버에서 (클라이언트가 요청한) 작업을 처리하고 그 결과를 담아 전송하는데에 사용되는 변수이다. *HttpBody는 전달하고자 하는 데이터를 담는데 어떤 객체든 대입할 수 있게끔 제네릭타입으로 정의되어있다.

정리하면 ResponseEntity는 서버의 처리결과를 클라이언트에 전송할때 사용한다. 즉 클라이언트가 요청한 일련의 작업을 처리한 후에 그 결과를 ResponseEntity에 담아 클라이언트로 전송한다.

### ResponseData (가칭)

우리가 여기서 구현할 부분은 ResponseEntity의 멤버 변수 중 HttpBody에 대입할 객체의 클래스를 정의하는 것이다.

위에서 언급했다시피 HttpBody는 ResponseEntity 클래스 내에 제네릭타입으로 정의되어있다. 따라서 어떤 객체든 대입할 수 있다. 그래서 우리는 HttpBody에 대입할 우리만의 ResponseData(가명) 클래스를 만들어서 사용하고자 한다.


우선 Body에 담아 보내고 싶은 데이터는

1. **상태코드**(HttpStatus에 있는데 걍 한번 더 넣어줌)
2. **디테일한 상태 정보**
3. **결과 데이터**(있다면)

이다.

우리는 상태코드와 디테일한 상세 정보를 담고있는 ResponseCode라는 Enum 객체를 사용했다.

```java
@Getter
@Builder
@ToString
public class ResponseData<T> {

  private final LocalDateTime timestamp = LocalDateTime.now();
  private final ResponseCode code;
  private T data;

}
```

짠. 만들었다.

우리는 현재 시간을 담아두는 timestamp라는 변수를 선언하였지만 꼭 필요한 건 아니다.

이제 얘네를 ResponseEntity에 담아야한다. 사실 이 부분은 없어도 된다.(있어야함) 하지만 이 코드를 만들어두지 않으면 ResponseEntity를 만들어서 보내야할때마다 ResponseEntity객체를 생성해서 그 안에 상태코드와 Body(ResponseCode와 데이터)를 담아 보내야한다. 그래서 미리 ResponseData(가명) 클래스 안에 정적 메소드로 만들어 두는 것이다.

아래 코드는 ReponseCode만 담을때, data도 담을 때, Headers도 담을 때를 구분하여 매개변수로 각각의 데이터를 받아 ReponseEntity에 담아 ReponseEntity를 반환하는 메소드들이다.

```java
//response data가 없을 때
  public static ResponseEntity<ResponseData> toResponseEntity(ResponseCode responseCode) {
    return ResponseEntity
        .status(responseCode.getHttpStatus())
        .body(ResponseData.builder()
            .code(responseCode)
            .build()
        );
  }

  //response data가 있을 때
  public static <T> ResponseEntity<ResponseData<T>> toResponseEntity(ResponseCode responseCode,
      T data) {
    return ResponseEntity
        .status(responseCode.getHttpStatus())
        .body(ResponseData.<T>builder()
            .code(responseCode)
            .data(data)
            .build()
        );
  }

  //response header가 있을 때 ..
  public static <T> ResponseEntity<ResponseData<T>> toResponseEntity(ResponseCode responseCode,
      MultiValueMap<String, String> header, T data) {
    return ResponseEntity
        .status(responseCode.getHttpStatus())
        .header(String.valueOf(header))
        .body(ResponseData.<T>builder()
            .code(responseCode)
            .data(data)
            .build()
        );
  }
```

### 사용하기

예를들어 ResponseCode만 담아 ReponseEntity를 만들고 싶을 때는 아래와 같이 만들어 주면 된다.

실제로 사용할 때는 콘솔에 출력하는게 아니라 클라이언트로 ReponseEntity를 전송해야 할 것이다.

```java
System.out.println(ResponseData.toResponseEntity(ResponseCode.LOGIN_SUCCESS));
```

결과

⬇️

```java
<200 OK OK,ResponseData(timestamp=2021-09-06T20:38:05.985989, code='HttpStatus : 200 OK, detail : 로그인 성공', data=null),[]>
```

이렇게 HTTP상태코드와 우리가 Body에 넣어준 ResponseData객체의 정보가 잘 담긴 것을 볼 수 있다.

한가지 유의할 점은 ResponseData와 ResponseCode 클래스에 모두 @ToString 선언을 해주어야한다. 우리의 경우 ResponseData에는 어노테이션으로 ToString을 선언해 주었지만 ResponseCode에는 toString 메소드를 오버라이딩했다.