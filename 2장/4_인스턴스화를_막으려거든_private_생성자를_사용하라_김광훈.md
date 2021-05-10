# item 4. 인스턴스화를 막으려거든 private 생성자를 사용하라
## Why ??
- 정적 멤버만 담은 유틸리티 클래스는 인스턴스로 만들어 사용하려고 설계한 것이 아니다.
- 생성자를 명시하지 않으면 컴파일러가 자동으로 기본 생성자를 만들어준다.

## 주의 사항
- 추상 클레스로 만드는 것으로는 인스턴스화를 막을 수 없다.
- 하위 클래스를 만들어 인스턴스화하면 그만이다.
- 다른 개발자들이 코드를 보았을 때 상속해서 쓰라는 뜻으로 오해할 수도 있다.

## How ??
- 모든 생성자는 명시적이든 묵시적이든 상위 클래스의 생성자를 호출하게 된다.
- 따라서 private으로 선언하여 생성자 생성을 막고, 또한 하위 클래스가 상위 클래스의 생성자에 접근 또한 막을 수 있다.

## Example
- Effective Java 책에서는 Math 유틸클래스를 예시로 언급하였다.
```java
public final class Math {

    private Math() { } // private constructor

    public static final double E = 2.7182818284590452354;

    public static final double PI = 3.14159265358979323846;

    private static final double DEGREES_TO_RADIANS = 0.017453292519943295;

    private static final double RADIANS_TO_DEGREES = 57.29577951308232;

    @HotSpotIntrinsicCandidate
    public static double sin(double a) {
        return StrictMath.sin(a); // default impl. delegates to StrictMath
    }
    
    public static double asin(double a) {
        return StrictMath.asin(a); // default impl. delegates to StrictMath
    }
}
```
- 추상클래스 예제
```java
/* 추상 클래스 */
public class AbstractPrivateConstructorTest {
    
}

/* 하위 클래스 */
public class PrivateConstructorTest extends AbstractPrivateConstructorTest {
    public PrivateConstructorTest() { }
}

/* 테스트 */
@Test
void 추상클래스_Private_생성자_테스트() {
    PrivateConstructorTest privateConstructorTest = new PrivateConstructorTest();

    assertThat(privateConstructorTest).isInstanceOf(AbstractPrivateConstructorTest.class);
}

```

## 추가 고민: 테스트 코드를 작성할 수 있을까 ??
- Java Reflection 기능을 사용하면 가능하다.
- 하지만 단순 테스트 커버리지를 높히기 위한 테스트이기 때문에 의미가 없다고 생각한다.
- 참고 블로그: https://atin.tistory.com/630
#### 예제 클래스
```java
public class PrivateConstructorTest {

    private PrivateConstructorTest() {
    //   throw new IllegalStateException();
    }
}
```

#### 예제 테스트
````java
class PrivateConstructorTestTest {

    @Test
    void Private_생성자_클래스_인스턴스_생성_Test() throws NoSuchMethodException, InvocationTargetException, InstantiationException, IllegalAccessException {
        // given
        Constructor<PrivateConstructorTest> constructor = PrivateConstructorTest.class
                .getDeclaredConstructor();
        
        // when
        constructor.setAccessible(true);
        PrivateConstructorTest privateConstructorTest =  constructor.newInstance();

        // then
        assertThat(privateConstructorTest).isInstanceOf(PrivateConstructorTest.class);
    }
}
````