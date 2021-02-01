# [Effective Java] Item 19. 상속을 고려해 설계하고 문서화하라. 그러지 않았다면 상속을 금지하라.

### 핵심 정리
```
상속용 클래스 설계하기 
1. 클래스 내부에서 스스로 어떻게 사용하는지 (자기 사용 패턴) 모두 문서로 남겨야 한다.
2. 문서화한 것은 그 클래스가 쓰이는 한 반드시 지켜야 한다. (그러지 않으면 그 내부 구현 방식을 믿고 활용하던 하위 클래스를 오동작하게 만들 수 있다.)
3. 다른 이가 효율 좋은 하위 클래스를 만들 수 있도록 일부 메서드를 protected로 제공해야 할 수도 있다. 
4. 클래스를 확장해야 할 명확한 이유가 떠오르지 않으면 상속을 금지하자
5. 상속을 금지하려면 클래스를 final로 선언하.거나 생성자 모두를 외부에서 접근할 수 없도록 만들면 된다.
```

---

### 상속용 메서드를 정의할 때 주의해야 할 점

#### 1. 메서드를 재정의하면 어떤 일이 일어나는지 정확히 정리하여 문서로 남겨야 한다. 즉, 상속용 클래스는 재정의할 수 있는 메서드들을 내부적으로 어떻게 이용하는지 문서로 남겨야 한다.
> 재정의 가능한 메서드: `public과 protected 메서드 중 final이 아닌 모든 메서드`
- 재정의 가능한 메서드의 API 설명
    - 어떤 순서로 호출하는지
    - 각각의 호출 결과가 이어지는 처리에 어떤 영향을 주는지
    - 재정의 가능 메서드를 호출할 수 있는 모든 상황
    - 메서드의 내부 동작 (Implementation Requirements @implSpec)

