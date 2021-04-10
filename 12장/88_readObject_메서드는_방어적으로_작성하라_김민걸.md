# readObject 메서드는 방어적으로 작성하라.

## readObject 메서드

 - 우리는 InputObjectStream / OutputObjectStream 를 통해 객체를 읽고 쓴다.
 - 이 클래스에 포함된 메서드가 readObject() / writeObject() 이다.
 - 클래스에 readObject() / writeObject() 가 정의되어 있다면, 기본 직렬화 과정에서 이 메서드를 통해 직렬화와 역직렬화를 수행한다.
    - 커스텀한 직렬화 (직렬화에 특정 처리를 하고 싶을 때) 사용.
    - private 메서드로 작성해야 한다.
    - 이 메서드들의 처음에 defaultWriteObject() / defaultReadObject() 를 호출하여 기본 직렬화를 실행하게 해야한다.
    - 리플렉션을 통해 작업을 수행한다.

<br>
<br>
    
## readObject 의 문제점
 - 새로운 객체를 만들어내는 특이한 public 생성자와 같다고 할 수 있다.
 - 따라서, 생성자처럼 `유효성검사`, `방어적 복사` 를 수행해야한다. 그렇지 않으면, 불변식을 보장하지 못한다.
 
### 불변식을 보장하지 못하는 사례 : Period 클래스 유효성 검사
 - readObject() 를 정의하지 않아서, 자바의 기본 직렬화를 수행한다.
 ```java
public final class Period implements Serializable {

    private Date start;
    private Date end;

    public Period(Date start, Date end) {
        this.start = new Date(start.getTime()); // 방어적 복사
        this.end = new Date(end.getTime());
        if (this.start.compareTo(this.end) > 0) { // 유효성 검사
            throw new IllegalArgumentException(start + " after " + end);
        }
    }

    public Date start() {
        return new Date(start.getTime());
    }

    public Date end() {
        return new Date(end.getTime());
    }
}
 ```

 - 아래와 같은 바이트스트림으로 Period 객체로 역직렬화한다면?

```java
public class BogusPeriod {
    // 불변식을 깨뜨리도록 조작된 바이트 스트림
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
        System.out.println(p.start);
        System.out.println(p.end);
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

 - 위 바이트스트림의 정보는, start 의 시각이 end 의 시각보다 느리게 조작했다.
 - 즉, 불변식을 꺠뜨린 객체로 역직렬화하도록 조작되었다.
 
 ```
 Fri Jan 01 12:00:00 PST 1999 // start 가 더 느리다.
 Sun Jan 01 12:00:00 PST 1984 // end 가 더 이르다.
```

#### 해결방법
 - readObject 를 정의하고, 유효성 검사를 실시한다.
 - Period 클래스에 다음의 메서드를 추가한다.
 ```java
 private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
     s.defaultReadObject(); // 기본 직렬화 수행
     if (start.compareTo(end) > 0) { // 유효성 검사
         throw new InvalidObjectException(start + " 가 " + end + " 보다 늦을 수 없습니다.");
     }
 }
 ```


<br>
<br>

### 불변식을 보장하지 못하는 사례 : Period 클래스 방어적 복사
 - 직렬화된 바이트 스트림 끝에 private Date 필드로의 참조를 추가하면 가변 Period 인스턴스를 만들 수 있다.
 ```java
public class MutablePeriod {
    public final Period period;

    public final Date start;

    public final Date end;

    public MutablePeriod() {
        try {
            ByteArrayOutputStream bos = new ByteArrayOutputStream();
            ObjectOutputStream out = new ObjectOutputStream(bos);

            // 불변식을 유지하는 Period 를 직렬화.
            out.writeObject(new Period(new Date(), new Date()));

            /*
             * 악의적인 start, end 로의 참조를 추가.
             */
            byte[] ref = { 0x71, 0, 0x7e, 0, 5 }; // 악의적인 참조
            bos.write(ref); // 시작 필드
            ref[4] = 4; // 악의적인 참조
            bos.write(ref); // 종료 필드

            // 역직렬화 과정에서 Period 객체의 Date 참조를 훔친다.
            ObjectInputStream in = new ObjectInputStream(new ByteArrayInputStream(bos.toByteArray()));
            period = (Period) in.readObject();
            start = (Date) in.readObject();
            end = (Date) in.readObject();
        } catch (IOException | ClassNotFoundException e) {
            throw new AssertionError(e);
        }
    }
}
```

```java
    public static void main(String[] args) {
        MutablePeriod mp = new MutablePeriod();
        Period mutablePeriod = mp.period; // 불변 객체로 생성한 Period
        Date pEnd = mp.end; // MutablePeriod 클래스의 end 필드
        
        pEnd.setYear(78); // MutablePeriod 의 end 를 바꿨는데 ?
        System.out.println(mutablePeriod.end()); // Period 의 값이 바뀐다.
        
        pEnd.setYear(69);
        System.out.println(mutablePeriod.end());
    }
```
 - 결과
 ```
Fri Apr 07 19:59:32 KST 1978
Mon Apr 07 19:59:32 KST 1969
 ```

 - 불변 객체 Period 를 직렬화 / 역직렬화한다고 생각할 수 있지만,
 - 위의 방법으로 불변식을 깨뜨릴 수 있다.
 - 실제로 String 이 불변이라는 사실에 기댄 보안 문제들이 존재한다.
 
#### 해결법
 - 객체를 역직렬화할 때는 클라이언트가 소유해서는 안되는 객체 참조를 갖는 필드를 모두 방어적 복사한다.
 - 불변 클래스 안의 모든 private 가변 요소를 방어적 복사한다.
 ```java
private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
    s.defaultReadObject();

    // 방어적 복사를 통해 인스턴스의 필드값 초기화
    start = new Date(start.getTime());
    end = new Date(end.getTime());

    // 유효성 검사
    if (start.compareTo(end) > 0)
        throw new InvalidObjectException(start +" after "+ end);
}
```
 - 유효성 검사보다 먼저 방어적 복사
    - 반대라면, 유효성 검사 이후 방어적 복사 이전에 불변식을 깨뜨릴 틈이 생긴다. (item 50)
 - final 필드는 방어적 복사가 불가능하므로, 필드를 final 이 아니게 해야함.
 
<br>
<br>

## 기본 readObject() vs readObject 직접 정의 ?
 `transient 필드를 제외한 모든 필드의 값을 매개변수`로 받아 `유효성 검사 없이` 필드에 대입하는 `public 생성자` 를 추가해도 괜찮은가 ?
 - Yes -> 기본 readObject
 - No -> readObject 직접 정의 후 유효성 검사와 방어적 복사 수행. or 직렬화 프록시 패턴사용.
 
### 마지막 팁
 - readObject 메서드에서 재정의 가능 메서드를 호출하면 안된다. (item 19)
    - 클래스가 final 이 아닌 경우에만 해당
    - 이 클래스의 하위 클래스가 불리기 이전에 생성자의 재정의된 메서드가 실행되므로 오류를 뱉게 될 것이다.
    