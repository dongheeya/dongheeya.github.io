<h1> 리팩터링, 테스팅, 디버깅</h1>
  
  새 프로젝트라고 모든 것을 처음부터 시작하는 것은 아니다.
  많은 새 프로젝터는 예전 자바로 구현된 기존 코드 기반으로 시작한다.<br/>
  <br/>
  이 장에서는 기존 코드를 이용해서 새로운 프로젝트를 시작하는 상황을 가정한다.<br/>
  람다 표현식을 이용해서 가동성과 유연성을 높이려면 기존 코드를 어떻게 리팩터링해야되는지 설명한다.<br/>
  
  <h2> 9.1 가독성과 유연성을 개선하는 리팩터링</h2>
   <h3> 9.1.1 코드 가독성 개선</h3>
   - 코드 가독성이란? 일반적으로 어떤 코드를 다른 사람도 쉽게 이해할 수 있음을 의미한다.<br/>
   - 코드 가독성을 개선한다는 것은 우리가 구현한 코드를 다른 사람이 쉽게 이해하고 유지보수할 수 있게 만드는 것을 의미한다.<br/>
   - 코드 가독성을 높이려면 코드의 문서화를 잘하고, 표준 코등 규칙을 준수하는 등의 노력이 필요하다.<br/>
   
   <br/>
   9장에서는 람다, 메서드 참조, 스트림을 활용해서 코드 가독성을 개선할 수 있는 세 가지 리팩터링 예제를 소개한다.
   - 익명 클래스를 람다 표현식으로 리팩터링하기<br/>
   - 람다 표현식을 메서드 참조로 리팩터링하기<br/>
   - 명령형 데이터 처리를 스트림으로 리팩터링하기<br/>
  
  <h3> 9.1.2 익명 클래스를 람다 표현식으로 리팩터링하기</h3>
   - 하나의 추상 메서드를 구현하는 익명 클래스는 람다 표현식으로 리팩터링할 수 있다.<br/>
   - 하지만 모든 익명 클래스를 람다 표현식으로 변환할 수 있는 것은 아니다.<br/>
     1) 익명 클래스에서 사용한 this와 super는 람다 표현식에서 다른 의미를 갖는다.<br/>
        익명 클래스에서 this는 익명클래스 자신을 가리키지만 람다에서 this는 람다를 감싸는 캘래스를 가리킨다.<br/>
     2) 익명 클래스는 감싸고 있는 클래스의 변수를 가릴 수 있다.<br/>
        하지만 다음 코드에서 보여주는 것처럼 람다 표현식으로는 변수를 가릴 수 없다,,,,,,?<br/>
  
  ```
  int a = 10;
  Runnable r1 = new Runnable() {
    public void run() {
      int a = 2; // 잘 동작.
      System.out.println(a);
    }
  };

  Runnable r2 = () -> {
      int a = 2; // 컴파일 에러.
      System.out.println(a);
  }
  
  ```
   - 마지막으로 익명 클래스를 람다 표현식으로 바꾸면 콘텍스트 오버로딩에 따른 모호함이 초래될 수 있다.<br/>
   ex) Task 라는 Runnable 과 같은 시그니처를 갖는 함수형 인터페이스
    
    interface Task {
        public void execute();
    }
    public static void doSomething(Runnable r){ r.run(); }
    public static void doSomething(Task a){ r.execute(); }

    // Task를 구현하는 익명클래스 전달가능.
    doSomething(new Task() {
        public void execute() {
            System.out.println("Danger danger!!");
        }
    });
    
    즉, doSomething은 Runnable과 Task중 어느것을 가리키는지 알수 없이 모호하다.
    그래서 명시적 형변환을 이용해서 모호함을 제거할 수 있다.
    
    //명시적 형변환으로 모호한 제거 가능.
    doSomething((Task)() -> System.out.println("Danger danger!!"));

  <h3> 9.1.3 람다 표현식을 메서드 참조로 리팩터링하기. </h3>
  람다 표현식은 쉽게 전달할 수 있는 짧은 코드이다.<br/>
  하지만, 람다표현식 대신 메서드 참조를 이용하면 가독성을 높일 수 있다.<br/>
  메서드 참조의 "메서드명"으로 코드의 의도를 명확하게 알릴 수 있다.<br/>
  
  ex) 6장에서 소개한 칼로리 수준으로 요리하는 그룹화하는 코드이다.
  ```
     Map<CaloricLevel, List<Dish>> dishesByCaloricLevel = menu.stream()
        .collect(
            groupingBy(dish -> {
                if(dish.getCalories() <= 400) return CaloricLevel.DIET;←
                else if (dish.getCalories() <= 700) return CaloricLevel.NORMAL;←
                else return CaloricLevel.FAT;← 이 부분을 아래와 같이 변환할 수 있다.
            }));
            
    // 람다표현식을 별도의 메서드로 추출한 후 groupingBy에 인수로 전달.
    Map<CaloricLevel, List<Dish>> dishesByCaloricLevel = 
        menu.stream().collect(groupingBy(Dish::getCaloricLevel));

    public class Dish {
        ...
        public CaloricLevel getCaloricLevel() {
            if(dish.getCalories() <= 400) return CaloricLevel.DIET;
            else if (dish.getCalories() <= 700) return CaloricLevel.NORMAL;
            else return CaloricLevel.FAT;
        }
    }
 ```
 
  sum, maximum 등 자주 사용하는 리듀싱 연산은 메서드 참조와 함 께 사용가능한 내장 헬퍼 메서드 제공
  
  ```
// 저수준 리듀싱 연산
int totalCalories = 
    menu.stream().map(Dish::getCalories)
                 .reduce(0, (c1, c2) -> c1 + c2);

// 내장 컬렉터 이용
int totalCalories = menu.stream().collect(summingInt(Dish::getCalories));
  ```
 <h3> 9.1.4 명령형 데이터 처리를 스트림으로 리팩터링하기</h3>
 - 이론적으로는 반복잔를 이용한 기존의 모든 컬렉션 처리 코드를 스트림 API로 바꿔야한다.<BR/>
 - 이유는 다음과 같다.<BR/>
    1) 스트림API는 데이터 처리 파이프라인의 의도를 더 명확하게 보여주기 떄문이다.<BR/>
    2) 데이터 처리 파이프라인의 의도를 더 명확하게 보여주기 때문이다.<BR/>
    3) 쇼트서킷과 게으름 이라는 강력한 최적화할 수 있다.<BR/>
    4) 멀티코어 아키텍처를 활용할 수 있다.<BR/>
 
``` 
 // 필터링과 추출로 엉킨코드. (전체구현을 살펴봐야 전체 코드를 이해할 수 있고, 병렬화하기 어렵다.)
List<String> dishNames = new ArrayList<>();
for(Dish dish: menu) {
    if(dish.getCalories() > 300) {
        dishNames.add(dish.getName());
    }
}

// 스트림을 이용해 직접적으로 기술하고 쉽게 병렬화
menu.parallelStream()
    .filter(d -> d.getCalories() > 300)
    .map(Dish::getName)
    .collect(toList());
 ``` 
 <h3> 9.1.5 코드 유연성 개선</h3>
 - 다양한 람다를 전달해서 다양한 동작을 표현하여 보다 변화하는 요구사항에 대응할 수 있는 코드를 구현할 수 있다.
