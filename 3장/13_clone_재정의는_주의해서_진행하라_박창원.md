# 아이템13. clone 재정의는 주의해서 진행하라



## 일반적인 방식의 Clone 메서드 구현

- Cloneable 을 구현하는 클래스는 clone 메서드가 정확히 동작해야 한다.
- 일반적인 clone 메서드 규약은 제약이 심하지 않다. `Object` 클래스의 명세를 살펴보면 다음과 같다.

> x.clone() != x 는 참이여야 한다.
>
> x.clone().getClass() == x.getClass() 는 참이여야 한다.
>
> x.clone().equals(x) 는 참이여야 한다.

- 위의 일반적인 clone 메서드 규약은 기초자료형을 필드로 담고 있을 경우 쉽게 규약을 지킬 수 있다.
- 아래 X라는 객체는 int와 String을 필드로 갖는다. 그리고 Cloneable 인터페이스를 구현하였다.

```java
class X implements Cloneable {
    int field1;
    String field2;

    public X(int field1, String field2) {
        this.field1 = field1; this.field2 = field2;
    }

    @Override protected X clone() {
        try {
            return (X) super.clone();
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
}
```



그리고 다음 코드로 규약이 지켜지는지 확인해보자.

```java
public static void main(String[] args) throws CloneNotSupportedException {
  X x = new X(1, "X");

  boolean isCloneSame = x != x.clone();
  System.out.println(isCloneSame); // true

  boolean isTypeEqual = x.clone().getClass() == x.getClass();
  System.out.println(isTypeEqual); // true, not an absolute requirement

  boolean equals = x.clone().equals(x); // true, not an absolute requirement
  System.out.println(equals);

  X clonedX = x.clone();
  boolean isFields1Equals = clonedX.field1 == x.field1;
  boolean isFields2Equals = clonedX.field2 == (x.field2);
  System.out.println("is field 1 Equal? " + isFields1Equals);
  System.out.println("is field 2 Equal? " + isFields2Equals);
}
```

- 첫 번째 boolean 값은 true가 나온다. 
- 두 번째 boolean 값에도 true가 나온다.
- 세 번째는 false가 나온다. 왜 일까? X라는 클래스가 equals를 오버라이드하지 않았기 때문이다.
- 마지막 2개의 boolean은 X의 필드가 같은가를 확인한다. field1은 기초 자료형인 int이므로 같다. String 역시 같은 문자열이므로 JVM이 내부적으로 같은 객체를 사용하였다.



## 참조형 타입을 필드로 갖는 객체의 Cloneable 구현

- 위에서 살펴본 예제는 기초 자료형 또는 String을 필드로 갖는 경우 Cloneable을 구현한 예제이다. 
- 참조형 타입을 필드로 갖는다면, 단순히 super.clone() 을 구현해서는 안된다.

다음 *아이템 7*의 Stack 클래스의 잘못된 Cloneable 구현 예제를 살펴보자. 

```java
public class Stack_BadImplementation implements Cloneable {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack_BadImplementation() {
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
        elements[size] = null; // Eliminate obsolete reference
        return result;
    }

    private void ensureCapacity() {
        if (elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size + 1);
    }

    @Override
    protected Object clone() throws CloneNotSupportedException {
        return super.clone();
    }
}
```

이 클래스를 cloneable하게 만들고 싶다면, clone 메서드가 단순히 super.clone()을 호출하면 될 것 같다. 하지만 `size` 필드의 경우 정확히 복제 하지만, `elements` 필드는 복제 후에도 여전히 원본 Stack 인스턴스의 필드를 참조한다. **이런 레퍼런스 타입을 복제하려면 어떻게 해결해야 할까?** `clone` 메서드는 원본 객체에 내부도 카피하도록 한다.

```java

public static void main(String[] args) throws CloneNotSupportedException {

  /*
   * 메서드가 단순히 super.clone() 만을 반환한다면,
   * 복제한 필드는 원본에 있는 배열을 그대로 참조하게 된다.
   */
  Stack_BadImplementation stack = new Stack_BadImplementation();
  int e = 1;
  stack.push(e);
  e += 1;

  Stack_BadImplementation clonedStack = (Stack_BadImplementation) stack.clone();
  int clonedElement = (int) clonedStack.pop();
  clonedElement += 1; // 원본 객체의 필드 또한 수정하게 된다.

  // 둘 다 똑같이 2를 반환한다.
  System.out.println(e);
  System.out.println(clonedElement);
}
```

