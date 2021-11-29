<h1>리액티브 프로그래밍</h1>
리액티브 프로그래밍 패러다임의 중요성이 증가하는 이유? <br/>
- 빅데이터 : 보통 빅데이터는 페타바이트 단위로 구성되며 매일 증가한다.<br/>
- 다양한 환경 : 모바이 디바이스에서 수천 개의 멀티 코어 프로세서로 실행되는 크라우드 기반 클러스터에 이르기까지 다양한 환경에 애플리케이션이 배포된다.<br/>
- 사용 패턴 : 사용자는 1년 내내 항상 서비스를 이용할 수 있으며 밀리초 단위의 응답 시간을 기대한다.<br/>

리액티브 프로그래밍에서는 다양한 시스템과 소스에서 들어오는 데이터 항목 스트림을 비동기적으로 처리하고 합쳐서 이런 문제를 해결한다.<br/>
실제로 이런 패러다임에 맞게 설계된 애플리케이션은 발생한 데이터 항목을 바로 처리함으로 사용자에게 높은 응답성을 제공한다.<br/>

<h2>리액티브 매니패스토</h2>
리액티브 애플리케이션과 시스템 개발의 핵심 원칙을 공식적으로 정의한다.<br/>
- 반응성 : 리액티브 시스템은 빠를 뿐 아니라 더 중요한 특징으로 일정하고 예상할 수 있는 반응 시간을 제공한다. 결과적으로 사용자가 기대치를 가질 수 있다. <br/>
기대치를 통해 사용자의 확신이 증가하면서 사용할 수 있는 애플리케이션이라는 확인을 제공할 수 있다.<br/>
- 회복성 : 장애가 발생해도 시스템은 반응해야 한다. 컴포넌트 실행 복제, 여러 컴포넌트의 시간과 공간 분리, 각 컴포넌트가 비도기적으로 작업을 다른 컴포넌트에 위임하는 등
리액티브 매니페이스는 회복성을 달성할 수 있는 다양한 기법을 제시한다.<br/>
- 탄력성 : 애플리케이션의 생명주기 동안 다양한 작업 부하를 받게 되는데 이 다양한 작업 부하로 애플리케이션의 반응성이 위협받을 수 있다.<br/>
- 메세지 주도 : 회복성과 탄력성을 지원하려면 약한 결합, 고립, 위치 투명성 등을 지원할 수 있도록 시스템을 구헝하는 컴포넌트의 경계를 명확하게 정의해야 한다.<br/>

