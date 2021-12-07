<h1>함수형 프로그래밍 기법</h1>

<h2>함수는 모든 곳에 존재한다.</h2>
함수형 프로그래밍이란 함수나 메서드가 수학적 함수처럼 동작함을, 즉 부작용 없이 동작함을 의미했다.<br/>
함수형 언어 프로그래머는 함수형 프로그래밍이라는 용어를 좀 더 폭넓게 사용한다.<br/>
함수를 마치 일반값처럼 사용해서 인수로 전달하거나, 결과로 반환하거나, 자교 구조에 저장할 수 있음을 의미한다.<br/>
일반값처럼 취급할 수 있는 함수를 일급 함수라고 한다.<br/>
자바 8에서는 :: 연산자로 메서드 참조를 만들거나 (int x) -> x+1 같은 람다 표현식으로 직접 함숫값을 표현해서 메서드를 함숫값으로 사용할 수 있다.<br/>

```
Function<String, Integer> strToInt = Integer::parseInt;
```

<h2>고차원 함수</h2>
지금까지 filterApples 에 함숫값으로 Apple::isGreenApple을 전달해서 동작 파라미터화를 달성하는 용도로만 사용했다.<br/>
이는 함숫값 활용의 일부에 불과하다. 다음 코드 그리고 아래 그림이 보여주는 것처럼 함수를 인수로 받아서 다른 함수로 반환하는 정적 메서드 Comparator.comparing도 있었다.<br/>

