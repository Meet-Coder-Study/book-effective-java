# item 31. 한정적 와일드카드를 사용해 API 유연성을 높이라

Java의 제네릭 타입시스템에는 변성 개념이 존재한다. 변성은 무엇인가?

## 변성 `variance`

변성은 `List<String>`과 `List<Object>`와 같이 base 타입이 같고 타입 인자가 다른 여러 타입이 서로 어떤 관계를 갖는지 설명하는 개념이다. 변성에는 다음 3가지 종류가 있다.

1. 공변성 `covariant`

    '공변하다'라고 말할 수 있는 경우는 `A`가 `B`의 하위타입일 때 `List<A>`가 `List<B>`의 하위타입인 경우이다.

2. 반공변성 `contravariant`

    말그대로 공변성의 반대 경우이다. 즉, `B`가 `A`의 하위타입일 때 `List<A>`가 `List<B>`의 하위타입인 경우이다.

3. 무변성 `invariant`

    `A`가 `B`의 타입이지만 `List<A>`와 `List<B>`는 아무 관계가 없는 경우 무변성, 무공변이라고 한다.

    > Effective Java에서는 불공변이라고 소개

    자바의 제네릭은 기본적으로 무변성을 따른다.

즉, Java는 무변성을 따르기 때문에 `List<String>`과 `List<Object>`는 아무런 관계가 없다. `String`이 `Object`의 하위 타입임에도 불구하고 Java는 `List<String>`을 `List<Object>`의 하위타입으로 판단하지 않는다.

사실 `List<String>`은 `List<Object>`가 하는일을 제대로 수행할 수 없다. `List<Object>`는 어떤 객체도 들어갈 수 있는 반면 `List<String>`은 문자열만 받을 수 있기 때문이다. 따라서 하위 타입이 아니라는 것이 말이되는 이야기이다.

## 한정적 와일드카드

```java
package edu.pkch.generic;

import java.util.ArrayList;
import java.util.List;

public class Stack<E> {
    private final List<E> stack = new ArrayList<>();
    private int position = -1;
    
    public void push(E element) {
        position += 1;
        stack.add(element);
    }
    
    public E pop() {
        E popElement = stack.remove(position);
        position -= 1;
        return popElement;
    }
    
    public boolean isEmpty() {
        return position == -1;
    }

		public int stackSize() {
        return position + 1;
    }
}
```

이런 Stack이 있다고 가정한다. 즉, 제네릭 `E` 타입의 값들을 관리하는 stack이다. 다만, 다음과 같이 컬랙션을 받아 stack에 넣는 pushAll 메서드를 추가한다고 가정한다.

```java
public void pushAll(Collection<E> elements) {
    position += elements.size();
    stack.addAll(elements);
}
```

위 메서드는 Java 컴파일러 입장에서는 문제없이 컴파일된다. 다만 런타임에 다음과 같은 코드가 있는 경우 문제가 된다.

```java
Stack<Number> numberStack = new Stack<>();
List<Integer> integers = Arrays.asList(1, 2, 3);
numberStack.pushAll(integers);
```

`Number`는 Java에서 `Integer`, `Double` 등의 상위 타입이다. `numberStack`을 Number 타입으로 선언하고 pushAll에 `Integer` 타입의 `Collection`을 인자로 넣는 경우 다음과 같은 컴파일 에러가 나타난다.

```
Required type: Collection<Number>
Provided: List<Integer>
```

위와 같은 에러는 Java 제네릭이 기본적으로 무공변이기 때문이다. 즉, `Integer`가 실제로는 `Number`의 하위 타입이라도 제네릭에서는 다른 타입으로 받아들여지기 때문이다.

이를 보완하기 위해서 Java는 한정적 와일드카드 `?`를 제공한다.

```java
public void pushAll(Collection<? extends E> elements) {
    position += elements.size();
    stack.addAll(elements);
}
```

즉, 위와 같이 한정적 와일드카드를 활용하여 `Collection<E>`에서 `Collection<? extends E>`로 변경하면 `E` 타입의 하위타입을 Java 컴파일러가 인식할 수 있다. 따라서 위 코드가 정상적으로 동작할 수 있다. 이를 **공변적**이라고 말한다.

```java
@Test
@DisplayName("공변 테스트")
void variance() {
    // given
    Stack<Number> numberStack = new Stack<>();

    // when
    List<Integer> integers = Arrays.asList(1, 2, 3);
    numberStack.pushAll(integers);

    // then
    assertThat(numberStack.stackSize()).isEqualTo(3);
}
```

