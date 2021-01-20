# Item 13. clone 재정의는 주의해서 진행하라

### Cloneable 인터페이스

 - Cloneable 인터페이스 ?  
 복제해도 되는 클래스임을 명시하는 믹스인 인터페이스
```
믹스인 : 다른 클래스의 부모클래스가 되지 않으면서 다른 클래스에서 사용할 수 있는 메서드를 포함하는 클래스
'상속' 이 아닌 '포함'
```   
```java
public interface Cloneable {
}
```

 - clone() 메서드 ?
```java
java.lang.Object

protected native Object clone() throws CloneNotSupportedException;
```

 - Cloneable 인터페이스는 빈 인터페이스 !  
 따라서 Cloneable 을 구현했다고 clone() 을 호출할 수 있는 것이 아니다.
 
 - clone() 메서드는 따로 오버라이딩 해줘야 한다 !
   또 clone() 은 접근제한자가 protected 라서 같은 패키지에서만 접근가능.
   
## 그럼 Cloneable 왜 씀 ?

 - clone() 은 모든 클래스에서 오버라이딩 할 수 있다.
 - 하지만 Cloneable 인터페이스를 구현하지 않은 클래스의 clone() 은 예외를 던진다 !!  
 
 - super.clone() 을 사용하지 않을 거라면, Cloneable 필요 없음.
 
 - Cloneable 을 구현한 클래스만 clone() 을 사용할 수 있다.
 - clone() 은 오버라이딩하는 메소드이다.
 - 따라서 Cloneable 은 상위 클래스 (Object) 에 정의된 clone() 의 동작방식을 변경한다는 의미이다.
 
 
## 문제점 ?

#### clone 의 **허술한** 규약
```
1. x.clone() != x // 복사한 객체는 원본 객체와 독립적임.
2. x.clone().getClass() == x.getClass(); // 복사한 객체와 원본 객체는 같은 클래스임.
3. x.clone().equals(x)
관례상, clone() 은 super.clone() 을 통해 객체를 얻어서 반환한다.

위 조건은 필수가 아닌선택
```


1. 생성자 연쇄
  - 어떤 클래스에서 clone() 을 new 키워드를 통해 복사하는 방식으로 재정의했다고 생각해보자.
  - 이 재정의는 컴파일상에 문제가 없다.
  - 하지만 그 클래스를 상속한 클래스에서 super.clone() 을 한다면, 하위 클래스의 타입이 아닌 상위 클래스의 타입 객체를 반환한다.
  
  - 클래스가 final 이면 하위 클래스가 없으니 new 를 써도 괜찮다. (하지만 Cloneable 구현이유도 없다)
  

## 해결법
클래스의 필드가 모두 불변이면 아래처럼 하면 된다.

 - super.clone() 호출
 - 클라이언트가 형변환을 할 필요없게 공변 변환 타이핑을 적용
 - 결과 반환
 
 
## 가변 객체를 참조하는 필드가 았다면 ?
가변 필드를 super.clone() 으로 복제해버리면,  
값을 복제하는 것이 아니라 주소값을 복사하여 문제가 생길 수 있다.  

clone 은 원본 객체에 아무런 영향을 끼치지 않는 동시에 복제된 객체의 불변식을 보장해야 한다.  
clone() 에서 재귀적으로 가변 객체 참조 필드를 복사하면 된다.  

그런데 만약 가변 참조 필드가 final 이라면 새로운 값을 참조할 수 없으므로 위의 방식을 적용할 수 없다.  
**가변 객체를 참조하는 필드는 final 로 선언하라** 라는 용법과 충돌하게 된다.

따라서 Cloneable 을 구현하면 final 을 때버려야 하는 상황이 올 수 있다.


## clone() 재귀 호출이 부족하다면 Deep copy 를 하라
만약 배열 안에 가변 참조 객체가 있다면?

재귀적인 clone() 호출으로는 부족하다.

그럴때는 Deep copy 를 한다.
```java
    public Entry deepCopy() {
        return new Entry(key, value, next == null ? null : next.deepCopy());
    }
```
```java
    @Override
    public CustomHashTable clone() {
        try {
            CustomHashTable result = (CustomHashTable) super.clone();
            result.buckets = new Entry[buckets.length];

            for (int i = 0; i < buckets.length; i++) {
                if (buckets[i] != null)
                    result.buckets[i] = buckets[i].deepCopy();
            }

            return result;
        } catch (final CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
```

이 방식은 원소 수만큼 연결리스트를 만든다.  
따라서 원소 수만큼의 스택 프레임을 소비하고, 스택 오버플로를 일으킬 수 있다.  

그럴땐 반복자를 이용하여 구현할 수 있다.
```java
    public Entry deepCopy() {
        Entry result = new Entry(key, value, next);
        for (Entry p = result; p.next != null; p = p.next) {
            p.next = new Entry(p.next.key, p.next.value, p.next.next);
        }
        return result;
    }
```

## 고수준 API 를 이용한 가변객체 복제
 - Collection 의 put, add 같은 API 를 이용하여 요소를 복제
 - 상대적으로 느리다.

## clone()에서는 재정의 될 수 있는 메서드를 호출하면 안된다.
 - 하위 클래스에서 재정의한 메서드를 쓸 수 없게 된다.
 
## clone() 를 재정의 할 때에는 public 접근자, throw 던지지 않기
 - public 이어야 사용하기가 쉽다.
 - throw 를 던지는 것을 계속 정의하면 하위 클래스에도 계속 try-catch 를 써줘야 한다.
 
## 요약
 1. Cloneable 을 구현하면, clone() 을 재정의해야 한다.
 2. 접근자는 public, 반환형은 자기자신의 타입으로 변경한다.
 3. 공변 변환 타이핑을 통해 자기자신의 타입을 반환한다.
 4. super.clone() 으로 복제 한다.
 5. 깊은 복제 방식으로 가변 참조 객체를 복사한다.
 
## 결론 : 복제는 그냥 복사 생성자와 복사 팩터리를 써라
배열만이 clone() 을 재대로 사용하는 유일한 예이다.
복사하는 대상이 배열이 아니면 복사 생성자, 복사 팩터리가 좋다.

복사 생성자
```java
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack(Stack s) {
        this.elements = s.elements.clone();
        this.size = s.size;
    }
}
```
기본 생성자가 필요하면 추가하기


복사 팩터리
```java
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public static Stack newInstance(Stack s) {
        return new Stack(s.elements, s.size);
    }
}
```

