# 아이템10. equals는 일반 규약을 지켜 재정의하라

- equals 메서드 재정의는 곳곳에 함정이 도사리고 있어 끔찍한 결과를 초래한다.
- 따라서 아예 재정의를 하지 않는 것이 더 좋을 때도 있다.

## 사전 지식

### 동치

> 수학과 논리학에서 동치(同値)란 두 문장이 논리적으로 같다는 것을 의미한다. 이것은 한 문장이 참이면 다른 한 문장도 참이고, 한 문장이 거짓이면 다른 문장도 거짓이 된다는 것을 뜻한다. - 위키백과

### 동치 관계

> 집합을 서로 같은 원소들로 이루어진 부분집합으로 나누는 연산

## equals 재정의 하지 않는 경우

### 각 인스턴스가 본질적으로 고유할 경우

- 값을 표현하는 것이 아니라 동작하는 개체를 표현하는 클래스일 경우를 말한다.
- 예를들어 Thread,

### 인스턴스의 '논리적 동치성'을 검사할 일이 없을 경우

> 논리적 동치성이란?

### 상위 클래스에서 재정의한 equals가 하위 클래스에도 딱 들어맞는 경우

```java
public abstract class AbstractList<E> extends AbstractCollection<E> implements List<E> {
	public boolean equals(Object o) {
        if (o == this)
            return true;
        if (!(o instanceof List))
            return false;

        ListIterator<E> e1 = listIterator();
        ListIterator<?> e2 = ((List<?>) o).listIterator();
        while (e1.hasNext() && e2.hasNext()) {
            E o1 = e1.next();
            Object o2 = e2.next();
            if (!(o1==null ? o2==null : o1.equals(o2)))
                return false;
        }
        return !(e1.hasNext() || e2.hasNext());
    }
}
```

- 위 코드와 같이 List의 대부분은 eqauls 메서드가 재정의 되어 있고 구현체들은 모두 이걸 사용하게 된다.

### 클래스가 private이거나 package-private이고 equals 메서드를 호출할 일이 없을 경우

- equlas 자체를 호출하는 걸 막고 싶다면 아래와 같이 하는것이 좋다

```java
@Override
public boolean equals (Object object) {
	throw new AssertionError();
}
```

### 인스턴스 통제 클래스

- 인스턴스가 2개 이상 만들어지지 않기 때문에 논리적 동치성과 객체 식별성이 같은 의미가 되기 때문이다.
- Enum, 싱글톤

## 그렇다면 언제 equals 메서드를 재정의해야 하냐?

- 객체 식별성이 아니라 논리적 동치성을 확인해야 하는데, 상위 클래스의 equals가 논리적 동치성을 비교하도록 재정의되지 않았을 때다.
- 이러한 점을 이용해 Map 키와 Set 원소로 사용할 수 있게 된다.

## equals 메서드 일반 규약

- equals 메서드는 동치 관계를 구현하며, 다음을 만족한다.
- 여기서 모든 참조값은 null이 아닐 때이다.
- 아래 Member 클래스를 이용해 하나씩 구현해보도록 하겠습니다.

```java
public class Member {
		private String email;
    private String name;
    private Integer age;

    public Member(String email, String name, Integer age) {
        this.email = email;
        this.name = name;
        this.age = age;
    }
}
```

### 반사성(reflexivity)

```java
x.equlas(x) == true
```

- 단순히 말하면 객체는 자기 자신과 같아야 한다는 뜻이다.
- 이걸 지키는지를 확인할 수 있는 방법은 인스턴스가 들어있는 컬렉션에 contains 메서드를 호출해 true가 나오면 된다.

```java
@Override
    public boolean equals(Object o) {
        if (o == null) {
            return false;
        }
        return this == o;
    }
```

```java
public static void main(String[] args) {
        List<Member> members = new ArrayList<>();
        Member member = new Member("rutgo", 29);
        members.add(member);

        System.out.println(members.contains(member)); // true
    }
```

### 대칭성(symmetry)

```java
x.equals(y) == y.equals(x)
```

- 서로에 대한 동치 여부에 똑같이 답해야 한다는 뜻입니다.
- 이 부분은 어길수도 있는데 아래와 같은 상황일 것 이다.

```java
@Override
public boolean equals(Object o) {
	if (null == o) {
		return false;
	}
	if (o instanceof Member) {
		return email.equalsIgnoreCase(((Member)o).email);
	}
	if (o instanceof String) {
		return email.equalsIgnoreCase((String)o);
	}
	return false;
}
```

