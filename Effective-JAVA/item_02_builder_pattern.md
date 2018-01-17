## 2장 객체의 생성과 삭제

### 규칙 2. 생성자 인자가 많을 때는 Builder 패턴 적용 또한 고려하자.



<br>



static factory method와 constructor는 둘 다 선택적 인자가 많은 경우에 적응하기 힘들다는 공통적인 단점을 가지고 있다.



<br>



#### 1. 점층적 생성자 패턴(telescoping constructor pattern)

선택적 인자가 10여개가 넘을 시, 보통 **점층적 생성자 패턴(telescoping constructor pattern)**을 적용한다.

필수 인자 생성자 정의 후, 추가적으로 선택적 인자를 받는 생성자를 추가하여 해당 생성자를 호출하는 방식으로 대표적인 예는 다음과 같다.

```java
public class NutritionFacts{
  private final int servingSize;
  private final int servings;
  private final int calories;
  private final int fat;
  private final int sodium;
  private final int carbohydrate;
  
  public NutritionFacts(int servingSie, int servings){
    this(servingSize, servings, 0);
  }
  
  public NutritionFacts(int servingSie, int servings, int calories){
    this(servingSize, servings, calories, 0);
  }
  
  public NutritionFacts(int servingSie, int servings, int calories, int fat){
    this(servingSize, servings, calories, fat, 0);
  }
  
  public NutritionFacts(int servingSie, int servings, int calories, int fat, int sodium){
    this(servingSize, servings, calories, fat, sodium, 0);
  }
  
  public NutritionFacts(int servingSie, int servings, int calories, int fat, int sodium, int carbohydrate){
    this.servingSize = servingSize;
    this.servings = servings;
    this.calories = calories;
    this.fat = fat;
    this.sodium = sodium;
    this.carbohydrate = carbohydrate;
  }
}
```

> ~~정말 그지같다.~~

코드만 봐도 알 수 있다싶이, 점층적 생성자 패턴은 인자수가 증가 할 수록 작성하기도, 읽기도 어려운 코드가 된다.

특히 코드만 봐서 해당 인자가 어떤 인자인지 알 기가 힘들다. (인자의 순서를 하나씩 세어봐야 한다.)



<br>



#### 2. 자바빈 패턴(JavaBeans Pattern) 

두번째 방법으로는 자바빈 규약에 따르는 클래스를 생성 한 뒤, setter 메서드로 각 값들을 설정해 주는 방법이 있다.

```java
public class NutritionFacts{
  private final int servingSize;
  private final int servings;
  private final int calories;
  private final int fat;
  private final int sodium;
  private final int carbohydrate;
  
  public NutritionFacts(){}

  //Setter Method
  public void setServingSize(int servingSize){ this.servingSize = servingSize; }
  public void setServings(int servings){ this.servings = servings; }
  public void setCalories(int calories){ this.calories = caloires; }
  public void setFat(int fat){ this.fat = fat; }
  public void setSodium(int sodium){ this.sodium = sodium; }
  public void setCarbohydrate(int carbohydrate){ this.arbohydrate = arbohydrate; }
}
```

자바빈 패턴은 점층적 생성자 패턴에 있던 문제는 없지만, 1회의 함수 호출로 객체 생성을 마칠 수 없으므로 객체 일관성이 깨질 수 있다.

또한 자바빈 패턴은 immutable 클래스를 만들 수 없다는 단점이 있다.



<br>



#### 3. 빌더 패턴 (Builder Pattern)

빌더 패턴은 점층적 생성자 패턴의 안전성과 자바빈 패턴의 가독성을 결합한 대안이다.

객체를 생성하는 빌더 클래스(builder class)를 만들고, 이 생성자에 필수 인자를 전달하여 빌더 객채(builder object)를 만든다.

그 후 빌더 객체에 정의된 setter method를 호출하여 선택적 인자들을 추가하고 build 메서드를 호출하여 immutable 객체를 만든다.

