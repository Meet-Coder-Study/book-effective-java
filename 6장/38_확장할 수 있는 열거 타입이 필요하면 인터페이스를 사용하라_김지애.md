# [Effective Java] item 38. 확장할 수 있는 열거 타입이 필요하면 인터페이스를 사용하라

## 핵심 정리
- 열거타입 자체는 확장할 수 없지만, `인터페이스와 그 인터페이스를 구현하는 기본 열거 타입을 함께 사용해 같은 효과를 낼 수 있다.`
- 이를 통해 클라이언트는 이 인터페이스를 구현해 자신만의 열거 타입(혹은 다른 타입)을 만들 수 있다.
- API가 (기본 열거 타입을 직접 명시하지 않고) 인터페이스 기반으로 작성되었다면 기본 열거 타입의 인스턴스가 쓰이는 모든 곳을 새로 확장한 열거 타입의 인스턴스로 대체해 사용할 수 있다.

---

### 타입 안전 열거 타입 
```java
public final class Direction {

  public static final Direction NORTH = new Direction("N");
  public static final Direction SOUTH = new Direction("S");
  public static final Direction EAST = new Direction("E");
  public static final Direction WEST = new Direction("W");

  private Direction() {
    ...
  }
}
```
- jdk1.5 이전에 enum이 없을 때 사용하던 방식

### 열거 타입
```java
enum Direction {
    NORTH, SOUTH, EAST, WEST;
}
```

- `열거 타입은 거의 모든 상황에서 타입 안전 열거 패턴(typesafe enum pattern) 보다 우수`하다. 
- 단, 예외가 하나 있으니, 타입 안전 열거 패턴은 확장할 수 있으나 `열거 타입은 확장할 수 없다`.

- `연산 코드(operation code)`에서 이따금 API가 제공하는 `기본 연산 외 사용자 확장 연산을 추가할 수 있도록 열어줘야 할 때`외에는 대부분의 상황에서 열거 타입을 확장하는 것은 좋지 않은 생각이다.
    - 열거 타입을 확장하면, 확장한 타입의 원소는 기반 타입의 원소로 취금하지만 그 반대는 성립하지 않을 수 있다.
    - 열거 타입을 확장하면 기반 타입과 확장 타입들의 원소 모두를 순회할 방법도 마땅하지 않다.
- 열거 타입을 확장하려면 `열거 타입이 임의의 인터페이스를 구현할 수 있다는 사실을 이용`하면 된다.
- `연산 코드용 인터페이스를 정의`하고 `열거 타입이 이 인터페이스를 구현`하게 하면 된다. 이 때 `열거 타입이 그 인터페이스의 표준 구현체 역할`을 한다.

다음은 Operation 타입을 확장할 수 있게 만든 코드이다.

##### 인터페이스를 이용해 확장 가능 열거 타입을 흉내냈다.
```java
public interface Operation {
    double apply(double x, double y);
}
```

```java
public enum BasicOperation implements Operation {
    PLUS("+") {
        public double apply(double x, double y) { return x + y; }
    },
    MINUS("-") {
        public double apply(double x, double y) { return x - y; }
    },
    TIMES("*") {
        public double apply(double x, double y) { return x * y; }
    },
    DIVIDE("/") {
        public double apply(double x, double y) { return x / y; }
    };

    private final String symbol;

    BasicOperation(String symbol) {
        this.symbol = symbol;
    }

    @Override public String toString() {
        return symbol;
    }
}
```
- 열거 타입인 BasicOperation은 확장할 수 없지만 인터페이스인 Operation은 확장할 수 있고, 이 인터페이스를 연산의 타입으로 사용하면 된다.

- 예를 들어 앞의 연산 타입을 확장해 지수 연산(EXP)와 나머지 연산(REMAINDER)을 추가해보자. 이를 위해 우리가 할 일은 Operation 인터페이스를 구현한 열거 타입을 작성하는 것 뿐이다.

