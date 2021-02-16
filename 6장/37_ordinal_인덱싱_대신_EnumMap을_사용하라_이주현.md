## Item 37 ordinal 인덱싱 대신 EnumMap을 사용하라
  - ordinal
    - Returns the ordinal of this enumeration constant (its position in its enum declaration, where the initial constant is assigned an ordinal of zero).
    
```java
package other;

public enum Rainbow {
    RED, ORANGE, YELLOW, GREEN, BLUE, INDIGO, VIOLET
}

class EnumEx {
    public static void main(String[] args) {
        int redIndex = Rainbow.RED.ordinal();
        System.out.println(redIndex); // 0

        int yellowIndex = Rainbow.YELLOW.ordinal();
        System.out.println(yellowIndex); // 2
    }
}

```

<br>

```java
import java.util.EnumMap;
import java.util.HashSet;
import java.util.Map;
import java.util.Set;

public class Plant {
    enum LifeCycle { ANNUAL, PERENNIAL, BIENNIAL }

    final String name;
    final LifeCycle lifeCycle;

    public Plant(String name, LifeCycle lifeCycle) {
        this.name = name;
        this.lifeCycle = lifeCycle;
    }

    @Override
    public String toString() {
        return name;
    }

    public static void main(String[] args) {
        Set<Plant>[] plantsByLifeCycle = (Set<Plant>[]) new Set[Plant.LifeCycle.values().length];

        for (int i = 0; i < plantsByLifeCycle.length; i++) {
            plantsByLifeCycle[i] = new HashSet<>();
        }
        
        for (Plant p : garden) {
            plantsByLifeCycle[p.lifeCycle.ordinal()].add(p);
        }
        

        Map<LifeCycle, Set<Plant>> plantByLifeCycle = new EnumMap<>(Plant.LifeCycle.class);

        for (Plant.LifeCycle lc : Plant.LifeCycle.values()) {
            plantByLifeCycle.put(lc, new HashSet<>());
        }
    }
}
```
<br>

<img width="630" alt="캡처 10" src="https://user-images.githubusercontent.com/50076031/108075525-d0394780-70ad-11eb-9af0-c286bb85a81e.PNG">

<br>

### EnumMap 이란 ?

```java
public class EnumMap<K extends Enum<K>, V> extends AbstractMap<K, V> implements java.io.Serializable, Cloneable { ... }

public abstract class AbstractMap<K,V> implements Map<K,V> { ... }
```

  - EnumMap은 특정 enum 타입만을 key로 사용하는 Map의 구현체
  - enum은 ordinal 이라는 순차적인 정수값을 가지고 있음
  - 순차적인 정수값 ==> 배열 인덱스(index)
  - `EnumMap은 내부 데이터를 Array에 저장`
  - 특정 enum 타입만을 key로 가지기 때문에 인터페이스가 명확해짐
  - EnumMap의 생성자 - 3개

```java
public EnumMap(Class<K> keyType)

public EnumMap(EnumMap<K, ? extends V> m)

public EnumMap(Map<K, ? extends V> m)
```

<br>

```java
package other;

import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;

import java.util.EnumMap;
import java.util.HashMap;
import java.util.Map;

public class EnumMapTest {
    enum TestEnum {
        AA, BB, CC, DD
    }

    public static ObjectMapper objectMapper = new ObjectMapper();

    public static void main(String[] args) throws JsonProcessingException {
        final Map<TestEnum, Integer> map = new HashMap<>();
        map.put(TestEnum.AA, 1);
        map.put(TestEnum.BB, 2);
        map.put(TestEnum.CC, 3);
        map.put(TestEnum.DD, 4);
        String mapMapper = objectMapper.writeValueAsString(map);

        final Map<TestEnum, Integer> enumMap = new EnumMap<>(TestEnum.class);
        enumMap.put(TestEnum.AA, 1);
        enumMap.put(TestEnum.BB, 2);
        enumMap.put(TestEnum.CC, 3);
        enumMap.put(TestEnum.DD, 4);
        String enumMapper = objectMapper.writeValueAsString(enumMap);

        System.out.println("HashMapMapper: " + mapMapper);
        System.out.println("EnumMapMapper: " + enumMapper);

    }
}

// HashMapMapper: {"CC":3,"BB":2,"AA":1,"DD":4}
// EnumMapMapper: {"AA":1,"BB":2,"CC":3,"DD":4}
```

### 핵심 정리
  - 배열의 인덱스를 얻기 위해 ordinal보단 EnumMap을 사용하라.
  - HashMap의 경우 일정 이상의 자료가 저장되면 자체적으로 resizing을 한다.
  - EnumMap은 시작부터 사이즈가 enum으로 제한되기 때문에 문제가 발생할 수 없다.
