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

<h3>11.1.2 null때문에 발생하는 문제</
