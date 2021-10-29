<h1>Chapter 11. null대신 Optional 클래스</h1>

1) NullPointerException <br/>
 : 자바로 프로그램을 개발하면서 한번이라도 안 겪어본 사람은 없을 것이다.<br/>
 : 초급자, 중급자, 남녀 노소를 불문하고 모든 자바 개발자를 괴롭히는 예외긴 하지만, null이라는 포현을 사용하면서 당연히 치러야할 대가이다<br/>
 : 거시적인 프로그래밍 관점에서는 조금 다르게 null을 바라보고 있다.<br/>
2) 토그 호어(1965년)이 알골이라는 언어를 설계하면서 처음 "null"참조가 등장했다.<br/>
 : 그는 구현하기가 쉬웠기 때문에 null을 도입했다.<br/>
 : 컴파일러의 자동확인 기능으로 모든 참조를 안전하게 사용할 수 있을 것이라는 목표를 정하고 만들었다.<br/>
 : 하지만 이로 인해 NullPointerException이라는 귀찮은 예외를 계속해서 겪고있다.<br/>
 
<h2>11.1 값이 없는 상황을 어떻게 처리할까?</h2>
ex) 자동차와 자동차 보험을 갖고 있는 사람 객체를 중첩 구조로 구현했다고 하자.

```
Persion/Car/Insurance 데이터 모델

public class Person {
  private Car car;
  private Car getCar() { return car;}
}

public class Car {
  private Insurance insurance;
  public Insurance getInsurance() { return insurance; }
}

public class Insurance {
  private String name;
  pupblic String getName() { return name;} 
}

public String getCarInsuranceName(Person person){
  return person.getCar().getInsurance().getName();
} 
```
<u>다음 코드에서의 문제점은 무엇일까?</u>
- 차를 소유하지 않은 사람이 있을 수 있다. -> getCar를 호출한경우 !
- getInsurance는 null 참조의 보험 정보를 반환하려 할 것이므로 런타임에 NullPointerException이 발생하면서 프로그램이 종료된다.
- Person이 null이라면 어떻게 될까? getInsurance가 null을 반환한다면?

<h2>11.1.1 보수적인 자세로 NullPointerException줄이기</h2>
예기치 않은 NullPointerException을 피하려면 어떻게 해야할까?<br/>
대부분의 프로그래머는 필요한 곳에 다양한 null확인 코드를 추가해서 null 예외 문제를 해결하려 할 것이다.<br/>
다음은 null확인 코드를 추가해서 NullPointerException을 줄이려는 코드다.<br/>

```
null 안전 시도 1: 깊은 의심

public String getCarInsuranceName(Person person){
  if(person != null){
    Car car = person.getCar();
    if(car != null){
      Insurance insurance = car.getInsurance();
      if(insurance != null){
        return insurance.getName();
      }
    }
  }
  return "Unknown";
}
```
위 코드에서는 변수를 참조할 때마다 null 확인하며 중간 과정에 하나라도 null참조가 있으면 unkwon이라는 문자열을 반환한다.<br/>
모든 변수가 null인지 의심하므로 변수를 접근할 때마다 중첩된 if가 추가되면서 코드 들여쓰기 수준이 증가한다.<br/>
이와 같은 반복 패턴 코드를 '깊은 의심'이라고 부른다.<br/>
죽, 변수가 null인지 의심되어 중첩 if블록을 추가하면 코드 들여쓰기 수준이 증가한다.<br/>
이를 반복하다보면 코드의 구조가 엉망이 되고 가독성이 떨어진다.<br/>

```
null 안전 시도 2: 너무 많은 출구

public String getCarInsuranceName(Person person){
  if(person != null){
    return "Unknown";
  }
  Car car = person.getCar();
  if(car != null){
    return "Unkwon";
  }
  
  Insurance insurance = car.getInsurance();
  if(insurance == null){
    return "Unknown";
  }
  return insurance.getName();
}
```

