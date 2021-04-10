# [아이템 89. 인스턴스 수를 통제해야 한다면 readResolve보다는 열거 타입을 사용하라](https://github.com/Meet-Coder-Study/book-effective-java/issues/89)



아이템 3의 싱글톤 패턴(Singleton pattern)을 보면, 싱글톤 패턴 클래스의 생성자는 오로지 하나의 인스턴스만 생성하도록 보장한다.

```java
public class Elvis {
  public static final Elvis INSTANCE = new Elvis();
  private Elvis() { ... }
  public void leavingTheBuilding() {...}
}
```



만약 이 클래스에 Serializable을 구현한다면 어떻게 될까?

다음 코드를 살펴보고 결과를 예상해보자.

```java

public class item89 {
	public static void main(String[] args) throws IOException, ClassNotFoundException {
		Elvis elvis = Elvis.getInstance();

		byte[] serialized;
		// 직렬화
		try (ByteArrayOutputStream baos = new ByteArrayOutputStream()) {
			try (ObjectOutputStream oos = new ObjectOutputStream(baos)) {
				oos.writeObject(elvis);
				serialized = baos.toByteArray();
			}
		}

		// 역직렬
		Elvis deserializedElvis;
		try (ByteArrayInputStream bais = new ByteArrayInputStream(serialized)) {
			try (ObjectInputStream ois = new ObjectInputStream(bais)) {
				 deserializedElvis = (Elvis) ois.readObject();
			}
		}
    
		System.out.println(elvis == deserializedElvis);
		System.out.println(format("before: %s, after: %s", elvis, deserializedElvis));
	}
}
```



정답은 두 인스턴스는 같지 않다는 것이다. `readObject` 메소드로 인해 인스턴스가 생성된다.

>false
>before: me.cwpark.chapter12.Elvis@4fccd51b, after: me.cwpark.chapter12.Elvis@6615435c

**Serializable을 구현하면 더 이상 이 클래스는 싱글톤이 아니다.**



만약 `Serializable`을 반드시 구현해야하며, 그 인스턴스는 반드시 싱글톤이여야만 한다면 어떻게 대처 해야할까?

**readResolve를 사용하면 인스턴스를 내가 원하는 것으로 바꿀 수 있다.**

즉, `readResolve()` 메서드는 역직렬화 후에 인스턴스를 생성하기 위해 호출된다.



그렇다면, 역직렬화를 통해 새로 생성된 객체는 어떻게 되는걸까? 

새로 생성된 객체는 Garbage Collector의 GC대상이 된다.



코드를 통해 확인해보자.

먼저, 앞서 작성한 Elvis 클래스에  `readResolve()` 메서드를 구현한다

```java
public class Elvis implements Serializable {

	private static final Elvis INSTANCE = new Elvis();

	private Elvis() {}

	public static Elvis getInstance() { return INSTANCE; }

	private Object readResolve() {
		return INSTANCE;
	}
}
```

다시 우리가 작성한 item89 클래스의 main() 메서드를 실행해보자.

> true
> before: me.cwpark.chapter12.Elvis@4fccd51b, after: me.cwpark.chapter12.Elvis@4fccd51b

똑같은 객체가 성공적으로 반환되므로, 직렬화를 하면서 동시에 싱글톤 패턴을 유지할 수 있다.



'*아이템 88. readObject 메서드는 방어적으로 작성하라*' 에서 설명하듯이, `readObject` 메서드는 생성자와 같은 수준으로 인식하고 작성해야 함을 말해준다.

이와 비슷하게 `readResolve` 메서드 역시 역직렬화 시점에 필드가 nontransient 레퍼런스 타입이라면 다른 객체로 바뀔 위험이 있다.



이러한 공격을 방지하기 위해 레퍼런스 필드는 `transient` 로 선언해야한다.

하지만 더 간결한 방법이 있다.

Elvis를 enum 타입으로 만드는 것이다.



Enum 타입과 같이 인스턴스가 통제된 직렬화 가능한 클래스를 작성하면 선언된 객체 이외에는 어떠한 인스턴스도 있지 않음을 보장한다.

(단, 리플렉션의 `AccessibleObject.setAccessible`  이용한 방법은 제외)

```java
public enum Elvis {
  INSTANCE;
  private String[] referenceField = {"One","Two", "Tre"};
  public String[] getReferenceField() { ... }
}
```



**결론**

직렬화가 필요하며 인스턴스 수가 하나임을 보장하고 싶을 때, enum 타입을 가장 먼저 고려하라.

만약 enum 타입을 사용하는 것이 불가능하며, 인스턴스 수를 제한해야 한다면, `readResolve` 를 구현해야 한다.

또한, 해당 클래스의 모든 필드는 원시타입(primitive) 혹은 임시타입(transient)으로 정의되어야 한다.













