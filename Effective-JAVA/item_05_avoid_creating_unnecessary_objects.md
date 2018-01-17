## 2장 객체의 생성과 삭제

### 규칙 5. 불필요한 객체의 생성을 피하라



<br>



#### Immutable 클래스의 경우

기능적으로 동일한 객체는 필요할 때 마다 만드는 것 보다 재사용하는 편이 낫다.

객체를 재사용 하면 더욱 빠르고 우아하며, 특히 immutable 객체는 언제나 재사용할 수 있기 때문에 추가 객체 생성보단 재사용을 고려하자.

```java
String s = new String("stringette");
```

위 코드는 절대로 피해야할 극단적인 예시이며, 위 코드가 실행 될 때마다 String 객체를 새로 생성하는데 정말 쓸데없는 짓이다.

String 생성자에 전달되는 `"stringette"`는 그 자체로 String 객체이이기 때문이며, `"stringette"`는 String 생성자로 만들어지는 모든 객체와 기능적으로 같다. 

따라서 해당 코드는 그냥 아래 처럼 사용하는 것이 더 낫다.

```java
String s = "stringette";
```

이 경우는 앞의 생성자와 달리 매번 객체를 생성하는 대신 동일한 String 객체를 생성한다. 

(같은 JVM 위에서 돌아가는 모든 코드가 해당 객체를 재사용하게 된다.)



또한 생성자와 정적 팩토리 메서드를 둘 다 제공하는 immutable 클래스일 경우, 생성자가 아닌 정적 팩토리 메서드를 이용하면 불필요한 객체 생성을 피할 수 있다

> ex) new Boolean(true)보다는 Boolean.valueOf(String)이 대체로 바람직하다.

생성자는 매번 새로운 객체를 생성하지만 정적 펙토리 메서드는 대체로 기존에 생성해두었던 객체를 재사용한다.



<br>



#### Mutable 클래스의 경우

immutable 클래스 뿐만이 아니라 mutable 클래스에서도 객체를 재사용 할 수 있다.

예를들어 Date 객체의 경우는 mutable 이지만 변경이 필요 없는 경우도 있다.

```java
public class Person {
  private final Date birthDate;
  
  public boolean isBabyBoomer() {
    Calendar gmtCar = Calendar.getInstance(TimeZone.getTimeZone("GMT"));
    gmtCal.set(1946, Calendar.JANUARY, 1, 0, 0, 0);
    Date boomStart = gmtCal.getTime();
    gmtCal.set(1965, Calendar.JANUARY, 1, 0, 0, 0);
    Date boonEnd = gmtCal.getTime();
    return birthDate.compareTo(boomStart) >= 0 &&
      birthDate.compareTo(boomEnd) < 0
  }
}
```

위 예시에선 isBabyBoomer 메서드를 호출할 때 마다, Calendar 객체, TimeZone 객체, Date 객체 두개를 생성한다. 

이 경우에는 정적 초기화 블록(static initializer)을 통해 생성하여 재사용 하는 것이 좋다.

```java
public class Person {
  private final Date birthDate;
  private static final Date BOOM_START;
  private static final Date BOOM_END;
  static {
    Calendar gmtCar = Calendar.getInstance(TimeZone.getTimeZone("GMT"));
    gmtCal.set(1946, Calendar.JANUARY, 1, 0, 0, 0);
    Date BOOM_START = gmtCal.getTime();
    gmtCal.set(1965, Calendar.JANUARY, 1, 0, 0, 0);
    Date BOOM_END = gmtCal.getTime();
  }
  
  public boolean isBabyBoomer() {
    return birthDate.compareTo(boomStart) >= 0 &&
      birthDate.compareTo(boomEnd) < 0
  }
}
```

위 코드 같은 경우 class가 로드되는 시점에서 각 날짜 관련 객체들을 한번만 생성한다. 이전 코드에서 매번 생성하는것보다 더욱 효율적이다. 