- 아래와 같이 메서드의 내부 동작과 호출 결과가 어떤 영향을 끼치는지를 명시해 주면 좋다.
    
    [AbstractCollection Java 11 document](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/AbstractCollection.html#remove(java.lang.Object))
    
    <details>
    <summary>AbstractCollection의 remove 메서드 문서 (재정의 가능한 메서드의 API 문서 예시)</summary>
    <div markdown="1">

    > public boolean remove(Object o)
    > 주어진 원소가 이 컬렉션 안에 있다면 그 인스턴스를 하나 제거한다. 더 정확하게 말하면 이 컬렉션 안에 'Object.equals(e, e)가 참인 원소' e가 하나 이상 있다면 그 중 하나를 제거한다. 주어진 원소가 컬렉션 안에 있다면 true를 반환한다.
    >
    > @implSpec (Implementation Requirements)
    > 이 메서드는 컬렉션을 순회하며 주어진 원소를 찾도록 구현되었다. 주어진 원소를 찾으면 반복자의 remove 메서드를 사용해 컬렉션에서 제거한다. 이 컬렉션이 주어진 객체를 갖고 있으나, 이 컬렉션의 iterator 메서드가 반환한 반복자가 remove 메서드를 구현하지 않았으면 UnsupportedOperationException을 던지니 주의하자.
    > 
    - 이 설명으로 iterator 메서드를 재정의하면 remove 메서드의 동작에 영향을 줌을 알 수 있다.
    - iterator 메서드로 얻는 반복자의 동작이 remove 메서드의 동작에 주는 영향도 정확히 설명했다.
    - 클래스를 안전하게 상속할 수 있도록 하려면 내부구현 방식을 설명해야만 한다.
    - @implSpec 태그는 선택값이라 활성화하려면 `-tag "implSpec:a:Implementation Requirements:"를 지정해주어야 한다.

    ```java
    /**
        * {@inheritDoc}
        *
        * @implSpec
        * This implementation iterates over the collection looking for the
        * specified element.  If it finds the element, it removes the element
        * from the collection using the iterator's remove method.
        *
        * <p>Note that this implementation throws an
        * {@code UnsupportedOperationException} if the iterator returned by this
        * collection's iterator method does not implement the {@code remove}
        * method and this collection contains the specified object.
        *
        * @throws UnsupportedOperationException {@inheritDoc}
        * @throws ClassCastException            {@inheritDoc}
        * @throws NullPointerException          {@inheritDoc}
        */
        public boolean remove(Object o) {
            Iterator<E> it = iterator();
            if (o==null) {
                while (it.hasNext()) {
                    if (it.next()==null) {
                        it.remove();
                        return true;
                    }
                }
            } else {
                while (it.hasNext()) {
                    if (o.equals(it.next())) {
                        it.remove();
                        return true;
                    }
                }
            }
            return false;
        }

    ```
    
    </div>
    </details>


#### 2. 효율적인 하위 클래스를 어려움 없이 만들 수 있게 하기 위해 클래스의 내부 동작 과정 중간에 끼어들 수 있는 훅(hook)을 잘 선별하여 protected 메서드 형태로 공개해야 할 수도 있다.
.
    [AbstractList Java 11 document](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/AbstractList.html#removeRange(int,int))

<details>
<summary>java.util.AbsractList의 removeRange 메서드</summary>
<div markdown="1">

> protected void removeRange(int fromIndex, int toIndex)
> fromIndex(포함)부터 toIndex(미포함)까지의 모든 원소를 이 리스트에서 제거한다.
> toIndex 이후의 원소들은 앞으로 (index만큼씩) 당겨진다. 이 호출로 리스트는 'toIndex - fromIndex'만큼 짧아진다. (toIndex == fromIndex라면 아무 효과도 없다.)
> 이 리스트 혹은 이 리스트의 부분리스트에 정의된 clear 연산이 이 메서드를 호출한다. 리스트 구현의 내부 구조를 활용하도록 이 메서드를 재정의하면 이 리스트와 부분리스트의 clear 연산 성능을 크게 개선할 수 있다.
> **Implementation Requirements**: 이 메서드는 fromIndex에서 시작하는 리스트 반복자를 얻어 모든 원소를 제거할 때까지 ListIterator.next와 ListIterator.remove를 반복 호출하도록 구현되었다. **주의: ListIterator.remove가 선형 시간이 걸리면 이 구현의 성능은 제곱에 비례한다.**
>
> Parameters:
> fromIndex     제거할 첫 원소의 인덱스
> toIndex       제거할 마지막 원소의 다음 인덱스

- List의 구현체의 최종 사용자는 removeRange 메서드에 관심이 없다. 그럼에도 이 메서드를 제공하는 이유는 단지 하위 클래스에서 부분리스트의 clear 메서드를 고성능으로 만들기 쉽게 하기 위해서다.
- removeRange 메서드가 없다면 하위 클래스에서 clear 메서드를 호출하려면 (제거할 원소 수의) 제곱에 비례해 성능이 느려지거나 메커니즘을 밑바닥부터 새로 구현해야 했을 것이다.

```java
    /**
     * Removes from this list all of the elements whose index is between
     * {@code fromIndex}, inclusive, and {@code toIndex}, exclusive.
     * Shifts any succeeding elements to the left (reduces their index).
     * This call shortens the list by {@code (toIndex - fromIndex)} elements.
     * (If {@code toIndex==fromIndex}, this operation has no effect.)
     *
     * <p>This method is called by the {@code clear} operation on this list
     * and its subLists.  Overriding this method to take advantage of
     * the internals of the list implementation can <i>substantially</i>
     * improve the performance of the {@code clear} operation on this list
     * and its subLists.
     *
     * @implSpec
     * This implementation gets a list iterator positioned before
     * {@code fromIndex}, and repeatedly calls {@code ListIterator.next}
     * followed by {@code ListIterator.remove} until the entire range has
     * been removed.  <b>Note: if {@code ListIterator.remove} requires linear
     * time, this implementation requires quadratic time.</b>
     *
     * @param fromIndex index of first element to be removed
     * @param toIndex index after last element to be removed
     */
    protected void removeRange(int fromIndex, int toIndex) {
        ListIterator<E> it = listIterator(fromIndex);
        for (int i=0, n=toIndex-fromIndex; i<n; i++) {
            it.next();
            it.remove();
        }
    }
```
</div>
</details>

#### 3. 상속용으로 설계한 클래스는 배포 전에 반드시 하위 클래스를 만들어 검증하라.
- 꼭 필요한 protected 멤버를 놓쳤다면 하위 클래스를 작성할 때 그 빈자리가 확연히 드러난다. 거꾸로, 하위 클래스를 여러 개 만들 때까지 전혀 쓰이지 않는 protected 멤버는 사실 private이었어야 할 가능성이 크다.
- 이러한 하위 클래스는 3개 이상 작성하되 하나 이상은 제 3자가 작성하는게 적당하다.
- 널리 쓰일 클래스를 상속용으로 설계한다면 반드시 문서화한 내부 사용 패턴과, protected 메서드와 필드를 구현하면서 이 결정이 그 클래스의 성능과 기능에 영원한 족쇄가 될 수 있음을 명시한다.

#### 4. 상속용 클래스의 생성자는 직접적으로든 간접적으로든 재정의 가능 메서드를 호출해서는 안 된다.
- 상위 클래스의 생성자가 하위 클래스의 생성자보다 먼저 실행되므로 하위 클래스에서 재정의한 메서드가 하위 클래스의 생성자보다 먼저 호출된다. 
- 이 때 재정의한 메서드가 하위 클래스의 생성자에서 초기화하는 값에 의존한다면 의도대로 동작하지 않을 것이다.
    ```java
    public class Super {
        // 잘못된 예시 - 생성자가 재정의 가능 메서드를 호출한다.
        public Super() {
            overrideMe();
        }

        public void overriedMe() {

        }
    }
    ```
    ```java
    public final class Sub extends Super {
        // 초기화되지 않은 final 필드, 생성자에서 초기화한다.
        private final Instant instant;

        Sub() {
            instant = Instant.now();
        }

        // 재정의 가능 메서드, 상위 클래스의 생성자가 호출한다.
        @Override public void overrideMe() {
            System.out.println(instant);
        }

        public static void main(String[] args) {
            Sub sub = new Sub();
            sub.overrideMe();
        }
    }
    ```
    - 이 프로그램은 instance를 두 번 출력하지 않고, 첫 번째는 null을 출력한다.
    - 상위 클래스의 생성자는 하위 클래스의 생성자가 인스턴스 필드를 초기화하기도 전에 overrideMe를 호출하기 때문이다.
    - private, final, static 메서드는 재정의가 불가능하니 생성자에서 안심하고 호출해도 된다.

#### 5. clone과 readObject 모두 직접적으로든 간접적으로든 재정의 가능 메서드를 호출해서는 안 된다.
    - Clonable이나 Serializable을 구현할지 정해야 한다면 이들을 구현할 때 따르는 제약도 생성자와 비슷하다는 점에 주의해야 한다.
    - readObject의 경우 하위 클래스의 상태가 미처 다 역직렬화되기 전에 재정의한 메서드부터 호출하게 된다. 
    - clone의 경우 하위 클래스의 clone 메서드가 복제본의 상태를 (올바른 상태로) 수정하기 전에 재정의한 메서드를 호출한다.
    - clone이 잘못되어 깊은 복사를 하다가 원본 객체의 일부를 참조하고 있다면 원본 객체에까지도 피해를 줄 수 있다.

#### 6. Serializable을 구현한 상속용 클래스가 readResolve나 write Replace 메서드를 갖는다면 이 메서드들은 private이 아닌 protected로 선언해야 한다.
    - private로 선언한다면 하위 클래스에서 무시된다.
    - 상속을 허용하기 위해 내부 구현을 클래스 API로 공개하는 예 중 하나다.

`상속용으로 설계하지 않은 클래스는 상속을 금지하는 것이 가장 좋다.`

---

### 상속을 금지하는 방법

#### 1. 클래스를 final로 선언한다.

#### 2. 모든 생성자를 private이나 package-private으로 선언하고 public 정적 팩터리를 만들어준다.

기존과 같이 구체 클래스를 상속해 계측, 통지, 동기화, 기능 제약 등의 일부 기능을 추가해야 하는 것이 아니라 상속을 금지하고 대신 아이템 18에서 설명한 래퍼 클래스 패턴을 사용하자.

---

### 상속을 허용해야 한다면?
`재정의 가능 메서드를 호출하는 자기 사용 코드를 완벽하게 제거하자.`

클래스의 동작을 유지하면서 재정의 가능 메서드를 사용하는 코드를 제거하는 방법
    - 각각의 재정의 가능 메서드는 자신의 본문 코드를 private '도우미 메서드'로 옮기고, 이 도우미 메서드를 호출하도록 수정한다.
    - 재정의 가능 메서드를 호출하는 다른 코드들도 모두 이 도우미 메서드를 직접 호출하도록 수정한다.

---

### 참고 자료
- Effective Java 3/E