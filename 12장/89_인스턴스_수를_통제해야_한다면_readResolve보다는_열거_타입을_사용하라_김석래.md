---
description: 인스턴스 수를 통제해야 한다면 readResolve보다는 열거 타입을 사용하라
---

# Item 89

## Intro

- 싱글턴으로 구현된 클래스는 인스턴스를 하나만 만들어지는 것을 보장한다.

## 싱글턴을 직렬화하는 경우 문제점

- 클래스에 `implements Serializable` 을 추가하는 순간 더 이상 싱글턴이 아니게 된다.
- 기본 직렬화를 쓰지 않더라도[아이템 87](), 그리고 명시적인 readObject를 제공하더라도[아이템 88]() 소용없다.
- 어떤 readObject를 사용하든 이 클래스가 초기화될 때 만들어진 인스턴스와는 별개인 인스턴스를 반환하게 된다.

> **readResolve 기능을 이용하는 경우**

- readObject가 만들어낸 인스턴스를 다른 것으로 대체할 수 있다.
- `역직렬화한 객체의 클래스`가 **readResolve 메서드**를 적절히 정의해뒀다면, `역직렬화 후 새로 생성된 객체`를 **인수**로 이 메서드가 호출
- 이 메서드가 반환한 객체 참조가 새로 생성된 객체를 대신해 반환된다.
- 대부분의 경우 이때 새로 생성된 객체의 참조는 유지하지 않으므로 바로 가비지 컬렉션 대상이 된다.

> **Serializable의 구현과 readResolve 메서드 제공**

- **싱글턴의 속성을 유지하기 위한 방법**
	- **readResolve()**
		- 역직렬화한 객체는 무시하고 클래스 초기화 때 만들어진 인스턴스를 반환
		- 이때 인스턴스의 직렬화 형태는 아무런 실 데이터를 가질 이유가 없기 때문에 모든 인스턴스 필드를 transient로 선언

- **readResolve()를 인스턴스 통제 목적으로 사용하려는 경우**
	- 객체 참조 타입 인스턴스 필드는 모두 transient로 선언해야 한다.
	- 위 조건이 충족되지 않은 경우 [아이템 88]() 내용처럼 MutablePeriod 공격과 비슷한 방식으로 readResolve() 수행되기 전에 역직렬화된 객체의 참조를 공격할 여지가 남는다.

## 역직렬화 공격 방식

> **역직렬화 공격방식**

- 싱글턴이 `Transient`가 아닌 **참조 필드**를 가지고 있다면, 그 필드 내용은 `readResolve()`가 실행되기 전에 **역직렬화** 된다.
- 그렇다면 잘 조작된 스트림을 써서 해당 참조 필드의 내용이 역직렬화되는 시점에 그 역직렬화된 인스턴스의 참조를 훔쳐올 수 있다.

> **역직렬화 공격방식을 더 상세히 보기**

- readResolve 메서드와 인스턴스 필드 하나를 포함한 `도둑(stealer) 클래스`를 작성한다.
- 해당 클래스의 인스턴스 필드는 도둑이 `숨길` 직렬화된 싱글턴을 참조하는 역할을 한다.
- 직렬화된 스트림에서 싱글턴의 비휘발성 필드를 이 도둑의 인스턴스로 교체한다.
- **이제 싱글턴은 도둑을 참조하고 도둑은 싱글턴을 참조하는 순환고리가 만들어졌다.**
- 싱글턴이 도둑을 포함하므로 싱글턴이 역질렬화될 때 도둑의 readResolve 메서드가 먼저 호출된다.
- 그 결과, 도둑의 readResolve 메서드가 수행될 때 도둑의 인스턴스 필드에는 역직렬화 도중인(그리고 readResolve가 수행되기 전인) 싱글턴의 참조가 담겨 있게 된다.
- 도둑의 readResolve 메서드는 이 인스턴스 필드가 팜조한 값을 정적 필드로 복사하여 readResolve가 끝난 후에도 참조할 수 있도록 한다.
- 그런 다음 이 메서드는 도둑이 숨긴 transient가 아닌 필드의 원래 타입에 맞는 값을 반환한다.
- 이 과정을 생략하면 직렬화 시스템이 도둑의 참조를 이 필드에 저장하려 할 때 VM이 ClassCastException을 던진다.

## 잘못된 싱글턴 예시

> **`transient`가 아닌 참조 필드를 가지고 있는 경우**

```java
public class Elvis implements Serializable {
    public static final Elvis INSTANCE = newElvis();
    private Elvis() { }

    private String[] favoriteSongs = {"Hound Dog", "Heartbreak Hotel"};

    public void printFavorites() {
        System.out.println(Arrays.toString(favoriteSongs));
    }

    private Object readResolve() {
        return INSTANCE;
    }
}
```

## non-transient 참조 필드를 훔쳐오는 도둑(stealer) 클래스

```java
public class ElvisStealer implements Serializable {
    static Elvis impersonator;
    private Elvis payload;

    private Object readResolve() {
		// resolve되기 전의 Elvis 인스턴스의 참조를 저장한다. 
        impersonator = payload;
		// favoriteSongs 필드에 맞는 타입의 객체를 반환한다. 
        return new String[]{"A Fool Such as I"};
    }

    private static final long serialVersionUID = 0;
}
```

