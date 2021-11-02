<h1>디폴트 메서드</h1>
자바 8에서는 기본 구현을 포함하는 인터페이스를 정의하는 두 가지 방법을 제공한다. <br/>
1) 정적 메서드 : 인터페이스 내부에 사용<br/>
2) 디폴트 메서드 : 기존 코드의 구현을 바꾸도록 강요하지 않으면서도 인터페이스를 바꿀 수 있다.<br/><br/>

```
default void sort(Comparator<? super E> c){
  Collection.sort(this, c);
}
```

변환 형식 void 앞에 default라는 새로운 키워드가 등장했다.<br/>
default 키워드는 해당 메서드가 디폴트 메서드임을 가리킨다.<br/>

디폴트 메서드는 주로 라이브러리 설계자들이 사용한다.<br/>
디폴트 메서드를 사용하면 자바 API의 호환성을 유지하면서 라이브러리를 바꿀 수 있다.<br/>

![13-1 인터페이스에 메서드 추가](https://user-images.githubusercontent.com/87962572/139839147-4550b426-f31e-4754-a772-7ef1a85ca4ea.PNG)

인터페이스를 사용하게 되면, 새로 추가된 메서드를 구현하도록 인터페이스를 구현하는 기존 클래스를 고쳐야 했기 때문이다.<br/>
디폴트 메서드를 사용하면 인터페이스의 기본 구현을 그대로 상속하므로 인터페이스에 자유롭게 새로운 메서드를 추가할 수 있게 된다.<br/>

<h2>변화하는 API</h2>

```
public interface Resizeable extends Drawable { 
  int getWidth();
  int getHeight();
  void setWidth(int width);
  void setHeight(int height);
  void setAbsoluteSize(int width, int height);
}
```

라이브러리를 즐겨 이용하는 사용자 중 한 명은 직접 Resizable를 구현하는 Ellipse 클래스를 만들었다.
```
public class Ellipse implements Resizable{

}
```

```
public class Game{
  public static void main(String...args){
    List<Resizable> resizableShapes = Arrays.asList(new Square(), new Rectangle(), new Ellipse());
    Utils.paint(resizableShapes);
  }
}

public class Util {
  public static void paint(List<Resizable> l){
    l.forEach(r -> {
      r.setAbsoluteSize(42,42); ☜ 각 모양에 setAbsoluteSize 호출
      r.draw();
    }
  }
}
```

![13-2](https://user-images.githubusercontent.com/87962572/139841542-feb0ef37-6c9d-45fa-9397-9e0f6b12aaab.PNG)

몇 개월이 지나자 Resizeable를 구현하는 Square과 Rectangle 구현을 개선해달라는 많은 요청을 받았다.

```
public interface Resizeable extends Drawable { 
  int getWidth();
  int getHeight();
  void setWidth(int width);
  void setHeight(int height);
  void setAbsoluteSize(int width, int height);
  void setRelativeSize(int wFactor, int hFactor); ☜ API 버전2에 추가된 새로운 메서드
}
```

<h3>사용자가 겪는 문제</h3>
Resizeable을 고치면 몇 가지 문제가 발생한다.<br/>
1) Resizeable을 구현하는 모든 클래스는 setRelativeSize메서드를 구현해야 한다.<br/>
하지만 라이브러리 사용자가 직접 구현한 Ellipse는 setRelativeSize 메서드를 않는다.<br/>
인터페이스에 새로운 메서드를 추가하면 바이너리 호환성은 유지된다.<br/>
<b>* 바이너리 호환성이란, 새로 추가된 메서드를 호출하지만 않으면 새로운 메서드 구현이 없이도 기존 클래스 파일 구현이 잘 동작한다는 의미다.</b><br/>

Ellipse 객체가 인수로 전달되면 Ellipse는 setRelativeSize메서드를 정의하지 않았으므로 런타임에 다음과 같은 에러가 발생할 것이다.<br/>

```
Exception in thread "main" java.lang.AbstractMethodError: lambdasinaction.chap9.Ellipse.setRelativeSize(II)V
```

공개된 API를 고치면 기존 버전과의 호환성 문제가 발생한다.<br/>
□ 라이브러리를 관리하기가 복잡하다.
□ 사용자는 같은 코드에 예전 버전과 새로운 버전 두 가지 라이브러리를 모두 사용해야 하는 상황이 생긴다.
□ 프로젝트에서 로딩해야 할 클래스 파일이 많아지면서 메모리 사용과 로딩 시간 문제가 발생한다.

-> 이러한 문제점을 "디폴트 메서드"로 해결할 수 있다.
"디폴트 메서드"를 이용해서 API를 바꾸면 새롭게 바뀐 인터페이스에서 자동으로 기본 구현을 제공하므로 기존 코드를 고치지 않아도 된다.

<h3>바이너리 호환성, 소스 호환성, 동작 호환성</h3>
1. 바이너리 호환성 : 무언가를 바꾼 이후에도 에러 없이 기존 바이너리가 실행될 수 있는 상황을 의미한다.
2. 소스 호환성 : 코드를 고쳐도 기존 프로그램을 성공적으로 재컴파일할 수 잇음을 의미한다.
3. 코드를 바꾼 다음에도 같은 입력값이 주어지면 프로그램이 같은 동작을 실행하다는 의미다.

<h2>디폴트 메서드란 무엇인가?</h2>
디폴트 메서드 : 인터페이스를 구현하는 클래스에서 구현하지 않은 메서드는 인터페이스 자체에서 기본으로 제공한다.
default라는 키워드로 시작하며 다른 클래스에 선언된 메서드처럼 메서드 바디를 포함한다.

ex) 컬렉션 라이브러리에 Sized라는 인터페이스를 정의했다고 가정하자. 다음 코드에서 보여주는 것처럼 Sized 인터페이스는 추상 메서드 size와 디폴트 메서드 isEmpty를 포함한다.
```
public interface Sized {
  int size;
  default boolean isEmpty(){   ☜ 디폴트 메서드
    return sized() == 0;
  }
}
```

