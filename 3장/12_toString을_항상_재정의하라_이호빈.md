# 아이템 12 - toString을 항상 재정의하라

## toString()이 뭘까요?

`toString()`이란 `Object` 클래스의 메서드입니다. 자바에 모든 객체는 [Object 클래스](https://docs.oracle.com/javase/8/docs/api/java/lang/Object.html)를 상속하기 때문에 모든 객체는 `toString()` 메서드를 가지고 있습니다.

`toString()`은 아래와 같은 방식으로 객체를 표현하는 문자열을 리턴해줍니다.

```java
public String toString() {
    return getClass().getName() + "@" + Integer.toHexString(hashCode());
}
```

위 코드처럼 `클래스_이름@16진수로_표시한_해시코드` 를 반환합니다.

## toString()을 언제 사용할까요?

`toString()`은 기본적으로 객체를 표현하는 문자열을 리턴해주기 때문에 개발할 때 사용됩니다.

1. 콘솔로 객체를 확인할 때
    1. `System.out.println()`
    2. `System.out.print()`
    3. 객체에 문자열을 연결할 때(`SomeObject + ""`)
    4. assert 구문 사용 시
    5. 등등...
2. 디버깅 할 때
3. 로깅할 때

### `System.out.println()` 메서드 파고들기

```java
// System.out.println()
public void println(Object x) {
    String s = String.valueOf(x);
    synchronized (this) {
        print(s);
        newLine();
    }
}

// String.valueOf()
public static String valueOf(Object obj) {
    return (obj == null) ? "null" : obj.toString();
}

// Object.toString()
public String toString() {
    return getClass().getName() + "@" + Integer.toHexString(hashCode());
}
```

`toString()`을 사용한 테스트코드 작성은 [여기](https://github.com/aegis1920/my-lab/blob/master/effective-java/src/test/java/com/bingbong/effectivejava/item12/ToStringTest.java)에서 보실 수 있습니다.

## toString()을 왜 항상 재정의해야할까요?

toString()이 개발할 때 사용된다는 것을 알았습니다. 그렇다면 왜 항상 재정의해야할까요?

사용하는 예시로 콘솔로 확인하거나, 디버깅 하거나, 로깅할 때를 들었습니다. 예시로 든 모든 상황은 사용하는 주체가 개발자입니다.

즉, **개발자에게 객체가 보여지는데 `클래스_이름@16진수로_표시한_해시코드` 와 같은 방식으로 보여진다면 해당 객체 안에 뭐가 들었는지 알 수 없습니다.**

**사용하려는 객체에 toString()을 재정의하면 로깅 및 디버깅 시 개발자에게 좀 더 유익한 정보를 전해줄 수 있습니다.**

## toString()을 어떻게 재정의해야할까요?

재정의할 때 가장 중요한 건 **객체 스스로를 완벽히 설명하는 문자열**이어야 한다는 것입니다.

보통은 **객체가 가진 주요 정보를 모두 반환**하는 것이 좋습니다.

전화번호처럼 포맷이 정해져 있는 경우, 아래 코드와 같이 재정의하고 주석을 달아서 문서화를 해줄 수도 있습니다.

```java
/** 
 * 전화번호의 문자열 표현을 반환합니다.
 * 이 문자열은 XXX-YYYY-ZZZZ 형태의 11글자로 구성됩니다.
 * XXX는 지역코드, YYYY는 접두사, ZZZZ는 가입자 번호입니다.
 * 블라블라~
*/
@Override
public String toString() {
    return String.format("%03d-%04d-%04d", areaCode, prefix, lineNum);
}
```

다만 포맷을 정해준다면 앞으로의 유지보수에도 항상 이 포맷을 써야하기 때문에 신중히 포맷을 정할 필요가 있습니다.

보통은 포맷 여부와 상관없이 아래와 같은 방식으로 `toString()`을 재정의합니다.

```java
@Override
public String toString() {
    return "PhoneNumber{" +
                "areaCode='" + areaCode + '\'' +
                ", prefix='" + prefix + '\'' +
                ", lineNum='" + lineNum + '\'' +
                '}';
}
```

유틸리티 클래스는 `toString()`을 사용할 이유가 없고, `Enum` 은 아래 코드처럼 `toString()` 이미 정의되어 있으므로 `toString()`을 재정의하지 않아도 됩니다.

```java
public abstract class Enum<E extends Enum<E>> implements Comparable<E>, Serializable {

    private final String name;

    public String toString() {
        return name;
    }
}
```

개인적으로는 `toString()`을 항상 정의하기보다는 필드를 표현할 일이 있는 `Entity`나 `VO`, `DTO` 같은 성격의 객체에 `toString()`을 해놓으면 디버깅할 때 편한 것 같습니다.

## toString()의 예시

### BigInteger의 toString()

```java
public String toString(int radix) {
    if (signum == 0)
        return "0";
    if (radix < Character.MIN_RADIX || radix > Character.MAX_RADIX)
        radix = 10;

    // If it's small enough, use smallToString.
    if (mag.length <= SCHOENHAGE_BASE_CONVERSION_THRESHOLD)
       return smallToString(radix);

    // Otherwise use recursive toString, which requires positive arguments.
    // The results will be concatenated into this StringBuilder
    StringBuilder sb = new StringBuilder();
    if (signum < 0) {
        toString(this.negate(), sb, radix, 0);
        sb.insert(0, '-');
    }
    else
        toString(this, sb, radix, 0);

    return sb.toString();
}
```

### AbstractMap<K,V>의 toString() → HashMap의 상위 추상 클래스

```java
public String toString() {
    Iterator<Entry<K,V>> i = entrySet().iterator();
    if (! i.hasNext())
        return "{}";

    StringBuilder sb = new StringBuilder();
    sb.append('{');
    for (;;) {
        Entry<K,V> e = i.next();
        K key = e.getKey();
        V value = e.getValue();
        sb.append(key   == this ? "(this Map)" : key);
        sb.append('=');
        sb.append(value == this ? "(this Map)" : value);
        if (! i.hasNext())
            return sb.append('}').toString();
        sb.append(',').append(' ');
    }
}
```

### AbstractCollection<E>의 toString() → ArrayList의 상위 추상 클래스

```java
public String toString() {
    Iterator<E> it = iterator();
    if (! it.hasNext())
        return "[]";

    StringBuilder sb = new StringBuilder();
    sb.append('[');
    for (;;) {
        E e = it.next();
        sb.append(e == this ? "(this Collection)" : e);
        if (! it.hasNext())
            return sb.append(']').toString();
        sb.append(',').append(' ');
    }
}
```

## 고민 및 팁

### System.out.println()을 어떻게 테스트하나?

`ByteArrayOutputStream` 객체를 생성해준 후, `OutputStream`을 `set`하면 됩니다.

`@AfterEach`를 통해서 테스트가 끝나면 원래대로 되돌려줍시다.

```java
class ToStringTest {
    // Object 클래스의 toString 메서드를 호출했을 때
    private static final Pattern OBJECT_PATTERN = Pattern.compile("(.+)PhoneNumberByDefault@(.+)");
    
    private static final ByteArrayOutputStream outContent = new ByteArrayOutputStream();
    
    @BeforeEach
    void setUp() {
        System.setOut(new PrintStream(outContent));
    }
    
    @AfterEach
    void tearDown() {
        System.setOut(System.out);
    }

    @DisplayName("System.out.println 메서드에서 객체 호출")
    @Test
    void callToString_Println() {
        PhoneNumberByDefault phoneNumber = new PhoneNumberByDefault("010", "1234", "1234");
        System.out.println(phoneNumber);

        assertThat(OBJECT_PATTERN.matcher(outContent.toString()).find()).isTrue();
    }
}
```

### toString() 재정의 시 순환 참조를 조심하자

IDE에서 지원해주는 `toString()` 혹은 Lombok에 있는 `@ToString` 을 무분별하게 사용하다가는 `StackOverflowError` 가 일어날 수 있습니다.

아래 코드처럼 A에서는 b를 출력하려고 하고, B에서는 a를 출력하려고 해서 무한 호출로  `StackOverflowError` 가 일어납니다.

서로가 서로를 참조하는 필드를 `toString()` 에서 빼서 해결해줄 수 있습니다.

```java
class A {
    private B b;

    @Override
    public String toString() {
        return "A{" + "b='" + b + "}";
    }
}

class B {
    private A a;

    @Override
    public String toString() {
        return "B{" + "a='" + a + "}";
    }
}
```

## 요약

> Object 클래스의 toString()을 그대로 사용하면 클래스이름과 해시코드가 있는 문자열이 나온다  
해당 문자열은 개발자가 알아보기 힘드니 사용하려는 객체의 toString()을 재정의해서 알아보기 쉽게 만들자  
toString()을 재정의할 때는 '객체 스스로를 표현하고 있나'를 생각하고 순환 참조를 조심하자!

## 출처

- 이펙티브 자바 3판
