# item 37. ordinal 인덱싱 대신 EnumMap을 사용하라

# 🚨 ordinal()

> 해당 상수가 그 열거 타입에서 몇 번째 위치인지를 반환하는 메소드 
item 35

# 👀 예제코드

```sql
장바구니에 담긴 과자들을 타입(봉지과자, 박스과자, 통과자) 별로 분리하는 프로그램을 작성해보자!
```

```java
public class Snack {
  enum Type { BOX, BAG, BARREL }

  final String name;
  final Type type;

  Snack(String name, Type type) {
    this.name = name;
    this.type = type;
  }

  @Override
  public String toString() {
    return "Snack{" +
        "name='" + name + '\'' +
        '}';
  }
}
```

## (1) ordinal 인덱싱

```java
public class ShoppingBasket {
  public static void main(String[] args) {
    // 과자의 타입마다 분리하기 위해 Set 배열을 만들고, 초기 사이즈를 과자의 총 타입의 갯수로 지정한다.
    Set<Snack>[] snacksByType = (Set<Snack>[]) new Set[Type.values().length];

    // 초기화를 해준다.
    for (int i = 0 ; i < snacksByType.length ; i++) {
      snacksByType[i] = new HashSet<>();
    }

    // 장바구니에 과자를 담는다.
    Snack[] shoppingBasket = {
        new Snack("오레오오즈", Type.BOX),
        new Snack("포카칩", Type.BAG),
        new Snack("바나나킥", Type.BAG),
        new Snack("닭다리스낵", Type.BOX),
        new Snack("콘칲", Type.BAG)
    };

    // 장바구니에 담은 과자들을 타입별로 분리한다.
    for (Snack snack : shoppingBasket) {
      snacksByType[snack.type.ordinal()].add(snack); // 이 때, ordinal()을 사용한다. (BOX = 0, BAG = 1)
    }

    // 결과를 출력한다.
    for (int i = 0; i < snacksByType.length; i++) {
      System.out.printf("%s: %s%n", Type.values()[i], snacksByType[i]);
    }
  }

}
```

![Untitled](https://user-images.githubusercontent.com/42836576/108148538-5e481900-7114-11eb-86af-25404960f4da.png)

## 문제점

- 배열은 제네릭과 호환되지 않기 때문에 비검사 형변환이 수행되고, 컴파일이 안된다.
- 초기화를 할 때, 정수 값을 잘못 입력하면 `ArrayIndexOutOfBoundException`이 발생한다.
- `ordinal()`은 상수 선언 순서에 따라 변한다.

## (2) EnumMap 사용

### EnumMap이란?

Map 인터페이스에서 **키를 특정 열거형 타입만을 사용**하도록 하는 구현체

```java
public class ShoppingBasket {
  public static void main(String[] args) {
    // 과자의 타입마다 분리하기 위해 EnumMa을 사용한다.
    Map<Snack.Type, Set<Snack>> snacksByType = new EnumMap<>(Snack.Type.class);

    // 초기화를 해준다.
    for (Snack.Type type : Snack.Type.values()) {
      snacksByType.put(type, new HashSet<>());
    }

    // 장바구니에 과자를 담는다.
    Snack[] shoppingBasket = {
        new Snack("오레오오즈", Type.BOX),
        new Snack("포카칩", Type.BAG),
        new Snack("바나나킥", Type.BAG),
        new Snack("닭다리스낵", Type.BOX),
        new Snack("콘칲", Type.BAG)
    };

    // 장바구니에 담은 과자들을 타입별로 분리한다.
    for (Snack snack : shoppingBasket) {
      snacksByType.get(snack.type).add(snack);
    }

    // 결과를 출력한다.
    System.out.println(snacksByType);
  }
}
```

![Untitled 1](https://user-images.githubusercontent.com/42836576/108148542-5f794600-7114-11eb-9c6f-fed6a78b27fd.png)

## 그 전 예제코드랑 비교하면

- 코드가 더 간결해지고 성능도 비슷하다.
- 배열 인덱스를 계산하는 과정에서 오류가 날 일이 없고, **타입에 안전**하게 사용할 수 있다.
- **출력용 문자열**을 제공해주어, 결과를 확인할 때 따로 formatting을 해주지 않아도 된다.

## 3. Stream 사용

위 코드를 Stream을 사용해서 조금 더 단순하게 바꿀 수 있다. 

### 3-1. EnumMap을 사용하지 않는 경우

```java
public class ShoppingBasket {
  public static void main(String[] args) {
    // 장바구니에 과자를 담는다.
    Snack[] shoppingBasket = {
        new Snack("오레오오즈", Type.BOX),
        new Snack("포카칩", Type.BAG),
        new Snack("바나나킥", Type.BAG),
        new Snack("닭다리스낵", Type.BOX),
        new Snack("콘칲", Type.BAG)
    };

    // 장바구니에 담은 과자들을 타입별로 분리한 결과를 출력한다.
    System.out.println(Arrays.stream(shoppingBasket)
        .collect(groupingBy(s -> s.type)));
  }
}
```

![Untitled 2](https://user-images.githubusercontent.com/42836576/108148545-61430980-7114-11eb-853a-4051875b4fa6.png)

EnumMap을 사용할 때 얻는 **공간과 성능 이점이 사라진다**는 문제가 있으니 주의해서 사용해야한다. 

### 3-2. EnumMap을 이용해 데이터와 열거 타입을 매핑하는 경우

```java
public class ShoppingBasket {
  public static void main(String[] args) {
    // 장바구니에 과자를 담는다.
    Snack[] shoppingBasket = {
        new Snack("오레오오즈", Type.BOX),
        new Snack("포카칩", Type.BAG),
        new Snack("바나나킥", Type.BAG),
        new Snack("닭다리스낵", Type.BOX),
        new Snack("콘칲", Type.BAG)
    };

    // 장바구니에 담은 과자들을 타입별로 분리한 결과를 출력한다.
    System.out.println(Arrays.stream(shoppingBasket)
        .collect(groupingBy(s -> s.type, () -> new EnumMap<>(Type.class), toSet())));
  }
}
```

![Untitled 3](https://user-images.githubusercontent.com/42836576/108148549-630ccd00-7114-11eb-8170-96aa0046262f.png)

EnumMap 버전은 언제나 열거 타입당 하나씩 중첩 맵을 만들지만, Stream에서 사용하면 **해당 열거 타입에 속하는 객체가 있을 때만** 만든다.

# ❤ 핵심 정리

배열의 인덱스를 얻기 위해 `ordinal()`을 사용하는 것 보다는 `EnumMap`을 사용하는 것이 좋다.