##### 확장 가능 열거 타입
 ```java
 public enum ExtendedOperation implements Operation {
    EXP("^") {
        public double apply(double x, double y) {
            return Math.pow(x, y);
        }
    },
    REMAINDER("%") {
        public double apply(double x, double y) {
            return x % y;
        }
    };
    private final String symbol;
    ExtendedOperation(String symbol) {
        this.symbol = symbol;
    }
    @Override public String toString() {
        return symbol;
    }
 }
 ```

 새로 작성한 연산은 기존 연산을 쓰던 곳이면 어디든 쓸 수 있다. Operation 인터페이스를 사용하도록 작성되어 있기만 하면 된다.

 개별 인스턴스 수준에서 뿐 아니라 타입 수준에서도, 기본 열거 타입 대신 확장된 열거 타입을 넘겨 확장된 열거 타입의 원소 모두를 사용하게 할 수도 있다. 

##### 첫번째 대안) ExtendedOperation의 모든 원소 테스트하기
```java
public static void main(String[] args) {
    double x = Double.parseDouble(args[0]);
    double y = Double.parseDouble(args[1]);
    test(ExtendedOperation.class, x, y);
}
private static <T extends Enum<T> & Operation> void test(
        Class<T> opEnumType, double x, double y) {
    for (Operation op : opEnumType.getEnumConstants())
        System.out.printf("%f %s %f = %f%n",
                x, op, y, op.apply(x, y));
}
```

- main 메서드는 test 메서드에 ExtendedOperation의 class 리터럴을 넘겨 확장된 연산들이 무엇인지 알려준다. 여기서 class 리터럴은 한정적 타입 토큰 역할을 한다.

- opEnumType 매개변수의 선언(`<T extends Enum<T> & Operation> Class <T>`)은 솔직히 복잡한데, `Class 객체가 열거 타입인 동시에 Oepration의 하위 타입이어야 한다`는 뜻이다.

- 열거 타입이어야 원소를 순회할 수 있고, Operation이어야 원소가 뜻하는 연산을 수행할 수 있기 때문이다.

##### 두번쨰 대안) Class 객체 대신 한정적 와일드카드 타입인 Collection<? extends Operation>을 넘기는 방법
```java
public static void main(String[] args) {
    double x = Double.parseDouble(args[0]);
    double y = Double.parseDouble(args[1]);
    test(Arrays.asList(ExtendedOperation.values()), x, y);
}
private static void test(Collection<? extends Operation> opSet,
                            double x, double y) {
    for (Operation op : opSet)
        System.out.printf("%f %s %f = %f%n",
                x, op, y, op.apply(x, y));
}
```

두번째 코드는 그나마 덜 복잡하고 여러 구현 타입의 연산을 조합해 호출할 수 있게 되었다. 반면, 특정 연산에서는 EnumSet과 EnumMap을 사용하지 못한다.

두 대안 프로그램 모두 명령줄 인수로 4와 2를 넣어 실행하면 다음 결과를 출력한다.

```
4.000000 ^ 2.000000 = 16.000000
4.000000 % 2.000000 = 0.000000
```

---

#### 열거 타입에서 인터페이스를 이용해 확장 하는 경우 사소한 문제점
- 인터페이스를 이용해 확장 가능한 열거 타입을 흉내 내는 방식에는 `열거 타입끼리 구현을 상속할 수 없다`는 사소한 문제점이 있다.

- 아무 상태에도 의존하지 않는 경우에는 디폴트 구현을 이용해 인터페이스에 추가하는 방법이 있다. 반면 Operation 예는 연산 기호를 저장하고 찾는 로직이 BasicOperation과 ExtendedOepration 모두에 들어가야만 한다.

- 이 경우에는 중복량이 적으니 문제되진 않지만, 공유하는 기능이 많다면 그 부분을 별도의 도우미 클래스나 정적 도우미 메서드로 분리하는 방식으로 코드 중복을 없앨 수 있을 것이다.

---
#### 참고

- 자바 라이브러리에도 이번 아이템에서 소개한 패턴을 사용한다. 그 예로 java.nio.file.LinkOption 열거 타입은 CopyOption과 OpenOption 인터페이스를 구현했다.
    ![item38_1](https://user-images.githubusercontent.com/37948906/108061147-3bc5e980-709b-11eb-8386-04a29110337c.PNG)

---

### 참고 자료
- Effective Java 3/E