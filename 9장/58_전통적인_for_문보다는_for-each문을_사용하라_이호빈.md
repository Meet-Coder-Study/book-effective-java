# 아이템 58 - 전통적인 for 문보다는 for-each문을 사용하라

반복을 구현할 수 있게 해주는 로직인 for문에 대해서 알아보겠습니다!

for문에는 여러 종류가 있습니다.

## 전통적인 for문

### 컬렉션 순회하기

컬렉션인 경우, Iterator(반복자)를 사용해서 순회할 수도 있고, 인덱스 값으로 순회할 수도 있습니다.

```java
@DisplayName("전통적인 for문, 반복자로 컬렉션 순회")
@Test
void originForByIterator() {
    List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6);
    List<Integer> resultNumbers = new ArrayList<>();

    // 반복자 순회
    for (Iterator<Integer> i = numbers.iterator(); i.hasNext(); ) {
        resultNumbers.add(i.next());
    }

    // 인덱스 값 순회
    for (int i = 0; i < numbers.size(); i++) {
        numbers.get(i);
    }

    assertThat(resultNumbers).containsExactly(1, 2, 3, 4, 5, 6);
}
```

### 배열 순회하기

배열은 인덱스 값으로 순회합니다.

```java
@DisplayName("배열로 컬렉션 순회")
@Test
void originForByArray() {
    List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6);
    int[] resultNumbers = new int[numbers.size()];

    for (int i = 0; i < resultNumbers.length; i++) {
        resultNumbers[i] = numbers.get(i);
    }

    assertThat(resultNumbers).containsExactly(1, 2, 3, 4, 5, 6);
}
```

### 소소한 성능 팁

전통적인 for문 안에서 가운데에는 조건이 들어가게 됩니다. 1번 방법으로 하면 size() 메서드를 매번 호출하는데요. 컬렉션의 size()는 보통 고정된 값이므로 변수로 빼주면 성능상의 이점을 좀 더 가져갈 수 있습니다. 대신 가독성이 떨어질 수 있으므로 사이즈가 크지 않다면 1번 방법도 괜찮다고 생각합니다.

```java
    // (1)
    for (int i = 0; i < numbers.size(); i++) {
        numbers.get(i);
    }

    // (2)
    int numbersSize = numbers.size();
    for (int i = 0; i < numbersSize; i++) {
        numbers.get(i);
    }
```

> 궁금해서 직접 1000만번 돌려봤는데 0.001초 차이가 나네요ㅎㅎ

### 반복자를 사용했을 때 조심해야할 점

반복자를 사용할 때 아래와 같은 실수를 할 수 있습니다. 조심해야 될 부분은 next() 메서드인데요. next 메서드는 두 가지 역할을 갖고 있습니다. 반복자의 커서를 움직임과 동시에 element를 가져옵니다.

예를 들어, 아래 코드와 같이 원소가 6개인 컬렉션을 이중 for문으로 순회합니다.

```java
@DisplayName("iterator의 next() 잘못 사용 시")
@Test
void iteratorNext_fail() {
    List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6);

    int count = 0;

    for (Iterator<Integer> i = numbers.iterator(); i.hasNext(); ) {
        for (Iterator<Integer> j = numbers.iterator(); j.hasNext(); ) {
            i.next();
            j.next();
            count++;
        }
    }

    assertThat(count).isNotEqualTo(36);
    assertThat(count).isEqualTo(6);
}
```

36번 돌거라고 했던 기대와 달리 6번이 돌게 됩니다. 왜냐면 i 반복자의 next 메서드가 2번째 for문에서 돌고있기 때문입니다.

수정하면 아래와 같이 됩니다.

```java
@DisplayName("iterator의 next()를 제대로 사용 시")
@Test
void iteratorNext_success() {
    List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6);
    int count = 0;

    for (Iterator<Integer> i = numbers.iterator(); i.hasNext(); i.next()) {
        for (Iterator<Integer> j = numbers.iterator(); j.hasNext(); j.next()) {
            count++;
        }
    }

    assertThat(count).isEqualTo(36);
}
```

이처럼 반복자를 통해 실수할 여지가 있으며 인덱스 또한 i와 j를 바꿔써도 컴파일 에러는 나지 않기 때문에 실수할 수 있습니다.

이런 실수를 방지해줄 수 있는 것이 향상된 for문입니다.

## for-each(Enhanced for)

Java 5버전부터 나온 향상된 for-each문입니다.

반복자나 인덱스를 사용하는 것에 비해서 가독성도 훨씬 좋고 순서에 따라 처리할 수 있습니다.

```java
@DisplayName("for-each(향상된 For문)으로 컬렉션 순회")
@Test
void enhancedForEachByCollection() {
    List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6);
    List<Integer> resultNumbers = new ArrayList<>();

    for (Integer number : numbers) {
        resultNumbers.add(number);
    }

    assertThat(resultNumbers).containsExactly(1, 2, 3, 4, 5, 6);
}
```

