# [Effective Java] Item 24. 멤버 클래스는 되도록 static으로 만들라

## 중첩 클래스(nested class)란?

- 중첩 클래스(nested class)란?
    - 다른 클래스 안에 정의된 클래스
    - 자신을 감싼 바깥 클래스에서만 쓰여야 한다.
- 중첩 클래스의 종류
    - 정적 멤버 클래스
    - (비정적) 멤버 클래스
    - 익명 클래스
    - 지역 클래스
- 중첩 클래스를 언제 그리고 왜 사용해야 할까?

---

## 정적 멤버 클래스
- 특징
    - class 내부에서 static으로 선언된 클래스
    - 다른 클래스 안에 선언된다.
    - 바깥 클래스의 private 멤버에도 접근할 수 있다
    - 다른 점은 일반 클래스와 동일
```java
public class Developer {
    private String name;
    private String year;

    public static class Skill {
        private String type;
        private int level;
    }
}
```
```java
public class Calculator {
    public enum Operation { // 열거 타입도 암시적 static 이다.
        PLUS, MINUS, MULTIPLE, SUBTRACT
    }
}

```

---

## 비정적 멤버 클래스
- 바깥 클래스의 인스턴스와 암묵적으로 연결
- 비정적 멤버 클래스의 인스턴스 메서드에서 정규화된 this(클래스명.this)를 사용해 바깥 인스턴스의 메서드를 호출하거나 참조를 가져올 수 있다.
- 개념상 중첩 클래스의 인스턴스가 바깥 인스턴스와 독립적으로 존재할 수 있다면 정적 멤버 클래스로 만들어야 한다.
- 비정적 멤버 클래스와 바깥 인스턴스 사이의 관계는 멤버 클래스가 인스턴스화될 때 확립되며, 더이상 변경할 수 없다.
-  어댑터를 정의하는데 자주 쓰인다.
    - 어떤 클래스의 인스턴스를 감싸 마치 다른 클래스의 인스턴스처럼 보이게 하는 뷰로 사용
    - Map 인터페이스의 구현체는 아래와 같이 (keySet, entrySet, value 메서드가 반환하는) 자신의 컬렉션 뷰를 구현할 때 비정적 멤버 클래스를 사용한다.
    - 비슷하게 Set과 List같은 다른 컬렉션 인터페이스 구현들도 자신의 반복자를 구현할 때 비정적 멤버 클래스를 주로 사용한다.

#### HashMap의 KeySet 비정적 멤버 클래스
```java
public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable {
    ...
    public Set<K> keySet() {
        Set<K> ks = keySet;
        if (ks == null) {
            ks = new KeySet();
            keySet = ks;
        }
        return ks;
    }
    final class KeySet extends AbstractSet<K> {
        public final int size()                 { return size; }
        public final void clear()               { HashMap.this.clear(); }
        public final Iterator<K> iterator()     { return new KeyIterator(); }
        public final boolean contains(Object o) { return containsKey(o); }
        public final boolean remove(Object key) {
            return removeNode(hash(key), key, null, false, true) != null;
        }
        public final Spliterator<K> spliterator() {
            return new KeySpliterator<>(HashMap.this, 0, -1, 0, 0);
        }
        public final void forEach(Consumer<? super K> action) {
            Node<K,V>[] tab;
            if (action == null)
                throw new NullPointerException();
            if (size > 0 && (tab = table) != null) {
                int mc = modCount;
                for (Node<K,V> e : tab) {
                    for (; e != null; e = e.next)
                        action.accept(e.key);
                }
                if (modCount != mc)
                    throw new ConcurrentModificationException();
            }
        }
    }
}
```

#### LinkedList의 Iterator 비정적 멤버 클래스 구현
```java
public class LinkedList<E>
    extends AbstractSequentialList<E>
    implements List<E>, Deque<E>, Cloneable, java.io.Serializable
{
    ...
    public ListIterator<E> listIterator(int index) {
        checkPositionIndex(index);
        return new ListItr(index);
    }
    private class ListItr implements ListIterator<E> {
        private Node<E> lastReturned;
        private Node<E> next;
        private int nextIndex;
        private int expectedModCount = modCount;
        ...
    }
}
```

### 멤버 클레스에서 바깥 인스턴스에 접근할 일이 없다면 무조건 static을 붙여서 정적 멤버 클래스로 만들자.
- static을 생략하여 비정적 멤버 클래스로 생성할 경우 바깥 인스턴스로의 숨은 외부 참조를 갖게 된다.
    - 이런 숨은 외부참조는 추가적인 시간과 메모리 공간을 소비한다.
    - 더 심각한 문제는, 가비지 컬렉션이 바깥 클래스의 인스턴스를 수거하지 못하는 메모리 누수가 생길 수 있다.
    - 참조가 눈에 보이지 않아 문제의 원인을 찾기 어려워 때로는 심각한 상황을 초래할 수 있다.

