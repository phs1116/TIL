## 2장 객체의 생성과 삭제

### 규칙 6. 유효기간이 지난 객체 참조(만기 참조)는 폐기하라.



<br>



C나 C++처럼 메모리 관리를 직접 해야하는 언어를 사용하다 가비지 컬렉터가 포함된 언어를 사용하면 프로그래밍이 매우 쉬워진다.

그래서 메모리 관리가 필요하다는 사실을 망각하게 되지만 이를 주의해야 한다.



아래 스택 코드를 참고하자.



```java
public class Stack {
  private Object[] elements;
  private int size = 0;
  private static final int DEFAULT_INITIAL_CAPACITY = 16;
  
  public Stack() {
    elements = new Object[DEFAULT_INITIAL_CAPACITY];
  }
  
  public void push(Object e) {
    ensureCapacity();
    elements[size++] = e
  }
  
  public Object pop(){
    if(size == 0){
      throw nw EmptyStackException();
    }
    return elements[--size];
  }
  
  public void ensureCapacity(){
    if(elements.length == size){
      elements = Arrays.copyOf(elements, 2 * size + 1);
    }
  }
}
```

위 스택 코드에는 뚜렷한 문제점은 없으나, 메모리 누수(memory leack) 이슈가 있다. 이로 인헤 가비지 컬렉터의 부담이 커져 성능이 저하되거나 메모리 요구량이 증가할 것이다. 극단 적인 경우엔 디스크 페이징(disk paging)이 발생할 것이며, 심지어는 OutOfMemoryError가 발생할 것이다.



<br>



#### 만기 참조 (obsolete reference)

메모리 누수가 발생하는 문제는 스택의 크기가 커졌다가 다시 줄어드면서 제거한 객체들을 가비지 컬렉터가 처리하지 못해서 발생한다.

제거된 객체는 스택을 사용하는 프로그램이 더이상 참조하지 않음에도 불구하고, 스택에서는 여전히 참조하고 있다.

이처럼 다시는 사용하지 않을 참조를 **만기 참조(obsolete reference)**라고 말한다. 즉 이 코드의 문제점은 만기 참조가 제거되지 않았다는 것에 있다.

> 스택에서 size보다 작은 곳에서 참조되어 실제로 사용되는 부분(active portion)을 제외하고, 
>
> size보다 큰 곳에서 참조하고 있는 객체들은 전부 만기 참조이다. 

만약 만기 참조가 제거되지 못하면 만기 참조뿐만이 아니라, 그 객체를 참조하는 다른 객체들도 가비지 컬렉터에 의해 제거되지 못한다.

즉 만기참조가 여러개가 있다면 더욱 많은 객체들이 가비지 컬렉터의 대상에서 제외 될 수 있다.



<br>



#### 만기 참조 객체는 무조건 null로 만들어라.

이런 문제에 대한 해결 방법에서 가장 간단한 방법은, 만기 참조 객체는 무조건 null로 만드는 것이다. 앞선 Stack 클래스의 경우에는 스택에서 pop된 객체의 참조를 그 즉시 null로 만든다.

```java
public Object pop(){
  if(size == 0){
    throw new EmptyStackException();
  }
  Object result = elements[--size];
  elements[size] = null; // 만기 참조 제거
  return result; 
}
```

이 같이 null로 바꿔주면 실수로 해당 참조 객체를 사용하더라도 NullPointException이 발생하기 때문에, 프로그램의 오작동 대신 바로 오류를 발견 할 수 있다.



<br>



#### 객체 참조를 null로 처리하는 것은 규범(norm)이라기 보단 예외적인 조치가 되어야 한다.

이러한 메모리 누수에 관련된 오류를 접하게 되면, 참조를 항상 null로 처리해야한다는 강박관념에 사로잡히곤 한다. 하지만 객체 참조를 null로 처리하는 것은 규범(norm)이라기 보단 예외적인 조치가 되어야 한다.

만기 참조를 제거하는 가장 좋은 방법은 해당 참조가 보관된 참조 객체가 스코프를 벗어나 생명주기를 마치게 하는 것이다. 변수를 정의할 때 그 스코프를 최대한 좁게 만들면 자연스럽게 해결된다.

위 Stack 클래스는 자체적으로 메모리를 관리(Object 배열)한다는 것이 문제이다. 이 배열에서 실제로 사용되는 부분에 있는 객체는 할당된 객체지만 그 이상의 객체는 반환 가능한 객체들이다. 하지만 가비지 컬렉터는 알 도리가 없기에 전부 유효하다고 판단하여 처리하지 않는다. 

이렇게 메모리를 직접 관리하는 클래스일 경우가 만기 참조를 null로 처리하는 예외적인 케이스이, 메모리 누수가 발생하지 않도록 특히 주의해야한다.



<br>



#### 메모리 누수가 발생하는 대표적인 예

##### 캐시(cache)

캐시(cache)도 메모리 누수가 흔히 발생하는 장소이다. 참조를 안에 넣어두고 잊어버리는 경우가 많기 때문이다.

###### **캐시 메모리 누수 해결책**

캐시에서의 메모리 누수에 대한 해결책으로 WeakHashMap을 가지고 캐시를 구현하는 방법이 있다. 

WeakHashMap은 가지고있는 키/객체 쌍이 다른 곳에서 참조되지 않고 있을 경우, 가비지 컬렉터의 대상이 되는 HashMap이다. 

즉 객체가 만기 참조가 되는 순간 캐시의 키/객체 쌍은 자동으로 삭제된다.다만 이 경우는 캐시에 보관되는 항목의 수명이 키에 대한 외부 참조의 수명에 따라 결정되는 상황에만 적용이 가능하다.

하지만 캐시에 보관되는 항목은 보관 기간에 따라 결정되는 것이 일반적이다. 캐시는 보관 기간이 오래된 항목들을 비워야한다. 이런 작업은 백그라운드 쓰레드를 이용해 처리 할 수 있고(Timer or ScheduledThreadPoolExecutor 이용), 캐시에 새로운 항목을 추가할 때 처리할 수도 있다.

LinkedHashMap을 이용하면 removeEldsetEntry 메서드를 이용해 두번째 접근법의 구현이 용이하다. 더욱 복잡한 캐시일 경우에는 java.lang.ref를 직접 사용해 구현해야 한다.



<br>



##### 콜백(callback)

콜백 등록 기능을 제공하는 API를 사용하는 클라이언트가 콜백을 명시적으로 제거하지 않을 경우, 적절한 조취를 취하기 전까지 메모리는 점유된 상태로 남아있게 된다.

###### **콜백 메모리 누수 해결책**

콜백에 대해 약한 참조(weak reference)만 저장하는 방법이다. WeakHashMap의 키로 저장하는 것이 그 예이다.



<br>



#### 정리

1. 보통의 경우, 만기 참조되는 객체는 스코프를 벗어나게 해서 제거하자.
2. 메모리를 직접 관리하는 클래스일 경우 만기 참조되는 참조객체를 항상 null로 셋팅하자.
3. WeakHashMap과 같이 Weak Reference(약한 참조)를 이용해 만기 참조를 제거하자.