향상된 for문은 내부적으로 Iterable, Iterator 인터페이스를 사용하는데요. 그래서 향상된 for문을 사용하기 위해선 Iterable 인터페이스를 구현한 객체이어야 합니다. (아래처럼 iterator()를 통해 반복자를 가져온 후 동작합니다)

```java
public static void example(List<String> words) {
    Iterator var1 = words.iterator();

    while(var1.hasNext()) {
        String word = (String)var1.next();
        ...
    }
}
```

향상된 for-each문은 내부적으로 메서드를 호출하는 비용이 있어 전통적인 for문보다는 느립니다. 그리고 항상 전체를 순회하기 때문에 중간부터 순회하거나 순회하는 도중에 요소를 삭제하는 일은 불가능합니다.

```java
@DisplayName("향상된 for문에서 remove 사용 시 예외 발생")
@Test
void enhancedForEachRemove() {
    List<Integer> numbers = new ArrayList<>(Arrays.asList(1, 2, 3, 4, 5, 6));

    assertThatThrownBy(() -> {
        // 삭제는 되나 총 원소가 5개가 되고 돌려질 때 예외 발생
        for (Integer number : numbers) {
            if (number.equals(3)) {
                // 컬렉션의 remove
                numbers.remove((Integer) 3);
            }
        }
    }).isInstanceOf(ConcurrentModificationException.class);
}
```

참고로 순회하면서 중간에 요소를 삭제해야할 때는 반복자의 remove() 메서드를 사용해야한다고 하네요. Iterator의 remove 메서드는 아래와 같습니다.

```java
default void remove() {
    throw new UnsupportedOperationException("remove");
}
```

Java 8에 추가된 default 메서드인 Collection의 removeIf()를 보면 반복자를 통해 삭제하고 있습니다.

```java
numbers.removeIf(i -> i.equals(3));
```

```java
default boolean removeIf(Predicate<? super E> filter) {
    Objects.requireNonNull(filter);
    boolean removed = false;
    final Iterator<E> each = iterator();
    while (each.hasNext()) {
        if (filter.test(each.next())) {
            each.remove();
            removed = true;
        }
    }
    return removed;
}
```

## Iterable 인터페이스의 forEach

Java 8에서 Itearable 인터페이스에 추가된 default 메서드인 forEach()입니다. 

```java
@DisplayName("iterable For문으로 컬렉션 순회")
@Test
void iterableForEachByCollection() {
    List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6);
    List<Integer> resultNumbers = new ArrayList<>();

    numbers.forEach(resultNumbers::add);

    assertThat(resultNumbers).containsExactly(1, 2, 3, 4, 5, 6);
}
```

내부를 보시면 향상된 for문을 사용하고 있습니다.

```java
default void forEach(Consumer<? super T> action) {
    Objects.requireNonNull(action);
    for (T t : this) {
        action.accept(t);
    }
}
```

재밌는 건 이렇게 Iterable 인터페이스의 forEach는 내부적으로 향상된 for문을 사용하고 있고 향상된 for문은 내부적으로 반복자(Iterator)를 통해 반복을 구현하고 있습니다.

## Stream 인터페이스의 forEach

이 메서드는 저번 스터디에서 들으셨다시피 연산용도로 사용하면 안됩니다. Stream의 forEach는 최종 연산입니다. toCollect()처럼 깔끔하게 끝내야하는데 여기서 연산을 하는 건 Stream의 의도가 아닙니다. 게다가 연산 용도로 사용한다면 동시성 보장이 어렵고 가독성이 떨어집니다. print용으로 써야합니다.

```java
@DisplayName("stream For문으로 컬렉션 순회")
@Test
void streamForEachByCollection() {
    List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6);
    List<Integer> resultNumbers = new ArrayList<>();

    // 이렇게 쓰지 마세요!
    numbers.stream().forEach(resultNumbers::add);

    assertThat(resultNumbers).containsExactly(1, 2, 3, 4, 5, 6);
}
```

내부구현입니다

```java
void forEach(Consumer<? super T> action);
```

## 요약

> 가독성과 실수를 예방하는 면에서 전통적인 for문보다는 향상된 for-each문이 좋으나 상황에 따라 다릅니다  
각각의 장단점을 알고 상황에 따라 잘 선택해서 사용합시다

## 출처

- 이펙티브 자바 3판
- [https://woowacourse.github.io/javable/post/2020-05-14-foreach-vs-forloop/](https://woowacourse.github.io/javable/post/2020-05-14-foreach-vs-forloop/)
- [https://woowacourse.github.io/javable/post/2020-08-31-java-loop/](https://woowacourse.github.io/javable/post/2020-08-31-java-loop/)
