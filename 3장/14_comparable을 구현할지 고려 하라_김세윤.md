# 아이템14. Comparable을 구현할지 고려하라

```java
package java.lang;
import java.util.*;

public interface Comparable<T> {

    public int compareTo(T o);
}
```

- 단순 동치성 비교 + 순서까지 비교 + 제네릭
- 이 3가지의 특징을 가지고 있는 메서드가 바로 compareTo이다.
- 따라서 Comparable을 구현한 객체의 배열은 손쉽게 정렬이 가능하다.
- 아울러 자바 플랫폼 라이브러리의 모든 값 클래스, 열거 타입은 모두 Compareable을 구현하고 있다.

## 규약

- 객체가 매개변수로 들어온 객체보다 작으면 음의 정수(-1), 같으면(0), 크다면 양의 정수(+1)을 반환한다.
- 객체와 매개변수로 들어온 객체의 타입이 다르다면 `ClassCastException` 을 반환한다.

### x.compareTo(y) == -y.compareTo(x)

- 두 객체 참조의 순서를 바꿔 비교해도 예상한 결과가 나와야 한다는 의미다.
- 즉, x.compareTo(y)가 1이라면 y.compareTo(x)는 -1이 나와야 된다는 규약이다.

### 추이성 (x.compareTo(y) > 0 && y.compareTo(z) > 0) == x.compareTo(z) > 0

- 두 번째 규약은 첫번째가 두 번째보다 크고 두 번째가 세 번째보다 크다면

### x.compareTo(y) == 0 == (x.compareTo(z) == y.compareTo(z))

- 크기가 같은 객체들끼리는 어떤 객체와 비교하더라도 항상 같아야 한다는 뜻이다.

### (x.compareTo(y) == 0) == x.eqauls(y)

- 이 권고는 필수가 아니지만, 꼭 지키는것이 좋다. (혹시 지키지 못하면 명시해줘야 한다.)
- 지키지 않는다면 컬렉션에 넣으면 해당 컬렉션이 구현한 인터페이스에 정의한 동작과 엇박자를 낼 것이다.
- 정렬된 컬렉션(TreeSet 등)은 equals가 아닌 compareTo를 사용해 동치성을 비교하기 떄문이다.

```java
@Test
void test() {
    BigDecimal number1 = new BigDecimal("1.0");
    BigDecimal number2 = new BigDecimal("1.00");

    Set<BigDecimal> set1 = new HashSet<>();
    set1.add(number1);
    set1.add(number2);

    Set<BigDecimal> set2 = new TreeSet<>();
    set2.add(number1);
    set2.add(number2);

    assertThat(set1).hasSize(2);
    assertThat(set2).hasSize(1);
}
```

### 주의사항

- equals()와 같이 상속을 사용해 새로운 값을 추가하면 규약을 지킬 방법이 없다.
- equals()와 같이 상속이 아닌 컴포지션을 사용하면 이 문제점은 해결할 수 있다.

### 작성 요령

1. 타입을 인수로 받는 제네릭 인터페이스이므로 컴파일 시 인수타입은 정해진다.
2. 동치인지를 비교하는 게 아니라 순서를 비교한다.
3. compareTo()에서 관계연산자 <, >를 사용하는 이전 방식은 거추장스럽게 오류를 유발하기 객체의 compare메서드를 사용하자.
4. 핵심 필드가 여러 개이면 어느 것을 먼저 비교하느냐에 따라 중요해진다. 따라서 핵심적인 필드를 먼저 비교하자

### 예제

- 순서가 학년 기준으로 정렬하고, 학년이 똑같다면 이름기준으로, 동명이인이라면 나이 순으로 순서를 비교할 수 있도록 만들어 보자.

```java
public class Student implements Comparable<Student> {
    private int grade;
    private String name;
    private int age;

    @Override
    public int compareTo(Student o) {
        int result = Integer.compare(grade, o.grade);

        if (result == 0) {
            result = CharSequence.compare(name, o.name);
            if (result == 0) {
                result = Integer.compare(age, o.age);
            }
        }

        return result;
    }
}
```

## Comparator