![17-1](https://user-images.githubusercontent.com/87962572/143775103-2f9ba773-dff4-40bf-849d-100ea104daff.PNG)

<h3>애플리케이션 수준의 리액티브</h3>
애플리케이션 수준 컴포넌트의 리액티브 프로그래밍의 주요 기능은 비동기로 작업을 수행할 수 있다는 점이다.<br/>
이벤트 스트림을 블록하지 않고 비동기로 처리하는 것이 최선 멀티코어 CPU의 사용률을 극대화할 수 있는 방법이다.<br/>
이 목표를 달성할 수 있도록 리액티브 프레임워크와 라이브러리는 스레드를 퓨쳐, 액터, 일련의 콜백을 발생시키는 이벤트 루프 등과 공유하고 처리할 이벤트를 변환하고 관리한다.<br/>

스레드를 다시 쪼개는 종류의 기술을 이용할 때는 메인 이벤트 루프 안에서는 절대 동작을 블럭하지 않아야 한다는 중요한 전재 조건이 항상 따른다.<br/>
데이터베이스나 파일 시스템 접근, 작업 완료까지 얼마나 걸릴 지 예측이 힘든 원격 서비스 호출 등 모든 I/O 관련 동작이 블록 동작에 속한다.<br/>

![17-2](https://user-images.githubusercontent.com/87962572/143775327-8f27af7d-d08b-4140-a1e3-0ed97dc61001.PNG)
예를 들어, 두 스레드를 포함하는 풀이 있고, 이벤트 스트림 세 개를 처리하는 상황을 가정하자.<br/>
한 번에 오직 두 개의 스트림을 처리할 수 있는 상황이므로 가능하면 이들 스트림은 두 스레드를 효율적이고 공정하게 공유해야 한다.<br/>
어떤 스트림의 이벤트를 처리하다보니 파일 시스템 기록 또는 블록되는 API를 이용해 데이터베이스에서 파일을 가져오는 등 느린 I/O 작업이 시작되었다.<br/>
그림에서 보여주듯이 이런 상황에서 스레드 2는 I/O 동작이 끝나기를 기다리며 소모된다.<br/>
스레드 1은 여전히 첫 번째 스트림을 처리할 수 있지만 이전에 시작된 블록 동작이 끝나기 전까지 세 번째 스트림은 처리되지 못한다.<br/>

RxJava, Akka 같은 리액티브 프레임워크는 별도로 지정된 스레드 풀에서 블록 동작을 실행시켜 이 문제를 해결한다.<br/>
메인 풀의 모든 스레드는 방해받지 않고 실행되므로 모든 CPU 코어가 가장 최적의 상황에서 동작할 수 있다.<br/>
CPU관련 작업과 I/O 관련 작업을 분리하면 조금 더 정밀하게 풀의 크기 등을 설정할 수 있고 두 종류의 작업의성능을 관찰할 수 있다.<br/>

<h3>시스템 수준의 리액티브</h3>
리액티브 시스템 특징 :
 1) 여러 애플리케이션이 한 개의 일관적인, 회복할 수 있는 플랫폼을 구성하게 해준다.<br/>
 2) 애플리케이션 중 하나가 실패해도 전체 시스템은 계속 운영될 수 있도록 도와주는 소프트웨어 아키텍처다.<br/>
 3) 애플리케이션을 조립하고 상호소통을 조절한다.<br/>
 4) 주요 속성으로 "메세지 주도"를 꼽을 수 있다.<br/>
 5) 수신자와 발신자가 각각 수신자 메세지, 발신 메세지와 결합하지 않도록 이들 메세지를 비동기로 처리해야 한다.(정의된 목적지 하나를 향햠)<br/>
 6) 각 컴포넌트를 완전히 고립시키려면 이들이 결합되지 않도록 해야하며, 시스템이 장애(회복성)와 높은 부하(탄력성)에서도 반응성을 유지할 수 있다.<br/>
 7) 컴포넌트에서 발생한 장애를 고립시킴으로 문제가 주변의 다른 컴포넌트로 전파되면서 전체 시스템 장애로 이어지는 것을 막음으로 회복성을 제공한다. (회복성 = 결함 허용 능력)<br/>
  - "회복성" : 시스템에 장애가 발생했을 때 서서히 성능이 저하되는 것이 아니라 문제를 격리함으로 장애에서 완전히 복구되어 건강한 상태로 시스템이 돌아온다.<br/>
 
<h3>리액티브 스트림과 플로 API</h3>
리액티브 프로그래밍은 리액티브 스트림을 사용하는 프로그래밍이다. 리액티브 스트림은 잠재적으로 무한의 비동기 데이터를 순서대로 그리고 블록하지 않은 역압력을 전제해 처리하는 표준
기술이다.
예기치 않는 방식으로 이벤트를 잃어버리는 등의 문제가 발생하지 않는다.
부하가 발생한 컴포넌트는 이벤트 발생 속도를 늦추라고 알리거나, 얼마나 많은 이벤트를 수신할 수 있는지 알리거나, 다른 데이터를 받기전에 기존의 데이터를 처리하는 데 얼마나 시간이 걸리는지를
업스트림 발행자에게 알릴 수 있어야 한다.


<h3>Flow 클래스 소개</h3>
java.util.concurrent.Flow는 자바9에서 추가된 리액티브 프로그래밍을 제공하는 클래스이다.<br/>
정적 컴포넌트 하나를 포함하고 있으며 인스턴스화할 수 없다.<br/>
리액티브 스트림 프로젝트의 표준에 따라 프로그매이 발행-구독 모델을 지원할 수 있도록 Flow 클래스는 중첩된 인터페이스 4개를 포함한다.<br/>

- Publisher
- Subscriber
- Subscription
- Processor 

