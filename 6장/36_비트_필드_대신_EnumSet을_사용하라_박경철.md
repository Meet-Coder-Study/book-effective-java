# item 36. 비트 필드 대신 EnumSet을 사용하라

열거 값들이 집합으로 사용되는 경우 비트 마스킹을 활용한 정수 열거 패턴을 사용하곤 했다.

```java
public class Text {
    public static final int BOLD = 1 << 0;
    public static final int ITALIC = 1 << 1;
    public static final int UNDERLINE = 1 << 2;
    public static final int STRIKETHROUGH = 1 << 3;

    public void applyStyles(int styles) {
        // ...
    }
}
```

굵은체 `BOLD`는 첫번째 비트, 기울임체 `ITALIC`는 두번째 비트, 밑줄 `UNDERLINE`은 세번째 비트, 취소선 `STRIKETHROUGH`는 네번째 비트로 구분하는 것이다. styles는 이들 비트를 조합한 int 값이 매개변수로 들어간다.

```java
text.applyStyles(BOLD | UNDERLINE); // BOLD | UNDERLINE은 3
text.applyStyles(3);
```

위와 같이 비트의 OR연산을 통해 여러 상수를 하나의 집합으로 모을 수 있다. 이렇게 만들어진 집합을 비트 필드라고 한다.

단, 비트 필드는 정수 열거 상수의 단점을 그대로 지닌다. 비트 필드 값이 그대로 출력이 되면 이를 해석하기 너무 어렵다.

위 예시에서도 `BOLD`와 `UNDERLINE`이 적용된 값이 3인데 3만 보고서는 이를 파악하기 매우 힘들다. 비트 필드 하나에 녹아있는 모든 원소 순회도 까다롭다. 마지막으로 필요한 최대 비트를 API 작성시 예측하여 `int`나 `long` 같은 적절한 타입을 선택해야한다.

이에 대한 완벽한 대안이 바로 `EnumSet`이다. enum 상수 값으로 구성된 집합을 효과적으로 표현하며 `Set` 인터페이스를 완벽히 구현할 수 있다.

`EnumSet`의 내부는 사실 비트 백터로 구성된다. 원소가 64개 이하라면 대부분의 경우 `long` 변수 하나로 표현하여 비트 필드와 비슷한 성능을 보여준다.

> 참고로 원소가 64개 이하라면 RegularEnumSet, 65개 이상이면 JumboEnumSet을 사용

```java
public static <E extends Enum<E>> EnumSet<E> noneOf(Class<E> elementType) {
    Enum<?>[] universe = getUniverse(elementType);
    if (universe == null)
        throw new ClassCastException(elementType + " not an enum");

    if (universe.length <= 64)
        return new RegularEnumSet<>(elementType, universe);
    else
        return new JumboEnumSet<>(elementType, universe);
}
```

RegularEnumSet의 내부를 보면 다음과 같다.

```java
class RegularEnumSet<E extends Enum<E>> extends EnumSet<E> {
    private static final long serialVersionUID = 3411599620347842686L;

    private long elements = 0L;

    void addAll() {
        if (universe.length != 0)
            elements = -1L >>> -universe.length;
    }

    void complement() {
        if (universe.length != 0) {
            elements = ~elements;
            elements &= -1L >>> -universe.length;  // Mask unused bits
        }
    }

    // ...
}
```

> universe는 추상클래스 EnumSet에 정의되어있음. 비트마스크의 역할

원소가 64개 이하에서 사용하는 `RegularEnumSet`은 64비트로 표현하는 `long` 타입의 `elements`를 통해 Set을 구현한다.

```java
public boolean contains(Object e) {
    if (e == null)
        return false;
    Class<?> eClass = e.getClass();
    if (eClass != elementType && eClass.getSuperclass() != elementType)
        return false;

    return (elements & (1L << ((Enum<?>)e).ordinal())) != 0;
}

public boolean add(E e) {
    typeCheck(e);

    long oldElements = elements;
    elements |= (1L << ((Enum<?>)e).ordinal());
    return elements != oldElements;
}

public boolean remove(Object e) {
    if (e == null)
        return false;
    Class<?> eClass = e.getClass();
    if (eClass != elementType && eClass.getSuperclass() != elementType)
        return false;

    long oldElements = elements;
    elements &= ~(1L << ((Enum<?>)e).ordinal());
    return elements != oldElements;
}
```

이때 `ordinal`을 통해 해당 Enum 인스턴스의 위치에 해당하는 비트를 증가/감소하는 방식으로 add/remove가 이뤄지고 해당 Enum 인스턴스의 위치에 해당하는 비트가 0인지 1인지로 `EnumSet`에 존재하는지 확인할 수 있다.

`addAll`과 `removeAll`도 마찬가지로 비트 산술 연산을 통해 최적화한다.

```java
public boolean addAll(Collection<? extends E> c) {
    if (!(c instanceof RegularEnumSet))
        return super.addAll(c);

    RegularEnumSet<?> es = (RegularEnumSet<?>)c;
    if (es.elementType != elementType) {
        if (es.isEmpty())
            return false;
        else
            throw new ClassCastException(
                es.elementType + " != " + elementType);
    }

    long oldElements = elements;
    elements |= es.elements;
    return elements != oldElements;
}

public boolean removeAll(Collection<?> c) {
    if (!(c instanceof RegularEnumSet))
        return super.removeAll(c);

    RegularEnumSet<?> es = (RegularEnumSet<?>)c;
    if (es.elementType != elementType)
        return false;

    long oldElements = elements;
    elements &= ~es.elements;
    return elements != oldElements;
}
```

> 참고로 인자로 들어오는 Collection이 RegularEnumSet이 아니라면 AbstractSet의 addAll이나 removeAll을 호출한다. 하지만 여기서도 결국 Set에 들어있는 원소들을 순회하면서 하나씩 add / remove하므로 똑같이 산술 연산으로 처리함을 알 수 있다.

Enum과 EnumSet을 활용해서 위 정수 열거 패턴의 Text를 다음과 같이 변경할 수 있다.

```java
public class Text {
    public enum Style {
        BOLD, ITALIC, UNDERLINE, STRIKETHOUGH
    }

    private Set<Style> styles = new EnumSet.noneOf(Style.class);

    public void applyStyles(Set<Style> styles) {
        this.styles.addAll(styles);
    }
}
```

다음과 같이 Set을 받는 매개변수 `styles`를 Text 객체의 styles에 addAll하면 정수 열거 패턴의 `applyStyles`와 동일하게 사용할 수 있다.

참고로 enum의 갯수가 65개 이상인 경우 사용하는 JumboEnumSet은 long 배열을 활용한다.

```java
class JumboEnumSet<E extends Enum<E>> extends EnumSet<E> {
    private static final long serialVersionUID = 334349849919042784L;

    private long elements[];

    private int size = 0;

    JumboEnumSet(Class<E>elementType, Enum<?>[] universe) {
        super(elementType, universe);
        elements = new long[(universe.length + 63) >>> 6];
    }

    // ...
}
```

> 대부분의 enum의 원소 갯수가 64개를 넘어갈일이 없다고 개인적으로 생각하므로 `RegularEnumSet`만 어느정도 알면 되지 않을까 생각합니다.