- 극단적인 예로 email의 값이 같다면 같은 인스턴스라고 정의하고, 대소문자를 구분하지 않는다고 해봅시다.

```java
public static void main(String[] args) {
    Member rutgo = new Member("ksy90101", "rutgo", 29);
    Member seyun = new Member("ksy90101", "seyun", 29);
    String email = "ksy90101";
    System.out.println(rutgo.equals(seyun)); // true
    System.out.println(rutgo.equals(email)); // true
		System.out.println(email.equals(rutgo)); // false
}
```

- 위와 같은 상황에서는 당연히 true가 나올텐데, 아래와 같은 상황에서도 true가 나올것이다. 반대로 `email.equals(rutgo)` 는 false가 나올 것이다.
- 당연한건데 String의 equals()는 Member의 존재를 모른다. 따라서 지금 같은 경우 대칭성을 명백히 위반하게 된다. 반사성과 같이 컬렉션으로 테스트를 해보자.

```java
public static void main(String[] args) {
        List<Member> members = new ArrayList<>();
        Member rutgo = new Member("ksy90101", "rutgo", 29);
        Member seyun = new Member("ksy90101", "seyun", 29);
        String email = "ksy90101";

        members.add(rutgo);

        System.out.println(members.contains(email)); // false
        System.out.println(members.contains(seyun)); // true
    }
```

- 지금과 같은 경우에는 false가 나오게 된다. 이건 단순히 OpenJDK의 구현방식 때문이다.

```java
public boolean contains(Object o) {
    Iterator<E> it = iterator();
    if (o==null) {
        while (it.hasNext())
            if (it.next()==null)
                return true;
    } else {
        while (it.hasNext())
            if (o.equals(it.next()))
			return true;
		}
		return false;
}
```

- 위의 코드를 살펴보면 o.equals()를 사용하게 된다. 즉, String.eqauls()를 사용하기 때문에 false인것이다. 그렇다면 이게 갑작스럽게 it.next().equals(o);로 변경된다면 true가 나오게 된다.
- 즉, 그 객체를 사용하는 다른 객체를 사용하는 다른 객체들이 어떻게 반응할지 알 수가 없다는 것이다.
- 이 문제는 해결하는 것은 간단하다 String과 Member의 equals()를 연동하겠다 라는 꿈을 버리면 된다.

```java
@Override
public boolean equals(Object o) {
    if (null == o) {
        return false;
    }
    if (o instanceof Member) {
        return email.equalsIgnoreCase(((Member)o).email);
    }
    return false;
}
```

### 추이성(transitivity)

```java
x.equals(y) == true
y.equals(z) == true
x.equals(z) == true
```

- 간단한 예제로 스터디 멤버라는 객체가 있고 멤버를 상속받는다고 생각해봅시다.
- 아울러 멤버 객체의 equals()는 아래와 같이 정의되어 있습니다.

```java
@Override
public boolean equals(Object o) {
    if (!(o instanceof Member)) {
        return false;
    }
    Member member = (Member)o;

    return email.equals(member.email) && name.equals(member.name) 
						&& age.equals(member.age);
}
```

```java
public class StudyMember extends Member {
    private String study;

    public StudyMember(String email, String name, Integer age, String study) {
        super(email, name, age);
        this.study = study;
    }

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof StudyMember)) {
            return false;
        }

        return super.equals(o) && ((StudyMember)o).study.equals(study);
    }
}
```

- 위의 코드에 가장 큰 문제는 무엇일까? 바로 Object로 Member가 들어오면 false를 반환한다는 점이다.  즉, 대칭성이 깨지게 된다.
- 또한 Study.equals()에서는 study 필드값을 검사하지 않고 비교하기 때문에 true가 나오기도 한다.

```java
public static void main(String[] args) {
    Member rutgo = new Member("ksy90101", "rutgo", 29);
    StudyMember studyMember = new StudyMember("ksy90101", "rutgo", 29, "effective java study");

    System.out.println(rutgo.equals(studyMember)); // true
    System.out.println(studyMember.equals(rutgo)); // false
}
```

- 일단 첫번째 문제인 대칭성을 해결해보자.

```java
@Override
public boolean equals(Object o) {
		if (!(o instanceof Member)) {
        return false;
    }

    if (!(o instanceof StudyMember)) {
        return o.equals(this);
    }

    return super.equals(o) && ((StudyMember)o).study.equals(study);
}
```

- 이런식으로 한다면 손쉽게 대칭성 문제를 해결할 수 있을 것이다. 그러나 아직 문제가 남았다. 아래 코드와 같이 추이성은 지켜지지 않게 된다.