Publisher가 항목을 발행하면 Subscriber가 한 개 씩 또는 한 번에 여러 항목을 소비하는 데 Subscription이 이 과정을 관리할 수 있도록 Flow 클래스는 관련된 인터페이스와 정적 메서드를
제공한다.<br/>
Publisher는 수많은 일련의 이벤트를 제공할 수 있지만 Subscriber의 요구사항에 따라 역압력 기법에 의해 이벤트 제공 속도가 제한된다.<br/>
Publisher는 자바의 함수형 인터페이스( 한 개의 추상 메서드만 정의)로,<br/>
Subscrier는 Publisher가 발행한 이벤트의 리스너로 자신을 등록할 수 있다.<br/>
Subscription은 Publisher와 Subscriber 사이의 제어 흐름, 역압력을 관리한다.<br/>

```
//Flow.Publisher 인터페이스
@FunctionalInterface
public interface Publisher<T>{
  void subscribe(Subscriber<? super T> s);
}

// Subscriber인터페이스는 Publisher가 관련 이벤트를 발행할 때 호출할 수 있도록 콜백 메서드 4개를 정의한다.
public interface Subscriber<T> {
  void onSubscribe(Subscription s);
  void onNext(T t);
  void onError(Throwable t);
  void onComplete();
}
```

이벤트 스트림은 영원히 지속되거나 아니면 onComplete 콜백을 통해 더 이상의 데이터가 없고 종료됨을 알릴 수 있으며 또는 Publisher에 장애가 발생했을 때는 onError를 호출할 수 있다.<br/>
Subscriber가 Publisher에 자신을 등록할 때 Publisher는 처음으로 onSubscribe 메서드를 호출해 Subscription 객체를 전달한다.<br/>
Subscription 인터페이스는 메서드 두 개를 정의한다.<br/>
Subscription은 첫 번째 메서드로 Publisher에게 주어진 개수의 이벤트를 처리할 준비가 되었음을 알릴 수 있다.<br/>
두 번째 메서드로는 Subscription을 취소, 즉 Publisher에게 더 이상 이벤트를 받지 않음을 통지한다.<br/>

```
//Flow.Subscription 인터페이스
public interface Subscription {
  void request(long n);   ▶ 주어진 n개의 이벤트를 처리
  void cancel();          ▶ 더 이상 이벤트를 받지 않음.
}
```

Publisher는 반드시 Subscription의 request 메서드에 정의된 개수 이하의 요소만 Subscriber에 전달해야한다.<br/>
하지만 Publisher는 지정된 개수보다 적은 수의 요소를 onNext로 전달할 수 있으며 동작이 성공적으로 끝났으면 onComplete를 호출하고 문제가 발생하면 <br/>
onError를 호출해 Subscription을 종료할 수 있다.<br/>

Subscriber는 요소를 받아 처리할 수 있음을 Publisher에 알려야 한다. 이런 방식으로 Subscriber는 Publisher에 역압력을 행사할 수 있고 Subscriber가 관리할 수 없이<br/>
너무 많은 요소를 받는 일을 피할 수 있다.<br/>
더욱이 onComplete나 onError 신호를 처리하는 상황에서 Subscriber는 Publisher나 Subscription의 어떤 메서드도 호출할 수 없으며 Subscrption이 취소되었다고 가정해야 한다.<br/>
마지막으로 Subscriber는 Subscription ,request 메서드 호출 없이도 언제든 종료 시그널을 받을 준비가 되어있어야 하며, Subscription.cancel()이 호출된 이후에라도 한 개 이상의 onNext
를 받을 준비가 되어있어야 한다.<br/>

Publishr와 Subscriber는 정확하게 Subscription을 공유해야 하며 각각이 고유한 역할을 수행해야 한다. 그러러면 onSubscribe와 onNext 메서드에서 Subscriber는 request 메서드를 동기적으로
호출할 수 있어야한다. <br/>
표준에서는 Subscription.cancel()메서든 몇 번을 호출해도 한 번 호출한 것과 같은 효과를 가져야 하며, 여러 번 이 메서드를 호출해도 다른 추가 호출에 별 영향이 없도록 
스레드에 안전해야 한다고 명시하자.<br/>

다음은 플로 API에서 정의하는 인터페이스를 구현한 애플리케이션의 평범한 생명주기를 보여준다.<br/>

