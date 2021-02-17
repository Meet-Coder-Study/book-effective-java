# 아이템 36 - 비트 필드 대신 EnumSet을 사용하라

## 비트 필드가 뭘까?

예시를 들면서 이해해보겠습니다.

```java
public enum Text {
    BOLD, ITALIC, UNDERLINE, STRIKETHROUGH ...
}
```

여러 개의 Enum(열거) 값이 있습니다. 여기서 BOLD, ITALIC의 값만 따로 사용하고 싶다면 어떻게 해야할까요? 매개변수로 하나하나 넣어줘야겠죠? 이런 방식으로 한다면 매번 함수를 만들어야 합니다.

그래서 예전에는 정수 열거 패턴을 사용했다고 합니다. 아래처럼 말이죠

```java
public class Text {
    public static final int BOLD = 1 << 0; // 첫 번째 비트 // 1 // 이진수 : 0001
    public static final int ITALIC = 1 << 1; // 두 번째 비트 // 2 // 이진수 : 0010
    public static final int UNDERLINE = 1 << 2; // 세 번째 비트 // 4 // 이진수 : 0100
    public static final int STRIKETHROUGH = 1 << 3; // 네 번째 비트 // 8 이진수 : 1000

    public void applyStyles(int styles) {
        // ...
    }

    public static void main(String[] args) {
        // ...
        text.applyStyles(BOLD | ITALIC);
    }
} 
```

즉, 비트값으로 표현된 Enum들에게 OR 비트 연산자(`|`)를 넣어서 Enum 값들을 합쳐서 표현할 수 있습니다.

```java
text.applyStyles(BOLD | ITALIC); // 첫 번째와 두 번째 OR 비트 연산으로 3
```

이렇게 OR 비트 연산자를 통해 만들어진 집합을 비트 필드라고 부릅니다.

## 왜 비트 필드를 사용하면 안 좋을까?

위 예제를 그대로 사용해보겠습니다.

```java
text.applyStyles(BOLD | ITALIC); // 첫 번째와 두 번째 비트 OR 비트 연산으로 3 // 01 | 10 -> 11
```

두 상수 OR 비트 연산의 값으로 3이 나오게 됩니다. 즉, applyStyles() 안에서 3이란 값으로 로직을 실행해야 하는데 3이라는 숫자가 BOLD, ITALIC 이라는 정보를 가지고 있나요? 가지고 있지 않습니다. 해석하기가 어려워지는 거죠.

게다가 Enum 개수가 늘어날 때마다 그만큼 비트값도 커지므로 자료형으로 int나 long을 선택할 지 정해야 합니다. 

또 있습니다. 열거값들을 순회하고 싶다면? 다른 로직을 만들어야 합니다.

단점이 정말 많은데 이런 비트필드를 완전히 대체할 수 있는 API가 바로 EnumSet입니다.

## EnumSet을 사용하면 왜 좋을까?

EnumSet은 Set 인터페이스를 완벽히 구현하며, 타입 안전하고, 다른 Set 구현체와도 함께 사용할 수 있습니다.

EnumSet의 내부는 비트 벡터로 구현되어 있으며 원소의 개수가 64개 이하라면 long 변수 하나로 표현할 수 있어서 비트필드에 비견되는 성능을 보여줍니다.

정보도 그대로 가지고 있고 비트필드를 사용하면 할 수 없었던 순회도 할 수 있습니다.

```java
public class Text {

    public enum Style { BOLD, ITALIC, UNDERLINE, STRIKETHROUGH }

    public static void applyStyles(Set<Style> styles) {
        //...
    }

    public static void main(String[] args) {
        // ...
        text.applyStyles(EnumSet.of(Style.BOLD, Style.ITALIC, Style.UNDERLINE));
    }
}
```

## EnumSet 내부 살펴보기

### 사용한 of() 메서드

```java
public static <E extends Enum<E>> EnumSet<E> of(E first, E... rest) {
    EnumSet<E> result = noneOf(first.getDeclaringClass());
    result.add(first);
    for (E e : rest)
        result.add(e);
    return result;
}
```

### noneOf() 메서드

여기서 64개 이하면 RegularEnumSet, 이상이면 JumboEnumSet으로 가게 되네요

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

### RegularEnumSet

long 하나(elements)로 표현하고 있으며 ordinal()이 여기서 쓰이는 걸 볼 수 있습니다

```java
class RegularEnumSet<E extends Enum<E>> extends EnumSet<E> {
    private static final long serialVersionUID = 3411599620347842686L;
    
    private long elements = 0L;

    RegularEnumSet(Class<E>elementType, Enum<?>[] universe) {
        super(elementType, universe);
    }

    void addRange(E from, E to) {
        elements = (-1L >>>  (from.ordinal() - to.ordinal() - 1)) << from.ordinal();
    }

    //...
}
```

이렇게 설명했지만 저는 EnumSet을 사용해본 적이 없습니다. 실무에서도 많이 쓰이는 지 궁금하네요ㅎㅎ

## 요약

> Enum 값을 묶을 일이 있다면 EnumSet을 사용하자
