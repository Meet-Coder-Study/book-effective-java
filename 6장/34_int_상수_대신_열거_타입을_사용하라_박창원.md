# 아이템 34. int 상수 대신 열거 타입을 사용하라



열거 타입(enumerated type)은 고정된 상수의 집합으로 구성된다.

예를 들어, 계절, 태양계의 행성이름, 카드의 패 이름(스페이드, 다이아몬드, 클럽, 하트) 등이 있다.

## 자바 5 이전의 열거 타입 사용방법

열거 타입이 자바에 추가되기 전에 가장 흔하게 사용하던 패턴은 *열거 패턴(enum pattern)*이다. 

열거 패턴의 구현 방법은 `static final int` 로 값을 정의하는 것이다. 

```java
public static final int APPLE_FUJI = 0;
public static final int APPLE_PIPPIN = 1;
public static final int APPLE_GRANNY_SMITH = 2;
```

열거 패턴은 구현하기 쉽지만 그만큼 허술하다. 열거 패턴의 문제점은 다음과 같다.

- 컴파일러가 이해하는 값은 정수이므로 다른 값을 실수로 사용하더라도 알아차리지 못한다는 것이다.
- 정수 값이 변하면 클라이언트는 재 컴파일해야 한다. 
- 정수형 열거 타입의 이름을 문자열로 표현하기 힘들다. `APPLE_FUJI`를 프린트할 수 있는가?



## 문자열 열거 패턴

자바는 5버전 부터 문자열 열거 패턴(String enum pattern)을 지원한다.

문자열 열거 패턴은 우리가 아는 enum으로 타입을 구성하는 것이다.

```java
public enum Apple { FUJI, PIPPIN, GRANNY_SMITH }
public enum Orange { NAVEL, TEMPLE, BLOOD}
```



자바 열거형 타입 자체는 어떻게 구현될까?

- 열거 타입 또한 클래스로 취급되어, 각 필드마다 하나의 인스턴스로 구성된다.
- 각 필드는  public static final 로 정의된다.
- 싱글톤으로 구현되어 있다.



열거 타입의 장점은 무엇일까?

- 컴파일 타임에 안전하다. `Apple` 이라는 열거 타입이 기대되는 자리에 다른 열거 타입은 올 수 없으며, null도 올 수 없다.



열거 타입을 풍부하게 사용해보자. 다음은 태양계의 8개 행성들을 열거 타입으로 구현했다.

```java
public enum Planet {
	MERCURY(3.302e+23,2.439e6),
	VENUS(4.869e+24,6.052e6),
	EARTH(5.975e+24, 6.378e6),
	MARS(6.419e+23,3.393e6),
	JUPITER(1.899e+27,7.149e7),
	SATURN(5.685e+26,6.027e7),
	URAUS(8.683e+25,2.556e7),
	NEPTUNE(1.024e+26,2.477e7);

	private final double mass;
	private final double radius;
	private final double surfaceGravity;
  // 생성자, getter, setter
  
	public double surfaceWeight(double mass) {
		return mass * surfaceGravity;
	}
}
```

자바는 enum 타입과 연관된 값들을 응집도 있게 사용할 수 있도록 지원한다. 

각 행성들은 질량과 반지름을 가지고 있을 것이다. 이러한 특성들을 enum 타입 내부에 함께 구현할 수 있다. 



## 열거 타입으로 계산기 구현하기

열거 타입으로 계산기를 구현해보자. 책에서 나온 내용이지만, 단순하게 생각했다면 이런식으로 구현할 것이다.

```java
public enum Operation {
  PLUS, MINUS, TIMES, DIVIDE;
  
  public double apply(double x, double y) {
    switch (this) {
      case PLUS: return x + y;
      case MINUS: return x - y;
      case TIMES: return x * y;
      case DIVIDE: return x / y;
    }
    throw new AssertionError("알 수 없는 연산 :" + this);
  }
}
```

이 코드에는 문제점이 있다.

- throw 구문 없이는 컴파일되지도 못한다. 
- 제곱과 같은 새로운 Operation 타입이 추가된다면, switch 구문의 case를 추가해야하며 이 부분에서 실수하기 쉽다.



위 단점을 개선한 코드를 작성해보자.

```java
public enum Operation {
  PLUS {public double apply(double x, double y) { return x + y; }},
  MINUS {public double apply(double x, double y) { return x - y; }},
  TIMES {public double apply(double x, double y) { return x * y; }},
  DIVIDE {public double apply(double x, double y) { return x / y; }};
  public abstract double apply(double x, double y);
}
```

열거 타입에 추상 메서드를 구현한 것이 특징이다. Operation 각 타입에 추상 메서드를 구현하였다.

이러한 메서드를 상수-한정 메서드 구현(constant-specific method implementation)이라고 한다.

이제 구현한 것을 사용하는 클라이언트 코드를 작성해보자.

```java
public static void main(String[] args) {
  double x = 2.0;
  double y = 4.0;
  for (Operation op : Operation.values()) {
    System.out.printf("%f %s %f = %f%n", x, op, y, op.apply(x, y));
  }
}
```

>2.000000 PLUS 4.000000 = 6.000000</br>
>2.000000 MINUS 4.000000 = -2.000000</br>
>2.000000 TIMES 4.000000 = 8.000000</br>
>2.000000 DIVIDE 4.000000 = 0.500000</br>

결과가 썩 만족스럽지 못하다. PLUS, MINUS 등의 문자가 수학 기호로 표시되면 가독성을 늘릴 수 있을 것이다.

```java
public enum Operation {
	PLUS("+") {public double apply(double x, double y) { return x + y; }},
	MINUS("-") {public double apply(double x, double y) { return x - y; }},
	TIMES("*") {public double apply(double x, double y) { return x * y; }},
	DIVIDE("/") {public double apply(double x, double y) { return x / y; }};
	public abstract double apply(double x, double y);

	private final String symbol;

	Operation(String symbol) { this.symbol = symbol; }

	@Override public String toString() { return symbol; }
}
```

위와 같이 개선한 코드는 연관 값을 추가하였다. 이 값을 추가하기 위해 필드와 생성자, toString 또한 추가하였다는 것에 유의하자.

> 2.000000 + 4.000000 = 6.000000</br>
> 2.000000 - 4.000000 = -2.000000</br>
> 2.000000 * 4.000000 = 8.000000</br>
> 2.000000 / 4.000000 = 0.500000</br>



