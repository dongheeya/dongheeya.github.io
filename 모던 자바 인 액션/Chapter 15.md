<h1>Chapter 15. CompletableFuture와 리액티브 프로그래밍 컨셉의 기초</h1>
최근 소프트웨어 개발 방법을 획기적으로 뒤집는 2가지 추세가 있다.<br/>
1) 애플리케이션을 실행하는 하드웨어에 관한것<br/>
 - 멀티코어 프로세서가 발전하면서 애플리케이션의 속도는 멀티 코어 프로세서를 얼마나 잘 활용할 수 있도록 소프트웨어를 개발하는 가에 따라 달라질 수 있음. <br/>
 - 한 개의 큰 태스트를 병렬로 실행할 수 있는 개별 하위 태스크로 분리할 수 있다.<br/>

2) 애플리케이션을 어떻게 구성하는가?<br/>
 - 인터넷 서비스에서 사용하는 애플리케이션이 증가하고 있는 현상을 반영.<br/>
 - 하나의 거대한 애플리케이션 대신 작은 서비스로 애플리케이션을 나누는 것이다.<br/>
 - 서비스가 작아진 대신 네트워크 통신이 증가한다.<br/>

앞으로 만들 웹 애플리케이션은 다양한 소스의 콘텐츠를 가져와서 사용자가 삶을 풍요롭게 만들도록 합치는 메시업 형태가 될 가능성이 크다.<br/>

![15-1](https://user-images.githubusercontent.com/87962572/141975530-92a9e2f1-c0b1-4afb-a0f1-519295b25543.PNG)

ex) 여러 언어로 특정 주제의 댓글 추세를 페이스북이나, 트위터 api로 찾은 다음 내부 알고리즘과 가장 일치하는 주제로 등급을 매겨서 주어진 프랑스 사용자들에<br/>
제공할 소셜미디어 정서를 수집하고 요약하는 웹사이트를 만들 수 있다.<br/>
구글 번역을 이용해 커멘트를 프랑스어로 번역하거나 구글 맵스를 이용해 작성자의 위치 정보를 얻고, 정보를 모두 합쳐서 웹사이트에 표시할 수 있다.<br/>
이런 애플리케이션을 구현하려면 인터넷으로 여러 웹 서비스를 접근해야하고, 서비스의 응답을 기다리는 동안 연산이 블록되거나 귀중한 cpu 클록 사이클 자원을 낭비하고 싶지는 않다.<br/>

반면 벙렬성이 아니라 동시성을 필요로 하는 상황 즉 조금씩 연관된 작업을 같은 CPU에서 동작하는 것 또는 애플리케이션을 생산성을 극대화할 수 있도록
코어를 바쁘게 유지하는 것이 목표라면, <br/>
원격 서비스나 데이터베이스 결과를 기다리는 스레드를 블록함으로 연산 자원을 낭비하는 일은 피해야 한다.<br/>

동시성은 단일 코어 머신에서 발생할 수 있는 프로그래밍 속성으로 실행이 서로 겹칠 수 있는 반면,<br/>
병렬성은 병렬 실행을 하드웨어 수준에서 지원한다.<br/>

![15-2](https://user-images.githubusercontent.com/87962572/141976200-057f816c-c2a6-4157-9b4a-e93b1dc1e341.PNG)

<h2>동시성을 구현하는 자바 지원의 진화</h2>

멀티 코어 CPU에서 효과적으로 프로그래밍을 실행할 필요성이 커지면서 이후 자바 버전에서는 개선된 동시성 지원이 추가되었다.<br/>
자바 8에서는 스트림과 새로 추가된 람다 지원에 기반한 병렬 프로세싱이 추가되었다.<br/>
자바는 Future를 조합하는 기능을 추가하면서 동시성을 강화했고, 자바9에서는 분산 비동기 프로그래밍을 명시적으로 지원한다.<br/>

<h3>스레드와 높은 수준의 추상화</h3><br/>
멀티코더 설정에서는 스레드의 도움없이 프로그램이 노트북의 컴퓨팅 파워를 모두 활용할 수 없다.<br/>
프로그램이 스레드를 사용하지 않는다면 효율성을 고려해 여러 프로세서 코어 중 한개만 사용할 것이다.<br/>

ex) 네개의 코어를 가진 CPU에서 이론적으로 프로그램을 네 개의 코어에서 병렬로 실행함으로 실행 속도를 네 배까지 향상시킬 수 있다.<br/>
학생이 제출한 숫자 1,000,000 개를 저정한 배열을 처리하는 다음 예제를 살펴보자.<br/>

