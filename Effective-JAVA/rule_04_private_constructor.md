## 2장 객체의 생성과 삭제

### 규칙 4. 객체 생성을 막을 때는 private 생성자를 사용하라.



<br>



때로는 정적 메서드나 필드만 모은 클래스를 만들고 싶을 때가 있다. 

(ex : 기본 자료형 혹은 배열에 적용되는 메서드 모음(java.lang.Math, java.lang,Arrays), 팩토리 메서드 등(java.util.Collections))

이런 유틸리티 클래스는 일반적으로 객체를 만들어 사용하지 않는다. 하지만 생성자를 생략 할 경우 컴파일러가 public으로 기본 생성자를 만든다.

따라서 유틸리티 클래스 특성과 사용자 의도에 맞지 않게 객체 생성에 대한 API가 공개 되는 경우도 잦다.

이런 경우를 위해 객체를 abstract로 선언 해도 상속으로 구현하는 순간 객체의 생성이 가능하기 때문에, private 생성자를 통해 객체의 생성을 방지해야 한다.



<br>



```java
public class UtilityClass {
  // private 생성자로 객체의 생성을 방지한다.
  private UtilityCalss(){
    throw new AssertionError();
  }
  
  ...
}
```

생성자가 private 이므로 클래스 외부에서 사용 할 수 없고, 이를 상속한 경우에도 super 생성자를 호출 할 수 없으므로 상속을 통한 객체 생성도 불가능하다.



<br>



> 싱글턴 패턴에서도 이 규칙으로 객체의 추가 생성을 방지한다.

