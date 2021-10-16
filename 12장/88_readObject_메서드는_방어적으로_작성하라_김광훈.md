# Item 88. readObject 메서드는 방어적으로 작성하라.
## 1. 핵심 정리
- readObject 메서드를 작성할 때는 언제나 public 생성자를 작성하는 자세로 임해야 한다.
- readObject 는 어떤 바이트 스트림이 넘어오더라도 유효한 인스턴스를 만들어내야 한다.
- 바이트 스트림이 진짜 직렬화된 인스턴스라고 가정해서는 안 된다.
- readObject 메서드를 작성하는 지침
```
(1) private 이어야 하는 객체 참조 필드는 각 필드가 가리키는 객체를 방어적으로 복사하라 
(2) 모든 불별식을 검사하여 어긋나는 게 발견되면 InvalidObjectException 을 던진다.
(3) 역직렬화 후 객체 그래프 전체의 유효성을 검사해야 한다면 ObjectInputValidation 인터페이스를 사용해라
(4) 직접적이든 간접적이든, 재정의할 수 있는 메서드는 호출하지 말자 
```  

## 2. 불편클래스의 직렬화 문제점 
#### 2.1 Item 50. 복습
- "Item 50. 적시에 방어적 복사본을 만들라" 에서는 불변인 날짜 범위 클래스를 만드는데 있어 가변인 Date 필드를 이용했다.
- 그래서 불변식을 지키고 불변을 유지하기 위해 생성자와 접근자에서 Date 객체를 방어적으로 복사하느라 코드가 길어졌다.
- Example
```java
public final class Period {
    private final Date start;
    private final Date end;

    /**
     * @param  start 시작 시각
     * @param  end 종료 시각; 시작 시각보다 뒤여야 한다.
     * @throws IllegalArgumentException 시작 시각이 종료 시각보다 늦을 때 발생한다.
     * @throws NullPointerException start나 end가 null이면 발생한다.
     */
    public Period(Date start, Date end) {
        this.start = new Date(start.getTime()); // 가변인 Date 클래스의 위험을 막기 위해 새로운 객체로 방어적 복사를 한다.
        this.end = new Date(end.getTime());

        if (this.start.compareTo(this.end) > 0) {
            throw new IllegalArgumentException(start + " after " + end);
        }
    }

    public Date start() { return new Date(start.getTime()); }
    public Date end() { return new Date(end.getTime()); }
    public String toString() { return start + " - " + end; }
    // ... 나머지 코드는 생략
}
```

#### 2.2 직렬화를 하기 위해서 과연 implements Serializable 만 추가하면 될까?
- Period 클래스는 물리적 표현과 논리적 표현이 같기 때문에 기본 직렬화 형태를 사용해도 무방해보인다.
- 하지만 실제로는 불변식을 보장하지 못한다. 
- readObject 가 또 다른 public 생성자이기 때문에 인수가 유효한지 검증해야하고 필요하다면 매개변수를 방어적으로 복사까지 해야한다.


## 3. readObject
#### 3.1 readObject 란?
- 매개변수로 바이트 스트림을 받는 생서자로 정의할 수 있다.
- 보통 바이트스트림은 정상적으로 생성된 인스턴스를 직렬화해서 만들어 진다.
- 반대로 말하면, 정상적으로 생성되지 않은 바이트스트림을 받으면 문제가 발생한다.

#### 3.2 불변식을 보장 받지 못하는 사례 
- Example 
```java
public class BogusPeriod {
    
    // 정상적이지 않은 바이트스트림
    private static final byte[] serializedForm = {
        (byte)0xac, (byte)0xed, 0x00, 0x05, 0x73, 0x72, 0x00, 0x06,
        0x50, 0x65, 0x72, 0x69, 0x6f, 0x64, 0x40, 0x7e, (byte)0xf8,
        0x2b, 0x4f, 0x46, (byte)0xc0, (byte)0xf4, 0x02, 0x00, 0x02,
        0x4c, 0x00, 0x03, 0x65, 0x6e, 0x64, 0x74, 0x00, 0x10, 0x4c,
        0x6a, 0x61, 0x76, 0x61, 0x2f, 0x75, 0x74, 0x69, 0x6c, 0x2f,
        0x44, 0x61, 0x74, 0x65, 0x3b, 0x4c, 0x00, 0x05, 0x73, 0x74,
        0x61, 0x72, 0x74, 0x71, 0x00, 0x7e, 0x00, 0x01, 0x78, 0x70,
        0x73, 0x72, 0x00, 0x0e, 0x6a, 0x61, 0x76, 0x61, 0x2e, 0x75,
        0x74, 0x69, 0x6c, 0x2e, 0x44, 0x61, 0x74, 0x65, 0x68, 0x6a,
        (byte)0x81, 0x01, 0x4b, 0x59, 0x74, 0x19, 0x03, 0x00, 0x00,
        0x78, 0x70, 0x77, 0x08, 0x00, 0x00, 0x00, 0x66, (byte)0xdf,
        0x6e, 0x1e, 0x00, 0x78, 0x73, 0x71, 0x00, 0x7e, 0x00, 0x03,
        0x77, 0x08, 0x00, 0x00, 0x00, (byte)0xd5, 0x17, 0x69, 0x22,
        0x00, 0x78
    };

    public static void main(String[] args) {
        Period p = (Period) deserialize(serializedForm);
        System.out.println(p);
    }
    
    static Object deserialize(byte[] sf) {
        try {
            return new ObjectInputStream(new ByteArrayInputStream(sf)).readObject();
        } catch (IOException | ClassNotFoundException e) {
            throw new IllegalArgumentException(e);
        }
    }
}
```