위 코드로는 중첩 if문을 줄일수 있다.<br/>
하지만 메서드에 네 개의 출구가 생겼기 떄문에 유지보수가 어려워진다.<br/>
게다가 null일 때 반환되는 기본값 "Unkwon"이 세 곳에서 반복되고 있는데 같은 문자열을 반복하면서 오타 등의 실수가 생길 수 있다.<br/>
또한 만약에 누군가 null일 수 있다는 사실을 깜빡하게된다면?!<br/>
<b>따라서 값이 있거나 없음을 표현할 수 있는 좋은 방법이 필요하게되었다.</b><br/>

<h3>11.1.2 null때문에 발생하는 문제</h3>
자바에서 null참조를 사용하면서 발생할 수 있는 이론적, 실용적 문제를 확인하자.<br/>
- 에러의 근원이다 : NullPointerException은 자바에서 가장 흔히 발생하는 에러이다.<br/>
- 코드를 어지럽힌다 : 때로는 중첩된 null확인 코드를 추가해야하므로 null 때문에 코드 가독성이 떨어진다.<br/>
- 아무 의미가 없다 : null 은 아무 의미도 표현되지 ㅇ낳는다. 특히 정적 형식 언어에서 값이 없음을 표현하는 방법으로는 적절하지 않다.<br/>
- 자바 철학에 위배된다 : 자바는 개발자로부터 모든 포인터를 숨겼다. 하지만 예외가 있는데 그것이 바로 null 포인터다.<br/>
- 형식 시스템에 구멍을 만든다 : null은 무형식이며 정보를 포함하고 있지 않으므로 모든 참조 형식에 null을 할당할 수 있다.<br/> 
이런 식으로 null이 할당되기 시작하면서 시스템의 다른 부분으로 null이 퍼졌을 때 애초에 null이 어떤 의미로 사용되었는지 알 수 없다.<br/>
<br/>

<h3>11.1.3 다른 언어는 null 대신 무얼 사용하나?</h3>
최근 그루비 같은 언어에서는 안전 내비게이션 연산자(?.) 도입해서 null문제를 해결했다.<br/>
사람들이 그들의 자동차에 적용한 보험회사 이름을 가져오는 그루비 코드 예제다.<br/>

```
def carInsuranceName = person?.car?.insurance?.name
```

그루비를 모르는 독자라도 위 코드를 쉽게 이해할 수 있을 것이다.<br/>
어떤 사람은 자동차를 가지고 있지 않을 수 있으며 따라서 Person객체의 car참조는 null이 할당되어 있을 수 있다.<br/>
그루비 안전 내비게이션 연산자를 이용하면 null참조 예외 걱정 없이 객체에 접근할 수 있다.<br/>

모든 자바 개발자가 아전 내비게이션 연산자를 간절히 원하는 것이 아니다.<br/>
<u>null예외 문제를 해결할 수 있지만 이는 문제의 본질을 해결하는 것이 아니라 문제를 뒤로 미루고 숨기는 것이나 마찬가지다.</u>

하스켈, 스칼라 등의 함수 언어는 아예 다른 관점에서 null 문제를 접근한다.<br/>
하스켈은 선택형값을 저장할 수 있는 Maybe라는 형식을 제공한다.<br/>
Maybe는 주어진 형식의 값을 갖거나 아니면 아무 값도 갖지 않을 수 있다.<br/>
따라서 null참조 개념은 자연스럽게 사라진다.<br/>
스칼라도 T형식의 값을 갖거나 아무 값도 갖지 않을 수 있는 Option[T]라는 구조를 제공한다.<br/>

