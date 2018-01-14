# Effective Java

## 2장 객체의 생성과 삭제

### 규칙 1.  생성자 대신 정적 팩터리 메서드를 사용할수 없는지 고민하자.

객체를 생성할때 일반적으로 public으로 선언되어있는 생성자를 사용한다. 

하지만 생성자 외에 static factory method(정적 팩토리 메서드)를 사용해서 객체를 생성 할 수 있다.



#### 정적 팩토리 메서드의 장점

1. 생성자와 달리 static factory method에는 **이름이 존재한다. **

   해당 메서드가 어떤 객체를 생성하는지 알 수 있어 가독성을 높일수 있다.

2. 생성자와 달리 호출 될 때마다 **새로운 객체를 생성할 필요가 없다.**

   - 이뮤터블 클래스일 경우 이미 만든 객체를 활용할 뿐만 아니라 객체를 캐시하여 재사용 하여 객체를 불필요하기 생성하는 것을 피할 수 있다.
   - 특정시점에 객체가 얼마나 존재하는지 제어할수 있다.(instance-controlled class). 이를 통해 싱글턴 패턴 클래스, 객체 생성이 불가능한 클래스 생성 등을 만들수 있다. 또한 이뮤터블 클래스일 경우 같은 객체가 존재하지 않도록 할 수 있다.

3. 생성자와 달리 자료형의 **하위 자료형 객체를 반환 할 수 있다.**

   - 반환되는 객체의 클래스를 유영하게 결정 할 수 있다.

   - 구현 세부 사항을 감출수 있으므로 간결한 API가 가능하며 이는 인터페이스 기반 프레임 워크에 적합하다.

   - 대표적인 예 : JDK 1.5에 도입된 EnumSet -

     EnumSet은 public 생성자가 없으며 정적 팩토리 메서드만 가지고있다.

     EnumSet은 정적 팩토리 매서드로 두가지 구현체 중 하나를 반환한다.

     - RegularEnumSet : enum 상수들이 64개 이하일경우, enum의 갯수를 long 변수 하나만 사용해 표현한다.
     - JumboEnumSet : enum상수들이 64개 이상일 경우, enum의 갯수를 long형 배열을 사용해 표현한다.

   - Service Provider Framework : 정적 팩토리 메서드가 작성된 시기에 반환될 구현체 클래스가 존재하지 않아도 무방하다.

     - 대표적인 예 :  JDBC
     - Service Provider Framework는 세가지 핵심 컴포넌트로 구성된다.
       - Service Interface : 서비스 제공자가 구현한다. (JDBC : Connection)
       - Provider Registration API : 구현체를 서비스에 등록하여 클라이언트가 사용 가능하도록한다. (JDBC : DriverManger.registDriver)
       - Service Access API : 클라이언트에게 서비스 구현체를 제공한다.(DriverManger.getConnection)
       - Service Provider Interface : 선택적이며, 서비스 제공자가 구현한다. 서비스 구현체의 객체를 생성하기 위한 인터페이스이다. 이가 없으면 자바의 리플렉션등으로 객체가 생성된다.(JDBC : Driver)

4. **형인자 자료형(parameterized type) 객체를 만들 때 편리하다.**

   형인자 자료형 클래스의 생성자를 호출할 때는, 문맥상 형인자가 명백하더라도 반드시 인자로 형인자를 전달해야한다.

   ```java
   Map<String, List<String>> m = new HashMap<String, List<String>>();
   ```

   이처럼 형인자가 늘어남에 따라 길고 복잡한 코드가 만들어진다. 하지만 정적 팩토리 메서드로 컴파일러가 형인자를 유추하도록 할 수 있다. 이를 자료형 유추(type interface)라고 부른다. 

   ```java
   public static <K, V> HashMap<K, V> newInstance(){
     return new HashMap<K, V>();
   }
   Map<String, List<String>> m = HashMap.newInstance();
   ```

> JDK 1.7 이후부터는 생성자를 이용해서도 형인자 유추가 가능하다.



#### 정적 팩토리 메서드의 단점

1. 정적 팩토리 메서드만 있는 클래스는 public이나 protected로 선언된 생성자가 없으므로 하위 클래스를 만들 수 없다.
2. 정적 팩토리 메서드가 다른 정적 메서드와 확연히 구분되지 않는다.

#### 요약

정적 팩터리 메서드와 public 생성자는 용도가 서로 다르다. 그 차이점과 장단점을 이해하고 정적 팩토리 메서드가 효과적인 경우가 많으니 무조건 public 생성자만 만드는 것은 삼가자.
