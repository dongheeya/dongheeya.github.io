<h1>OOP와 FP의 조화 : 자바와 스칼라 비교</h1>
스칼라는 객체지향과 함수형 프로그래밍을 혼합한 언어다.<br/>
정적 형식의 프로그래밍 언어로 함수형의 기능을 수행하면서도 JVM에서 수행되는 언어이므로 자바 느낌을 원하는 프로그래머가 찾는다.<br/>
스칼라는 자바에 비해 많은 기능을 제공한다. <br/>
스칼라는 복잡한 형식 시스템, 형식 추론, 패턴 매칭, 도메인 전용 언어를 단순하게 저의할 수 있는 구조 등을 제공한다<br/>
스칼라 코드에서는 모든 자바 라이브러리를 사용할 수 있다.<br/>
스칼라를 이용하면 자바에 비해 더 간결하고 가독성이 좋은 코드를 구현할 수 있다는 사실을 발견할 수 있다.<br/>

<h2>스칼라 소개</h2>
<h3>hello beer</h3>
스칼라 문법과 기능을 자바와 비교해보자.<br/>
고전의 'Hello World' 예제를 맥주로 바꿨다.<br/>

```
Hello 2 bottles of beer
Hello 3 bottles of beer
Hello 4 bottles of beer
Hello 5 bottles of beer
Hello 6 bottles of beer

// 명령형으로 위 문장을 출력하는 스칼라 프로그램이다.
object Beer {
  def main(args: Array[String]){
    var n : Int = 2
    while( n <= 6){
      println(s"Hello ${n} bottles of beer")   <- 문자열 보간법
      n += 1
    }
  }
}
```

Object를 선언하는 과정을 살펴보자.<br/>
자바에서는 클래스 내에 main 메서드를 선언했다.<br/>
스칼라에서는 object로 직접 싱글턴 객체를 만들 수 있다. 위 예제에서는 object를 Beer 클래스를 정의하고 동시에 인스턴스화했다.<br/>
한 번에 단 하나의인스턴스만 생성된다.<br/>
object 내부에 선언된 메서드는 정적 메서드로 간주할 수 있다.<br/>
바로 main 메서드의 시그니처에 명시된 static이 없는 것도 이것 떄문이다.<br/>

main바디는 구문이 세미클론으로 끝나지 않는다.(선택사항)<br/>
while안에서 가변변수 n을 증가시키고, n값이 바뀔 떄마다 println으로 화면에 n값을 출력한다.<br/> 
이러한 스칼라의 문자열 바간법은 문자열 자체에 변수와 표현식을 바로 삽입하는 기능이다.<br/>
위 코드에서 s"Hello ${n} bottles of beer"이라는 문자열에 변수 n을 직접 사용했다.<br/>
문자열에 접두어 s를 붙이면 이러한 마법 같은 일이 일어난다.<br/>

<h3>함수형 스칼라</h3>

```
public class Foo {
  public static void main(String[] args) {
    IntStream.rangeClosed(2, 6).forEach(n -> System.out.println("Hello " + n + " bottles of beer"));
  }
}

//스칼라는 자바보다 더 간결하게 구현이 가능하다.

object Beer {
  def main(args: Array[String]){
    2 to 6 foreach { n => println(s"Hello ${n} bottles of beer") } // 스칼라는 2to 6이라는 표현식으로 숫자 범위를 만들 수 있다.
  }
}

```

스칼라에는 기본형이 없다.<br/>
스칼라는 자바보다 완전한 객체지향 언어다.<br/>
스칼라의 Int 객체는 다른 Int를 인수로 받아 범위를 반환하는 to라는 메서드를 지원한다.<br/>
따라서 2.to(6)이라고 구현할 수도 있다.<br/>
하지만 인수 하나를 받는 메서드는 인픽스 형식으로 구현할 수 있다.<br/>
람다 표현식 문법은 자바와 비슷하지만 다만 -> 대신 => 를 사용한다.<br/>

<h3>컬렉션 만들기</h3>

