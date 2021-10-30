---
description: 공개된 API 요소에는 항상 문서화 주석을 작성하라
---

# Item 56

- API를 쓸모 있게 하려면 잘 작성된 문서도 곁들여야 한다.
- 자바에서는 자바독(Javadoc)이라는 유틸리티가 이를 돕는다.

- [Presentation](https://github.com/SeokRae/TIL/blob/master/java/effactive/item56/item56.pdf)

## Intro

- 자바독은 소스코드 파일에서 문서화 주석(doc comment: 자바독 주석)이라는 특수한 형태로 기술된 설명을 추려 API 문서로 변환해준다.
- 자바 버전이 올라가며 추가된 중요한 자바독 태그
	- 자바 5의 `@literal`, `@code`
	- 자바 8의 `@implSpec`
	- 자바 9의 `@index`

> API를 올바로 문서화 하려면 공개된 모든 클래스, 인터페이스, 메서드, 필드 선언에 문서화 주석을 달아야 한다.

- 직렬화할 수 있는 클래스의 경우 직렬화 형태에 관해서도 적어야 한다.[아이템 87]()
- 문서화 주석이 없다면 자바독도 그저 공개 API요소들의 '선언'만 나열해주는게 전부이다.
- 문서가 잘 갖춰지지 않은 API는 쓰기 헷갈려서 오류의 원인이 되기 쉽다.

- 기본 생성자에는 문서화 주석을 달 방법이 없으니 공개 클래스는 절대 기본 생성자를 사용하면 안된다.
- 유지보수까지 고려한다면 대다수의 공개되지 않은 `클래스`, `인터페이스`, `생성자`, `메서드`, `필드`에도 문서화 주석을 달아야 한다.

## 메서드용 문서화 주석

- 해당 메서드와 클라이언트 사이의 규약을 명료하게 기술해야 한다.
- 상속용으로 설계된 클래스의 메서드가 아니라면 무엇을 하는지 기술해야 한다. [아이템 19]()
- 즉, how가 아닌 what을 기술해야한다.

- 문서화 주석에는 클라이언트가 해당 메서드를 호출하기 위한 전제조건을 모두 나열해야 한다.
- 메서드가 성공적으로 수행된 후에 만족해야 하는 사후조건(postcondition)도 모두 나열해야 한다.

### 문서화 주석 정보

- 일반적으로 전제 조건은 `@throws` 태그로 비검사 예외를 선언하여 암시적으로 기술한다.
	- 비검사 예외 하나가 전제조건 하나와 연결되는 것이다.
- 또한, `@param` 태그를 이용해 그 조건에 영향 받는 매개변수에 기술할 수도 있다.

### 부작용

- 전제조건과 사후조건뿐만 아니라 부작용도 문서화해야 한다.
- 부작용이란 사후조건으로 명확히 나타나지는 않지만 시스템의 상태에 어떠한 변화를 가져오는 것을 뜻한다.
- 예로 백그라운드 스레드를 시작시키는 메서드라면 그 사실을 문서에 밝혀야 한다.

### 메서드의 계약(contract)

- 이를 완벽히 기술하려면 모든 매개변수에 `@param` 태그를 달아야 한다.
- 반환 타입이 void가 아니라면 `@return` 태그를 달아야 한다.
- 발생할 가능성이 있는 모든 예외에 `@throws` 태그를 달아야 한다.[아이템 74]()

### 태그 컨벤션

- `@param` 태그와 `@return` 태그의 설명은 해당 **매개변수가 뜻하는 값**이나 **반환값을 설명하는 명사구** 또는 **산술 표현식**을 쓰기도 한다.
- 예시
	- BigInteger API
		- @throws 태그의 설명은 if로 시작해 해당 예외를 던지는 조건을 설명하는 절이 뒤따른다.
		- 역시 관례상 @param, @return, @throws 태그의 설명에는 마침표를 붙이지 않는다.

```java
public interface List<E> extends Collection<E> {
    // Positional Access Operations

    /**
     * Returns the element at the specified position in this list.
     *
     * @param index index of the element to return
     * @return the element at the specified position in this list
     * @throws IndexOutOfBoundsException if the index is out of range
     *         ({@code index < 0 || index >= size()})
     */
    E get(int index);

}
```

- 문서화 주석 안의 HTML 요소들이 최종 HTML 문서에 반영된다.
- 자바독 설명에 HTML table을 넣는 것도 가능하다.

- `{@code}` 태그
	- 태그로 감산 내용을 코드용 폰트로 렌더링한다.
	- 태그로 감싼 내용에 포함된 HTML 요소나 다른 자바독 태그를 무시한다.
		- HTML 메타문자인 `<` 기호 등을 별다른 처리 없이 바로 사용할 수 있다.
		- 문서화 주석에 여러 줄로 된 코드 예시를 넣으려면 `{@code}` 태그를 다시 `<pre>` 태그로 감싸면 된다.
		- `<pre></pre>`를 사용하면 코드의 줄바꿈이 그대로 유지된다.
	- 단, @기호에는 무조건 탈출문자를 붙여야 하니 문서화 주석 안의 코드에서 어노테이션을 사용한다면 주의해야 한다.

- 영문 문서화 주석에서 쓴 **`this list`** 는 관례상, 인스턴스 메서드의 문서화 주석에 쓰인 `this`는 호출된 메서드가 자리하는 객체를 가리킨다.

### 클래스 상속용으로 설계 시 문서화 주석

- 자기사용 패턴(self-use pattern)에 대해서 문서에 남겨 다른 프로그래머에게 그 메서드를 올바로 재정의하는 방법을 알려줘야 한다.
- 자기사용 패턴을 자바 8에 추가된 `@implSpec` 태그로 문서화한다.
- 일반적인 문서화 주석은 해당 메서드와 클라이언트 사이의 계약을 설명한다.

- 반면, `@implSpec` 주석은 해당 메서드와 하위 클래스 사이의 계약을 설명하여, 하위 클래스들이 그 메서드를 상속하거나 super 키워드를 이용해 호출할 때 그 메서드가 어떻게 동작하는지를 명확히 인지하고 사용하도록 해줘야 한다.

```java
public interface List<E> extends Collection<E> {

    /**
     * Returns true< if this list contains no elements.
     * @implSpec
     * This implementation returns {@code this size ==0}.
     *
     * @return true if this list contains no elements
     */
    boolean isEmpty();
}
```

- 자바 11까지도 자바독 명령줄에서 `-tag "implSpec:a:Implementation Requirements:"` 스위치를 켜주지 않으면 `@implSpec` 태그를 무시해 버린다.

### API 설명에 HTML 메타 문자를 포함하기 위한 처리

- `{@literal}` 태그로 감싸면 HTML 마크업이나 자바독 태그를 무시하게 해준다.
- `{@code}` 태그와 비슷하지만 코드 폰트로 렌더링하지 않는다.

```text
* {@literal |r| < 1}
```

- 문서화 주석은 코드에서건 변환된 API 문서에서건 읽기 쉬워야 한다는게 일반 원칙이다.

## API 문서에서의 가독성

- 각 문서화 주석의 첫 번째 문장은 해당 요소의 요약 설명(summary description)으로 간주된다.
- 요약 설명은 반드시 대상의 기능을 고유하게 기술해야 한다.
	- 헷갈리지 않으려면 한 클래스(혹은 인터페이스) 안에서 요약 설명이 똑같은 멤버(혹은 생성자)가 둘 이상이면 안된다,

- 다중정의된 메서드가 있는 경우 각 메서드들의 설명은 같은 문장으로 시작하는게 자연스럽겠지만 문서화 주석에서는 허용되지 않는다.
- 요약 설명에서는 마침표(.)에 주의해야 한다.
	- 요약 설명이 끝나는 판단 기준은 처음에 발견되는 {<마침표><공백><다음 문장 시작>} 패턴의 <마침표>까지이다.
	- 여기서 <공백>은 스페이스, 탭, 줄바꿈(혹은 첫 번째 블록 태그)이다.
	- <다음 문장 시작>은 '소문자가 아닌' 문자이다.
- 가장 좋은 해결책은 의도하지 않은 마침표를 포함한 텍스트를 {@literal}로 감싸주는 것이다.

```java
/**
 * A suspect, such as Colonel Mustard  or {@literal Mrs. Peacock}
 */
public class Suspect {

}

/**
 * 머스타드 대령이나 {@literal Mrs. 피콕} 같은 용의자. 
 */
public class Suspect {

}
```

- 자바 10부터 {@summary}라는 요약 전용 태그가 추가되어 깔끔하게 처리할 수 있다.

```java
/**
 * {@summary A suspect, such as Colonel Mustard or Mrs. Peacock.}
 */
public enum Suspect {

}
```

- 주석 작성 규약에 따르면 요약 설명은 완전한 문장이 되는 경우가 드물다.
- 메서드와 생성자의 요약 설명은 해당 메서드와 생성자의 동작을 설명하는 (주어가 없는) 동사구여야 한다.

```text
ArrayList(int initialCapacity): Constructs an empty list with the specified initial capacity.
Collection.size(): Returns the number of elements in this collection.
```

- 2인칭 문장(return the number) 이 아닌 3인칭 문장(returns the number)으로 써야 한다.

### 클래스, 인터페이스, 필드의 요약 설명은 대상을 설명하는 명사절이어야 한다.

- 클래스와 인터페이스의 대상은 그 인스턴스이고, 필드의 대상은 필드 자신이다.

```text
Instant:An instantaneous point on the time-line.
Math.PI: The double value that is closer than any other to pi, the ratio of
the circumference of a circle to its diameter.
```

- 자바 9부터는 자바독이 생성한 HTML 문서에 검색(색인) 기능이 추가되어 광대한 API 문서들을 누비는 일이 한결 수월해졌다.
- API 문서 페이지 오른쪽 위에 있는 검색창에 키워드를 입력하면 관련 페이지들이 드롭다운 메뉴로 나타난다.

- 클래스, 메서드, 필드 같은 API 요소의 색인은 자동으로 만들어지며, 원한다면 {@index} 태그를 사용해 여러분의 API에서 중요한 용어를 추가로 색인화할 수 있다.

```java
/**
 * This method complies with the {@index IEEE 754} standard. 
 */
```

## 제네릭, 열거 타입, 어노테이션에서 문서화 주석

### 제네릭 타입이나 제네릭 메서드를 문서화

- 모든 타입 매개변수에 주석을 달아야 한다.

```java
/**
 * @param <K> the type of keys maintained by this map
 * @param <V> the type of mapped values
 */
public interface Map<K, V> {

}
```

### 열거 타입을 문서화

- 상수들에도 주석을 달아야 한다.
- 열거 타입 자체와 그 열거 타입의 public 메서드도 주석을 달아야 한다.

```java
public enum OrchestraSection {
    /** Woodwinds, such as flute, clarinet, and oboe */
    WOODWIND,
    /** Brass instruments, such as french horn and trumpet */
    BRASS,
    /** Percussion instruments, such as timpani and cymbals */
    PERCUSSION,
    /** Stringed instruments, such as violin and cello */
    STRING;
}
```

### 어노테이션 타입을 문서화

- 필드, 멤버들에도 모두 주석을 달아야 한다.
- 필드 설명은 명사구로 한다.
- 어노테이션 타입의 요약 설명은 프로그램 요소에 이 어노테이션을 단다는 것이 어떤 의미인지를 설명하는 동사구로 한다.

```java
/**
 * Indicates that the annotated method is a test method that
 * must throw the designated exception to pass.
 */
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
    /**
     * The exception that the annotated test method must throw
     * in order to pass. (The test is permitted to throw any
     * subtype of the type described by this class object.)
     */
    Class<? extends Throwable> valueO;
}
```

- 패키지를 설명하는 문서화 주석은 package-info.java 파일에 작성한다.
- 이 파일은 패키지 선언을 반드시 포함해야 하며 패키지 선언 관련 어노테이션을 추가로 포함할 수도 있다.

- 자바 9부터 지원하는 모듈 시스템도 이와 비슷하다. [아이템 15]()
- 모듈 시스템을 사용하다면 모듈 관련 설명은 module-info.java 파일에 작성하면 된다.

## API 문서화에서 자주 누락되는 설명 두 가지

- 스레드 안전성과 직렬화 안전성
- 클래스 혹은 정적 메서드가 스레드 안전하든 그렇지 않든, 스레드 안전 수준을 반드시 API 설명에 포함해야 한다.[아이템 82]()
- 직렬화 할 수 있는 클래스라면 직렬화 형태도 API 설명에 기술해야 한다.[아이템 87]()

### 자바독은 메서드 주석 '상속'시킬 수 있다.

- 문서화 주석이 없는 API 요소를 발견하면 자바독이 가장 가까운 문서화 주석을 찾아준다.
- 이때 상위 '클래스'보다 그 클래스가 구현한 '인터페이스'를 먼저 찾는다.

- [The Javadoc Reference Guide](https://docs.oracle.com/javase/7/docs/technotes/tools/windows/javadoc.html)

- 또한 {@inheritDoc} 태그를 사용해 상위 타입의 문서화 주석을 일부 상속할 수 있다.
- 클래스는 자신이 구현한 인터페이스의 문서화 주석을 재사용할 수 있다는 뜻이다.
- 이 기능을 활용하면 거의 똑같은 문서화 주석 여러 개를 유지보수하는 부담을 줄일 수 있지만, 사용하기 까다롭고 제약도 조금 있다.
- [메서드 주석 상속화](https://docs.oracle.com/javase/6/docs/technotes/tools/solaris/javadoc.html#inheritingcomments)

### 문서화 주석 주의사항

- 여러 클래스가 상호작용하는 복잡한 API의 경우 문서화 주석 외에도 전체 아키텍처를 설명하는 별도의 설명이 필요할 때가 있다.
- 이런 설명 문서가 있다면 관련 클래스나 패키지의 문서화 주석에서 그 문서의 링크를 제공해주면 좋다.

## 자바독 문서를 올바르게 작성했는지 확인하는 기능

- 자바 7에서는 CLI에서 -Xdoclint 스위치를 켜주면 기능이 활성화된다.
- 자바 8부터는 기본으로 작동한다.

- 체크 스타일(checkstyle) 같은 IDE 플러그인을 사용하면 더 완벽하게 검사된다.
- 자바독이 생성한 HTML 파일을 HTML 유효성 검사기로 돌리면 문서화 주석의 오류를 한층 더 줄일 수 있다.
- HTML 유효성 검사기는 잘못 사용한 HTML 태그를 찾아준다.
- 로컬에 내려받아 사용할 수 있는 설치형 검사기
- 웹에서 바로 사용할 수 있는 W3C 마크업 검사 서비스[W3C-validator]도 있다.
- 자바 9와 10의 자바독은 기본적으로 HTML 4.01 문서를 생성, 명령줄에서 -html5 스위치를 켜면 HTML5 버전으로 만들어준다.

## 정리

- 문서화 주석은 API를 문서화하는 가장 훌륭하고 효과적인 방법이다.
- 공개 API인 경우 표준 규약을 일관되게 설명을 달아야 한다.
- 문서화 주석에 임의 HTML 태그를 사용할 수 있다.
	- 단, HTML 메타문자는 특별하게 취급해야 한다.