- 위의 예제를 살펴보면, 별다른 validation check 를 수행하지 않기 때문에 정상적이지 않은 바이트스트림이 들어오더라도 객체가 생성된다.

## 4. 불변식을 보장하기 위한 방법
#### 4,1 유효성 검사
- 해결방법
```java
private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
    s.defaultReadObject(); // 기본 직렬화를 수행한다.
    if (start.compareTo(end) > 0) { // 유효성 검사를 수행한다.
        throw new InvalidObjectException(start + " 가 " + end + " 보다 늦다.");
    }
}
```
- Period 의 readObject 메서드가 defaultReadObject 를 호출한 다음 역직렬화된 객체가 유효한지 검사해야한다.
- 만약 유효성 검사에 살패한다면, InvalidObjectException 을 던지자
- 이것으로 허용되지 않은 Period 인스턴스를 생성하는 것은 막을 수 있다.
- 하지만 이것으로 끝이 아니다. !!

#### 4.2 방어적 복사
- Example
```java
public class MutablePeriod {
    
   public final Period period;
   public final Date start;
   public final Date end;

   public MutablePeriod() {
       try {
           ByteArrayOutputStream bos = new ByteArrayOutputStream();
           ObjectOutputStream out = new ObjectOutputStream(bos);

           // 불변식을 유지하는 Period 를 직렬화한다.
           out.writeObject(new Period(new Date(), new Date()));

           /*
            * bos 값에 악의 적인 바이트스트림을 주입한다.
            */
           byte[] ref = { 0x71, 0, 0x7e, 0, 5 }; // 악의적인 바이트스트림
           bos.write(ref);                       // start 필드
           ref[4] = 4;                           // 악의적인 바이트스트림
           bos.write(ref);                       // end 필드

           // 역직렬화 과정에서 Period 객체의 Date 참조를 훔친다.
           ObjectInputStream in = new ObjectInputStream(new ByteArrayInputStream(bos.toByteArray()));
           
           period = (Period) in.readObject();
           start  = (Date) in.readObject();
           end    = (Date) in.readObject();
           
       } catch (IOException | ClassNotFoundException e) {
           throw new AssertionError(e);
       }
   }
}
```

```java
public static void main(String[] args) {
    MutablePeriod mp = new MutablePeriod();
    Period p = mp.period;
    Date pEnd = mp.end;

    pEnd.setYear(78);      // end 필드를 80년으로 수정한다.
    System.out.println(p);
        
    pEnd.setYear(69);      // end 필드를 60년으로 수정한다.
    System.out.println(p);
    }
```

- 문제점
    - start, end 는 final 변수이지만, 해당 필드에 접근하여 수정이 가능하다.
    - 해당 문제는 Period의 readObject 메서드가 방어적 복사를 충분히 하지 않은 데 잏다.
    - 객체를 역직렬화할 때는 클라이언트가 소유해서는 안 되는 객체 참조를 갖는 필드를 모두 반드시 방어적으로 복사해야 한다.
    - 따라서 readObject 에서는 불변 클래스 안의 모든 private 가변 요소를 방어적으로 복사해야 한다.


- 해결방법
```java
private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
    s.defaultReadObject();

    // 가변 요소들을 방어젇으로 복사한다.
    start = new Date(start.getTime());
    end   = new Date(end.getTime());

    // 불변식을 만족하는지 검사한다.
    if (start.compareTo(end) > 0) {
        throw new InvalidObjectException(start + "가 " + end + "보다 늦다.");
    }
}
```
- 주의 사항
    - final 필드는 방어적 복사가 불가능하니 주의하자.
    - 그렇기 때문에 readObject 메서드를 사용하려면 start 와 end 필드에서 final 키워드를 모두 제거해야한다.
    
## 5. 기본 readObject 를 사용해도 되는 기준
- 판단 기준
    - `transient` 필드를 제외한 모든 필드의 값을 매개변수로 받아 유효성 검사를 없이도 필드에 대입하는 public 생성자를 추가해도 괜찮은지 

- 판단 기준에 부합한다면 기본 readObject 메서들 사용해도 좋다.
  

- 그게 아닌 경우 유효성 검사와 방어적 복사를 수행해야 한다.
    - 책에서 추천하는 방법은 직렬화 프록시 패턴을 사용하는 것이다.
    - 마지막으로 final 이 아닌 직렬화 가능 클래스라면, 생성자처럼 readObject 메서드도 재정의 가능 메서드를 호출해서는 안 된다.
    - 왜냐하면 하위 클래스의 상태가 완전히 역직렬화되기 전에 하위 클래스에서 재정의된 메서드가 실행되기 때문이다.







