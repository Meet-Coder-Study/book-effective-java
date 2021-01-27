# item17) 변경 가능성을 최소화하라

### 불변 클래스
 - 클래스의 인스턴스 값을 수정할 수 없는 클래스
 - String, Wrapper Class, BigInteger, BigDecimal
 
### 특징
 - 설계, 구현, 사용이 쉽다.
 - 오류에서 가변 클래스보다 훨씬 안전하다.
 
### 구현 방법
 - 객체의 상태를 변경하는 메서드를 제공하지 않음. (예. setter)
 - 클래스를 확장할 수 없도록 함.
   - 하위 클래스에서 객체가 변경될 수 있다.
   - 대표적인 방법은 final 키워드
 - 모든 필드를 final 로 선언.
   - 여러 스레드에서 접근해도 값이 바뀌지 않는다.
   - 다중 스레드 환경에서도 안전하다.
 - 모든 필드를 private 로 선언.
 - 자신 외에는 내부의 가변 컴포넌트에 접근할 수 없도록 함.
   - 클래스의 가변 참조 객체 필드에 대한 항목.
   - 클라이언트에서 이 필드의 참조를 얻으면 안된다.
   - 서버에서도 접근자 메서드로 이 필드값을 반환하면 안된다.
   - 생성자, 접근자, readObject 메서드 모두에서 방어적 복사를 수행한다.
   

<br>
<br>

   
### 불변 클래스 Person

```java
public final class Money {
    private final String currency;
    private final int amount;
    private final String[] cards;

    public Money(String currency, int amount) {
        this.currency = currency;
        this.amount = amount;
    }

    public Money plus(int additionMoney) {
        return new Money(this.name, this.amount + additionMoney);
    }

    public String[] getCards() {
        return cards.clone();
    }
}
```

 - 첫번째 항목
   - 클래스의 필드 currency, amount 를 바꿀 수 있는 메서드가 없음.
   - 나이를 늘리는 메서드는 새로운 Money 객체를 반환한다.
   
   ```
   함수형 프로그래밍 :  자료 처리를 수학적 함수의 계산으로 취급하고 상태와 가변 데이터를 멀리하는 프로그래밍 패러다임의 하나
   
   순수 함수 : 부작용이 없는 함수
   부작용(side-effect) : 함수의 실행이 외부에 영향을 미치지 않고, 외부에 의해 함수의 실행이 영향받지 않는 함수.
   
   나이를 늘릴 때, 해당 객체의 값(속성)을 변경한다면 그 객체를 사용하는 다른 객체들이 영향을 받는다.
   따라서 새로운 객체를 만들어 반환함으로써 side-effect 를 줄일 수 있다.
   ```
   ```
   새로운 객체를 반환하는 메서드는 명명 규칙으로 전치사를 사용.
   (add 말고 plus)
   
   Integer, Double 같은 클래스들의 valueOf() 
   ```
 
 - 두번째 항목
   - 클래스를 final 로 지정하여 확장이 불가능하게 함.
 
 - 세번째 항목
   - 모든 필드를 final 로 선언.
 
 - 네번째 항목
   - 모든 필드를 private 으로 선언.
   
 - 다섯번째 항목
   - 가변 참조 객체 cards 는 final 로 지정하더라도 안의 값에 대해서는 불변이 아님.
   - 가변 객체 cards 를 얻어오는 필드는 clone() 을 이용.

   
   
