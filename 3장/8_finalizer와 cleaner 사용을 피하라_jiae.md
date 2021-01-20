# [Effective Java] Item 8. finalizer와 cleaner 사용을 피하라

---

## **자바의 두 가지 객체 소멸자**

### **finalized**
- 예측할 수 없고 상황에 따라 위험할 수 있어서 일반적으로 불필요함.
- 오작동, 낮은 성능, 이식성 문제의 원인이 되기도 함.
- Java 9부터 deprecated API로 지정되어 `cleaner`를 대안으로 사용.

### **cleaner**
- finalized 보다는 덜 위험하지만 여전히 예측할 수 없고, 느리고, 일반적으로 불필요함

### **자바의 finallized와 cleaner 부작용**
1. 자바의 finalizer와 cleaner는 C++의 파괴자(destructor)와는 다른 개념이다. 자바에서 접근할 수 없게 된 객체는 가비지 컬렉터가 회수하고 프로그래머는 아무런 작업도 하지 않아도 된다. 만약 `비메모리 자원을 회수`해야 한다면 `try-with-resources`와 `try-finally`를 이용해 해결한다.
2. `finalizer와 cleaner는 즉시 수행된다는 보장이 없다. 즉, finalizer와 cleaner로는 제때 실행되어야 하는 작업은 절대 할 수 없다.`
   - 예를 들어, 파일 닫기와 같은 작업을 finalizer나 cleaner에게 맡기면 중대한 오류가 발생할 수 있다.
   - `finalizer와 cleaner가 수행되는 시간은 전적으로 가비지 컬렉터 알고리즘에 달렸으며`, 가비지 컬렉터의 종류마다 천차만별이다. (즉, JVM의 버전과 옵션에 따라 달라질 수 있다)
   - finalizer가 다른 쓰레드보다 우선순위가 낮아 실행될 기회를 얻지 못하고 OutOfMemoryError가 발생할 수 있기 때문에 주의해야 한다.
3. `프로그램의 생애주기와 상관 없는, 상태를 영구적으로 수정하는 작업에서는 절대 finalizer나 cleaner에 의존해서는 안된다.`
   - 예를 들어 데이터베이스의 공유 자원의 lock 해제를 finalizer나 cleaner에게 맡기면 분산 시스템 전체가 서서히 멈출 수 있다.
   - System.gc나 system.runFinalization 메서드를 사용하더라도 finalizer나 cleaner의 실행 가능성을 높여줄 수는 있으나 실행을 보장해주진 않는다.
4. `finalizer 동작 중 발생한 예외는 무시되며, 처리할 작업이 남았더라도 그 순간 종료된다.`
   - 보통의 경우엔 잡지 못한 예외가 쓰레드를 중단시키고 스택 추적 내역을 출력하지만, 같은 일이 finzlizer에서 일어난다면 경고조차 출력하지 않는다. (그나마 cleaner를 사용하는 라이브러리는 자신의 쓰레드를 통제하기 때문에 이러한 문제는 발생하지 않는다.)
5. `finalizer와 cleaner은 심각한 성능 문제도 동반한다.`
   - finalizer를 사용한 객체를 생성하고 파괴할 때 AutoCloseable 객체를 생성하고 try-with-resources로 닫는 것보다 시간이 수십 배 더 걸릴 수 있다. finalizer가 가비지 컬렉터의 효율을 떨어뜨리기 때문이다. 뒤에서 살펴볼 안전망 방식에서는 상대적으로 시간이 덜 소모되지만 try-with-resources를 이용하는 것보다 시간이 오래 소요된다.
7. `finalizer를 사용한 클래스는 finalizer 공격에 노출되어 심각한 보안 문제를 일으킬 수 있다.`
   - 생성자나 직렬화 과정(readObject, readResolve)에서 예외가 발생하면, 이 생성되다 만 객체가 악의적인 하위 클래스의 fializer가 수행될 수 있게 한다.
   - 이 finalizer는 정적 필드에서 자신의 참조를 할당하여 가비지 컬렉터가 수집하지 못하게 막을 수 있다. 이렇게 일그러진 객체가 만들어지고 나면, 이 객체의 메서드를 호출해 애초에 허용되지 않았을 작업을 수행하게 할 수도 있다.
   - 따라서, `객체 생성을 막으려면 생성자에서 예외를 던지는 것만으로 충분하지만, finalizer가 있다면 final 클래스로 만들어서 하위 클래스를 만들 수 없도록 하거나, final이 아닌 클래스는 아무일도 하지 않는 finalize 메서드를 만들고 final로 선언해야 한다.`

