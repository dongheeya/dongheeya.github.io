<h1>Chapter 14. 자바 모듈 시스템</h1>
 : 자바 9에서 가장 많이 거론되는 새로운 기능은 바로 모듈 시스템이다.<br/>
 : 새로운 자바 모듈 시스템이 어디에 사용될 수 있고, 개발자는 이로부터 어떤 이익을 얻을 수 있는지 설명한다.<br/>
 
 <h2>14.2: 소프트웨어 유추</h2>
 모듈화란 ? <br/>
 궁극적으로 소프트웨어 아키텍처 즉 고수준에서는 기반 코드를 바꿔야 할 때 유추하기 쉬우므로 생산성을 높일 수 있는 소프트웨어 프로젝트가 필요하다.<br/>
 ☞ 관심사 분리, 정보 은닉<br/>
 
 <h3>관심사 분리(SoC)</h3>
 관심사 분리는 프로젝트 프로그램을 고유의 기능으로 나누는 동작을 권장하는 원칙이다.<br/>
 
 ex) 다양한 형식으로 구성된 지출을 파싱하고 분석한 다음 결과를 고객에게 요약 보고하는 회계 애플리케이션을 개발하다고 하자.<br/>
 이때, 서로 거의 겹치지 않는 코드 그룹으로 분리하여, (파싱, 분석, 레프트 기능)이라는 모듈 단위로 쪼갤 수 있으며,<br/>
 클래스르르 그룹화한 모듈을 이용해 애플리케이션의 클래스 간의 관계를 시각적으로 보여줄 수 있다.<br/>
 
 SoC원칙의 장점 3가지<br/>
 1) 개별 기능을 따로 작업할 수 있으므로 팀이 쉽게 협업할 수 있다.<br/>
 2) 개별 부분을 재사용하기 쉽다<br/>
 3) 저체 시스템을 쉽게 유지보수할 수 있다.<br/>

 <h3>정보 은닉</h3>
 정보 은닉은 세부 구현을 숨기도록 장려하는 원칙이다.<br/>
 소프트웨어를 개발할 때 요구사항은 자주 바뀌므로 세부 구현을 숨김으로 프로그램의 어떤 부분을 바뀌었을 때 다른 부분까지 영향을 미칠 가능성을 줄일 수 있다.<br/>
 캡슐화는 특정 코드 조각이 애플리케이션의 다른 부분과 고립되어 있음을 의미한다.<br/>
 
 하지만, 자바9 이전까지는 클래스와 패키지가 의도된 대로 공개되었는지를 컴파일러로 확인할 수 있는 기능이 없었다.<br/>
 
 <h3>자바 소프트웨어</h3>
 자바에서는 public, protected, private 등의 접근 제한자와 패키지 수준 접근 권한 등을 이용해 메서드, 필드 클래스의 접근을 제어했다.<br/>
 하지만 이러한 방식은 최종 사용자에게 원하지 않는 메서드까지 공개하는 상황이 발생했다.<br/>
 설계자는 자신의 클래스에서 개인적으로 사용할 용도라고 생각할 수 있겠지만, 과정이 어쨌든 결과적으로 클래스에 public 필드가 잇다면 사용자 입장에서는
 당연히 사용할 수 있겠다고 생각할 수 있게 된다.<br/>
 
 <h2>자바 모듈 시스템을 설계한 이유</h2>
 자바 언어와 컴파일러에 <u>새로운 모듈 시스템이 추가된 이유</u>를 설명한다.<br/>
 
 <h3>모듈화의 한계</h3>
 자바 9 이전까지는 모듈화된 소프트웨어 프로젝트를 만드는 데 한계가 있었다. (위에서도 언급)<br/>
 
 <h3>제한된 가시성 제어</h3>
 자바는 정보를 감출 수 있는 4가지의 접근자를 제공한다.<br/>
 public, protected, 패키지 수준, private 이렇게 4가지의 가시성 접근자가 있다.<br/><br/>
 
 if) 한 패키지의 클래스와 인터페이스를 다른 패키지로 공개하려면 public을 쓸수 밖에 없는 구조이다.<br/><br/>
 ☞ 이런식으로 이들 클래스와 인터페이스는 모두에게 공개된다.<br/>
 ☞ 원하지 않게 모든 사용자가 이 내부 구현을 마음대로 사용할 수 있다.<br/>
 
 그 결과로, 2가지의 단점이 생긴다.<br/>
 1) 기존의 애플리케이션을 망가뜨리지 않고 라이브러리 코드를 바꾸기가 어려워진다.<br/>
 2) 보안 측면에서 코드가 노출되었으므로 코드의 임의로 조작하는 위협을 더 많이 노출된다.<br/>


