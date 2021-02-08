# 멤버 클래스는 되도록 static 클래스로 만들라.

## 중첩 클래스 (Nested Class) : 자신을 둘러싼 바깥 클래스에서만 쓰이는 클래스
 - 정적 멤버 클래스 (static member class)
 - 비정적 멤버 클래스
 - 익명 클래스
 - 지역 클래스
 
## 정적 멤버 클래스
 - 클래스 내부의 static 클래스
> 바깥 클래스의 private 멤버에도 바로 접근 가능. 이외에는 일반 클래스와 같다.

 
 - private 정적 멤버 클래스 : 바깥 클래스의 구성 요소를 나타낼 때
```java
public class Person {
    private String firstName;
    private String lastName;
    
    private static class Computer { // private, public
        private String name;
        private int price;

        public Computer(String name, int price) {
            this.name = name;
            this.price = price;
        }

        public int getPrice() {
            return price;
        }
    }
}
```

 - public 정적 멤버 클래스 : 바깥 클래스와 함께 쓰일 때만 유용한 public 도우미 클래스
```java
public class Calculator {
    public enum Operation1 {
        PLUS(Integer::sum),
        MINUS((x, y) -> x - y);

        private BiFunction<Integer, Integer, Integer> calculate;

        Operation1(BiFunction<Integer, Integer, Integer> calculate) {
            this.calculate = calculate;
        }

        public BiFunction<Integer, Integer, Integer> getFunction() {
            return calculate;
        }
    }


    public int Sum(int x, int y) {
        return Operation1.PLUS.getFunction().apply(x, y);
    }
}
```
```java
public static void main(String[] args) {
       Calculator c = new Calculator();
       System.out.println(c.Sum(1, 2)); // 3

       System.out.println(Calculator.Operation1.PLUS); // PLUS
   }
```

<br>
<br>

## 비정적 멤버 클래스
 - 비정적 멤버 클래스의 인스턴스는 바깥 클래스의 인스턴스와 암묵적으로 연결
 - 따라서, 비정적 멤버 클래스의 인스턴스 메서드에서 정규화된 this 를 사용하여 바깥 인스턴스의 참조를 가져올 수 있음.
 > 정규화된 this == 클래스명.this

 - 비 정적 멤버 클래스는 인스턴스를 감싸서 마치 다른 클래스처럼 보이게 하는 뷰인 어댑터에서 주로 쓰인다.
```java
public class TreeMap<K,V> extends AbstractMap<K,V> implements NavigableMap<K,V>, Cloneable, java.io.Serializable {
...
    // View class support
    class Values extends AbstractCollection<V> {
        public Iterator<V> iterator() {
            return new ValueIterator(getFirstEntry());
        }

        public int size() {
            return TreeMap.this.size();
        }

        public boolean contains(Object o) {
            return TreeMap.this.containsValue(o);
        }

        public boolean remove(Object o) {
            for (Entry<K,V> e = getFirstEntry(); e != null; e = successor(e)) {
                if (valEquals(e.getValue(), o)) {
                    deleteEntry(e);
                    return true;
                }
            }
            return false;
        }

        public void clear() {
            TreeMap.this.clear();
        }

        public Spliterator<V> spliterator() {
            return new ValueSpliterator<>(TreeMap.this, null, null, 0, -1, 0);
        }
    }    


    class EntrySet extends AbstractSet<Map.Entry<K,V>> {
        public Iterator<Map.Entry<K,V>> iterator() {
            return new EntryIterator(getFirstEntry());
        }

        public boolean contains(Object o) {
            if (!(o instanceof Map.Entry))
                return false;
            Map.Entry<?,?> entry = (Map.Entry<?,?>) o;
            Object value = entry.getValue();
            Entry<K,V> p = getEntry(entry.getKey());
            return p != null && valEquals(p.getValue(), value);
        }

        public boolean remove(Object o) {
            if (!(o instanceof Map.Entry))
                return false;
            Map.Entry<?,?> entry = (Map.Entry<?,?>) o;
            Object value = entry.getValue();
            Entry<K,V> p = getEntry(entry.getKey());
            if (p != null && valEquals(p.getValue(), value)) {
                deleteEntry(p);
                return true;
            }
            return false;
        }
    }
...
}
```
 - TreeMap.this, getFirstEntry() 는 바깥 클래스의 인스턴스가 있어야 쓸 수 있다.
 - 따라서 비정적 멤버 클래스의 인스턴스안에 관계정보가 저장되어 메모리 공간을 차지하고, 생성 시간도 더 걸린다. 
 - 또한 바깥 클래스의 인스턴스를 GC 가 회수하지 못한다.
```java
    public Set<Map.Entry<K,V>> entrySet() {
        EntrySet es = entrySet;
        return (es != null) ? es : (entrySet = new EntrySet());
    }
```
위의 코드에서 비정적 멤버 클래스 EntrySet의 객체를 생성한다. 암묵적으로 바깥 클래스와 멤버 클래스간의 연결되는 관계가 비정적 멤법 클래스의 인스턴스 안에 만들어지고,  
Map 객체가 사용하는 곳이 없더라도 이 관계때문에 GC 가 일어나지 못한다.

<br>

### 결론
#### 개념상 중첩 클래스의 인스턴스가 바깥 인스턴스와 독립적으로 존재할 수 있다면, 정적 멤버 클래스로 만들어야 한다.  
#### 멤버 클래스에서 바깥 인스턴스에 접근할 일이 없다면 무조건 static 을 붙여서 정적 멤버 클래스로 만들자.