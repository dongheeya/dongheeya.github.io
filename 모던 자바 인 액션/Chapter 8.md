<h1>Chapter 8. 컬렉션 API 개선<h1>
  <h2> 8.1 컬렉션 팩토리 </h2>
  
  List<String> friends = new ArrayList<>();  
  friends.add("AA");
  friends.add("BB");  
  friends.add("CC");  
  
  세 문자열을 저장하는 데도 많은 코드가 필요하다.
  다음과 같이 Arrays.asList 팩토리 메서드를 이용하면 간단하게 줄일수 있다.
  
  List<String> friends = Arrays.asList("AA","BB","CC");
  
  <u>하지만 고정크기의 리스트를 만들었으므로 갱신할수은 있지만, 수정/삭제는 불가능하다.  </u>
  수정/갱신하려고하면 UnsupportedOperationException이 발생한다.  
  
  집합인 경우에는 어떨까?  
  리스트를 인수로 받는 HashSet를 이용할 수 있다.  
  Set<String> friends = new HashSet<>(Arrays.asList("aa","bb","cc"));  
  또는 다음처럼 스트림 API를 사용할 수 있다.  
  
  Set<String> friends = Stream.of("aaa","bbb","ccc").collect(Collector.toSet());  
  
  하지만 두 방법 모두 내부적으로 불필요한 객체 할당을 필요로 하며, 결과를 반환할 수 없다.  
  
  
  <h3> 8.1.1 컬렉션 팩토리 </h3>
  List.of 팩토리 메소드를 이용해서 간단하게 리스트를 만들 수 있다.  
  
  List<String> friends = List.of("aaa","bb","cc");  
  
  해당 팩토리 메서드를 통해서 생성된 List객체는 변경 불가능하다. 이렇게 제약을 줌으로써 컬렉션이 의도치 않게 변하는 것을 막도록 한다.  
  
  <h3> 8.1.2 집합 팩토리 </h3>
  Set<String> friends = Set.of("aaa","bb","cc");(o)  
  Set<String> friends = Set.of("aaa","bb","aaa");(x) -> Set은 오직 고유의 요소만 포함할 수 있다는 원칙이 존재한다.  
  
  <h3> 8.1.3 맵 팩토리 </h3>
  Map<String, Integer> ageOfFriends = Map.of("Raphael",10,"Olivia",5,"Thibaut", 7);  <- Olivia=5,Raphael=10,Thibaut=7
                                                                                        
  열 개 이하의 키와 값 쌍을 가진 작은 맵을 만들 때는 이 메소드가 유용하다.  
  Map.Entry<K, V> 객체를 인수로 받으며 가변 인수로 구현된 <u>Map.ofEntries</u> 팩토리 메서드를 이용하는 것이 좋다.  
  
  import static java.util.Map.entry;  
  Map<String,Integer> ageOfFriends = Map.ofEntries(entry("Raphael",10),  
  ("Olivia",5),  
  ("Thibaut", 7));  
  
 <h2> 8.2 리스트와 집합 처리 </h3>
   자바 8에서는 List, Set 인터페이스에 다음과 같은 메서드를 추가했다.
   removeIf : 프레디케이트를 만족하는 요소를 제거한다. List나 set을 구현하거나 그 구현을 상속받는 모든 클래스에서 이용할 수 있다.  
   replaceALL : 리스트에서 이용할 수 있는 기능으로 UnaryOperator 함수를 이용해 요소를 바꾼다  
   sort : List 인터페이스에서 제공하는 기능으로 리스트를 정렬한다.
  
  <h3> 8.2.1 removeIf 메서드 </h3>
  removeIf가 추가된 이유를 살펴보자.  
  아래의 코드는 ConcurrentModificationException을 발생시킨다.  
  for-each 구문은 내부적으로 Iterator 객체를 사용하기 때문에 Iterator 객체, Collection 객체 총 2개의 객체가 컬렉션을 관리한다.  
  따라서 반복자의 상태가 컬렉션의 상태와 서로 동기화되지 않게된다.  

  for (Transaction transaction : transactions) {
    if (Character.isDigit(transaction.getReferencecode().charAt(0))) {  
        transactions.remove(transaction);  
    }  
  }
  이를 해결해보면 아래와 같이 할 수 있는데 코드가 복잡해졌다.

  for (Iterator<Transaction> iterator = transactions.iterator(); iterator.hasNext(); ) {  
    Transaction transaction = iterator.next();  
    if (Character.isDigit(transaction.getReferenceCode().charAt(0))) {  
        iterator.remove();  
    }  
  }
  이를 removeIf를 사용해서 간결하게 바꿀 수 있다.

  transactions.removeIf(transaction -> Character.isDigit(transaction.getReferenceCode().charAt(0)));
  
  <h3> 8.2.2 replaceAll 메서드 </h3>
  List의 각 요소를 새로운 요소로 바꿀 수 있다.  
  
  List의 요소를 바꿀때도 Iterator 객체, 컬렉션 변경을 혼용해서 사용하면 문제가 일어날 수 있다. 이것을 replaceAll로 쉽게 해결할 수 있다.

  referenceCodes.replaceAll(code -> Character.toUpperCase(code.charAt(0)) + code.substring(1));