```
//자바
Map<String, Integer> authorsToAge = new HashMap<>();
authorsToAge.put("Raoul", 23);
authorsToAge.put("Mario", 40);
authorsToAge.put("Alan", 53);

//스칼라
val authorsToAge = Map("Raoul" -> 23, "Mario" -> 40, "Alan" -> 53)
```

우선 -> 이라는 문법으로 키를 값에 대응시켜 맵을 만들수 있다.

```
//자바
Map<String, Integer> authorsToAge = Map.ofEntries(entry("Raoul", 23), entry("Mario", 40), entry("Alan", 53));

//스칼라
val authors = List("Raoul", "Mario", "Alan")

val numbers = Set(1, 1, 2, 3, 5, 8)
```

authorsToAge의 형식을 지정하지 않는다. 스칼라는 자동으로 변수형을 추론하는 기능이 있다.<br/>
var 대신 val이라는 키워드를 사용했다.<br/>
val은 변수가 읽기 전용, 즉 변수에 값을 할당할 수 없음을 의미한다.<br/>
var은 키워드를 읽고 쓸 수 있는 변수를 가리킨다.<br/>

<h3>불변과 가변</h3>

지금까지 만든 컬렉션은 기본적으로 불변이라는 점을 기억하자.<br/>
일단 컬렉션을 만들면 변경할 수 없다.<br/>
컬렉션이 불변이므로 프로그램에서 언제 컬렉션을 사용하든 항상 같은 요소를 갖게 되고 함수형 프로그램에서 유용하게 활용할 수 있다.<br/>

스칼라의 불변 컬렉션을 갱신할 때는 가능한 많은 자료를 공유하는 새로운 컬렉션을 만드는 방법으로 자료구조를 갱신한다.<br/>
결과적으로 암묵적인 데이터 의존성을 줄일 수 있다.<br/>
즉, 언제, 어디서 컬렉션을 갱신했는지 크게 신경 쓰지 않아도 된다.<br/>

```
val numbers = Set(2, 5, 3);
val newNumbers = numbers + 8 <- 여기에서 +는 집합에 8을 더하는 메서드의 연산 결과로 새로운 Set객체를 생성한다.
println(newNumbers)  <- (2, 5, 3, 8)    
println(numbers)     <- (2, 5, 3)
```

숫자 집합은 바뀌지 않는다. 대신 새로운 요소가 추가된 새로운 집합이 생성된다.

<h3>컬렉션 사용하기</h3>

```
val fileLines = Source.fromFile("data.txt").getLines.toList()
val linesLongUpper = fileLines.filter(l => l.length() > 10).map(l => l.toUpperCase())

// 첫 번째 행은 기본적으로 파일의 모든 행을 문자열 행으로 변환한다.
// 두번쨰 행은 두 연산의 파이프라인을 생성한다.

val linesLongUpper = fileLines filter (_.length() > 10) map(_.toUpperCase())

//위 코드는 인픽스 개념과 언더스코어( _ )를 사용했다. 언더스코어는 인수로 대치된다.
//즉, 이 코드에서 _.length()는 1 => 1.length()로 해석할 수 있다.
//filter와 map으로 전달된 함수에서는 언더스코어는 처리되는 행으로 바운드된다.
```

