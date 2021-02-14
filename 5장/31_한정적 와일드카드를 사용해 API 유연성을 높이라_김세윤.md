# 아이템31. 한정적 와일드카드를 사용해 API 유연성을 높이라.

## 공변(covariant), 불공변(invariant)

- 공변과 불공변은 변함에 차이가 있다. 즉, 공변은 `함께 변하는 것`이고 불공변은 `함께 변하지 못한다`는 의미이다.

```java
class List<T> {...}
```

- 이떄 List가 공변이면
    - List<String>은 List<Object>의 하위 타입
- 불공변은
    - 위의 하위 타입이 성립하지 않음
- 그러나 이제 배울 매개변수화 타입(제네릭)은 List<String>이 List<Object>의 역할을 제대로 수행하지 못하기 때문에 리소코프 치환 원칙에 어긋나 불공변이라고 한다.
- 즉 아래와 같이 사용하지 못한다는 것이다.

```java
ArrayList<String> strings = new ArrayList<Object>();
ArrayList<Object> objects = new ArrayList<String>();
```

- 그럼 공변은 뭐가 잇을까? 바로 배열이다.

```java
Object[] objects = new String[1];
```

### 제네릭에서 불공변의 문제점

```java
package com.spring.test;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.EmptyStackException;
import java.util.List;

public class Stack<E> {
    private List<E> elements;
    private int size = 0;

    public Stack() {
        this.elements = new ArrayList<>();
    }

    public static void main(String[] args) {
        Stack<Number> stack = new Stack<>();
        Iterable<Integer> integers = Arrays.asList(1);
        stack.pushAll(integers);
    }

    public void push(E o) {
        elements.add(o);
        size++;
    }

    public <E> E pop() {
        if (size == 0) {
            throw new EmptyStackException();
        }
        E element = (E)elements.get(size);
        elements.remove(size--);
        return element;
    }

    public void pushAll(Iterable<E> src) {
        for (E e : src) {
            push(e);
        }
    }

    public boolean isEmpty() {
        return elements.size() == 0;
    }
}
```

```java
error: incompatible types: Iterable<Integer> cannot be converted to Iterable<Number>
        stack.pushAll(integers);
                      ^
```

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d84eace4-e18c-4b72-bdd0-edd1663e4882/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d84eace4-e18c-4b72-bdd0-edd1663e4882/Untitled.png)

- 하위 타입이여서 논리적으로 잘 동작할 것 같지만 불공변이기 때문에 아래와 같이 에러가 발생한다.
- 따라서 자바는 한정적 와일드카드 타입을 지원하게 된다.

## 한정적 와일드 카드 타입을 사용할 때

- 원소의 생산자나 소비자용 입력 매개변수 일 때
- 원소가 생산자와 소비자 역할을 동시에 사용하지 말 것, 이는 타입을 정확히 지정해야하는 상황을 뜻한다.
- 즉, 유연성을 극대화 하기 위해서다.

## 생산자(producer) & 소비자(Consumer)

- 생산자란 입력 매개변수로부터 이 컬렉션으로 원소를 옮겨 담는다는 뜻

```java
public void pushAll(Iterable<? extends E> src) {
	for (E e : src) {
		push(e);
	}
}
```

- 소비자란 컬렉션 인스턴스의 원소를 입력 매개변수로 옮겨 담는다는 뜻

```java
public void popAll(Collection<? super E> dst) {
	while (!isEmpty()) {
		dst.add(pop());
	}
}
```

## PECS(Producer-Extends, Consumer-Super)

- 매개변수화 타입 T가 생산자라면 <? extends T>를 사용하고, 소비자라면 <? super T>를 사용하라.

### 주의 점

- 반환 타입에는 한정적 와일드카드 타입을 사용하면 안된다. 사용하는 순간 클라이언트 코드에서도 와일드 카드 타입을 신경써야 하기 때문에 API에 문제가 있는 것이다.

## Comparable & Comparator

- Comparable<E>보다는 Comparable<? super E>를 사용하라
- Comparator<E>보다는 Comparator<? super E>를 사용하라

## 타입 매개변수와 와일드 카드

```java
public static <E> void swap(List<E> list, int i, int j);
public static void swap(List<?> list, int i, int j);
```

- 둘중에 뭐가 더 좋을까?
- 기본 규칙을 생각해보자.
- 메서드 선언에 타입 매개변수가 한 번만 나오면 와일드 카드로 대체하라.
- 이때 비한정적 타입 매개변수라면 비한정적 와일드카드로, 한정적 타입 매개변수라면 한정적 와일드카드를 사용하라.
- 따라서 2번째가 더 좋은 코드이다.

### bounded Wlidcard vs unbounded Wlidcard

- 한정적 와일드카드
    - 특정 타입을 제한한다는 뜻으로 `<T extends Number>`와 같은 것이다.
- 비한장적 와일드카드
    - `?` 와 같은 타입으로 어떤 타입이 오던 관계가 없다는 뜻이다.

### 2번째의 메서드의 문제점

- List<?>에는 null 외에는 어떤 값도 넣을수가 없다.
- 따라서 private 도우미 메서드를 선언해줘야 한다.

```java
public static <E> void swapHelper(List<E> list, int i, int j) {
    list.set(i, list.set(j, list.get(i))); 
}
```

## 결론

- PECS(Producer-Extends, Consumer-Super) 공식을 기억하자
- 반환 타입에는 한정적 와일드카드 타입을 사용하면 안된다. 사용하는 순간 클라이언트 코드에서도 와일드 카드 타입을 신경써야 하기 때문에 API에 문제가 있는 것이다.
- Comparable<E>보다는 Comparable<? super E>를 사용하라
- Comparator<E>보다는 Comparator<? super E>를 사용하라
- 메서드 선언에 타입 매개변수가 한 번만 나오면 와일드 카드로 대체하라.
