<h1>webflux의 고찰</h1>

<h2>1. webFlux란 ?</h2>
 - Spring Framework 5에 추가된 모듈 <br/>
 - client, server에서 reactive 스타일의 어플리케이션 개발을 도와주는 모듈<br/>
 
<h2>2. webFlux의 탄생 이유?</h2>
 - 적은 양의 스레드와 최소한의 하드웨어 자원으로 동시성 핸들링하기 위함.<br/>
 - netty를 사용하여 non-blocking 서버를 사용하면딤.<br/>
 - java8의 함수형 API 자체가 논블로킹 어플리케이션 API의 토대가 되었음.<br/>
 - Reactive를 대응하기 위해!<br/>
   -> Reactive라는 것은 I/O 이벤트에 따라 네트웨크 컴포넌트를 반응하고 이벤트나 다른 것들에 UI 컨트롤러가 반응하게 하는 것.<br/>

<h2>3. webFlux를 사용해야될 때</h2>
 - 비동기- 논블로킹 리액티브 개발<br/>
 - 효율적으로 동작하는 고성능 웹어플리케이션 개발<br/>
 - 서비스간 호출이 많은 msa 아키텍처에 적합<br/>
 
<h2>4. webFlux의 단점</h2>
 - 오류처리가 다소 복잡<br/>
 
<h2>5. Mono vs Flux</h2>
 - webflux에서 자주 언급되는 객체는 Flux와 Mono이다.<br/>
 
 1) Mono : 0~1개의 데이터 전달할때 사용<br/>
     - 따라서 보통 여러 스트림을 하나의 결과를 모아줄 때 사용<br/>
 2) Flux : 0~n개의 데이터 전달할때 사용<br/>
     - Mono를 합쳐서 하나의 여러 개의 값을 여러개의 값을 처리할 때 사용<br/>
     
 Q1)그러면 굳이.. Flux도 0~1개의 데이터 전달이 가능한데, 굳이 한개까지만 데이터를 처리할 수 있는 Mono라는 타입이 필요한가?<br/>
 A1) 데이터 설계를 할때 결과가 없거나 하나의 결과값만 받는 것이 명백한 경우,  List나 배열을 사용하지 않는 것처럼, <br/>
 Multi Result가 아닌 하나의 결과셋만 받게 될 경우에는 불필요하게 Flux를 사용하지 않고 Mono를 사용하게 된다.<br/>
 
<h2>6. Spring MVC vs Spring WebFlux</h2>

<img width="419" src="https://user-images.githubusercontent.com/87962572/171545028-1974c276-31f9-4882-9d2c-5a109469a2b4.png">
1) 공통점 : Spring MVC와 WebFlux의 공통점은 @Controller, Reactive 클라이언트입니다. 
  둘 다 Tomcat, Jetty, Undertow와 같은 서버에서 실행할 수 있습니다. 
2) 차이점 :
  - Spring MVC에서는 명령형 논리, JDBC, JPA를 가질 수 있습니다. 
  - Spring WebFlux에서는 기능적 엔드 포인트, 이벤트 루프, 동시성 모델을 가질 수 있습니다. 
  - Spring WebFlux는 Netty 서버에서 실행할 수 있다는 장점이 있습니다.

<h3>6-1.Spring MVC 또는 WebFlux를 사용하는 경우</h2>