```
long sum= 0;
for ( int i =0; i< 1_000_000; i++){
  sum += starts[i];
}
```

위의 코드는 한 개의 코어로 며칠 동안 작업을 수행한다.<br/>
반면, 아래 코드는 첫 스레드를 다음 처럼 실행한다.<br/>

```
long sum0 = 0;
for( int i= 0; i< 250_000; i++){
  sum0 += stats[i];
}
```

그리고 네 번째 스레드는 다음으로 끝낸다.

```
long sum3 = 0;
for( int i= 750_000; i< 250_000; i++){
  sum0 += stats[i];
}
```

메인 프로그램은 네 개의 스레드를 완성하고 자바의 .start()로 실행한 다음 .join()으로 완료될 때까지 기다렸다가 다음을 계산한다.<br/>

sum = sum0 + ... + sum3;

자바 스트림으로 내부 반복을 통해서 쉽게 병렬처리를 할 수 있다.

```
sum = Arrays.stream(stats).parallel().sum();
```

결론적으로 병렬 스트림 반복은 명시적으로 스레드를 사용할 것에 비해 높은 수준의 개념이라는 사실을 알 수 있다.<br/>
다시말해, 스트림을 이용해 스트림 사용 패턴을 추상화할 수 있다.<br/>

쓸모 없는 코드가 라이브러리 내부로 구현되면서 복잡성도 줄어든다는 장점이 더해진다.<br/>

<h3>Executor와 스레드 폴</h3>
<h4>스레드의 문제</h4>
자바 스레드는 직접 운영체제 스레드에 접근한다.<br/>
운영 체제 스레드를 마들고 종료하려면 비싼 비용을 치러야 하며, 더욱이 운영 체제 스레드의 숫자는 제한되어 잇는 것이 문제다.<br/>
자바 애플리케이션이 예상치 못한 방식으로 크래시될 수 있으므로 기존 스레드가 실행되는 상태에서 계속 새로운 스레드를 만드는 상황이 일어나지 않도록 주의해야 한다.<br/>

<h4>풀 그리고 스레드 풀이 더 좋은 이유</h4>
자바 ExecutorService는 테스크를 제출하고 나중에 결과를 수집할 수 있는 인터페이스를 제공한다.<br/>
프로그램은 newFiexedThreadPool 같은 팩토리 메서드 중 하나를 이용해 스레드 플을 만들어 사용할 수 있다.<br/>

ex) ExecutorService newFixedThreadPool(int nThreads)

이 메서드는 워커 스레드라 불리는 nThread를 포함하는 ExecutorService를 만들고 이들을 스레드 풀에 저장한다.<br/>
스레드 풀에서 사용하지 않은 스레드로 제출된 태스크를 먼저 온 순서대로 실행한다.<br/>
큐의 크기조정, 거부정책, 태스트 종류에 따른 우선순위 등 다양한 설정을 할 수 있다.<br/>

프로그래머는 Runnable이나 Callable을 통한 태스크를 제공하면 Thread가 이를 실행한다.<br/>

<h3>스레드 풀 그리고 스레드 풀이 나쁜 이유</h3>
- k 스레드를 가진 스레드 풀은 오직 k만큼의 스레드를 동시에 실행할 수 있다. 이전에 태스크 중 하나가 종료되기 전까지는 스레드에 할당하지 않는다.<br/>
불필요하게 많은 스레드를 만드는 일을 피할 수 있으므로 보통 이 상황은 아무 문제가 되지 않지만 잠을 자거나
I/O를 기다리거나 네트워크 연결을 기다리는 태스크가 있다면 주의해야한다.<br/>