자바8은 '선택형값'개념의 영향을 받아서 java.util.Optional<T>라는 새로운 클래스를 제공한다.<br/>
이 장에서는 java.util.Optional<T> 이용해서 값이 없는 상황을 모델링하는 방법을 설명한다.<br/>
마지막으로 새로운 Optional 클래스의 기능을 살펴보고, 새로운 기능을 효과적으로 사용하는 방법을 보여주는 몇가지 예제를 소개한다.<br/>
 
 <h2>11.2 Optional 클래스 소개</h2>
 자바 8은 하스켈과 스칼라의 영향을 받아서 java.util.Optional<T>라는 새로운 클래스를 제공한다.<br/>
 Optional은 선택형과 캡슐화하는 클래스다.<br/>
 
 ex) 어떤 사람이 차를 소유하는있지 않다면 Person 클래스의 car 변수는 null을 가져야 할 것이다.<br/>
 새로운 Optional을 이욯라 수 있으므로 null을 할당하는 것이 아니라 [그림11-1]에서 보여주는 것처럼 변수형을 Optional<Car>로 설정할 수 있다.<br/>
 
 값이 있으면 Optional클래스는 값을 감싼다.<br/>
 반면 값이 없으면 Optional.empty 메서드로 Optional을 반환한다.<br/>
 
 null참조와 Optional.empty()는 서로 무엇이 다른지 궁금할 것이다.<br/>
 의미상 둘이 비슷하지만 실제로는 차이점이 많다.<br/>
 null을 참조하려 하면 NullPointerException이 발생하지만 Optional.empty()는 Optional 객체이므로 이를 다양한 방식으로 활용할 수 있다.<br/>
 
 null 대신 Optional을 사용하면서 Car형식이 Optional<Car>로 바뀌었다.<br/>
 값이 없을 수 있음을 명시적으로 보여준다.<br/>
 반면 Car형식을 사용했을 때는 Car에 null참조가 할당될 수 있는데 이것이 올바른 값인지 아니면 잘못된 값인지 판단할 아무 정보도 없다.<br/>
 
 ```
 public class Person { 
  private Optional<Car> car; ☜ 사람이 차를 소유했을 수도 소유하지 않앗을 수도 있음로 Optional로 정의한다.
  pubilc Optional<Car> getar(){
   return car;
  }
 }
 
 public class Car { 
  private Optional<Insurance> insurance; ☜ 자동차가 보험에 가입되어 있을 수도 가입되어 있지 않을수도 있으므로 Optional로 정의한다.
  public Optional<Insurance> getInsurance(){
   return insurance;
  }
 }
 
 public class Insurance { 
  private String name; ☜ 보험회사에는 반드시 이름이 있다.
  public String getName(){
   return name;
  }
 }
 ```
 
 Optional 클래스를 사용하면서 모델의 의미가 더 명확해졌음을 확인할 수 있다.<br/>
 사람은 Optional<Car>를 참조하여 자동차는 Optional<Insurance>를 참조하는데, 이는 사람이 자동차를 소유했을 수 도 아닐 수도 있으며,<br/>
 자동차는 보험에 가입되어 있을 수도 아닐 수도 있음을 명확히 설명한다.<br/>
 
 보험회사 이름은 Optional<String>이 아니라 String 형식으로 선언되어 있는데, 이는 보험회사는 반드시 이름을 가져야 함을 보여준다.<br/>
 따라서 보험회사 이름을 참조할 때 NullPointException이 발생할 수도 있다는 정보를 확인할 수 있다.<br/>
 하지만 보험회사 이름이 null인지 확인하는 코드를 추가할 필요는 없다.<br/>
 
 Optional을 이용하면 값이 없는 상황이 우리 데이터에 문제가 있는 것인지 아니면 알고리즘의 버그인지 명확하게 구분할 수 있다.<br/>
 모든 null참조를 Optional로 대치하는 것은 바람직하지 않다.<br/>
 Optional의 역할은 더 이해하기 쉬운 API를 설게하도록 돕는 것이다.<br/>
 메서드의 시그니처만 보고도 선택형값인지 여부를 구별할 수 있다.<br/>
 Optional이 등장하면 이를 언랩해서 값이 없을 수 있는 상황에 적절하게 대응하도록 강제하는 효과가 있다.<br/>
 
 <h2>11.3 Optional 적용 패턴</h2>
 실제로는 Optional을 어떻게 활용할 수 있을까?<br/>
 Optional로 감싼 값을 실제로 어떻게 사용할 수 있을까?<br/>
 
 <h3>11.3.1 Optional 객체 만들기</h3><br/>
 <b>빈 Optional</b><br/>
 이전에도 언급했듯이 정적 팩토리 메서드 Optional.empty로 빈 Optional 객체를 얻을 수 있다.<br/>
 
 ```
 Optional<Car> optCar = Optional.empty();
 ```
 
 <b>null이 아닌 값으로 Optional 만들기 </b><br/>
 또는 정적 팩토리 메서드 Optional.of로 null이 아닌 값을 포함하는 Optional을 만들 수 있다.<br/>
 
 ```
 Optional<Car> optCar = Optional.of(car);
 ```
 
 이제 car가 null이라면 즉시 NullPointerException이 발생한다.(Optional을 사용하지 않다면 car의 프로퍼티에 접근하려 할때 에러가 발생했을 것이다.)<br/>
 
 <b>null값으로 Optional만들기 </b>
 마지막으로 정적 팩토리 메서드 Optional.ofNullable로 null값을 저장할 수 있는 Optional을 만들 수 있다.<br/>
 
 ```
 Optional<Car> optCar = Optional.ofNullable(car);
 ```
 
 car가 null이면 빈 Optional 객체가 반환된다.<br/>
 그런데 Optional에서 어떻게 값을 가져오는지는 아직 살펴보지 않았다.<br/>
 get 메서드를 이용해서 Optional의 값을 가져올 수 있는데, 이는 곧 살펴볼 것이다.
 그런데 Optional이 비어있으면 get을 호출했을 때 예외가 발생한다.<br/>
 <u>즉, Optional을 잘못 사용하면 결국 null을 사용했을 때와 같은 문제를 겪을 수 있다.</u><br/>

 <h3>11.3.2 맵으로 Optional의 값을 추출하고 변환하기</h3> 
 보통 객체의 정보를 추출할 때는 Optional을 사용할 떄가 많다.
 예를 들어 보험회사의 이름을 추출한다고 가정하자.
 
 이름 정보를 접근하기 전에 insurance가 null인지 확인해야한다.
 
 ```
 String name= null;
 if(insurance != null){
  name = insurance.getName();
 }
 ```
 
 ```
 Optional<Insurance> optInsurance = Optional.ofNullable(insurance);
 Optional<String> name = OptInsurance.map(Insurance::getName);
 ```
 
 Optional이 값을 포함하면 map인수로 제공된 함수가 값을 바꾼다.
 Optional이 비어있으면 아무 일도 일어나지 않는다.
 
 ![11-2 스트림과 Optional의 map 메서드 비교](https://user-images.githubusercontent.com/87962572/139439835-bab786da-bbf6-4d8c-b281-6d1e0afe2e78.PNG)
 
 
 <h3>11.3.3 flapMap 으로 Optional 객체 연결</h3>
 map을 사용하는 방법을 배웠으므로 다음처럼 map을 이용해서 코드를 재구현할 수 있다.
 
 
 ```
 Optional<Person> optPerson = Optional.of(person); ☜ null이 아닌 값을 포함,  null이면 에러
 Optional<String> name = optPerson.map(Person::getCar).map(Car::getInsurance).map(Insurance::getName);
 ```
 ![11-3 이차원 Optional](https://user-images.githubusercontent.com/87962572/139439822-b30236fe-5da9-46b1-b261-dc4b548f97f3.PNG)
 
 
 <u> 안타깝게도 위 코드는 컴파일 되지 않는다</u>
 optPerson의 형식은 Optional<People>이므로 map으로 호출할 수 있다.
 하지만 getCar은 Optional<Car>형식의 객체를 반환한다.
 map의 연산의 결과는 Optional<Optional<Car>> 형식의 객체다.
 getInsurance는 또 다른 Optional 객체를 반환하므로 getInsurane 메서드를 지원하지 않는다.
 
 
 flatMap을 사용하면 이런 문제를 해결할 수 있다.<br/>
 flatMap은 함수의 인수로 받아서 다른 스트림을 반환하는 메서드다.<br/>
 보통 인수로 받은 함수를 스트림의 각 요소에 적용하면 스트림의 스트림이 만들어진다<br/>
 하지만, flatMap은 인수로 받은 함수를 적용해서 생성된 각각의 스트림에서 콘텐츠만 남긴다.<br/>
 즉, 함수를 적용해서 생성된 모든 스트림이 하나의 스트림으로 병합되어 평준화된다.<br/>
 
 ![11-4 스트림과 Optional의 flatMap 메서드 비교](https://user-images.githubusercontent.com/87962572/139439832-06082766-4117-473f-a7e3-4767e225699c.PNG)
 
 flatMap 메서드로 전달된 함수는 각각의 정사각형을 두 개의 삼각형을 포함하는 스트림으로 변환한다.<br/>
 map을 적용한 결과로 세 개의 스트림을 포함하는 하나의 스트림이 생성된다.<br/>
 flatMap 메서드 덕분에 이차원 스트림이 여섯개의 삼각형을 포함하는 일차원 스트림으로 바뀐다.<br/>
 
 <b>Optional 로 자동차의 보험회사 이름 찾기</b>
 
 ```
 public String getCarInsuranceName(Optional<Person> person){
  return person.flatMap(Person::getCar)
               .flatMap(Car::getInsurance)
               .map(Insurance::getName).orElse("Unkwon");
 }
 ```
 
 Optional을 활용하여 null을 확인하느라 조기 분기문을 추가해서 코드를 복잡하게 만들지 않으면서도 쉽게 이해할 수 있는 코드로 완성했다.
 
 <h3>11.3.4 Optional 스트림 조작</h3>
 자바 9에서는 Optional을 포함하는 스트림을 쉽게 처리할 수 있도록 Optional에 stream() 메서드를 추가했다.
 Optional 스트림을 값을 가진 스트림으로 변환할 때 이 기능을 유용하게 활용할 수 있다.
 
 ex) List<Person>을 인수로 받아 자동차를 소유한 사람들이 가입한 보험회사의 이름을 포함하는 Set<String>을 반환하도록 메서드를 구현해야한다.
 ```
 public Set<String> getCarInsuranceNames(List<Person> persons){
  return persons.steram().map(Person::getCar) ☜ 사람목록을 각 사람이 보유한 자동차의 Optional<Car> 스트림으로 반환
        .map(OptCar -> optCar.flatMap(Car::getInsurance)) ☜ flatMap연산을 이용해 Optional<Car>을 해당 Optional<Insurance>로 변환
        .map(OptIns -> optCar.map(Insurance::getName)) ☜ Optional<Insurance> 를 해당 이름의 Optional<String>으로 매핑
        .flatMap(Optional::stream) ☜ Stream<Optional<String>>을 현재 이름을 포함하는 Stream<String>으로 변환
        .collect(toSet()); ☜ 결과 문자열을 중복되지 않은 값을 갖도록 집합으로 수집
 }
 ```
 - getCar메서드가 단순히 Car가 아니라 Optional<Car>를 반환하므로 사람이 자동차를 가지지 않을 수도 있는 상황이다.<br/>
 - 따라서 첫번째 map변환을 수행하고 Stream<Optional<Car>>를 얻는다.<br/>
 - 이어지는 두 개의 map연산을 이용해 Optional<Car>를 Optional<Insurance>로 변환한 다음에 스트림이 아니라 각각의 요소에 했던 것처럼 각각을 Optional<String>으로 변환한다.<br/>
 - 세 번의 변환 과정을 거친 결과 Stream<Optional<String>>을 얻는데 사람이 차를 갖고 있지 않거나 또는 차가 보험에 가입되어 있지 않아 결과가 비어있을 수 있다.<br/>
 - 마지막 결과를 얻으려면 빈 Optional을 제거하고 값을 언랩해야 한다는 것이 문제다.<br/>
 - 다음 코드처럼 filter, map을 순차적으로 이용해 결과를 얻을 수 있다.<br/>
 
 <h3>11.3.5 디폴트 액션과 Optional 언랩</h3>
 - get()은 값을 읽는 가장 간단한 메서드면서 동시에 가장 안전하지 않은 메서드다. 메서드는 get은 래핑된 값이 있으면 해당 값을 반환하고 값이 없으면 NoSuchElementException을 발생한다.
 따라서 Optional에 값이 반드시 있다고 가정할 수 있는 상황이 아니면 get 메서드를 사용하지 않는 것이 바람직하다.<br/>
 - orElse(T other) 를 사용했다. orElse메서드를 이용하면 Optional이 값을 포함하지 않을 때 기본값을 제공할 수 있다.<br/>
 - orElseGet(Supplier<? extends T> other) 는 orElse 메서드에 대응하는 게으른 버전의 메서드다. Optional에 값이 없을 때만 Supplier가 실행되기 때문이다.<br/>
 - orElseThrow(Supplier<? extends x> exceptionSupplier)는 Optional이 비어있을 때 예외를 발생시킨다는 점에서 get메서드와 비슷하다.<br/>
 - ifPresent(Consumer<? super T> consumer)를 이용하면 값이 존재할 때 인수로 넘겨준 동작을 실행할 수 있다. 값이 없으면 아무 일도 일어나지 않는다.<br/>
 
<h3>11.3.6 두 Optional 합치기</h3>
이제 Person과 Car정보를 이용해서 가장 저렴한 보험료를 제공하는 보험회사를 찾는 몇몇 복잡한 비즈니스 로직을 구현한 외부 서비스가 있다고 가정하자.<br/>
2개의 Optional을 인수로 받아서 Optional<Insurance>를 반환하는 null안전 버전의 메서드를 구현해야 한다고 가정하자.<br/>
인수로 전달한 값 중 하나라도 비어있으면 빈 Optional<Insurance>를 반환한다.<br/>
Optional클래스는 Optional이 값을 포함하는 지 여부를 알려주는 ifPresent라는 메서드로 제공한다.<br/>
따라서 isPresent를 이용해서 다음처럼 코드를 구현할 수 있다.<br/>

```
public Optional<Insurance> nullSafeFindCheapestInsurance(Optional<Person> person, Optional<Car> car){
 if(person.ifPresent() && car.isPresent()){
  return Optional.of(findCheadestInsurance(person.get(), car.get());
 }else{
  return Optional.empty();
 } 
}
```
이 메서드의 장점은 person과 car의 시그니처만으로 둘 다 아무값도 반환하지 않을 수 있다는 정보를 명시적으로 보여준다는 것이다.
Optional클래스에서 제공하는 기능을 이용해서 이 코드를 더 자연스럽게 개선할 수 없을까? ☞ filter

<h3>11.3.7 필터로 특정값 거르기</h3>
ex) 보험회사 이름이 'CambridgeInsurance'인지 확인해야 한다고 가정하자.<br/>
이 작업을 안전하게 수행하려면 다음 코드에서 보여주는 것처럼 Insuracne 객체가 null 인지 여부를 확인한 다음에 getName메서드를 호출해야 한다.<br/>

```
Insurance insurance = ...;
if(insurance != null && "CambridgeInsurance".equals(insurance.getName())){
 System.out.println("ok");
}
```

Optional 객체에 filter 메서드를 이용해서 다음과 같이 코드를 재구현할 수 있다.<br/>

```
Optional<Insurance> optInsurance = ...;
optInsurance.filter(insuracne -> "CambridgeInsurance".equals(insurance.getName())).
ifPresent(x -> System.out.println("ok"));
```

filter 메서드는 프레디케이트를 인수로 받는다.<br/>
Optional객체가 값을 가지며 프레디케이트와 일치하면 filter 메서드는 그 값을 반환하고 그렇지 않으면 빈 Optional 객체를 반환한다.<br/>
Optional은 최대 한 개의 요소를 포함할 수 있는 스트림과 같다고 설명했으므로 이 사실을 적용하면 filter 연산의 결과를 쉽게 이해할 수 있다.<br/>
Optional이 비어있다면 filter 연산은 아무 동작도 하지 않는다.<br/>

```
empty	: 빈 Optional 인스턴스 반환
filter	: 값이 존재하며 프레디케이트와 일치하면 값을 포함하는 Optional 반환하고 없으면 빈 Optional반환
flatMap :	값이 존재하면 인수로 제공된 함수를 적용한 결과 Optional 반환하고 값이 없으면 빈 Optional 반환
get :	값이 존재하면 Optional이 감싸고 있는 값을 반환하고 값이 없으면 NoSuchElementException 발생
ifPresent :	값이 존재하면 지정된 Consumer를 실행하고, 값이 없으면 아무 일도 일어나지 않음
ifPresentOrElse :	값이 존재하면 지정된 Consumer를 실행하고, 값이 없으면 아무 일도 일어나지 않음
isPresent :	값이 존재하면 true, 없으면 false 반환
map :	값이 존재하면 제공된 매핑 함수를 적용
of :	값이 존재하면 값을 감싸는 Optional을 반환하고 값이 없으면 NullPointerException 발생
ofNullable :	값이 존재하면 값을 감싸는 Optional을 반환하고 값이 없으면 빈 Optional 반환
or :	값이 존재하면 같은 Optional을 반환하고 값이 없으면 Supplier에서 만든 Optional을 반환
orElse :	값이 존재하면 값을, 없으면 기본값 반환
orElseGet :	값이 존재하면 값을, 없으면 Supplier에서 제공하는 값을 반환
orElseThrow	: 값이 존재하면 값을 반환하고 값이 없으면 Supplier에서 생성한 예외를 발생
stream :	값이 존재하면 존재하는 값만 포함하는 스트림 반환하고 값이 없으면 빈 스트림 반환
```

* 잠재적으로 null이 될 수 있는 대상을 Optional로 감싸기 *

```
Object value = map.get("key");
```

Map의 get 메서드는 요청한 키에 대응하는 값을 찾지 못했을 때 null을 반환한다.
지금까지도 실펴본 것처럼 null을 반환하는 것보다는 Optional을 반환하는 것이 더 바람직하다.
get 메서드의 시그니처는 우리가 고칠 수 없지만 get메서드의 반환값은 Optional로 감쌀 수 있다.

```
Optional<Object> value = Optional.ofNullable(map.get("key"));
```

* 예외와 Optional 클래스 *
자바 API는 어떤 이유에서 값을 제공할 수 없을 때 NULL을 반환하는 대신 예외를 발생시킬 때도 있습니다.<br/>
예를들어, Integer.parseInt(String) 인 경우에도 문자열을 정수로 변환하는 정적 메서드이며 문자열을 반환하지 못할 때 NumberFormatException을 발생한다.<br/>
즉, 문자열이 숫자가 아니라는 사실을 예외로 알리는 것이다.<br/>

정수로 변환할 수 없는 문자열 문제를 빈 Optional로 해결할 수 있다.<br/>
즉, parseInt가 Optional을 반환하도록 모델링할 수 있다.<br/>

```
public static Optional<Integer> stringToInt(String s){
 try{
  return Optional.of(Integer.parseInt(s));
 }catch(NumberFormatException e){
  return Optional.empty();
 }
}
```