![20-1](https://user-images.githubusercontent.com/87962572/146003759-732a20a9-ebd8-41d1-bffb-eed9544b0e19.PNG)

<h3>튜플</h3>

ex) 사람의 이름과 전화번호를 그룹화하는 튜플을 만들려고 한다.<br/>
새로 클래스를 만들고 객체로 인스턴스화하지 않고 ("Raoul","+ 44 007007007), ("Alan", "+44 003133700") 같은 식으로 바로 튜플을 만들 수 있다면 좋을 것같다.<br/>
자바에서는 튜플을 제공하지 않는다. 따라서 직접 자료구조를 만들어야한다.<br/>

```
public class Pair<X, Y> {
 public final X x;
 public final Y y;
 public Pair(X x, Y y){
 this.x = x;
 this.y = y;
 }
}

// 그리고 명시적으로 클래스를 인스턴스화 해야한다.
Pair<String, String> raoul = new Pair<>("Raoul", "+44 7700 700042");
Pair<String, String> alan = new Pair<>("Alan", "+44 7700 700314");
```

이때의 단점은 만약에 세 개의 요소를 그룹화하는 튜플이 필요한 경우에는 사용할 수 없다는 점이다<br/>
즉, 프로그램의 가독성과 유지보수성을 떨어뜨린다.<br/>

스칼라는 튜플 축약어 즉, 간단한 문법으로 튜플을 만들 수 있는 기능을 제공한다.<br/>

```
val raoul = ("Raoul", "+44 7700 700042")
val alan = ("Alan", "+44 7700 700314")

val book = (2018 "Modern Java in Action", "Manning") <- (Int, String, String)형식의 튜플
val numbers = (42, 1337, 0, 3, 14)                  <- (Int, Int, Int, Int, Int) 형식의 튜플

println(book._1)  <- 2018을 출력 
println(numbers._4) <- 3을 출력
```

<h3>스트림</h3>

자바의 스트림은 요청할 때만 평가되었다.(게으른 평가)<br/>
이러한 특성 떄문에 메모리 오버플로 없이 무한 시퀀스를 표현할 수 있음을 확인했다.<br/>

스칼라에서도 스트림이라는 게으르게 평가되는 자료구조를 제공한다.<br/>
스칼라의 스트림은 자바의 스트림보다 다양한 기능을 제공한다.<br/>
스칼라의 스트림은 이전 요소가 접근할 수 있도록 기존 계산법을 기억한다.<br/>
또한 인덱스를 제공하므로 리스트처럼 인덱스로 스트림의 요소에 접근할 수 있다.<br/>
이러한 기능이 추가되면서 스칼라의 스트림은 자바의 스트림에 비해 메모리 효율성이 조금 떨어진다.<br/>
이전 요소를 참조하려면 요소를 '기억(캐시)'해야되기 떄문이다.<br/>

<h3>옵션</h3>
자바의 Optional과 같은 기능을 제공한다.
즉, 사용자는 메서드의 시그니처만 보고도 Optional값이 반환될 수 있는지 여부를 알 수 있다.
null 대신 Optional을 사용하면 null 포인터 예외를 방지할 수 있다.

ex) 사람의나이가 최소 나이보다 클 때 보험회사 이름을 반환하는 코드

```
public String getCarInsuranceName(Optional<Person> person, int minAge) {
  return person.filter(p -> p.getAge() >= minAge)
   .flatMap(Person::getCar)
   .flatMap(Car::getInsurance)
   .map(Insurance::getName)
   .orElse("Unknown");
}

def getCarInsuranceName(person: Option[Person], minAge: Int) =
 person.filter(_.age >= minAge)
 .flatMap(_.car)
 .flatMap(_.insurance)
 .map(_.name)
 .getOrElse("Unknown")
 
```

자바에서는 orElse라는 메서드를 사용했고, 스칼라에서는 getOrElse라는 메서드를 사용했다는 점을 제외하면 두 코드는 구조가 같다는 것을 확인할 수 있따.
자바와 호환성 떄문에 스칼라에도  null이 존재한다. 하지만 되도록 null을 사용하지 않는 것이 좋다.

<h2>함수</h2>
스칼라의 함수는 어떤 작업을 수행하는 일련의 명령어 그룹이다.
명령어 그룹을 쉽게 추상화할 수 있는 것도 함수 덕분이며 동시에 함수는 함수형 프로그래밍의 중요한 기초석이다.

자바에서는 클래스와 관련된 함수에 메서드라는 이름이 사용된다.
익명 함수의 일종인 람다 표현식도 살펴봤다.

스칼라에서는 다음과 같은 3가지 기능을 제공한다.
1) 함수 형식 : 함수 형식은 자바 함수 디스크립터의 개념을 표현하는 편의 문법이다.
2) 익명 함수 : 익명 함수는 자바의 람다 표현식과 달리 비지역 변수 기록에 제한을 받지 않는다.
3) 커링 지원 : 커링은 여러 인수를 받는 함수를 일부 인수를 받는 여러 함수로 분리하는 기법이다.

<h3>스칼라 일급 함수</h3>
스칼라의 함수는 일급값이다. 즉, Integer나 String처럼 함수를 인수로 전달하거나, 결과로 반환하거나, 변수에 저장할 수 있다.

