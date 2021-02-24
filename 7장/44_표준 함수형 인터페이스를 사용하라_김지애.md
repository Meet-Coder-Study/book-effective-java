# [Effective Java] item 44. 표준 함수형 인터페이스를 사용하라

## 핵심정리
- 이제 자바도 람다를 지원하기 때문에 지금부터 API를 설계할 때 람다도 염두에 두어야 한다.
- 입력값과 반환값에 함수형 인터페이스 타입을 활용하라. 보통은 java.util.function 패키지의 표준 함수형 인터페이스를 사용하는 것이 가장 좋은 선택이다.
- 단 흔치는 않지만 직접 함수형 인터페이스를 만들어 쓰는 편이 나을 수도 있음을 잊지 말자.

---

### 불필요한 함수형 인터페이스를 만들지 말고 표준 함수형 인터페이스를 사용하라
#### LinkedHashMap의 removeEldestEntry를 재정의하여 캐시로 사용할 경우
```java
protected boolean removeEldestEntry(Map.Entry<K,V> eldest) {
    return false;
}
```
```java
protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
    return size() > 100;
}
```
- put메서드에서 removeEldestEntry 메서드를 호출하여 true가 반환되면 맵에서 가장 오래된 원소를 제거한다.
- 즉, 100개 이상이 되면 새로운 키가 더해질때마다 하나씩 제거하여 가장 최근 원소 100개를 유지하여 캐시와 같이 사용할 수 있다.

#### 람다를 사용하여 함수형 인터페이스를 만들어 사용할 경우
##### 불필요한 함수형 인터페이스 - 대신 표준 함수형 인터페이스를 사용하라.
```java
@FuntionalInterface 
interface EldestEntryRemovalFunction<K, V> {
    boolean remove(Map<K, V> map, Map.Entry<K, V> eldest);
}
```
<details>
<summary>진짜 되나 실험</summary>
<div markdown="1">

```java
import java.util.Map;

@FunctionalInterface
public interface EldestEntryRemovalFunction<K, V> {
    boolean remove(Map<K, V> map, Map.Entry<K, V> eldest);
}
```
```java
import java.util.LinkedHashMap;
import java.util.Map;
public class MyLinkedHashMap{
    public static void main(String[] args) {
        // OverrideLinkedHashMap
        Map<Integer, Integer> overrideMap = new OverrideLinkedHashMap<>();
        for (int i = 0; i < 10; i++){
            overrideMap.put(i, i);
        }
        System.out.println(overrideMap.keySet());
        
        // FunctionalLinkedHashMap
        Map<Integer, Integer> functionalMap = new FunctionalLinkedHashMap<>((map, eldest) -> map.size() > 5);
        for (int i = 0; i < 10; i++){
            functionalMap.put(i, i);
        }
        System.out.println(functionalMap);
    }
    private static class OverrideLinkedHashMap<K, V> extends LinkedHashMap<K, V> {
        @Override
        protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
            return size() > 5;
        }
    }
    private static class FunctionalLinkedHashMap<K, V> extends LinkedHashMap<K, V> {
        private EldestEntryRemovalFunction<K, V> eldestEntryRemovalFunction;
        public FunctionalLinkedHashMap(EldestEntryRemovalFunction<K, V> function) {
            this.eldestEntryRemovalFunction = function;
        }
        @Override
        protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
            return eldestEntryRemovalFunction.remove(this, eldest);
        }
    }
}
```
```
[5, 6, 7, 8, 9]
{5=5, 6=6, 7=7, 8=8, 9=9}
```

</div>
</details>

- 이를 함수형 인터페이스로 만들면 위와 같이 선언할 수 있다. 이 인터페이스도 잘 동작하기는 하지만, 굳이 사용할 이유는 없다. `자바 표준 라이브러리에 이미 같은 모양의 인터페이스가 준비되어 있기 때문이다.`

### 표준 함수형 인터페이스
- java.util.function 패키지를 보면 다양한 용도의 표준 함수형 인터페이스가 담겨 있다. `필요한 용도에 맞는 게 있다면, 직접 구현하지 말고 표준 함수형 인터페이스를 활용하라.` 그러면 API가 다루는 개념의 수가 줄어들어 익히기 더 쉬워진다.
- 또한 표준 함수형 인터페이스들은 `유용한 디폴트 메서드를 많이 제공하므로 다른 코드와의 상호운용성도 크게 좋아질 것이다.`
    - (예) Predicate 인터페이스
        - 프레디키트(predicate)들을 조합하는 메서드를 제공한다. 
        - LinkedHashMap 예에서는 직접 만든 EldestEntryRemovalFunction 대신 표준 인터페이스인 `BiPredicate<Map<K, V>>`, `Map.Entry<K, V>`를 사용할 수 있다.