```java
public static void main(String[] args) {
    Member rutgo = new Member("ksy90101", "rutgo", 29);
    StudyMember studyMember = new StudyMember("ksy90101", "rutgo", 29, "effective java study");
    StudyMember studyMember2 = new StudyMember("ksy90101", "rutgo", 29, "Blog Posting study");
    System.out.println(rutgo.equals(studyMember)); // true
    System.out.println(rutgo.equals(studyMember2)); // true
    System.out.println(studyMember.equals(studyMember2)); // false
}
```

- 이러한 예제를 보면 즉, 구체 클래스를 확장해 새로운 값을 추가하면서 equals 규약을 만족시킬 방법은 존재하지 않는다는걸 알수 있다. 이러한 점은 상속의 단점 중 하나가 될 수 있다.
- 그렇다면 아래의 코드는 어떨까?

```java
@Override
public boolean equals(Object o) {
    if (o == null || o.getClass() != this.getClass()) {
        return false;
    }
    Member member = (Member)o;

    return email.equals(member.email) && name.equals(member.name) && age.equals(member.age);
}
```

- 이렇게 된다면 같은 객체일 때만 비교할 수 있다. 즉, StudyMember와 Member는 무조건 다르다고 할 것인데, 가장 큰 문제가 리스코프 치환 원칙(LSP)를 위반하게 된다는 점이다.
- 즉, 어떤 타입에 있어 중요한 속성이라면 그 하위 타입에서도 마찬가지로 중요하기 때문에 그 타입의 모든 메서드가 하위 타입에서도 똑같이 잘 작동해야 한다라는 것이다.
- 간단하게 말하자만 StduyMember도 결국 Member이므로 Member로 활용될 수 있어야 한다는 점이다.
- 그럼 아예 방법이 없는 것일까?
- 가장 좋은건 `아이템 18의 조언처럼 상속 대신 컴포지션을 사용하라` 과 뷰(view) 메서드를 Public으로 추가하는 것이다.
- 자바 라이브러리에서도 잘못된 상속의 equals() 구현한 경우가 있는데 바로 `java.sql.Timestamp`와 `java.util.Date`가 된다.

```java
public class Date
    implements java.io.Serializable, Cloneable, Comparable<Date>
{
	public boolean equals(Object obj) {
        return obj instanceof Date && getTime() == ((Date) obj).getTime();
    }
}
```

```java
public class Timestamp extends java.util.Date {
	public boolean equals(Timestamp ts) {
        if (super.equals(ts)) {
            if  (nanos == ts.nanos) {
                return true;
            } else {
                return false;
            }
        } else {
            return false;
        }
    }
}
```

- 가장 큰 점은 대칭성을 위반한 것이다. 즉 섞어 쓰면 잘못된 동작을 할 수 있다.
- 따라서 아래와 같이 주의점이 적혀 있다.

> This type is a composite of a java.util.Date and a separate nanoseconds value. Only integral seconds are stored in the java.util.Date component. The fractional seconds - the nanos - are separate. The Timestamp.equals(Object) method never returns true when passed an object that isn't an instance of java.sql.Timestamp, because the nanos component of a date is unknown. As a result, the Timestamp.equals(Object) method is not symmetric with respect to the java.util.Date.equals(Object) method. Also, the hashCode method uses the underlying java.util.Date implementation and therefore does not include nanos in its computation.

> 추상 클래스의 하위 클래스에서 라면 equals 규약을 지키면서도 값을 추가할 수 있다. `아이템 23의 태그 달린 클래스보다는 클래스 계층구조`를 활용하라를 참고하면 된다.

### 일관성(consistency)

```java
x.equals(y) == true;
x.equals(y) == true;
x.equals(y) == true;
```

- 가장 좋은 방법은 바로 불변이다.
- 그러나 일관성을 깨트리게 하는 것은 equals의 판단에 신뢰할 수 없는 자원이 있는 것이다.

```java
public final class URL implements java.io.Serializable {

	public boolean equals(Object obj) {
        if (!(obj instanceof URL))
            return false;
        URL u2 = (URL)obj;

        return handler.equals(this, u2);
    }
}
```

