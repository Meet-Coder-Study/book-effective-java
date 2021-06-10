---
description: ordinal 인덱싱 대신 EnumMap을 사용하라
---

# Item 37

- [Presentation](https://github.com/SeokRae/TIL/blob/master/java/effactive/item37/item37.pdf)

## Intro

- 배열이나 리스트에서 원소를 꺼낼 때 ordinal 메서드 [아이템 35]()로 인덱스를 얻는 코드가 있다.

```java
/**
 * 식물을 나타내는 클래스
 */
public class Plant {
    // 식물의 생애 주기를 관리하는 열거 타입
    enum LifeCycle {
        ANNUAL, // 한해살이
        PERENNIAL, // 여러해살이
        BIENNIAL // 두해살이
    }

    final String name;
    final LifeCycle lifeCycle;

    Plant(String name, LifeCycle lifeCycle) {
        this.name = name;
        this.lifeCycle = lifeCycle;
    }

    @Override
    public String toString() {
        return name;
    }
}
```

- 정원에 심은 식물들을 배열 하나로 관리하고, 이들을 생애주기(한해살이, 여러해살이, 두해살이)별로 묶는다.
- 생애주기 별로 총 3개의 집합을 만들고 정원을 한 바퀴 돌며 각 식물을 해당 집합에 넣는다.

> 이때, 집합들을 배열 하나에 넣고, 생애주기의 ordinal 값을 그 배열의 인덱스로 사용할 수 있다.

```java
class Client {
    public void addPlant(List<Plant> garden) {
        // 생애주기 3가지로 만들어지는 Set
        Set<Plant>[] plantsByLifeCycle = new Set[Plant.LifeCycle.values().length];

        for (int i = 0; i < plantsByLifeCycle.length; i++) {
            plantsByLifeCycle[i] = new HashSet<>();
        }

        for (Plant p : garden) {
            plantsByLifeCycle[p.lifeCycle.ordinal()].add(p);
        }

        // 인덱스의 의미를 알 수 없어 직접 레이블을 달아 데이터 확인 작업 필요
        for (int i = 0; i < plantsByLifeCycle.length; i++) {
            System.out.printf("%s: %s%n", Plant.LifeCycle.values()[i], plantsByLifeCycle[i]);
        }
    }
}
```

- 정원의 식물 입력 테스트

```java
class ClientTest {

    private List<Plant> garden;

    @BeforeEach
    void setUp() {
        garden = Arrays.asList(
                new Plant("ANNUAL_TREE_1", Plant.LifeCycle.ANNUAL),
                new Plant("ANNUAL_TREE_2", Plant.LifeCycle.ANNUAL),
                new Plant("ANNUAL_TREE_3", Plant.LifeCycle.ANNUAL),
                new Plant("BIENNIAL_TREE_1", Plant.LifeCycle.BIENNIAL),
                new Plant("PERENNIAL_TREE_1", Plant.LifeCycle.PERENNIAL)
        );
    }

    // ANNUAL: [ANNUAL_TREE_3, ANNUAL_TREE_2, ANNUAL_TREE_1]
    // PERENNIAL: [PERENNIAL_TREE_1]
    // BIENNIAL: [BIENNIAL_TREE_1]
    @DisplayName("정원에 있는 식물 등록하기")
    @Test
    void testCase1() {
        // given
        Client client = new Client();

        // then
        client.addPlant(garden);
    }
}
```

> 위 코드에 문제가 한가득이다.

- 배열은 제네릭과 호환되지 않으니 비검사 형변환을 수행해야 하고, 깔끔하게 컴파일되지 않는다. [아이템 28]()
- 배열은 각 인덱스의 의미를 모르니 출력 결과에 직접 레이블을 달아야 한다.
- 가장 심각한 문제는 정확한 정숫값을 사용하고 있다는 것을 개발자가 직접 보장해야 한다.
	- 정수는 열거 타입과 달리 타입 안전하지 않기 때문이다.

- 잘못된 값을 사용하는 경우 배열의 인덱스에 따른 값이 의도한 값을 보장할 수 없고, ArrayIndexOutOfBoundsException을 발생할 수 있다.

## 해결책

- 배열은 실질적으로 열거 타입 상수를 값으로 매핑하는 역할을 한다.
	- 이는 Map을 사용할 수 있도록 한다.
	- 열거 타입을 키로 사용하도록 설계 한 **아주 빠른 Map의 구현체(EnumMap)** 가 존재한다.

> EnumMap을 사용 예시

- Enum 타입을 Key 값으로 하는 EnumMap 구현체

```java
class Client {
    public void addPlantTypeEnumMap(List<Plant> garden) {
        Map<Plant.LifeCycle, Set<Plant>> plantByLifeCycle = new EnumMap<>(Plant.LifeCycle.class);

        for (Plant.LifeCycle lc : Plant.LifeCycle.values()) {
            plantByLifeCycle.put(lc, new HashSet<>());
        }

        for (Plant p : garden) {
            plantByLifeCycle.get(p.lifeCycle).add(p);
        }

        System.out.println(plantByLifeCycle);
    }
}
```

- EnumMap 을 사용하여 구현된 테스트

```java

class ClientTest {

    private List<Plant> garden;

    @BeforeEach
    void setUp() {
        garden = Arrays.asList(
                new Plant("ANNUAL_TREE_1", Plant.LifeCycle.ANNUAL),
                new Plant("ANNUAL_TREE_2", Plant.LifeCycle.ANNUAL),
                new Plant("ANNUAL_TREE_3", Plant.LifeCycle.ANNUAL),
                new Plant("BIENNIAL_TREE_1", Plant.LifeCycle.BIENNIAL),
                new Plant("PERENNIAL_TREE_1", Plant.LifeCycle.PERENNIAL)
        );
    }

    // {ANNUAL=[ANNUAL_TREE_3, ANNUAL_TREE_2, ANNUAL_TREE_1], PERENNIAL=[PERENNIAL_TREE_1], BIENNIAL=[BIENNIAL_TREE_1]}
    @DisplayName("EmumMap을 이용해 정원의 식물 등록하기")
    @Test
    void testCase2() {
        Client client = new Client();

        // then
        client.addPlantTypeEnumMap(garden);
    }
}
```

- Set<EnumType>[] 을 사용하여 관리하는 것보다 EnumMap을 사용하는 것이 간결하고 성능도 비슷하다.
- 안전하지 않은 형변환을 쓰지 않고, 맵의 키인 열거 타입이 그 자체로 출력용 문자열을 제공하기 때문에 출력 결과에 레이블을 달 일도 없다.
- 배열 인덱스를 계산하는 과정에서 오류가 날 가능성도 배제할 수 있다.
- EnumMap의 성능이 ordinal을 쓴 배열에 비견되는 이유는 그 내부에서 배열을 사용하기 때문이다.
- 내부 구현 방식을 안으로 숨겨서 **Map의 타입 안전성**과 **배열의 성능**을 모두 얻어낸 것이다.

- EnumMap의 생성자가 받는 키 타입의 Class 객체는 한정적 타입 토큰으로, 런타임 제네릭 타입 정보를 제공한다. [아이템 33]()

> 스트림 [아이템 45]()을 사용하여 맵을 관리하면 코드를 더 줄일 수 있다.

- Prototype 의 스트림 기반 코드

```java
class ClientTest {

    private List<Plant> garden;

    @BeforeEach
    void setUp() {
        garden = Arrays.asList(
                new Plant("ANNUAL_TREE_1", Plant.LifeCycle.ANNUAL),
                new Plant("ANNUAL_TREE_2", Plant.LifeCycle.ANNUAL),
                new Plant("ANNUAL_TREE_3", Plant.LifeCycle.ANNUAL),
                new Plant("BIENNIAL_TREE_1", Plant.LifeCycle.BIENNIAL),
                new Plant("PERENNIAL_TREE_1", Plant.LifeCycle.PERENNIAL)
        );
    }

    // {ANNUAL=[ANNUAL_TREE_1, ANNUAL_TREE_2, ANNUAL_TREE_3], BIENNIAL=[BIENNIAL_TREE_1], PERENNIAL=[PERENNIAL_TREE_1]}
    @DisplayName("Stream 기반으로 정원의 식물을 생애주기 별로 나열하기")
    @Test
    void testCase3() {
        Map<Plant.LifeCycle, List<Plant>> garden =
                this.garden.stream()
                        // ANNUAL -> BIENNIAL -> PERENNIAL 순으로 확인하기 위한 설정
                        .sorted((o1, o2) -> o2.lifeCycle.ordinal() - o1.lifeCycle.ordinal())
                        .collect(groupingBy(p -> p.lifeCycle));

        System.out.println(garden);
    }
}
```

- 이 코드는 EnumMap이 아닌 고유한 맵 구현체를 사용하기 때문에 EnumMap을 써서 얻은 **공간과 성능 이점**이 사라진다는 문제가 발생한다.

- 위 문제점을 최적화 하기
	- 매개변수 3개 짜리 Collections.groupingBy 메서드는 mapFactory 매개변수에 원하는 맵 구현체를 명시해 호출할 수 있다.
	- 아래 코드는 Map을 빈번하게 사용하는 프로그램에서 최적화하는 방법이다.

```java
class ClientTest {

    private List<Plant> garden;

    @BeforeEach
    void setUp() {
        garden = Arrays.asList(
                new Plant("ANNUAL_TREE_1", Plant.LifeCycle.ANNUAL),
                new Plant("ANNUAL_TREE_2", Plant.LifeCycle.ANNUAL),
                new Plant("ANNUAL_TREE_3", Plant.LifeCycle.ANNUAL),
                new Plant("BIENNIAL_TREE_1", Plant.LifeCycle.BIENNIAL),
                new Plant("PERENNIAL_TREE_1", Plant.LifeCycle.PERENNIAL)
        );
    }

    // {ANNUAL=[ANNUAL_TREE_2, ANNUAL_TREE_1, ANNUAL_TREE_3], PERENNIAL=[PERENNIAL_TREE_1], BIENNIAL=[BIENNIAL_TREE_1]}
    @DisplayName("EnumMap을 이용해 데이터와 열거 타입을 매핑하는 테스트")
    @Test
    void testCase4() {
        EnumMap<Plant.LifeCycle, Set<Plant>> garden = this.garden.stream()
                .collect(groupingBy(
                        p -> p.lifeCycle, // Function<? super T, ? extends K> classifier
                        () -> new EnumMap<>(Plant.LifeCycle.class), // Supplier<M> mapFactory,
                        toSet() // Collector<? super T, A, D> downstream
                        )
                );

        System.out.println(garden);
    }
}
```

> 스트림과 EnumMap 비교

- 스트림을 사용하면 EnumMap만 사용했을 때와는 다르게 동작한다.
	- EnumMap 버전은 언제나 식물의 생애 주기당 하나씩의 중첩 맵을 만든다.
	- Stream은 해당 생애주기에 속하는 식물이 있을 때만 만든다.
	- EnumMap 버전에서는 Map을 3개 만들고, Stream 버전에서는 2개만 만든다.

- EnumMap 버전 사용 및 결과 값

```json
// Map<Plant.LifeCycle, Set<Plant>> plantByLifeCycle = new EnumMap<>(Plant.LifeCycle.class);
{
  ANNUAL=[
  ANNUAL_TREE_3,
  ANNUAL_TREE_1,
  ANNUAL_TREE_2
],
PERENNIAL=[],
BIENNIAL=[BIENNIAL_TREE_1]
}
```

- Collections.groupingBy enum 필드 값으로 그룹핑 및 결과 값

```json
// Map<Plant.LifeCycle, List<Plant>> garden = this.garden.stream().collect(groupingBy(p -> p.lifeCycle));
{
  ANNUAL=[
  ANNUAL_TREE_2,
  ANNUAL_TREE_1,
  ANNUAL_TREE_3
],
PERENNIAL=[PERENNIAL_TREE_1]
}
```

- groupingBy의 **최적화** 및 결과 값

```json
// EnumMap<Plant.LifeCycle, Set<Plant>> garden = this.garden.stream().collect(groupingBy(p -> p.lifeCycle,() -> new EnumMap<>(Plant.LifeCycle.class),toSet()));
{
  ANNUAL=[
  ANNUAL_TREE_2,
  ANNUAL_TREE_1,
  ANNUAL_TREE_3
],
BIENNIAL=[BIENNIAL_TREE_1]
}
```

## 두 열거 타입의 값들을 매핑하느라 ordinal을 (두 번이나) 쓴 배열들의 배열의 상황

- 두 개의 열거 타입을 억지로 매핑하기 위해 ordinal을 두 번이나 쓴 잘못된 방법

- 문제점
	- 컴파일러가 ordinal과 배열 인덱스의 관계를 알 수 없다.
	- 즉, Phase나 Phase.Transition 열거 타입을 수정하면서 표 TRANSITIONS 를 함께 수정하지 않거나 실수로 잘못 수정하면 런타임 오류가 발생할 것이다.
	- ArrayIndexOutOfBoundsException 이나 NullPointerException 을 던질 수도 있고, 예외없이 의도하지 않도록 동작할 수 있다.
	- 표의 크기는 상태의 가짓수가 늘어나면 제곱해서 커지며, null로 채워지는 칸도 늘어날 것이다.

```java
public enum Phase {
    SOLID,
    LIQUID,
    GAS;

    public enum Transition {
        MELT,
        FREEZE,
        BOIL,
        CONDENSE,
        SUBLIME,
        DEPOSIT;

        private static final Transition[][] TRANSITIONS = {
                {null, MELT, SUBLIME},
                {FREEZE, null, BOIL},
                {DEPOSIT, CONDENSE, null}
        };

        public static Transition from(Phase from, Phase to) {
            return TRANSITIONS[from.ordinal()][to.ordinal()];
        }
    }
}

```

## EnumMap으로 해결하기

- 전이 하나를 얻기 위해 이전 상태(from)와 이후 상태(to)가 필요
- Map 2개를 중첩하여 쉽게 해결해보기
	- 안쪽 Map은 이전 상태와 TRANSITION 을 연결
	- 바깥 Map은 이후 상태와 안쪽 Map을 연결
	- OuterMap -> 이후 상태 & InnerMap -> 이전 상태 & TRANSITION

- 전이 전후의 두 상태를 전이 열거 타입 Transition의 입력으로 받아, 이 Transition 상수들로 중첩된 EnumMap을 초기화 한다.

```java
public enum Phase {
    SOLID, LIQUID, GAS;

    public enum Transition {
        MELT(SOLID, LIQUID), FREEZE(LIQUID, SOLID),
        BOIL(LIQUID, GAS), CONDENSE(GAS, LIQUID),
        SUBLIME(SOLID, GAS), DEPOSIT(GAS, SOLID);

        private final Phase from;
        private final Phase to;

        Transition(Phase from, Phase to) {
            this.from = from;
            this.to = to;
        }

        private static final Map<Phase, Map<Phase, Transition>> m =
                Stream.of(values()) // enum 타입 두 개를 매핑한 필드 리스트
                        .collect(
                                groupingBy(
                                        t -> t.from, //
                                        () -> new EnumMap<>(Phase.class),
                                        toMap(
                                                t -> t.to,
                                                t -> t,
                                                (x, y) -> y,
                                                () -> new EnumMap<>(Phase.class)
                                        ))
                        );

        public static Transition from(Phase from, Phase to) {
            return m.get(from).get(to);
        }
    }
}
```

- 외부 Map의 타입인 Map<Phase, Map<Phase, Transition>> 의 의미
	- "이전 상태에서 '이후 상태에서 전이로의 Map' 에 대응시키는 Map"이라는 뜻
	- 이러한 Map의 Map을 초기화하기 위해 수집기(java.util.stream.Collector) 2개를 차례로 사용되었다.
	- 첫 번째 수집기인 groupingBy에서는 전이를 이전 상태를 기준으로 묶었다.
	- 두 번째 수집기인 toMap 에서는 이후 상태를 전이에 대응시키는 EnumMap을 생성한다.
		- 두 번째 수집기의 병합 함수인 (x, y) -> y는 선언만 하고 실제로는 쓰이지 않는다.
		- 이는 단지 EnumMap을 얻으려면 MapFactory가 필요하고 수집기들은 점층적 팩터리(telescoping factory)를 제공하기 때문이다.

## 새로운 상태를 추가하는 경우

- 새로운 상태인 플라즈마(PLASMA) 추가
	- 이 상태와 연결된 전이는 2가지
		- 첫 번째는 기체에서 플라즈마로 변하는 이온화(IONIZE)
		- 두 번째는 플라즈마에서 기체로 변하는 탈이온화(DEIONIZE)

### 배열로 만든 코드를 수정하는 경우

- 새로운 상수를 Phase에 1개, Phase.Transition 에 2개를 추가
- 원소 9개 짜리인 배열들의 배열을 원소 16개짜리로 교체해야 한다.

- 문제점
	- 원소 수가 너무 적거나 또는 많이 기입하거나, 잘못된 순서로 나열하는 경우 프로그램은 런타임에 문제을 일으킬 것이다.
	  (컴파일은 통과)

```java
public enum Phase {
    SOLID,
    LIQUID,
    GAS,
	PLASMA;

    public enum Transition {
        MELT,
        FREEZE,
        BOIL,
        CONDENSE,
        SUBLIME,
        DEPOSIT,
	    IONIZE,
	    DEIONIZE;

        private static final Transition[][] TRANSITIONS = {
                {null, MELT, SUBLIME, null}, // SOLID
                {FREEZE, null, BOIL, null}, // LIQUID
                {DEPOSIT, CONDENSE, null, null}, // GAS
        };

        public static Transition from(Phase from, Phase to) {
            return TRANSITIONS[from.ordinal()][to.ordinal()];
        }
    }
}
```

### EnumMap 버전으로 코드를 수정하는 경우

- 상태 목록에 PLASMA를 추가하고, 전이 목록에 IONIZE(GAS, PLASMA)와 DEIONIZE(PLASMA, GAS)만 추가하면 끝이다.

```Java
public enum Phase {
    SOLID, LIQUID, GAS,
    // 신규 PLASMA 추가
    PLASMA;

    public enum Transition {
        MELT(SOLID, LIQUID), FREEZE(LIQUID, SOLID),
        BOIL(LIQUID, GAS), CONDENSE(GAS, LIQUID),
        SUBLIME(SOLID, GAS), DEPOSIT(GAS, SOLID),
        // IONIZE, DEIONIZE 추가
        IONIZE(GAS, PLASMA), DEIONIZE(PLASMA, GAS);

        private final Phase from;
        private final Phase to;

        Transition(Phase from, Phase to) {
            this.from = from;
            this.to = to;
        }

        private static final Map<Phase, Map<Phase, Transition>> m =
                Stream.of(values()) // enum 타입 두 개를 매핑한 필드 리스트
                        .collect(groupingBy(t -> t.from, () -> new EnumMap<>(Phase.class),
                                toMap(t -> t.to, t -> t, (x, y) -> y, () -> new EnumMap<>(Phase.class))));

        public static Transition from(Phase from, Phase to) {
            return m.get(from).get(to);
        }
    }
}
```

- enum 두 개를 사용하여 데이터를 조합하여 사용하는 경우 2차원 배열을 사용하는 것보다 EnumMap을 사용하는 것이 좋다.
	- Collectors.groupingBy와 EnumMap의 조합으로 조회가 편리해지고, 성능 면에서도 이점이 있다.
	- 실제 내부에서는 Map의 Map이 배열의 배열로 구현되어 낭비되는 공간과 시간도 거의 없이 명확하고 안전하고 유지보수에 좋다.
	
## 정리

- enum에 null을 사용하는 경우 NullPointerException 을 발생시켜 문제가 발생한다.
- 여기 예제는 설명을 위해서 사용되었다.
- 배열의 인덱스를 얻기 위해 ordinal 을 쓰는 것을 일반적으로 좋지 않으니 EnumMap을 사용해야 한다.
- 다차원 관계는 EnumMap<K, EnumMap<K, V>>으로 표현하는 것이 좋다.
- Enum.ordinal()을 사용해서는 안된다. [아이템 35]() 사용하는 것은 일반 원칙의 특수한 사례이다.
