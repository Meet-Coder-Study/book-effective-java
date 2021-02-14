# item 29. 이왕이면 제네릭 타입으로 만들라.

 - 제네릭이 없는 Stack
 ```java
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public Object pop() {
        if (size == 0) {
            throw new EmptyStackException();
        }

        Object result = elements[--size];
        elements[size] = null;
        return result;
    }
}
 ```


```java
    @DisplayName("Object 스택은 문제가 있다!")
    @Test
    public void StackTest() {
        Stack stack = new Stack();

        stack.push(1);
        stack.push("1");

        assertEquals(stack.pop().getClass(), String.class);
        assertEquals(stack.pop().getClass(), String.class); // Integer !!
    }
 ```
<br>
 - 형변환을 클라이언트에서 해줘야 한다 !

## 제네릭 타입의 Stack 으로 만들기

1. 클래스 선언 타입 매개변수 추가하기.
1. Object 를 타입 매개변수로 바꾸기.


```java
public class GenericStack<E> {
    private E[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;
    
    public GenericStack() {
        elements = new E[DEFAULT_INITIAL_CAPACITY]; // 에러 발생!
        // 실체화 불가 타입으로는 배열을 만들 수 없음.
    }

    public void push(E e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public E pop() {
        if (size == 0) {
            throw new EmptyStackException();
        }

        E result = elements[--size];
        elements[size] = null;
        return result;
    }
}
 ```
 - 제네릭은 실체화 불가 타입.
 - 런타임 정보가 컴파일타임 정보보다 적다.
 - 배열 생성 불가.
 
 

## 첫번째 방법 : Object 배열로 생성하고 E 배열로 타입 캐스팅

 ![이미지](https://github.com/cmg1411/effectiveJava/blob/master/src/main/java/Chapter5/Day29/uncheckedException.png)
  - elements 는 private
  - E[] 로 선언되어 push(E) 만 담아 타입 캐스팅이 안전 !
  - @SuppressedWarning("unchecked") 붙이기.
  
  
## 두번째 방법 : 그냥 Object 배열로 선언하고 생성하기

 ![이미지](https://github.com/cmg1411/effectiveJava/blob/master/src/main/java/Chapter5/Day29/uncheckedException2.png)
 - push 에서 E 타입만 넣을 수 있으므로, elements 는 E 로 무조건 타입변환이 가능.
 - @SuppressedWarning("unchecked") 붙이기.
 
 
## 첫번째 방법의 장점
 - 첫번쨰 방법은 elements 배열을 쓰는 곳 여러군데가 있더라도 타입 캐스팅을 생성자에서 한번만 함.
 - (두번째 방법은 매번 타입 캐스팅을 해줘야 함.)
 - 첫번쨰 방법이 가독성이 좋음.
 
 
## 첫번쨰 방법의 단점
 - 힙 오염이 생길 수 있음
 - 만약 Stack 의 push 가 아래와 같이 구현되었다면 ?
```java
    private E[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    @SuppressWarnings("unchecked")
    public GenericStack() {
        elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e) {
        ensureCapacity();
        elements[size++] = (E) e;
    }
```
```java
    public E pop() {
        if (size == 0) {
            throw new EmptyStackException();
        }

        E result = elements[--size];
        elements[size] = null;
        return result;
    }
```
```java
    public static void main(String[] args) {
        GenericStack<String> s = new GenericStack<>();
        
        s.push("Stack");
        s.push(1);
        
        s.pop();
        s.pop();
    }
```
## 힙 오염 (Heap Pollution)
 - 매개변수화 타입이 매개변수화 타입이 아닌 것을 참조할 떄 발생하는 현상.
 - 매개변수화 타입은 타입 소거가 이루어진다.
 - 따라서 런타임 시점에는 타입 정보를 모른다.
 - 하지만 매개변수화 타입이 아닌 배열은 런타임 시점에 타입 정보를 안다.
 - 따라서 컴파일타임과 런타임의 정보가 달라서, UncheckedWarning 과 ClassCastException 이 발생.
 
## 힙 오염이 발생하는 경우
1. raw 타입과 매개변수화 타입을 같이 사용하는 경우
```java
List ln = new ArrayList<Number>();
List ls = new LinkedList<String>();
List<String> list;
list = ln; // unchecked warning + heap pollution
list = ls; // unchecked warning + NO heap pollution
```
2. unchecked 캐스팅을 하는 경우
```java
List<? extends Number> ln = new ArrayList<Long>();
List<Short> ls = (List<Short>) ln; // unchecked warning + heap pollution
List<Long>  ll = (List<Long>)  ln; // unchecked warning + NO heap pollution
```
## 타입 소거
|매개변수화 타입	|소거 후|
|---------|-----|
|```List<String>```	|	List|
|```Map.Entry<String,Long>```	|	Map.Entry|
|```Pair<Long,Long>[]```	|	Pair[]|
|```Comparable<? super Number>```	|	Comparable|

|타입 파라미터	|소거 후|
|---------|-----|
|```<T>```	|Object|
|```<T extends Number>```	|Number|
|```<T extends Comparable<T>>```	|Comparable|
|```<T extends Cloneable & Comparable<T>>```	|Cloneable|
|```<T extends Object & Comparable<T>>```	|Object|
|```<S, T extends S>```	|Object,Object|


|매개변수화 메서드	|소거 후|
|---------|-----|
|```Iterator<E> iterator()	```	|Iterator iterator()|
|```<T> T[] toArray(T[] a) 	```	|Object[] toArray(Object[] a)|
|```<U> AtomicLongFieldUpdater<U> newUpdater(Class<U> tclass, String fieldName)	```	|AtomicLongFieldUpdater newUpdater(Class tclass,String fieldName)|

[힙 오염](http://www.angelikalanger.com/GenericsFAQ/FAQSections/TechnicalDetails.html#Topic2)




## 진행하면서..
 - item 28 의 주제 (배열보다 리스트를 사용하라) 와 달리 이번 장에서는 배열을 사용했다.
 - 제네릭 타입 안에서 리스트를 사용하는 것이 항상 가능하지도, 더 좋은것도 아니다.
 
 
## 결론
```java
    public static void main(String[] args) {
        GenericStack<String> stack = new GenericStack<>();

        for (String s : List.of("hi", "hello", "bye")) {
            stack.push(s);
        }

        while (!stack.isEmpty()) {
            System.out.println(stack.pop().toUpperCase());
        }
    }
```
 - 클라이언트에게 영향을 주지 않는다.
 - stack.pop() 에서 명시적인 형변환이 필요 없다.
 - 제네릭 타입으로 만들자.