![17-3](https://user-images.githubusercontent.com/87962572/143776929-929235c7-b0f8-442a-9c3f-3d2af0d017d9.PNG)

리액티브 스트림에서 처리하는 이벤트의 변환 단계를 나타낸다.<br/>
processor가 에러를 수신하면 이로부터 회복하거나 즉시 onError 신호로 모든 Subscriber에 에러를 전파할 수 있다.<br/>
Subscriber가 Subscription을 취소하면 Processor는 자신의 업스트림 Subscription도 취소함으로 취소 신호를 전파해야 한다.<br/>

Subscriber 인터페이스의 모든 메서드 구현이 Publisher를 블록하지 않도록 강제하지만 이들 메서드가 이벤트를 동기적으로 처리해햐 하는지, <br/>
아니면 비동기적으로 처리해야 하는지는 지정하지 않는다.<br/>
이들 인터페이스에 정의된 메서드는 void를 반환하므로 온전히 비동기 방식으로 이들 메서드를 구현할 수 있다.<br/>

<h3>첫 번째 리액티브 애플리케이션 만들기</h3>

Flow 클래스에 정의된 인터페이스 대부분은 직접 구현하도록 의도된 것이 아니다<br/>
자바 9 java.util.concurrency.Flow 명세는 이들 라이브러리가 준수해야 할 규칙과 다양한 리액티브 라이브러리를 이용해 개발된 리액티브 애플리케이션이 서로 협동하고
소통할 수 있는 공용어를 제시한다.<br/>

예를들어, 온도를 전달하는 간단한 예제로 배운 4개의 인터페이스를 활용해보자.<br/>

```
import java.util.Random;
public class TempInfo {

 public static final Random random = new Random();
 private final String town;
 private final int temp;
 
 public TempInfo(String town, int temp) {
     this.town = town;
     this.temp = temp;
 }
 
 public static TempInfo fetch(String town) { ◀ 정적 팩토리 메서드를 이용해 해당 도시의 TempInfo 인스턴스를 만든다.
     if (random.nextInt(10) == 0)            ◀ 10분의 1 확률로 온도를 가져오기 작업이 실패한다.
        throw new RuntimeException("Error!"); 
     
     return new TempInfo(town, random.nextInt(100)); ◀ 0에서 99 사이에서 임의의 화씨 온도를 반영한다.
 }
 
 @Override
 public String toString() {
    return town + " : " + temp;
 }
 
 public int getTemp() {
    return temp;
 }
 
 public String getTown() {
    return town;
 }
 
}
```

간단한 도메인 모델을 정의한 다음에는 다음 예제에서 보여주는 것처럼 Subscriber가 요청할 때마다 해당 도시의 온도를 전소하도록 Subsciption을 구현한다.

```
import java.util.concurrent.Flow.*;
public class TempSubscription implements Subscription {
    private final Subscriber<? super TempInfo> subscriber;
    private final String town;
    public TempSubscription( Subscriber<? super TempInfo> subscriber, String town ) {
     this.subscriber = subscriber;
     this.town = town;
    }
    @Override
    public void request( long n ) {   
     for (long i = 0L; i < n; i++) {  ◀ Subscriber가 만든 요청을 한 개씩 반복
         try {
            subscriber.onNext( TempInfo.fetch( town ) );  ◀ 현재 온도를 Subsciber로 전달
         } catch (Exception e) {
            subscriber.onError( e );  ◀ 온도 가져오기를 실패하면 Subscriber로 에러를 전달
            break;
         }
        }
    }
    
    @Override
    public void cancel() {
        subscriber.onComplete();  ◀ 구독을 취소하면 완료 onComplete 신호를 Subscriber로 전달
    }
}
```

새 요소를 얻을 때마다 Subscription이 전달한 온도를 출력하고 새 레포트를 요청하는 Subscriber 클래스를 다음처럼 구현한다.

```
import java.util.concurrent.Flow.*;
public class TempSubscriber implements Subscriber<TempInfo> {
    private Subscription subscription;
    
    @Override
    public void onSubscribe( Subscription subscription ) {  ◀ 구독을 저장하고 첫 번째 요청을 전달
        this.subscription = subscription; 
        subscription.request( 1 );
    }
    
    @Override
    public void onNext( TempInfo tempInfo ) {  ◀ 수신한 온도를 출력하고 다음 정보를 요청
        System.out.println( tempInfo ); 
        subscription.request( 1 );
    }
    
    @Override
    public void onError( Throwable t ) {  ◀ 에러가 발생하면 에러 메세지 출력
        System.err.println(t.getMessage());
    }
    
    @Override
    public void onComplete() { 
        System.out.println("Done!");
    }
}

// 실제 동작할 수 있도록 Publisher를 만들고 TempSubscrier를 이용해 Publisher에 구독하도록 Main클래스를 구현한 코드이다.
public class Main {
 public static void main( String[] args ) {
  getTemperatures( "New York" ).subscribe( new TempSubscriber() );   ◀ 뉴욕에 새 Publisher를 만들고 TempSubscriber를 구독시킴
 }
 private static Publisher<TempInfo> getTemperatures( String town ) { ◀ 구독한 Subscriber에게 TempSubcription을 전송하는 Publisher를 반환
  return subscriber -> subscriber.onSubscribe( 
  new TempSubscription( subscriber, town ) );
 }
}
```

여기에서 스택 오버플로우가 발생한다면 어떻게 해결할 수 있을까? Executor를 TempSubscription으로 추가한 다음 다른 스레드에서 TempSubscriber로 세 요소를 전달하는 방법이 있다.

```
public class TempSubscription implements Subscription { 
 private static final ExecutorService executor = 
 Executors.newSingleThreadExecutor();
 
 @Override
 public void request( long n ) {
  executor.submit( () -> {  ◀ 다른 스레드에서 다음 요소를 구독자에게 보낸다.
   for (long i = 0L; i < n; i++) {
    try {
     subscriber.onNext( TempInfo.fetch( town ) );
    } catch (Exception e) {
     subscriber.onError( e );
     break;
    }
  }
 });
 }
}
```

<h3>Processor로 데이터 변환하기</h3>
Processor는 Subscriber이며 동시에 Publisher이다. 사실 Processor의 목적은 Publisher를 구독한 다음 수신한 데이터를 가공해 다시 제공하는 것이다.<br/>
화씨로 제공된 데이터를 섭씨로 변환해 다시 방출하는 다음의 예제를 통해 Processor를 구현해보자.<br/>

```
public class TempProcessor implements Processor<TempInfo, TempInfo> {  // TempInfo를 다른 TempInfo로 변환하는 프로세서
 private Subscriber<? super TempInfo> subscriber;
 
 @Override
 public void subscribe( Subscriber<? super TempInfo> subscriber ) {
  this.subscriber = subscriber;
 }
 
 @Override
 public void onNext( TempInfo temp ) {
  subscriber.onNext( new TempInfo( temp.getTown(), (temp.getTemp() - 32) * 5 / 9) );  // 섭씨로 변환한 다음 TempInfo를 다시 전송
 }
 
 @Override
 public void onSubscribe( Subscription subscription ) { // 다른 모든 신호는 업스트림 구독자에 전달
  subscriber.onSubscribe( subscription ); 
 }
 
 @Override
 public void onError( Throwable throwable ) {           // 다른 모든 신호는 업스트림 구독자에 전달
  subscriber.onError( throwable ); 
 }
 
 @Override
 public void onComplete() {      // 다른 모든 신호는 업스트림 구독자에 전달
  subscriber.onComplete(); 
 }
}
```
TempProcessor에서 로직을 포함하는 유일한 ㅔ서드인 onNext는 화씨를 섭씨로 변혼하는 다음 온도를 재전송한다.<br/>
Subscriber로 전달하며 Publisher의 subscribe메서드는 업스트림 Subscriber를 Processor로 등록하는 동작을 수행한다.<br/>

```
import java.util.concurrent.Flow.*;
public class Main {
 public static void main( String[] args ) {
  getCelsiusTemperatures( "New York" ).subscribe( new TempSubscriber() ); 
 }
 
 public static Publisher<TempInfo> getCelsiusTemperatures(String town) {
  return subscriber -> {
   TempProcessor processor = new TempProcessor();    ▶TempProcessor를 만들고 subscriber와 반환된 Publisher사이로 연결
   processor.subscribe( subscriber );
   processor.onSubscribe( new TempSubscription(processor, town) );
  };
 }
}
```

<h3>Observable 만들고 사용하기</h3>
Observable, Flowable 클래스는 다양한 종류의 리액티브 스트림을 편리하게 만들 수 있도록 여러 팩토리 메서드를 제공한다.

미리 정의한 몇 개의 요소를 이용해 간단한 Observable을 만들 수 있다.

```
// just 는 팩토리 메서드는 한 개 이상의 요소를 이용해 이를 방출하는 Observable로 변환한다.
// Observable의 구독자는 onNext("first"), onNext("second"), onComplete()의 순서로 메세지를 받는다.
Observable<String> strings = Obervale.just("first","second");

// 사용자와 실시간으로 상호작용하면서 지정된 속도로 이벤트를 방출하는 상황에서 유용하게 시용할 수 있는 다른 Oberservable  팩토리 메서드도 있다.
Observable<Long> onePerSec = Observable.interval(1, TimeUnit.SECONDS);
//interval 은 onePerSec라는 변수로 Oberservale을 반환해 할당한다.
// 이 Oberservale은 0에서 시작해 1초 간겨그로 long형식의 값을 무한으로 증가시키며 값을 방출한다.
```

최종 목표를 달성하기 전에 중간 과정으로 이들 온도를 매 초마다 출력할 수 있다.<br/>
그러려면 onePerSec에 가입해서 매 초마다 온도를 받고 이를 이용해 관심이 있는 도시의 온도를 출력해야 한다.<br/>
Obervable은 역압력을 지원하지 않으므로 Subscription의 request 메서드를 포함하지 않는다.<br/>

```
pubilc interface Observer<T>{
 void onSubscriber(Disposalble d);
 void onNext(T t);   // onNext 메서드의 시그니처에 해당하는 람다 표현식을 전달해 Observalble을 구독할 수 있다.
 void onError(Throwable t);  
 void onComplete();
}

// Observable onPerSec에 가입하고 뉴욕에서 매 초마다 발생하는 온도를 출력하는 기능을 코드 한 줄로 구현할 수 있다.
onePerSec.subscribe(i -> System.out.println(TempInf.fetch("New York")));

// sleep 메서드를 추가해 프로그램이 종료되는 걸 막는 방법도 있다. 현재 스레드에서 콜백을 호출하는 blokingSubscribe메서드를 사용하면 더 깔끔하게 문제를 해결할 수 있다.
blokingSubscribe 가 적합하다.
onePerSec.blockingSubscribe(
 i -> System.out.println(TempInfo.fetch("New York"))
);
```

출력결과는 다음과 같이 나타난다.
```
New York : 87
New York : 18
New York : 75
java.lang.RuntimeException: Error!
at flow.common.TempInfo.fetch(TempInfo.java:18)
at flow.Main.lambda$main$0(Main.java:12)
at io.reactivex.internal.observers.LambdaObserver
 .onNext(LambdaObserver.java:59)
at io.reactivex.internal.operators.observable
 .ObservableInterval$IntervalObserver.run(ObservableInterval.java:74
```

이는 설계상 온도를 가져오는 기능이 임의로 실패하기 때문이다. <br/>
이 기능을 일반화하고 처리되지 않은 예외를 추가해보자.<br/>

```
public static Obervable<TempInfo> getTemerature(String town) {
 return Observable.create(emitter -> 
  Observable.interval(1, TimeUnit.SECONDS)  ◀ 매 초마다 무한으로 증가하는 일련의 long 값을 방출하는 Observable
   .subscribe(i -> {
   if (!emitter.isDisposed()) {    ◀ 소비된 옵저버가 아직 폐기 되지않았으면 어떤 작업을 수행
     if ( i >= 5 ) { 
     emitter.onComplete();
     } else {
     try {
     emitter.onNext(TempInfo.fetch(town));  ◀ 아니면 온도를 Observer로 보고
     } catch (Exception e) {
     emitter.onError(e);  ◀ 에러가 발생하면 Observer에 알림
     }
   }
  }}))
}
```

필요한 이벤트를 전송하는 ObservableEmitter를 소비하는 함수로 Observable을 만들어 반환했다.<br/>
RxJava의 ObservableEmitter 인터페이스는 RxJava의 기본 Emitter를 상속한다.<br/>

```
public interface Emitter<T> {
  void onNext(T t);
  void onError(Throwable t);
  void onCompleter();
}
```

Emitter는 새 Disposable 을 설정하는 메서드와 시퀀스가 이미 다운스트림을 폐기했는지 확인하는 메서드 등을 제공한다.<br/>

아래의 코드에서 보여주는 것처럼 getTemperatur 메서드가 반환하는 Observable에 가입시킬 Observer를 쉽게 완성해서 전달된 온도를 출력할 수 있다.<br/>

```
public class TempObserver implements Observer<TempInfo> {
 @Override
 public void onComplete() {
  System.out.println( "Done!" );
 }
 
 @Override
 public void onError( Throwable throwable ) {
  System.out.println( "Got problem: " + throwable.getMessage() );
 }
 
 @Override
 public void onSubscribe( Disposable disposable ) {
 }
 
 @Override
 public void onNext( TempInfo tempInfo ) {
  System.out.println( tempInfo );
 }
}

public class Main {
 public static void main(String[] args) {
 Observable<TempInfo> observable = getTemperature( "New York" );   // 매 초마다 뉴욕의 온도 보고를 방출하는 Observable만들기
 observable.blockingSubscribe( new TempObserver() );    // 단순 Observer로 이 Observable에 가입해서 온도 출력하기
 }
}

```

<h3>Observable 을 변환하고 합치기</h3>
- 수평선 : 표시된 리액티브 스트림에 임의의 순서로 구성된 요소가 기하학적 모형이 나타난다.<br/>
- 특수 기호 : 에러나 완료 신호를 나타낸다<br/>
- 박스 : 해당 연산이 요소를 어떻게 변환하거나 여러 스트림을 어떻게 합치는 지 보여준다.<br/>

![17-4](https://user-images.githubusercontent.com/87962572/143868882-e7e35bb9-b00e-497b-9f67-b70866f62b1c.PNG)
1) Observable의 시간선이다. 시간은 왼쪽에서 오른쪽으로 흐른다.<br/>
2) Observable이 방출하는 항목이다<br/>
3) 세로 선은 Observable이 성공적으로 완료되었음을 가리킨다.<br/>
4) 점섬과 상자는 Observable에 적용되는 변환을 가리킨다. 상자 안의 텍스트는 변환의 종류를 가리킨다.<br/>
5) 변환 결과 Observable이다.<br/>
6) 어떤 이유로 Observable이 비정상 종료되어 에러가 발생하면 수직선이 X로 바뀐다.<br/>