![15-3](https://user-images.githubusercontent.com/87962572/141979472-a8d907ea-7769-4079-8fc1-0531ed9ecb7d.PNG)

네 개의 하드웨어 스레드와 5개의 스레드를 갖는 스레드 풀에 20개의 태스크를 제출했다고 가정하자.<br/>
처음 제출한 세 스레드가 잠을 자거나 I/O를 기다린다고 가정하자.<br/>
나머지 15개의 태스크를 두 스레드가 실행해야하므로 작업 효율성이 예상보다 절반으로 떨어진다.<br/>
그리고 처음 제출한 태스크가 기존 실행 중인 태스크가 나중에 테스크 제출을 기다리는 상황이라면 데드락에 걸릴 수 있다.<br/>

중요한 코드를 실행하는 스레드가 죽는 일이 발생하지 않도록 보통 자바 프로그램은 main이 반환하기 전에 모든 스레드의 작업이 끝나길 기다린다.<br/>
따라서 프로그램을 종료하기 전에 모든 스레드 풀을 종료하는 스관을 갖는 것이 중요하다.<br/>

<h3>스레드의 다른 추상화 : 중첩되지 않은 메서드 호출</h3>
태스크나 스레드가 메서드 호출 안에서 시작되면 그 메서드 호출은 반환하지 않고 작업이 끝나기를 기다렸다.<br/>
스레드 생성과 join이 한쌍처럼 중첩된 메서드 호출 내에 추가되었다.<br/>
<b>이를 엄격한 포크/조인이라고 한다.</b>

![15-4](https://user-images.githubusercontent.com/87962572/141980159-9aa7e043-5ce7-4eb9-bc67-1dd60b907086.PNG)

시작된 태스크를 내부 호출이 아니라 외부 호출에서 종료하도록 기다리는 좀 더 여유로은 방식의 포크/조인을 사용해도 비교적 안전하다.<br/>

사용자의 호출에 의해 스레드가 생성되고 메서드를 벗어나 게속 실행되는 동시성 형태에 초점을 둔다.<br/>

![15-4-1](https://user-images.githubusercontent.com/87962572/141980648-d2736d62-54fe-410f-8173-e120e0fab3ff.PNG)

특히, 메서드 호출자에 기능을 제공하도록 메서드가 반환된 후에도 만들어진 태스크 실행이 계속되는 메서드를 비동기 메서드라 한다.<br/>

<h3>스레드에 무엇을 바라는가?</h3>
일반적으로 모든 하드웨어 스레드를 활용해 병렬성의 장점을 극대화하도록 프로그램 구조를 만드는 것, 즉 프로그램을 작은 태스크 단위로 구조화하는 것을 목표로 한다.<br/>

<h2>동기 API와 비동기 API</h2>
두 가지 단계로 병렬성을 이용할 수 있다. <br/>
첫 번째로 외부 반복을 내부 반복으로 바꿔야한다.<br/>
그리도 스트림에 parallel()메서드를 이용하므로 자바 런타임 라이브러리가 복잡한 스레드 작업을 하지 않고 병렬로 요소가 처리되도록 할 수 있다.<br/>

루프 기반의 게산을 제외한 다른 상황에서도 병렬성이 유용할 수 있다.<br/>

ex) 다음과 같이 시그니처를 갖는 f,g 두 메서드의 호출을 합하는 예제를 살펴보자.

```
int f(int x);
int g(int x);
```

참고로 이들 메서드는 물리적 결과를 반환하므로 동기 API라고 부른다.<br/>
두 메서드를 호출하고 합계를 출력하는 코드가 있다.<br/>

```
int y = f(x);
int z = g(x);
System.out.println( y+z);
```

f와 g를 실행하는 데 오랜 시간이 걸린다고 가정했을 때, f,g의 작업을 컴파일러가 안전하게 이해하기 어려우므로 보통 자바 컴파일러는 코드<br/>
최적화와 관련된 아무 작업도 수행하지 않을 수 있다.<br/>
f와 g가 서로 상호작용하지 않는다는 사실을 알고 있거나 상호작용을 전혀 신경쓰지 않는다면 f와 g를 별도로 CPU 코어로 실행함으로 f와 g를 실행해 이를 구현할 수 있다.<br/>

```
class ThreadExample { 
  public static void main(String[] args) throws InterruptedException {
    int x = 1337;
    Result result = new Result();
    
    Thread t1 = new Thread(() -> {result.left = f(x); });
    Thread t2 = new Thread(() -> {result.right = f(x); });
    
    t1.start();
    t2.start();
    t1.join();
    t2.join();
    System.out.println(result.left + result.right);
  }
  
  private static class Result {
    private int left;
    private int right;
  }
}
```

Runnable  대신 Future API 인터페이스를 이용해 코드를 더 단순화할 수 있다.<br/>
이미 ExecutorService로 스레드 풀을 설정했다고 가정하면 다음처럼 코드를 구현할 수 있다.<br/>

```
public class ExecutorServiceExample{
  public static void main(String[] args0) throws ExecutionException, InterruptedException {
    int x = 1337;
    
    ExecutorService executorService = Executors.newFixedThreadPool(2);
    Future<Integer> y = executorService.submit(() -> f(x)); ▶ 여전히 submit 메서드 호출 같은 불필요한 코드로 오염되었다.
    Future<Integer> z = executorService.submit(() -> g(x)); ▶ 명시적 반복으로 병렬화 수행하던 코드를 스트림을 이용해 내부 반복으로 바꾼 것처럼 
    비슷한 방법으로 이 문제를 해결해야한다.
    
    System.out.println(y.get() + z.get());
    
    excutorService.shutdown();
  }
}
```

문제의 해결은 비동기API라는API를 바꿔서 해결할 수 있다.

<h3>Future 형식 API</h3>

대안을 이용하면 f,g의 시그니처가 다음처럼 바뀐다.
```
Future<Integer> f(int x);
Future<Integer> g(int x);
```

그리고 다음처럼 호출이 바뀐다.

```
Future<Integer> y = f(x);
Future<Integer> z = g(x);
System.out.println(y.get() + z.get());
```

메서드 f는 호출 즉시 자신의 원래 바디를 평가하는 태스크를 포함하는 Future 를 반환한다.<br/>
마찬가지로 메서드 g도 Future를 반환하며 세 번째 코드는 get() 메서드를 이용해 두 Future가 완료되어 결과가 합쳐지기를 기다린다.<br/>

<h3>리액티브 형식 API</h3>
```
void f(int x, IntConsumer dealWithResult);
```

f가 값을 반환하지 않는데 어떻게 프로그램이 동작할까?<br/>
f에 추가 인수로 콜백(람다)을 전달해서 f의 바디에서는 return 문으로 결과른 반환하는 것이 아니라, 결과가 준비되면 이를 람다로 호출하는 태스크를 만드는 것이 비결이다.<br/>
다시 말해 f는 바디를 실행하면서 태스크를 만든 다음 즉시 반환하므로 코드 형식이 다음처럼 바뀐다.<br/>

```
public class CallbackStyleSample{
  public static void main(String[] args){
    int x = 1337;
    Result result = new Result();
    
    f(x, (int y) -> {
      result.left = y;
      System.out.println((result.left + result.right));
    });
    
    g(x, (int z) -> {
      result.right = z;
      System.out.println((result.left + result.right));
    });
  }
}
```

결과가 달라졌다.
f와 g의 호출 합계를 정확하게 출력하지 않고 상황에 따라 먼저 계산된 결과를 출력한다.<br/>
때로는 +에 제공된 두 피연산자가 println이 호출도기 전에 업데이트돌 수 있다.<br/>

- if-then-else를 이용해 적적한 락을 이용해 두 콜백이 모두 호출되었는지 확인한 다음 println을 호출해 원하는 기능을 수행할 수 있다.<br/>
- 리액티브 형식의 API는 보통 한 결과가 아니라 일련의 이벤트에 반응하도록 설계되었으므로 Future를 이용하는 것이 더 적절하다.<br/>

<h3>잠자기(그리고 기타 블로킹 동작)는 해로운 것으로 간주</h3>
사람과 상호작용하거나 어떤 일이 일정 속도로 제한되어 일어나는 상황의 애플리케이션을 만들 때 자연스럽게 sleep()메서드를 사용할 수 있다.<br/>
하지만 스레드는 잠들어도 여전히 시스템 자원을 점유한다. 스레드를 단지 몇개 사용하는 상황에서는 큰 문제가 아니지만, 
스레드가 많아지고 대부분이 잠을 잔다면 문제가 심각해진다.<br/>

```
public class ScheduledExeutorServiceExample {
  public static void main(String[] args){
    ScheduledExecutorService scheduledExecutorService = Executors.newScheduledThreadPool(1);
    
    work1();
    scheduledExecutorService.schedule(ScheduledExecutorServiceExample::work2, 10, TimeUnit.SECONDS); ▶ work1()이 끝난 다음 10초 뒤에 work2()를 개별 task로 스케줄함
      
    scheduledExecutorService.shutdown();
  }
    
  public static void work1(){
    System.out.println("Hello from work1!");
  }

  public static void work2(){
    System.out.println("Hello from work2!");
  }
}
```

위의 코드는 다른 작업이 실행될 수 있도록 허용한다.<br/>
태스크가 실행되면 귀중한 자원을 점유하므로 태스크가 끝나서 자원을 해제하기 전까지 태스크를 계속 실행해야 한다.<br/>
태스크를 블록하는 것보다는 다음 작업을 태스크로 제출하고 현재 태스크는 종료하는 것이 바람직하다.<br/>

<h3>비동기 API에서 예외는 어떻게 처리하는가?</h3>
Future나 리액티브 형식의 비동기 API에서 호출된 메서드의 실제 바디는 별도의 스레드에서 호출되며<br/>
이때 발생하는 어떤 에러는 이미 호출자의 실행 범위와는 관계가 없는 상황이 된다.<br/>

Future를 구현한 CompletableFuture에서는 런타임get()메서드에 예외를 처리할 수 있는 기능을 제공하며, 예외에서 회복할 수 있도록<br/>
exceptionally() 같은 메서드로 제공한다.<br/>

리액티브 형식의 비동기 API에스는 return 대신 기존 콜배기 호출되므로 예외가 발생했을 때 실행될 추가 콜백을 만들어 인터페이스를 바꿔야 한다.<br/>

```
void f(int x, Comsumer<Integer> dealWithResult, Consumer<Throwable> dealWithException);

// f의 바디는 다음을 수행할 수 있다.
dealWithException(e);
```

콜백이 여러개면 이를 따로 제공하는 것보다는 한 객체로 이 메서드를 감싸는 것이 좋다.<br/>

```
void onComplete() ▶ 값을 다 소진했거나 에러가 발생해서 더 이상 처리할 데이터가 없을 때
void onError(Throwable throwable) ▶ 도중 에러가 발생했을 때
void onNext(T item) ▶ 값이 있을 때 
```

<h2>박스와 채널 모델</h2>
동시성 모델을 가장 잘 설계하고 개념화하려면 그림이 필요하다.<br/>
이 기법을 박스와 채널 모델이라고 부른다.<br/>

ex) f나 g를 호출하거나 p함수에 인수 x를 이용해 호출하고 그 결과를 q1와 q2에 전달하며 다시 이 두 호출의 결과로 함수 r을 호출한 다음 결과를 출력한다.<br/>
![15-7](https://user-images.githubusercontent.com/87962572/141988824-294eb6d0-3694-4703-8113-f893be611cff.PNG)

자바로 두 가지 방법으로 구현해 어떤 문제가 있는지 확인하자.

```
int t = p(x);
System.out.println(r(q1(t),q2(t)));
```

겉보기에는 깔끔해보이지만, 자바가 q1,q2를 차례로 호출하는데 이는 하드에어 병렬성의 활용과 거리가 멀다.<br/>

```
int t = p(x);
Future<Integer> a1 = executorService.submit(() -> q1(t));
Future<Integer> a2 = executorService.submit(() -> q2(t));
System.out.println(r(a1.get(), a2.get()));
```

박스와 채널 다이어그램의 모양상 p와 r을 Future로 감싸지 않았다.<br/>
p는 다른 어떤 작업보다 먼저 처리해야 하며 r은 모든 작업이 끝난 다음 가장 마지막으로 처리해야한다.<br/>

이런 대규모 시스템 구조가 얼마나 많은 수의 get()을 감당할 수 있는지 이해하기 어렵다.<br/>
이를 CompletableFuture와 콤비네이터를 이용해 문제를 해결한다.<br/>

<h2>CompletableFuture와 콤비네이터를 이용한 동시성</h2>

일반적으로 Future는 실행해서 get()으로 결과를 얻을 수 잇는 Callable로 만들어 진다.<br/>
하지만 CompletableFuture는 실행할 코드 없이 Future를 만들 수 있도록 허용하며 complete() 메서드를 이용해 나중에 어떤 값을 이용해 다른 스레드가 이를 완료할 수 있도<br/>
get()으로 값을 얻을수 있도록 허용한다.<br/>

```
public class CFComplete{  
  pubilc static void main(String[] args) throws ExcutionException, TnterrupedException[
    ExecutorService executorService = Executors.newFixedThreadPool(10);
    int x = 1337;
    
    completableFuture<Integer> a = new CompletableFuture<>();
    executorService.submt(() -> a.complete(f(x)));
    int b = g(x);
    System.out.println(a.get() + b);
    
    executorService.shutdown();
  }
}

public class CFComplete {
  public static void main(String[] args) throws ExecutionException, InterruptedException {
    ExecutorService executorService = Executors.newFixedThreadPool(10);
    int x = 1337;
    CompletableFuture<Integer> a = new CompletableFuture<>();
    executorService.submit(() -> b.complete(g(x)));
    int a = f(x);
    System.out.println(a + b.get());
    executorService.shutdown();
  }
}
```

f(x)의 실행이 끝나지 않거나 아니면 g(x)의 실행이 끝나지 않는 상황에서 get()을 가디려야 하므로 프로세시 자원을 낭비할 수 있다.
자바 8의 CompletableFuture를 이용하면 이 상황을 해결할 수 있었다.

자바에서는 자바8의 람다가 추가되면서 겨유 걸음마를 뗀 수준이다.

```
myStream.map(...).filter(...).sum()
```

다른 예로 compose(), andThen() 같은 메서드를 두 Function에 이용해 다른 Function을 얻을 수 있다.<br/>
CompletableFuture< T > 에 thenCombine메서드를 사용함으로 두 연산 결과를 더 효과적으로 더 할 수 있다.
thenCombine 메서는 다음과 같은 시그니처를 갖고 있다.

CompletableFuture< V > thenCombine(CompletableFuture < U > other, BiFunction< T, U, V > fn)

이 메서드는 두 개의 CompletableFuture 값 (T, U 결과 형식)을 받아 한 개의 새 값을 만든다.
처음 두 작업이 끝나면 두 결과 모두에 fn을 적용하고 블록하지 않은 상태로 결과 Future를 반환한다.

```
public class CFCombine {
  public static void main(String[] args) throws ExecutionException, InterruptedException {
    ExecutorService executorService = Executors.newFixedThreadPool(10);
    int x = 1337;
    CompletableFuture<Integer> a = new CompletableFuture<>();
    CompletableFuture<Integer> b = new CompletableFuture<>();
    CompletableFuture<Integer> c = a.thenCombine(b, (y, z)-> y + z); ▶ thenCombine 행이 핵심이다.
    executorService.submit(() -> a.complete(f(x)));
    executorService.submit(() -> b.complete(g(x)));
    System.out.println(c.get());
    executorService.shutdown();
  }
}
```

- Future a와 Future b의 결과를 알지 못한 상태에서 thenCombine은 두 연산이 끝났을 때 스레드 풀에서 실행된 연산을 만든다.
- 결과를 추가하는 세 번째 연산 c는 다른 두 작업이 끝날 때 까지는 스레드에서 실행되지 않는다.


![15-8](https://user-images.githubusercontent.com/87962572/141993843-828e8493-9332-455f-ad29-f0e5b9f219be.PNG)

thenCombine을 이용하면 f(x)와 g(x)가 끝난 다음에야 덧셈 계산이 실행된다.<br/>
get()을 기다리는 스레드가 큰 문제가 되지 않으므로 기존 자바 8의 Future를 이용한 방식도 해결 방법이 될 수 있다.<br/>
어떤 상황에서는 많은 수의 Future를 사용해야한다.<br/>
CompletableFuture와 콤비네이터를 이용해 get()에서 블록하지 않을 수 있고 그렇게 함으로 병렬 실행의 효율성은 높이고<br/>
데드락은 피하는 최상의 해결책을 구현할 수 있다.<br/>

<h2>발행-구독 그리고 리액티브 프로그래밍</h2>
Future와 CompletableFuture은 독립적 실행과 병렬성이라는 정식적 모델에 기반한다.<br/>
연산이 끝나면 get()으로 Future의 결과를 얻을 수 있다.<br/>
따라서 Future는 한번만 실행해 결과를 제공한다.<br/>

반면 리액티브 프로그래밍은 시간이 흐르면서 여러 Future 같은 객체를 통해 여러 결과를 제공한다.<br/>
먼저 온도계 객체를 예로 생각해보자.이 객체를 매 초마다 온도 값을 반복적으로 제공한다.<br/>
또 다른 예로 웹 서버 컴포넌트 응답을 기다리는 리스너 객체를 생각할 수 있다.<br/>
이 객체는 네트워크에 HTTP 요청이 발생하길 기다렸다가 이후에 결과 데이터를 생성한다.<br/>
다른 코드에서 온도 값 또는 네트워크 결과를 처리한다.<br/>
온도계와 리스너 객체는 다음 결과를 처리할 수 있도록 온도 결과나 다른 네트워크 요청을 기다린다.<br/>

Future같은 동작이 모두 사용되었지만 한 예제에서는 한번의 결과가 아니라 여러 번의 결과가 필요하다.<br/>
두번째 예제에서 눈여겨봐야할 또 다른 점은 모든 결과가 똑같이 중요한 반면 온도계 예제에서는 대부분의 사람에게 가장 최근의 온도만 중요하다.<br/>

```
구독자가 구독할 수 있는 발행자
이 연결을 구독이라 한다.
이 연결을 이용해 메세지를 전송한다.
```

<h3>두 플로를 합치는 예제</h3>
두 정보 소스로부터 발생하는 이벤트를 합쳐서 다른 구독자가 볼 수 있도록 발생하는 예를 통해 발행-구독의 특징을 간단하게 확인할 수 있다.
"=C1+C2"라는 공식을 포함하는 스프레드시트 셀 C3을 만들자.

```
private class SimpleCell {
  private int value = 0;
  private String name;
  public SimpleCell(String name){
      this.name = name;
  }
}

SimpleCell c2 = new SimpleCell("C2");
SimpleCell c1 = new SimpleCell("C1");
```
c1이나 c2의 값이 바뀌었을 때 c3가 두 값을 더하도록 어덯게 지정할 수 있을까?
c1과 c2에 이벤트가 발생했을 때 c3d을 구독하도록 해야한다.
그럴려면 다음과 같은 구독 Publisher< T > 가 필요하다.

```
interface Publisher< T > {
  void subscribe(Subscriber<? super T> subscriber);
}

// 이 인터페이스는 통신할 구독자를 인수로 받는다.
// SubScriber< T > 인터페이스는 onNext라는 정보를 전달할 단순 메서드를 포함하며 구현자가 필요한대로 이 메서드를 구현할 수 있다.
interface SubScriber< T > {
  void onNext(T t);
}

// 사실 Cell은 Publisher이며 동시에 Subscriber임을 알 수 있다.
private class SimpleCell implements Publisher<Integer>, Subscriber<Integer> {
  private int value = 0;
  private String name;
  private List<Subscriber> subscribers = new ArrayList<>(); 
  
  public SimpleCell(String name) {
    this.name = name;
  }
  
  @Override
  public void subscribe(Subscriber<? super Integer> subscriber) {
    subscribers.add(subscriber);
  }
  
  private void notifyAllSubscribers() { ▶ 새로운 값이 있음을 모든 구독자에게 알리는 메서드
    subscribers.forEach(subscriber -> subscriber.onNext(this.value));
  }
  
  @Override
  public void onNext(Integer newValue) {
    this.value = newValue;  ▶ 구독한 셀에 새 값이 생겼을 때 값을 갱신해서 반응함
    System.out.println(this.name + ":" + this.value); ▶ 값을 콘솔로 출력하지만 실제로는 UI의 셀을 갱신할 수 있음
    notifyAllSubscribers();  ▶ 값이 갱신되었음을 모든 구독자에게 알림
  }
}

SimpleCell c3 = new SimpleCell("C3");
SimpleCell c2 = new SimpleCell("C2");
SimpleCell c1 = new SimpleCell("C1");

c1.subscribe(c3);
c1.onNext(10); // c1을 10으로 갱신
c2.onNext(20); // c2을 20으로 갱신

C1:10
C3:10
C2:20
```
C3= C1+C2은 어떻게 구현할까? 

```
public class ArithmeticCell extends SimpleCell {
  private int left;
  private int right;
  public ArithmeticCell(String name) {
    super(name);
  }
  public void setLeft(int left) {
    this.left = left;
    onNext(left + this.right); 
  }
  
  public void setRight(int right) {
    this.right = right;
    onNext(right + this.left); 
  }
}

ArithmeticCell c3 = new ArithmeticCell("C3");
SimpleCell c2 = new SimpleCell("C2");
SimpleCell c1 = new SimpleCell("C1"); 
c1.subscribe(c3::setLeft);
c2.subscribe(c3::setRight);

c1.onNext(10); // C1의 값을 10으로 갱신
c2.onNext(20); // C2의 값을 20으로 갱신
c1.onNext(15); // C1의 값을 15으로 갱신

C1:10
C3:10
C2:20
C3:30
C1:15
C3:35
```

결과를 통해 15가 갱신되었을 때 C3이 즉시 반응해 자신의 값을 갱신한다는 사실을 확인할 수 있다.
발행자-구독자 상호작용의 멋진 점은 발행자 구독자의 그래프를 설정할 수 잇다는 점이다.

```
// C5=C3+C4처럼 C3과 C4에 의존하는 새로운 셀 C5를 만들 수 있다.
ArithmeticCell c5 = new ArithmeticCell("C5");
ArithmeticCell c3 = new ArithmeticCell("C3");

SimpleCell c4 = new SimpleCell("C4");
SimpleCell c2 = new SimpleCell("C2");
SimpleCell c1 = new SimpleCell("C1");

c1.subscribe(c3::setLeft);
c2.subscribe(c3::setRight);

c3.subscribe(c5::setLeft);
c4.subscribe(c5::setRight)
```

최종적으로 C1은 15, C2는 20, C4는 3이므로 C5는 38의 값을 갖는다.

