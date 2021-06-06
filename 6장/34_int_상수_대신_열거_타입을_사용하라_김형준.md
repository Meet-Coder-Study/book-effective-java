# Item34 int 상수 대신 열거 타입을 사용하라

아래와 같이 정수 열거 패턴(int enum pattern) 기법에는 단점이 많다.

    public static final int name = 0;
    public static final int departmentName = 1;
    public static final int companyName = 2;
    public static final int phoneNumber = 3;
    public static final int email 4;
 
 1. 타입 안전을 보장할 수 없다.
 2. 표현력도 좋지 않다.
 3. 동등 연산자(==)로 비교하더라도 컴파일러가 아무런 경고 메시지를 출력하지 않는다.
 4. 컴파일하면 그 값이 클라이언트 파일에 그대로 새겨지기 때문에, 상수의 값이 바뀌면 클라이언트도 반드시 다시 컴파일 해야 한다.
 
 이러한 정수 열거 패턴 뿐만이 아니라 문자열 열거 패턴(String enum pattern) 기법이 있는데, 해당 책에서는 문자열에 대한 오타가 있어 확인할 수 없어 런타임 버그가 생길 수 있어 안좋다고 한다. 

    public static final String blank = "";
    public static final String minus = "-";
    public static final Stirng atSign = "@";


이러한  열거 패턴을 자바에서는 깔끔히 씻어주는 동시에 여러 장점을 안겨주는 대안을 제시했는데, 그것이 바로 **EnumType** 이다.

    public enum Search { NAME, DEPARTMENTNANE, COMPANYNAME, PHONENUMBER, EMAIL }

자바의 **EnumType**은 완전한 형태의 클래스라서 다른 언어의 열거 타입보다 훨씬 강력하다고 한다.

# EnumType

**EnumType은 이렇다.**

 1. 클래스이다.
 2. 상수 하나당 자신의 인스턴스를 하나씩 만들어 public static final 필드로 공개한다.
 3. 밖에서 접근할 수 있는 constructor를 제공하지 않기 때문에 인스턴스를 직접 생성하거나 확장 할 수 없기 때문에 **EnumType** 선언으로 만들어진 인스턴스 들은 딱 하나만 존재한다.
 4. SingleTon은 원소가 하나뿐인 **EnumType**이라 할 수 있고, 거꾸로 **EnumType**은 SingleTon을 일반화한 형태라고 볼 수 있다.
 5. 컴파일타임 안정성을 제공한다.
 6.  각자의 이름공간이 있어서 이름이 같은 상수도 평화롭게 공존한다.
 7. **EnumType**의 toString 메서드는 출력하기에 적합한 문자열을 내어준다.

추가로 **EnumType**은 임의의 메서드나 필드를 추가할 수 있고, 임의의 인터페이스를 구현하게 할 수도 있다.

## 어떠한 상황에서 Method나 필드를 추가할까? 

    public enum IPhone {
	    IPONE11(6.1, "A13", 1200),
	    IPONE11PRO(5.8, "A13", 1200),
	    IPONE11PROMAX(6.5, "A13", 1200),
	    IPONESE(4.7, "A13", 700),
	    IPONE12(6.1, "A14", 1200),
	    IPONE12MINI(5.4, "A14", 1200),
	    IPONE12PRO(6.1, "A14", 1200),
	    IPONE12PROMAX(6.7, "A14", 1200);

		private final double inch;
		private final String process;
		private final int cameraPixel; 
		
		prvate static final double centimeter = 2.54;
		
		IPone(double inch, String process, int cameraPixel) {
			this.inch = inch;
			this.process = process;
			this.cameraPixel = cameraPixel;
		}
		
		public double inch();
		public String process();
		public int cameraPixel();

		public double calcurateInchToCentimeter(doublic inch) {
			return inch * centimeter;
		}
    }

 1. **EnumType** 상수 각각을 특정 데이터와 연결지으려면 생성자에서 데이터를 받아 인스턴스 필드에 저장하면 된다.
 2. 이 때 **EnumType**은 근본적으로 불변이라 모든 필드는 final이어야 한다.
 3. 그리고, 필드를 public으로  선언해도 되지만, private으로 두고 별도의 public 접근자 메서드를 두는게 낫다.