### private 정적 멤버 클래스는 흔히 바깥 클래스가 표현하는 객체의 한 부분(구성요소)을 나타낼 때 쓴다.
- key-value를 매칭시키는 Map 인스턴스에서 키-값 쌍을 나타내는 엔트리(Entry)는 맵과 연관이 있지만 엔트리의 메서드(getKey, getValue, setValue)은 맵을 직접 사용하지 않는다.
- 따라서, 엔트리를 비정적 멤버 클래스로 표현하는 것은 낭비고, private 정적 멤버 클래스가 가장 알맞는다.
- 엔트리를 선언할 때 실수로 static을 빠뜨려도 맵은 여전히 동작하겠지만, 모든 엔트리가 바깥 맵으로의 참조를 갖게 되어 공간과 시간을 낭비할 것이다.

#### HashMap의 Map.Entry를 구현한 Node 정적 멤버 클래스
```java
public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable {
    ...
    static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        V value;
        Node<K,V> next;

        Node(int hash, K key, V value, Node<K,V> next) {
            this.hash = hash;
            this.key = key;
            this.value = value;
            this.next = next;
        }

        public final K getKey()        { return key; }
        public final V getValue()      { return value; }
        public final String toString() { return key + "=" + value; }

        ...
    }
    ...
}
```

- 멤버 클래스가 공개된 클래스의 public이나 protected 멤버라면 정적이냐 아니냐는 두 배로 중요해진다. 멤버 클래스 역시 공개 API가 되니, 혹시라도 향후 릴리스에서 static을 붙이면 하위 호환성이 깨진다.

---

## 익명 클래스
- 익명 클래스는 이름이 없다. (익명이니까!)
- 익명 클래스는 바깥 클래스의 멤버도 아니다.
- 멤버와 달리, 쓰이는 시점에 선언과 동시에 인스턴스가 만들어진다.
- 정적 문맥에서라도 상수 변수 이외의 정적 멤버는 가질 수 없다. 즉, 상수 표현을 위해 초기화된 final 기본 타입과 문자열 필드만 가질 수 있다.
- 이전에는 즉석에서 작은 함수 객체나 처리 객체를 만드는데 익명 클래스를 사용했지만 이제 람다에게 물려주었다.
- 정적 팩터리 메서드를 구현할 때도 익명 클래스를 사용한다.
- 익명 클래스의 제약 사항
    - 선언한 지점에서만 인스턴스를 만들 수 있고, instanceof 검사나 클래스의 이름이 필요한 작업은 수행할 수 없다.
    - 익명 클래스를 사용하는 클라이언트는 그 익명 클래스가 상위 타입에서 상속한 멤버 외에는 호출할 수 없다.
    - 여러 인터페이스를 구현할 수 없고, 인터페이스를 구현하는 동시에 다른 클래스를 상속할 수도 없다.
    - 익명 클래스를 사용하는 클라이언트는 그 익명 클래스가 상위 타입에서 상속한 멤버 외에는 호출할 수 없다.
    - 익명 클래스는 표현식이 중간에 등장하므로 짧지 않으면 가독성이 떨어진다.

#### 익명 클래스 예시 (Comparator 구현) - 람다가 나오고 나서는 사용하지 않는 방식
```java
List<Integer> list = Arrays.asList(10, 5, 6, 7, 1, 3, 4);
Collections.sort(list, new Comparator<Integer>() {
    @Override
    public int compare(Integer o1, Integer o2) {
        return Integer.compare(o1, o2);
    }
});
System.out.println(list);
```
(Java 8 람다가 나오고 나서는 아래와 같이 사용합니다.)
```java
Collections.sort(list, (o1, o2) -> Integer.compare(o1, o2));
```
```java
Collections.sort(list, Comparator.comparingInt(o -> o));
```

---

## 지역 클래스
- 가장 드물게 사용됨.
- 지역 변수를 선언할 수 있는 곳이면 실질적으로 어디서든 선언할 수 있고, 유효 범위도 지역 변수와 같음.
- 멤버 클래스처럼 이름이 있고 반복해서 사용할 수 있음.
- 익명 클래스처럼 비정적 문맥에서 사용될 때만 바깥 인스턴스를 참조할 수 있으며, 정적 멤버는 가질 수 없으며 가독성을 위해 짧게 작성해야 한다.

#### 지역 클래스 예시 (Thread)
```java
public class TestThread {
    public static void main(String[] args) {
        Thread goodNightThread = new Thread(() -> {
            try {
                for (int i = 0; i <10; i++){
                    Thread.sleep(1000);
                    System.out.println("양 " + i + "마리...");
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });
        goodNightThread.start();
    }
}
```

---

## 핵심 정리
- 중첩 클래스에는 네 가지(정적 멤버 클래스, 비정적 멤버 클래스, 익명 클래스, 지역 클래스)가 있으며 각각의 쓰임이 다르다.
- 메서드 밖에서도 사용해야 하거나 메서드 안에 정의하기엔 너무 길다면 멤버 클래스로 만든다.
- 멤버 클래스의 인스턴스 각각이 바깥 인스턴스를 참조한다면 비정적으로, 그렇지 않으면 정적으로 만들자.
- 중첩 클래스가 한 메서드 안에서만 쓰이면서 그 인스턴스를 생성하는 지점이 단 한 곳이고 해당 타입으로 쓰기에 적합한 클래스나 인터페이스가 이미 있다면 익명 클래스로 만들고, 그렇지 않다면 지역 클래스로 만들자.

---

### 참고 자료
- Effective Java 3/E