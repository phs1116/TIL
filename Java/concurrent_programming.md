# 자바 병행 프로그래밍

## 병행 테스크 실행

### Runnable과 ExecutorService

Runnable 인터페이스는 다른 테스크와 동시에 실행하고자 하는 테스크를 나타낸다.

Runnable 인터페이스의 형태는 다음과 같다.

```java
public interface Runnable {
  void run();
}
```

run 메서드는 스레드 내에서 실행되는 메서드이다. 별도의 스레드에서 Runnable를 실행하려면 해당 Runnable를 실행할 스레드를 만들어야 한다.

하지만 매번 테스크를 위해 스레드를 생성하는것은 바람직 하지 않다. 예를 들어 짧은 시간 실행되는 테스크일 경우 스레드에 올려 시작하는데 드는 비용 보다, 하나의 스레드에 타임슬라이스를 쪼게 다수 실행하는 것이 더 효율적이다. 반대로 비용이 많이 드는 테스크일 경우 테스크 별로 스레드를 사용 하고, 프로세스 별로 스레드를 하나씩 사용하여 스레드 사이에 전환하는 비용을 줄이는 것이 좋다.

테스크에서 이런 스케쥴링을 관리 하기 힘들므로, 테스크 스케쥴링을 관리하는 **실행자 서비스(executor service)**에서 테스크를 스케쥴링하고 테스크를 수행할 스레드를 선택해 실행한다.



```java
Runnable task = ()-> {...};
ExecutorService executor = ...;
executor.execute(task);
```



### Executors

Executors 클래스에는 서로 다른 스케쥴링 정책을 가진 실행자 서비스를 만드는 정적 팩토리 메서드도 있다.

``Executors.newCachedThreadPool()``는 각 테스크는 가능하면 캐시된 유후 스레드(idle thread)에서 실행하며 모든 스레드가 실행중이면 새 스레드를 할당하여 실행하는 실행자 서비스이다. 병행 스레드의 갯수에는 제한이 없으며 유후 상태로 오래 머문 스레드는 종료된다.



``Executors.newFixedThreadPoop(nThreads)``는 고정 개수의 스레드 풀을 돌려준다. 테스크 실행을 요청하면 해당 테스크는 실행 가능한 스레드가 반환 될 때 까지 큐에 머문다. 고정 개수 스레드 풀은 계산 집약적인 테스크에 사용하거나, 서비스의 리소스 소비를 제한하기에 적합하다. ``Runtime.getRuntime().availableProcessors()``를 통해서 현재 가용 프로세서 개수를 알아내고 스레드 개수를 도출 할 수 있다.



### Future

Runnable은 테스크를 수행하지만 값을 반환하지 않는다. 결과를 계산하여 반환하는 테스크라면 Runnable아닌 Callable<V> 인터페이스를 사용해야한다.

Callable<V>.call은 Runnable.run()이랑 다르게 V타입 값을 반환한다.



```java
public interface Callable<V> {
  V call() throws Exception;
}
```

#### ExecutorService

Callable 인터페이스도 Runnable 인터페이스처럼 ExecutorService에 전달하여 사용한다.

```java
ExecutorService executor = Executors.newFixedThreadPool();
Callable<V> task = ...;
Future<V> result = executor.submit(task);
```

테스크를 ExecutorService에 제출하면 Future를 반환받는다. 

<br>

##### get

```java
V get() throw InterruptedException, ExecutionException
V get(long timeout, TimeUnit unit) throw InterruptedException, ExecutionException
```

Future 클래스에서 get 메서드를 호출하면 Callable 에서 값을 반환할 때 까지 block한다. call 메서드 내에서 예외를 던지거나 타임아웃되면  get은 ExecutionException을 던진다. 

<br>

##### cancel

```java
boolean cancel(boolean mayInterruptIfRunning)
```

cancel 메서드는 테스크가 실행 중인 경우에 테스크를 취소한다.

<br>

##### invokeAll

여러개의 서브테스크를 사용해야 할 경우 Callable 타입의 Collection을 ExecutorService.invokeAll에 인수를 전달한다.

```java
List<Callable<Object> tasks = new ArrayList({task1,task2,task3});
List<Future<Object> results = executor.invokeAll(tasks);
```

위 호출은 모든 테스크가 종료되어 값이 리턴될 때 까지 블록한다. 타임아웃을 인자로 받아 타임아웃 되면 완료되지 않은 모든 테스크를 취소하는 `invokeAll(log timeout, TimeUnit unit)` 메서드도 있다.

<br>

##### ExecutorCompletionService

ExecutionService.invokeAll처럼 모든 서브테스크들이 완료되기 전 까지 블록 될 필요가 없을 경우 사용한다.

ExecutorCompletionService는 완료되는 순서 대로 Future를 반환한다.

<br>

##### invokeAny

invokeAll과 비슷하지만 제출한 테스크중 하나라도 정상 종료 되면 즉시 Future을 반환하고 나머지 테스크를 취소한다. 



