---
description: 배열보다는 리스트를 사용하라
---

# Item 28

## Intro

- 배열과 제네릭의 관계

## 배열과 제네릭 타입의 두 가지 차이점

> 첫 번째 배열은 공변(covariant)이다.

- 배열
	- 공변(covariant)
	- Sub가 Super의 하위 타입이라면 배열 Sub[]은 배열 Super[]의 하위 타입이 된다.
	- 즉, 함께 변한다.

- 제네릭
	- 불공변(invariant)
	- 서로 다른 타입 Type1과 Type2가 있을 때, List<Type1>은 List<Type2>의 하위 타입도 아니고, 상위 타입도 아니다.

- 위 개념만 봤을 때 제네릭에 문제가 있을수 있다고 생각할 수 있지만 문제가 있는 쪽은 배열이다.

- 문법상 허용되지만 런타임에 실패하는 코드

```java
Object[]objectArray=new Long[1];
        objectArray[0]="타입이 달리 넣을 수 없다."; // ArrayStoreException을 던진다.
```

- 문법에 맞지 않아 컴파일되지 않은 코드

```java
List<Object> ol=new ArrayList<Long>(); // 호환되지 않는 타입이다.
        ol.add("타입이 달라 넣을 수 없다.");
```

> 두 번째 배열은 실체화(reify)된다.

- 배열은 런타임에도 자신이 담기로 한 원소의 타입을 인지하고 확인한다.
- Long 배열에 String을 넣으려 하면 ArrayStoreException이 발생한다.
- 반면, 제네릭은 타입 정보가 런타임에는 소거(erasure)된다.

- 소거
	- 원소 타입을 컴파일타임에만 검사하며, 런타임에는 알 수 조차 없다.
	- 제네릭이 지원되기 전의 레거시 코드와 제네릭 타입을 함께 사용할 수 있게 해주는 메커니즘
	- 자바 5가 제네릭으로 순조롭게 전환될 수 있도록 해줬다. [아이템 26]()

## 배열과 제네릭의 부조화

- 배열은 제네릭 타입, 매개변수화 타입, 타입 매개변수로 사용할 수 없다.
	- 즉, 코드를 new List<E>[], new List<String>[], new E[] 식으로 작성하면 컴파일할 때 제네릭 배열 생성 오류를 일으킨다.

- 제네릭 배열을 만들지 못하게 막은 이유?
	- 타입 안전하지 않기 때문이다.
	- 이를 허용하게 되는 경우 컴파일러가 자동 생성한 형변환 코드에서 런타임에 ClassCastException이 발생할 수 있다.
	- 런타임에 ClassCastException이 발생하는 일을 막아주겠다는 제네릭 타입 시스템의 취지에 어긋난다.

### 제네릭 배열이 가능한 경우를 가정한 예제

```java
class Example {
    public static void main(String[] args) {
        List<String>[] stringLists = new List<String>[1];
        List<Integer> intList = List.of(42);
        Object[] objects = stringLists;
        objects[0] = intList;
        String s = stringLists[0].get(0);
    }
}
```

> 제네릭 배열을 생성이 허용된다고 가정하는 경우

```java
List<String>[]stringLists=new List<String>[1];
```

> 원소가 하나인 List<Integer>를 생성

```java
List<Integer> intList=List.of(42);
```

> List<String>의 배열을 Object 배열에 할당

```java
Object[]objects=stringLists;
```

- 배열은 공변이니 문제가 없다.

> List<Integer>의 인스턴스를 Object 배열의 첫 원소로 저장

```java
objects[0]=intList;
```

- 제네릭은 소거 방식으로 구현되어서 성공한다.
- 즉, 런타임에는 List<Integer> 인스턴스의 타입은 단순히 List가 되고, List<Integer>[] 인스턴스의 타입은 List[]가 된다.
- 따라서 ArrayStoreException을 일으키지 않는다.

> 배열의 처음 리스트에서 첫 원소를 꺼내려하는 경우

- List<String> 인스턴스만 담겠다고 선언한 stringLists 배열에는 지금 List<Integer> 인스턴스가 저장되어 있다.

```java
String s=stringLists[0].get(0);
```

- 컴파일러는 꺼낸 원소를 자동으로 String으로 형변환하는데, 이 원소는 Integer이므로 런타임에 ClassCastException이 발생한다.
- 예외를 방지하기 위해서는 제네릭 배열이 생성되지 않도록 컴파일 오류를 내야 한다.

## 실체화 불가 타입(non-reifiable type)

- E, List<E>, List<String> 같은 타입을 실체화 불가 타입이라 한다.
- 실체화되지 않아서 런타임에는 컴파일타임보다 타입 정보를 적게 가지는 타입이다.
- 소거 매커니즘 때문에 매개변수화 타입 가운데 실체화될 수 있는 타입은 List<?>와 Map<?,?> 같은 비한정적 와일드카드 타입뿐이다. [아이템 26]()
- 배열을 비한정적 와일드 카드 타입으로 만들 수는 있지만, 유용하게 쓰일 일은 거의 없다.

