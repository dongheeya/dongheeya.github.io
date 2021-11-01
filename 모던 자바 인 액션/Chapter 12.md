<h1>Chapter 12. 새로운 날짜와 시간 API</h1>
 자바 API는 복잡한 애플리케이션을 만드는 데 필요한 여러 가지 유용한 컴포넌트를 제공한다.<br/>
 자바 1.0에서는 java.util.Date 클래스 하나로 날짜와 시간 관련 기능을 제공했다.<br/>
 Date라는 이름과 달리 Date클래스는 특정 시점을 미리초 단위로 표현했다.<br/>
 
 게다가 1900년을 기준으로 하는 오프셋, 0에서 시작하는 달 인덱스 등 모호한 설계로 유용성이 떨어졌다.<br/>
 
 ex) 2017년 9월 21일 
 
 ```
 Date date = new Date(117,8,21);
 ```
 
 <h2>LocalDate, LocalTime, Instant, Duration, Period 클래스</h2>
 <h3>12.1.1 LocalDate와 LocalTime 사용</h3>
 LocalDate 인스턴스는 시간을 제외한 날짜를 표현하는 불변 객체다.<br/>
 특히, LocalDate 객체는 어떤 시간대 정보도 포함하지 않는다.<br/>
 
 정적 팩토리 메서드 of로 LocalDate 인스턴스를 만들 수 있다.<br/>
 
 ```
 LocalDate date = LocalDate.of(2017,9,21); ☜ 2017-09-21
 int year = date.getYear();                ☜ 2017
 Month month = date.getMonth();            ☜ SEPTEMBER
 int day = date.getDayOfMonth();           ☜ 21
 DayOfWeek dow = date.getDayOfWeek();      ☜ Thursday
 int len = date.lengthOfMonth();           ☜ 31 (3월의 일 수)
 boolean leap = date.isLeapYear();         ☜ false (윤년이 아님)
 ```
  
 팩토리 메서드 now는 시스템 시계의 정보를 이용해서 현재 날짜 정보를 얻는다.<br/>
 
 ```
 LocalDate today = LocalDate.now();
 ```
 
 TemporalField는 시간 관련 객체에서 어떤 필드의 값에 접근할 지 정의하는 인터페이스다.<br/>
 get메서드에 TemporalField를 전달해서 정보를 얻는 방법도 있다.<br/>
 열거자 ChroneField는 TemporalField 인터페이스를 정의하므로 다음 코드에서 보여주는 것처럼 ChroneField 의 열거자 요소를 이용해서<br/>
 원하는 정보를 쉽게 얻을 수 있다.<br/>
 
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
 LocalDateTime은 LocalDate와 LocalTime을 쌍으로 갖는 복합 클래스다.<br/>
 즉, LocalDateTime은 날짜와 시간을 모두 표현할 수 있으며 다음 코드에서 보여주는 것처럼 직접 LocalDateTime을 만드는 방법도 있고<br/>
 날짜와 시간을 조합하는 방법도 있다.<br/>
 
 ```
 //2017-09-21T13:45:20
 LocalDateTime dt1 = LocalDateTime.of(2017,Month.SEPTEMBER,21,13,45,20);
 LocalDateTime dt2 = LocalDateTime.of(date,time);
 LocalDateTime dt3 = date.atTime(13,45,20);
 LocalDateTime dt4 = date.atTime(time);
 LocalDateTime dt5 = time.atDate(date);
 ```
 
 LocalDate의 atTime 메서드에 시간을 제공하거나 LocalTime의 atDate 메서드에 날짜를 제공해서 LocalDateTime을 만드는 방법도 있다.<br/>
 LocalDateTime의 toLocalDate나 toLocalTeimd메서드로 LocalDate나 LocalTime인스턴스를 추출할 수 있다.<br/>
 
 ```
 LocalDate date1 = dt1.toLocalDate(); ☜ 2017-09-21
 LocalTime time1 = dt1.toLocalTime(); ☜ 13:45:20
 ```
 
 <h3> Instant 클래스 : 기계의 날짜와 시간</h3>
 사람은 보통 주,날짜,시간,분으로 날짜와 시간을 계산한다.<br/>
 Instant 클래스는 유닉스 에포크 시간을 기준으로 특정 지점까지의 사간을 초로 표현한다.<br/>
 
 Instant 클래스 인스턴스를 만들수 있다.<br/>
 오버로드된 ofEpochSecond 메서드 버전에서는 두번째 인수를 이용해서 나노초 단위로 시간을 보정할 수 있다.<br/>
 두번째 인수에는 0에서 999,999,999 사이의 값을 지정할 수 있다.<br/>
 
 Instant는 차와 나노초 정보를 포함한다.
 따라서 Instant는 사람이 읽을 수 있는 시간 정보를 제공하지 않는다.
 
 <h3>Duration과 Period 정의</h3>
 Temporal 인터페이스를 구현하는데, Temporal 인터페이스는 특정 시간을 모델링하는 객체의 값을 어떻게 일고 조작할지 정의한다.<br/>
 Duration은 두 시간 객체 사이의 지속시간을 말한다.<br/>
 Duration 클래스의 정적 팩토리 메서드 between으로 두 시간 객체 사이의 지속시간을 만들 수 있다.<br/>
 
 ex) 두개의 LocalTime, 두 개의 LocalDateTime 또는 두 개의 Instant로 Duration을 만들 수 있다.
 ```
 Duration d1 = Duration.between(time1, time2);
 Duration d2 = Duration.between(dateTime1, dateTime2);
 Duration d3 = Duration.between(instant1, instant2);
 ```
 
 LocaldateTime은 사람이 사용하도록, Instant는 기계가 사용하도록 만들어진 클래스로 두 인스턴스는 서로 혼합할 수 없다.<br/>
 Duration 클래스는 초와 나노초로 시간 단위를 표현하므로 between메서드에 LocalDate를 전달할 수 없다.<br/>
 년,월,일로 시간을 표현할 때는 Period클래스를 사용한다.<br/>
 즉, Period클래스의 팰톨 메서드 between을 이용하면 두 LocalDate의 차이를 확인할 수 있다.<br/>
 
 ```
 Period tenDays = Period.between(LocalDate.of(2017,9,11), LocalDate.of(2017,9,21));
 ```
 
 <h3>TemporalAdjusters 사용하기</h3>
 때로는 다음주 일요일, 돌아오는 평일, 어떤 달의 마지막 날 등 좀 더 복잡한 날짜 조정 기능이 필요할 것이다.<br/>
 이때는 오버로드된 버전의 with메서드에 좀 더 다양한 동작을 수행하도록 하는 기능을 제공하는 TemporalAdjuster를 전달하는 방법으로 문제를 해결할 수 있다.<br/>
 
 ```
 LocalDate date1 = LocalDate.of(2014,3,18);
 LocalDate date2 = date1.with(nextOrSame(DayOfWeek.SUNDAY));
 LocalDate date3 = date2.with(lastDayOfMonth());
 ```
 
 다양한 TemporalAdjusters의 팩토리 메서드로 만들 수 있는 TemporalAdjster 리스트를 보여준다.<br/>
 ```
 dayOfWeekInMonth : 서수 요일에 해당하는 날짜를 반환하는 TemporalAdjster를 반환함
 firstDayOfMonth : 현재 달의 첫 번째 날짜를 반환하는 TemporalAdjuster를 반환함
 firstDayOfNextMonth : 다음 달의 첫번째 날짜를 반환하는 TemporalAdjuster를 반환함
 firstDayOfNextYear : 내년의 첫 번째 날짜를 반환하는 TemporalAdjuster를 반환함
 firstDayOfYear : 올해의 첫번째 날짜를 반환하는 TemporalAdjuster를 반환함
 firstInMonth : 현재 달의 첫번째 요일에 해당하는 날짜를 반환함
 lastDayOfMonth : 해당 월의 마지막 날짜를 리턴한다.
 lasyDayOfNextMonth : 해당 달의 마지막 날짜를 반환하는 TemporalAdjuster를 반환함
 lastDayOfNextYear :다음 달의 마지막 날짜를 반환하는 TemporalAdjuster를 반환함
 lastDayOfYear : 올해의 마지막 날짜를 반환하는 TemporalAdjuster를 반환함
 lastInMonth : 현재 달에서 현재 날짜 이후로 지정한 요일이 처음으로 나타나는 날짜를 반환하는 TemporalAdjuster를 반환함
 nextOrSame : 현재 날짜 이후로 지정한 요일이 처음/이전으로 나타나는 날짜를 반환함
 previousOrSame : TemporalAdjster를 반환함
 ```
 
 <h3>날짜와 시간 객체 출력과 파싱</h3>
 DateTimeFormatter = BASIC_ISO_DATE와 ISO_LOCAL_DATE 등의 상수를 미리 정의하고 있다.
 DateTimeFormatter를 이용해 날짜나 시간을 특정 형식의 문자열로 만들 수 있다.
 
 다른 두 개의 서로 다른 포매터로 문자열을 만드는 예제다.
 
 ```
 LocalDate date = LocalDate.of(2014,3,18);
 String s1 = date.format(DateTimeFormatter.BASIC_ISO_DATE);
 String s2 = date.format(DateTimeFormatter.ISO_LOCAL_DATE);
 ```
 
 반대로 날짜나 시간을 표현하는 문자열을 파싱해서 날짜 객체를 다시 만들 수 있다.
 ```
 LocalDate date1 = LocalDate.parse("20140418", DateTimeFormatter.BASIC_ISO_DATE);
 LocalDate date1 = LocalDate.parse("2014-04-18", DateTimeFormatter.ISO_LOCAL_DATE);
 ```
 
 패턴으로 DateTimeFormatter 만들기
 ```
 DateTimeFormatter formatter = DateTimeFormatter.ofPattern("dd/MM/yyyy");
 LocalDate date1 = LocalDate.of(2014,3,18);
 String formattedDate = date1.format(formatter);
 LocalDate date2 = LocalDate.parse(formattedDate, formatter);
 ```
 
 DateTimeFormatter  만들기
 
 ```
 DateTimeFormatter italianFormatter = new DateTimeFormatterBuilder()
 .appendText(ChronoField.DAY_OF_MONTH)
 .appendLiteral(". ")
 .appendText(ChronoField.MONTH_OF_YEAR_
 .appendLiteral(" ")
 .appendText(ChronoField.YEAR)
 .parseCaseInsensitive()
 .toFormatter(Locale.ITALIAN);
 ```
 
 <h2>다양한 시간대와 캘린더 활용 방법</h2>
 <h3>시간대 사용하기</h3>
 표준 시간이 같은 지역을 묶어서 시간대 규칙 집합을 정의한다.
 지역 ID로 특정 zoneId를 구분한다.
 
 ```
 ZoneId romeZone = ZoneId.of("Europe/Rome");
 ```
 
 지역 ID는 {지역}/{도시} 형식으로 이뤄지며 IANA TIME ZONE Database에서 제공하는 지역 집합 정보를 사용한다.
 
 ```
 ZoneId zoneId = TimeZone.getDefault().toZoneId();
 ```
 