![191](https://user-images.githubusercontent.com/87962572/144861396-d3a6899f-8436-4372-924c-b250d40d12ef.PNG)

```
Comparator<Apple> c = comparing(Apple::getWeight);

//3장에서는 함수를 조립해서 연산을 파이프라인을 만들 때 위 코드와 비슷한 기능을 활용했다.
Function<String, String> transformationPipeline = addHeader.andThen(Letter::checkSpelling).andThen(Letter::addFooter);
```

함수형 프로그래밍 커뮤니케이션에 따르면 Comparator.comparing처럼 다음 중 하나 이상의 동작을 수행하는 함수를 고차원 함수라 부른다.
- 하나 이상의 함수를 인수로 받음
- 함수를 결과로 반환

자바 8에서는 함수를 인수로 전달할 수 있을 뿐 아니라 결과를 반환하고, 지역 변수로 할당하거나, 구조체를 삽입할 수 있으므로 자바 8의 함수도 고차원 함수라고 할 수 있다.<br/>

스트림 연산으로 전달하는 함수는 부작용이 없어야 하며, 부작용을 포함하는 함수를 사용하면 문제가 발생한다는 사실을 설명했따.(부작용을 포함하는 함수를 사용하면 부정확한 결과가 발생하거나 레이스 컨디션 떄문에 예상치 못한 결과라 발생할 수 있다.) . <br/>
고차원 함수를 적용할 떄도 같은 규칙을 적용된다.<br/>
고차원 함수나 메서드를 구현할 때 어떤 인수가 전다될 지 알 수 없으므로 인수가 부작용을 포함한 가능성을 염두에 두어야 한다!<br/>
함수를 인수로 받아 사용하면서 코드가 정확히 어떤 작업을 수행학 프로그램의 상태를 어떻게 바꿀지 에측하기 어려워진다.<br/>
디버깅도 어려워진다.<br/>
따라서 인수로 전달된 함수가 어떤 부작용을 포함하게 될지 정확하게 문서화하는 것이 좋다.<br/>
물론 부작용을 포함하지 않을 수 있다면 가장 좋을 것이다!<br/>

<h3>커링</h3>
함수를 모듈화하고 코드를 재사용하는 데 도움을 주는 기법인 커링을 살펴보자.<br/>
커링이 무엇인지 살펴보기 앞서 예제를 확인하자.<br/>
대부분의 애플리케이션은 국제화를 지원해야 하는데 이때 단위 변환 문제가 발생할 수 있다.<br/>

보통 변환 요소와 기준치 조정 요소가 단위 변환 결과를 좌우한다.<br/>
예를 들어 다음은 섭씨를 화씨로 변환하는 공식이다.<br/>

CtoF(x) = x * 9 /5 +32

다음과 같은 패턴으로 단우를 표현할 수 있다.

1. 변환 요소를 곱함.
2. 기준치 조정 요소를 적용

다음과 같은 메서드로 변환 패텬을 적용할 수 있다.

```
static double converter(double x, double f, double b)}
  return x * f + b;
}
```

여기에서 x는 변환하려는 값이고, f는 변환 요소며, b는 기준치 조정 요소다.<br/>
세 개의 인수를 받는 converter라는 메서드를 만들어 문제를 해결하는 방법도 있지만, 인수에 변환 요소와 기준치를 넣는 일은 귀찮은 일이며 오타도 발생하기 쉽다.<br/>

각각을 변환하는 메서드를 따로 만드는 방법도 있지만 그러면 로직을 재활용하지 못한다는 단점이 있다.<br/>
기존 로직을 활용해서 변환기를 특정 상황에 적용할 수 있는 방법이 있다. 다음은 커링이라는 개념을 활용해서 한 개의 인수를 갖는 변환 함수를 생성하는 팩토리를 정의하는 코드다.<br/>

```
static DoubleUnaryOperator curriedConvertor(double f, double b){
  return (double x) -> x * f + b;
}
```

메서드에 변환 요소(f)와 기준치(b)만 넘겨주면 우리가 원하는 작업을 수행할 함수가 반환된다.<br/>
예를 들어 다음은 팩토리르 이용해서 원하는 변환기를 생성하는 코드다.<br/>

```
DoubleUnaryOperator convertCtoF = curriedConverter(9.0/5, 32);
DoubleUnaryOperator convertUSDtoGBP = curriedConverter(0.6, 0);
DoubleUnaryOperator convertKmtoMi = curriedConverter(0.6214, 0);
```

DoubleUnaryOperator는 applyAsDouble이라는 메서드를 정의하므로 다음처럼 변환기를 사용할 수 있다.

```
double gbp = convertUSDtoGBP.applyAsDouble(1000);
```

결과적으로 기존의 변환 로직을 재활용하는 유연하 코드를 얻었다!<br/>
우리가 어떤 작업을 햇는지 다시 생각햅자. x, f, b라는 세 인수를 converter 메서드로 전달하지 않고 f,b 두 가지 인수로 함수를 요청했으며 반환된 함수를 인수 x를 이용해서<br/>
x * f + b라는 결과를 얻었다.<br/>
이런 방식으로 변환 로직을 재활용할 수 있으며 다양한 변환 요소로 다양한 함수를 만들 수 있다.<br/>

<h3>커링의 이론적 정의</h3>
커링은 x와 y라는 두 인수를 받는 함수 f를 함 개의 인수를 받는 g라는 함수로 대체하는 기법이다.<br/>
이때 g라는 함수 역시 하나의 인수를 받는 함수를 반환한다.<br/>
함수 g와 원래 함수 f가 최종적으로 반환하는 값은 같다.<br/>
즉, f(x,y) = (g(x))(y) 가 성립한다.<br/>

물론 이 과정을 일반화할 수 있다. 여섯 개의 인수를 가진 함수를 커리해서 우선 2, 4, 6번째 인수를 받아 5번째 인수를 받는 함수를 반환하고 다시 이 함수는 남은 1, 3번째 인수를 받는
함수를 반환한다.

이와 같은 여러 과정이 끝까지 완료되지 않은 상태를 가리켜 '함수가 부분적으로 적용되었다.'라고 말한다.

<h2>영속 자료구조</h2>
함수형 프로그램에서는 함수형 자료구조, 불변 자료구조 등의 용어도 사용하지만 보통은 영속 자료구조라고 부른다.
함수형 메서드에서는 전역 자료구조나 인수로 전달된 구조를 갱신할 수 없다.
자료구조를 바꾼다면 같은 메서드를 두 번 호출했을 때 결과가 달라지면서 참조 투명성에 위배되고 인수를 결과로 단순하게 매핑할 수 있는 능력이 상실되기 떄문이다.

<h3>파괴적인 갱신과 함수형</h3>
자료구조를 갱신할 때 발생할 수 있는 문제를 확인해보자.
A에서 B가지 기차여행을 의미하는 가변 TrainJourney 클래스가 있다고 가정하자.
TrainJourney는 간단한 단방향 연결 리스트로 구현되며 여행 구간의 가격 등 상세 정보를 포함하는 int필드를 포함한다.

다음에서 보여주는 것처럼 기차여행에서는 여러 TrainJourney 객체를 연결할 수 있는 onward(이어지는 여정을 의미)라는 필드가 필요하다.
직통 열차나 여정의 마지막 구간에서는 onward가 null이 된다.

```
class TrainJourney {
 public int price;
 public TrainJourney onward;
 public TrainJourney(int p, TrainJourney t) {
  price = p;
  onward = t;
 }
}
```

이때 X에서 Y까지 그리고 Y에서 Z까지으 여행을 나타내는 별도의 TrainJourney 객체가 있다고 가정하자.
두 개의 TrainJourney 객체를 연결해서 하나의 여행을 만들 수 있을 것이다. (즉, X에서 Y 그리고 Z까지 이어지는 여행)

기존의 단순한 ㅕㅇ령행 메서드라면 다음처럼 기차여행을 연결할 수 있다.

```
static TrainJourney link(TrainJourney a, TrainJourney b){ 
  if (a==null) return b;
  TrainJourney t = a;
  while(t.onward != null){
    t = t.onward;
  }
 
  t.onward = b;  ◀ a의 TranJourney에서 마지막 여정을 찾아 a의 리스트 끝부분을 가리키는 null을 리스트 b로 대체한다.
  return a;
}
```

firstJourney = X에서 Y로의 경로이고 secondJourney = Y에서 Z로의 경로일 때 link(firstJourney, secondJourney)를 호출하면 firstJourney가 secondJourney를 포함하면서 파괴적인
갱신(즉, firstJourney를 변경시킴)이 일어난다.
결과적으로 firstJourney 변수는 X에서 Y로의 여정이 아니라 X에서 Z로의 여정을 의미하게 된다!
이렇게 되면 firstJourney에 의존하는 코드가 동작하지 않게 된다!

함수형에서는 이 같은 부작용을 수반하는 메서드를 제한하는 방식으로 문제를 해결한다. 계산 결과를 표현할 자료구조가 필요하면 기존의 자료구조를 갱신하지 않도록 새로운 자료구조를 
만들어야 한다.

```
static TrainJourney append(TrainJourney a, TrainJourney b){ 
  return a==null ? b : new TrainJourney(a.price, append(a.onward, b)); 
}
```

이 코드는 명확하게 함수형이며 기존 자료구조를 변경하지 않느다. 하지만 TrainJourney 전체를 새로 만들지 않는다.
a가 n요소의 시퀀스고 b가 m요소의 시퀀스라면 n+m 요소의 시퀀스를 반환한다.
여기서 n요소는 새로운 노드며 마지막 m 요소는 TrainJourney b와 공유되는 요소다.
주의할 점은 사용자 역시 append의 결과를 갱신하지 말아야 한다는 것이다.
만약 append의 결과를 갱신하면 시퀀스 b로 전달된 기차 정보도 바뀐다.

![1902](https://user-images.githubusercontent.com/87962572/145024931-68ef58d4-b10f-4144-b220-8ee572faeb01.PNG)

위에는 파괴적인 append이고 아래는 함수형 append의 차이점을 보여준다.

<h3>트리를 사용한 다른 예제</h3>
HashMap 같은 인터페이스를 구현할 떄는 이진 탐색 트리가 사용된다.
Tree는 문자열 키와 int값을 포함한다. 

ex) 이름 키와 나이 정보값을 포함할 수 있다.

```
class Tree {
  private String key;
  private int val;
  private Tree left, right;
  public Tree(String k, int v, Tree l, Tree r) {
    key = k; val = v; left = l; right = r;
  }
}
class TreeProcessor {
  public static int lookup(String k, int defaultval, Tree t) {
    if (t == null) return defaultval;
    if (k.equals(t.key)) return t.val;
    return lookup(k, defaultval, k.compareTo(t.key) < 0 ? t.left : t.right);
  }
  // other methods processing a Tree 
}
```

이제 이진 탐색 트리를 이용해서 문자열값으로 int를 얻을 수 있다. 주어진 키와 연관된 값을 어떻게 갱신할 수 있을지 살펴보자.

```
public static void update(String k, int newval, Tree t) {
 if (t == null) { /* 새로운 노드를 추가해야 함 */ }
 else if (k.equals(t.key)) t.val = newval;
 else update(k, newval, k.compareTo(t.key) < 0 ? t.left : t.right);
}
```

새로운 노드를 추가할 수 있는 가장 쉬운 방법은 update메서드가 탐색한 트리를 그대로 반환하는 것이다.
하지만 이 방법은 사용자가 update가 즉석에서 트리를 갱신할 수 있으며, 전달한 트리가 그대로 반환되고, 원래 트리가 비어있으면 새로운 노드가 반환될 수 있다는 사실을
모두 기억해야 한다는 점에서 그리 깔끔하지는 않는다.

```
public static Tree update(String k, int newval, Tree t) {
  if (t == null)
    t = new Tree(k, newval, null, null);
  else if (k.equals(t.key))
    t.val = newval;
  else if (k.compareTo(t.key) < 0)
    t.left = update(k, newval, t.left);
  else
    t.right = update(k, newval, t.right);
  return t;
}
```

두 가지 update 버전 모두 기존 트리를 변경한다. 즉, 트리에 저장된 맵의 모든 사용자가 변경에 영향을 받는다.

<h3>함수형 접근법 사용</h3>
우선 새로운 키/값 쌍을 저장할 새로운 노드를 만들어야 한다. 또 트리으 루트에서 새로 생성한 노드의 경로에 있는 노드들도 새로만들어야 한다.

```
// if-then-else 대신 하나의 조건문을 사용했는데 이렇게 해서 위 코드가 부작용이 없는 하나의 표현식임을 강조했다.
public static Tree fupdate(String k, int newval, Tree t) {
  return (t == null) ? 
    new Tree(k, newval, null, null) : k.equals(t.key) ?
      new Tree(k, newval, t.left, t.right) : k.compareTo(t.key) < 0 ?
        new Tree(t.key, t.val, fupdate(k,newval, t.left), t.right) :
          new Tree(t.key, t.val, t.left, fupdate(k,newval, t.right));
}
```

update와 fupdate의 차이는 뭘까?
이전에 update 메서드는 모든 사용자가 같은 자효구조를 공유하며 프로그램에서 누군가 자료구조를 갱신했을 때 영향을 받는다는 점을 설명했다.
따라서 비함수형 코드에서는 누군가 언제든 트리를 갱신할 수 있으므로 트리에 어떤 구조체의 값을 추가할 떄마다 값을 복사했다.
반면 fupdate는 순수한 함수형이다. 
fupdate에서는 새로운 Tree를 만든다. 하지만 인수를 이용해서 가능한 많은 정보를 공유한다.
여기서 fupdate를 호출하면 기존의 트리를 갱신하는 것이 아니라 새로운 노드를 만든다.

이와 같은 함수형 자료구조를 영속이라고 하며 따라서 프로그래머는 fupdate가 인수로 전달된 자료구조를 변화시키지 않을 것이라는 사실을 확실할 수 있다.
결과 자료구조를 바꾸지 말라는 것이 자료구조를 사용하는 모든 사용자에게 요구하는 단 한가지 조건이다.
결과 자료구조를 바꾸지 말라는 조건을 무시한다면 fupdate로 전달된 자료구조에 의도치 않은 그리고 언치 않는 갱신이 일어난다.

![19-4](https://user-images.githubusercontent.com/87962572/145027172-97e033c5-0f3f-44ac-bcee-65e0ec1b8d7f.PNG)

fupdate가 효율적일 떄가 더 많다.

<h2>스트림과 게으른 평가</h2>
<h3>자기 정의 스트림</h3>
ex) 소수 스트림을 계산

```
public static Stream<Integer> primes(int n) {
 return Stream.iterate(2, i -> i + 1)
 .filter(MyMathUtils::isPrime)
 .limit(n);
}

public static boolean isPrime(int candidate) {
 int candidateRoot = (int) Math.sqrt((double) candidate); 
 
 return IntStream.rangeClosed(2, candidateRoot).noneMatch(i -> candidate % i == 0);
}
```

이론적으로 소수로 나눌 수 있는 모든 수는 제외할 수 있다.
1. 소수를 선택할 숫자 스트림이 필요하다.
2. 스트림에서 첫 번째 수를 가져온다. 이 숫자는 소수다.
3. 이제 스트림의 꼬리에서 가져온 수로 나누어떨어지는 모든 수를 걸러 제외시킨다.
4. 이렇게 남은 숫자만 포함하는 새로운 스트림에서 소수를 찾는다. 이제 1번부타 이 과정을 반복하게 된다. 따라서 이 알고리즘은 재귀다.


<h3>1 단계 : 스트림 숫자 얻기</h3>

```
//2에서 시작하는 무한 숫자 스트림을 생성할 수 있다.
static Intstream numbers(){
  return IntStream.iterate(2, n -> n + 1);
}
```

<h3>2 단계 : 머리 획득</h3>

```
static int head(IntStream numbers){
  return numbers.findFirst().getAsInt();
}
```

<h3>3 단계 : 꼬리 필터링</h3>

```
static IntStream tail(IntStream numbers){
 return numbers.skip(1);
}

// 획득한 머리로 숫자를 필터링할 수 있다.
IntStream numbers = numbers();
int head = head(numbers);
IntStream filtered = tail(numbers).filter(n -> n % head != 0);
```

<h3>4 단계 : 재귀적으로 소수 스트림 생성</h3>
다음 코드에서 보여주는 것처럼 반복적을 머리를 얻어서 스트림을 필터링하려 할 수 있다.

```
static IntStream primes(IntStream numbers) {
  int head = head(numbers);
    return IntStream.concat(
      IntStream.of(head), 
      primes(tail(numbers).filter(n -> n % head != 0))
  );
}
```

4단게를 실행하면 java.lang.IllegalStateException: stream has already been operated upon or closed 라는 에러가 발생한다.
사실 우리는 스트림을 머리와 꼬리로 분리하는 두 개의 최종 연산 findFirst와 skip을 사용했다.
4장에서는 최종 연산을 스트림에 호출하면 스트림이 완전 소비된다는 사실을 설명했다!

<h3>게으른 평가</h3>
위 나쁜 소식보다 더 심각한 문제가 있다.
정적 메서드는 IntStream.concat은 두 개의 인스턴스를 인수로 받는다. 두 번째 인수가 primes를 직접 재귀적으로 호출하면서 무한 재귀에 빠진다!
재귀적 정의 허용하지 않음같은 자바 8의 스트림 규칙은 우리에게 아무 해도 끼치지 않으며 오히려 이 규칙 덕분에 데이터베이스 같은 질의를 표현하고 병렬화할 수 있는 능력을 얻을 수 있다.
결론적으로 concat의 두번째 인수에서 primes를 게으르게 평가하는 방식으로 문제를 해결할 수 있다.
이를 게으른 평가, 비엄격한 평가 또는 이름에 의한 호출이라고 부른다.

<h3>게으른 리스트 만들기</h3>
자바 8의 스트림은 요청할 떄만 값을 생성하는 블랙박스와 같다.
스트림에 일련의 연산을 적용하면 연산이 수행되지 않고 일단 저장된다.
스트림에 최종 연산을 적용해서 실제 계산을 해야하는 상황에서만 실제 연산이 이루어진다.
특히 스트림에 여러 연산을 적용할 때 이와 같은 특성을 활용할 수 있다.

일반적인 스트림의 형태인 게으른 리스트의 개념을 살펴본다.
또한 게으른 리스트는 고차원 함수라는 개념도 지원한다.
함숫값을 자료구조에 저장해서 함숫값을 사용하지 않은 상태로 보관할 수 있다.
하지만 저장한 함숫값을 호출하면 더 많은 자료구조를 만들 수 있다.


![19-5](https://user-images.githubusercontent.com/87962572/145029573-e45522f8-8fc5-4a85-83f2-6645c21e39b7.PNG)

기본적인 연결 리스트는 MyLinkedList라는 연결 리스트 형태의 클래스를 정의할 수 있다는 것을 기억하라.

```
interface MyList<T> {
  T head();
  MyList<T> tail();
  default boolean isEmpty() { 
    return true; 
  }
}

class MyLinkedList<T> implements MyList<T> {
  private final T head;
  private final MyList<T> tail;
  public MyLinkedList(T head, MyList<T> tail) {
    this.head = head;
    this.tail = tail;
  }
  public T head() {
    return head;
  }
  public MyList<T> tail() {
    return tail;
  }
  public boolean isEmpty() {
    return false;
  }
}

class Empty<T> implements MyList<T> { 
  public T head() { 
    throw new UnsupportedOperationException();
  }
  public MyList<T> tail() {
    throw new UnsupportedOperationException(); 
  }
}

MyList<Integer> l = new MyLinkedList<>(5, new MyLinkedList<>(10, new Empty<>()));
```

기본적인 게으른 리스트
Supplier< T > 를 void -> T 라는 함수형 디스크립터를 가진 팩토리로 생걱할 수 있다.
Supplier < T >로 리스트의 다음 노드를 생성할 것이다.

```
import java.util.function.Supplier;
class LazyList<T> implements MyList<T>{
  final T head;
  final Supplier<MyList<T>> tail;
  public LazyList(T head, Supplier<MyList<T>> tail) {
    this.head = head;
    this.tail = tail;
  }
  public T head() {
    return head;
  }
  public MyList<T> tail() {
    return tail.get(); 
  }
  public boolean isEmpty() {
    return false;
  }
}

// 이제 Supplier의 get의 메서드를 호출하면 LazyList의 노드가 만들어진다.

// tail인수로 Supplier를 전달하는 방식으로 n으로 시작하는 무한히 게으른 리스트를 만들 수 있다.

public static LazyList<Integer> from(int n) {
  return new LazyList<Integer>(n, () -> from(n+1));
}

// 만약 요청했을 때 코드가 실행되는 것이 아니라 2부터 시작해서 모든 수를 미리 계산하려 한다면 프로그램은 영원히 종료되지 않을 것이다.

LazyList<Integer> numbers = from(2);
int two = numbers.head();
int three = numbers.tail().head();
int four = numbers.tail().tail().head();
System.out.println(two + " " + three + " " + four);

```

<h3>소수 생성으로 돌아와서</h3>

```
public static MyList<Integer> primes(MyList<Integer> numbers) {
  return new LazyList<>(
    numbers.head(), 
      () -> primes(
        numbers.tail()
        .filter(n -> n % numbers.head() != 0)
        )
    );
}

// 게으른 필터 구현
public MyList<T> filter(Predicate<T> p) {
  return isEmpty() ?
  this : p.test(head()) ? new LazyList<>(head(), () -> tail().filter(p)) : tail().filter(p);
}
```

<h2>패턴 매칭</h2>
일반적으로 함수형 프로그래밍을 구분하는 또 하나의 중요한 특징으로 패턴 매칭을 들 수 있다.
이는 정규표현식과 관련된 매칭과는 다르다. 
여기서의 패턴 매칭은 한 개 이상의 파라미터에 대한 멀티 매칭을 의미한다. 
자바에서는 패턴 매칭 대신 if-then-else 구문을 중첩하여 구현할 수 있지만, 스칼라에서는 이미 패턴 매칭을 지원하고 있다. 
쉽게 표현하면 다수준(multilevel)의 switch 문이라고 설명할 수 있다. 아래 예제를 통해 패턴 매칭이 무엇인지 이해할 수 있다.

```
Expr simplifyExpression(Expr expr) {
 if (expr instanceof BinOp
 && ((BinOp)expr).opname.equals("+"))
 && ((BinOp)expr).right instanceof Number
 && ... // it's all getting very clumsy
 && ... ) {
 return (Binop)expr.left;
 }
 ... 
}

```

<h3>캐싱 또는 기억화</h3>
트리 형식의 토포로지를 갖는 네으퉈크 범위 내에 존재하는 노드의 수를 게산하는 computeNumberOfNodes(Range)라는 부작용 없는 메서드가 있다고 하자.
computeNumberOfNodes를 호출했을 때 구조체를 재귀적으로 탐색해야하므로 노드 계산 비용이 비싸다. 게다가 이와 같은 계산을 반복해서 수행해야 할 것 같다.
이때 참조 투명성이 유지되는 상황이라면 간단하게 추가 오버헤드를 피할 수 있는 ㄴ방법이 생긴다.
표준적인 해결책으로 기억화라는 기법이 있다.
기억화는 메서드에 래퍼로 캐시를 추가하는 기법이다.
래퍼가 호출되면 인수, 결과 쌍이 캐시에 존재하는지 먼저 확인한다.
캐시에 값이 존재하지 않으면 computeNumberOfNodes를 호출해서 결과를 계산한 다음에 새로운 인수, 결과 쌍을 캐시에 저장하고 결과를 반환한다.
엄밀히 따져서 캐싱, 즉 다수의 호출자가 공유하는 자료구조를 갱신하는 기법이므로 이는 순수 함수형 해결방식은 아니지만 감싼 버전의 코드는 참조 투명성을 유지할 수 있다.

<h3>콤비네이터</h3>
함수형 프로그래밍에선 함수를 인자로 받고 조합하며 결과로 함수를 반환하는 형식의 고차원 함수를 많이 사용하게 된다. 이처럼 함수를 조합하는 기능을 콤비네이터라고 부른다.\
ex) CompletableFuture클래스에는 thenCombine이라는 메서드가 추가되었다.
