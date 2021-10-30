---
description: clone 재정의는 주의하여 진행하라
---

# Item 13

## 객체 복사
  
> clone()은 사용하기에 문제가 있다.
  
- clone() 메서드가 선언된 곳이 Cloneable이 아닌 Object이다.
- clone() 메서드의 접근 제한자가 protected 이다.
- Cloneable을 구현하는 것만으로는 외부 객체에서 clone()를 호출할 수 없다.

> 앞으로 이야기할 내용

- clone() 메서드를 잘 동작할 수 있는 구현 방법
- clone()을 사용하기 위한 상황
- clone() 말고 다른 방법이 있는지

### Cloneable 인터페이스의 용도

- Object의 protected 메서드인 clone의 동작 방식을 결정한다.
- Cloneable 을 구현한 클래스의 인터페이스에서 clone을 호출하면 그 객체의 필드들을 하나하나 복사한 객체를 반환한다.
- Cloneable 을 구현하지 않은 클래스에서 clone을 호출하는 경우 CloneNotSupportedException을 던진다.

### clone 메서드의 허술한 일반 규약

- `복사`의 정확한 뜻은 그 객체를 구현한 클래스에 따라 다를 수 있다.

- 일반적인 의도
	- 아래 내용에 대한 요구가 반드시 참은 아니다.
	
	```java
	// 복사본가 원본이 일치하지 않는다.
	x.clone() != x
	
	// 클래스 인스턴스 타입은 같다.
	x.clone().getClass() == x.getClass()
	```

	- 아래 일반 식도 일반적으로는 참이지만, 필수는 아니다.
			
	```java
	// 동치성
	x.clone().equals(x)
	```

	- 관례상, clone()는 super.clone()을 호출해서 반환된 객체를 반환한다.

	```java
	x.clone().getClass() == x.getClass()
	```

	- 관례상, 반환된 객체와 원본 객체는 **독립적**이다.
	- 이를 만족하려면 super.clone() 으로 얻은 객체의 필드 중 하나 이상을 반환 전에 **수정**해야 할 수 있다.

### 주의사항 1

- clone 메서드가 super.clone()이 아닌, 생성자를 호출해 얻은 인스턴스를 반환해도 컴파일러는 동일한 인스턴스로 본다.
- 이 클래스의 하위 클래스에서 super.clone()을 호출한다면 잘못된 클래스의 객체가 만들어져, 결국 하위 클래스의 clone() 메서드가 제대로 동작하지 않게 된다.
	- 상위 클래스의 clone() 값을 반환하면 안되고, 하위 클래스 타입으로 변환한 뒤 반환 해야 한다.
	- clone() 을 재정의한 클래스가 final 클래스인 경우에는 걱정할 필요가 없다.
	
- 모든 필드가 기본 타입이거나 불변 객체를 참조하는 경우 이 객체는 완벽한 상태이므로 clone()를 제공하지 않는 것이 좋다.
	- 쓸데 없는 복사를 지양한다는 관점에서 불변 클래스는 굳이 clone 메서드를 제공하지 않는 것이 좋다.

```java
public class Coffee implements Cloneable {
    @Override
    public synchronized Coffee clone() {
        Coffee clone;
        try {
            clone = (Coffee) super.clone();
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
        return clone;
    }
}
```

- 재정의한 메서드의 반환 타입은 상위 클래스의 메서드가 반환하는 타입의 하위 타입일 수 있다.
- 클라이언트가 형변환 하지 않도록 인터페이스를 제공한다.
- try-catch 블록으로 Object의 clone 메서드가 **검사 예외(checked exception)** 로 제공되는 것을 **비검사 예외(unchecked exception)** 로 처리하도록 한다. [아이템 71]() 

```text
public class ChildCoffee extends Coffee implements Cloneable {
    @Override
    public synchronized ChildCoffee clone() {
        ChildCoffee clone;
        try {
            clone = (ChildCoffee) super.clone();
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
        return clone;
    }
}
```

### 가변 객체를 참조하는 경우 예시

- clone 메서드가 단순히 super.clone() 의 결과를 그대로 받아오는 경우?
	- 반환된 Stack 인스턴스의 size 필드는 올바른 값을 갖겠지만, elements 필드는 원본 Stack 인스턴스와 똑같은 배열을 참조할 것이다.
	- 원본이나 복제본 중 하나를 수정하면 다른 하나도 수정되어 불변식을 해친다는 말이다.
	- 따라서 프로그램이 이상하게 동작하거나 NullPointerException 이 발생하게 된다.

- clone 메서드는 사실상 생성자와 같은 효과를 낸다.
	- 즉, clone은 원본 객체에 아무런 해를 끼치지 않는 동시에 복제된 객체의 불변식을 보장해야 한다.
	
```java
public class Stack implements Cloneable {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        this.elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public Object pop() {
        if (size == 0)
            throw new EmptyStackException();
        Object result = elements[--size];
        elements[size] = null; // 다 쓴 참조 해제
        return result;
    }

    private void ensureCapacity() {
        if (elements.length == size) {
            elements = Arrays.copyOf(elements, 2 * size + 1);
        }
    }

    @Override
    public Stack clone() throws CloneNotSupportedException {
        return super.clone();
    }
}
```

