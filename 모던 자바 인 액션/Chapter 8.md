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
  Map.Entry<K, V> 객체를 인수로 받으며 가변 인수로 구현된 Map.ofEntries 팩토리 메서드를 이용하는 것이 좋다.  
  
  import static java.util.Map.entry;  
  Map<String,Integer> ageOfFriends = Map.ofEntries(entry("Raphael",10),  
  ("Olivia",5),  
  ("Thibaut", 7));
  

  
  
  
  
  
    