### 불변 객체의 장점
  1. 단순하다.
     1. 생성된 시점의 상태가 파괴될 때까지 간직
  1. 불변 객체는 근본적으로 스레드 안전하여 따로 동기화 할 필요 없다.
     1. Thread-safe 한 클래스를 만드는 가장 쉬운 방법.
     1. 불변 클래스는 한번 만든 인스턴스를 최대한 활용한다. (예. public static final 상)
     1. 따라서 불변 객체는 안심하고 공유할 수 있다.
     
     ```java
     java.math.BigDecimal
     
     public static BigDecimal valueOf(long val) {
         if (val >= 0 && val < ZERO_THROUGH_TEN.length)
             return ZERO_THROUGH_TEN[(int)val];
         else if (val != INFLATED)
             return new BigDecimal(null, val, 0, 0);
         return new BigDecimal(INFLATED_BIGINT, val, 0, 0);
     }
     
     
     private static final BigDecimal ZERO_THROUGH_TEN[] = {
         new BigDecimal(BigInteger.ZERO,       0,  0, 1),
         new BigDecimal(BigInteger.ONE,        1,  0, 1),
         new BigDecimal(BigInteger.TWO,        2,  0, 1),
         new BigDecimal(BigInteger.valueOf(3), 3,  0, 1),
         new BigDecimal(BigInteger.valueOf(4), 4,  0, 1),
         new BigDecimal(BigInteger.valueOf(5), 5,  0, 1),
         new BigDecimal(BigInteger.valueOf(6), 6,  0, 1),
         new BigDecimal(BigInteger.valueOf(7), 7,  0, 1),
         new BigDecimal(BigInteger.valueOf(8), 8,  0, 1),
         new BigDecimal(BigInteger.valueOf(9), 9,  0, 1),
         new BigDecimal(BigInteger.TEN,        10, 0, 2),
     };
     ```
     ```java
     @Before
     public void 설정() {
         tenUnder1 = BigDecimal.valueOf(9);
         tenUnder2 = BigDecimal.valueOf(9);
         tenOver1 = BigDecimal.valueOf(12);
         tenOver2 = BigDecimal.valueOf(12);
     
     @Test
     public void BigDecimal에서_10이하는_정적팩터리사용() {
         Assert.assertSame(tenUnder1, tenUnder2);
         Assert.assertEquals(System.identityHashCode(tenUnder1), System.identityHashCode(tenUnder2));
     
     @Test
     public void BigDecimal에서_10초과는_새로운객체생성() {
         Assert.assertNotSame(tenOver1, tenOver2);
         Assert.assertNotEquals(System.identityHashCode(tenOver1), System.identityHashCode(tenOver2));
     }
     ```
  1. 불변 객체는 객체의 필드로 쓰면 좋다.
     1. 불변식을 보장할 수 있다.
     1. Map 의 키나 Set 의 요소로 불변객체를 사용하면 불변식을 지키기 쉽다.
  1. 불변 객체는 실패 원자성을 제공한다.
     1. 실패 원자성 : 메서드 수행 중 예외가 발생해도 객체는 상태를 유지한다. (item76)
     
     
### 불변 객체의 단점
 1. 값이 다르면 무조건 독립된 객체를 생성해야 함.
    1. 원하는 객체를 만들기 까지 여러 단계를 거쳐야 한다면, 쓸모없는 객체가 많이 생긴다.
    1. 위 문제의 해결점으로 다단계 연산을 제하는 **가변 동반 클래스** 를 작성한다.
    ```java
    가변 동반 클래스 : 복잡한 다단계 연산을 기본으로 제공 해주는 클래스
    
    BigInteger 에는 package-private 으로 여러 클래스 존재
    String 에는 public 으로 제공되는 StringBuilder, StringBuffer 존재
    ```
    ![가변동반](./img/companion.png)
    
### 불변 클래스를 만드는 다른 방법
 - 두번째 규칙. 확장할 수 없도록 함
   - 모든 생성자를 private, package-private 으로 만들고, 정적 팩터리 메서드 제공.
   
 - 첫, 세번째 규칙. 모든 필드는 final 이고 변경자 메서드가 없음
   - 어떤 불변 클래스는 계산 비용이 큰 값을 final 이 아닌 필드에 캐싱하여 사용한다.
   ```java
   public final class String implements java.io.Serializable, Comparable<String>, CharSequence {
       @Stable
       private final byte[] value;
       private final byte coder;
       private int hash; // final 이 아닌 필
       private static final long serialVersionUID = -6849794470754667710L;
   
       ...
   
       public int hashCode() {
           int h = hash;
           if (h == 0 && value.length > 0) {
               hash = h = isLatin1() ? StringLatin1.hashCode(value)
                                     : StringUTF16.hashCode(value);
           }
           return h;
       }
   ```
   - 따라서 위의 규칙을 다음과 같이 완화할 수 있다.
      >어떤 메서드도 객체의 상태 중 외부에 비치는 값을 변경할 수 없다.
   
   
## 결론
 - 대부분의 경우 불변 클래스로 만들어야 한다. : setter 쓰지 말 것.
 - 성능 저하로 불변 클래스 구현이 힘들면 가변 동반 클래스를 이용.
 - 불변 클래스 구현이 불가하면 변경 가능 부분을 최소한으로 줄임.
   - 모든 필드는 private final 이어야 한다.
 - 객체의 상태를 초기화 하는 메서드는 생성자, 정적 팩터리 메서드 이외에는 없어야 함.
