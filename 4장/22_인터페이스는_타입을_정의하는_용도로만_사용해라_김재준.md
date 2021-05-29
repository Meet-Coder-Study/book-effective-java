**ITEM 22**

# 인터페이스는 타입을 정의하는 용도로만 사용하라



## 서론



클래스가 어떤 인터페이스를 구현한다는 것은 자신의 인스턴스로 무엇을 할 수 있는지를 

클라이언트에 얘기해주는 용도로만 사용해야한다.





## 용도에 맞지않은 인터페이스 사용



예를들어 상수 인터페이스가 있다.



```java
public interface PhysicalConstants {
    // 아보가드로 수 (1/몰)
    double AVOCADOS_NUMBER = 6.022_140_857e23;
    
    // 볼츠만 상수 (J/K)
    double BOLTZMANN_CONSTANT = 1.380_648_52e-23;
    
    // 전자 질량(kg)
    double ELECTRON_MASS = 9.109_383_56e-31;
}
```



클래스 내부에서 사용하는 상수는 외부 인터페이스(상수인터페이스) 가 아니라  내부 구현에 해당한다.

따라서 상수 인터페이스를 구현하는 것은 내부 클래스의 API로 노출하는 행위다.



상수 인터페이스는 사용자에게 혼란을 줄뿐아니라 클라이언트 코드가 내부 구현에 해당하는 이 상수들에 종속되게 한다.



## 상수를 제공하기위해선?

1. 열거타입으로 나타낸다(ITEM34)

   ```java
   public enum PhysicalConstantsEnum {
       AVOCADOS_NUMBER(6.022_140_857e23),
       BOLTZMANN_CONSTANT(1.380_648_52e-23),
       ELECTRON_MASS(9.109_383_56e-31);
   
       private final double value;
   
       PhysicalConstantsEnum(double value) {
           this.value = value;
       }
   
       public double getValue() {
           return value;
       }
   }
   ```

1. 인스턴스화를 할 수 없는 유틸리티 클래스에 담아 공개(ITEM4)

```java
public class PhysicalConstantsUtil {
    private PhysicalConstantsUtil() {
      //생성자를 private 으로 하여 인스턴스화 금지
    }
    
    // 아보가드로 수 (1/몰)
    public static double AVOCADOS_NUMBER = 6.022_140_857e23;

    // 볼츠만 상수 (J/K)
    public static double BOLTZMANN_CONSTANT = 1.380_648_52e-23;

    // 전자 질량(kg)
    public static double ELECTRON_MASS = 9.109_383_56e-31;
}
```



## 결론

인터페이스는 타입을 정의하는 용도로만 사용해야 한다. 상수 공개용 수단으로 사용하지 말자.



## 용어

상수 인터페이스 -> 인터페이스 내에 메서드 없이 오로지 static final 만 존재하는 인터페이스를 말한다.