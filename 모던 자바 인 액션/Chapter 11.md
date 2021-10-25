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
 
 
 
 
 
 
 