- 가변 객체를 갖는 클래스를 복제 하기 위해서는 내부 정보를 복사해야 한다.
	- 가장 쉬운 방법은 elements 배열의 clone을 재귀적으로 호출하는 것이다.

```java
public class Stack implements Cloneable {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        this.elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public Object pop() {
        if (size == 0)
            throw new EmptyStackException();
        Object result = elements[--size];
        elements[size] = null; // 다 쓴 참조 해제
        return result;
    }

    private void ensureCapacity() {
        if (elements.length == size) {
            elements = Arrays.copyOf(elements, 2 * size + 1);
        }
    }

    @Override
    public Stack clone() {
        try {
            Stack clone = (Stack) super.clone();
            // 배열의 clone은 런타임 타입과 컴파일 타입 모두가 원본 배열과 같은 배열을 반환한다.
	        // 따라서 배열을 복제할 때는 배열의 clone 메서드를 사용하라고 권장한다.
	        // 배열은 clone 기능을 제대로 사용하는 유일한 예이다.
            clone.elements = elements.clone();
            return clone;
        } catch(CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
}
```

### 근본적인 문제점

- Cloneable 아키텍처는 **'가변 객체를 참조하는 필드는 final로 선언하라'** 는 일반 용법과 충돌한다.
- 복제할 수 있는 클래스를 만들기 위해 일부 필드에서 final 한정자를 제거해야 할 수도 있다는 말이다.
- 배열은 원본과 같은 연결 리스트를 참조하여 원본과 복제본 모두 예기치 않게 동작할 가능성이 생긴다.
- 이를 해결하기 위해서는 각 버킷을 구성하는 연결 리스트를 복사 해야 한다.

> 연결 리스트를 복제하는 경우 주의사항

- 연겷리스트 전체를 복사하려는 경우 재귀적으로 호출하게 된다.
- 재귀 호출은 리스트의 원소 수만큼 스택 프레임을 소비하게 되기 때문에 스택 오버플로우를 일으킬 위험이 있다.
- 문제를 피하기 위해서는 재귀 호출 대신에 반복문을 사용하여 순회하는 방향으로 수정해야 한다.

## 복잡한 가변 객체를 복제하는 방법

- super.clone() 을 호출하여 얻은 **객체의 모든 필드를 초기 상태로 설정**
- 원본 **객체의 상태를 다시 생성하는 고수준 메서드들을 호출** 
- 하지만 이 방법은 저수준에서 바로 처리할 때보다는 느리다.
- Cloneable 아키텍처의 기초가 되는 필드 단위 객체 복사를 우회하기 때문에 전체 Cloneable 아키텍처와는 어울리지 않는 방식이다.

### clone() 메서드 내에서 상태 값을 수정하는 코드 작성 주의사항

- 생성자에서는 재정의될 수 있는 메서드를 호출하지 않아야 하는 것처럼 clone() 메서드도 마찬가지이다.

### clone() 메서드 재정의 시 주의사항

- clone() 메서드는 CloneNotSupportedException을 던진다고 선언되어 있지만 재정의한 메서드는 수정해야 한다.
- public clone 메서드에서는 throws 절을 없애야 한다.
- 검사 예외를 비검사예외로 수정해야 그 메서드를 사용하기에 편리하다.[아이템 71]()

### 동기화가 필요한 프로세스에서 Cloneable 구현

- Cloneable을 구현한 스레드 안전 클래스를 작성할 때는 clone 메서드 역시 적절히 동기화해줘야 한다. [아이템 78]()
- super.clone() 호출 외에 다른 할 일이 없더라도 clone을 재정의하고 동기화 해줘야 한다.

## 복제가 필요한 클래스를 설계하는 방법

- 상속용 클래스는 Cloneable을 구현하지 않는다.
- Object의 방식을 모방하여 clone()메서드를 구현해 `protected`로 두고 `CloneNotSupportedException`을 던지도록 선언할 수 있다. 
	- 이 방식은 Cloneable 구현 여부를 하위 클래스에서 선택하도록 여지를 주는 것이다.
	
- clone을 동작하지 않게 구현해놓고 하위 클래스에서 재정의하지 못하도록 할 수 있다.

```java
public class Stack implements Cloneable {
    @Override
    protected final Object clone() throws CloneNotSupportedException {
        throw new CloneNotSupportedException();
    }
}
```

## 정리

- Cloneable 인터페이스를 구현하는 모든 클래스는 clone을 재정의한다.
- 접근 제한자는 public으로 수정한다.
- 반환타입은 클래스 자신으로 수정한다.
- clone() 메서드는 가장 먼저 super.clone()을 호출한 후 필요한 필드를 적절하게 수정한다.
	- 객체의 내부에 숨어있는 모든 가변 객체를 복사하고, 복제본이 가진 객체 참조 모두가 복사된 객체들을 가리키도록 한다.
	- 내부 복사는 주로 clone을 재귀적으로 호출해 구현하지만, 이 방식이 항상 최선은 아니다.
	
- 기본 타입 필드와 불변 객체 참조만 갖는 클래스인 경우 어떤 필드도 수정할 필요가 없다.
	- 단, 일련번호나 고유 ID는 비록 기본 타입이나 불변일 지라도 수정해줘야 한다. 

> Cloneable 을 이미 구현한 클래스를 확장한다면 어쩔수 없이 clone을 잘 작동하도록 구현해야 한다.
