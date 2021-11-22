<h1>CompletableFuture : 안정적 비동기 프로그래밍</h1>
<h2>Future의 단순 활용</h2>
미래의 어느 시점에 결과를 얻는 모델에 활용할 수 있도록 Future 인터페이스를 제공하고 있다.<br/>
비동기 계산 모델링하는 데 Future를 이용할 수 있으며, Future는 계산이 끝났을 때 결과에 접근할 수 있도록 참조를 제공한다.<br/>
시간이 걸릴 수 있는 작업을 Future 내부로 설정하면 호출자 스레드가 결과를 기다리는 동안 다른 유용한 작업을 수행할 수 있다.<br/>
<b>Future를 이용하려면 시간이 오래 걸리는 작업을 Callable 객체 내부로 감싼 다음에 ExecutorService에 제출해야 한다.<br/></b>

ex) 세탁소 주인은 드라이클리닝이 언제 끝날지 적힌 영수증(Future)을 줄 것이다.<br/>
드라이클리닝이 진행되는 동안 우리는 원하는 일을 할 수 있다.<br/>
Future는 저수준의 스레드에 비해 직관적으로 이해하기 쉽다는 장점이 있다.<br/>

```
// Future로 오래 걸리는 작업을 비동기적으로 실행하기
ExecutorService executor = Executors.newCachedThreadPool();       ☞ 스레드 풀에 태스크를 제출하려면 ExecutorService를 만들어햐 한다.
Future<Double> future = executor.submit(new Callable<Double>() {  ☞ Callable 을 ExecutorService로 제출한다.
  public Double call(){
    return doSomeLongComputation();                               ☞ 시간이 오래 걸리는 작업은 다른 스레드에서 비동기적으로 실행한다.
  }
});
doSomethingElse();                                                ☞ 비동기 작업을 수행하는 동안 다른 작업을 한다.
try{
  Double result = future.get(1,TimeUnit.SECONDS); ☞ 비동기 작업의 결과를 가져온다. 결과가 준비되어 있지 않으면 호출 스레드가 블록된다. 하지만 최대 1초까지만 기다린다.
}catch(ExecutionException ee){
  // 계산 중 예외 발생
}catch(InterruptedException ie){
  // 현재 스레드에서 대기 중 인터럽트 발생
}catch(TimeoutException te){
  // Future가 완료되기 전에 타임아웃 발생
}
```

ExecutorService에서 제공하는 스레드가 시간이 오래 걸리는 작업을 처리하는 동안 우리 스레드로 다른 작업을 동시에 실행할 수 있다.<br/>
다른 작업을 처리하다가 시간이 오래 걸리는 작업의 결과가 시점이 되었을 때 Future의 get 메서드로 결과를 가져올 수 있다.<br/>
get 메서드를 호출했을 때 이미 게산되어 완료되면 결과가 준비되었다면 즉시 결과를 반환하지만 결과가 준비되지 않았다면 작업이 완료될 때까지 우리 스레드를 블록시킨다.<br/>

