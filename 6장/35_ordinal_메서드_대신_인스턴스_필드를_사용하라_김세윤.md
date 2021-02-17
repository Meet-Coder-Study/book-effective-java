# 아이템35. ordinal 메서드 대신 인스턴스 필드를 사용하라

## 서론

- 대부분의 열거 타입 상수는 자연스럽게 인덱스(하나의 정수값)을 갖게 된다.
- 따라서 Java enum은 ordinal 이라는 메서드를 제공해 인덱스를 반환해준다.
- 그러나 ordinal은 사용하면 안된다. 그 이유를 살펴보도록 하겠습니다.

## 잘못 사용한 예

```java
pbulic enum Number {
	ONE, TWO, THREE, FOUR

	pbulic int convertInt() {
		return ordinal() + 1;
	}
}
```

- 지금과 같이 Number라는 enum이 있고, enum의 Int형을 ordinal로 정의한다고 생각해봅시다.
- 이떄, 갑자기 Zero(0)이 필요하다고 생각해봅시다.
- 그렇다면 아래와 같은 코드가 될것 입니다.

```java
pbulic enum Number {
	ZERO, ONE, TWO, THREE, FOUR

	pbulic int convertInt() {
		return ordinal() + 1;
	}
}
```

- convertInt() 메서드를 실행해보면 ZERO는 1, ONE은 2를 반환할 것이다.
- 그렇다면 `+1` 을 빼면 되는거 아닌가 라는 생각을 가질수도 있습니다.
- 이 부분은 아래에 JPA Tip에서 다시 다루도록 하겠습니다.
- 또한 FIVE를 패스하고 SIX만 필요해서 추가한다고 생각해봅시다.

```java
pbulic enum Number {
	ZERO, ONE, TWO, THREE, FOUR, SIX

	pbulic int convertInt() {
		return ordinal() + 1;
	}
}
```

- 그럼 이때, SIX.convertInt()는 5를 반환하게 될것입니다.
- 해결법이 있긴합니다. DUMMY 값으로 FIVE를 추가해놓는거죠.
- 그러나 이것이 좋은 코드라고 말할 수 있을까요? 사용하지 않는 상수를 선언하는게 말이죠.

## 해결법

- 해결법은 아주 간단합니다. ordinal 메서드를 사용하지 않고 필드로 모두 선언해놓는 것입니다.

```java
pbulic enum Number {
	ONE(1), TWO(2), THREE(3), FOUR(4)
	
	private final int number;

	public Number(int number) {
		this.number = number;
	}

	public int getNumber() {
		return number;
	}
}
```

- 위에서 문제점이였던 0을 추가해도 0만 추가해주면 되기 때문에 확장성도 좋습니다. 또한 5를 제외하고 6을 추가한다고 해도 5를 추가하지 않고 6을 추가할 수 있기 때문에 훨씬 좋은 코드가 될 것입니다.

```java
pbulic enum Number {
	ZERO(0), ONE(1), TWO(2), THREE(3), FOUR(4), SIX(6)
	
	private final int number;

	public Number(int number) {
		this.number = number;
	}

	public int getNumber() {
		return number;
	}
}
```

## JavaDoc

- Java11의 ordinal 메소드의 JavaDoc을 보면 아래와 같이 써있습니다.

```java
Returns the ordinal of this enumeration constant (its position in its enum declaration, where the initial constant is assigned an ordinal of zero). Most programmers will have no use for this method. It is designed for use by sophisticated enum-based data structures, such as EnumSet and EnumMap.
```

- 간단하게 말하자면 "대부분 프로그래머는 이 메서드를 사용할 일이 없다. 이 메서드는 EnumSet과 EnumMap 같이 열거 타입 기반의 범용 자료구조에 쓸 목적으로 설계되었다" 라는 말입니다.

## JPA Tip

- JPA는 Enum 필드도 테이블 컬럼과 매핑해줍니다.
- 그때 사용하는 어노테이션이 `@Enumerated` 입니다.
- 여기서 우리는 EnumType을 설정해줘야 하는데, ORDINAL과 STRING이 있습니다.
- ORDINAL 타입은 방금 봤던 인덱스 기반의 값이며, STRING은 name() 메서드라고 생각하면 됩니다.
- 첫번째 문제점에서의 ZERO를 추가한 이후 그 도메인을 DB에서 불러올 경우 원래 ONE의 의미로 0을 저장하고 있던 필드는 ZERO가 나오게 될것입니다.
- 그렇다면 원래 있던 데이터들이 모두 꼬이게 될 것입니다.
- 그러기 때문에 JPA에서 `@Enumerated` 의 ORDINAL도 개인적으로 절대 사용하면 안됩니다.(김영한님도 사용하지 말라고 했습니다 😎)

## 결론

- ordinal 절대 쓰지 마세요. 🚫
