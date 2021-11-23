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

deplay를 이용해서 지연을 흉내 낸 다음에 임의의 계산값을 반환하도록 getPrice를 구현할 수 있다.<br/>
제품명에 charAt을 적용해서 임의의 게산값을 반환한다.<br/>

```
public double getPrice(String product){
  return calculatePrice(product);
}

private double calculatePrice(String product){
  delay();
  return random.nextDouble() * product.charAt(0) + product.charAt(1);
}
```

<h3>동기 메서드를 비동기 메서드로 변환</h3>
동기 메서드 getPrice를 비동기 메서드로 변환하려면 다음 코드처럼 먼저 이름(getPriceAsync)과 반환값을 바꿔야 한다.<br/>
Future는 결과값의 핸들일 뿐이며 계산이 완료되며 get 메서드로 결과를 얻을 수 있다.<br/>
getPriceAsync 메서드는 즉시 반환되므로 호출자 스레드는 다른 작업을 수행할 수 있다.<br/>

```
public Future<Double> getPriceAsync(String product) {
    CompletableFuture<Double> futurePrice = new CompletableFuture<>();  ☞ 계산 결과를 포함할 CompletableFuture를 생성한다.
        new Thread( () -> {
            double price = calculatePrice(product);  ☞  다른 스테르에서 비동기저그로 계산을 수행한다.
            futurePrice.complete(price);             ☞  오랜 시간이 걸리면 계산이 완료되며 Future에 값을 설정한다.
        }).start();
    return futurePrice;                              ☞ 계산 결과가 완료되길 기다리지 않고 Future를 반환한다.  
}
```

비동기 계산과 완료 결과를 포함하는 CompletableFuture 인스턴스를 만들었다.<br/>
실제 가격을 계산할 다른 스레드를 만든 다음에 오래 걸리는 계산 결과를 기다리지 않고 '결과를 포함할 Future 인스턴스를 바로 반환'했다.<br/>

```
Shop shop = new Shop("BestShop");
long start = System.nanoTime();
Future<Double> futurePrice = shop.getPriceAsync("my favorite product"); 
long invocationTime = ((System.nanoTime() - start) / 1_000_000);
System.out.println("Invocation returned after " + invocationTime 
 + " msecs");
 
// 제품의 가격을 계산하는 동안
doSomethingElse();
// 다른 상점 겸색 등 다른 작업 수행
try {
    double price = futurePrice.get();  ☞ 가격 정보가 있으면 Future에서 가격 정보를 읽고, 가격 정보가 없으면 가격 정보를 받을 때까지 블록한다.
    System.out.printf("Price is %.2f%n", price);
} catch (Exception e) {
    throw new RuntimeException(e);
}
long retrievalTime = ((System.nanoTime() - start) / 1_000_000);
System.out.println("Price returned after " + retrievalTime + " msecs");
```

상점은 비동기 API를 제공하므로 즉시 Future를 반환한다.<br/>
클라이언트는 반환된 Future를 이용해서 나중에 결과를 어더을 수 있다.<br/>
그 사이 클라이언트는 다른 상점에 가격 정보를 요청하는 등 첫 번째 상점으 결과를 기다리면서 대기하지 않고 다른 작업을 처리할 수 있다.<br/>

클라이언트가 특별히 할일이 없으면 Future의 get메서드를 호출한다.<br/>
이때 Future가 결과값을 가지고 있다면 Future에 포함된 값을 읽거나 아니면 값이 계산될 때까지 블록한다.<br/>

<h3>에러 처리 방법</h3>
위의 소스에서 가격을 계산하는 동안 에러가 발생하면 어떻게 될까?<br/>
예외가 발생하면 해당 스레드에만 영향을 미친다.<br/>
에러가 발생해도 가격 계산은 계속 진행되며 일으 순서가 꼬인다.<br/>
<b>결과적으로 클라이언트는 get 메서드가 반환될 때까지 영원히 기다리게 될 수도 있다.</b>

이처럼 블록 문제가 발생할 수 있는 상황에서는 타임아웃을 활용하는 것이 좋다.<br/>
그래야 문제가 발생했을 때 클라이언트가 영원히 블록되지 않고 타임아웃 시간이 지나면 TimeoutException을 받을 수 있다.<br/>
하지만 이때 왜 에러가 발생했는지 알 수 있는 방법이 없다.<br/>
따라서 completeExceptiionally 메서드를 이용해서 CompletableFuture 내부에서 발생한 예외를 클라이언트로 전달해야 한다.<br/>

