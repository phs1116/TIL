## 2장 객체의 생성과 삭제

### 규칙 3. private 생성자나 enum 자료형은 싱글턴 패턴을 따르도록 설계하라.



<br>



#### 싱글턴 (Singleton)

객체를 하나만 만들수 있는 클래스. 보통 유일할 수 밖에 없는 시스템 컴포넌트를 나타낸다.

클래스를 싱글턴으로 만들 경우, 클라이언트를 테스트하기 어려워 질 수 있다는 단점이 있다.



<br>



#### JDK 1.5 이전 싱글턴을 구현하는 방법.

- **정적 멤버**

   ```java
   public class Elvis {
     public static final Elvis INSTANCE = new Elvis();
     private Elvis() { ... }
     
     public void leaveTheBuilding() { ... }
   }
   ```

   private 생성자는 `Elvis.INSTANCE`를 초기화 할 때 한번만 호출되고, public 혹은 protected 생성자가 없으므로 Elvis는 private 생성작 생선한 객체 하나만 존재하게 된다.

   다만 위와 같은 경우에는 `Accessibleobject.setAccessible`메서드를 통해 리플렉션으로 private 생성자에 대한 접근 권한을 얻을 경우, 생성자를 여러번 호출 할 수 있다는 문제점이 있다. 

   이 경우, 두번째 객체를 생성할 경우 예외를 던지는 방어로직이 들어가도록 생성자를 수정하여야 한다.

   ​

   <br>

   ​

- **정적 팩토리 메서드**

   ```java
   public class Elvis {
     private static final Elvis INSTANCE = new Elvis();
     private Elvis(){ ... }
     public static Elvis getInstance() { return INSTANCE; }
     
     public void leaveTheBuilding(){ ... }
   }
   ```

   `Elvis.getInstance`는 항상 같은 객체에 대한 참조를 반환한다. `Elvis.getInstance`를 이용한 경우 외에는 Elvis 객체를 생성 할 수 없다.

   > 하지만 이 경우에도 앞선 공격은 가능하므로 이전 방식과 같이 방어로직을 작성해주어야 한다.

   이 방법의 경우 최근 JVM이 정적 팩터리 메서드 호출을 거의 항상 인라인(inline) 처리를 하기 때문에 1번과 비교해 성능상의 큰 차이점은 없다.

   다만 이 방법은 싱글턴을 포기 할 경우 조금 더 유연하게 대처할 수 있다. 또한 제네릭 타입을 수용하기 쉽다는 장점을 가지고 있다.

   ​

   <br>

   ​

다만 위 방법들 같은 경우는 직렬화(Serializable) 클래스를 만들 경우, Serializable를 구현 하는 것 만으로는 싱글턴이 깨질 위험이 있다.

싱글턴을 유지 하기 위해서는 모든 필드를 transient로 선언하고, readResolve 메서드를 추가해야한다. 

그렇지 않을 경우 직렬화 후 역직렬화 시에 매번 새로운 객체를 생성하게 된다.

```java
private Object readResolve(){
  //동일한 객체를 반환하면서, 가짜 객체는 GC가 처리하도록 한다.
  return INSTANCE;
}
```



<br>



#### JDK 1.5 이후 싱글턴을 구현하는 방법

- **enum 타입**

  ```java
  public enum Elvis {
    INSTANCE;
    
    public void leaveTheBuilding() { ... }
  }
  ```

  Effective JAVA에서 제안하는 가장 좋은 싱글턴 패턴 구현 방법으로, 

  기능적으로는 public 필드를 사용하는 구현법과 비슷하나, 좀 더 간결하고 직렬화가 자동으로 처리된다는 장점이 있다.

  또한 리플렉션을 통한 공격에도 안전하다.

  ​

<br>



>  TODO : 여러가지 싱글턴 패턴에 대해서도 따로 정리해보자.