ex) 트윗에 Java가 포함되어 있거나 짧은 문자열 등으 ㅣ조건으로 트윗을 필터링하려 한다.

```
def isJavaMentioned(tweet: String) : Boolean = tweet.contains("Java")
def isShortTweet(tweet: String) : Boolean = tweet.length() < 20

// 스칼라에서 기본 제공하는 filter로 바로 전달할 수 있다.
val tweets = List("I love the new features in Java", "How's it going?", 
                "An SQL query walks into a bar, sees two tables and says 'Can I join you?'")
                
tweets.filter(isJavaMentioned).foreach(println)
tweets.filter(isShortTweet).foreach(println)

def filter[T](p: (T) => Boolean):List[T]
```

파라미터 p의 형식은 (T) => Boolean 이다. 
자바에서는 함수형 인터페이스를 사용했는데 스칼라에서는 어떤 형식을 사용할까?
(T) => Boolean은 T라는 형식의 객체를 받아 Boolean을 반환함을 의미한다.

<h3>익명 함수와 클로저</h3>
스칼라도 익명 함수의 개념을 지원한다.
스칼라는 람다 표현식과 비슷한 문법을 지원한다.

```
// 익명함수
val isLongTweet : String => Boolean = (tweet : String) => tweet.length() > 60 

// 함수형 인터페이스의 인스턴스
val isLongTweet : String => Boolean = 
  new Function1[String, Boolean] {
    def apply(tweet: String): Boolean = tweet.length() > 60
 }
 
//isLongTweet 변수는 Function1 형식의 객체를 저장하므로 다음처럼 apply메서드를 호출할 수 있다.
isLongTweet.apply("A very short tweet") // false를 반환함 
```

<h3>클로저</h3>
클로저랑 함수의 비지역 변수를 자유롭게 참조할 수 있는 함수의 인스턴스를 가리킨다.
자바의 람다 표현식에는 람다가 정의된 메서드의 지역 변수를 고칠 수 없다는 제약이 있다.
이들 변수는 암시적으로 final로 취급된다.
즉, 람다는 변수가 아닌 값을 닫는다.

```
//자바
public static void main(String[] args) {
 int count = 0;
 Runnable inc = () -> count+=1;  <- 에러 :  count는 명시적으로 final 또는 final에 준하는 변수여야함
 inc.run();
 System.out.println(count);
 inc.run();
}

//스칼라
def main(args: Array[String]) {
 var count = 0
 val inc = () => count+=1 
 inc()
 println(count)  <- 1 출력
 inc()
 println(count)  <- 2 출력
}
```

<h3>커링</h3>
x, y라는 두 인수를 가진 f라는 함수가 있을 때 이는 하나의 인수를 받는 g라는 함수 그리고 g라는 함수는 다시 나머지 인수를 받는 함수로 반환되는 상황으로 볼 수 있다.
여러 인수를 가진 함수를 커링으로 일반화할 수 있다.
즉, 여러 인수를 받는 함수를 인수의 일부를 받는 여러 함수로 분할할 수 있다.
스칼라에서는 기존 함수를 쉽게 커리할 수 있는 방법을 제공한다.

```
// java 두 정수를 곱하는 간단한 메서드
static int multiply(int x, int y) {
 return x * y;
}
int r = multiply(2, 10);

// Function
static Function<Integer, Integer> multiplyCurry(int x) {
 return (Integer y) -> x * y;
}

// multiplyCurry가 반환하는 함수는 x와 인수 y를 곱한 값을 캡처한다. 다음처럼 map과 multiplyCurry를 연결해서 각 요소에 2를 곱할 수 있다.
Stream.of(1, 3, 5, 7)
 .map(multiplyCurry(2))
 .forEach(System.out::println);
```

결과가 2, 6, 10, 14가 출력된다. map은 인수로 Function을 받고 multiplyCurry가 Function을 반환하므로 위 코드는 문제없이 작동한다.

스칼라에서는 위의 복잡한 과정을 자동으로 처리하는 특수 문법을 제공한다.

```
def multiply(x : Int, y : Int) = x * y
val r = multiply(2,10);

def multiplyCurry(x :Int)(y : Int) = x * y 
val r = multiplyCurry(2)(10) 
```