```java
public class ElvisImpersonator {
    // 진짜 Elvis 인스턴스로는 만들어질 수 없는 바이트 스트림
	private static final byte[] serializedForm = {
            -84, -19, 0, 5, 115, 114, 0, 20, 107, 114, 46, 115,
            101, 111, 107, 46, 105, 116, 101, 109, 56, 57, 46, 69,
            108, 118, 105, 115, 98, -14, -118, -33, -113, -3, -32, 
		    70, 2, 0, 1, 91, 0, 13, 102, 97, 118, 111, 114, 105, 116, 
		    101, 83, 111, 110, 103, 115, 116, 0, 19, 91, 76, 106, 97, 
		    118, 97, 47, 108, 97, 110, 103, 47, 83, 116, 114, 105, 110, 
		    103, 59, 120, 112, 117, 114, 0, 19, 91, 76, 106, 97, 118, 
		    97, 46, 108, 97, 110, 103, 46, 83, 116, 114, 105, 110, 103, 
		    59, -83, -46, 86, -25, -23, 29, 123, 71, 2, 0, 0, 120, 112, 
		    0, 0, 0, 2, 116, 0, 9, 72, 111, 117, 110, 100, 32, 68, 111, 
		    103, 116, 0, 16, 72, 101, 97, 114, 116, 98, 114, 101, 97, 107, 
		    32, 72, 111, 116, 101, 108
    };
    public static void main(String[] args) {
        // ElvisStealer.impersonator를 초기화한 다음,
        // 진짜 Elvis(즉 Elvis.INSTANCE)를 반환한다.
        Elvis elvis = (Elvis) deserialize(serializedForm);
        Elvis impersonator = ElvisStealer.impersonator;

        elvis.printFavorites();
        impersonator.printFavorites();
    }
}
```

> **스트림을 이용해 2개의 싱글턴 인스턴스**

- 직렬화의 허점을 이용해 싱글턴 객체를 2개 생성한다.

- 설명
	- favoriteSongs 필드를 transient로 선언하여 이 문제를 고칠 수 있지만, 해당 클래스를 원소 하나짜리 열거 타입으로 바꾸는 편이 더 나은 선택이다.[아이템 3]()
	- 도둑 클래스 공격으로 보여줬듯이 readResolve 메서드를 사용해 '순간적으로' 만들어진 역질렬화된 인스턴스에 접근하지 못하게 하는 방법은 깨지기 쉽고 신경을 많이 써야 하는 작업이다.
	-

## Singleton을 보장하는 열거 타입 클래스

- 직렬화 가능한 인스턴스 통제 클래스를 열거 타입을 이용해 구현하면 선언한 상수 외의 다른 객체는 존재하지 않음을 자바가 보장해준다.
- 공격자가 AccessibleObject.setAccessible 같은 privileged 메서드를 악용하는 경우, 임의의 네이티브 코드를 수행할 수 있는 특권을 가로챈 공격자에게는 모든 방어가 무력화된다.

> **싱글턴을 보장하는 열거 타입 클래스**

```java
public enum Elvis {
    INSTANCE;
    private String[] favoriteSongs = {"Hound Dog", "Heartbreak Hotel"};

    public void printFavorites() {
        System.out.printtn(Arrays.toString(favoriteSongs));
    }
}
```

- 인스턴스 통제를 위해 readResolve를 사용하는 방식이 완전히 쓸모없는 것은 아니다.
- 직렬화 가능 인스턴스 통제 클래스를 작성해야 하는데, 컴파일 타임에는 어떤 인스턴스들이 있는지 알 수 없는 상황이라면 열거 타입으로 표현하는 것이 불가능하기 때문이다.

## readResolve 메서드의 접근성에 대한 이야기

- final 클래스인경우 readResolve 메서드는 private 접근 제한자 이어야 한다.

> **final 이 아닌 클래스의 경우 주의사항**

- **접근 제한자 설정시**
	- `private`으로 선언하면 하위 클래스에서 사용할 수 없다.
	- `package-private`으로 선언하면 같은 패키지에 속한 하위 클래스에서만 사용할 수 있다.
	- `protected`나 `public`으로 선언하면 이를 재정의하지 않은 모든 하위 클래스에서 사용할 수 있다.
	- `protected`나 `public`이면서 하위 클래스에서 재정의 하지않았다면, 하위 클래스의 인스턴스를 역직렬화하면 상위 클래스의 인스턴스를 생성하여 ClassCastException을 일으킬 수 있다.


## 핵심 정리

- 불변식을 지키기 위해 인스턴스를 통제해야 한다면 가능한 한 열거 타입을 사용한다.
- 여의치 않은 상황에서 직렬화와 인스턴스 통제가 모두 필요하다면 readResolve 메서드를 작성해 넣어야 하고, 그 클래스에서 모든 참조 타입 인스턴스 필드를 transient로 선언해야 한다.
