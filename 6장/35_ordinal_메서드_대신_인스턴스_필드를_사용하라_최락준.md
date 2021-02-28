# [item35] ordinal 메서드 대신 인스턴스 필드를 사용하라.

## ordinal 메서드란


* 모든 열거 타입이 가지고 있는 메서드
* 해당 상수가 그 열거 타입에서 몇 번째 위치 인지 반환한다.

## 문제가 뭘까?
몇번 째 위치인지를 반환한다는 시점에서 뭔가 싸늘하다.

그렇다면 왜 좋지 않은지 한번 알아 보자.


```java
public enum Ensemble {
	SOLO, DUET, TRIO, QUARTET, QUINTET,
	SEXTET, SETPTET, OCTET, NONET, DECTET;

	public int numberOfMusicians() { return ordinal() + 1 }
}
```

numberOfMusicians 메서드는 각 앙상블의 뮤지션 수를 반환한다. 하지만 다음과 같은 치명적인 문제가 있다.

* 중간에 값을 비울 수 없다.
* 같은 뮤지션 수를 같는 앙상블을 여러개 선언할 수 없다.

## 해결법

```java
public enum Ensemble {
	SOLO(1), DUET(2), TRIO(3), QUARTET(4), QUINTET(5),
	SEXTET(6), SETPTET(7), OCTET(8), NONET(9), DECTET(10);

	private final int numberOfMusicians;
	Ensemble(int size) { this.numberOfMusicians = size }
	public int numberOfMusicians() { return numberOfMusicians }
}
```
`열거 타입 상수에 연결된 값은 ordinal 메서드가 아닌 인스턴스 필드를 사용하여 저장하자.`

끝.