(x :Int)(y : Int) 같은 문법으로 multiplyCurry 메서드는 Int 파라미터 하나를 포함하는 인수 리스트 둘을 받았다.
반면 multiply는 Int 파라미터 둘을 구성된 리스트 하나를 인수로 받는다.
multiplyCurry를 호출한다면?
Int(파라미터 x) 하나로 multiplyCurry를 처음 호출(multiplyCurry(2))하면 파라미터 y를 인수로 받는 다른 함수를 반환하며 이를 x 캡처값(여기서는 2)에 곱한다.
두번째로 함수를 호출하면 x와 y를 곱한다.
즉, multiplyCurry를 처음 호출한 결과를 내부 변수에 저장했다가 재사용한다.

```
val multiplyByTwo : Int => Int = multiplyCurry(2)
val r = multiplyByTwo(10)
```

<h2>클래스와 트레이드</h2>
<h3>간결성을 제공하는 스칼라의 클래스</h3>
스칼라는 완전한 객체지향 언어이므로 클래스를 만들고 객체로 인스턴스화할 수 있다.
스칼라는 자바에서 클래스를 만들고 인스턴스화하는 방법과 문법적으로 비슷한 구조를 제공한다.

ex) 스칼라의 Hello 클래스 정의 코드다.

```
class Hello {
  def sayThankYou(){
    println("Thanks for reading our book")
  }
}

val h = new Hello()
h.sayThankYou()
```

<h3>게터와 세터</h3>
자바에서는 생성자를 정의하고 필드를 초기화했으며 두 개의 게터와 두 개의 세터도 정의했다.
하지만 간단한 클래스임에도 20행이 넘는 코드가 필요하다!
실제로는 비즈니스 로직을 처리하는 데 별 도움이 되지 않는 수많은 코드가 존재한다는 사실은 변하지 않는다

스칼라에서는 생성자, 게터, 세터가 암시적으로 생성되므로 코드가 훨씬 단순해진다.

```
class Student(var name: String, var id: Int) 
val s = new Student("Raoul", 1)  <- Student 객체 초기화
println(s.name)  <- 이름을 얻어 Raoul출력
s.id = 1337   <- id 설정
println(s.id)  <- 1337 출력
```

<h3>스칼라 트레이트와 자바 인터페이스</h3>
스칼라는 트레이트라는 유용한 추상 기능도 제공한다.
스칼라의 트레이트는 자바의 인터페이스를 대체한다.
트레이트로 추상 메서드와 기본 구현을 가진 메서드 두 가지를 모두 정의할 수 있다.

자바의 인터페이스처럼 트레이트는 다중 상속을 지원하므로 자바의 인터페이스와 디폴트 메서드 기능이 합쳐진 것으로 이해할 수 있다.
트레이트는 클래스와 달리 다중 상속될 수 있다.

ex) Sized라는 트레이트는 size라는 가변 필드와 기본 구현을 제공하는 isEmpty 메서드를 포함한다.

```
trait Sized {
 var size : Int = 0  <- size 필드
 def isEmpty() = size == 0  <- 기본 구현을 제공하는 isEmpty 메서드
}
```

트레이트를 클래스와 조합해서 선언할 수 있다.
다음은 항상 0의 크기를 갖는 Empty 클래스를 정의하는 예제다.

```
class Empty extends Sized  <- 트레이트 Sized에서 상속받는 클래스
println(new Empty().isEmpty())  <- true 출력
```

흥미롭게도 자바 인터페이스와는 달리 객체 트레이트는 인스턴스화 과정에서도 조합할 수 있다.

ex) Box 클래스를 만든 다음 어떤 Box 인스턴스는 트레이트 Sized 가 정의하는 동작을 지원하도록 결정할 수 있다.

```
class Box
val b1 = new Box() with Sized  <- 객체를 인스턴스화할 때 트레이트를 조합함
println(b1.isEmpty())  <- true를 출력
val b2 = new Box() 
b2.isEmpty()  <- 컴파일 에러 : Box 클래스 선언이 Sized를 상속하지 않았음.
```