(자주 호출되는 메서드였을 경우에는 더욱 성능 개선이 될 것이다.)

만약 isBabyBoomer 메서드가 한번도 호출 되지 않는 경우에는 쓸모없는 객체 생성일 수도 있다. 

이 경우에는 초기화 지연(lazy initialization)을 사용하면 개선 할 수 있다. 하지만 초기화 지연 또한 구현이 복잡해지고 성능 개선이 어렵기 때문에 추천되지 않는다.



<br>



#### 자동 객체화(autoboxing)에서의 유의점

JDK 1.5 이후부터 추가된 기능이며 기본 자료형과 객체 표현형을 섞어 사용 할 수 있도록 해준다. 

이 덕분에 기본 자료형과 객체 표현형의 의미적인 차이는 미미해졌지만 성능적은 차이는 무시하고 사용하기에는 어려움이 있다.

```java
public static void main(String[] args){
  Long sum = 0L;
  for(long i = 0; i < Integer.MAX_VALUE; i++) {
    sum += i;
  }
  System.out.println(sum);
}
```

위 코드는 계산 결과는 정확하지만 Long 객체에 값을 더해주기 때문에 기대했던 것보다 매우 나쁜 성능을 보여준다.

Long은 immutable 클래스이기 때문에 Long에 값을 더해줄 때마다 Integer.MAX_VALU(2^31)개 만큼의 새로운 Long 객체의 생성이 이루어진다.

따라서 객체 표현형 대신에 기본 자료형을 사용하고, 생각지도 못한 자동 객체화가 발생하지 않도록 유의해야 한다.

다만 객체를 만들어서 코드의 명확성과 단순성을 높이고 프로그램의 능력을 향상시킬 수 있는 경우에는 객체의 생성이 더 좋을 수 있다.



<br>



#### 객체 풀(object pool)의 사용

객체 풀을 만들어 추가적인 객체 생성을 막는 기법은 객체 생성비용이 극단적으로 높지 않다면 사용하지 않는 것이 좋다.

> 허용되는 극단적인 경우는 데이터베이스 커넥션 생성을 예를 들 수 있다.
>
> 데이터베이스 접속의 경우에는 라이센스에 따라 연결이 제한 될 수도 있고, 접속 비용이 높기 때문에 허용된다.

일반적으로 객체 풀을 만들 경우에는 코드가 어지러워지고 메모리 요구량이 증가하며 성능도 떨어진다. 특히 최신 JVM일 경우 GC가 최적화 되어 있어 객체 풀보다 가벼운(lightweight) 객체가 더욱 월등한 성능을 보여준다.



<br>



#### 방어적 복사(defensive copy)

규칙 39에서 다룰 내용으로 규칙 5와 다른 관점의 이야기이다.

새로운 객체를 생성해야 한다면 기존 객체를 재사용 하지 말아야 하며, 

방어적 복사가 요구되는 상황에서는 객체를 재사용 하는데 드는 비용이 같은 객체를 여러 벌 만드는 비용보다 훨씬 높다는 점을 유의해야 한다.

이러한 경우 방어적 복사본을 만들지 못하면 골치아픈 버그나 보안 결함이 생길 수 있다. 

이에 비해 추가적인 객체 생성은 코드 스타일 및 약간의 성능 희생이 있을 뿐이므로 방어적 복사를 고려하자.



<br>



#### 정리

1. 객체는 기능적으로 동일할 경우 새로 생성하는 것보다 재사용 하는 것이 낫다. Immutable 객체일 경우 특히 재사용을 고려하라.
2. autoboxing은 객체를 새로 생성하게 된다. 코드의 명확성과 단순성의 증가가 성능의 잇점보다 우위일 경우가 아니면 autoboxing을 통한 객체 생성을 주의하라.
3. 생성비용이 극단적으로 비싼 객체가 아닐 경우 객체 풀의 사용을 자제하라.