```java
public abstract class URLStreamHandler {
	protected boolean equals(URL u1, URL u2) {
        String ref1 = u1.getRef();
        String ref2 = u2.getRef();
        return (ref1 == ref2 || (ref1 != null && ref1.equals(ref2))) &&
               sameFile(u1, u2);
    }
	protected boolean sameFile(URL u1, URL u2) {
        // Compare the protocols.
        if (!((u1.getProtocol() == u2.getProtocol()) ||
              (u1.getProtocol() != null &&
               u1.getProtocol().equalsIgnoreCase(u2.getProtocol()))))
            return false;

        // Compare the files.
        if (!(u1.getFile() == u2.getFile() ||
              (u1.getFile() != null && u1.getFile().equals(u2.getFile()))))
            return false;

        // Compare the ports.
        int port1, port2;
        port1 = (u1.getPort() != -1) ? u1.getPort() : u1.handler.getDefaultPort();
        port2 = (u2.getPort() != -1) ? u2.getPort() : u2.handler.getDefaultPort();
        if (port1 != port2)
            return false;

        // Compare the hosts.
        if (!hostsEqual(u1, u2))
            return false;

        return true;
    }

	protected boolean hostsEqual(URL u1, URL u2) {
        InetAddress a1 = getHostAddress(u1);
        InetAddress a2 = getHostAddress(u2);
        // if we have internet address for both, compare them
        if (a1 != null && a2 != null) {
            return a1.equals(a2);
        // else, if both have host names, compare them
        } else if (u1.getHost() != null && u2.getHost() != null)
            return u1.getHost().equalsIgnoreCase(u2.getHost());
         else
            return u1.getHost() == null && u2.getHost() == null;
    }
}
```

- 위 코드와 같이 같은 URL인지를 검사할 때 IP 주소로 변경해 비교하게 된다. 이때 호스트 이름을 IP주수로 변경하는 것이 네트워크를 통해야 하는데 그 결과가 항상 같다고 보장할 수 없다.
- 이러한 문제를 피하기 위해 항상 equals는 항시 메모리에 존재하는 객체만을 사용한 결정적 계산만을 수행해야 한다는 것을 알 수 있다.

### non-null

```java
x.equals(null) == false;
```

- 이 부분은 쉽게 지킬수 있는데, equals()를 재정의 시 타입 검사를 하게 된다면 지켜지게 된다. 이걸 묵시적 null 검사라고 한다.

```java
@Override
public boolean equals(Object o) {
    if (!(o instanceof Member)) {
        return false;
    }
    Member member = (Member)o;

    return email.equals(member.email) && name.equals(member.name) && age.equals(member.age);
}
```

## equals 메서드 구현 방법 단계별 정리

1. == 연산자를 사용해 입력이 자기 자신의 참조인지 확인한다.
    - 단순한 성능 최적화용으로 비교 작업이 복잡한 상황일 때 값어치를 한다.
2. instanceof 연산자로 입력이 올바른 타입인지 확인한다.
    - 위에서 설명했듯이 묵시적 null 체크를 할 수 있다.
    - 또한 Collection interface처럼 특정 인터페이스의 타입 체크를 할 수도 있다.
3. 입력을 올바른 타입으로 형변환한다.
    - instanceof 검사를 했기 때문에 100% 성공한다.
4. 입력 객체와 자기 자신의 대응되는 핵심 필드들이 모두 일치하는지 하나씩 검사한다.

### 완성 코드

```java
@Override
public boolean equals(Object o) {
    if (this == o)
        return true;
    if (o == null || getClass() != o.getClass())
        return false;
    Member member = (Member)o;
    return email.equals(member.email) && name.equals(member.name) && age.equals(member.age);
}
```

## 주의 사항

- 기본 타입은 `==` 으로 비교하고 그 중 double, float는 Double.compare(), Float.compare()을 이용해 검사해야 한다. 이유는 부동소수점을 다뤄야 하기 때문이다.
- 배열의 모든 원소가 핵심 필드이면 Arrays.equals 메서드들 중 하나를 사용하자.
- null이 정상 값으로 취급할 때는 OBjects.equals()를 이용해 NPE를 방지하자.
- 최상의 성능을 바란다면 다를 가능성이 더 크거나 비교하는 비용이 싼 필드를 먼저 비교하자.
- 동기화용 락 필드 같이 객체의 논리적 상태와 관련 없는 필드는 비굫사면 안된다.
- equals를 재정의할 땐 hashCode도 반드시 재정의 하자
- Object 타입을 매개변수로 받는 equals 메서드는 선언하지 말자.

## 결론

- equals는 필요할 때만 재정의 하자.
- equals를 다 구현했다면 대칭적인가? 추이성이 있는가? 일관적인가?를 자문해보자.

## 참고자료
[이펙티브 자바 3판](https://www.aladin.co.kr/shop/wproduct.aspx?ItemId=171196410)
