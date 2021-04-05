# item 86. Serializable을 구현할 지는 신중히 결정하라

Serializable은 Java 직렬화를 사용하기 위한 시작점이다. `int`, `long`과 같은 primitive 타입은 Java에서 기본적으로 직렬화를 지원하지만 객체의 경우 직렬화를 사용하기 위해서는 Serializable을 구현해야한다.

> Java 직렬화 참고
[](https://github.com/Meet-Coder-Study/posting-review/pull/134/files?short_path=afab0a5#diff-afab0a521a171ab1b8fe20acdf103022abc1ef8b0a737a06da06beb7ad9b0093)

Serializable을 객체가 구현만 하면 해당 객체는 Java가 지원하는 직렬화 시스템의 지원을 받을 수 있다. 하지만 이는 매우 신중하게 결정해야한다. 사용하기는 편하지만 길게 봤을때 값비싼 일이 될 수 있기 때문이다.

참고로 Serializable 객체를 직렬화 할때는 ObjectOutputStream을 사용하며 Serializable을 구현하지 않은 객체를 직렬화하면 `java.io.NotSerializableException`가 발생한다.

## Serializable을 구현하면 릴리즈 뒤에는 수정하기 어렵다.

Java Serializable 참고: [https://woowabros.github.io/experience/2017/10/17/java-serialize2.html](https://woowabros.github.io/experience/2017/10/17/java-serialize2.html)

클래스가 Serializable을 구현하게 되면 직렬화된 바이트 스트림 인코딩도 하나의 공개 API가 된다. 때문에 이 클래스가 널리 퍼지면 그 직렬화 형태도 영원히 지원해야한다. 즉, Serializable을 구현한 순간부터 해당 객체의 직렬화 형태는 Java 직렬화에 묶이는 것이다. 기본 직렬화 형태에서는 private와 package-private 수준의 필드마저도 API로 공개가 된다. 즉, 캡슐화가 깨진다.

```java
public class Person implements Serializable {

    private final String name;
    private final int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
}
```

> 위 Person 클래스가 아래 예시의 가장 기본적인 형태. 모든 예시의 직렬화 기준은 위 형태.

```java
@Test
void serialize() throws IOException {
    Person person = new Person("pkch", 28);

    try (FileOutputStream fileOutputStream = new FileOutputStream(SERIALIZE_OBJECT_FILE_PATH)) {
        try (ObjectOutputStream objectOutputStream = new ObjectOutputStream(fileOutputStream)) {
            objectOutputStream.writeObject(person);
        }
    }
}
```

위와 같이 person 객체를 ObjectOutputStream을 통해 직렬화를 한 뒤에 이를 FileOutputStream을 통해 `person.ser`에 객체 내용을 저장하면 다음과 같이 나타난다.

![](https://user-images.githubusercontent.com/30178507/113633990-eaa0a200-96a8-11eb-8f6c-9cb8627566f1.png)

> 위 직렬화 결과를 보면 private 필드인 `name`과 `age`가 있는 것을 볼 수 있다. 이 때문에 캡슐화가 깨진다는 이야기를 하는 걸로 보임.

`SERIALIZE_OBJECT_FILE_PATH`에 저장된 직렬화 객체를 역직렬화 하려면 다음과 같이 하면된다.

```java
@Test
void deserialize() throws IOException {
    Person person = null;

    try (FileInputStream fileInputStream = new FileInputStream(SERIALIZE_OBJECT_FILE_PATH)) {
        try (ObjectInputStream objectInputStream = new ObjectInputStream(fileInputStream)) {
            person = (Person) objectInputStream.readObject();
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
    }

    assertThat(person.getName()).isEqualTo("pkch");
    assertThat(person.getAge()).isEqualTo(28);
}
```

`SERIALIZE_OBJECT_FILE_PATH`에 저장된 직렬화 객체 정보를 가져와서 `ObjectInputStream#readObject`로 다시 객체화 한 것이다.

단, 만약에 Person에 새로운 필드가 필요하다고 가정한다.

```java
public class Person implements Serializable {
    private final String name;
    private final int age;
    private final double height;
    private final double weight;

    public Person(String name, int age, double height, double weight) {
        this.name = name;
        this.age = age;
        this.height = height;
        this.weight = weight;
    }
}
```

키 `height`와 몸무게 `weight` 정보가 추가되었는데 이렇게 필드가 추가된 경우 앞서 직렬화된 객체를 역직렬화할 수 없다.

```
edu.pkch.serialize.Person; local class incompatible: stream classdesc serialVersionUID = -6765962567694553436, local class serialVersionUID = -2416939271889238383
java.io.InvalidClassException: edu.pkch.serialize.Person; local class incompatible: stream classdesc serialVersionUID = -6765962567694553436, local class serialVersionUID = -2416939271889238383
```

기본적으로 serialVersionUID는 정의하지 않으면 해당 객체의 hashCode를 기반으로 설정이 되는데 height와 weight가 추가되면서 serialVersionUID이 바뀐 것이다. 때문에 실패가 발생한다.

때문에 이런 문제를 방지하기 위해서 serialVersionUID를 관리해야한다.

```java
public class Person implements Serializable {
    private static final long serialVersionUID = 1L;

    private final String name;
    private final int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
}
```

serialVersionUID를 `1`로 정의하면 필드가 추가되더라도 해당 직렬화 정보에 serialVersionUID가 `1`인 경우 `Person`이라는 것을 알 수 있기 때문에 Person 객체로 다시 역직렬화가 된다.

```java
public class Person implements Serializable {
    private static final long serialVersionUID = 1L;

    private final String name;
    private final int age;
    private final double height;
    private final double weight;

    public Person(String name, int age, double height, double weight) {
        this.name = name;
        this.age = age;
        this.height = height;
        this.weight = weight;
    }
}
```

```java
@Test
void deserialize() throws IOException {
    Person person = null;

    try (FileInputStream fileInputStream = new FileInputStream(SERIALIZE_OBJECT_FILE_PATH)) {
        try (ObjectInputStream objectInputStream = new ObjectInputStream(fileInputStream)) {
            person = (Person) objectInputStream.readObject();
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
    }

    assertThat(person.getName()).isEqualTo("pkch");
    assertThat(person.getAge()).isEqualTo(28);
    assertThat(person.getHeight()).isEqualTo(0);
    assertThat(person.getWeight()).isEqualTo(0);
}
```

기존에 Person은 name과 age만 존재했으므로 height와 weight에는 double의 기본값인 0으로 할당되어 역직렬화된다.

물론 serialVersionUID 없이 `ObjectOutputStream#putFields`와 `OutputInputStream#readFields`를 사용하면 원래의 직렬화를 유지하면서도 클래스 필드 추가 / 제거가 가능하지만 복잡한 코드가 추가될 뿐 아니라 지저분해진다. 때문에 **특별한 문제가 없다면 serialVersionUID를 관리해야한다.**

### serialVersionUID의 한계

serialVersionUID는 클래스에 필드가 추가 / 제거 되는 경우에 역직렬화 에러가 발생하는 문제를 해결해주지만 기존에 존재하던 변수 이름을 변경했을때 해당 데이터는 누락된다.

```java
public class Person implements Serializable {
    private static final long serialVersionUID = 1L;

    private final String name;
    private final int na2;

    public Person(String name, int na2) {
        this.name = name;
        this.na2 = na2;
    }
}
```

이렇게 `age` 변수명을 `na2`로 변경하고 역직렬화했을때 값을 가져오지 못한다. 기존에 작성했던 테스트는 깨지게 된다.

```
org.opentest4j.AssertionFailedError: 
Expecting:
 <0>
to be equal to:
 <28>
```

즉, `28`로 예상하고 테스트를 작성했지만 값을 역직렬화 하지 못해 int의 기본값인 `0`으로 세팅된 것이다.

위 경우는 깂의 누락이 있을 뿐 에러는 발생하지 않는다. 하지만 기존에 존재하는 변수의 타입이 변경되면 이야기가 다르다.

```java
public class Person implements Serializable {
    private static final long serialVersionUID = 1L;

    private final String name;
    private final long age;

    public Person(String name, long age) {
        this.name = name;
        this.age = age;
    }
}
```

이렇게 기존에 존재하는 age 필드의 타입을 int에서 long으로 변경했다. 이 상황에서 역직렬화를 하면 다음과 같은 에러가 발생한다.

```
edu.pkch.serialize.Person; incompatible types for field age
java.io.InvalidClassException: edu.pkch.serialize.Person; incompatible types for field age
```

이렇게 Java 직렬화 시스템은 타입에 엄격하다. 타입이 변경이 되면 에러가 발생하기 때문에 주의가 필요하다.

## 버그와 보안 구멍이 생길 위험이 높아진다. `item 85`

객체를 생성하는 가장 기본적인 방법은 생성자를 이용하는 것이다. 근데 `ObjectInputStream#readObject`는 객체를 만들어 낼 수 있는 마법같은 메서드이다. 즉, 객체를 Serializable로 구현하면 생성자 이외에 객체를 생성할 수 있는 **숨은 생성자**가 생기는 것이다.

기본 역직렬화를 통해 불변식이 깨질 수 있으며, 허가되지 않은 접근에 쉽게 노출될 수 있다.

## 해당 클래스의 신버전을 릴리즈할 때 테스트할 것이 늘어난다.

앞서 본 Serializable의 문제점과 같이 구버전의 직렬화 형태가 신버전에서 역직렬화가 가능한지, 그 역도 가능한지 테스트해야한다. 즉, 테스트의 양이 직렬화 가능 클래스의 수와 릴리즈 횟수에 비례한다.

릴리즈 할 때마다 반드시 양방향 직렬화/역직렬화가 가능한지 확인하고 원래의 객체를 충실히 복제가능한지 반드시 확인해야한다.

## Serializable 구현 여부에 신중할 것

객체를 전송할 때나 저장할 때 Java 직렬화를 사용하는 프레임워크용으로 만든 클래스라면 선택의 여지없이 Serializable을 구현해야할 것이다. 참고로 이 경우에 Serializable의 구현 클래스에 사용되는 컴포넌트 클래스도 Serializable을 구현해야한다.

이 경우 Serializable 구현에 따른 이점과 비용을 생각해서 구현하는 것이 좋다. 참고로 BigInteger, Integer 같은 **값 객체**나 컬렉션 객체는 Serializable을 구현했고 스레드 풀과 같이 **동작을 표현하는 객체**는 Serializable을 구현하지 않았다.

> 사실 Java 직렬화는 최대한 사용하지 않는 것이 좋아보임.
Spring Data Redis의 경우 기본적으로 Java 직렬화를 사용하지만 다른 직렬화 형태로 변경할 수 있음. 보통 Json 직렬화 형태로 변경하여 사용하는 것으로 보임

## 상속용으로 설계된 클래스는 Serializable을 구현하면 안되며, 인터페이스도 Serializable을 확장하면 안된다.

이 규칙을 따르지 않고 Serializable을 확장, 구현하면 앞서 언급한 Java 직렬화의 문제를 고스란히 하위 구현 클래스들이 가지게 된다.

상속용으로 설계된 클래스 중 Serializable을 구현한 대표적인 예시로는 Throwable과 Component가 있다.
Throwable은 RMI를 통해 클라이언트로 예외를 보내기 위해서, Component는 GUI를 전송하고 저장하고 복원하기 위해 Serializable을 구현했다.

Java RMI 참고: [https://java.ihoney.pe.kr/54](https://java.ihoney.pe.kr/54)

```java
public class Throwable implements Serializable {
    /** use serialVersionUID from JDK 1.0.2 for interoperability */
    private static final long serialVersionUID = -3042686055658047285L;

    // ...
}
```

## 직렬화와 확장이 모두 가능할 때

클래스의 인스턴스 필드가 직렬화 및 확장이 모두 가능하다면 몇 가지 주의사항이 있다.

1. 인스턴스 필드의 값 중에 불변식을 보장해야할 게 있다면 반드시 하위 클래스에서 **finalize** 메서드를 재정의하지 못하게 해야한다.

    finalize를 재정의하면서 final 키워드를 붙여서 선언하는 것이다. 이렇게 하지 않으면 finalizer 공격에 취약해질 수 있다. `item 8`

2. 인스턴스 필드중 기본값 `int는 0, Object는 null 등` 으로 설정되면 위배되는 불변식이 있다면 `readObjectNoData` 메서드를 반드시 추가해야한다.

    ```java
    private void readObjectNoData() throws InvalidObjectException {
    	throw new InvalidObjectException("스트림 데이터가 필요합니다");
    }
    ```

    Java 4부터 지원하는 메서드로 기존 직렬화 가능 클래스에 직렬화 가능 상위 클래스를 추가하는 드문 경우를 위한 메서드이다.

    readObjectNoData 명세 참고: [https://docs.oracle.com/javase/7/docs/platform/serialization/spec/input.html#6053](https://docs.oracle.com/javase/7/docs/platform/serialization/spec/input.html#6053)

    ```java
    public class Person implements Serializable {
        private static final long serialVersionUID = 1L;

        private String name;
        private long age;

        public Person() {
        }

        public Person(String name, long age) {
            this.name = name;
            this.age = age;
        }
    }

    public class Employee implements Serializable {
        private final String company;
        private final String team;

        public Employee(String company, String team) {
            this.company = company;
            this.team = team;
        }
    }
    ```

    아까 정의한 Person과 새로 정의한 Employee이다. 새로 정의한 Employee를 먼저 직렬화해서 사용한다고 가정한다.

    ```java
    @Test
    void serialize() throws IOException {
        Employee employee = new Employee("woowabros", "서비스개발팀");

        try (FileOutputStream fileOutputStream = new FileOutputStream(SERIALIZE_OBJECT_FILE_PATH)) {
            try (ObjectOutputStream objectOutputStream = new ObjectOutputStream(fileOutputStream)) {
                objectOutputStream.writeObject(employee);
            }
        }
    }
    ```

    이렇게 직렬화되면 직렬화한 정보를 담은 파일에는 `company`와 `team` 정보만 가지고 있게 된다. 여기서 만약에 Employee에 Person을 상속하면 어떻게 될까? 참고로 Person은 Serializable을 구현하고 있다.

    ```java
    public class Employee extends Person {
        private final String company;
        private final String team;

        public Employee(String company, String team) {
            this.company = company;
            this.team = team;
        }

        public String getCompany() {
            return company;
        }

        public String getTeam() {
            return team;
        }
    }
    ```

    이렇게 변경한 뒤 기존에 직렬화 데이터는 Person에서 가지고 있는 name과 age를 알 수 없으므로 각각 기본값을 할당받게 된다.

    ```java
    assertThat(employee.getCompany()).isEqualTo("woowabros");
    assertThat(employee.getTeam()).isEqualTo("서비스개발팀");
    assertThat(employee.getName()).isEqualTo(null);
    assertThat(employee.getAge()).isEqualTo(0);
    ```

    이때 만약 상위 Person의 값들이 기본값이 되면 안되는 경우에 다음과 같이 readObjectNoData를 사용해야한다. 상위 Person의 값들이 기본값이 아닌 다른 값으로 설정하는 것이기 때문에 readObjectNoData를 Person에 정의해야한다.

    ```java
    private void readObjectNoData() {
        this.name = "pkch";
        this.age = 28;
    }
    ```

    이렇게 정의하면 기존에 name과 age가 각각 기본값인 `null`과 `0`으로 할당됐던 것과 달리 readObjectNoData로 정의한 `pkch`와 `28`로 할당된다.

    ```java
    assertThat(employee.getCompany()).isEqualTo("woowabros");
    assertThat(employee.getTeam()).isEqualTo("서비스개발팀");
    assertThat(employee.getName()).isEqualTo("pkch");
    assertThat(employee.getAge()).isEqualTo(28);
    ```

    > 참고로 readObjectNoData는 ObjectInputStream에서 직렬화 데이터에 없는 필드에 대해서 값을 정의할 때 호출한다. `ObjectInputStream#readSerialData` 참고

## 상위 클래스에서 직렬화를 지원하지 않을 때

상속용 클래스에서 Serializable를 구현하지 않는다면 하나만 생각하면 된다. 상속용 클래스가 Serializable을 지원하지 않는 경우 하위 구현 클래스가 Serializable을 구현할 때 부담이 늘어난다.

이때 상위 클래스에서 인자가 없는 기본 생성자를 지원하면 하위 클래스에서 Serializable로 간단하게 직렬화를 구현할 수 있다. 만약 지원하지 않는다면 하위 클래스에서는 직렬화 프록시 패턴을 사용해야한다. `item 90`

## 내부 클래스는 Serializable을 구현하면 안된다.

내부 클래스는 바깥 인스턴스의 참조와 유효 범위 안의 지역변수들을 저장하기 위해 컴파일러가 자동으로 생성한 필드가 추가된다. 익명 클래스와 지역 클래스의 이름 짓는 규칙이 언어 명세에도 없기 때문에 이 필드들이 클래스 정의에 어떻게 추가되는지도 정의되지 않았다.

따라서 내부 클래스의 직렬화 형태는 불분명하므로 Serializable을 구현하면 안된다. 단, 정적 멤버 클래스는 Serializable 구현으로 Java 직렬화가 가능하다.
