<h1>Chapter 12. 새로운 날짜와 시간 API</h1>
 자바 API는 복잡한 애플리케이션을 만드는 데 필요한 여러 가지 유용한 컴포넌트를 제공한다.
 자바 1.0에서는 java.util.Date 클래스 하나로 날짜와 시간 관련 기능을 제공했다.
 Date라는 이름과 달리 Date클래스는 특정 시점을 미리초 단위로 표현했다.
 
 게다가 1900년을 기준으로 하는 오프셋, 0에서 시작하는 달 인덱스 등 모호한 설계로 유용성이 떨어졌다.
 
 ex) 2017년 9월 21일 
 
 ```
 Date date = new Date(117,8,21);
 ```
 
 <h2>LocalDate, LocalTime, Instant, Duration, Period 클래스</h2>
 <h3>12.1.1 LocalDate와 LocalTime 사용</h3>
 LocalDate 인스턴스는 시간을 제외한 날짜를 표현하는 불변 객체다.
 특히, LocalDate 객체는 어떤 시간대 정보도 포함하지 않는다.
 
 정적 팩토리 메서드 of로 LocalDate 인스턴스를 만들 수 있다.
 
 ```
 LocalDate date = LocalDate.of(2017,9,21); ☜ 2017-09-21
 int year = date.getYear();                ☜ 2017
 Month month = date.getMonth();            ☜ SEPTEMBER
 int day = date.getDayOfMonth();           ☜ 21
 DayOfWeek dow = date.getDayOfWeek();      ☜ Thursday
 int len = date.lengthOfMonth();           ☜ 31 (3월의 일 수)
 boolean leap = date.isLeapYear();         ☜ false (윤년이 아님)
 ```
  
 팩토리 메서드 now는 시스템 시계의 정보를 이용해서 현재 날짜 정보를 얻는다.
 
 ```
 LocalDate today = LocalDate.now();
 ```
 
 TemporalField는 시간 관련 객체에서 어떤 필드의 값에 접근할 지 정의하는 인터페이스다.
 get메서드에 TemporalField를 전달해서 정보를 얻는 방법도 있다.
 열거자 ChroneField는 TemporalField 인터페이스를 정의하므로 다음 코드에서 보여주는 것처럼 ChroneField 의 열거자 요소를 이용해서
 원하는 정보를 쉽게 얻을 수 있다.
 
 ```
 int year = date.get(ChronoField.YEAR);
 int month = date.get(ChronoField.MONTH_OF_YEAR);
 int day = date.get(ChronoField.DAY_OF_MONTH);
 ```
 
 시/분/초를 읽기 위해서는 LocalTime을 사용한다.
 ```
 LocalTime time = LocalTime.of(13,45,20);
 int hour = time.getHour();
 int minute = time.getMinute();
 int second = time.getSecond();
 ```
 
 
 <h2>12.1.2 날짜와 시간 조합</h2>
 LocalDateTime은 LocalDate와 LocalTime을 쌍으로 갖는 복합 클래스다.
 즉, LocalDateTime은 날짜와 시간을 모두 표현할 수 있으며 다음 코드에서 보여주는 것처럼 직접 LocalDateTime을 만드는 방법도 있고
 날짜와 시간을 조합하는 방법도 있다.
 
 ```
 //2017-09-21T13:45:20
 LocalDateTime dt1 = LocalDateTime.of(2017,Month.SEPTEMBER,21,13,45,20);
 LocalDateTime dt2 = LocalDateTime.of(date,time);
 LocalDateTime dt3 = date.atTime(13,45,20);
 LocalDateTime dt4 = date.atTime(time);
 LocalDateTime dt5 = time.atDate(date);
 ```
 
 LocalDate의 atTime 메서드에 시간을 제공하거나 LocalTime의 atDate 메서드에 날짜를 제공해서 LocalDateTime을 만드는 방법도 있다.
 LocalDateTime의 toLocalDate나 toLocalTeimd메서드로 LocalDate나 LocalTime인스턴스를 추출할 수 있다.
 
 ```
 LocalDate date1 = dt1.toLocalDate(); ☜ 2017-09-21
 LocalTime time1 = dt1.toLocalTime(); ☜ 13:45:20
 ```
 
 <h3> Instant 클래스 : 기계의 날짜와 시간</h3>
 사람은 보통 주,날짜,시간,분으로 날짜와 시간을 계산한다.
 Instant 클래스는 유닉스 에포크 시간을 기준으로 특정 지점까지의 사간을 초로 표현한다.
 
 Instant 클래스 인스턴스를 만들수 있다.
 오버로드된 ofEpochSecond 메서드 버전에서는 두번째 인수를 이용해서 나노초 단위로 시간을 보정할 수 있다.
 두번째 인수에는 0에서 999,999,999 사이의 값을 지정할 수 있다.
 다음 네가지 ofEpochSecond호출 콛는 같은 Instant를 반환한다
 
 
