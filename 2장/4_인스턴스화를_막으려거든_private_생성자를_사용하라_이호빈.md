# item 4. 인스턴스화를 막으려거든 private 생성자를 사용하라

## 상황 생각해보기

인스턴스화를 막는다는 건 인스턴스를 생성하지 못하게 한다는 것과 같습니다.

어떤 상황에서 인스턴스를 생성할 필요가 없을까요?

수학의 제곱근을 예시로 생각해보겠습니다. 9의 제곱근은 항상 3이 나옵니다. 

제곱근처럼 **동일한 입력값을 넣으면 항상 동일한 결과를 리턴하는 메서드**를 만든다고 했을 때, **메서드는 어디서든 사용될 수 있으며 있는 그대로 재사용이 가능합니다.**

즉, 이런 상황에서는 인스턴스를 생성하지 않고 전역에서 사용할 수 있으며 항상 재사용이 가능하도록 하는 것이 더 효율적입니다. 우리는 그 상황을 정적(static) 선언으로 구현할 수 있습니다.

**이렇게 항상 전역으로 사용할 수 있는 정적 필드와 정적 메서드가 모인 클래스를** **유틸성 클래스**라고 부릅니다. 그래서 보통 **유틸성 클래스는 필드와 메서드 모두 정적(static) 선언으로 구현**되어 있습니다.

## 정적 필드와 정적 메서드만 있는 게 왜 문제가 될까?

```java
public class CustomStringUtils {

    public static boolean isBlank(String input) {
        return input == null || input.trim().isEmpty();
    }
    
    // ... 다른 static 메서드들
}

```

위 처럼 코드를 작성할 때, 메서드를 사용하는데는 문제가 없습니다.

다만, 그대로 사용하게 되면 **컴파일러가 기본 생성자를 생성하므로 다른 클래스에서 `new CustomStringUtils()` 와 같이 인스턴스를 생성하는 것이 가능합니다.**

**즉, 정적 필드와 정적 메서드밖에 없는데 인스턴스를 생성할 수 있게 됩니다.**

만약 협업하고 있는 상황이라면 다른 개발자에게 기본 생성자를 생성할 수 있는 여지를 주어 혼란을 초래할 수 있습니다. 또한 굳이 인스턴스를 사용하도록 열어둘 이유도 없습니다.

## 인스턴스를 생성하지 못하게 하려면 어떻게 해야할까?

```java
public class CustomStringUtils {
    
    private CustomStringUtils() {
        throw new IllegalStateException("유틸리티 클래스를 인스턴스화할 수 없습니다!");
    }

    public static boolean isBlank(String input) {
        return input == null || input.trim().isEmpty();
    }
    
    // ... 다른 static 메서드들
}

```

**생성자의 접근제한자를 `private` 으로 주면** 다른 클래스에서 생성자를 호출할 수 없으므로 인스턴스를 생성하지 못하게 됩니다.

생성자가 분명히 존재하는데 호출할 수 없는 로직이므로 개발자 입장에서는 헷갈릴 수 있습니다. 이는 주석을 달거나 Exception을 던져서 알기 쉽도록 도움을 줄 수 있습니다.

## 유틸성 클래스의 예시

앞서 말했듯이 모두 `static` 으로 선언되어 있으며, 생성자는 `private` 으로 되어있습니다.

### java.lang.Math

```java
public final class Math {

    /**
     * Don't let anyone instantiate this class.
     */
    private Math() {}

    public static final double E = 2.7182818284590452354;
    public static final double PI = 3.14159265358979323846;
    
    // ...

    public static int max(int a, int b) {
        return (a >= b) ? a : b;
    }
}
```

### java.util.Arrays

```java
public class Arrays {

    private static final int MIN_ARRAY_SORT_GRAN = 1 << 13;

    // Suppresses default constructor, ensuring non-instantiability.
    private Arrays() {}
    
    // ...
    @SafeVarargs
    @SuppressWarnings("varargs")
    public static <T> List<T> asList(T... a) {
        return new ArrayList<>(a);
    }
```

## 고민

### 인스턴스를 생성할 수 없는데 생성자 테스트코드를 작성할 수 있을까?

리플렉션을 이용해서 테스트가 가능합니다. 참고로 테스트 라이브러리는 `Junit 5` 를 사용했습니다.

```java
@DisplayName("유틸리티 클래스 생성 실패 - 예외 처리")
@Test
void constructor() throws Exception {
    Constructor<CustomStringUtils> constructor = CustomStringUtils.class
                                                    .getDeclaredConstructor();
    constructor.setAccessible(true);

    assertThatThrownBy(constructor::newInstance)
        .isInstanceOf(InvocationTargetException.class)
        .getCause()
        .isInstanceOf(IllegalStateException.class);
}
```

`setAccessible(true)` 으로 접근 가능하도록 만든 후에 `newInstance()` 메서드를 호출하면 `private` 생성자도 테스트가 가능합니다. 

**다만, 개인적으로는 해당 테스트코드가 의미가 있는지 잘 모르겠습니다. 테스트 커버리지가 낮다는 이유로 테스트 커버리지를 높이기 위해 테스트코드를 작성하기보다 현재 테스트코드가 의미있는지 생각해보면 좋을 것 같습니다.**

## 요약

> 생성자가 없을 때 기본생성자는 자동으로 만들어진다  
정적 변수와 정적 메서드만 있는 유틸리티 클래스가 다른 곳에서 인스턴스를 생성할 이유가 없다   
다른 곳에서 생성자를 호출하지 못하도록 생성자의 접근제한자를 private으로 만들자

## 출처

- 이펙티브 자바 3판
- [http://kwon37xi.egloos.com/4844149](http://kwon37xi.egloos.com/4844149)
 