### **finalizer와 cleaner를 대신할 AutoCloseable**

- `Autocloseable을 구현하고, 클라이언트에서 인스턴스를 다 쓰고 나면 close 메서드를 호출하자.` (일반적으로 예외가 발생해도 제대로 종료되도록 try-with-resources를 사용해야 한다.)

- 이 때, 각 인스턴스는 자신이 닫혔는지 추적할 수 있도록 `close 메서드`에서 이 객체가 더 이상 유효하지 않음을 필드에 기록하고, 다른 메서드는 이 필드를 검사해서 객체가 닫힌 후에 불렸다면 IllegalStateException을 던져야 한다.

### **finalizer와 cleaner의 적절한 쓰임새**
1. 자원의 소유자가 close 메서드를 호출하지 않는 것에 대비한 안전망 역할
    - 자바 라이브러리의 FileInputStream, FileOutputStream, ThreadPoolExecutor가 대표적으로 이런 예시에서 사용 중

      <details>
      <summary>FileInputStream (Java 11)</summary>
      <div markdown="1">

      ```java
      public class FileInputStream extends InputStream {
         ...
         /**
         * @apiNote
         * 이 스트림에서 사용하는 리소스를 해제하려면 close 메서드를 직접 호출하거나 try-with-resources로 호출해야 합니다.
         */
         @Deprecated(since="9", forRemoval = true)
         protected void finalize() throws IOException {
         }
      }
      ```

      </div>
      </details>

      <details>
      <summary>FileInputStream (Java 8)</summary>
      <div markdown="1">

      ```java
      public class FileInputStream extends InputStream {
         ...
         protected void finalize() throws IOException {
            if ((fd != null) &&  (fd != FileDescriptor.in)) {
                  /* if fd is shared, the references in FileDescriptor
                  * will ensure that finalizer is only called when
                  * safe to do so. All references using the fd have
                  * become unreachable. We can call close()
                  */
                  close();
            }
         }
      }
      ```

      </div>
      </details>

2. 네이티브 피어(native peer)와 연결된 객체에서 사용
    > 네이티브 피어: 일반 자바 객체가 네이티브 메서드를 통해 기능을 위임한 네이티브 객체. 
    - 자바 객체가 아니기 때문에 가비지 컬렉터에서는 이 존재를 알지 못함. 따라서 cleaner나 finalizer가 나서서 처리하기에 적당하다.
    - 단, 성능 저하를 감당할 수 없거나 네이티브 피어가 사용하는 자원을 즉시 회수해야 한다면 앞서 설명한 close 메서드를 사용해야 한다.

### **cleaner를 안전망으로 활용하는 AutoCloseable 클래스**

```java
import java.lang.ref.Cleaner;

public class Restaurant implements AutoCloseable {
    private static final Cleaner cleaner = Cleaner.create();

    private static class Table implements Runnable {
        int cups; // 치워야 하는 컵
        int plates; // 치워야 하는 접시

        Table (int cups, int plates) {
            this.cups = cups;
            this.plates = plates;
        }

        @Override public void run() {
            System.out.println("자리 정리중...");
            this.cups = 0;
            this.plates = 0;
        }
    }

    private final Table table;

    private final Cleaner.Cleanable cleanable;

    public Restaurant (int cups, int plates) {
        table = new Table(cups, plates);
        cleanable = cleaner.register(this, table);
    }
    @Override
    public void close() throws Exception {
        cleanable.clean();
    }
}
```

```java
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.*;

class RestaurantTest {
    @Test
    void quickManager() throws Exception {
        try (Restaurant restaurant = new Restaurant(4, 4)){
            System.out.println("즐거운 청소 시작!");
        } finally {
            System.out.println("청소 완료!");
        }
    }

    @Test
    void lazyManager() throws Exception {
        new Restaurant(4, 4);
        System.out.println("언젠간 하지 뭐..");
    }
}
```

