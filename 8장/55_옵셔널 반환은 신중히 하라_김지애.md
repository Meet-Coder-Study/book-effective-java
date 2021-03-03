# [Effective Java] item 55. 옵셔널 반환은 신중히 하라

### 메서드가 특정 조건에서 값을 반환할 수 없을 때
#### 자바 8 이전의 방식의 단점
1. 예외를 던진다.
    - 예외는 진짜 예외적인 상황에서만 사용해야 한다.
    - 예외를 생성할 때 스택 추적 전체를 캡쳐하므로 비용도 만만치 않다.
2. null을 반환한다. 
    - null을 반환할 수 있는 메서드를 호출할 때는, (null이 반환될 일이 절대 없다고 확신하지 않는 한) 별도의 null 처리 코드를 추가해야 한다.


#### 자바 8부터 생긴 `Optional<T>`
- `Optional<T>`는 null이 아닌 T 타입 참조를 하나 담거나, 혹은 아무것도 담지 않을 수 있다.
- 아무것도 담지 않은 옵셔널은 '비었다'고 말한다. 반대로, 어떤 값을 담은 옵셔널은 '비지 않았다'고 한다.
- `옵셔널은 원소를 최대 1개 가질 수 있는 '불변' 컬렉션`이다.
- `보통은 T를 반환하지만, 특정 조건에서는 아무것도 반환하지 않아야 할 때 T 대신 Optional<T>를 반환`하도록 선언하면 된다. 그러면 유효한 반환 값이 없을 때는 빈 결과를 반환하는 메서드가 만들어진다.
- 옵셔널을 반환하는 메서드는 예외를 던지는 메서드보다 유연하고 사용하기 쉬우며, null을 반환하는 메서드보다 오류 가능성이 적다.

##### 컬렉션에서 최댓값을 구한다(컬렉션이 비어있으면 예외를 던진다)
```java
public static <E extends Comparable<E>> E max(Collection<E> c) {
    if (c.isEmpty())
        throw new IllegalArgumentException("Empty collection");

    E result = null;
    for (E e : c)
        if (result == null || e.compareTo(result) > 0)
            result = Objects.requireNonNull(e);

    return result;
}
```

##### 컬렉션에서 최댓값을 구해 Optional<E>로 반환한다.
```java
public static <E extends Comparable<E>>
Optional<E> max(Collection<E> c) {
    if (c.isEmpty())
        return Optional.empty();

    E result = null;
    for (E e : c)
        if (result == null || e.compareTo(result) > 0)
            result = Objects.requireNonNull(e);

    return Optional.of(result);
}
```

위와 같이 빈 컬렉션을 건냈을 때 IllegalArgumentException을 던지는 것보다 `Optional<E>`를 반환하는 편이 더 낫다.

- `Optional.empty()`: 빈 Optional을 만드는 메서드
- `Optional.of(value)`: 값이 든 Optional을 만드는 메서드
    - (주의) `Optional.of(value)`에 null을 넣으면 NullPointException을 던진다. null 값도 허용하는 Optional을 만들려면 `Optional.ofNullable(value)`를 사용해야 한다.
    - `Optional을 반환하는 메서드에서는 절대 null을 반환하지 말자`

#### 스트림의 종단 연산에서 옵셔널을 반환하는 방식을 사용할 경우
##### 컬렉션에서 최댓값을 구해 `Optional<E>`로 반환한다. - 스트림 버전
```java
public static <E extends Comparable<E>>
Optional<E> max(Collection<E> c) {
    return c.stream().max(Comparator.naturalOrder());
}
```

#### Optional을 반환했을 때의 장점
- 옵셔널은 검사 예외와 취지가 비슷하다. 즉, 반환 값이 없을 수도 있음을 API 사용자에게 명확히 알려준다. 이렇게 검사 예외를 던진다면 클라이언트는 반드시 이에 대해 대처하는 코드를 작성해야 한다.

#### Optional을 반환했을 때 클라이언트가 취할 행동

##### 옵셔널 활용 1 - 기본 값을 정해둘 수 있다.
```java
String lastWordInLexicon = max(words).orElse("단어 없음...");
```

##### 옵셔널 활용 2 - 원하는 예외를 던질 수 있다.
```java
Toy myToy = max(toys).orElseThrow(TemperTantrumException::new);
```

##### 옵셔널 활용 3 - 항상 값이 채워져 있다고 가정한다. (NoSuchElementException을 주의한다)
```java
Element lastNobleGas = max(Elements.NOBLE_GASES).get();
```