## values

**EnumType**은 자신 안에 정의된 상수들의 값을 배열에 담아 반환하는 정적 메서드인 values를 제공한다.,


## **EnumType**을 선언한 Class 혹은 그 패키지에서만 유용한 기능은 pirvate이나 package-private 메서드로 구현하라.

이렇게 구현된 EnumType 상수는 자신을 선언한 클래스 혹은 패키지에서만 사용할 수 있는 기능을 담게 된다.

--> 해당 기능이 노출해야할 할 합당한 이유가 없다면 pirvate으로 혹은 package-pirvate으로 선언하라 ( item 15)

--> 널리 쓰인다면 톱레벨 클래스로 만들고, 특정 톱레벨 클래스에서만 쓰인다면 해당 클래스의 멤버 클래스로 만든다. (item 24)

## 상수별 메서드 구현(constant-specific method implementaltion)

    public enum Operation {
	    PLUS("+") {
		    public douvle apply(double x, double y) { return x + y };
		},
		MINUS("-") {
			public douvle apply(double x, double y) { return x - y };
		},
		TIMES("*") {
			public douvle apply(double x, double y) { return x * y };
		},
		DIVIDE("/") {
			public douvle apply(double x, double y) { return x / y };
		};
		
		private final String symbol;
		Operation(String symbol) {this.symbol = symbol;}
		@Override public String toString() {return symbol;}
		public abstract double apply(double x, double y);
    }

**EnumType**은 상수별로 다르게 동작하는 코드를 구현하는 더 나은 수단을 제공한다.

**EnumType**에 apply라는 추상 메서드를 선언하고 각 상수에서 자신에 맞게 재정의하는 방법이다. 


## 전략 열거 타입 패턴

	 enum PayrollDay {  
		 MONDAY(WEEKDAY),  
		 TUESDAY(WEEKDAY),  
		 WEDNESDAY(WEEKDAY),  
		 THURSDAY(WEEKDAY),  
		 FRIDAY(WEEKDAY),  
		 SATURDAY(WEEKEND),  
		 SUNDAY(WEEKEND);  
    
		 private final PayType payType;  
	    
		 PayrollDay(PayType payType) {  
			 this.payType = payType;  
		 }  
	    
		 int pay(int minutesWorked, int payRate) {  
			 return payType.pay(minutesWorked, payRate);  
		 }  
	    
		 enum PayType {  
			 WEEKDAY {  
				 int overtimePay(int minutesWorked, int payRate) {  
					 return minutesWorked <= MINS_PER_SHIFT ? 0 : (minutesWorked - MINS_PER_SHIFT) * payRate / 2;  
				 }  
			 },  
	    
			 WEEKEND {  
				 int overtimePay(int minutesWorked, int payRate) {  
					 return minutesWorked * payRate / 2  
				 }  
			 };  
	    
			 abstract int overtimePay(int minutesWorked, int payRate);  
			 private static final int MINS_PER_SHIFT = 8 * 60;  
	    
			 int pay(int minutesWorked, int payRate) {  
				 int basePay = minutesWorked & payRate;  
				 return basePay + overtimePay(minutesWorked, payRate);  
			 }  
		 }  
	}

열거 타입 상수 일부가 같은 동작을 공유한다면 해당 패턴을 사용하자.

# 결론

**EnumType**은 필요한 원소를 컴파일러타임에 다 알 수 있는 상수 집합이라면 항상 열거 타입을 사용하자.
**EnumType**에 정이된 상수 개수가 영원히 고정 불변일 필요는 없다. 그 이유는 나중에 상수가 추가돼도 바이너리 수준에서 호환되도록 설계되었기 때문이다.