```java
public class NutritionFacts{
  private final int servingSize;
  private final int servings;
  private final int calories;
  private final int fat;
  private final int sodium;
  private final int carbohydrate;
  
  private NutritionFacts(Builder builder){
    this.servingSize = builder.servingSize;
    this.servings = builder.servings;
    this.calories = builder.calories;
    this.fat = builder.fat;
    this.sodium = builder.sodium;
    this.carbohydrate = builder.carbohydrate;
  }
  
  //builder class
  public static class Builder {
    //필수 인자
    private final int servingsize;
    private final int servings;
    //선택적 인자(초기화)
    private final int calories = 0;
    private final int fat = 0;
    private final int sodium = 0;
    private final int carbohydrate = 0;
   
    public Builder(int servingSize, int servings){
      this.servingSize = servingSize;
      this.servings = servings;
    }
    
    public Builder calories(int calories){
      this.calories = calories;
      return this;
    }
    
    public Builder fat(int fat){
      this.fat = fat;
      return this;
    }
    
    public Builder sodium(int sodium){
      this.sodium = sodium;
      return this;
    }
    
    public Builder carbohydrate(int carbohydrate){
      this.carbohydrate = carbohydrate;
      return this;
    }
    
    public NutritionFacts build(){
      return new NutritionFacts(this);
    }
  }  
}
```

> 해당 코드는 모든 인자가 final로 immutable한 NutritionFacts 객체이다.

위에서 작성한 빌더 패턴은 객체 자신을 반환하므로 다음과 같이 chaining하여 사용 할 수 있다.

```java
NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8).calories(100).sodium(35).carbohydrate(27).build();
```



<br>



##### 빌더 패턴의 장점

또한 빌더패턴을 적용하면서 얻을 수 있는 장점은 다음과 같다.

1. 불변식(invariant)을 적용 할 수 있다.

   build 메서드, 혹은 build가 호출 되기 전에 불변식에 위반되었는지 검사 할 수 있다.

   ```java
   public NutritionFacts build(){
     if(/*불변식 검사*/){
       throw new IllegalArgumentExcepiton();
     }
     return new NutritionFacts(this);
   }
   ```

   ```java
   public Builder carbohydrate(int carbohydrate){
     if(/*불변식 검사*/){
       throw new IllegalArgumentExcepiton();
     }
     this.carbohydrate = carbohydrate;
     return this;
   }
   ```

2. 객체를 생성 할 때 여러 개의 가변인자(varargs, ex : String…)를  인자로 받을 수 있다.

   가변인자는 생성자 및 메서드에 하나만 사용 가능하다. 하지만 빌더 패턴을 적용하면 각 선택적 인자의 setter 메서드에 가변 인자를 적용 할 수 있다.

   (기본적으로 메서드 시그니쳐의 마지막 인자에만 지정한다.)

3. 빌더 패턴은 유연하다.

   하나의 빌더 객체로 여러 객체를 만들 수 있다.

   기존에 사용했던 빌더 객체에서 setter 메서드로 값만 바꾼 후 build 하면 다른 객체가 생성된다.

   또한 빌더 객체는 어떤 필드의 값은 자동으로 채우도록 할 수 있다.(예 : 자동으로 증가하는 일련 번호 등.)

4. 인자가 설정된 빌더는 훌륭한 추상적 팩토리이다.

   클라이언트는 빌더를 다른 메서드에 넘겨서 하나 이상의 객체를 만들도록 할 수 있다.

   다음과 같은 제네릭을 적용한 인터페이스면 어떤 객체를 만드는 빌더냐에 관계 없이 모든 빌더에 적용 할 수 있다.

   (빌더 객체를 인자로 받는 모든 메서드에 전달 할 수 있다.)

   ```java
   public interface Builder<T> {
     public T build();
   }
   ```

   >  빌더 객체의 인자를 제한 할 경우, 보통 한정적 와일드카드 자료형을 통해 인자의 자료형을 제한한다.
   >
   >  ex ) Tree buildTree(Builder<? extends Node> nodeBuilder){ … }




<br>



##### 빌더 패턴의 단점

객체를 생성 하려면 우선 빌더 객체를 생성해야한다는 단점이 있다.

실무에서는 빌더 객체 생성에 의한 오버헤드의 문제는 없겠지만, 성능이 중요한 상황에선 그렇지 않을 수 도 있다.

또한 점층적 생성자 패턴에 비해 많은 코드를 요구하기 때문에 인자가 충분히 많은 상황에서 이용해야 한다.



<br>



#### 정리

1. 클래스에 선택적 인자가 많을 경우 빌더 패턴을 고려하자.
2. 빌더 패턴의 장점 : 불변식 검증이 가능하다. 빌더 객체를 재사용 할 수 있다. 훌륭한 추상적 팩토리로 유연하게 사용 할 수 있다.
3. 빌더 패턴의 단점 : 빌더 객체를 추가로 생성해야한다. 코드량이 많이 증가한다.