- 위 예제에서 출력되길 원하는 값은 1씩 나오는 것이다.
- 이러한 문제를 해결하려면 clone() 메서드를 아래와 같이 구현한다.

```java
@Override public Stack clone() {
  try {
    Stack result = (Stack) super.clone();
    result.elements = elements.clone();
    return result;
  } catch (CloneNotSupportedException e) {
    throw new AssertionError();
  }
}
```

- 위 예제는 clone 메서드를 올바르게 구현한 것이다. `super.clone()` 만 단순히 호출한 것이 아니라. 내부의 레퍼런스 타입인 `elements` 또한 clone 메서드를 호출하였다. 

- 지금처럼 clone 메서드를 재귀적으로 호출하면 완벽하게 객체 하나를 복제할 수 있을 것 같다. 하지만, 어떤 경우는 이것만으로도 불충분하다. 다음의 예제를 살펴보자.

```java
public class HashTable implements Cloneable {
  private Entry[] buckets = ...;
  private static class Entry {
    final Object key;
    Object value;
    Entry next;
    // contstructor...
  }
  
  // 잘못된 clone 메서드 예
  @Override public HashTable clone() {
    try {
      HashTable result = (HashTable) super.clone();
      result.buckets = buckets.clone();
      return result;
    } catch (CloneNotSupportedException e) {
      throw new AssertionError();
    }
  }
}
```

위 HashTable의 clone 메서드를 Stack의 clone메서드를 구현했던 것처럼 재귀적으로 clone을 호출해보았다. 이 HashTable은 원본과 같은 값을 참조하는 buckets을 갖는다. 이런 문제의 경우 다음처럼 해결할 수 있다.

```java
public class HashTable implements Cloneable {
  private Entry[] buckets = ...;
  private static class Entry {
    final Object key;
    Object value;
    Entry next;
    // constructor
  }
  Entry deepCopy() {
    return new Entry(key, value, next == null ? null : next.deepCopy());
  }
  @Override public HashTable clone() {
    try {
      HashTable result = (HashTable) super.clone();
      result.buckets = new Entry[buckets.length];
      for (int i = 0; i < buckets.length; i++) 
        if (buckets[i] != null)
          result.buckets[i] = buckets[i].deepCopy();
      return result;
    } catch (CloneNotSupportedException e) {
      throw new AssertionError();
    }
  }
}
```

배열 내부를 정확히 복제하기 위해 `deepCopy` 라는 메서드를 새로 정의하였다. 단순히 생성자를 호출하도록 한 것이다.  이 방식에도 문제점이 하나 있다. bucket의 크기가 크지 않다면 괜찮지만 너무 크다면 콜 스택 오버플로가 발생한다. 콜 스택 오버플로를 막기 위해서 다음과 같이 `deepCopy` 메서드를 수정하자. 알고리즘 풀이에서 흔히 사용되는 것 처럼 recursive를 iterative로 바꾸면 된다.

```java
Entry deepCopy() {
  Entry result = new Entry(key, value, next);
  for (Entry p = result; p.next != null; p = p.next)
    p.next = new Entry(p.next.key, p.next.value, p.next.next);
  return result;
}
```



## 그냥 정적 팩토리 메서드로 구현하세요.

-  지금까지 살펴본 clone 메서드를 구현하면서 정확히 동작하길 원하기란 어렵다.(게다가, clone 메서드는 thread safe하지 않다.)
- 가장 쉽고 효과적인 방법은 복제용 생성자나 복제용 정적 팩토리 메서드를 만드는 것이다.
- 이 예제는 Collections 이나 Map 인터페이스에서도 이미 구현된 것을 알 수 있다.
- 아래 코드는 Collections의 copy 정적 메서드의 내부 구현이다.
- 원본과 List에서 원소를 가져와 그대로 복제 대상에 넣는 것을 알 수 있다.

```java
    public static <T> void copy(List<? super T> dest, List<? extends T> src) {
        int srcSize = src.size();
        if (srcSize > dest.size())
            throw new IndexOutOfBoundsException("Source does not fit in dest");

        if (srcSize < COPY_THRESHOLD ||
            (src instanceof RandomAccess && dest instanceof RandomAccess)) {
            for (int i=0; i<srcSize; i++)
                dest.set(i, src.get(i));
        } else {
            ListIterator<? super T> di=dest.listIterator();
            ListIterator<? extends T> si=src.listIterator();
            for (int i=0; i<srcSize; i++) {
                di.next();
                di.set(si.next());
            }
        }
    }
```