```
public Future<Double> getPriceAsync(String product) {
    CompletableFuture<Double> futurePrice = new CompletableFuture<>();
    new Thread( () -> {
    try {
        double price = calculatePrice(product);
        futurePrice.complete(price); ☞ 계산이 정상적으로 종료되면 Future에 가격 정보를 저장한 채로 Future를 종료한다.
    } catch (Exception ex) {
        futurePrice.completeExceptionally(ex);  ☞ 도중에 문제가 발생하면 발생한 에러를 포함시켜 Future를 종료한다.
    }
    }).start();
    
    return futurePrice;
}
```

이제 클라이언트는 가격 계산 메서드에서 발생한 예외 파라미터를 포함하는 ExecutionExeption을 받게된다.<br/>

<h3>팩터리 메서드 supplyAsync 로 CompletableFuture만들기</h3>
지금까지 CompletableFuture를 직접 만들었다. 좀 더 간단하게 CompletableFuture을 만드는 방법도 있다.

```
public Future<Double> getPriceAsync(String product){
  return CompletableFuture.supplyAsync(() -> calculatePrice(product));
}
```

supplyAsync 메서드는 Supplier를 인수로 받아서 CompletableFuture를 반환한다.
CompletableFuture는 Supplier를 실행해서 비동기적으로 결과를 생성한다.
ForkJoinPool의 Executor 중 하나가 Supplier를 실행할 것이다.
두 번째 인수를 받은 오버로드 버전의 supplyAsync 메서드를 이용해서 다른 Executor를 지정할 수 있다.
결국 모든 다른 CompletableFuture의 팩토리 메서드에 Executor를 선택적으로 전달할 수 있다.

<h2>비블록 코드 만들기</h2>
동기 API를 이용해서 최저가격 검색 애플리케이션을 개발해야한다.

```
List<Shop> shops = Arrays.asList(new Shop("BestPrice"), new Shop("LetsSaveBig"), new Shop("MyFavoriteShop"), new Shop("BuyItAll"));

// 제품명을 입력하면 상점 이름과 제품 가격 문자열 정보를 포함하는 List를 반환하는 메서드를 구현해야한다.
public List<String> findPrices(String product){
  return shops.stream().map(shop -> String.format("%s price is $.2f", shop.getName(), shop.getPrice(product)))
              .collect(toList());
}

// findPrices의 결과와 성능 확인
long start = System.nanoTime();
System.out.println(findPrices("myPhone27S");
long duration = (System.nanoTime() - start) / 1_000_000;
System.out.println("Done is "+ duration  " msecs");
```
다음은 예제 실행 결과다.

[BestPrice price is 123.26, LetsSaveBig price is 169.47, MyFavoriteShop price 
is 214.13, BuyItAll price is 184.74]
Done in 4032 msecs

<h3>병렬 스트림으로 요청 병렬화하기</h3>

```
public List<String> findPrices(String product) {
  return shops.parallelStream()
        .map(shop -> String.format("%s price is %.2f", shop.getName(), shop.getPrice(product)))
        .collect(toList());
}
```

새로운 버전의 성능을 확인해보자.
[BestPrice price is 123.26, LetsSaveBig price is 169.47, MyFavoriteShop price
 is 214.13, BuyItAll price is 184.74]
Done in 1180 msecs

<h3>CompletableFuture로 비동기 호출 구현하기</h3>

```
List<CompletableFuture<String>> priceFutures =
  shops.stream()
               .map(shop -> CompletableFuture.supplyAsync(
                   () -> String.format("%s price is %.2f", shop.getName(), shop.getPrice(product))))
               .collect(toList());
```

CompletableFuture를 포함하는 리스트 List< CompletableFuture < String > >를 얻을 수 있다.<br/>
리스트의 CompletableFuture는 각각 게산 결과가 끝난 상점의 이름 문자열을 포함한다.<br/>

두번째 map연산을 List< CompletableFuture < String > > 에 적용할 수 있다.<br/>
리스트의 모든 CompletableFuture에 join을 호출해서 모든 동작이 끝나기를 기다린다.<br/>
CompletableFuture클래스의 join메서드는 Future 인터페이스의 get 메서드와 같은 의미를 갖는다.<br/>
다만 join은 아무 예외도 발생시키지 않는다는 점이 다르다.<br/>