<h3>클래스 경로</h3>
앞부분에서 이해하기 쉽고 유지보수하기 쉬운 소프트웨어 즉 추론하기 쉬운 소프트웨어의 장점을 소개했다.<br/>

클래스 경와 JAR조합에서는 몇 가지 약점이 존재한다.<br/>
1) 첫째, 클래스 경로에는 같은 클래스를 구분하는 버전 개념이 없다.<br/>
파싱 라이브러리의 JSONParser 클래스를 지정할 때 버전 1.0을 사용하는지 버전 2.0을 사용하는지 지정할 수 없으므로 <br/>
클래스 경로에 두 가지 버전의 같은 라이브러리가 존재할 때 어떤 일이 일어날지 예측할 수 없다.<br/>

2) 클래스 경로는 명시적인 의존성을 지원하지 않는다.<br/>
각각의 JAR 안에 있는 모든 클래스는 classes라는 한 주머니로 합쳐진다.<br/>
즉 한 JAR가 다른 JAR에 포함된 클래스 집합을 사용하라고 명시적으로 의존성을 정의하는 기능을 제공하지 않는다.<br/>
-> 이 또한, 클래스 경로 때문에 어떤 일이 일어나는지 파악하기 어렵다.<br/>

<h3>거대한 JDK</h3>
자바 개발 키트(JDK)는 자바 프로그램을 만들고 실행하는 데 도움을 주는 도구의 집합이다.<br/>
시간이 흐르면서 JDK의 덩치가 커졌고, 모바일이나 JDK 전부를 필요로 하지 않는 클라우드 환경에서 문제가 되었다.<br/>
또한 자바의 낮은 캡슐화 지원 때문에 JDK의 내부 API가 외부에 공개되었고, 여러 라이브러리에서 JDK 내부 클래스를 사용했다. <br/>
결과적으로 호환성을 깨지 않고는 관련 API를 바꾸기 어려운 상황이 되었다.<br/>

이런 문제들 때문에 JDK 자체도 모듈화할 수 있는 자바 모듈 시스템 설계의 필요성이 제기되었다.<br/>
즉 JDK에서 필요한 부분만 골라서 사용하고, 클래스 경로를 쉽게 유추할 수 있으며, 플랫폼을 진화시킬 수 있는 강력한 캡슐화를 제공할 새로운 건축 구조가 필요했다.<br/>


<h3>자바 모듈 : 큰 그림</h3>
자바 8는 모듈이라는 새로운 자바 프로그램 구조 단위를 제공한다.<br/>
모듈은 module이라는 새 키워드에 이름과 바디를 추가해서 정의한다.<br/>
모듈 디스크립터는 module-info.java 라는 특별한 파일에 저장된다.<br/>

한 개 이상의 패키지를 서술하고 캡슐화할 수 잇지만 단순한 상황에서는 이들 패키지 중 한 개만 노출시킨다.<br/>

