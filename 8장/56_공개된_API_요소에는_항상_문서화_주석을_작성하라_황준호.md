### 아이템56. 공개된 API 요소에는 항상 문서화 주석을 작성하라

코드가 변경되면 문서도 매번 함께 수정해줘야 하는데, javadoc이라는 유틸리티가 이 귀찮은 작업을 도와준다.

메서드용 문서화 주석에는? 

- how가 아닌 what을 기술해야 한다. (어떻게 동작하는지가 아니라 무엇을 하는지 기술해야 한다.)
- 클라이언트가 해당 메서드를 호출하기 위한 전제조건과 사후조건을 모두 나열해야 한다.
- 전제조건은 @throws로 비검사 예외를 선언한다. 비검사 예외 하나당 전제조건 하나와 연결된다.
- @param으로 그 전제조건에 영향받는 매개변수에 기술한다.

```java
/**
 * Returns the element at the specified position in this list.
 *
 * <p>This method is <i>not</i> guaranteed to run in constant
 * time. In some implementations it may run in time proportional
 * to the element position.
 *
 * @param  index index of element to return; must be
 *         non-negative and less than the size of this list
 * @return the element at the specified position in this list
 * @throws IndexOutOfBoundsException if the index is out of range
 *         ({@code index < 0 || index >= this.size()})
 */
E get(int index) {
    return null;
}
```

- HTML 태그를 사용할 수 있다

  - `<p>`, `<i>`등등.. `<table>`도 가능

- `@param`

  - 전제조건에 영향받는 매개변수에 붙인다
  - 모든 매개변수에 붙이는게 좋다
  - 관례상 명사구를 쓰고, 마침표를 붙이지 않는다

- `@return`

  - 반환타입이 void가 아니라면 붙인다
  - void가 아니라도 메서드의 설명과 일치할 경우엔 생략해도 된다
  - 관례상 명사구를 쓰고, 마침표를 붙이지 않는다

- `@throws`

  - 비검사 예외를 선언.
  - 비검사 예외 하나당 전제조건 하나를 연결한다.
  - 관례상 마침표를 붙이지 않는다

- `{@code ...코드... }`

  - 태그로 감싼 내용을 코드용 폰트로 렌더링한다
  - 태그로 감싼 내용에 포함된 HTML요소나 다른 자바독 태그를 무시한다
  - `<pre>{@code ...코드... }</pre>` 와 같이 사용하면 여러줄로 된 코드도 작성 가능하다. 단 @을 쓸땐 탈출문자 붙여야 함

- `@implSpec`

  - 해당 메서드와 하위 클래스 사이의 계약을 설명한다
  - 하위 클래스들이 그 메서드를 상속하거나 super키워드를 이용해 호출할 때 그 메서드가 어떨게 동작하는지를 명확히 인지하고 사용하도록 해야한다

- `{@literal ... }`

  - <, >, &등의 HTML메타문자를 포함시킨다.
  - `{@code ...코드... }` 와 비슷하지만 코드 폰트로 렌더링하진 않는다

- 첫 문장은 주로 요약 설명이다

  - 한 클래스 혹은 한 인터페이스 안에 요약설명이 중복되면 안된다. (특히 오버로딩된 메서드들에서 특히 조심하자)

  - 마침표에 주의해야 한다. 

    - ```java
      /**
       * A suspect, such as Colonel Mustard or Mrs. Peacock. 
      */
      ```

      -> `A suspect, such as Colonel Mustard or Mrs.` 까지 요약설명으로 간주된다.

      ```java
      /**
       * A suspect, such as Colonel Mustard or {@literal Mrs. Peacock}.
      */
      ```

      `@literal`로 감싸줌으로써 해결한다. 자바10부터는 요약 설명 전용 태그 `@summary`가 추가되었다. 이를 활용하면 더 깔끔하게 나타낼 수 있다.

      ```java
      /**
       * {@summary A suspect, such as Colonel Mustard or Mrs. Peacock.}
      */
      ```

    - 메서드와 생성자의 요약 설명

      - 주어가 없는 동사구여야 한다. 2인칭 문장(Return)이 아닌 3인칭 문장(Returns)을 사용해야 한다

        ```java
        /**
         * Constructs an empty list with the specified initial capacity.
         * ...
         */
        public ArrayList(int initialCapacity) { ... }
        ```

        ```java
        /**
         * Returns the number of elements in this collection. 
         * ...
         */
        int size();
        ```

    - 클래스, 인터페이스, 필드의 요약 설명

      - 명사절이어야 한다.

        ```java
        /**
         * The {@code double} value that is closer than any other to
         * <i>pi</i>, the ratio of the circumference of a circle to its
         * diameter.
         */
        public static final double PI = 3.14159265358979323846;
        ```

- `{@index ... }`

  - 중요한 용어를 추가로 색인화할 수 있다.

    ```java
    /**
     * This method complies with the {@index IEEE 754} standard.
     */
    public void function() {
    }
    ```

- 제네릭 문서화

  - 모든 타입 매개변수에 주석을 달아야 한다.

    ```java
    /**
     * An object that maps keys to values.  A map cannot contain duplicate keys; 
     * ...
     * @param <K> the type of keys maintained by this map
     * @param <V> the type of mapped values
     */
    public interface Map<K,V> { ... }
    ```

- 열거 타입 문서화

  - 상수들, 열거 타입 자체, public메서드에도 주석을 달아야 한다

    ```java
    /**
     * An instrument section of a symphony orchestra.
     */
    public enum OrchestraSection {
        /** Woodwinds, such as flute, clarinet, and oboe. */
        WOODWIND,
    
        /** Brass instruments, such as french horn and trumpet. */
        BRASS,
    
        /** Percussion instruments, such as timpani and cymbals. */
        PERCUSSION,
    
        /** Stringed instruments, such as violin and cello. */
        STRING;
    }
    ```

- 애너테이선 타입 문서화

  - 타입 자체, 멤버들에도 모두 주석을 달아야 한다.

  - 필드 설명은 명사구로 한다

  - 요약 설명은 동사구로 한다

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
        Class<? extends Throwable> value();
    }
    ```

<br/>

#### 결론

공개 API라면 표준 규약을 잘 지키며 문서화하자.

