# item8. finalizer와 cleaner 사용을 피하라

# 🥱 Finalizer와 Cleaner

## 1. Finalizer란?

Object에 존재하는 `finalize()`를 의미한다. 클래스의 객체가 더 이상 사용되지 않으면 GC가 자동으로 호출한다.

```java
effective java
2
3
4
5
```

```java
public class Finalizable {
  private BufferedReader reader;

  public Finalizable() {
    InputStream input = this.getClass()
        .getClassLoader()
        .getResourceAsStream("file.txt");

    this.reader = new BufferedReader(new InputStreamReader(input));
  }

  public String readFirstLine() throws IOException {
    return reader.readLine();
  }

  @Override
  public void finalize() {
    try {
      reader.close();
      System.out.println("Closed BufferedReader in the finalizer");
    } catch (IOException e) {
      // ...
    }
  }

  @Test
  public void gc_호출시_finalizer_동작확인() throws IOException {
    String firstLine = new Finalizable().readFirstLine();
    assertEquals("effective java", firstLine);
    System.gc();
  }

}
```

## 2. Cleaner란?

- Java 9에서는 fianlizer가 deprecated 됐고 cleaner가 새로 생겼다.
- cleaner는 별도의 쓰레드를 사용해서 finalizer보다는 덜 위험하지만, 여전히 예측불가하고, 느리며 불필요하다.

## 3. 문제점

### (1) 성능 저하

- 객체가 필요없어진 시점 ~ finalize(또는 cleaner)가 실행되는 시점까지 얼마나 소요될지 **예측이 불가능하다.**
- GC의 효율을 떨어트리게 되므로 **객체 생성, 소멸 시간이 많게는 수백배까지 차이**가 날 수 있다.

### (2) 실행이 안될 가능성 존재

- 반드시 실행됨을 보장할 수 없다.
    - `System.gc()`, `System.runFinalization()` : 실행될 '가능성'만 높여줄 뿐, 반드시 실행함을 보장하지는 않는다.
    - `System.runFinalizersOnExit()` : 실행 보장을 위해 제공된 메소드이지만, 치명적인 결함이 존재해서 현재는 deprecated 되었다.

### (3) 예외 발생 시 무시

- Finalize를 실행하는 동안 catch되지 않는 예외가 발생하면 예외가 무시되고, Finalize가 끝난다.
    - 이로인해 객체가 망가질 가능성이 있고, 다른 스레드에서 해당 객체에 접근하면 문제가 생긴다.
- Cleaner는 해당되지 않는다.

### (4) 인스턴스 반납 지연

- Finalize 스레드는 다른 스레드보다 **우선순위가 낮기** 때문에 시스템에 문제를 일으킬 수 있다.
    - GC를 실행 전에 무조건 실행해야하는 finalizer의 우선순위가 낮기 때문에 회수가 계속 밀리다가 OutOfMemory가 발생할 가능성이 있다.
- Cleaner 역시나 GC에 의존하기 때문에 사용하지 않는 것이 좋다.

### (5) 보안 문제

- 생성자나 직렬화 과정에서 예외가 발생하면, 악의적으로 하위 클래스의 finalizer가 수행될 수도 있다.
    - 그렇게되면 하위 클래스는 GC의 대상에서 벗어나기 때문에 GC의 대상이 되지 않는다.

예제코드 ([참고](https://yangbongsoo.tistory.com/8?category=919799))

```java
class Positive {

  Integer value = 0;

  public Positive(Integer value) {
    if (value <= 0) {
      throw new IllegalArgumentException("Value must be positive");
    }
    this.value = value;
  }

  @Override
  public String toString() {
    return value.toString();
  }
}

class AttackPositive extends Positive {
  static Positive positive;

  public AttackPositive(Integer value) {
    super(value);
  }

  @Override
  protected void finalize() throws Throwable {
    System.out.println("finalizer!");
    positive = this; // gc되지 않음
  }

  public static void main(String[] args) {
    try {
      new AttackPositive(-1);
    } catch (Exception e) {
      System.out.println(e.getMessage());
    }

    System.gc();
    System.runFinalization();

    if (positive != null) {
      System.out.println("Positive object : " + positive + " created!");
    }
  }
}
```

## 4. 사용하는 경우

### (1) 안전망 역할

- 자원의 소유자가 `close()`를 호출하지 않는 것에 대비하는 경우
    - 늦게라도 회수하는 것이 안 하는 것보다 낫기 때문

### (2) 네이티브 피어 정리

- C/C++이나 어셈블리 프로그램을 컴파일한 기계어 프로그램이자, **GC가 존재를 알지 못하는 리소스** ([예제코드](https://github.com/dkelosky/java-jni))
- 단, 성능 저하를 감당할 수 있고, 네이티브 피어가 심각한 자원을 가지고 있지 않는 경우에 괜찮다.

# 😎 대안책 - AutoCloseable

- 클라이언트에서 인스턴스를 다 쓰고 나면 `close()`를 호출하면 된다.
- 예외가 발생해도 제대로 종료할 수 있게 처리해주는 `try-with-resouces`를 사용하는 것이 좋다.

```java
public class CloseableResource implements AutoCloseable {

  private BufferedReader reader;

  public CloseableResource() {
    InputStream input = this.getClass()
        .getClassLoader()
        .getResourceAsStream("file.txt");
    reader = new BufferedReader(new InputStreamReader(input));
  }

  public String readFirstLine() throws IOException {
    return reader.readLine();
  }

  @Override
  public void close() {
    try {
      reader.close();
      System.out.println("Closed BufferedReader in the close method");
    } catch (IOException e) {
      // ...
    }
  }

  @Test
  public void whenTryWResourcesExits_thenResourceClosed() throws IOException {
    try (CloseableResource resource = new CloseableResource()) {
      String firstLine = resource.readFirstLine();
      assertEquals("effective java", firstLine);
    }
  }
**
}
```

# 🐇 맺으며

finalizer와 cleaner는 사용하지말자! (아주 특수한 경우에만 조금 고려해보자)