# item 11. equals를 재정의하려거든 hashCode도 재정의하라

baeldung hashcode: [https://www.baeldung.com/java-hashcode](https://www.baeldung.com/java-hashcode)

## hashCode

Java의 모든 객체는 `hashCode` 메서드를 가진다. Java에서는 HashMap, HashSet 등 hash를 활용한 Collection들을 자주 볼 수 있는데 모두 `hashCode`를 기반으로 하고 있다.

> hashcode 관련해서 NAVER d2 블로그의 [Java HashMap은 어떻게 동작하는가?](https://d2.naver.com/helloworld/831311)를 참고하면 좋을거 같습니다.

`Object#hashCode`구현은 다음과 같다.

```java
public native int hashCode();
```

즉, native call을 통해 해당 객체의 메모리 해쉬 주소를 가져온다. 이 값은 `System#identityHashCode`와 동일한 값을 가진다.

## hashCode의 3가지 규약

**equals를 재정의 했다면 hashCode도 반드시 재정의해야한다.** 그렇지 않으면 hashCode의 일반 규약을 어기게 된다. 이 경우 `HashMap`, `HashSet`과 같이 hashCode를 기반으로 하는 클래스의 원소로 사용하는 경우 예상치 못한 버그를 발견할 수 있다.

`Object#hashCode`는 다음과 같은 규약을 가지고 있다.

1. equals가 변경되지 않았다면 애플리케이션이 실행되는 동안 그 객체의 hashCode는 항상 같은 값을 반환해야한다. 단, 애플리케이션이 재실행하는 경우 값이 달라질 수는 있다.
2. equals가 두 객체를 같다고 판단했다면, 두 객체의 hashCode는 똑같은 값을 반환해야한다.
3. equals가 두 객체를 다르다고 판단하더라도 두 객체의 hashCode가 서로 다른 값을 반환할 필요는 없다. 단, 다른 객체에 대해서는 다른 값을 반환해야 해시 테이블의 성능이 좋아진다.

위 규약 중 두번째 규약을 주의깊게 봐야한다. equals를 재정의하여 논리적으로 같다고 정의했을때 만약 hashCode를 기본 구현으로 사용한다면 서로 다른 값을 반환한다. 즉, equals 재정의로 2번 규약을 어기게되는 것이다.

특히 hashCode는 hash를 기반으로 하는 컬랙션인 HashMap과 HashSet 등에 매우 중요하다.

```java
@Override
public int hashCode() {
    return 1;
}
```

만약 위와 같이 모든 인스턴스에 같은 `hashCode`를 가진다면 어떻게 될까? 물론 Java에서는 hashCode가 같았을때 Exception을 던지지는 않기 때문에 Java 컴파일러에게는 문제가 없는 코드이다. 다만, 모든 객체에게 똑같은 hashCode만을 전달하기 때문에 모든 객체가 해시테이블의 버킷 하나에 담긴다. 때문에 해시테이블의 성능이 매우 떨어지게 된다.

```java
public class SameHashCodeNumber {
    private final int number;

    public SameHashCodeNumber(int number) {
        this.number = number;
    }

    public int getNumber() {
        return number;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        SameHashCodeNumber that = (SameHashCodeNumber) o;
        return number == that.number;
    }

    @Override
    public int hashCode() {
        return 1;
    }
}
```

위와 같이 항상 hashCode를 1을 리턴하는 `SameHashCodeNumber`가 있다고 가정한다.

```java
@Test
@DisplayName("hashCode가 동일한 값을 리턴할 때")
void sameHashCodeNumber() {
    // given
    long start = System.currentTimeMillis();
    Map<SameHashCodeNumber, Integer> sameHashCodeNumbers = IntStream.range(1, 10_000)
            .mapToObj(SameHashCodeNumber::new)
            .collect(Collectors.toMap(sameHashCodeNumber -> sameHashCodeNumber, SameHashCodeNumber::getNumber));

    // when
    Integer actual = sameHashCodeNumbers.get(new SameHashCodeNumber(100));
    long end = System.currentTimeMillis();

    // then
    System.out.println(
            String.format("총 실행 시간: %d", end - start)
    );
    assertThat(actual).isEqualTo(100);
}
```

![](https://user-images.githubusercontent.com/30178507/105105555-a6652300-5af7-11eb-8c5c-817f66043aa2.png)

의외로 조회시간의 성능은 빠르다. 데이터를 넣는 삽입에 걸리는 시간이 너무 느리다. 위 경우 1만개의 데이터를 `HashMap`으로 생성하는데도 1초가 넘게 걸린다.

이 hashCode 구현을 `Objects#hash` 를 사용하도록 변경해본다.

```java
public class DiffHashCodeNumber {
    private final int number;

    public DiffHashCodeNumber(int number) {
        this.number = number;
    }

    public int getNumber() {
        return number;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        DiffHashCodeNumber that = (DiffHashCodeNumber) o;
        return number == that.number;
    }

    @Override
    public int hashCode() {
        return Objects.hash(number);
    }
}
```

위 `DiffHashCodeNumber`는 동일한 hashCode를 반환하지 않는다.

```java
@Test
@DisplayName("hashCode가 다른 값을 리턴할 때")
void diffHashCodeNumber() {
    // given
    long start = System.currentTimeMillis();
    Map<DiffHashCodeNumber, Integer> diffHashCodeNumbers = IntStream.range(1, 10_000_000)
            .mapToObj(DiffHashCodeNumber::new)
            .collect(Collectors.toMap(diffHashCodeNumber -> diffHashCodeNumber, DiffHashCodeNumber::getNumber));

    // when
    Integer actual = diffHashCodeNumbers.get(new DiffHashCodeNumber(5_000));
    long end = System.currentTimeMillis();

    // then
    System.out.println(
            String.format("총 실행 시간: %d", end - start)
    );
		assertThat(actual).isEqualTo(5000);
}
```

![](https://user-images.githubusercontent.com/30178507/105105557-a6fdb980-5af7-11eb-83ed-174731b09b77.png)

hashCode를 인스턴스마다 다르게 준 코드에서는 천만의 데이터를 삽입하더라도 2초 가량밖에 소요되지 않았다.

이렇게 차이가 나는 이유는 해시 충돌이 유무 때문인데 hashCode가 같다면 모든 인스턴스에서 해시충돌이 일어날 것이다. 이때 HashMap에서 해시 충돌을 피하기 위한 구현인 `Separate Chaining`에 따라 하나의 버킷에 모든 데이터가 삽입되기 때문이다.

## 규약에 맞는 hashCode 메서드 짜기

좋은 해시 함수라면 서로 다른 인스턴스끼리 다른 해시코드를 반환해야한다. 이는 hashCode의 세번째 규약이기도 하다. 가장 이상적인 해시 함수는 주어진 인스턴스들을 32비트 정수 범위에 균일하게 분배해야한다.

1. int 변수 result를 선언한 후 값 c로 초기화한다. 이때 c는 해당 객체의 첫번째 핵심 필드를 다음 소개하는 2.a 방식으로 계산한 해시코드이다. 참고로 여기 소개되는 핵심 필드는 equals 비교에 사용되는 필드이다. **equals에서 사용되지 않는 필드는 반드시 hashCode에서도 제외해야한다.** 이를 어기면 hashCode의 두번째 규약을 어기기 때문이다.
2. 해당 객체의 나머지 핵심 필드 f에 대해 각각 다음 작업을 수행한다.

   a. 필드의 해시코드 c를 계산한다.

   - 기본 필드라면 Type.hashCode(f)를 수행한다. 이때 Type은 기본 타입에 매핑되는 래퍼 타입이다.
   - 참조 필드이면서 클래스의 equals가 이 필드의 equals를 재귀적으로 호출해 비교한다면 이 필드의 hashCode가 재귀적으로 호출한다. 만약 필드의 값이 `null`이면 0을 반환한다.
   - 배열이라면 핵심 원소 각각을 별도의 필드로 나눈다. 별도로 나눈 필드를 위 규칙을 적용하여 해시코드를 계산한 후 다음 소개되는 2.b 방식으로 갱신한다. 모든 원소가 핵심이라면 `Arrays.hashCode`를 이용한다.

   b. 2.a로 계산한 해시코드 c로 result를 갱신한다.

   ```java
   result = 31 * result + c
   ```

3. result를 반환한다.

사실 위 구현을 `Objects#hash`가 구현하고 있다.

```java
public final class Objects {

// ...

	public static int hash(Object... values) {
	    return Arrays.hashCode(values);
	}

// ...

}

public final class Arrays {

// ...

	public static int hashCode(Object a[]) {
      if (a == null)
          return 0;

      int result = 1;

      for (Object element : a)
          result = 31 * result + (element == null ? 0 : element.hashCode());

      return result;
  }

// ...

}
```

`Objects#hash`는 `Arrays#hashCode`를 통해 hashCode를 계산한다. 단, 위 코드는 입력 인수를 담기위한 배열을 만들고 인자로 기본 타입이 존재한다면 박싱과 언박싱이 일어나므로 성능이 민감하지 않은 상황에서 사용해야한다. 주로 객체가 해시의 키로 많이 사용되는 경우가 성능에 민감한 경우라고 할 수 있을 거 같다.

성능에 민감한 경우라면 다음과 같이 hashCode를 lazy loading 하는 방식을 사용해 볼 수 있다.

```java
public class LazyLoadHashCodeNumber {
    private final int number;
    private final int number2;

    private int hashCode;

    public LazyLoadHashCodeNumber(int number, int number2) {
        this.number = number;
        this.number2 = number2;
    }

		@Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        LazyLoadHashCodeNumber that = (LazyLoadHashCodeNumber) o;
        return number == that.number && number2 == that.number2;
    }

    @Override
    public int hashCode() {
        int result = hashCode;
        if (result == 0) {
            result = Integer.hashCode(number);
            result = 31 * result + Integer.hashCode(number2);
            hashCode = result;
        }
        return result;
    }
}
```

> 참고로 int는 기본적으로 0 값을 가진다.

위와 같이 필드 hashCode가 0이라면 hashCode 값을 초기화하고 0이 아니라면 이미 계산된 hashCode를 반환하는 방식이다.