<h3>UTC/Greenwich 기준의 고정 오프셋</h3>
때로는 UTC/GMT를 기준으로 시간대를 표현하기도 한다.
예를 들어 뉴옥은 런던보다 5시간 느리다.라고 표현할 수 있다.
ZoneId의 서브클래스인 ZoneOffset클래스로 런던의 그리니치 0도 자오선과 시간값의 차이를 표현할 수 있다.

```
ZoneOffset newYorkOffset = ZoneOffset.of("-05:00");
```

실제로 미국 동부 표준시의 오프셋값은 -05:00 이다.
하지만 위 예제에서 정의한 ZoneOffset으로는 서머타임을 제대로 처리할 수 없으므로 권장하지 않는 방식이다.

```
LocalDateTime dateTime = LocalDateTime.of(2014,Month.MARCH,18,13,45);
OffsetDateTime dateTimeInNewYork = OffsetDateTime.of(date, newYorkOffset);
```

<h3>대안 캘린더 시스템 사용하기</h3>
ISO-8601 캘린더 시스템은 실질적으로 전세계에서 통용된다. <br/>
자바8에서는 추가로 4개의 캘린더 시스템을 제공한다.<br/>
ThaiBuddhistDate, MinguoDate, JapaneseDate, hijrahDate 4개의 클래스가 각가의 캘린더 시스템을 대표한다.<br/>
시간과 날짜를 나타낼때에는 LocalDate를 사용하면 좋다.<br/>
최소한 1년은 정해진 수의 달로 이루어졌을 것이라고 가정할 수 있다.<br/>
이와 같은 가정은 특히 멀티캘린더 시스템에서는 적용되지 않는다.<br/>
프로그램의 입출력을 지역화하는 상황을 제외하고는 모든 데이터 저장, 조작, 비즈니스 규칙 해석 등의 작업에서 LocalDate를 사용해야 한다.<br/>

