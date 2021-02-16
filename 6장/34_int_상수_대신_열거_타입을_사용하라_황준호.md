### 아이템34. int 상수 대신 열거 타입을 사용하라

정수 열거 패턴과 열거 타입의 예로는 다음과 같다.

```java
//정수 열거 패턴(따라하지 말것!)
public class Constants {
    public static final int MONDAY = 0;
    public static final int TUESDAY = 1;
    public static final int WEDNESDAY = 2;
    public static final int THURSDAY = 3;
    public static final int FRIDAY = 4;
    public static final int SATURDAY = 5;
    public static final int SUNDAY = 6;

    public static final int APPLE = 0;
    public static final int ORANGE = 1;
    public static final int GRAPE = 2;
    public static final int MELON = 3;
}
--------------------------------------------------------------
//열거 타입
public enum Day {
    MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY
}
public enum Fruit {
    APPLE, ORANGE, GRAPE, MELON
}
```

다음과 같은 이유로, 정수 열거 패턴 보다는 열거 타입을 사용하는게 더 좋다.

- 어떤 게 타입 안정성이 좋은가?

  - 정수 열거 패턴 :-1:

    ```java
    public void function(int day) { //요일 관련 상수만 들어가야 하는 메서드
        ...
    }
    
    ...
    function(MONDAY); //OK
    function(APPLE); //과일 관련 상수가 들어가도 컴파일러는 모른다!
    ```

  - 열거 타입 :+1:

    ```java
    public void function(Day day) { //요일만 들어가야 하는 메서드
        ...
    }
    
    ...
    function(MONDAY); //OK
    function(APPLE); //컴파일 에러!
    ```

- 어떤 게 프로그램이 깨지기 어려운가?

  - 정수 열거 패턴 :-1:
    - 컴파일하면 그 값이 클라이언트 파일에 그대로 새겨진다
    - 상수의 값이 바뀌면 클라이언트도 다시 컴파일 해야 한다
  - 열거 타입 :+1:
    - 공개되는 건 필드의 이름뿐이라서 추가하거나 순서를 바꿔도 다시 컴파일하지 않아도 된다.
    - 상수를 하나 제거하더라도 제거한 상수를 참조하지 않는 클라이언트에는 아무 영향이 없다. 제거한 상수를 참조하는 클라이언트에는 컴파일 오류가 발생하여 즉시 알아챌 수 있다.

- 문자열로 출력하기에 좋은가?

  - 정수 열거 패턴 :-1:

    - ```java
      System.out.println(Constants.MONDAY); //0
      ```

    - 의미가 아닌 숫자로만 보여서 썩 도움이 안된다

  - 열거 타입 :+1:

    - ```java
      System.out.println(Day.MONDAY); //MONDAY
      ```

    - `toString()`는 출력하기에 적합한 메서드를 내어준다

그 외에도 열거 타입의 장점은 많다

- 인스턴스를 통제할 수 있다

  - 클라이언트가 인스턴스를 직접 생성하거나 확장할 수 없다 (public 생성자를 제공하지 않으므로)
  - 열거 타입 인스턴스들은 딱 하나만 존재함이 보장된다

- 임의의 메서드나 필드를 추가할 수 있고 임의의 인터페이스를 구현하게 할 수도 있다

  - ```java
    public enum Day {
        MONDAY("월요일", x -> x),
        TUESDAY("화요일", x -> x),
        WEDNESDAY("수요일", x -> x),
        THURSDAY("목요일", x -> x),
        FRIDAY("금요일", x -> x),
        SATURDAY("토요일", x -> 1.5 * x),
        SUNDAY("일요일", x -> 1.5 * x);
    
        private final String korean;
        private final Function<Double, Double> additionalPay;
    
        Day(String korean, Function<Double, Double> additionalPay) {
            this.korean = korean;
            this.additionalPay = additionalPay;
        }
    
        public static void printAllHourlyWage(double hourlyWage) {
            System.out.printf("기본 시급이 %.2f원 일때,%n", hourlyWage);
            for (Day day : values()) {
                System.out.printf("%s의 시급은 %.2f원 입니다.%n", day.korean, day.additionalPay.apply(hourlyWage));
            }
        }
    }
    ---------------------------------------------------------------------------------------------------------
    Day.printAllHourlyWage(10000);
    //기본 시급이 10000.00원 일때,
    //월요일의 시급은 10000.00원 입니다.
    //화요일의 시급은 10000.00원 입니다.
    //수요일의 시급은 10000.00원 입니다.
    //목요일의 시급은 10000.00원 입니다.
    //금요일의 시급은 10000.00원 입니다.
    //토요일의 시급은 15000.00원 입니다.
    //일요일의 시급은 15000.00원 입니다.
    ```

  - 열거 타입 상수 일부가 같은 동작을 공유할땐 전략 열거 타입으로 개선할 수 있다

    ```java
    public enum Day {
        MONDAY("월요일", WEEKDAY),
        TUESDAY("화요일", WEEKDAY),
        WEDNESDAY("수요일", WEEKDAY),
        THURSDAY("목요일", WEEKDAY),
        FRIDAY("금요일", WEEKDAY),
        SATURDAY("토요일", WEEKEND),
        SUNDAY("일요일", WEEKEND);
    
        private final String korean;
        private final DayType dayType;
    
        Day(String korean, DayType dayType) {
            this.korean = korean;
            this.dayType = dayType;
        }
    
        public static void printAllHourlyWage(double hourlyWage) {
            System.out.printf("기본 시급이 %.2f원 일때,%n", hourlyWage);
            for (Day day : values()) {
                System.out.printf("%s의 시급은 %.2f원 입니다.%n", day.korean, day.dayType.apply(hourlyWage));
            }
        }
    
        enum DayType {
            WEEKDAY(x -> x), WEEKEND(x -> 1.5 * x);
    
            private final Function<Double, Double> additionalPay;
    
            DayType(Function<Double, Double> additionalPay) {
                this.additionalPay = additionalPay;
            }
    
            public double apply(double hourlyWage) {
                return additionalPay.apply(hourlyWage);
            }
        }
    }
    ```

    

#### 결론

정수 열거 패턴보다는 열거 타입을 쓰자. 읽기 쉽고 강력하다.

열거 타입에서 상수마다 다르게 동작해야 할 땐 상수별로 메서드를 구현해서 사용하자, 일부 상수가 같은 행동을 공유한다면 전략 열거 패턴을 사용하자