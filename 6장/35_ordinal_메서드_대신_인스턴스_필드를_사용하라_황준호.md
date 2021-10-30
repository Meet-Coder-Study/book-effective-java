## 아이템35. ordinal 메서드 대신 인스턴스 필드를 사용하라

### `ordinal()`이란?

- enum안의 해당 상수가 몇번째인지 반환하는 메서드

```java
public enum Ensemble {
  SOLO, DUET, TRIO, QUARTET;
  
  public int numberOfMusicians() {
    return ordinal() + 1;
  }
}

// SOLO.numberOfMusicians() == 1
// DUET.numberOfMusicians() == 2
// TRIO.numberOfMusicians() == 3
```

### `ordinal()`을 쓰면 안되는 이유

- 상수 선언 순서를 바꾸는 순간 `ordinal()`을 사용하는 메서드는 오동작한다

- 이미 사용중인 정수와 값이 같은 상수는 추가할 방법이 없다

  예 : `TRIO`가 이미 있으므로 3명이 연주하는 앙상블을 나타내는 상수를 추가할 수 없다

- 값을 중간에 비워둘 수 없다

  그래서 굳이 더미 상수를 같이 추가해야 한다. -> 가동성, 실용성이 안좋다

### `ordinal()` 말고 인스턴스 필드를 사용하자

```java
public enum Ensemble {
  SOLO(1), DUET(2), TRIO(3), QUARTET(4);
  
  private final int numberOfMusicians;
  
  Ensemble(int size) {
    this.numberOfMusicians = size;
  }
  
  public int numberOfMusicians() {
    return numberOfMusicians;
  }
}

// SOLO.numberOfMusicians() == 1
// DUET.numberOfMusicians() == 2
// TRIO.numberOfMusicians() == 3
```

> 대부분 프로그래머는 이 메서드를 쓸 일이 없다. 이 메서드는 `EnumSet`과 `EnumMap` 같이 열거 타입 기반의 범용 자료구조에 쓸 목적으로 설계되었다 (`Enum`의 API 문서)

