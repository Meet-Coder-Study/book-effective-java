**ITEM 42**

# 익명 클래스보다는 람다를 사용하라



자바 8 이전에는 다음과 같이 익명클래스를 사용했다.

```java
@Test
@DisplayName("자바8 이전에 사용하던 익명클래스")
void anonymousClass() {
  List<String> word = new ArrayList<>(List.of("ab","abc","a"));
  
  Collections.sort(word, new Comparator<String>() {
    @Override
    public int compare(String o1, String o2) {
      return Integer.compare(o1.length(), o2.length());
    }
  });
  
}
```



하지만 자바 8 이후부터 <u>**인터페이스 내에 추상메서드가 단 하나만**</u> 존재하면 람다식을 사용할수 있게 되었다.

![image](https://user-images.githubusercontent.com/64793712/108632944-cc923000-74b4-11eb-8f82-a0bfa0bca906.png)



위에서 사용하던 익명클래스 방식을  람다표현식으로  다음과 같이 간결하게 표현 할 수 있다.

```java
Collections.sort(word,(s1,s2)->Integer.compare(s1.length(),s2.length()));
```



람다 표현식을 사용하게 되면 타입을 <u>생략</u> 할 수 있는데



생략할 수 있는 이유는

컴파일러가 대신 인터페이스에 대한 <u>타입추론을 대신</u> 해주기 때문이다.

컴파일러가 <u>타입추론을 할때 정보를 얻는 방법은 **대부분 제네릭**을</u> 통해 얻는다.



만약 위의 예제에서 **<u>List<String.></u>** 이 아니라 <u>로 타입인 List</u> 였다면 <u>컴파일 오류</u>가 발생한다.

```java
@Test
@DisplayName("로 타입 List 임으로 타입추론 불가(컴파일에러)")
void compileError(){
  List word = new ArrayList(List.of("ab","abc","a"));
  Collections.sort(word,(s1, s2) -> Integer.compare(s1.length(),s2.length()));
}
```

 해당 코드에서 s1, s2 의 타입 추론 할 수 없음으로 String 의 length()를 호출할수 없다.



## 열거타입에서의 람다사용



상수마다 메서드의 행동을 다르게 해야하는 경우 

상수별 메서드 구현(constant-specific method implementation)을 했었다.(Item34)

```java
public enum CARD {

  KAKAO("카카오"){
    @Override
    int point(int money) {
      return money/100;
    }
  },SAMSUNG("삼성"){
    @Override
    int point(int money) {
      return money*2/100;
    }
  },HYUNDAI("현대"){
    @Override
    int point(int money) {
      return money*3/100;
    }
  };

  private String name;
  abstract int point(int money);
}
 
```



해당 방식으로 구현하는 것보다 <u>함수객체(람다)를 인스턴스 필드에 저장해서</u> 보기 편한 코드로 변경해보자.

```java
public enum CARD {

  KAKAO("카카오",money->money/100),
  SAMSUNG("삼성",money->money*2/100),
  HYUNDAI("현대",money->money*3/100);

  private final String name;
  private final Function<Integer, Integer> op;

  CARD(String name ,Function<Integer,Integer> op) {
    this.name = name;
    this.op =op;
  }
  public int point(int money){
    return op.apply(money);
  }
  public String getName() {
    return name;
  }
}
```

람다 방식으로 구현하면 코드가 매우 깔끔해진다.

그렇다고 해서 상수 별 메서드 구현 방식을 쓰지 않는것은 아니다. 



**상수 별 메서드 구현 방식을 사용하는 상황**

1. 람다는 이름이 없고 문서화를 하기 어렵다

    -> 코드 자체로 동작이 명확히 설명되지 않거나 코드 줄 수가 많아지면 람다를 사용하지말자.

   

2. 열거 타입 생성자에 넘겨지는 인수들의 타입도 컴파일 타임에 추론된다.

    -> 열거 타입 생성자 안의 람다는 열거타입의 인스턴스 맴버에 접근할 수 없다.

   ![image-20210222035037055](/Users/kimjajan/Library/Application Support/typora-user-images/image-20210222035037055.png)





## 람다가 대체 할 수 없는 곳



1. 추상 클래스의 인스턴스를 만들 때 람다사용은 불가능하다.

2. 인터페이스의 추상 메서드가 여러 개면 람다로 표현 불가능하다.

3. 람다는 자신을 참조할 수 없다.

   -> 람다에서의 this 키워드는 바깥 인스턴스를 가리킨다.

   ```java
   public interface Foo {
     int anyThing(int i);
   }
   
   
   public class Test {
   
     private final int value = 20;
   
     public Foo foo = new Foo() {
       final int value = 10;
       @Override
       public int anyThing(int i) {
         return i + this.value; //익명 클래스는 인스턴스 자신을 가리킨다.
       }
     };
   
     public Foo foo2 = i -> i + this.value; //람다는 바깥 인스턴스를 가리킨다.
   
   }
   ```





## 주의사항



람다도 익명 클래스처럼 직렬화 형태가 구현별로(가상 머신 별로) 다를 수 있다.

따라서 람다를 직렬화 하는 일은 극히 삼가야 한다.(익명 클래스의 인스턴스도 마찬가지다)



직렬화를 해야 할 함수 객체가 있다면 private 중첩클래스(아이템24)의 인스턴스를 사용하자