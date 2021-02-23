# 아이템 36 - 비트 필드 대신 EnumSet을 사용하라

## 비트 필드가 뭘까?

저는 비트 필드라는 단어부터 처음 들었습니다ㅎㅎ Enum, EnumSet이 나오기 전에 쓰이던 방법이더라고요! 그래서 Enum이 나오기 전에는 어떻게 열거 타입을 표현했는지 알고 가려고 합니다.

자바에서 Enum(열거) 타입을 지원하기 전에는 아래처럼 클래스 안에 상수로 표현해서 사용하곤 했습니다 (정수 열거 패턴이라고도 합니다)

### 정수 열거 패턴

```java
public class Text {
    public static final int BOLD = 0;
    public static final int ITALIC = 1;
    public static final int UNDERLINE = 2;
    public static final int STRIKETHROUGH = 3;
}
```

이 방식은 단점이 매우 많은데요. 자세한 부분은 이펙티브 자바의 아이템 34를 보시면 이해가 더욱 잘 가실겁니다.

어쨌든, Enum이 없던 시절에도 상수들을 집합(Set)처럼 관리해야될 때가 있었습니다. 어떻게 관리했을까요?

여기서 비트 필드가 나옵니다. 아래 코드를 보시죠.

### 비트 필드

```java
public class Text {
    public static final int BOLD = 1 << 0; // 첫 번째 비트 // 1 // 이진수 : 0001
    public static final int ITALIC = 1 << 1; // 두 번째 비트 // 2 // 이진수 : 0010
    public static final int UNDERLINE = 1 << 2; // 세 번째 비트 // 4 // 이진수 : 0100
    public static final int STRIKETHROUGH = 1 << 3; // 네 번째 비트 // 8 이진수 : 1000

    public void applyStyles(int styles) {
        // ...
    }
}
```

클래스 안에 상수로 표현할 때 쉬프트 연산을 사용해 비트로 표현하는 것입니다. 이렇게 사용하면 상수 값들을 비트로 표현할 수 있습니다.

즉, 비트값으로 표현된 Enum들에게 OR 비트 연산자(|)를 넣어서 Enum 값들을 합쳐서 표현할 수 있게 됩니다!

```java
text.applyStyles(BOLD | ITALIC); // 첫 번째와 두 번째 비트 OR 연산으로 3 // 01 | 10 -> 11
```

그런데 왜 비트 필드를 사용하면 안 좋을까요?

## 왜 비트 필드를 사용하면 안 좋을까?

아래 코드를 보시죠.

```java
text.applyStyles(BOLD | ITALIC); // 첫 번째와 두 번째 비트 OR 연산으로 3
```

당연하게도 `text.applyStyles(3)` 메서드에 인자로 들어가는 값은 OR 비트 연산으로 3이 들어가게 됩니다. 

즉, applyStyles() 안에서 3이란 값으로 로직을 실행해야 하는데 3이라는 숫자는 BOLD, ITALIC 이라는 정보를 가지고 있지 않습니다. 해석하기가 어려워지는 거죠.

게다가 Enum 개수가 늘어날 때마다 그만큼 쉬프트 연산하는 비트값도 커지므로 자료형으로 int나 long을 선택할 지 정해야 합니다. 

또 있습니다. 열거값들을 순회하고 싶다면? 다른 로직을 만들어야 합니다.

> 이후, Enum과 EnumSet이 나오게 되면서 전부 Enum 위주로 바뀌게 됩니다.

## 정수 열거 패턴, 비트필드를 완전히 대체할 수 있는 Enum과 EnumSet

### Enum

위에서 작성했던 정수 열거 패턴은 Enum의 등장으로 완전 대체됩니다. 너무나도 쉽게 추가할 수 있죠. 장점도 많습니다. 자세한 부분은 아까처럼 이펙티브 자바 아이템 34를 보시기 바랍니다ㅎㅎ

```java
public enum Text {
    BOLD, ITALIC, UNDERLINE, STRIKETHROUGH ...
}
```

### EnumSet

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

long 하나(elements)로 표현하고 있으며 Enum의 ordinal()이 여기서 쓰이는 걸 볼 수 있습니다

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

## EnumSet vs Set

호기심이 많으신 분이라면 여기서 궁금한 점이 있을 거예요

> "Enum을 집합으로 표현하고 싶다면 그냥 HashSet에 넣어도 똑같은 거 아닌가??"

물론 기능은 같습니다만 성능적으로 큰 차이가 있습니다.

EnumSet의 모든 메서드는 비트 연산을 사용하고 있으며 64개 이하라면 하나의 long 비트만을 사용합니다. 각 계산에 대해 하나의 비트만 검사하는 EnumSet과 해시 코드를 계산해야 하는 HashSet을 비교한다면 당연히 EnumSet이 빠릅니다.

즉, Enum 값을 집합으로 저장할 일이 있다면 가능한 EnumSet을 사용하는 것이 좋습니다.

## 요약

> 정수 열거 패턴, 비트 필드는 Enum, EnumSet이 나와서 쓰이지 않는 방법이다. 쓰지 말자  
> Enum 값을 집합으로 저장할 일이 있다면 비트 연산으로 성능 좋은 EnumSet을 사용하자

## 출처

- 이펙티브 자바 3판
- [https://www.baeldung.com/java-enumset](https://www.baeldung.com/java-enumset)