##### 옵셔널 활용 4 - 기본 값 설정 비용이 클 경우
```java
public T orElse(T other)

public static String orElseBenchmark() {
    return Optional.of("baeldung").orElse(getRandomName());
}
```
```java
public T orElseGet(Supplier<? extends T> other)

public static String orElseGetBenchmark() {
    return Optional.of("baeldung").orElseGet(() -> getRandomName());
}
```
- `Supplier<T>`를 인수로 받는 `orElseGet`을 사용하면 초기 설정 비용을 낮출 수 있다.

##### 옵셔널 활용 5 - isPresent 메서드
- 옵셔널이 채워져 있으면 true, 비어있으면 false를 반환
- 원하는 모든 메서드를 수행할 수 있으나 신중히 사용하자. isPresent를 사용하기 전에 앞의 옵셔널 활용 1~4로 표현할 수 있는지 면밀히 검토하자. 그 편이 더 짧고 명확하며 용법에 맞는 코드이다.
```java
public class ParentPid {
    public static void main(String[] args) {
        ProcessHandle ph = ProcessHandle.current();

        // Inappropriate use of isPresent
        Optional<ProcessHandle> parentProcess = ph.parent();
        System.out.println("Parent PID: " + (parentProcess.isPresent() ?
                String.valueOf(parentProcess.get().pid()) : "N/A"));

        // Equivalent (and superior) code using orElse
        System.out.println("Parent PID: " +
            ph.parent().map(h -> String.valueOf(h.pid())).orElse("N/A"));
    }
}
```
##### 스트림을 사용할 경우

혹은 java8의 stream을 이용하여 아래와 같이 표현할 수 있다. 옵셔널에 값이 있다면 (Optional::isPresent) 그 값을 꺼내 (Optional::get) 스트림에 매핑한다.
```java
streamOfOptionals
    .filter(Optional::isPresent)
    .map(Optional::get)
```

자바 9부터는 Optional을 stream으로 변환해주는 Optional.stream() 메서드도 추가되엇다. 옵셔널에 값이 있으면 그 값을 원소로 담은 스트림으로, 값이 없다면 빈 스트림으로 변환한다. 이를 Stream의 flatMap 메서드와 조합하면 아래와 같이 바꿀 수 있다.

```java
streamOfOptionals.flatMap(Optional::stream)
```

#### 옵셔널을 사용하면 안되는 경우
- `컬렉션, 스트림, 배열, 옵셔널 같은 컨테이너 타입은 옵셔널로 감싸면 안된다.`
    - 빈 `Optional<List<T>>`를 반환하기보다는 빈 `List<T>`를 반환하는 게 좋다. 빈 컨테이너를 그대로 반환하면 클라이언트에 옵셔널 처리 코드를 넣지 않아도 된다.

#### 옵셔널을 사용하면 좋은 경우
- `결과를 알 수 없으며, 클라이언트가 이 상황을 특별하게 처리해야 한다`면 `Optional<T>`를 반환한다.
- 단, Optional을 사용하면 새로 할당하고 객체를 초기화하며 값을 꺼내기 위해 메서드를 호출하는 한 단계를 더 거치기 때문에 성능이 저하될 수 있다. 따라서, `성능이 중요한 상황에는 옵셔널이 맞지 않을 수 있다.`

#### int, long, double 전용 옵셔널 클래스가 있음을 기억하고, 박싱된 기본 타입을 담은 옵셔널을 반환하는 일은 없도록 하자
- OptionalInt
- OptionalLong
- OptionalDouble

#### 옵셔널을 맵의 값으로 사용하면 절대 안된다.
- 맵 안에 키가 없다는 사실을 나타내는 방법이 두 가지가 된다. 
    1. 하나는 키 자체가 없는 경우
    2. 키는 있지만 그 키가 속이 빈 옵셔널인 경우
- 복잡성을 높여서 오류 가능성을 키우기 때문에 사용하지 말자. 더 일반화하여 이야기하면 `옵셔널을 컬렉션의 키, 값, 원소나 배열의 원소로 사용하는 게 적절한 상황은 거의 없다.`

---

### 핵심 정리
- 값을 반환하지 못할 가능성이 있고, 호출할 때마다 반환 값이 없을 가능성을 염두에 둬야 하는 메서드라면 옵셔널을 반환해야할 상황일 수 있다.
- 하지만 옵셔널 반환에는 성능 저하가 뒤따르니, 성능에 민감한 메서드라면 null을 반환하거나 예외를 던지는 편이 나을 수도 있다.
- 옵셔널을 반환값 이외의 용도로 쓰는 경우는 매우 드물다.

### 참고 자료
- Effective Java 3/E