![14-2 자바 모듈 디스크립터의 핵심 구조](https://user-images.githubusercontent.com/87962572/140742641-d2e23061-6c7c-4d57-ba7f-b7777562f2a7.PNG)

<h2>자바 모듈 시스템으로 애플리케이션 개발하기</h2>
자바 9 모듈 시스템 전반 과정에서 작은 모듈화 애플리케이션을 구조화하고, 패키지하고, 싫애하는 방법을 나열한다.<br/>

<h3>애플리케이션 셋업</h3>
자바 모듈 싯템을 적용하려면 코드를 구현할 예제 프로젝트가 필요하다.<br/>
애플리케이션을 다음과 같은 여러 작업을 처리해야 한다.<br/>

- 파일이나 URL에서 배용 목록을 읽는다.
- 비용의 문자열 표현을 파싱한다.
- 통계를 계산한다.
- 유용한 요약 정보를 표시한다.
- 각 태스크의 시작, 마무리 지점을 제공한다.

애플리케이션의 개념을 모델링할 여러 클래스와 인터페이스를 정의해야한다
자바 모듈 시스템을 여러 기능으로 분리할 수 있다.

- 다양한 소스에서 데이터를 읽음(Reader, HttpReader, FileReader)
- 다양한 포맷으로 구성된 데이터를 파싱(Parser, JSONParser, ExpenseJSON-Parser)
- 도메인 객체를 구체화(Expense)
- 통계를 계산하고 반환(SummaryCalculator, SummaryStatistics)
- 다양한 기능을 분리 조정(ExpensesApplication)

<h3>세부적인 모듈화와 거친 모듈화</h3>
가장 좋은 모듈 방식은, <b>시스템을 실용적으로 분해하면서 진화하는 소프트웨어 프로젝트가 이해하기 쉽고 고치기 쉬운 수준으로 적절하게 모듈화되어 있는지 
주기적으로 확인하는 프로세스를 갖는것</b> 이다.


<h3>자바 모듈 시스템 기초</h3>

![14-4-3 디렉토리 구조](https://user-images.githubusercontent.com/87962572/140744379-e2b4deaa-29e2-44ec-922a-da7b3524590f.PNG)

module-info.java 라는 파일이 프로젝트 구조의 일부에 포함되어, 소스 코드 파일 루트에 위치해야하며,
모듈의 의존성 그리고 어떤 기능을 외부로 노출할지를 정의한다.

<h2>여러 모듈 활용하기</h2>
<h3>exports 구문</h3>

```
module expenses.readers {
 //exports 패키지명
 exports com.example.expenses.readers; 
 exports com.example.expenses.readers.file; 
 exports com.example.expenses.readers.http; 
}
```

exports 라는 구문이 새로 등장했는데 exports는 다른 모듈에서 사용할 수 있도록 특정 패키지를 공개 형식으로 만든다.<br/>
기본적으로 모듈 내의 모든 것은 캡슐화된다.<br/>
다른 모듈에서 사용할 수 있는 기능이 무엇인지 명시적으로 결정해야 한다.<br/>

```
|─ expenses.application 
 |─ module-info.java
 |─ com 
  |─ example 
    |─ expenses 
      |─ application
        |─ ExpensesApplication.java
 
|─ expenses.readers 
 |─ module-info.java
 |─ com 
  |─ example 
    |─ expenses 
      |─ readers
        |─ Reader.java
      |─ file
        |─ FileReader.java
      |─ http
        |─ HttpReader.jav
```

<h3>requires 구문</h3>
module-info.java
특정 모듈을 import할때 

```
module expenses.readers {
 //requires 모듈명
 requires java.base;  <= 패키명이 아니라 모듈명이다.
 
 exports com.example.expenses.readers;  <= 모듈명이 아니라 패키지 명이다.
 exports com.example.expenses.readers.file; 
 exports com.example.expenses.readers.http; 
}
```

<h3>이름 정하기</h3>
모듈의 이름 규칙을 설명할 때가 되었다.<br/>
expenses.application 처럼 모듈과 패키지 개념이 혼동되지 않도록 단순한 접근 방식을 사용했다.<br/>
모듈명은 노출된 주요 API 패키지와 이름이 같아야 한다는 규칙도 따라야한다.<br/>
모듈명은 패키지를 포함하지 않거나 어떤 다른 이유로 노출된 패키지 중 하나와 이름이 일치하지 않는 상황을 제외하면,<br/>
작성자의 인터넷 도메인명을 역순으로 시작해야한다.<br/>

<h3>컴파일과 패키징</h3>
메이븐을 이용해 컴파일한다면, 먼저 각 모듈에 pom.xml을 추가해야 한다. <br/>
또한 전체 프로젝트 빌드를 조정할 수 있도록 모든 모듈의 부모 모듈에도 pom.xml을 추가하고 의존성을 기술해야 한다.<br/>

<h3>자동 모듈</h3>
HttpReader를 저수준으로 구현하지 않고 아파치 프로젝트의 httpClient 같은 특화 라이브러리를 사용해 구현한다고 가정하자.<br/>
이때 단순하게 requires구분으로 expenses.reader 프로젝트의 module-info.java에 추가한다고 하자.<br/>
그랬을때 다음과 같은 에러가 발생한다.

```
[ERROR] module not found : httpclient
```

의존성을 기술하도록 pom.xml도 갱신해야한다.

```
<dependencies>
 <dependency>
 <groupId>org.apache.httpcomponents</groupId>
 <artifactId>httpclient</artifactId>
 <version>4.5.3</version>
 </dependency>
 </dependencies>
```

이제 mvn clean package를 실행하면 프로젝트가 올바로 발드된다.<br/>
하지만 httpClient는 자바 모듈이 아니다. 자바 모듈로 사용하려는 외부 라이브러리인데 모듈화가 되어있지 않는 라이브러리다.<br/>
자바는 JAR 를 자동 모듈이라는 형태로 적절하게 변환한다.<br/>
모듈 경로상에 있으나 module-info 파일을 가지지 않는 모든 jar는 자동 모듈이 된다.<br/>
그리고 암묵적으로 자신의 모든 패키지를 노출시킨다.<br/>

<h2>모듈 정의와 구문들</h2>
<h3>requires</h3>
requires 구문은 컴파일 타임과 런타임에 한 모듈이 다른 모듈에 의존함을 정의한다.<br/>

```
module com.iteratrlearning.application {
  // requires의 인수는 모듈명
  requires com.iteratrlearning.ui;
}
```

com.iteratrlearning.ui에서 외부로 노출한 공개 형식을 com.iteratrlearning.application에서 사용할 수 있다.

<h3>exports</h3>
exports 구문은 지정한 패키지를 다른 모듈에서 이용할 수 있도록 공개 형식으로 만든다.<br/>
아무 패키지도 공개하지 않는 것이 기본 설정이다.<br/>

```
module com.iteratrlearning.ui {
  requires com.iteratrlearning.core;
  
  //exports 인수는 패키지명
  exports com.iteratrlearning.ui.panels;  ☞ 공개 형식
  exports com.iteratrlearning.ui.widgets; ☞ 공개 형식
}
```

<h3>requires transitive</h3>
다른 모듈에서 제공하는 공개 형식을 한 모듈에서 사용할 수 있다고 지정할 수 있다.<br/>

```
module com.iteratrlearning.ui { 
 requires transitive com.iteratrlearning.core;
 exports com.iteratrlearning.ui.panels;
 exports com.iteratrlearning.ui.widgets;
}
module com.iteratrlearning.application { 
 requires com.iteratrlearning.ui;
}
```

결과적으로 com.iteratrlearning.application 모듈은 com.iteratrlearning.core에서 노출한 공개 형식에 접근할 수 있다.


<h3>exports to</h3>
exports to 구문을 이용해 사용자에게 공개할 기능을 제한함으로 가시성을 좀 더 정교하게 제어할 수 있다.<br/>

```
module com.iteratrlearning.ui { 
 requires com.iteratrlearning.core;
 exports com.iteratrlearning.ui.panels;
 exports com.iteratrlearning.ui.widgets to
    com.iteratrlearning.ui.widgetuser;
}
```

com.iteratrlearning.ui.widgets 의 접근 권한을 가진 사용자의 권한을 com.iteratrlearning.ui.widgetuser 로 제한할 수 있다.

<h3>open 과 opens</h3>
모듈 선언에 open한정자를 이용하면 모든 패키지를 다른 모듈에 반사적으로 접근을 허용할 수 있다.<br/>
open한정자는 모듈의 가시성에 다른 영향을 미치지 않는다.<br/>

```
open module com.iteratrlearning.ui {

}
```

exports-to 로 노출한 패키지를 사용할 수 있는 모듈을 한정했던 것처럼,<br/>
open에 to를 붙여 반사적인 접근을 특정 모듈에만 허용할 수 있다.<br/>

<h3>use와 provides</h3>
provides 구문으로 서비스 제공자를 use 구문으로 서비스 소비자를 지정할 수 잇는 기능을 제공한다.<br/>