```java

package java.util;

import java.io.Serializable;
import java.util.function.Function;
import java.util.function.ToIntFunction;
import java.util.function.ToLongFunction;
import java.util.function.ToDoubleFunction;
import java.util.Comparators;

@FunctionalInterface
public interface Comparator<T> {

    int compare(T o1, T o2);

    boolean equals(Object obj);

    default Comparator<T> reversed() {
        return Collections.reverseOrder(this);
    }

    default Comparator<T> thenComparing(Comparator<? super T> other) {
        Objects.requireNonNull(other);
        return (Comparator<T> & Serializable) (c1, c2) -> {
            int res = compare(c1, c2);
            return (res != 0) ? res : other.compare(c1, c2);
        };
    }

    default <U> Comparator<T> thenComparing(
            Function<? super T, ? extends U> keyExtractor,
            Comparator<? super U> keyComparator)
    {
        return thenComparing(comparing(keyExtractor, keyComparator));
    }

    default <U extends Comparable<? super U>> Comparator<T> thenComparing(
            Function<? super T, ? extends U> keyExtractor)
    {
        return thenComparing(comparing(keyExtractor));
    }

    default Comparator<T> thenComparingInt(ToIntFunction<? super T> keyExtractor) {
        return thenComparing(comparingInt(keyExtractor));
    }

    default Comparator<T> thenComparingLong(ToLongFunction<? super T> keyExtractor) {
        return thenComparing(comparingLong(keyExtractor));
    }

    default Comparator<T> thenComparingDouble(ToDoubleFunction<? super T> keyExtractor) {
        return thenComparing(comparingDouble(keyExtractor));
    }

    public static <T extends Comparable<? super T>> Comparator<T> reverseOrder() {
        return Collections.reverseOrder();
    }

    @SuppressWarnings("unchecked")
    public static <T extends Comparable<? super T>> Comparator<T> naturalOrder() {
        return (Comparator<T>) Comparators.NaturalOrderComparator.INSTANCE;
    }

    public static <T> Comparator<T> nullsFirst(Comparator<? super T> comparator) {
        return new Comparators.NullComparator<>(true, comparator);
    }

    public static <T> Comparator<T> nullsLast(Comparator<? super T> comparator) {
        return new Comparators.NullComparator<>(false, comparator);
    }

    public static <T, U> Comparator<T> comparing(
            Function<? super T, ? extends U> keyExtractor,
            Comparator<? super U> keyComparator)
    {
        Objects.requireNonNull(keyExtractor);
        Objects.requireNonNull(keyComparator);
        return (Comparator<T> & Serializable)
            (c1, c2) -> keyComparator.compare(keyExtractor.apply(c1),
                                              keyExtractor.apply(c2));
    }

    public static <T, U extends Comparable<? super U>> Comparator<T> comparing(
            Function<? super T, ? extends U> keyExtractor)
    {
        Objects.requireNonNull(keyExtractor);
        return (Comparator<T> & Serializable)
            (c1, c2) -> keyExtractor.apply(c1).compareTo(keyExtractor.apply(c2));
    }

    public static <T> Comparator<T> comparingInt(ToIntFunction<? super T> keyExtractor) {
        Objects.requireNonNull(keyExtractor);
        return (Comparator<T> & Serializable)
            (c1, c2) -> Integer.compare(keyExtractor.applyAsInt(c1), keyExtractor.applyAsInt(c2));
    }

    public static <T> Comparator<T> comparingLong(ToLongFunction<? super T> keyExtractor) {
        Objects.requireNonNull(keyExtractor);
        return (Comparator<T> & Serializable)
            (c1, c2) -> Long.compare(keyExtractor.applyAsLong(c1), keyExtractor.applyAsLong(c2));
    }

    public static<T> Comparator<T> comparingDouble(ToDoubleFunction<? super T> keyExtractor) {
        Objects.requireNonNull(keyExtractor);
        return (Comparator<T> & Serializable)
            (c1, c2) -> Double.compare(keyExtractor.applyAsDouble(c1), keyExtractor.applyAsDouble(c2));
    }
}
```

- 일련의 비교자 생성 메서드와 팀을 꾸려 메서드 연쇄 방식으로 비교자를 생성할 수 있게 되었다.
- 이 방식의 간결함에 매혹되지만, 약간의 성능 저하가 뒤따르게 된다.
- 아울러 정적 임포트 기능을 사용하면 코드가 훨씬 깔끔해진다는 장점이 있다.
- 위의 코드에서 보듯이 수많은 보조 생성 메서드들로 중무장하고 있다.
- 또한 객체 참조용 비교자 생성 메서드도 준비하고 있다.

### 예제코드

```java
import java.util.Comparator;

public class Student implements Comparable<Student> {
    private static final Comparator<Student> COMPARATOR =
            Comparator.comparingInt((Student student) -> student.grade)
                    .thenComparing((Student student) -> student.name)
                    .thenComparingInt((Student student) -> student.age);

    private int grade;
    private String name;
    private int age;

    @Override
    public int compareTo(Student o) {
        return COMPARATOR.compare(this, o);
    }
}
```

### 주의 사항

- 두 값의 차이를 가지고 비교를 하는 방법도 있다. 아래의 코드처럼 hashCode를 비교한다고 해보자.

```java
private static final Comparator<Student> HASHCODE_COMPARATOR = new Comparator<Student>() {
    @Override
    public int compare(Student o1, Student o2) {
        return o1.hashCode() - o2.hashCode();
    }
};
```

- 위와 같은 코드는 가장 큰 단점이 있다. 바로 정수 오버플로우나 부동소수점 계산 방식에 따른 오류를 발생시킬수도 있다는 것이다.
- 따라서 위의 코드처럼 사용하는것을 추천한다.

```java
private static final Comparator<Student> HASHCODE_COMPARATOR = new Comparator<Student>() {
    @Override
    public int compare(Student o1, Student o2) {
        return Integer.compare(o1.hashCode(), o2.hashCode());
    }
};

private static final Comparator<Student> HASHCODE_COMPARATOR = Comparator.comparingInt(
            Object::hashCode);
```

## 결론

- 알파벳, 숫자, 연대 같이 순서가 명확한 값 클래스를 작성한다면 반드시 Comparable 인터페이스를 구현하자.
- `compareTo` 메서드에서 필드의 값을 비교할 때 `<` 와 `>` 연산자는 쓰지 말아야 한다. 그 대신 박싱된 기본 타입 클래스가 제공하는 `정적 compare` 메서드나 `Comparator` 인터페이스가 제공하는 비교자 생성 메서드를 사용하자.
- 두 값의 차이로 비교값을 사용하지 말자.

## 참고자료
[이펙티브 자바 3판](https://www.aladin.co.kr/shop/wproduct.aspx?ItemId=171196410)