> 배열을 제네릭으로 만들 수 없어 귀찮을 때도 있다.

- 제네릭 컬렉션에서는 자신의 원소 타입을 담은 배열을 반환하는게 보통은 불가능하다. (이 문제에 대한 해결책 [아이템 33]())
- 제네릭 타입과 가변인수 메서드(varargs method, [아이템 53]())를 함께 쓰면 해석하기 어려운 경고 메시지를 받게 된다. - ?
- 가변인수 메서드를 호출할 때마다 가변인수 매개변수를 담을 배열이 하나 만들어지는데, 이때 그 배열의 원소가 실체화 불가 타입이라면 경고가 발생하는 것이다.
	- 이 문제는 @SafeVarargs 어노테이션으로 대처할 수 있다. [아이템 32]()

> 배열로 형변환하려는 경우

- 제네릭 배열 생성오류나 비검사 형변환 경고가 뜨는 경우 배열인 E[] 대신 컬렉션인 List<E>를 사용하면 해결된다.
- 코드가 조금 복잡해지고 성능이 살짝 나빠질 수도 있지만, 그 대신 타입 안전성과 상호운용성은 좋아진다.

### 구체화된 예시

> 생성자에서 컬렉션을 받는 Chooser 클래스

- 컬렉션 안의 원소 중 하나를 무작위로 선택해 반환하는 choose 메서드를 제공한다.
- 생성자에 어떤 컬렉션을 넘기느냐에 따라 이 클래스를 다양한 기능으로 사용할 수 있다.

> 제네릭을 쓰지 않고 구현 예제

- 제네릭이 필요한 로직
- 혹시나 타입이 다른 원소가 들어있었다면 런타임에 형변환 오류가 날 것이다.

```java
import java.util.concurrent.ThreadLocalRandom;

public class Chooser {
    private final Object[] choiceArray;

    public Chooser(Collection choices) {
        choiceArray = choices.toArray();
    }

    public Object choose() { // 해당 메서드 호출 시 반환된 Object를 원하는 타입으로 형변환 필요
        Random rnd = ThreadLocalRandom.current();
        return choiceArray[rnd.nextInt(choiceArray.length)];
    }
}
```

> 제네릭으로 리펙토링

- [아이템 29]() 개념에 따라 제네릭으로 리펙토링
- 아래 클래스 컴파일 시 toArray()의 타입을 Object[] 타입으로 형변환할 수 없어 예외를 발생시킨다.

```java
public class Chooser<T> {
    private final T[] choiceArray;

    public Chooser(Collection<T> choices) {
        choiceArray = choices.toArray();
    }
}
```

- Object 배열을 T배열로 형변환한다.
	- 그러면 경고가 뜬다.
	- T가 무슨 타입인지 알 수 없어 컴파일러는 이 형변환이 런타임에도 안전한지 보장할 수 없다는 메시지이다.
	- 제네릭에서는 원소의 타입 정보가 소거되어 런타임에는 무슨 타입인지 알 수 없다.
	- 해당 코드는 동작하지만 컴파일러가 안전을 보장하진 못한다.
	- 안전하다고 확신하는 경우 주석을 남기고 어노테이션을 달아 경고를 숨겨도 된다.
	- 경고의 원인을 제거하는 편이 가장 좋다. [아이템 27]()

```java
public class Chooser<T> {
    private final T[] choiceArray;

    public Chooser(Collection<T> choices) {
        choiceArray = (T[]) choices.toArray();
    }
}
```

## 비검사 형변환 경고를 제거하기 위해 배열 대신 리스트 사용

- 리스트 기반 Chooser를 통한 타입 안전성 확보
	- 성능 상으로 조금 느려짐
	- 런타임에 ClassCastException()을 만날 일은 없다.

```java
public class Chooser<T> {
    private final List<T> choiceList;

    public Chooser(Collection<T> choices) {
        choiceList = new ArrayList<>(choices);
    }

    public T choose() {
        Random rnd = ThreadLocalRandom.current();
        return choiceList.get(rnd.nextInt(choiceList.size));
    }
}
```

## 핵심 정리

- 배열과 제네릭에는 매우 다른 타입 규칙이 적용된다.
- 배열은 공변이고 실체화된다.
- 제네릭은 불공변이고 타입정보가 소거된다.
- 결과적으로
	- 배열은 런타임에는 타입 안전하지만 컴파일 타임에는 그렇지 않다.
	- 제네릭은 런타임에는 타입 안전하지 않고 컴파일 타임에는 안전하다.
- 둘 을 섞어서 쓰기란 쉽지 않기 때문에 컴파일 오류나 경고를 만나면 가장 먼저 배열을 리스트로 대처ㄹ하는 방법을 적용하도록 한다.
