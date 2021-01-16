# Item 3. private 생성자나 열거타입으로 싱글턴임을 보증하라.
----
## 싱글턴(Singleton)

### 정의
인스턴스를 오직 하나만 생성할 수 있는 클래스

### 사용 예
- 무상태 객체 (stateless)
- 설계상 유일해야 하는 시스템 컴포넌트
- DBCP(DataBase Connection Pool), 로그기록 객체 등 

### 장점
한번의 객체 생성으로 재사용이 가능해져 메모리 낭비를 방지할 수 있다. 또한 전역성을 갖기 때문에 다른 객체와 공유가 용이히다.

### 단점
클래스를 싱글턴으로 만들면 이를 사용하는 클라이언트를 테스트하기 어려워질 수 있다.
타입을 인터페이스로 정의한 다음 그 인터페이스를 구현해 만든 싱글턴이 아니라면 싱글턴 인스턴스를 가짜 구현(mock)으로 대체할 수 없기 때문이다.

```java
public class Printer {
    private static final Printer INSTANCE = new Printer();

    public static final Printer getInstance() {
        return INSTANCE;
    }

    private int maxPrintPage;
    private Printer() {
        this.maxPrintPage = 50;
    }

    public void print(int printPage) {
        if (prnitPage > maxPrintPage) {
            throw new Exception();
        }
    }
}

class Service {
    private final Printer printer = Printer.getInstance();

    // printer의 print 테스트
    // maxPrintPage 값 바꿔 테스트하고 싶을 경우엔....
    public void print(int printPage){
        printer.print(printPage);
    }
}
```

```java
public interface Printer {
    void print(int printPage);
}

class RealPrinter implements Printer {
    private static final Printer INSTANCE = new RealPrinter();

    public static final Printer getInstance() {
        return INSTANCE;
    }

    private int maxPrintPage = 50;
    private Printer() { }

    @Override
    public void print(int printPage) {
        if (prnitPage > maxPrintPage) {
            throw new Exception();
        }
    }
}

class MockPrinter implements Printer {
    private static final Printer INSTANCE = new MockPrinter();
    
    public static final Printer getInstance() {
        return INSTANCE;
    }
    private int maxPrintPage;
    private MockPrinter() { }

    // 테스트를 위한 코드
    public void changeMaxPrintPage(int mockPrintPage){
        this.maxPrintPage = mockPrintPage;
    }

    @Override
    public void print(int printPage) {
        if (prnitPage > maxPrintPage) {
            throw new Exception();
        }
    }
}

```
-----

## 싱글턴을 만드는 방식

### 1. public static final 필드 방식의 싱글턴
```java
public class Printer {
    public static final Printer INSTANCE = new Printer();
    private Printer() { ... }

    public void print() { ... }
}
```
public, protected 생성자가 없으므로 클래스가 초기화될 때 만들어진 인스턴스가 전체 시스템에서 하나 뿐임이 보장된다.

예외적으로 권한이 있는 클라이언트는 리플렉션 API인 `AccessibleObject.setAccessible`을 사용해 private 생성자를 호출할 수 있다. 이러한 공격을 방지하고자 한다면, 생성자에서 두 번 객체를 생성하려고 할 때 예외를 던지게 하면 된다.

> [리플렉션 API](https://sas-study.tistory.com/275) : 컴파일된 바이트 코드를 통해 해당 클래스의 메소드, 타입, 변수까지 접근가능한 자바 API

#### 장점
- 해당 클래스가 싱글턴임이 API에 명백히 드러난다.
- 간결하다.
  
### 2. 정적 팩터리 방식의 싱글턴
```java
public class Printer {
    private static final Printer INSTANCE = new Printer();
    private Printer() { ... }
    public static Printer getInstance() { return INSTANCE; }

    public void print() { ... }
}
```
`Printer.getInstance`는 항상 같은 객체의 참조를 반환하므로 제 2의 인스턴스를 만들지 않는다.
1의 상황과 마찬가지로 리플랙션 API에 의해 예외적인 상황이 발생할 수는 있다.

#### 장점
- API를 바꾸지 않고도 싱글턴이 아니게 변경할 수 있다.
    - 호출하는 스레드별로 다른 인스턴스를 넘기게 하는 등 ...
- 정적 패터리를 제네릭 싱글턴 팩터리로 만들 수 있다. (아이템 30)
- 정적 팩터리의 메서드 참조를 공급자(Supplier)로 사용할 수 있다는 점 (아이템 43, 44)
    - `Printer::getInstance` -> `Supplier<Printer>`

> [Supplier Interface](https://m.blog.naver.com/zzang9ha/222087025042): 매개변수를 받지 않고 단순히 무엇인가를 반환하는 함수형 인터페이스 -> Lazy Evaluation 
  

#### 유의점
1번과 2번 방식으로 만들어진 싱글턴 클래스를 직렬화하려면 단순히 `Serializable`을 구현한다고 선언하는 것만으로는 부족하다. 

모든 인스턴스 필드에 `transient`를 선언하고, `readResolve` 메서드를 제공해야만 역직렬화시에 새로운 인스턴스가 만들어지는 것을 방지할 수 있다. 만약 이렇게 하지 않으면 초기화해둔 인스턴스가 아닌 다른 인스턴스가 반환된다.
```java
private Object readResolve() {
    return INSTANCE;
}
```

#### 참고
Thread safe, Lazy Initialization, Double-checked locking, Holder 방식
- [https://jeong-pro.tistory.com/86](https://jeong-pro.tistory.com/86)
- [https://elfinlas.github.io/2019/09/23/java-singleton/](https://elfinlas.github.io/2019/09/23/java-singleton/)
- [https://medium.com/webeveloper/싱글턴-패턴](https://medium.com/webeveloper/%EC%8B%B1%EA%B8%80%ED%84%B4-%ED%8C%A8%ED%84%B4-singleton-pattern-db75ed29c36)
  
### 3. 열거 타입 방식의 싱글턴
```java
public enum Printer {
    INSTANCE;

    public void print() { ... }
}
```
앞의 예제들에 비해 더 간결하고, 추가 노력 없이 직렬화할 수 있으며, 아주 복잡한 직렬화 상황이나 리플렉션 공격에서도 제2의 인스턴스가 생기는 일을 막아준다.
**대부분 상황에서는 원소가 하나뿐인 열거 타입이 싱글턴을 만드는 가장 좋은 방법이다.** 단, 만들려는 싱글턴이 Enum 외의 클래스를 상속해야 한다면 이 방법은 사용할 수 없다.