<h2> 8.3 맵 처리 </h2>  
  <h3> 8.3.1 forEach메서드 </h3>  
  
  맵에서 키와 값을 반복하면서 확인하는 작업은 귀찮은 작업 중 하나이다.  
  Map.Entry<K,V>의 반복자를 이용하면 맵의 항복 집합을 반복 할 수 있다.  
  
  for(Map.Entry<String, Integer> entry; ageOfFriends.entrySet()){  
    String friend = entry.getKey();  
    Integer age = entry.getValue();  
    System.out.println(friend+" is "+ age+" years old");
  }
  
  이를 자바 8에서는
  ageOfFriends.forEach((friend, age) -> System.out.println(friend+" is "+ age+" years old"));
  
  <h3> 8.3.2 정렬 메서드 </h3>  
  다음 두 개의 새로운 유틸리티를 이용하면 맵의 항복을 값 또는 키를 기준으로 정렬할 수 있다.
  
  - Entry.comparingByValue : value 기준으로 정렬
  - Entry.comparingByKey  : key 기준으로 정렬
  Map<String, String> favoriteMoview = Map.ofEntries(entry("Raphael","Star Wars"), entry("Cristina","Matrix"), entry("Olivia","James Bond"));
  
  favoriteMovies.entrySet().stream()  
    .sorted(Entry.comparingByKey())  
    .forEachOrdered(System.out::println);  
 결과는   
  Cristina=Matrix  
  Olivia=James Bond  
  Raphael=Star Wars
 
 <h3> 8.3.3 getOrDefault 메서드 </h3>  
  기존에는 찾으려는 키가 존재하지 않으면 널이 반환되므로 NullPointerException을 방지하려면 요청 결과가 널인지 확인해야한다.  
  getOrDefault를 이용하면쉽다.  
  <u>첫번째 인수로 키를, 두번째 인수로 기본값을 받으며 맵에 키가 존재하지 않으면 두 번쨰 인수로 받은 기본값을 반환한다.</u>
  
  Map<String, String> favoriteMovies =   
    Map.ofEntries(  
        entry("Raphael", "Star Wars"),  
        entry("Olivia", "James Bond")  
    );  
  
  System.out.println(favoriteMovies.getOrDefault("Olivia","Matrix")); <- James Bond 출력  
  System.out.println(favoriteMovies.getOrDefault("Thibaut","Matrix")); <- Matrix 반환
  key가 존재하더라도 저장된 값이 null이면 null을 반환한다. 
 
 <h3> 8.3.4 계산 패턴 </h3>  
 Map에 key가 존재하는지 여부에 따라 어떤 동작을 실행하고 결과를 저장해야하는 상황일 때 사용하는 패턴이다.  

 - computeIfAbsent: 제공된 key에 해당하는 값이 없거나 null이면, key를 이용해 새 값을 계산하고 맵에 추가한다. 정보를 캐시할 때 사용가능하다.
 - computeIfPresent: 제공된 key가 존재하면 새 값을 계산하고 맵에 추가한다.
 - compute: 제공된 key로 새 값을 계산하고 맵에 저장한다.
  
 <h3> 8.3.5 삭제 패턴 </h3>   
 자바 8에서는 키가 특정한 값과 연관되었을 때만 항목을 제거하는 오버로드 버전 메서드를 제공한다.
  
  ex) favoriteMovies.remove(key, value); // map의 키가 Raphael이면서 value가 Star 인 경우만 삭제
  
 <h3> 8.3.6 교체 패턴 </h3>   
  
 - replaceAll: BiFunction을 적용한 결과로 각 항목의 값을 교체한다.  
 - replace: key가 존재하면 map의 value를 바꾼다. key가 특정 value로 매핑되었을 때만 value를 교체하는 버전의 메서드도 있다.
 
  ex) favoriteMovies.replaceAll((friend, movie) -> movie.toUpperCase());  <- 모든 value를 대문자로 변경
  ex) favoriteMovies.replace("Raphael", "harry potter");   
      favoriteMovies.replace("Olivia", "james bond", "Harry Potter");
  
 <h3> 8.3.7 합침 </h3>   
  두 그룹의 연락처를 포함하는 맵을 합친다고 가정하자. 
  <b>이때에는 putAll를 활용한다.</b>
  
  Map<String, String> family = Map.ofEntries(  
        Map.entry("Teo", "Star Wars"),  
        Map.entry("Cristina", "James Bond")  
  );
  Map<String, String> friends = Map.ofEntries(  
        Map.entry("Raphael", "Star Wars")  
  );
        
  Map<String, String> everyone = new HashMap<>(family);
  everyone.putAll(friends);
  
  <u>중복된 key가 없다면 문제없이 동작한다.</u>
  merge메서드는 중복된 key를 어떻게 합칠지 결정하는 BiFunction을 인수로 받아 중복된 key가 존재하는 map을 합칠 수 있게 해준다.

  Map<String, String> family = Map.ofEntries(Map.entry("Teo", "Star Wars"),Map.entry("Cristina", "James Bond"));  
  Map<String, String> friends = Map.ofEntries(Map.entry("Raphael", "Star Wars"),Map.entry("Cristina", "Matrix"));   
  
  이렇게 Cristina 가 family 와 friends에 모두 있을 경우에는 다음과 같이 충돌을 해결할 수 있다.  
  Map<String, String> everyone = new HashMap<>(family);
  friends.forEach((key, value) -> everyone.merge(key, value, (movie1, movie2) -> movie1 + " & " + movie2));

 <h2>8.4 개선된 ConcurrentHashMap</h2>
  ConcurrentHashMap 클래스는 동시성 친화적이며 최신 기술을 반영한 HashMap 버전이다.  
  내부 자료구조의 특정 부분만 잠궈 동시 추가, 갱신 작업을 허용한다.  

 <h3>8.4.1 리듀스와 검색</h3>
  - forEach: 각 (key, value) 쌍에 주어진 액션을 실행  
  - reduce: 모든 (key, value) 쌍을 제공된 리듀스 함수를 이용해 결과로 합침  
  - search: null이 아닌 값을 반환할 때까지 각 (key, value) 쌍에 함수를 적용  
  
  다음처럼 키에 합수 받기, 값, Map.Entry, (키, 값) 인수를 이용한 네 가지 연산 형태를 지원한다.  
  - key, value로 연산 (forEach,reduce,search)
  - key로 연산(forEachKey,reduceKeys,searchKeys)
  - value로 연산(forEachValue,reduceValues,searchValues)
  - Map.Entry 객체로 연산(forEachEntry,reduceEntries,searchEntries)
 
  <b>각 연산은 ConcurrentHashMap의 상태를 잠그지 않고 연산을 수행한다. </b>
  따라서 이 연산에 제공한 함수는 계산이 진행되는 동안 바뀔 수 있는 객체, 값, 순서 등에 의존하지 않아야한다.

  병렬성 기준값을 지정해야한다.
  맵의 크기가 주어진 기준값보다 작다면 순차적으로 연산 실행. 기준값을 1로 주면 공통 스레드 풀을 이용해 병렬성 극대화한다.

  int, long, double 등 기본값에는 특화연산(reduceValuesToInt 등)이 존재하므로 박싱작업이 필요없다.

 
  <h3>8.4.2 계수</h3>
  map의 매핑 개수를 반환하는 mappingCount 메서드를 제공.  

  <h3>8.4.3 집합 뷰</h3>
  ConcurrentHashMap을 set으로 반환하는 KeySet 메서드 제공. map을 바꾸면 set도 바뀌고 set을 바꾸면 map도 영향을 받는다.  
  ConcurrentHashMap을.newKeySet()으로 ConcurrentHashMap으로 유지되는 set을 만들수도있다.  
  
  


  
  
  
  
    