이제 Sized 인터페이스를 구현하는 모든 클래스는 isEmpty 의 구현도 상속받는다.
즉, 인터페이스에 디폴트 메서드를 추가하면 소스 호환성이 유지된다.

<h3>선택형 메서드</h3>
디폴트 메서드를 이용하면 remove같은 메서드에 기본 구현이 제공할 수 있으므로 인터페이스를 구현하는 클래스에서 빈 구현을 제공할 필요가 없다.

```
interfae Iterator<T> {
  boolean hasNext();
  T next();
  default void remove(){
    throw new UnsupportedOperationException();
  }
}
```

기본 구현이 제공되므로 Iterator 인터페이스를 구현하는 클래스는 빈 remove 메서드를 구현할 필요가 없어졌고, 불필요한 코드를 줄일 수 있다.

<h3>동작 다중 상속</h3>
디폴트 메서드를 이용하면 기존에는 불가능했던 동작 다중 상속 기능을 구현할 수 있다.

![13-3](https://user-images.githubusercontent.com/87962572/139844498-d4fc5292-2dd3-47e8-b0cf-297dd7fc8301.PNG)

자바 클래스는 한 개의 다른 클래스만 상속할 수 있지만 인터페이스는 여러 개 구현할 수 있다.
```
public class ArrayList<E> extends AbstractList<E>  ☜  한개의 클래스를 상속받는다.
       implements List<E>, RandomAccess, Cloneable, Serializable { ☜  4개의 인터페이스를 구현한다.
}
```

<b>다중 상속 형식</b><br/>
ArrayList는 List<E>, RandomAccess, Cloneable, Serializable, Iterable, Collection의 서브 형식이 된다.<br/>
따라서 디폴트 메서드를 사용하지 않아도 다중 상속을 활용할 수 있다.<br/>
  
<b>기능이 중복되지 않는 최소의 인터페이스</b>

먼저 setRotationAngle 과 getRotationAngle 두 개의 추상 메서드를 포함하는데 Rotatable 인터페이스를 정의한다.
인터페이스는 다음 코드에서 보여주는 것처럼 setRotationAngle과 getRotationAngle 메서드를 이용해서 디폴트 메서드 rotateBy도 구현한다.

  ```
  public interface Rotatable {
    void setRotationAngle(int angleInDegrees);
    int getRotationAngle();
    default void rotateBy(int angleInDegrees){
      setRotationAngle((getRotationAngle() + angleInDegrees) % 260);
    }
  }
  ```
  
  setRotationAngle과 getRotationAngle의 구현을 제공해야하지만 rotateBy는 기본 구현이 제공되므로 따로 구현이 제공하지 않아도 된다.
  마찬가지로 이전에 살펴본 두 가지 인터페이스 Moveable과 Resizable을 정의해야한다.
  
  <b>인터페이스 조합</b>
  Monster 클래스는 Rotatable, Moveable, Resizable 인터페이스의 디폴트 메서드를 자동으로 상속받는다.  
  
![13-4](https://user-images.githubusercontent.com/87962572/139849660-227ceed5-f2cb-492c-abf5-aa27d407c16e.PNG)

  예를 들어, moveVertically의 구현을 더 효율적으로 고쳐야 한다고 가정하자.
  디폴트 메서드 덕분에 Moveable 인터페이스를 직접 고칠 수 있고 따라서 Moveable을 구현하는 모든 클래스도 자동으로 변경한 코드를 상속한다.
  
  <h3>해석 규칙</h3>
  자바 클래스는 하나의 부모 클래스만 상속받을 수 있지만 여러 인터페이스를 동시에 구현할 수 잇다.
  
  ```
  public interface A {
    default void hello(){
      System.out.println("Hello from A");
    }
  }
  
  
  public interface B extends A {
    default void hello(){
      System.out.println("Hello from B");
    }
  }
  
  
  public class C implements B, A {
    public static void main(String...args){
      System.out.println("Hello from C");
    }
  }
  ```
  
  <h3>3가지 규칙</h3>
  다른 클래스나 인터페이스로부터 같은 시그니처를 갖는 메서드를 상속받을 때는 세가지 규칙을 따라야한다.
  1. 클래스가 항상 이긴다. 클래스나 슈퍼클래스에서 정의한 메서드가 디폴트 메서드보다 우선권을 갖는다.
  2. 1번 규칙 이외의 상황에서는 "서브 인터페이스"가 이긴다. 상속관계를 갖는 인터페이스에서 같은 시그니처를 갖는 메서드를 정의할 때는
  서브 인터페이스가 이긴다. 즉, B가 A를 상속받는다면 B가 A를 이긴다.
  3. 여러 인터페이스를 상속받는 클래스가 명시적으로 디폴트 메서드를 오버라이드하고 호출해야한다.
  
  ![13-5](https://user-images.githubusercontent.com/87962572/139851080-f1177074-e42a-4d5c-bbfe-33225fbb2178.PNG)

  B와 A는 hello라는 디폴트 메서드를 정의한다.
  또한 B는 A를 상속받는다.
  
  컴파일러는 누구의 hello 메서드 정의를 사용할까?
  2번 규칙에 의해서 B가 A를 상속받았으므로 컴파일러는 B의 hello를 선택한다.
  ☞ Hello from B
  
  ![13-6](https://user-images.githubusercontent.com/87962572/139851445-e5b89a2f-11d2-44df-9cf6-468cdb4533cd.PNG)
  
  1번 규칙은 클래스의 메서드 구현이 이긴다고 설명한다.<br/>
  D는 hello를 오버라이드하지 않았고 단순히 인터페이스 A를 구현했다.<br/>
  따라서 D는 인터페이스 A의 디폴트 메서드 구현을 상속받는다.<br/>
  2번 규칙에서는 슈퍼클래스나 슈퍼클래스에 메서드 정의가 없을 때 디폴트 메서드를 정의하는 서브 인터페이스가 선택된다.<br/>
  따라서, 컴파일러는 인터페이스 A의 hello나 인터페이스 B의 hello 둘 중 하나를 선택해야 한다.<br/>
  여기서 B가 A가 상속받는 관계이므로 이번에도 'Hello from B' 가 출력된다.<br/>
  
  <h3>충돌 그리고 명시적인 문제 해결</h3>
  
  지금까지는 1번 2번과 같은 규칙으로 해결할 수 있었다.
  
![13-7](https://user-images.githubusercontent.com/87962572/139852826-4672beea-cecc-4bf9-a4fc-bf45ef12212d.PNG)
  
  인터페이스 간에 상속관계가 없으므로 2번 규칙을 적용할 수 없다.
  그러므로 A와 B의 hello 메서드를 구별할 기준이 없다.
  자바 컴파일러는 어떤 메서드를 호출해야 할지 알수 없으므로 "Error:class C inherits unrelated defaults for hello() from types B and A."
  
  이렇게 구별할 기준이 없을 경우에는 "명시적 선택"을 해야한다.
  
  ```
  public class C implements B, A{
    void hello(){
      B.super.hello();
    }
  }
  ```

  <h3>다이아몬드 문제</h3>
  
  ```
  public interface A {
    default void hello(){
      System.out.println("Hello from A");
    }
  }
  
  public interface B extends A {}
  public interface C extends A {}
  public class D implements B, C {
    public static void main(String... args){
      new D().hello();
    }
  }
  ```
  
  ![13-8](https://user-images.githubusercontent.com/87962572/139853624-6426d9a9-50a2-4fcd-98f2-b1e1ab20f8cc.PNG)

  다이어그램 모양이 다이아몬드를 닮아서 이를 "다이아몬드 문제"라고 부른다
  
  D는 B와 C 중 누구의 디폴트 메서드를 정의 상속받을까? 실제로 메서드 선언은 A뿐이므로 A의 디폴트 메서드가 출력된다.
  ☞ Hello from A
  
  