```
public List<String> findPrices(String product) {
  List<CompletableFuture<String>> priceFutures =
    shops.stream()
    .map(shop -> CompletableFuture.supplyAsync(  ☞ CompletableFuture로 각각의 가격을 비동기적으로 계산한다.
      () -> shop.getName() + " price is " + shop.getPrice(product)))
    .collect(Collectors.toList());
  
  return priceFutures.stream().map(CompletableFuture::join) ☞ 모든 비동기 동작이 끝나기를 기다린다.
          .collect(toList());
}
```

두 map연산을 하나의 스트림 처리 파이프라인으로 처리하지 않고 두 개의 스트림 파이프라인으로 처리했다는 사실에 주목하자. <br/>
스트림 연산은 게으름 특성이 있으므로 하나의 파이프라인으로 연산을 처리했다면 모든 가격 정보 요청 동작이 동기적, 순차적으로 이루어지는 결과가 된다.<br/>

![16-2](https://user-images.githubusercontent.com/87962572/143036200-2801c47f-739f-48b2-bf90-ca2e2240e570.PNG)

CompletableFuture로 각 상점의 정보를 요청할 때 기존 요청 작업이 완료되어야 join이 결과를 반환하면서 다음 상점으로 정보를 요청할 수 있기 때문이다.

위의 사진의 윗부분은 순차적으로 평가를 진행하는 단일 파이프라인 스트림 처리 과정을 보여준다. (점선 표시 부분)
즉, 이전 요청의 처리가 완전히 끝난 다음에 새로 만든 CompletableFuture가 처리된다.

반면, 아래쪽은 우선 CompletableFuture를 리스트로 모든 다음에 다른 작업과 독립적으로 각자의 작업을 수행하는 모습을 보여준다.

[BestPrice price is 123.26, LetsSaveBig price is 169.47, MyFavoriteShop price 
 is 214.13, BuyItAll price is 184.74, ShopEasy price is 166.08]
Done in 5025 msecs 

순차적인 블록 방식의 구현에 비해서는 빨라졌지만 병렬 스트림을 사용한 구현보다는 2배가 느리다.

<h3>커스텀 Executor 사용하기</h3>
애플리케이션이 실제로 필요한 작업량을 고려한 풀에서 관리하는 스레드 수에 맞게 Executor를 만들수 있으면 좋을 것이다.

풀에서 관리하는 스레드 수를 어떻게 결정할 수 있을까?

```
스레드 풀 크기 조절

스레드 풀이 너무 크면 cpu와 메모리 자원을 서로 경쟁하느라 시간을 낭비할 수 있다.
반면, 스레드 풀이 너무 작으면 cpu의 일부 코어는 활용되지 않을 수 있다. 

Nthreads = Ncpu * U cpu * (1 + W/C)

공식에서 Nthreads, U cpu, W/C 는 각각 다음을 의미한다.
 - N cpu는 Runtime.getRuntime().availableProcessors()가 반환하는 코어 수 
 - U cpu는 0과 1 사이의 값을 갖는 cpu 활용 비율
 - W/C 는 대기시간과 계산 시간의 비율
```

애플리케이션은 상점의 응답을 대략 99 퍼센트의 시간만큼 기다리므로 W/C 비율을 100으로 간주할 수 있다.
즉, 대상 cpu 활용률이 100퍼센트라면 400 스테드를 갖는 풀을 만들어야 함을 의미한다.

스레드 수가 너무 많으면 오히려 서버가 크래시될 수 있으므로 하나의 Executor에서 사용할 스레드의 최대 개수는 100 이하로 설정하는 것이 바람직 하다.

```
private final Executor executor = 
  Executors.newFixedThreadPool(Math.min(shops.size(), 100),  ☞ 상점 수 만큼의 스레드를 갖는 풀을 생성한다.(스레드 수의 범위는 0과 100사이).
    (Runnable r) -> {
      Thread t = new Thread(r);
      t.setDaemon(true);  ☞  프로그램 종료를 방해하지 않는 데몬 스레드를 사용한다.
      return t;
    });
```

우리가 만드는 풀은 데몬 스레드를 포함한다.
자바에서 일반 스레드가 실행 중이면 자바 프로그램은 종료되지 않는다.
어떤 이벤트를 한없이 기다리면서 종료되지  않는 일반 스레드가 잇으면 문제가 될 수 있다.
반면, 데몬 스레드는 자바 프로그램이 종료될 때 강제로 실행이 종료될 수 있다.

결국, 애플리케이션의 특성에 맞는 Executor를 만들어 CompletableFuture를 활용하는 것이 바람직하다.
비동기 동작을 많이 사용하는 상황에서는 지금 살펴본 기법이 가장 효과적일 수 있음을 기억하자.

<h2>비동기 작업 파이프라인 만들기</h2>
우리와 계약을 맺은 모든 상점이 하나의 할인 서비스를 사용하기로 했다고 가정하자.
할인 서비스에서는 서로 다른 할인율을 제공하는 다섯 가지 코드를 제공한다.

```
public class Discount {
 public enum Code {
  NONE(0), SILVER(5), GOLD(10), PLATINUM(15), DIAMOND(20);
  
  private final int percentage;
  
  Code(int percentage) {
    this.percentage = percentage;
  }
 }
}

public String getPrice(String product) {
 double price = calculatePrice(product);
 Discount.Code code = Discount.Code.values()[
 random.nextInt(Discount.Code.values().length)];
 return String.format("%s:%.2f:%s", name, price, code); 
}

private double calculatePrice(String product) {
 delay();
 return random.nextDouble() * product.charAt(0) + product.charAt(1);
}
```

<h3>할인 서비스 구현</h3>
우리의 최저가격 검색 애플리케이션은 여러 상점에서 가격 정보를 얻어오고, 결과 문자열을 파싱하고, 할인 서버에 질의를 보낼 준비가 되었다.
할인 서버에서 할인율을 확인해서 최종 가격을 계산할 수 있다.

```
public class Quote {
 private final String shopName;
 private final double price;
 private final Discount.Code discountCode;
 public Quote(String shopName, double price, Discount.Code code) {
   this.shopName = shopName;
   this.price = price;
   this.discountCode = code;
 }

 public static Quote parse(String s) {
   String[] split = s.split(":");
   String shopName = split[0];
   double price = Double.parseDouble(split[1]);
   Discount.Code discountCode = Discount.Code.valueOf(split[2]);
   
   return new Quote(shopName, price, discountCode);
 }
 
 public String getShopName() { return shopName; }
 public double getPrice() { return price; }
 public Discount.Code getDiscountCode() { return discountCode; }
}
```

상점에서 얻은 문자열을 정적 팩토리 메서드 parse로 넘겨주면 상점 이름, 할인전 가격, 할이된 가격 정보를 포함하는 Quote 클래스 인스턴스가 생성된다.

Discount 서비스에서는 Quote 객체를 인수로 받아 할인된 가격 문자열을 반환하는 applyDiscount 메서드로 제공한다.

```
public class Discount {
  public enum Code {
    // 소스 생략
  }
 
  public static String applyDiscount(Quote quote) {
    return quote.getShopName() + " price is " + Discount.apply(quote.getPrice(), quote.getDiscountCode());
  }
  
  private static double apply(double price, Code code) {
    delay(); 
    return format(price * (100 - code.percentage) / 100);
  }
}
```

<h3>할인 서비스 사용</h3>
Discount는 원격 서비스이므로 다음 코드에서 보여주는 것처럼 1초의 지연을 추가한다.
가장 쉬운 방법으로 findPrices 메서드를 구현한다.

```
public List<String> findPrices(String product) {
  return shops.stream()
    .map(shop -> shop.getPrice(product)) ◀ 상점에서 할인 전 가격 얻기
    .map(Quote::parse)                   ◀ 상점에서 반환한 문자열을 Quote 객체로 반환한다.
    .map(Discount::applyDiscount)        ◀ Discount 서비스를 이용해서 각 Quote에 할인을 적용한다.
    .collect(toList());        
}
```

순차적으로 다섯 상점에 가격 정보를 요청하느라 5초가 소요되었고, 다섯 상점에서 반환한 가격 정보에 할인 코드를 적용할 수 있도록 할인 서비스에 또 시간이 소요되었다.
병렬 스트림을 이용하면 성능을 쉽게 개선할 수 있다고는 하지만, 병렬 스트림에서는 스트림이 사용하는 스레드 풀의 크기가 고정되어 있어서 상점 수가 늘어낫을 때 처럼
검색 대상이 확장되었을 때 유연하게 대응할 수 없다는 사실도 확인했다.
따라서 CompletableFuture에서 수행하는 태스크를 설정할 수 있는 커스텀 Executor를 정의함으로써 우리의 cpu사용을 극대화할 수 있다.

<h3>동기 작업과 비동기 작업 조합하기</h3>
CompletableFuture에서 제공하는 기능으로 findPrices 메서드를 비동기적으로 재구현하자.

```
public List<String> findPrices(String product) {
  List<CompletableFuture<String>> priceFutures =
    shops.stream().map(shop -> CompletableFuture.supplyAsync( 
                        () -> shop.getPrice(product), executor))
                  .map(future -> future.thenApply(Quote::parse)) 
                  .map(future -> future.thenCompose(quote -> 
                      CompletableFuture.supplyAsync(
                      () -> Discount.applyDiscount(quote), executor)))
                  .collect(toList());
                  
return priceFutures.stream().map(CompletableFuture::join).collect(toList());
}
```

세가지 변환 과정을 보여준다. 위에 사용한 세 개의 map연산을 적용한다.

![16-3](https://user-images.githubusercontent.com/87962572/143040949-1029db62-232f-46ea-919f-da801c70a705.PNG)

<h3>가격 정보 얻기</h3>
팩토리 메서드 supplyAsync 에 람다 표현식을 전달해서 비동기적으로 상점에서 정보를 조회했다.
반환의 결과는 Stream < CompletableFuture < String > > 이다. 각 CompletableFuture는 작업이 끝났을 때 해당 상점에서 반환하는 문자열 정보를 포함한다.

<h3>Quote 파싱하기</h3>
두 번째 변환 과정에서는 첫 번째 문자열을 Quote로 반환한다. 파싱 동작에서는 언격 서비스나 i/o가 없으므로 원하는 즉시 지연 없이 동작을 수행할 수 있다.
첫 번째 과정에서 생성된 CompletableFuture에 thenApply 메서드를 호출한 다음에 문자열을 Quote 인스턴스로 변환하는 Function으로 전달한다.

여기서 thenApply 메서드는 끝날때까지 블록하지 않는다!
즉, CompletableFuture가 동작을 완전히 완료한 다음에 thenApply 메서드로 전달된 람다 표현식을 적용할 수 있다.

thenComplose메서드는 첫 번째 연산의 결과를 두번째 연산으로 전달한다.
즉, 첫 번쨰 CompletableFuture에 thenCompose 메서드를 호출하고 Function에 넘겨주는 식으로 두 CompletableFuture를 조합할 수 있다.

세 개의 map 연산 결과 스트림의 요소를 리스트로 수집하면 List< CompltableFuture < String > > 형식의 자료를 얻을 수 있다.
마지막으로 CompletableFuture가 완료됙를 기다렸다가 join으로 결과 값을 추출할 수 있다.

<h3>독립 CompletableFuture와 비독립 completabldFuture 합치기</h3>

첫번째 CompletableFuture의 동작 완료와 관게없이 두 번째 completableFuture를 실행할 수 있어야한다.
이런 상황에서는 thenCombine 메서드를 사용하면 된다.
thenCombine 메서드는 BiFunction을 두번째 인수로 받는다. BiFunction은 두 개의 CompletableFuture 결과를 어떻게 합칠지 정의한다.
thenCompose 와 마찬가지로 thenCombine 메서드에도 Async버전이 존재한다.
thenCombineAsync 메서드에서는 BiFunction이 정의하는 조합 동작이 스레드 풀로 제출되면서 별도의 태스트에서 비동기적으로 수행된다.

예제에서는 유로 가격정보를 제공하는데 고객에게는 항상 달러 가격을 보여줘야한다고 가정하자. 그랬을 때 주어진 상품의 가격을 상점에 요청하는 한편 원격 환율 
교환 서비스를 이용해서 유로와 달러의 현재 환율을 비동기적으로 요쳥해야 한다.
두 가지 데이터를 얻었으면 가격에 환율을 곱해서 결과를 합칠 수 있다.

```
Future<Double> futurePriceInUSD = 
  CompletableFuture.supplyAsync(() -> shop.getPrice(product))  ◀ 제품가격 정보를 요청하는 첫 번째 태스크를 생성한다.
    .thenCombine(
      CompletableFuture.supplyAsync(
        () -> exchangeService.getRate(Money.EUR, Money.USD)),  ◀ 환율 정보를 요청하는 독립적인 두 번째 태스크를 생성한다.
      (price, rate) -> price * rate ◀ 두 결과를 곱해서 가격과 환율 정보를 합친다.
  ));
```

위에서 합치는 연산은 단순한 곱셈이므로 별도의 태스크에서 수행하여 자원을 낭비할 필요가 없다.

![16-4](https://user-images.githubusercontent.com/87962572/143044077-fc736514-cd76-41ee-a6ea-4eeaf22bd919.PNG)

<h3>Future의 리플렉션과 CompletableFuture의 리플렉션</h3>

```
ExecutorService executor = Executors.newCachedThreadPool();  ◀ 태스크를 스레드 풀에 제출할 수 있도록 ExecutorService를 생성한다.
final Future<Double> futureRate = executor.submit(new Callable<Double>() { 
  public Double call() {
    return exchangeService.getRate(Money.EUR, Money.USD);  ◀ EUR,USD 환율 정보를 가져올 Future를 생성한다.
}});
 
Future<Double> futurePriceInUSD = executor.submit(new Callable<Double>() { 
  public Double call() {
    double priceInEUR = shop.getPrice(product);  ◀  두 번째 Future로 상점에서 요청 제품의 가격을 검색한다.
    return priceInEUR * futureRate.get();        ◀ 가격을 검색한 Future를 이용해서 가격과 환율을 곱한다.
}});
```

Executor에 EUR와 USD 간의 환율 검색 외부 서비스를 이용하는 Callable을 submit로 인수로 전달해서 첫번째 Future를 만들었다.
상점에서 해당 제품의 가격을 EUR로 반환하는 두번째 Future를 만들었다. 마지막으로 EUR 가격 정보에 환율을 곱한다.
가격과  환율을 곱하는 세번째 Future를 만드는 것과 같다.

<h3>타입아웃 효과적으로 사용하기</h3>
Future의 계산 결과를 읽을 때는 무한정 기다리는 상황이 발생할 수 있으므로 블록을 하지 않는 것이 좋다.
orTimeout 메서드는 지정된 시간 지난 후에 CompletableFuture를 TimeoutException으로 완료하면서 또 다른 CompletableFuture를 반환할 수 있도록 ScheduledThreadExcutor를 활용한다.
이 메서드를 이용하면 계산 파이프라인을 연결하고 여기에서 TimeoutException이 발생했을 때 사용자가 쉽게 이해할 수 있는 메세지를 제공할 수 있다.
Future가 3초 후에 작업을 끛내지 못할 경우 TimeoutException이 발생하도록 메서드 체인의 끝에 orTimeout메서드를 추가할 수 있다.

```
Future<Double> futurePriceInUSD = 
  CompletableFuture.supplyAsync(() -> shop.getPrice(product)) 
  .thenCombine(
    CompletableFuture.supplyAsync(
      () -> exchangeService.getRate(Money.EUR, Money.USD)), 
      (price, rate) -> price * rate 
    ))
  .orTimeout(3, TimeUnit.SECONDS);
```

일시적으로 서비스를 이용할 수 없는 상황에서는 꼭 서버에서 얻은 값이 아닌 미리 지정된 값을 사용할 수 잇는 상황도 있다.

```
Future<Double> futurePriceInUSD = 
   CompletableFuture.supplyAsync(() -> shop.getPrice(product)) 
     .thenCombine(
     CompletableFuture.supplyAsync(
       () -> exchangeService.getRate(Money.EUR, Money.USD))
       .completeOnTimeout(DEFAULT_RATE, 1, TimeUnit.SECONDS),  ◀ 환전 서비스가 일 초 안에 결과를 제공하지 않으면 기본 환율값을 사용
       (price, rate) -> price * rate 
     ))
   .orTimeout(3, TimeUnit.SECONDS);
```

orTimeout 메서드처럼 completeOnTimeout메서드는 completableFuture를 반환하므로 이 결과를 다른 CompletableFuture 메서드와 연결 할 수 있다.
2개의 타입아웃을 설정했는데, 한 개는 3초 안에 연산을 마치지 못하는 상황에서 발생하고, 다른 타임아웃은 1초 이후에 환율을 얻지 못했을 때 발생한다.
하지만, 두 번째 타임아웃이 발생하면 미리 지정된 값을 사용한다.

<h2>CompletableFuture의 종료에 대응하는 방법</h2>
항상 1초를 지연하는 delay 대신 다음 코드처럼 randomDeplay메서드를 사용한다.

```
private static final Random random = new Random();
  public static void randomDelay() {
   int delay = 500 + random.nextInt(2000);  // 0.5초에서 2.5초 사이의 임의의 지연을 흉내 내는 메서드
   try {
    Thread.sleep(delay);
   } catch (InterruptedException e) {
    throw new RuntimeException(e);
   }
  }
```