#### **quickManager**
```
즐거운 청소 시작!
자리 정리중...
청소 완료!
```
#### **lazyManager**
```
언젠간 하지 뭐..
```

<details>
<summary> 방(room) 자원을 수거하기 전에 반드시 청소(clean)해야 하는 Room 클래스 (책 예시))</summary>
<div markdown="1">
방(room) 자원을 수거하기 전에 반드시 청소(clean)해야 하는 Room 클래스 (책 예시)

```java
public class Room implements AutoCloseable {
   private static final Cleaner cleaner = Cleaner.create();

   // 청소가 필요한 자원. 절대 Room을 참조하면 안됨.
   private static class State implements Runnable {
      int numJunkPiles; // 방(room) 안의 쓰레기 수

      State(int numJunkPiles) {
         this.numJunkPiles = numJunkPiles;
      }

      // close 메서드나 cleaner가 호출
      @Override public void run(){
         System.out.println("방 청소");
         numJunkPiles = 0;
      }
   }

   // 방의 상태. cleanable과 공유한다.
   private final State state;

   // cleanable 객체. 수거 대상이 되면 방을 청소한다.
   private final Cleaner.Cleanable cleanable;

   public Room (int numJunkPiles) {
      state = new State(numJunkPiles);
      cleanable = cleaner.register(this, state);
   }

   @Override public void close(){
      cleanable.clean();
   }

}
```

- State 중첩 클래스: cleaner가 방을 청소할 때 수거할 자원들을 담고 있음.
    - numJunkPiles 필드: 수거할 자원 (방 안의 쓰레기 수)
    - Runnuble을 구현하고, 그 안의 run 메서드는 cleanable에 의해 딱 한 번만 호출됨. 이 cleanable 객체는 Room 생성자에서 cleaner에 Room과 State를 등록할 때 얻는다.
    - run 메서드가 호출되는 상황: Room의 close 메서드를 호출할 때
    - 가비지 컬렉터가 Room을 회수할 때까지 클라이언트가 close를 호출하지 않는다면, cleaner가 State의 run 메서드를 호출.
- State 인스턴스는 '절대로' Room 인스턴스를 참조해서는 안됨. Room 인스턴스를 참조할 경우 순환 참조가 생겨 가비지 컬렉터가 Room 인스턴스를 회수해갈 기회가 오지 않음. ==> State가 정적 중첩 클래스인 이유!! (정적이 아닌 중첩 클래스는 자동으로 바깥 객체의 참조를 갖게 되기 때문)
- Room의 cleaner는 단지 안전망으로 쓰임. 클라이언트가 모든 Room 생성을 try-with-resources 블록으로 감쌋다면 자동 청소는 전혀 필요하지 않음.

#### 잘 짜여진 클라이언트 코드 예시 (try-with-resources 블록을 사용한)
```java
public class Adult {
   public static void main(String[] args) {
      try (Room myRoom = new Room(7)) {
         System.out.println("안녕~");
      }
   }
}
```

기대한대로 Adult 프로그램은 "안녕~"을 출력한 후, 이어 "방청소"를 출력한다.

#### 방청소를 절대 하지 않는 프로그램
```java
public class Teenager {
   public static void main(String[] args) {
      new Room(99);
      System.out.println("아무렴");
   }
}
```

"아무렴"에 이어 "방청소"가 출력될까? 하지만 "방 청소"는 출력되지 않았다. 이렇게 cleaner의 동작에만 의존하면 예측 할 수 없는 상황이 생긴다.

</div>
</details>


#### cleaner의 명세
> System.exit을 호출할 때의 cleaner의 동작은 구현하기 나름이다. 청소가 이뤄질지 보장은 하지 못한다.

명세에는 명시하지 않았지만, 일반적인 프로그램의 종료에서도 마찬가지다. 각 컴퓨터에서 어떤식으로 동작할지를 보장할 수 없다.

### 핵심 정리

```
cleaner (Java 8까지는 finalizer)는 안전망 역할이나 중요하지 않은 네이티브 자원 회수용으로만 사용하자. 물론 이런 경우라도 불확실성과 성능 저하에 주의해야 한다.
```
---


### 참고 자료
- Effective Java 3/E