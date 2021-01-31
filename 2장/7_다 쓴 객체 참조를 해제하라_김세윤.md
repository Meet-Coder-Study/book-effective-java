# 아이템 7. 다 쓴 객체 참조를 해제하라.

- JVM은 GC가 메모리를 관리해줍니다.
- GC가 있다고 해서 메모리 관리를 개발자가 신경을 쓰지 않아도 된다고 오해할 수 있지만, 절대 사실이 아닙니다.
- 아래와 같이 스택 코드가 있다고 생각해봅시다.

![image](https://user-images.githubusercontent.com/53366407/104404306-b6cc4980-559d-11eb-81f3-6dfb09900d1f.png)

```java
public class Stack {
	private static final int DEFAULT_INITAL_CAPACITY = 16;

	private Obejct[] elements;
	private int size = 0;
	
	public Stack() {
		elements = new Object[DEFAULT_INITAL_CAPACITY];
	}

	public void push(Object e) {
		ensureCapacity();
		elements[size++];
	}

	public Object pop() {
		if (size == 0) {
			throw new EmptyStackException();
		}
		return elements[--size];
	}

	private void ensureCapacity() {
		if (elements.length == size) {
			elements = Arrays.copyOf(elements, 2 * size + 1);
		}
}
```

- 봤을 때 별 문제가 없다고 생각할 수도 있지만, pop()의 기능은 Stack의 특징은 LIFO 처럼 가장 최근에 들어온 요소가 반환되고 삭제되는 메서드 입니다.
- `elements[--size]` 이와 같이 반환값을 해놓는다면 실제 값은 삭제되지 않고 인덱스만 한칸씩 이동하는 것으로 메모리 누수가 발생하게 됩니다.

### 테스트

```java
import org.junit.jupiter.api.Test;

class StackTest {

    @Test
    void stackTest() {
        Stack stack = new Stack();

        for (int i = 0; i < 10; i++) {
            stack.push(i);
        }

        for (int i = 0; i < 10; i++) {
            stack.pop();
        }
    }
}
```

- 10개의 값을 넣고 10개의 값을 제거하는 로직인데, 실제 디버깅을 돌려보도록 하겠습니다. (두번쨰 for문의 i가 9일때 기준의 사진입니다.)

![image](https://user-images.githubusercontent.com/53366407/104404319-bcc22a80-559d-11eb-9628-9ee2df53367b.png)

- 즉 모든 값을 pop 했지만 실제 elements에는 값이 남아있는 걸 확인할 수 있습니다.
- GC가 해당 요소의 메모리 관리를 하겠지만 활동과 메모리 사용량이 늘어나 성능이 저하될것입니다. 즉, 디스크 페이징이나 OutOfMemoryError를 발생시킨다는 의미입니다.
- 그렇다면 현재 코드에서 메모리 누수가 발생하는 이유는 무엇일까요?
- 바로 다 쓴 참조(obsolete reference)를 여전히 가지고 있다는 것이다. 즉, elements 배열의 활성 영역 밖의 참조들이 모두 여기에 해당한다는 것입니다.
- GC는 의도치 않게 객체를 살려두는 메모리 누수를 찾는 것이 아주 까다롭다. 객체 참조 하나를 살려두면 GC는 그 객체뿐 아니라 그 객체가 참조하는 모든 객체를 회수해가지 못하고 성능에 영향을 줄 수 있습니다.

### 해결 방안

- 아주 간단하다. 해당 참조를 다 쓰면 null(참조 해제)로 초기화 해주면 된다.

```java
public Object pop() {
	if (size == 0) {
		return new EmptyStackException();
	}
	Object result = elements[--size];
	elements[size] = null;
	return result;
```

- 위와 똑같은 예제코드를 돌린 이후에 결과입니다.

![image](https://user-images.githubusercontent.com/53366407/104404326-c0ee4800-559d-11eb-9cee-6dee06b7e374.png)

- null을 사용함으로써 또 다른 이점은 해당 요소가 없는(null 처리한 참조) 메모리 공간을 사용하려고 하면 프로그램은 즉시 NPE를 발생시키며 프로그램을 종료할 것이다.

## 모든 것을 null로 처리하는 것이 좋은가?

- 모든 객체를 null로 만들면 프로그램을 필요 이상으로 지저분하게 만들 뿐이다.
- 객체 참조를 null 처리하는 일은 예외적인 경우여야 한다.
- 즉, 참조 해제의 가장 좋은 방법은 참조를 담은 변수를 유효 범위 밖으로 밀어내는 것이다.
- 이부분은 `아이템 57. 지역변수의 범위를 최소화하라`를 참고하면 알 수 있다.

## WeakHashMap & LinkedHashMap

- 메모리 누수의 주범을 알기 전에 캐시, 리스너, 콜백의 메모리 누수를 해결하기 위한 콜렉션인 약한 참조 해시맵에 대해서 간단하게 알아보도록 하겠습니다.

### 강한 참조 vs 부드러운 참조 vs 약한 참조

- 강한 참조란 `Integer prime = 1` 와 같이 일반적인 유형의 참조이다. 이때 GC의 대상이 되지 않는다.
- 부드러운 참조란 `SoftReference<Integer> soft = new SoftReference<>(prime);` 와 같이 더 이상 원본은 없고 대상을 참조하는 객체만 존재할 경우 GC대상으로 들어가도록 JVM은 동작한다.
- 약한 참조란 WeakReference<Integer> weak = new WeakReference<>(prime);와 같이 prim가 null이 되면 GC의 대상이 된다.

### WeakHashMap

- 약한 참조 해시맵으로 WeakReference의 특성을 이용해 Key에 해당하는 객체가 더이상 사용되지 않는다면 해당 객체를 자동으로 GC에 넣는다.
- HashMap 예제

```java
public static void main(String[] args) {
        Map<Integer, String> map = new HashMap<>();

        Integer key1 = 1000;
        Integer key2 = 2000;

        map.put(key1, "test a");
        map.put(key2, "test b");

        key1 = null;

        System.gc();  //강제 Garbage Collection

        map.entrySet().stream().forEach(el -> System.out.println(el));
    }
```

```java
2000=test b
1000=test a
```

```java
public static void main(String[] args) {
        Map<Integer, String> map = new WeakHashMap<>();

        Integer key1 = 1000;
        Integer key2 = 2000;

        map.put(key1, "test a");
        map.put(key2, "test b");

        key1 = null;

        System.gc();  //강제 Garbage Collection

        map.entrySet().stream().forEach(el -> System.out.println(el));
    }
```

```java
2000=test b
```

### LinkedHashMap

- 캐시에 새로운 항목이 추가될 때 removeEldestEntry 메소드를 실행하는데 이게 가장 오래된 캐시를 제거하는 것이다.

```java
public static void main(String[] args) {
        LinkedHashMap<Integer, Integer> map = new LinkedHashMap<>(1000, 0.75f, true) {

            private final static int MAX = 10;

            protected boolean removeEldestEntry(java.util.Map.Entry<Integer, Integer> eldest) {
                return size() >= MAX;
            }
        };

        for (int i = 0; i < 20; i++) {
            map.put(i, i);
        }

        for (Map.Entry<Integer, Integer> string : map.entrySet()) {
            System.out.println(string);
        }
    }
```

```java
11=11
12=12
13=13
14=14
15=15
16=16
17=17
18=18
19=19
```

## 메모리 누수의 주범

### Stack처럼 자기 메모리를 직접 관리하는 경우

- 배열로 저장소 풀을 만들어 원소를 관리하면, 활성 영역에 속한 원소들은 사용되고 비활성 영역은 쓰이지 않는데 GC는 이러한 정보를 알수가 없다.
- 따라서 null을 참조함으로써 GC에게 비활성 영역의 부분을 알려줘야 한다.

### 캐시

- 객체 참조를 캐시에 넣고 객체를 다 쓴 이후에도 그냥 두는 경우가 있다.
- 해결방법
    1. 캐시 외부에서 키(key)를 참조하는 동안만 엔트리가 살아 있는 캐시가 필요한 경우에는 WeakHashMap을 사용하자.
    2. 엔트리의 유효 기간을 정해두자.
        - 그러나 이 방법은 유효 기간을 계산하는 것이 어렵다.
    3. 쓰지 않는 엔트리를 청소하자
        - ScheduledThreadPoolExecutor와 같은 백그라운드 스레드를 활용하거나 캐시에 새 엔트리를 추가할 때 부수 작업으로 수행하는 방법을 이용하면 된다.
        - LinkedHashMap은 removeEldestEntry 메서드를 사용해 후자의 방식으로 처리한다.
- 더 복잡한 캐시를 만들기 위해서는 java.lang.ref 패키지를 직접 활용하면 된다.

### 리스너(Listener) 혹은 콜백(Callback)

- 콜백이란 이벤트가 발생하면 특정 메소드를 호출해 알려주는 것입니다.(1개)
- 리스너는 이벤트가 발생하면 연결된 리스너(핸들러)들에게 이벤트를 전달합니다.(n개)
- 클라이언트가 콜백을 등록만 하고 해지하지 않는다면 콜백은 쌓이게 될 것이다.
- 이럴 때 콜백을 약한 참조(weak reference)로 저장하면 GC가 즉시 수거해간다.
- 예를 들어 WeakHashMap에 키로 저장해두면 된다.

## 결론

- 메모리 누수를 방지하는 방법은 다쓴 객체 참조를 null로 처리하는 것과 지역변수의 범위를 최소화 하는 방법이다.
- 모든 것을 null로 처리한다고 해서 좋은 것은 아니다. 가장 좋은 방법은 지역 변수의 범위를 최소화 하는 방법이다.
- 메모리 누수의 주범은 자기 메모리를 직접 관리하는 경우, 캐시, 리스너, 콜백이다.
- 자기 메모리를 직접 관리하는 클래스라면 프로그래머는 항시 메모리 누수에 주의해야 한다.
- 메모리 누수는 철저한 코드리뷰, 힙 프로파일링 도구를 통해 디버깅을 해야 발견할 수 있기 때문에 메모리 누수를 철저히 신경써야 합니다.

## 참고자료

[Java - Collection - Map - WeakHashMap (약한 참조 해시맵) - 조금 늦은, IT 관습 넘기 (JS.Kim)](http://blog.breakingthat.com/2018/08/26/java-collection-map-weakhashmap/)
[이펙티브 자바 3판](https://www.aladin.co.kr/shop/wproduct.aspx?ItemId=171196410)
