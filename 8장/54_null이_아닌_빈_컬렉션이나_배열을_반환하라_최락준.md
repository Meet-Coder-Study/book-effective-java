# 아이템 54 null이 아닌 빈 컬렉션이나 배열을 반환하라.

## null을 반환하는 경우

다음은 컬렉션이 비었을 때 null을 반환하는 메서드이다.

```java
public List<Cheese> getCheeses() {
	return cheesesInStock.isEmpty() ? null : new ArrayList<>(cheesesInStock);
}
```

위의 getCheeses 메서드가 null을 반환할 수 있으므로 항상 이를 대비하는 방어코드를 넣어줘야 한다.


## 빈 컬렉션을 반환

null 대신에 다음과 같이 빈 컬렉션을 반환할 수 있다.
```java
public List<Cheese> getCheeses() {
	return new ArrayList<>(cheesesInStock);
}
```

null보다 빈 컬렉션을 반환하는 것이 좋은 이유는

첫째, null 대신 빈 컬렉션을 반환하더라도 성능 차이가 크지 않다.

둘째, 빈 컬렉션은 굳이 새로 할당하지 않고도 반환할 수 있다.

빈 컬렉션을 새로 할당하지 않고 반환하는 방법은 빈 `불변` 컬렉션 반환을 통해 가능하다.

## 빈 불변 컬렉션 반환

```java
public List<Cheese> getCheeses() {
	return cheesesInStock.isEmpty() ? Collections.emptyList() : new ArrayList<>(cheesesInStock);
}
```

Collections를 이용하면 빈 불변 컬렉션을 반환할 수 있다.

불변이기 때문에 **공유로부터 안전**하고 매번 **새로 할당하지 않고 사용**할 수 있다.

## 배열의 경우

배열도 마찬가지이다.

빈 배열을 반환하거나, 길이 0짜리 배열을 미리 선언해두고 불변인 배열을 만들어 사용하면 된다.

## 정리

```

null이 아닌 빈 배열이나 컬렉션을 반환하라.

null은 처리하기 어렵고 성능이 좋은 것도 아니다.

```
