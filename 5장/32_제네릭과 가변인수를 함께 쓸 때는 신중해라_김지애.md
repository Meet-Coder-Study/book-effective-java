# [Effective Java] item 32. 제네릭과 가변인수를 함께 쓸 때는 신중해라

## 가변인수
```java
public static void display(String ...infos) {
    for (String info: infos) {
        System.out.println(s);
    }
}
```

- 가변인수는 메서드에 넘기는 인수의 개수를 클라이언트가 조절할 수 있게 해준다.

## 가변인수와 제네릭
- `가변인수(varargs) 메서드`와 `제네릭`은 자바 5 때 함께 추가되었지만 잘 어울리지 않는다. 그 이유는 `가변인수 메서드를 호출하면 가변인수를 담기 위한 배열이 자동으로 하나 만들어지기 떄문이다.` (아이템 28에서 제네릭 배열 생성은 허용되지 않는 내용이 있었다.)
- 그 결과 `varargs 매개변수에 제네릭이나 매개변수화 타입(실체화 불가 타입)이 포함되면 알기 어려운 컴파일 오류가 발생`한다.
![item32_1](https://user-images.githubusercontent.com/37948906/107365170-c6e53380-6b1f-11eb-9a59-9ed6d1e96efb.PNG)

#### 실체화 불가 타입
-  (아이템 28에서 소개된 내용) 거의 모든 제네릭과 매개변수화 타입은 실체화 불가 타입이다. 실체화 불가 타입은 런타임에는 컴파일 타임보다 정보를 적게 담고 있다. 

#### Heap Pollution (힙 오염)
- `매개변수화 타입의 변수가 타입이 다른 객체를 참조하면 힙 오염이 발생`한다. 이렇게 다른 타입 객체를 참조하는 상황에서는 컴파일러가 자동 생성환 형변환이 실패할 수 있으니, 제네릭 타입 시스템이 약속한 타입 안전성의 근간이 흔들려버린다.

#### 제네릭과 varargs를 혼용하면 타입 안전성이 깨진다!
- 이 메서드에는 형변환하는 곳이 보이지 않는데도 인수를 건네 호출하면 ClassCastException을 던진다. 마지막 줄에 컴파일러가 생성한 (보이지 않는) 형변환이 숨어 있기 때문이다.
    - 아이템 28에 있던 코드의 응용이다. 제네릭 배열인 `List<String>[] stringLists`을 `Object[] objects`에 저장한다. 그리고 `object[0] = intList`도 런타임시에는 제네릭이 소거방식이기 때문에 `List<Integer>`가 `List`가 되고, `List<Integer>[]`는 `List[]`가 되기 때문에 허용된다. 즉, object[0]에는 Integer타입이 저장되어 있는데, 이 값을 String에 넣으려고 하면서 ClassCastException이 발생한다.
- `이처럼 타입 안전성이 깨지니 제네릭 varargs 배열 매개변수에 값을 저장하는 것은 안전하지 않다.`

#### 그런데 왜 `List<String>... stringLists`이 코드는 경고로 끝냈을까?

그 답은 제네릭이나 매개변수화 타입의 varargs 매개변수를 받는 메서드가 실무에서 매우 유용하기 때문이다. 그래서 언어 설계자는 이 모순을 수용하기로 했다. 

사실 자바 라이브러리도 이런 메서드를 여럿 제공하는데, `Arrays.asList(T... a), Collections.addAll(Collection<? super T> c, T...elements), EnumSet.of(E first, E... rest)`가 대표적이다. 다행인 점은 앞서 보여준 위험한 메서드와는 달리 이들은 타입 안전하다.

---

### @SafeVarags 애너테이션

- 자바 7 전에는 제네릭 가변인수 메서드의 작성자가 호출자 쪽에서 발생하는 경고에 대해서 해줄 수 있는 일이 없었다. 따라서 사용자는 이 경고들을 없애려면 호출하는 곳 모두 SuppressWarnings("unchecked") 애너테이션을 달아야 했다.
- 자바 7부터 `@SafeVarargs` 애너테이션이 추가되어 제네릭 가변인수 메서드 작성자가 클라이언트 측에서 발생하는 경고를 숨길 수 있게 되었다.

#### `@SafeVarargs` 애너테이션은 메서드 작성자가 그 메서드가 타입 안전함을 보장하는 장치다. 컴파일러는 이 약속을 믿고 그 메서드가 안전하지 않을 수 있다는 경고를 더 이상 하지 않는다.
    메서드가 안전한 게 확실하지 않다면 절대 @SafeVarargs 애너테이션을 달아서는 안 된다. 

#### 아래 두 조건을 만족하는 제네릭 varargs 메서드는 안전하다.
1. varargs 매개변수 배열에 아무것도 저장하지 않는다.
2. 그 배열(혹은 복제본)을 신뢰할 수 없는 코드에 노출하지 않는다.

> @SafeVarargs 애너테이션은 재정의할 수 없는 메서드에만 달아야 한다. 재정의한 메서드도 안전할지는 보장할 수 없기 때문이다. 자바 8에서 이 애너테이션은 오직 정적 메서드와 final 인스턴스 메서드에만 붙일 수 있고, 자바 9부터는 private 인스턴스 메서드에도 허용된다.

#### 안전하지 않은 가변인수 메서드 예시
```java
public class PickTwo {
    static <T> T[] toArray(T... args) {
        return args;
    }

    static <T> T[] pickTwo(T a, T b, T c) {
        switch(ThreadLocalRandom.current().nextInt(3)) {
            case 0: return toArray(a, b);
            case 1: return toArray(a, c);
            case 2: return toArray(b, c);
        }
        throw new AssertionError(); // Can't get here
    }

    public static void main(String[] args) {
        String[] attributes = pickTwo("Good", "Fast", "Cheap");
        System.out.println(Arrays.toString(attributes));
    }
}
```
- pickTwo에서는 Object[]를 반환한다. Object[]는 String[]의 하위 타입이 아니므로 pickTwo의 반환값을 attributes에 저장하기 위해 형변환하다가 ClassCastException을 던진다. 

- `제네릭 varargs 매개변수 배열에 다른 메서드가 접근하도록 허용하면 안전하지 않다`

---

### 제네릭 varargs 매개변수를 안전하게 사용하는 방법
1. @SafeVarargs로 제대로 애노테이트된 또 다른 varargs 메서드에 넘기는 것은 안전하다.
2. 그저 이 배열 내용의 일부 함수 호출만 하는 (varargs를 받지 않는) 일반 메서드에 넘기는 것도 안전하다.

##### 제네릭 varargs 매개변수를 안전하게 사용하는 예시
```java
public class TimeCapsule {
    
    @SafeVarargs
    static <T> List<T> saveCapsule(List<? extends T>... lists) {
        List<T> result = new ArrayList<>();
        for (List<? extends T> list : lists)
            result.addAll(list);
        return result;
    }

    public static void main(String[] args) {
        List<String> timeCapsule = saveCapsule(
            List.of("편지", "인형"), List.of("USB", "편지"), List.of("반지","사진"));
        System.out.println(timeCapsule);
    }
}
```
```
[편지, 인형, USB, 편지, 반지, 사진]
```

 - @SafeVarargs 애너테이션을 사용해야 할 때를 정하는 규칙은 간단하다. `제너릭이나 매개변수화 타입의 varargs 매개변수를 받는 모든 메서드에 @SafeVarargs를 달라.` 그래야 사용자를 헷갈리게 하는 컴파일러 경고를 없앨 수 있다. 
 - 즉, `안전하지 않은 varargs 메서드는 절대 작성해서는 안된다`

---

##### @SafeVarargs로 제대로 애노테이트된 또 다른 varargs 메서드에 넘기는 방식

정적 팩터리 메서드인 List.of를 활용하면 다음 코드와 같이 이 메서드에 임의 개수의 인수를 넘길 수 있다. 이렇게 사용하는 게 가능한 이유는 List.of에도 @SafeVarargs 애너테이션이 달려있기 때문이다. 이런 @SafeVarargs로 제대로 애노테이트된 List.of과 같은 메서드를 이용하는 방법도 가능하다.

</div>
</details>

<details>
<summary>List.of 메서드는 어떻게 구현되어 있을까?</summary>
<div markdown="1">

### List.of 메서드는 어떻게 구현되어 있을까?
#### List 클래스
```java
@SafeVarargs
    @SuppressWarnings("varargs")
    static <E> List<E> of(E... elements) {
        switch (elements.length) { // implicit null check of elements
            case 0:
                return ImmutableCollections.emptyList();
            case 1:
                return new ImmutableCollections.List12<>(elements[0]);
            case 2:
                return new ImmutableCollections.List12<>(elements[0], elements[1]);
            default:
                return new ImmutableCollections.ListN<>(elements);
        }
    }
```
#### ImmutableCollections 클래스
```java
@SafeVarargs
ListN(E... input) {
    // copy and check manually to avoid TOCTOU
    @SuppressWarnings("unchecked")
    E[] tmp = (E[])new Object[input.length]; // implicit nullcheck of input
    for (int i = 0; i < input.length; i++) {
        tmp[i] = Objects.requireNonNull(input[i]);
    }
    elements = tmp;
}
```
</div>
</details>

#### List.of를 이용해서 구현하기
```java
public class SafePickTwo {
    static <T> List<T> pickTwo(T a, T b, T c) {
        switch(ThreadLocalRandom.current().nextInt(3)) {
            case 0: return List.of(a, b);
            case 1: return List.of(a, c);
            case 2: return List.of(b, c);
        }
        throw new AssertionError();
    }

    public static void main(String[] args) {
        List<String> attributes = pickTwo("Good", "Fast", "Cheap");
        System.out.println(attributes);
    }
}
```

---

## 핵심 정리
- 가변인수와 제네릭은 궁합이 좋지 않다. `가변인수의 기능은 배열을 노출하여 추상화가 완벽하지 못하고, 배열과 제네릭의 타입 규칙이 서로 다르기 때문`이다. 즉, 제네릭 varargs 매개변수는 타입 안전하지 않다.
- 메서드에 제네릭 (혹은 매개변수화된) varargs 매개변수를 사용하고자 한다면, 먼저 그 메서드가 아래와 같이 타입 안전한지 확인한 다음 `@SafeVarargs` 애너테이션을 달아 사용하는데 불편함이 없게 하자.
- 메서드가 타입 안전한 경우
    1. varargs 매개변수 배열에 아무것도 저장하지 않는다.
    2. 그 배열(혹은 복제본)을 신뢰할 수 없는 코드에 노출하지 않는다.

---

### 참고 자료
- Effective Java 3/E