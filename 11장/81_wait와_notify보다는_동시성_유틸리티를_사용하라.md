# item 81. wait와 notify보다는 동시성 유틸리티를 사용하라

Java에는 동시성을 다루기 위해서 wait와 notify를 지원한다. 다만, 이 메서드들은 Java 5부터 지원하는 다양한 동시성 유틸리티 덕분에 wait와 notify를 사용할 이유가 줄었다.

## wait와 notify

synchronized 블록 참고: [http://tutorials.jenkov.com/java-concurrency/synchronized.html](http://tutorials.jenkov.com/java-concurrency/synchronized.html)

baeldung 참고: [https://www.baeldung.com/java-wait-notify](https://www.baeldung.com/java-wait-notify)

wait와 notify 그리고 notifyAll은 모두 Object에서 지원하는 메서드들이다.

- wait

    갖고 있는 고유 락을 해제하고 스레드를 잠들게하는 메서드.

    호출하는 메서드가 반드시 고유 락을 가지고 있어야 사용이 가능하다. 즉, `synchronized` 블록 내부에서 호출이 되어야한다. 그렇지 않으면 `IllegalMonitorStateException`가 발생한다.

- notify

    잠들어 있던 스레드 중 임의로 하나를 골라 깨우는 메서드. 단, notify는 잠들어 있는 스레드 중 어떤 스레드를 깨울지 선택할 수 없으므로 보통 notifyAll을 사용한다.

- notifyAll

    잠들어 있던 스레드 모두를 깨우는 메서드.

다음은 wait를 사용하는 표준 방식이다.

```java
synchronized (obj) {
	while(condition) {
		obj.wait();	
	}
	// ...
}
```

wait 전에 조건을 검사하여 조건이 이미 충족되었다면 wait를 건너뛰도록 만드는 것이다. wait를 호출할 때는 반드시 대기 반복문 `wait loop` 내에서 호출을 해야하며 반복문 밖에서는 호출하면 안된다.

대기 후에 조건을 검사하여 조건이 충족되지 않았다면 다시 대기하게 만드는 것은 안전 실패를 막는 조치이다. 만약 조건을 만족하지 않았는데 스레드가 동작을 이어간다면 락이 보호하는 불변식을 깨트릴 위험이 있다.

조건을 만족하지 않아도 스레드가 깨어날 수 있는 상황은 다음과 같다.

- 스레드가 notify를 호출한 다음 대기중이던 스레드가 깨어난 사이에 다른 스레드가 락을 얻어 그 락이 보호하는 상태를 변경하는 경우
- 조건이 만족하지 않았음에도 다른 스레드가 실수로 혹은 악의적으로 notify를 호출하는 경우. 참고로 공개된 객체를 락을 통해 대기하는 경우 이런 위험에 노출된다.
- 대기 중인 스레드 중 일부만 조건이 충족되어도 `notifyAll`로 모든 스레드를 깨울 수 있다.
- 대기 중인 스레드가 드물게 `notify` 없이도 깨어날 수 있다. 이를 허위 각성`spurious wakeup`이라고 한다.

### example

```java
public class Data {
    private String packet;

    private boolean transfer = true;

    public synchronized void send(String packet) {
        while (!transfer) {
            try {
                wait();
            } catch (InterruptedException e)  {
                Thread.currentThread().interrupt();
            }
        }
        transfer = false;

        this.packet = packet;
        notifyAll();
    }

    public synchronized String receive() {
        while (transfer) {
            try {
                wait();
            } catch (InterruptedException e)  {
                Thread.currentThread().interrupt();
            }
        }
        transfer = true;

        notifyAll();
        return packet;
    }
}
```

Data에는 send와 receive 메서드가 존재하는데 이들 메서드에 `synchronized` 키워드를 붙임으로써 락을 얻을 수 있도록 했다.

그리고 `transfer`가 true일때 receive를 하고자하면 Data에 접근한 현재 Thread를 대기 상태로 바꾸며 `transfer`가 false일때 send를 하면 해당 Data에 접근한 Thread를 대기상태로 바꾼다.

즉, `transfer`가 true일때는 send가 가능하고 false일때는 receive만 가능하다.

```java
public class Sender implements Runnable {
    private Data data;

    public Sender(Data data) {
        this.data = data;
    }

    @Override
    public void run() {
        String packets[] = {
                "First packet",
                "Second packet",
                "Third packet",
                "Fourth packet",
                "End"
        };

        for (String packet : packets) {
            data.send(packet);

            // Thread.sleep() to mimic heavy server-side processing
            try {
                Thread.sleep(ThreadLocalRandom.current().nextInt(1000, 5000));
            } catch (InterruptedException e)  {
                Thread.currentThread().interrupt();
            }
        }
    }
}

public class Receiver implements Runnable {
    private Data load;

    public Receiver(Data load) {
        this.load = load;
    }

    public void run() {
        for(String receivedMessage = load.receive();
            !"End".equals(receivedMessage);
            receivedMessage = load.receive()) {

            System.out.println(receivedMessage);

            try {
                Thread.sleep(ThreadLocalRandom.current().nextInt(1000, 5000));
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }
    }
}

public static void main(String[] args) {
    Data data = new Data();
    Thread sender = new Thread(new Sender(data));
    Thread receiver = new Thread(new Receiver(data));

    sender.start();
    receiver.start();
}
```

Sender와 Receiver는 위와 같고 각각 Runnable을 구현하여 하나의 스레드에서 실행할 수 있도록 만들었다. 이를 실행하면 다음과 같은 결과가 나타난다.

```
First packet
Second packet
Third packet
Fourth packet
```

즉, Sender가 제공하는 데이터 순서대로 receiver가 받는 것을 알 수 있다.

다만, 이를 만족하기 위해서 `synchronized`와 `wait`, `notifyAll`을 올바르게 사용하는 것이 매우 까다롭고 복잡함을 알 수 있다. 또한 `synchronized`를 통한 락은 성능 저하도 일으킨다.

### synchronized

synchronized는 객체 인스턴스, 메서드, static 메서드에 사용이 가능하다. 이를 통해 객체 자체나 method에 락을 제공할 수 있다.

단, 이 락 때문에 성능저하가 있을 수 있다. synchronized에 접근하고 나올때 약간의 오버헤드가 존재한다.

```java
public class SynchronizedNumber {
    private int number;

    public SynchronizedNumber(int number) {
        this.number = number;
    }

    public int get() {
        return number;
    }

    public synchronized int syncGet() {
        return number;
    }

    public synchronized void syncSet(int number) {
        this.number = number;
    }
}

@Test
void synchronizedMethod_versus_nonSynchronized() {
    SynchronizedNumber synchronizedNumber = new SynchronizedNumber(1);

    long start = System.nanoTime();
    IntStream.rangeClosed(1, 100_000_000)
            .forEach(notUse -> synchronizedNumber.get());
    long end = System.nanoTime();

    System.out.println("nonSynchronized: " + (end - start) + "ns");

    start = System.nanoTime();
    IntStream.rangeClosed(1, 100_000_000)
            .forEach(notUse -> synchronizedNumber.syncGet());
    end = System.nanoTime();

    System.out.println("synchronized: " + (end - start) + "ns");
}
```

SynchronizedNumber에 `synchronized`키워드가 있는 get과 없는 syncGet을 가지고 아래 테스트코드와 같이 성능을 측정했을때 다음과 같은 결과가 나타난다.

```
nonSynchronized: 3890810ns
synchronized: 1709781484ns
```

즉, `synchronized`만 차이가 있음에도 `synchronized`에 접근하고 나오는 오버헤드때문에 차이가 벌어짐을 알 수 있다.

또한 `synchronized` 블록에 한계점도 존재하는데 블록에 제공한 인스턴스 자체에 synchronized 락을 걸기 때문에 읽기 작업만 필요하더라도 락으로 인해 작업이 대기되는 한계점이 있다.

이런 문제 때문인지 Java 5부터 제공하는 동시성 유틸리티에서는 `synchronized`를 사용하지 않고 자체에서 최적화하여 성능적으로도 우수한 기능들을 제공한다. 따라서 성능저하 및 사용 복잡성 때문에 Java 5에서 도입된 고수준의 동시성 유틸리티를 사용하는 것을 권장한다.

## 동시성 유틸리티

`java.util.concurrent` 패키지가 제공하는 고수준 동시성 유틸리티는 다음과 같은 범주로 나눌 수 있다.

1. 실행자 프레임워크 `item 80, Executor Framework`

    `ExecutorService`, `Executors` 등을 지원하는 실행자 프레임워크

2. 동시성 컬렉션 `Concurrent Collections`
3. 동기화 장치 `Synchronizer`

### 동시성 컬렉션

동시성 컬렉션은 List, Queue, Map과 같은 표준 컬렉션 인터페이스에 동시성을 가미하여 구현한 컬렉션이다. 높은 동시성을 구현하기 위하여 내부적으로 동시성을 제어한다. 때문에 동시성 컬렉션을 사용한다면 동시성을 무력화하는 것은 불가능하며 외부에서 락을 추가로 사용하면 오히려 성능이 저하된다.

> 락을 준다는 것을 `synchronized`를 제공한다는 것으로 이해

특히 `Collections.synchronizedMap`으로 제공되는 `SynchronizedMap` 보다 `ConcurrentHashMap`을 사용하는 것이 훨씬 좋다.

```java
@Test
void synchronizedMap() {
    // given
    Map<String, Integer> counts = new HashMap<>();
    Map<String, Integer> synchronizedMap = Collections.synchronizedMap(counts);

    synchronizedMap.put("park", 100);
    synchronizedMap.put("lee", 200);
    synchronizedMap.put("kim", 300);

    // when
    List<String> names = Arrays.asList("park", "lee", "kim");

    long start = System.nanoTime();
    IntStream.rangeClosed(1, 100_000_000)
            .forEach(number -> {
                int index = number % 3;
                synchronizedMap.get(names.get(index));
            });
    long end = System.nanoTime();

    System.out.println("synchronizedMap execute time: " + (end - start) + "ns");
}

@Test
void concurrentHashMap() {
    // given
    ConcurrentHashMap<String, Integer> concurrentHashMap = new ConcurrentHashMap<>();

    concurrentHashMap.put("park", 100);
    concurrentHashMap.put("lee", 200);
    concurrentHashMap.put("kim", 300);

    // when
    List<String> names = Arrays.asList("park", "lee", "kim");

    long start = System.nanoTime();
    IntStream.rangeClosed(1, 100_000_000)
            .forEach(number -> {
                int index = number % 3;
                concurrentHashMap.get(names.get(index));
            });
    long end = System.nanoTime();

    System.out.println("concurrentHashMap execute time: " + (end - start) + "ns");
}

@Test
void hashMap() {
    // given
    Map<String, Integer> map = new HashMap<>();

    map.put("park", 100);
    map.put("lee", 200);
    map.put("kim", 300);

    // when
    List<String> names = Arrays.asList("park", "lee", "kim");

    long start = System.nanoTime();
    IntStream.rangeClosed(1, 100_000_000)
            .forEach(number -> {
                int index = number % 3;
                map.get(names.get(index));
            });
    long end = System.nanoTime();

    System.out.println("HashMap execute time: " + (end - start) + "ns");
}
```

각각의 테스트를 돌렸을때 아래와 같은 결과가 나타난다.

```
synchronizedMap execute time: 1728714623ns
concurrentHashMap execute time: 672566310ns
HashMap execute time: 844882515ns
```

SynchronizedMap은 모든 메서드에 `synchronized` 키워드가 붙어 있는 반면 `ConcurrentHashMap`은 `synchronized` 대신 내부적으로 동시성을 보장하도록 최적화하여 구현되어있다.

## 동기화 장치

Java Synchronizers 참고: [https://dzone.com/articles/the-java-synchronizers](https://dzone.com/articles/the-java-synchronizers)

동기화 장치는 스레드가 다른 스레드를 기다릴 수 있게 하여 서로 작업을 조율할 수 있도록 해준다. 자주 쓰이는 동기화 장치로는 CountDownLatch와 Semapore가 있으며 CyclicBarrier와 Exchanger, Phaser도 있다.

### CountDownLatch

CountDownLatch는 하나 이상의 스레드를 기다리게 만드는 동기화 장치이다.

```java
@Test
void countDownLatch() throws InterruptedException {
    // given
    CountDownLatch countDownLatch = new CountDownLatch(1);
    ExecutorService executorService = Executors.newSingleThreadExecutor();
    
    executorService.submit(() -> {
        try {
            Thread.sleep(3000);
            countDownLatch.countDown();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    });

    countDownLatch.await();
    assertThat(countDownLatch.getCount()).isEqualTo(0);
}
```

위와 같이 CountDownLatch에 1을 세팅하고 Latch에 세팅된 카운트가 0이 될때까지 `countDownLatch.await`에서 기다리게 만든다.

### Semaphore

Semaphore는 특정 리소스에 접근하는 것을 제한하도록 지원하는 동기화 장치이다.

```java
Semaphore semaphore = new Semaphore(4);
```

위와 같이 세마포어를 생성하면 접근 가능한 스레드의 갯수를 4개로 제한하는 것이다.

그리고 `Semaphore#acquire`로 세마포어를 하나 사용하고 자원을 다 사용한 후에 `Semaphore#release`로 사용한 세마포어를 다시 돌려놔야한다. 그렇지 않으면 다른 스레드들이 접근할 수 없게 된다.