```

1. 이미 작동중인 Spring MVC 애플리케이션이 있다면 Spring WebFlux로 변환 할 필요가 없으며,  Spring MVC는 작성하고 디버그하는 쉬운 방법 인 명령형 프로그래밍을 사용합니다.
 
2. non-blocking 웹 스택을 생성하고자한다면, 선택 가능한 서버 (Netty, Tomcat, Jetty, Undertow 및 Servlet 3.1+ 컨테이너)를 제공하는 Spring WebFlux를 선택할 수 있습니다. 
웹 엔드 포인트 및 반응 라이브러리 (Reactor, RxJava 또는 기타)를 선택할 수 있습니다.
 
3. Spring WebFlux는 Java 8 lambda 또는 Kotlin과 함께 사용하는 가볍고 기능적인 웹 프레임 워크에 유용합니다.
 
4. 마이크로 서비스 애플리케이션에서 우리는 Spring MVC와 Spring WebFlux 컨트롤러의 혼합 애플리케이션을 가질 수 있습니다. 
Spring WebFlux 엔드 포인트도 가질 수 있습니다.
 
5. 애플리케이션이 JPA, JDBC 또는 네트워킹 API에 의존하는 경우 Spring MVC가 최선의 선택이다.
 
(Webflux 는 Asynchronous Non-blocking I/O 을 방식을 활용하여 성능을 끌어 올릴 수 있는 장점이 있다. 
즉, Non Blocking 기반으로 코드를 작성해야 한다. 
만약 Blocking 코드가 있다면 Webflux를 사용하는 의미가 떨어지게 된다. 
얼마 전까지는 Java 진영에 Non Blocking 을 지원하는 DB Driver가 없었지만, 최근에 R2DBC 가 릴리즈되어 이제는 Java 에서도 Non Blocking 으로 DB 를 접근할 수 있게 되었다.)

```
 
 참고url :
 https://heeyeah.github.io/spring/2020-02-29-web-flux/
 https://devuna.tistory.com/108




---

<h1>NHN WEBFLUX동영상 시청후 정리(20220608) </h1>

1. WebFlux를 선택한 이유?
2. WebFlux가 느렸던 이유 - 성능 측정 결과/ 성능 개선 사항

Spring MVC  
- 보통 할당한 처리를 모두 처리할때까지 기다림<BR/>
- Thread pool은 보통 200개<BR/>

- Thread가 모든 Task를 처리하고 하나의 Thread가 처리하는 동안에 다른 Thread는 기다리고 있는다.<BR/>
( A Thread가 처리하고 있으면 B Thread는 waiting 을 하고 있고,<BR/>
 A Thread가 완료처리되면 B Thread가 runnable 상태를 왔다갔다함)<BR/>
- cpu가 한정된 곳에서 Thread 200개들이 저마다 서로 cpu를 점유하기 위해 경합함<BR/>


WebFlux는 = Event 단위로 처리되는 단위임 = Event Pool<BR/>
- Netty기반으로 동작하며,<BR/>
- Netty는 THread 개수는 machine의 core개수의 X2배를 지니고 있음<BR/>

- I/O가 시작되는 것도 Event, I/O가 종료되어 처리되는 것도 Event로 취급 <BR/>
- 따라서 모든 Event 단위가 Queue에 들어가므로<BR/>
- Thread 상태가 blocking되지 않는다.<BR/>
- Context Switing overhead도 줄어들음<BR/>
- 또한 Thread 상태가 Event 단위로 보다 작게 처리되어 cpu점유하기위한 경합도 줄어들음<BR/>

- 높은 처리량 : Event단위로 처리되어 cpu사용량을 줄어들음/ 그래서 전반적으로 성능이 높아짐<BR/>

2. 나의 WebFlux가 느린 이유?<BR/>

2-1.log() <- 해당 log는 blocking 메서드이므로 webFlux에서 blocking이 포함되면 결국 spring MVC보다 못한 결과가 된다.<br/>
2-2.map()과 flapMap()의 차이점<br/>
- map : 동기식 함수를 사용 -> map을 계속 사용하게 되면 성능이 느려진다.<br/>
- flapMap() : 비동기 형식<br/>

3. BlockHound : Blocking 메서드를 찾아주는 라이브러리
4. Lettuce 설정 : command설정마다 ping command를 synchronous로 실행(성능 저하됨)

5. Avoiding Reactor Meltdown : 상황에 따라서 Blocking API를 사용해야될때 Blocking 소스를 위한 별도의 pool를 만들어줌