![16-1](https://user-images.githubusercontent.com/87962572/142866979-28104602-f345-463f-af7b-3125011cb642.PNG)

이 시나리오의 문제점은?<br/>
작업이 끝나지 않은 문제가 있을 수 있으므로 get메서드를 오버로드래서 우리 스레드가 대기할 최대 타임아웃 시간을 설정하는 것이 좋다.<br/>

<h3>Future 제한</h3>
Future 인터페이스가 비동기 계산이 끝났는지 확인할 수 있는 isDone 메서드, 계산이 끝나길 기다리는 메서드, 결과 회수 메서드 등을 제공함을 보여준다.<br/>
Future의 결과가 있을 때 이들의 의존성은 표현하기가 어렵다. 요구사항을 쉽게 구현할 수 있어야야 한다.

- 두 개의 비동기 계산 결과를 하나로 합친다. 두 가지 계산 결과는 서로 독립적일 수 있으며 또한 두 번째 결과가 첫 번째 결과에 의존하는 상황일 수 있다.<br/>
- Future 집합이 실행하는 모든 태스크의 완료를 기다린다.<br/>
- Future 집합에서 가장 빠른 완료되는 태스크를 기다렸다가 결과를 얻는다.<br/>
- 프로그램적으로 Future를 완료시킨다. (즉, 비동기 동작에 수동으로 겨로가 제공).<br/>
- Future 완료 동작에 반응한다. (즉, 결과를 기다리면서 블록되지 않고 결과가 준비되었다는 알람을 받은 다음에 Future의 결과로 원하는 추가 동작을 수행할 수 있음).<br/>

★ 이 장에서는 위에 설명한 기능을 선언형으로 이용할 수 있도록 자바 8에서 새로 제공하는 CompletableFuture 클래스를 살펴본다.

<h3>CompletableFuture로 비동기 애플리케이션 만들기</h3>
예산을 줄일 수 있도록 여러 온라인상점 중 가장 저렴한 가격을 제시하는 상점을 찾는 애플리케이션을 만드는 완성해가는 예제를 이용해서 CompletableFuture의 기능을 살펴보자.<br/>
이 애플리케이션을 만드는 동안 다음과 같은 기술을 배울 수 있다.<br/>
- 첫째, 고객에게 비동기 API를 제공하는 방법을 배운다.<br/>
- 둘째, 동기 API를 사용해야 할 때 코드를 비블록으로 만드는 방법을 배운다. 두 개의 비동기 동작을 파이프라인으로 만드는 동작과 두 개의 동작 결과를 하나의 비동기 계산으로 합치는<br/>
방법을 살펴본다. 예를 들어 온라인상점에서 우리가 사려는 물건에 대응한느 할인 코드를 반환한다고 가정하자.<br/>
우리는 다른 원격 할인 서비스에 접근해서 할인 코드레 해당하는 할인율을 찾아야 한다.<br/>
그래야 원래 가격에 할인율을 적용해서 최종 결과를 계산할 수 있다.<br/>
- 셋째, 비동기 동작의 완료에 대응하는 방법을 배운다. 즉, 모든 상점에서 가격 정보를 얻을 때까지 기다리는 것이 아니라 각 상점에서 가격 정보를 얻을 때마다 즉시 최저가격을 찾는 <br/>
애플리케이션을 갱신하는 방법을 설명한다.<br/>

* 동기 API : 메서드를 호출한 다음에 메서드가 계산을 완료할 때까지 기다렸다가 메서드가 반환되면 호출자는 반환된 값으로 계속 다른 동작을 수행한다.<br/>
호출자와 비호출자가 각각 다른 스레드에서 실행되는 상황이었더라도 호출자는 피호출자의 동작 완료를 기다렸을 것이다. 이처럼 동기 API를 사용하는 상황을 블록 호출이라고 한다.<br/>

* 비동기 API : 메서드가 즉시 반환되며 끝내지 못한 나머지 작업을 호출자 스레드와 동기적을 실행될 수 있도록 다른 스레드에 할당된다. (비블록 호출)<br/>
다른 스레드에 할당된 나머지 계산 결과는 콜백 메서드를 호출해서 전달하거나 호출자가 '계산 결과가 끝날 때까지 기다림'메서드를 추가로 호출하면서 전달된다.<br/>
주로 I/O 시스템 프로그래밍에서 이와 같은 방식으로 동작을 수행한다.<br/>
계산 동작을 수행하는 동안 비동기적으로 디스크 접근을 수행한다.<br/>

<h2>비동기 API 구현</h2>

```
public class Shop{
  // 제품명에 해당하는 가격을 반환하는 메서드 정의 코드.
  public double getPrice(String product){
    // 구현해야함.
  }
}
```

getPrice 메서드는 상점의 데이터베이스를 이용해서 가격 정보를 얻는 동시에 다른 외부 서비스에도 접근할 것이다.

```
public static void delay(){
  try {
    Thread.sleep(1000L);
  }catch(InterrupedException e){
    throws new RuntimeException(e);
  }
}
```

deplay를 이용해서 지연을 흉내 낸 다음에 임의의 계산값을 반환하도록 getPrice를 구현할 수 있다.
제품명에 charAt을 적용해서 임의의 게산값을 반환한다.

```
public double getPrice(String product){
  return calculatePrice(product);
}

private double calculatePrice(String product){
  delay();
  return random.nextDouble() * product.charAt(0) + product.charAt(1);
}
```
