# 리플렉션 보다는 인터페이스를 사용하라



## 리플랙션(reflection) 이 무엇일까?

리플렉션은 임의의 클래스에 접근할 수 있는 기능으로써 원래라면 접근 불가능한 필드 또는 생성불가능한 객체여도 
리플랙션을 이용하여 얼마든지 생성하거나 조작할 수 있다.

예시를 통해 알아보자.



## 생성할 수 없는 인스턴스 생성

```java
public class JaeJoon { 
    private String name;
    private int age;

    private JaeJoon(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public int getAge() {
        return age;
    }
}
```

여기 JaeJoon 이라는 클래스는 생성자의 접근제어자가 private 이고 기본생성자도 존재하지 않기 때문에 외부에서 절대로 인스턴스화 하지 못한다.

해당 클래스를 리플렉션을 이용하여 외부에서 생성하여 각각의 get 메서드를 호출해보자.



**예제**

```java
public static void main(String[] args) throws Exception{
    Constructor<JaeJoon> constructor =
        JaeJoon.class.getDeclaredConstructor(String.class, Integer.class); // (1)

    constructor.setAccessible(true); // (2) 

    JaeJoon kim = constructor.newInstance("Kim", 20); // (3) 

    System.out.println(kim.getName());
    System.out.println(kim.getAge());
}
```

1. 먼저 해당(JaeJoon) 클래스에 가져오고 싶은 생성자를 가져옵니다.
   -> 여기서 `getDeclaredConstructor()`  메서드에 넘기는 파라미터에 따라 맞는 생성자를 가져오게됩니다.

2. `setAccessible()` 은 해당 생성자의 접근지정자가 private 임으로  접근을 허용한다는 의미로 사용합니다.
3.  `newInstance()` 말 그대로 인스턴스를 생성하게 됩니다.



이 처럼 리플랙션을 이용하게된다면 **클래스의 생성자, 필드, 메서드** 등을 얼마든지 가져와서 사용할 수 있게됩니다.



리플랙션의 가장 큰 장점은 **런타임시점에 코드**를 조작할 수 있는것이 가장 큰 장점입니다.

따라서 **컴파일시점에 이용할 수 없는 클래스를 사용해야하는 경우**에만 사용해야합니다.



## 실제로 리플렉션코드를 작성한적이 있는가?

Spring 을 사용하고 있다면 내부적으로는 DI 등 다양하게 리플렉션을 이용하지만

실제로 사용하는 개발자가 리플렉션을 이용하는 경우는 얼마나 될까요..



(이렇게 끝나면 아쉽죠 ㅎ . ㅎ)

최근에 제가 리플랙션을 이용했어야 했던 경우를 간단하게 말하고자합니다.

[설명 은 여기서 !](https://k3068.tistory.com/101)





## 결론

1. 컴파일 타임에는 알 수 없는 클래스를 사용할때 리플렉션을 이용하도록하자!
2. 리플렉션을 이용하는것은 비용이 큰 작업입니다.(또한 사용하게된다면 적절한 예외처리는 필수 이겠죠?!!?)









## 



