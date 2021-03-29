# item 81. wait 와 notify 보다는 동시성 유틸리티를 애용하라

# 😎 요약

- wait와 notifiy는 올바르게 사용하기가 아주 까다로우니 **고수준 동시성 유틸리티**를 사용하자

# 🖐 wait와 notify?

**스레드의 상태 제어**를 위한 메소드

`wait()`는 가지고 있던 고유 락을 해제하고, 스레드를 잠들게 하는 역할을 하는 메서드이고,

`notify()`는 잠들어 있던 스레드 중 임의로 하나를 골라 깨우는 역할을 하는 메서드이다.

([예제코드](https://programmers.co.kr/learn/courses/9/lessons/278))

```java
public class ThreadB extends Thread{
	// 해당 쓰레드가 실행되면 자기 자신의 모니터링 락을 획득
   // 5번 반복하면서 0.5초씩 쉬면서 total에 값을 누적
   // 그후에 notify()메소드를 호출하여 wiat하고 있는 쓰레드를 깨움
    @Override
    public void run(){
        synchronized(this){
            for(int i=0; i<5 ; i++){
                System.out.println(i + "를 더합니다.");
                total += i;
                try {
                    Thread.sleep(500);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            notify(); // wait하고 있는 쓰레드를 깨움
        }
    }
}
```

```java
public class ThreadA {
    public static void main(String[] args){
        // 앞에서 만든 쓰레드 B를 만든 후 start 
        // 해당 쓰레드가 실행되면, 해당 쓰레드는 run메소드 안에서 자신의 모니터링 락을 획득
        ThreadB b = new ThreadB();
        b.start();

        // b에 대하여 동기화 블럭을 설정
        // 만약 main쓰레드가 아래의 블록을 위의 Thread보다 먼저 실행되었다면 wait를 하게 되면서 모니터링 락을 놓고 대기       
        synchronized(b){
            try{
                // b.wait()메소드를 호출.
                // 메인쓰레드는 정지
                // ThreadB가 5번 값을 더한 후 notify를 호출하게 되면 wait에서 깨어남
                System.out.println("b가 완료될때까지 기다립니다.");
                b.wait();
            }catch(InterruptedException e){
                e.printStackTrace();
            }

            //깨어난 후 결과를 출력
            System.out.println("Total is: " + b.total);
        }
    }
}
```

```java
b가 완료될때까지 기다립니다.
0를 더합니다.
1를 더합니다.
2를 더합니다.
3를 더합니다.
4를 더합니다.
Total is: 10
```

하지만, wait와 notify는 올바르게 사용하기가 아주 까다롭기 때문에, 대신 **고수준 동시성 유틸리티**를 사용하는 것이 좋다.

# 🤯 고수준 동시성 유틸리티

크게 세 가지로 나뉜다.

1. 실행자 프레임워크
2. 동시성 컬렉션 (concurrent collection)
3. 동기화 장치 (synchronizer)

## 1. 실행자 프레임워크

아이템 80번에서 자세히 다루기 때문에 이 아이템에서는 다루지 않는다.

## 2. 동시성 컬렉션

표준 컬렉션(`List`, `Queue`, `Map`) + 동시성 = **동시성 컬렉션**(`CopyOnWriteArrayList`, `ConcurrentHashMap`, `ConcurrentLinkedQueue`)  ([모두 보기](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/package-summary.html))

동시성 컬렉션의 동시성을 무력화하는 것은 불가능하며, 외부에서 락을 걸면 오히려 속도가 느려진다. 동시성을 무력화하지 못하므로 여러 메서드를 원자적으로 묶어 호출할 수도 없다.

### 상태 의존적 메서드

위의 문제를 해결하기 위해 여러 기본 동작들을 하나로 묶는 **상태 의존적 메서드**가 추가되었다.

```java
putIfAbsent("key"); // 키에 매핑된 값이 없을 때에만 새 값을 집어넣고, 없으면 그 값을 반환한다.
```

### Collections.synchronized~

Collections에서 제공해주는 `synchronized~()`를 사용하여 동기화한 컬렉션을 만드는 것 보다는 동시성 컬렉션을 사용하는 것이 성능적으로도 훨씬 좋다

## 3. 동기화 장치

컬렉션 인터페이스 중 일부는 작업이 성공적으로 완료될 때 까지 기다리도록 확장되었다. 

### 대표적인 예시 - BlockingQueue

```java
public interface BlockingQueue<E> extends Queue<E> {
		/**
     * Retrieves and removes the head of this queue, waiting if necessary
     * until an element becomes available.
     *
     * @return the head of this queue
     * @throws InterruptedException if interrupted while waiting
     */
    E take() throws InterruptedException;
}
```

BlockingQueue의 `take()`는 큐의 원소를 꺼내는 역할을 하는데, 만약 큐가 비어있다면 새로운 원소가 추가될 때까지 기다린다. ThreadPoolExcutor을 포함한 대부분의 실행자 서비스(아이템 80)에서 BlockingQueue를 사용한다.

### 동기화 장치의 종류

- CountDownLatch, Semaphore
    - 가장 자주 쓰인다.
- CylicBarrier, Exchanger
    - 상대적으로 덜 쓰인다.
- Phaser
    - 가장 강력하다.

### 동기화 장치의 종류 - CountDownLatch

하나 이상의 스레드가 또 다른 하나 이상의 스레드 작업이 끝날 때까지 기다리게 한다. 생성할 때 int 값을 받는데, 이 값이 `countDown()`을 몇 번 호출해야 대기 중인 스레드를 깨우는지 결정한다.

```java
public class CountDownLatchTest {
  public static void main(String[] args) {

    ExecutorService executorService = Executors.newFixedThreadPool(5);
    try {
      long result = time(executorService, 3, () -> System.out.println("hello"));
      System.out.println("총 걸린 시간 : " + result);
    } catch (Exception e) {
      e.printStackTrace();
    } finally {
      executorService.shutdown();
    }
  }

  public static long time(Executor executor, int concurrency, Runnable action) throws InterruptedException {
    CountDownLatch ready = new CountDownLatch(concurrency);
    CountDownLatch start = new CountDownLatch(1);
    CountDownLatch done = new CountDownLatch(concurrency);

    for (int i = 0; i < concurrency; i++) {
      executor.execute(() -> {
        ready.countDown(); // 타이머에게 준비가 됐음을 알린다.
        try {
          // 모든 작업자 스레드가 준비될 때까지 기다린다.
          start.await();
          action.run();
        } catch (InterruptedException e) {
          Thread.currentThread().interrupt();
        } finally {
          // 타이머에게 작업을 마쳤음을 알린다.
          done.countDown();
        }
      });
    }

    ready.await(); // 모든 작업자가 준비될 때까지 기다린다.
    long startNanos = System.nanoTime();
    start.countDown(); // 작업자들을 깨운다.
    done.await(); // 모든 작업자가 일을 끝마치기를 기다린다.
    return System.nanoTime() - startNanos;
  }
}
```

```java
hello
hello
hello
총 걸린 시간 : 316400
```

`executor`는 `concurrency` 매개변수로 지정한 값(위 코드에서는 3)만큼의 스레드를 생성할 수 있어야 한다. 그렇지 않으면 메서드 수행이 끝나지 않는데 이를 **스레드 기아 교착 상태**라고 한다.

시간을 잴 때는 시스템 시간과 무관한 **System.nanoTime**을 사용하는 것이 더 정확하다.

# 🐾 wait와 notify 메서드를 사용해야하는 상황에는?

어쩔 수 없이 `wait()`와 `notify()`를 사용해야하는 상황에는?

1. 반드시 **동기화 영역 안**에서 사용해야 하며, 
2. 항상 **반복문 안**에서 사용해야 한다.

## wait()

```java
synchronized (obj) {
    while (조건이 충족되지 않았다) {
        obj.wait(); // 락을 놓고, 깨어나면 다시 잡는다.
    }

    ... // 조건이 충족됐을 때의 동작을 수행한다.
}
```

반드시 반복문 밖에서는 절대 호출하면 안 된다. 또한 대기 전에 조건을 검사하여 조건을 이미 충족되었다면 wait를 하지 않게 해두어야 하는데, 이는 **응답 불가 상태**를 예방하기 위해서이다.

한편, 대기한 이후에 조건을 검사하여 조건을 충족하지 않았을 때 다시 대기하게 하는 경우도 있는데, 이는 조건이 만족되지 않아도 스레드가 깨어날 수 있는 상황이 몇 가지 있기 때문이다. (**안전 실패** 예방)

- `notify`를 호출하여 대기 중인 스레드가 깨어나는 사이에 다른 스레드가 락을 거는 경우
- 조건이 만족되지 않았지만 실수 혹은 악의적으로 `notify`를 호출하는 경우
- 대기 중인 스레드 중 일부만 조건을 충족해도 `notifyAll`로 모든 스레드를 깨우는 경우
- 대기 중인 스레드가 드물게 `notify` 없이 깨어나는 경우. 허위 각성(spurious wakeup)이라고 한다.

## notify()

일반적으로 `notify()`보다는 `notifyAll()`을 사용하는 것이 안전하다.

# 😎 정리

- 코드를 새로 작성한다면 `wait`와 `notify`를 쓸 이유가 거의(어쩌면 전혀) 없다.
- 만약 사용해야한다면 `wait`는 while문 안에서 호출해야한다.
- 일반적으로 `notify()`보다는 `notifyAll()`을 사용하자
- 혹시라도 `notify`를 사용하다면 응답 불가 상태에 빠지지 않도록 조심하자