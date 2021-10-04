
 자바 7이 등장하기 전에는 데이터 컬렉션을 병렬로 처리하기가 어려웠다.  
 데이터를 서브파트로 분할하고 분한될 서브파트를 각각 스레드로 할당한다.  
 스레드는 할당한 다음에는 의도치 않은 레이스 컨디션이 발생하지 않도록 적절한 동기화를 추가해야하고, 마지막으로 부분 결과를 합쳐야된다.  
 자바 7은 더 쉽게 병렬화를 수행하면서 에러를 최소화할 수 있도록 <b>포크/조인 프레임워크</b> 기능을 제공한다.  
 
 이번 chapter에서는 스트림으로 데이터 컬렉션 관련 동작이 얼마나 쉽게 실행가능한지를 보여준다.  
 스트림을 이용하면 순차 스트림을 병렬 스트림으로 자연스럽게 바꿀 수 있다.  
 병렬 스트림이 내부적으로 어떻게 처리되는지 알아야 스트림을 잘못 사용하는 상황을 피할 수 있다.  
 
 <h1>7.1 병렬 스트림</h1>  
 스트림 인터페이스를 이용하면 아주 간단하게 요소를 병렬로 처리할 수 있다.  
 컬렉션에서 parallelStream을 호출하면 병렬 스트림이 생성된다.  
 병렬 스트림이란 각각의 스레드에서 처리할 수 이도록 스트림 요소를 여러 청크로 분할한 스트림이다.  
 따라서 병렬 스트림을 이용하면 모든 멀티 코더 프로세서가 각각의 청크를 처리하도록 할당한다.  
 
 ex) 숫자 n을 인수로 받아 1~n 까지의 모든 숫자의 합계를 반환하는 메서드를 구현한다고 가정하자.  
 숫자로 이루어진 무한 스트림을 만든 다음에 인수로 주어진 크기로 스트림을 제한하고 두 숫자를 BinaryOperator로 리듀싱 작업을 수행할 수 있다.  
 public long sequentialSum(long n){  
  
  return Stream.iterate(1L, i -> i+1) <- 무한 자연수 스트림 생성  
               .limit(n) <- n 개 이하로 제한  
               .reduce(0L, Long::sum); <- 모든 숫자를 더하는 스트림 리듀싱 연산  
 }  
 
 vs 전통적인 자바에서는 다음과 같이 사용  
 public long iterativeSum(long n){  
  long result = 0;  
  for(long l = 1L; i<= n; i++) result += i;  
  return result;  
 }  
  
 특히 n값이 커지게 되면 이 연산으로 병렬 처리하는 거시 좋을 것이다.
 
 
 <h3>7.1.1 순차 스트림을 병렬 스트림으로 변환하기</h3>
 
 순차 스트림에 parallel 메서드를 호출하면 기존의 함수형 리듀싱 연산(숫자 합계 계산)이 병렬로 처리된다.  
 
 public long parallelSum(long n){  
  return Stream.iterate(1L, i -> i+1) <- 무한 자연수 스트림 생성  
               .limit(n) <- n 개 이하로 제한  
               .parallel() <- <b>스트림을 병렬 스트림으로 변환</b>  
               .reduce(0L, Long::sum); <- 모든 숫자를 더하는 스트림 리듀싱 연산  
 }  
 
 reduce연산으로 스트림의 모든 숫자를 더하고, 이전 코드와 다르게 여러 청크로 스트림이 분할되어 있다.  
 따라서 리듀싱 연산을 여러 청크에 병렬로 수행할 수 있다.  
 
 ![7-1병렬리듀싱연산](https://user-images.githubusercontent.com/87962572/135857609-55627b17-ed53-4fd9-89e4-1d4986042c0c.PNG)

 마지막으로 리듀싱 연산으로 생성된 부분 결과를 다시 리듀싱 연산으로 합쳐서 전체 스트림의 리듀싱 결과를 도출한다.
 
 사실 순차 스트림에 parallel을 호출해도 스트림 자체에는 변화가 아무것도 일어나지 않지만,   
 <u>내부적으로 parallel을 호출하면 이후 연산이 병렬로 수행해야함을 의미하는 불리언 플래그가 설정된다.</u>  
 반대로 sequential 로 병렬 스트림을 순차 스트림으로 바꿀 수 있다.  
 
 ex) stream.parallel().filter(...)  
 .sequentail()  
 .map(...)  
 .parallel()  
 .reduce();
   
 parallel과 sequential 두 메서드 중 최종적으로 호출된 메서드가 전체 파이프라인에 영향을 미친다.
 이 예제에서도 파이프라인의 마지막 호출은 parallel이므로 파이프라인은 전체적으로 병렬로 실행된다.
 
 [병렬 스트림에서 사용하는 스레드 폴 설정]
 * 병렬 스트림은 내부적으로 ForkJoinPool을 사용한다. 기본적으로 ForkJoinPool은 프로세서 수, 즉 Runtime.getRuntime(), availableProcessors()가 반환하는 값에 
 * 상응하는 세레드를 가진다.
 * 일반적으로 기기의 프로세서 수와 같으므로 특별한 이유가 없다면 ForkJoinPool의 기본값을 그대로 사용할 것을 권장한다.

 <h3>7.1.2 스트림 성능 측정</h3>
 
 병렬화를 이용하면 순차나 반복 형식에 비해 성능이 더 좋아질 것이라 추측했다.  
 하지만 sw공학에서는 추측은 위험한 방법이고 "측정"을 통해서 확인해야된다.  
 자바 마이크로벤치카므 하니스(HMH)라는 라이브러리를 이용해 작은 벤치마크를 구현할 것이다.  
 이 JMH를 이용하면 간단하고 어노테이션 기반 방식을 지원하며, 안정적으로 자바 프로그램이나 자바 가상 머신을 대상으로 하는 다른 언어용 벤치마크를 구현할 수 있다.  
 
 pom.xml에 몇가지 의조성을 추가해 프로젝트에서 JMH를 사용할 수 있다.  
 
 ![7 1 2 dependency](https://user-images.githubusercontent.com/87962572/135859163-f9f14b97-5ee3-4b8e-a958-2c29174ff410.PNG)
 
 첫번째 라이브러리는 핵심 JMH 구현을 포함하고, 두번쨰는 자바 아카이브 JAR파일을 만드는데 도움을 준다.  

 또한 아래와 같은 설정을 통해 벤치마크를 편하게 실행할 수 있다.  
 
 ![예제7-1](https://user-images.githubusercontent.com/87962572/135859183-64be4990-4c75-4860-8779-60b41fc10866.PNG)


 예제7-1 n개의 숫자를 더하는 함수의 성능 측정  
 
 @BenchmrkMode(Mode.AverageTime)<- 벤치마크 대상 메서드를 실행하는 데 걸린 평균 시간 측정  
 @OutputTimeUnit(TimeUnit.MILLISECONDS)  <- 벤치마크 결과 밀리초 단위로 출력  
 @Fork(2, jvmArgs={"-Xms4G","-Xmx4G"})   <- 4Gb의 힙 공간을 제공한 환경에서 두 번 벤치마크를 수행해 결과의 신뢰성 확보  
 public class parallelStreamBenchmark{  
   private static final long N =10_000_000L;  
     
   @Benchmark <- 벤치마스 대상 메서드  
   public long sequentailSum(){   
    return Stream.iterator(1L, i->i+1).limit(N).reduce(0L, Long::sum);  
   }  
     
   @TearDown(Level.Invocation)  
   public void tearDown(){  
    System.gc();  
   }  
  }
  
  결과는 정확하지 않을 수 있음을 기억하자.   
  기계가 지원하는 코어의 갯수 등이 실행 시간에 영향을 미칠 수 있기 떄문이다.  
   
 
  병렬 버전이 더 느린 경우는 다음과 같은 문제를 확인해야된다.  
   - 반복 결과로 박싱된 객체가 만들어졌으므로 숫자를 더하려면 언박싱을 해야된다.
   - 반복 작업은 병렬로 수행할 수 있든 독립 단위로 나누기가 어렵다. -> <b>이는 이전 연산의 결과에 따라 다음 함수의 입력값이 달라지기 떄문에 iterate연산을 청크로 분할하기가 어렵다.</b>
   
   - 리듀싱 과정을 시작하는 시점에 전체 숫자 리스트가 준비되지 않았으므로 스트림을 병려로 처리할 수 있도록 청크로 분할할 수 없다.
   - 스트림이 병렬로 처리되도록 지시했고, 각각의 합계가 다른 스레드에서 수행되었지만 결국 순차처리 방식과 크게 다른점이 없으므로 스레드 할당에 오버헤드만 증가하게된다.
 
  <b>더 특화된 메서드 사용</b>
  멀티 코어 프로세서를 활용해서 합계 연산을 실행하려면 어떻게 해야할까?  
  이때에는 LongStream.rangeClosed라는 메서드를 사용하면 좋다.  
   - LongStream.rangeClosed는 기본형 long을 직접 사용하므로 박싱과 언박싱 오버헤드가 없어진다.
   - LongStream.rangeClosed는 쉽게 청크로 분할할 수 있는 숫자 범위를 생산한다.  
   
  LongStream.rangeClosed(1, N).parallel() // 스트림을 병렬스트림으로 변환  
  .reduce(0L, Long::sum);  
                 
  해당 예제코드는 순차 실행보다 빠른 성능을 갖는다.  

  결과적으로 병렬화를 위해서 스트림 분할, 서로 다른 스레드에서의 리듀싱 연산, 결과값 합치기 등의 오버헤드가 발생하기 때문에 최소한 멀티코어간의 데이터 이동 오버헤드보다   
  훨씬 오래 걸리는 작업을 처리할 때 효과를 볼 수 있다.  
   
  즉, 상황에 따라 병렬화를 사용하는 것이 좋지 않을 떄도 있다.  
  
 <h3>7.1.3 병렬 스트림의 올바른 사용법</h3>

 보통 병렬 스트림을 잘못 사용해서 발생하는 문제는 공유된 상태를 바꾸는 알고리즘을 사용하기 때문에 일어난다.  
  
 public long sideEffectSum(long n){  
  Accumulator acc = new Accumulator();  
  LongStream.rangeClosed(1, n).forEach(accumulator::add);  
  return acc.total;  
 }  
  
 public Class Accumulator{  
  public long total =0;  
  public void add(long val) { total += val; }  
 }  
 
 위의 코드의 문제점은, 본질적으로 순차 실행할 수 없도록 구현되어 있으므로 병렬로 실행하면 참사가 일어난다.  
 특히, total 을 접근할 때마다 데이터 레이스 문제가 일어난다.  
 동기화 문제를 해결하다보면 결국 병렬화라는 특성이 없어진다.  
   
 public long sideEffectParellelSum(long n){  
  Accumulator acc = new Accumulator();  
  LongStream.rangeClosed(1, n).parallel().forEach(accumulator::add);  
  return acc.total;  
 }  
 
 
 메서드 성능은 둘째 치고, 올바른 결과값이 나오지 않는다.  
 여러 스레드에서 동시에 누적자, 즉 total += value를 실행하면서 이런 문제가 발생한다.  
 <u>즉, 병렬 스트렘을 사용했을 때 이상한 결과가 나오지 않으려면 상태 공유에 따른 부작용을 피해야한다.</u>  
 
<h3>7.1.4 병렬 스트림 효과적으로 사용하기</h3> 
 - 확신이 서지 않으면 직접 측정하라. 무조건 병렬 스트림으로 바꾸는것이 능사가 아니다. 더욱이 병렬 스트림의 수행과정은 투명하지 않을 떄가 많다.  
 - 박싱을 주의하라. 자동 박싱과 언박싱은 성능을 크게 저하시킬 수 있는 요소이다.  
 - 순차 스트림보다 병렬 스트림에서 성능이 떨어지는 연산이 있다. 특히 limit나 findFirst 처럼 요소의 순서에 의존하는 연산을 병렬 스트림에서 수행하려면  
    비싼 비용을 치러야한다.   
 - 스트림에서 수행하는 전체 파이프라인 연산 비용을 고려하라. 처리해야되는 요소 수가 n이고, 하나의 요소를 처리하는 비용을 q라고 한다면 전체는 n* q 라고 전체 처리 비용을 산정할
    수 있다. q가 높아진다는 것은 병렬 스트림으로 성능 개선할 수 있는 가능성이 있음을 의미한다.  
 - 데이터에서는 병렬 스트림이 도움되지 않는다. 소량의 데이터를 처리하는 상황에서는 병렬화 과정에서 생기는 부가 비용을 상쇄할 수 있을만큼의 이득을 얻지 못하기 때문이다.  
 - 스트림을 구성하는 자료구조가 적적한지 확인하라.     
  ex) ArrayList, LinkedList중에서 LinkedList는 분할하려면 무조건 모든 요소를 탐색해야하기 때문에 비효율    
  반면, range팩토리 메서드로 만든 기본형 스트림은 분할이 쉽다.  
 - 스트림의 특성과 파이프라인의 중간 연산이 스트림의 특성을 어떻게 바꾸는지에 따라 분해 과정의 성능이 달라질 수 있다.    
 - 최종 연산의 병합 과정 비용을 살펴봐라. 병합 과정의 비용이 비싸다면 병렬 스트림으로 얻는 성능의 이익이 서브스트림의 부분 결과를 합치는 과정에서 상쇄될 수 있다.  


<h2>7.2 포크/조인 </h2>
 - 포크/조인 프레임워크는 병렬화할 수 있는 작업을 재귀적으로 작은 적업으로 분할한 다음에 서브태스트 각각의 결과를 합쳐서 전체 결과를 만들도록 설계되었다.
 - 서브태스크를 스레드 풀의 worker 스레드에 분산 할당하는 ExecutorService 인터페이스를 구현한다.

<h3>7.2.1 Recursive Task 활용 </h3>
 - 스레드 풀을 이용하려면 RecursiveTask<R>의 서브클래스를 만들어야 한다.   
 - 여기서 R은 병렬화된 태스크가 생성하는 결과 값 혹은 결과가 없을때 RecursiveAction 형식이다.   
   
 protected abstract R compute();  
 
 추상메서드 compute()는 태스크를 <u>서브태스크로 분할하는 로직과 더 분할할 수 없을때 개별 서브 테스크에서 돌아가는 로직을 구현해야 한다.</u>  
 
 예제 7-2 포크/조인 프레임워클이용해서 병렬 합계 수행  
 public class ForkJoinSumCalculator extends java.util.concurrent.RecursiveTask<Long> {   <- RecursiveTask를 상속받아 포크/조인 프레임워크에서 사용할 태스크를 생성
  private final long[] number;  
  private final int start; <- 이 서브태스크에서 처리할 배열의 초기 위치와 최종 위치  
  private final int end;  
  public static final long THRESHOLD = 10_000; <- 이 값 이하의 서브태스크는 더이상 분할 할 수 없다.,  
  
  public ForkJoinSumCalculator(long[] numbers){  
   this(numbers, 0, numbers.length);  
  }  
                                                  
  private ForkJoinSumCalculator(long[] numbers, int start, int end){  
   this.numbers = number;  
   this.start = start;  
   this.end = end;   
  }
                                                  
  @Override  
  protected Long compute(){          <- RecursiveTask의 추상 메서드 오버라이드
   int length = end- start;  
   if(length <=THERSHOLD){  
    return computeSequendtially();  
   }  
  ForkJoinSumCalculator leftTask = new ForkJoinSumCalculator(numbers, start, start+length/2);  
  leftTask.fork();  
  ForkJoinSumCalculator rightTask = new ForkJoinSumCalculator(numbers, start + length/2 , end);  <- 배열의 나머지 절반을 더하도록 서브태스트를 생성한다.  
  Long rightResult = rightTask.compute(); <- 두 번째 서브태스크를 동기 실행한다.  
  Long leftResult = leftTask.join(); <- 첫 번째 서브태스크의 결과 를 읽거나 아직 결과가 없으면 기다린다.   
  return leftResult+ rightResult;  
 
  }  
  private long computeSequentially(){ <- 더이상 분할이 불가능한 경우에 수앻나늘 로직  
   long sum= 0;  
   for(int i= start; i< end; i++){  
    sum += numbers[i];            
   }  
   return sum;  
  }  
   
 }  
                            
 위 메서드는 n까지의 자연수 덧셈 작업을 병렬로 수행하는 방법을 더 직관적으로 보여준다.  
 다음 코드처럼 ForkJoinSumCalculator의 생성자로 원하는 수의 배열을 넘겨줄 수 있다.  

 public static long forkJoinSum(long n){  
  long[] numbers = LongStream.rangeClosed(1,n).toArray();  
  ForkJoinTask<Long> task = new ForkJoinSumCalculator(numbers);  
  return new ForkJoinPool().invoke(task);  
 }  
 
 -> LongStream으로 n까지 자연수를 포함하는 배열을 생성하고, ForkJoinSumCalculator의 생성자로 전달해서  
 ForkJoinTask를 만들었다.  
 마지막으로 생성한 태스크를 새로운 ForkJoinPool의 invoke메서드로 전달했다.  
 
<h3>7.2.2 포크/조인 프레임워크를 제대로 사용하는 방법</h3>  
 - join메서드를 태스크에 호출하면 태스크가 생산하는 결과가 준비될 떄까지 호출자를 블록시킨다.  
  더 서브태스크가 모두 시작된 다음에 join을 호출해야된다. 그렇지 않으면 각 스버태스크가 다른 태스크가 끝나길 가디리는 일이 발생하여 순차 알고르짐보다 느리고 복잡하게 된다.  
 - RecursiveTask 내에서는 ForkJoinPool의 invoke 메서드를 사용하지 말아야된다. 대신 compute나 fork메서드를 직접 호출할 수 있다.  
 - 서브태스크에 fork메서드를 호출해서 ForkJoinPool의 일정을 조절할 수 있다.  
 한쪽 작업에는 fork을 호출 하는 것보다는 compute를 호출하는 것이 효율적이다.  
 - 포크/조인 프레임워크를 이용하는 병렬 계산 디버깅하기 어렵다.  
 - 병렬 스트림에서 순차 처리보다 무조건 빠를거라는 생각은 버려야된다.  
 
<h3>7.2.3 작업 훔치기 </h3>
 작업 훔치기 기법에서는 ForkJoinPool의 모든 스레드를 거의 공정하게 분할한다.  
 각각의 스레드는 자신에게 할당된 태스크를 포함하는 이중 연결 리스트를 참조하면서 작업이 끝날 때마다 큐의 헤드에서 다른 태스크를 가져와서 작업을 처리한다.  
 할일이 없어진 스레드는 다른 스레드 큐의 꼬리에서 작업을 훔쳐온다. 모든 태스크가 끝날 때 까지 이 과정을 반복한다.  
 
<h2>7.3. Spliterator 인터페이스 </h2>
  
자바 8 부터 제공되는 Spliterator 인터페이스는 Iterator와 소스의 요소 탐색 기능을 제공하는 점에서 같지만 병렬 작업에 특화되어 있다.  
 
public interface Spliterator<T> {  
  boolean tryAdvance(Consumer<? super T> action );  
  Spliterator<T> trySplit();  
  long estimateSize();  
  int characteristics();  
}  
여기서 T는 탐색하는 요소의 타입이다.
tryAdvance() : Spliterator의 요소를 순차적으로 소비하면서 더 탐색할 요소가 있으면 true 반환
trySplit() : Spliterator의 일부 요소를 분할해서 두 번째 Spliterator 생성

<h3>7.3.1 분할 과정 </h3>
재귀적으로 분할이 일어나고 trySplit()의 결과가 null이 되는 건 더 이상 분할이 불가능하다는 뜻이므로 모든 Spliterator의 trySplit()의 결과가 null이면 분할을 중지한다.  



 
