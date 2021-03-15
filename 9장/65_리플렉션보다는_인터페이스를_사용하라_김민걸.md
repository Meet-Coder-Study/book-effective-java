# 리플렉션보다는 인터페이스를 사용하라.

## *Reflection*

> Reflection is commonly used by programs which require the ability to examine or modify the runtime behavior of applications running in the Java virtual machine. This is a relatively advanced feature and should be used only by developers who have a strong grasp of the fundamentals of the language. With that caveat in mind, reflection is a powerful technique and can enable applications to perform operations which would otherwise be impossible.
> 출처 : 오라클 API 문

 - 리플렉션이란, 런타임에 동작하는 것.
 
 - 구체적인 클래스 타입을 알지 못해도 컴파일된 클래스파일을 통해 그 클래스의 메소드, 타입, 변수들을 접근할 수 있도록 해주는 자바 API
 - java.lang.reflection 사용
 
#### 구체적인 클래스 타입을 알지 못한다 ?
 - Runner 의 타입을 몰라 Object 타입을로 쓰면
 - 당연히 run() 을 쓰지 못하겠죠 ?
```java
public static void main(String[] args) {
    Object runner = new Runner();
    runner.run(); // 컴파일 오류 !
}
```

#### 내가 짜는 클래스의 타입을 모르는 경우가 있어 ?
 - 코드를 작성할 시점에는 어떤 타입의 클래스를 사용할지 모르는 경우
 - 런타임에 그 떄 실행되고 있는 클래스의 정보를 가져와서 실행해야 한다 !
 - 예시
    - IDE 의 자동완성 : 사용자가 어떤 클래스를 작성했는지 IDE 를 개발하는 시점에는 알 수 없다.
    - Java Class Loader : 마찬가지로, 동적으로 로딩할 클래스파일의 클래스들이 어떤 것들이 있을지 알 수 없다.
    - @Autowired : 이 에너테이션을 만드는 시점에는 어떤 클래스에 이 어노테이션이 붙을 지 알 수 없었을 것이다.
    
#### 어떻게 가능하나 ?
 - 자바 클래스파일 정보는 컴파일되면 클래스파일로 변환되어 메모리의 Static 영역에 위치하게 된다.
 - static 영역에 있는 정보는 언제든지 접근할 수 있기 떄문에, 클래스의 이름만 알면 클래스정보들을 빼올 수 있다.
 
<br>
<br>
<br>
 
## 정말 간단한 사용법
 - 런타임에 클래스에 접근하여, 클래스의 생성자, 메서드, 필드에 접근할 수 있다.
    - 생성자 : Constructor
    - 메서드 : Method
    - 필드 : Field
 - 위의 각 인스턴스를 사용해, 객체를 생성하거나 메서드를 실행하거나 필드를 조작할 수 있다.
    - 생성자로 인스턴스 생성
    ```java
   1. 클래스 얻기
   Class<?> class = Class.forName();
   
   2. 생성자 얻기
   Constructor<?> constructor = class.getDeclaredConstructor();
   
   3. 클래스 인스턴스 만들기
   constructor.newInstance();    
    ```
   - 메서드 실행
   Method.invoke();
   

<br>
<br>
<br>


## Reflection 의 단점
 1. 컴파일타임의 이점을 누릴 수 없다.
    1. 컴파일타입의 타입 검사.
    1. 컴파일타임의 예외 검사.
 1. 코드가 장황해지고 더러워진다.
 1. 성능이 떨어진다.
 
 확신 없이 리플렉션이 필요한가 아닌가의 고민이 된다면, 필요없을 가능성이 크다.
 
## 리플렉션은 제한된 형태로만 사용해야 이점만 취할 수 있다.
 - 리플렉션은 **컴파일타임에 알 수 없는 객체의 생성** 에만 쓰자.
 - 생성된 객체는 인터페이스로 참조하여 사용하자.
 
 
## 인스턴스 생성 예제
```java
    public static void main(String[] args) {
        Class<? extends Set<String>> cl = null;
        try {
            // 여기서 args[0] 의 인자에 의해 클래스가 결정된다.
            // args[0] 이 무엇인지에 따라 동작이 달라진다.
            cl = (Class<? extends Set<String>>) Class.forName(args[0]);
        } catch (ClassNotFoundException e) {
            fatalError("클래스를 찾을 수 없습니다 !!");
        }

        Constructor<? extends Set<String>> cons = null;
        try {
            cons = cl.getConstructor();
        } catch (NoSuchMethodException e) {
            fatalError("매개변수없는 생성자가 없습니다 !!");
        }

        Set<String> s = null;
        try {
            s = cons.newInstance();
        } catch (IllegalAccessException e) {
            fatalError("생성자에 접근할 수 없습니다.");
        } catch (InstantiationException e) {
            fatalError("클래스를 인스턴스화할 수 없습니다.");
        } catch (InvocationTargetException e) {
            fatalError("생성자가 예외를 던졌습니다." + e.getCause());
        } catch (ClassCastException e) {
            fatalError("Set 을 구현하지 않은 클래스입니다.");
        }

        s.addAll(Arrays.asList(args).subList(1, args.length));
        System.out.println(s);
    }
```

```java
args[0] 가 HashSet 일 때 : [a, b, c, d, e, f, z, j, k, l]
args[0] 가 TreeSet 일 때 : [a, b, c, d, e, f, j, k, l, z]
```
 
 
<br>
<br>

## 위의 코드로 알 수 있는 리플렉션의 단점 두가지
 1. 일반적으로 생성했으면 컴파일타임에 잡을 수 있는 다수의 예외를 런타임에서야 잡을 수 있다.
 1. 일반적으로 생성했으면 생성자 호출 한줄로 끝날 인스턴스 생성을 몇십줄에 걸쳐서야 해냈다.

<br>
<br>
 
## 결론
 - 인터페이스 Set<String> 타입으로 인스턴스를 만들었고
 - 이후 인스턴스 s 를 이용해 기존에 우리가 하던 작업들을 할 수 있따.
 
 <br>
 
 - 물론 계속해서 리플렉션을 이용하여 읽어들인 클래스의 메서드를 수행하거나 할 수 있지만, 리플렉션에는 여러 단점이 존재한다.
 - 리플렉션은 인스턴스 생성에만 사용하고, 생성된 인스턴스는 인터페이스로 형변환하여 사용하자.