여기서 `extends`로 한정하는 경우는 제네릭 `E`의 구현 클래스로 한정하는 경우이다. 따라서 `E`가 `Number`인 경우 인자로 `Integer` 같은 구현 클래스가 인자로 허용되는 것이다.

`extends`의 반대로 `super`도 지원한다. `super`는 상위 추상 클래스로 한정하는 경우이다.

```java
public void popAll(Collection<E> dst) {
    dst.addAll(stack);
}
```

`pushAll`과 같이 인자로 받은 `Collection`에 현재 스택의 값들을 모두 넣는 `popAll` 메서드를 구현한다고 가정한다. 이때 한정적 와일드카드를 쓰지 않아도 컴파일 에러가 발생하지 않는다. 다만 사용할 때 문제가 될 수 있다.

```java
Stack<Number> numberStack = new Stack<>();
List<Integer> integers = Arrays.asList(1, 2, 3);
numberStack.pushAll(integers);

List<Object> dst = new ArrayList<>();
numberStack.popAll(dst);
```

다음과 같이 `Number` 타입의 Stack에 popAll을 호출할 때 `Object`를 담는 컬랙션을 전달한다고 가정한다. 이경우는 다음과 같은 컴파일 에러를 알려준다.

```
Required type: Collection<Number>
Provided: List<Object>
```

이렇게 상위 타입을 받아도 되는 경우에는 `super` 키워드를 제공한다.

```java
public void popAll(Collection<? super E> dst) {
    dst.addAll(stack);
}
```

인자를 `Collection<E>`에서 `Collection<? super E>`로 변경한다면 문제없이 동작한다. 왜냐면 이 경우는 `Object`가 `Integer`의 상위타입이기 때문이다. 이를 **반공변적**이라고 말한다.

```java
@Test
@DisplayName("반공변 테스트")
void contraVariance() {
    // given
    Stack<Number> numberStack = new Stack<>();
    List<Integer> integers = Arrays.asList(1, 2, 3);
    numberStack.pushAll(integers);

    // when
    List<Object> dst = new ArrayList<>();
    numberStack.popAll(dst);

    // then
    assertThat(dst.size()).isEqualTo(3);
}
```

예시를 보면 `extends`를 사용하는 경우는 제네릭 타입을 생산하는데 사용하고 있다. 반면에 `super`를 사용하는 경우는 제네릭 타입을 소비하는데 사용한다. 따라서 producer-extends, consumer-super를 기억하면 좀 더 편하게 사용할 수 있다. 이렇게 API의 유연성을 극대화하기 위해서는 한정적 와일드카드를 사용하는 것이 유리하다. 다만 제네릭 타입이 생산자, 소비자로써 둘다 사용된다면 한정적 와일드카드를 사용할 필요가 없다.

## 타입 매개변수와 와일드카드

타입 매개변수와 와일드카드는 공통적인 부분이 있다. 때문에 메서드 정의시에 둘 중 어떤것을 사용해도 좋을 때가 있다.

```java
public static <E> void swap(List<E> list, int i, int j);
public static void swap(List<?> list, int i, int j);
```

위 두 메서드는 둘다 큰 문제는 없다. 때문에 기본적으로 메서드 선언에 타입 매개변수가 한번만 등장한다면 와일드 카드로 대체한다. 즉, 위 두 `swap`에서는 두번째 `swap`이 적절한 것이다.

단, 두번째 `swap`은 다음과 같은 문제점이 있다.

```java
public static void swap(List<?> list, int i, int j) {
	list.set(i, list.set(j, list.get(i)));
}
```

위 코드는 컴파일 에러가 발생한다. 비한정적 와일드카드 `List<?>` 컬랙션은 `null`만 넣을 수 있기 때문이다. 이를 해결하기 위한 가장 좋은 방법은 실제 타입을 알려주는 도우미 메서드를 사용하는 것이다. 이때 도우미 메서드는 제네릭 메서드여야한다.

```java
public static void swap(List<?> list, int i, int j) {
	helper(list, i, j);
}

private static <E> void helper(List<E> list, int i, int j) {
	list.set(i, list.set(j, list.get(i)));
}
```

이때는 도우미 메서드가 타입을 알고 있기 때문에 컴파일에러 없이 동작한다.
