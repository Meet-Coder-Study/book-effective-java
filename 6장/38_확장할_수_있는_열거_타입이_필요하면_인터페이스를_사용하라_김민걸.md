# 확장할 수 있는 열거 타입이 필요하면 인터페이스를 사용하라
### enum 타입에는 extends 를 쓸 수 없다 !

* enum 의 바이트코드를 보면 자동으로 java.lang.Enum 을 상속하고 있다.

```java
final enum Chapter6/Day38/enumEx$Day extends java/lang/Enum {
	...
}
```

* 자바는 다중 상속이 지원되지 않기 때문에, enum 클래스에 다른 클래스를 상속받을 수 없다.


### 대부분의 상황에서 enum 을 확장하는 것은 안좋다 !
* enum 은 상수 집합인데, 하위 상수는 상위 타입의 요소로 인정하지만 상위 타입의 상수는 하위 타입으로 인정못하는 것은 이상하다.
* 상위, 하위타입의 모든 상수를 순회하는 방법이 없다.
* 설계와 구현이 복잡해진다.


### 아주 가끔은 enum 을 확장해야 할 수 도 있다.
* opcode : 연산코드, 기계마다 가지는 기계가 할 수 있는 연산.
* 기계마다 다른 연산을 제공하거나, 기본 연산을 확장한 연산을 제공할 수 있어야 한다.


### 상속이 안되면 인터페이스 구현으로 !
* 연산 인터페이스
```java
public interface  Operation {
    double apply(double x, double y);
}
```

* 지구인 연산 enum
```java
public enum EarthOperation implements Operation {
    PLUS("+") {
        @Override
        public double apply(double x, double y) {
            return x + y;
        }
    },
    MINUS("-") {
        @Override
        public double apply(double x, double y) {
            return x - y;
        }
    },
    TIMES("*") {
        @Override
        public double apply(double x, double y) {
            return x * y;
        }
    },
    DIVIDE("/") {
        @Override
        public double apply(double x, double y) {
            return x / y;
        }
    };

    private final String symbol;

    BasicOperation(String symbol) {
        this.symbol = symbol;
    }
}
```

* 인터페이스를 구현하므로 abstract 메서드 선언은 하지 않고 상수별로 메서드를 재정의할 수 있다.
* 모든 상수의 재정의 메서드가 같다면, enum 메서드 구현에서 @Override 로 재정의하면 된다.



그런데 화성인의 연산은 더하기와 빼기만 있고, 지구인의 결과에 *2 를 한다고 해보자.
그렇다면 화성인만의 enum 이 새로 필요하다.

```java
public enum MarsOperation implements Operation {
    MARS_PLUS("++") {
        @Override
        public double apply(double x, double y) {
            return (x + y) * 2;
        }
    },
    MARS_MINUS("--") {
        @Override
        public double apply(double x, double y) {
            return (x - y) * 2;
        }
    };

    private final String symbol;

    MarsOperation(String symbol) {
        this.symbol = symbol;
    }
}
```




### 새로운 화성인의 연산은 기존에 쓰던 지구인의 연산을 갈아치울 수 있다 !
* 기존에 쓰던 지구인의 enum 타입이 인터페이스인 Operation 으로 선언되어 있기만 하면.
* 구현체 객체만 EarthOperation 상수에서, MarsOperation 상수로 바꿔주면 된다.


### 타입 수준으로 다형성을 적용할 수 있다.
* 다음과 같이  하나의 메서드로 Operation 하위 타입 모두를 사용할 수 있다.

```java
public static void main(String[] args) {
    double x = 1.5;
    double y = 3.0;

    System.out.println("--지구--");
    test(EarthOperation.class, x, y);
    System.out.println("--화성--");
    test(MarsOperation.class, x, y);
}

```


* 한정적 타입 매개변수를 이용한 방법
```java
private static <T extends Enum<T> & Operation> void test (Class<T> opEnumType, double x, double y) {
    for (Operation op : opEnumType.getEnumConstants()) {
        System.out.printf("%f %s %f = %f%n", x, op, y, op.apply(x, y));
    }
}

```


* 한정적 와일드 카드를 이용한 방법

```java
public static void main(String[] args) {
    double x = 1.5;
    double y = 3.0;

    System.out.println("--지구--");
    test(EnumSet.allOf(EarthOperation.class), x, y);
    System.out.println("--화성--");
    test(EnumSet.allOf(MarsOperation.class), x, y);
    System.out.println("--지구의 더하기와 화성의 더하기--");
    test(Set.of(BasicOperation.PLUS, MarsOperation.MARS_PLUS), x, y);
}

private static void test (Collection<? extends Operation> opSet, double x, double y) {
    for (Operation op : opSet) {
        System.out.printf("%f %s %f = %f%n", x, op, y, op.apply(x, y));
    }
}
```

이 방법의 장점은 Operation 을 구현한 다른 타입의 enum 상수들을 조합해서 인자로 보낼 수 있다.


반면 다른 enum 의 상수에 대해선 EnumSet, EnumMap 이 제한된다.
```java
Set<MarsOperation> s = EnumSet.of(MarsOperation.MARS_PLUS, MarsOperation.MARS_MINUS); // 가능

Set<Operation> s = EnumSet.of(EarthOperation.MINUS, EarthOperation.PLUS); // 불가능

Set<Operation> s = EnumSet.of(EarthOperation.MINUS, MarsOperation.MARS_PLUS); // 불가능
```


### 자바 라이브러리의 예시

java.nio.file.LinkOption
```
public enum LinkOption implements OpenOption, CopyOption {
    /**
     * Do not follow symbolic links.
     *
     * @see Files#getFileAttributeView(Path,Class,LinkOption[])
     * @see Files#copy
     * @see SecureDirectoryStream#newByteChannel
     */
    NOFOLLOW_LINKS;
}

```