### Java.util.function 패키지의 6가지 기본 인터페이스
`java.util.function 패키지에는 총 43개의 인터페이스`가 담겨 있다. 전부 기억하긴 어렵겠지만, `기본 인터페이스 6개`만 기억하면 나머지를 충분히 유추해 낼 수 있다. 이 기본 인터페이스들은 모두 참조 타입용이다. 하나씩 살펴보자.

### 기본 함수형 인터페이스 정리 표
| 인터페이스 | 함수 시그니처 | 의미 |  예 |
| - | - | - | - |
| `UnaryOperator<T>` | `T apply(T t)` | 반환 값과 인수의 타입이 같은 함수, 인수는 1개 | `String::toLowerCase` |
|`BinaryOperator<T>` | `T apply(T t1, T t2)` | 반환 값과 인수의 타입이 같은 함수, 인수는 2개 | `BigInteger::add` |
| `Predicate<T>` | `boolean test(T t)` | 인수 하나를 받아 boolean을 반환하는 함수 | `Collection::isEmpty` |
| `Function<T, R>` | `R apply(T t)` | 인수와 반환 타입이 다른 함수 | `Arrays::asList` |
| `Supplier<T>` | `T get()` | 인수를 받지 않고 값을 반환(혹은 제공)하는 함수 | `Instant::now` |
| `Consumer<T>` | `void accept(T t)` | 인수 하나 받고 반환 값은 없는 함수 | `System.out::println` |

---

표준 함수형 인터페이스 대부분은 기본 타입(int, long, double)만 지원한다. 그렇다고 `기본 함수형 인터페이스에 박싱된 기본 타입을 넣어 사용하지는 말자.` 동작은 하지만 "박싱된 기본 타입 대신 기본 타입을 사용하라"라는 아이템 61의 조언을 위배한다. 특히 계산량이 많을 때는 성능이 처참히 느려질 수 있다.

### 표준 함수형 인터페이스가 아니라 직접 작성하는 것이 좋은 경우

##### 1. 자주 쓰이며, 이름 자체가 용도를 명확히 설명해주는 경우
##### 2. 반드시 따라야하는 규약이 있는 경우
##### 3. 유용한 디폴트 메서드를 제공할 수 있는 경우

자주 보아온 `Comparator<T>` 인터페이스를 떠올려보자. 구조적으로는 `ToIntBiFunction<T,U>`와 동일하다. 심지어 자바 라이브러리에 `Comparator<T>`를 추가할 당시 `ToIntBiFunction<T,U>`가 이미 존재했더라도 `ToIntBiFunction<T,U>`를 사용하면 안 됐다. Comparator가 독자적인 인터페이스로 살아남아야 하는 이유가 몇 개 있다.
1. API에서 굉장히 자주 사용되는데 지금의 이름이 그 용도를 아주 훌륭히 설명해준다.
2. 구현하는 쪽에서 반드시 지켜야할 규약을 담고 있다.
3. 비교자들을 변환하고 조합해주는 유용한 디폴트 메서드들을 듬뿍 담고 있다.

```java
// Comparator
@FunctionInterface
public interface Comparator<T> {
    int compare(T o1, T o2);
}

// ToIntBiFunction
@FunctionalInterface
public interface ToIntBiFunction<T, U> {
    int applyAsInt(T t, U u);
}
```

---

### 직접 만든 함수형 인터페이스에는 항상 @FunctionalInterface 애너테이션을 사용하라.

#### @FuncionalInterface의 세 가지 목적
- 첫 번째, 해당 클래스의 코드나 설명 문서를 읽을 이에게 그 인터페이스가 람다용으로 설계된 것임을 알려준다. 
- 두 번째, 해당 인터페이스가 추상 메서드를 오직 하나만 가지고 있어야 컴파일되게 해준다.
- 세 번쨰, 그 결과 유지보수 과정에서 누군가 실수로 메서드를 추가하지 못하게 막아준다.

### 함수형 인터페이스를 사용할 때의 주의점
서로 다른 함수형 인터페이스를 같은 위치의 인수로 받는 메서드들을 다중정의(Overloading)해서는 안된다. 클라이언트에게 불필요한 모호함만 안겨줄 뿐이며, 이 모호함으로 인해 실제로 문제가 일어나기도 한다. 

```java
public interface ExecutorService extends Executor {
    // Callable<T>와 Runnable을 각각 인수로 받게 Overloading함
    // submit 메서드를 사용할 때마다 형변환이 필요하다
    <T> Future<T> submit(Callback<T> task);
    Future<?> submit(Runnable task);
}
```

---

### 참고 자료
- Effective Java 3/E