아래 그림은 MAP(Observable이 발행하는 요소를 반환)과 merge(두 개 이상의 Observable이 방출한 이벤트를 하나로 합침)의 예다.

![17-5](https://user-images.githubusercontent.com/87962572/143869359-3fe14fbf-bece-4ee5-bd56-5882dcbf543d.PNG)

map, merge를 이용해 어떻게 기존에 구현한 RxJava 예제를 개선하고 기능을 추가할 수 있을지 궁금할 것이다.<br/>
map을 이용하면 다음 예제에서 볼 수 있는 것처럼 플로 api processor를 이용하는 것보다 조금 더 깔끔하게 화씨를 섭씨로 바꿀 수 있다.<br/>

```
// getTemperature 메서드가 반환하는 Observable을 받아 화씨를 섭씨로 바꾼 다음 매 초 한 개씩 온도를 다시 방출하는 또 다른 Observable을 반환한다.
public static Observable<TempInfo> getCelsiusTemperature(String town) {
  return getTemperature( town ).map( temp -> new TempInfo( temp.getTown(), (temp.getTemp() - 32) * 5 / 9) );
}

// 한 개 이상 도시으 온도 보고를 합친다.
public static Observable<TempInfo> getCelsiusTemperatures(String... towns) {
 return Observable.merge(Arrays.stream(towns).map(TempObservable::getCelsiusTemperature).collect(toList()));
}
```

온도를 얻으려는 도시 집합을 포함하는 가변 인수를 받는다.<br/>
이 가변 인수를 문자열 스트림으로 변환한 다음 각 문자열의 getcelsiusTemperature 메서드로 전달한다.<br/>
이런식으로 각 도시는 매 초마다 도시의 옫노를 방출하는 Obserable로 변신한다.<br/>
마지막으로 Observabledml 스트림은 리스트로 모아지며 다시 리스트는 Observable 가 제공하는 정적 팩토리메서드 merge로 전달된다.<br/>
이 메서드는 Observable의 Iterable을 인수로 받아 마치 한 개의 Observable처럼 동작하도록 결과를 합친다. <br/>
