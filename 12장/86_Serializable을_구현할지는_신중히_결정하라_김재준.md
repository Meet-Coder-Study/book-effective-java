**ITEM 86**

# Serializable 을 구현할지는 신중히 결정하라

-----

어떤 클래스의 인스턴스를 직렬화 하기위해선 Serializable 을 impl 하게 된다면 쉽게 사용 할 수 있다.

하지만 지원하기는 쉬워보이지만 길게 보면 아주 값 비싼 일이다.





## Serializable을 구현하면 릴리스한 뒤에는 수정하기 어렵다.

클래스가 Serializable을 구현하면 직렬화된 바이트 스트림 인코딩(직렬화형태)도 하나의 공개 API가 된다.

그래서 이 클래스가 널리 퍼진다면 그 직렬화 형태도 영원히 지원해야 하는 것이다.

즉 클래스의 private 과 package-private 인스턴스 필드마저 API로 공개하는 꼴이 된다.



뒤늦게 클래스 내부 구현을 손보면 원래의 직렬화 형태와 달라지게 된다.

한쪽은 구버전 인스턴스를 직렬화하고 다른 쪽은 신버전 클래스로 역질렬화한다면 실패를 맛볼 것이다.



직렬화 예제

```java
@Test
@DisplayName("객체 직렬화")
void serializable() throws IOException {
    Member member = new Member("KJJ",26,"서울 어딘가");

    byte[] serializedMember;
    try (ByteArrayOutputStream baos = new ByteArrayOutputStream()) {
        try (ObjectOutputStream oos = new ObjectOutputStream(baos)) {
            oos.writeObject(member);
            // serializedMember -> 직렬화된 member 객체
            serializedMember = baos.toByteArray();
        }
    }
    // 바이트 배열로 생성된 직렬화 데이터를 base64로 변환
    System.out.println(Base64.getEncoder().encodeToString(serializedMember));
}
```



결과

![image-20210407175802931](/Users/kimjajan/Library/Application Support/typora-user-images/image-20210407175802931.png)



역직렬화 예제

```java
@Test
@DisplayName("역 직렬화")
void deserializable() throws IOException, ClassNotFoundException {
    String base64Member = getSerializedMember(); // 앞에서 직렬화한 값을 바인딩해줘야합니다.
    byte[] serializedMember = Base64.getDecoder().decode(base64Member);
    Member member;
    try (ByteArrayInputStream bais = new ByteArrayInputStream(serializedMember)) {
        try (ObjectInputStream ois = new ObjectInputStream(bais)) {
            // 역직렬화된 Member 객체를 읽어온다.
            Object objectMember = ois.readObject();
            member = (Member) objectMember;
        }
    }
    assertThat(member.getName()).isEqualTo("KJJ");
    assertThat(member.getAge()).isEqualTo(26);
    assertThat(member.getAddress()).isEqualTo("서울 어딘가");
}
```



현재까지 초기버전 릴리즈 상태에서는 행복하게 모든 테스트가 통과할 것이다.

하지만 추 후 Member 클래스에 필드가 추가되거나 타입이 변경된다면 어떻게 될까?

```java
public class Member implements Serializable {

    private String name;
    private int age;
    private String address;
    // 이메일 추가
    private String email;
    
    // getter,생성자 생략 
}
```



기존 직렬화된 Member 객체를 역직렬화 했을때 우리는 email 은 null  로 설정되고 나머지 필드들은 제대로 역직렬화 되는 것을 기대할 것이다.

하지만 다음과 같은 오류를 만난다.

![image-20210407181615217](/Users/kimjajan/Library/Application Support/typora-user-images/image-20210407181615217.png)

해당 예외가 발생한 이유는 모든 직렬화된 클래스는 serialVersionUID 의 이름으로 고유 식별번호를 부여 받는다.

하지만 serialVersionUID 를 클래스내에 static fianl long 필드로 이번호를 명시하지 않으면 시스템이 런타임에 암호해시 함수(SHA-1) 을 적용해 자동으로 클래스 안에 생성한다.

그래서 나중에 클래스를 수정하게 된다면 serialVersionUID 값도 변하게된다.  따라서 serialVersionUID 을 꼭 명시 해주자.



```java
public class Member implements Serializable {
  	// serialVersionUID 꼭 명시 할 것 !
    private static final long serialVersionUID = 1L;
    private String name;
    private int age;
    private String address;
}
```



추가한뒤 email 을 추가하지 않은 클래스 Member 를 다시 직렬화 후  Member 클래스에 email 을 추가한뒤 다시 역직렬화 테스트를 돌려보자.

![image-20210407182502133](/Users/kimjajan/Library/Application Support/typora-user-images/image-20210407182502133.png)

그럼 다음과 같이 성공 할 것이다.



그렇다면 다른 문제점은 없는걸까?

만약 사람이 int 값이 허용하는 값을 넘어선 나이를 살 수 있게되었다고 가정하자

그러면 우리 프로그래머들은 오버플로우를 피하기위해 타입을 변경 할 것이다. 한번 해보자.

```java
public class Member implements Serializable {
    private static final long serialVersionUID = 1L;
    private String name;
    //type 변경
    private long age;
    private String address;
 }
```



바로 다음과 같은 오류를 만날 것이다.

![image-20210407183249822](/Users/kimjajan/Library/Application Support/typora-user-images/image-20210407183249822.png)

이 처럼 초기 버전의에서 Serializable  객체가 구현하고 있다면 추후 버전에서 이전 버전에 영향없이 소스코드 수정은 매우 어렵다.(미래를 예측할 수 없기 때문에) 







## Serializable 구현은 버그와 보안 구멍이 생길 위험이 높아진다.



객체는 생성자를 사용해 만드는 게 기본이다. 즉 직렬화는 언어의 기본 메커니즘을 우회하는 객체생성 기법이다.

기본 방식을 따르든 재정의해 사용하든, 역직렬화는 일반 생성자이 문제가 그대로 적용되는 '숨은 생성자'다.

이 생성자는 전면에 드러나지 않으므로 "생성자에서 구축한 불변식을 모두 보장해야하고 생성 도중 공격자가 객체 내부를 들여다 볼수 없도록 해야한다" 를

떠올리기는 쉽지 않다. 

즉 기본 역직렬화를 사용하면 불변식 깨짐과 허가되지 않은 접근에 쉽게 노출된다는 뜻이다.



예제

만약 Member 가 생성될때 26 살이면 오류를 던지고 싶다. 

```java
public class Member implements Serializable {
    private static final long serialVersionUID = 1L;
    private String name;
    private int age;
    private String address;
    private String email;

    public Member(String name, int age, String address) {
        if(age ==26){
            throw new IllegalArgumentException();
        }
        this.name = name;
        this.age = age;
        this.address = address;
    }
}    
```

기존 생성자 방식으로 해당 Member 객체를 생성한다면 당연히 오류를 던질 것이다.

하지만 직렬화를 통한 객체생성은 불변식을 무시하고 해당 객체에 26살이 바인딩되어 생성된다.





## Serializable 구현은 해당 클래스의 신버전을 릴리스할 때 테스트할 것이 늘어난다.



직렬화 가능 클래스가 수정되면 신버전인스턴스를 직렬화한 후 구버전으로 역직렬화할 수 있는지, 그리고 그 반대도 가능한지를 검사해야한다.

따라서 테스트해야 할 향이 직렬화 가능 클래스의 수와 릴리스 횟수에 비례해 증가한다.(커스텀 직렬화 형태를 잘 설계해놨다면 이러한 테스트 부담을 줄일수 있다(아이템 87,90))



## Serializable 구현 여부는 가볍게 결정할 사안이 아니다.

단 객체를 전송하거나 저장할 때 자바 직렬화를 이용하는 프레임워크용 으로 만든 클래스라면 선택의 여지가 없다.

Serializable 을 반드시 구현해야 하는 다른 클래스의 컴포넌트로 쓰일 클래스도 마찬가지다.



하지만 Serializable 구현에 따른 비용이 적지 않으니, 클래스를 설계 할때마다 그이득과 비용을 잘 저울질해야 한다.

역사적으로 BigInteger 와 Instant 같은 '값'  클래스와 컬렉션 클래스들은 Serializable을 구현하고 스레드 풀처럼 '동작' 하는 객체를 표현하는 클래스 들은 대부분

Serializable 을 구현하지 않았다.





## 상속용 클래스가 Serializable 을 지원하지 않는 경우



상속용 클래스가 직렬화를 지원 하지 않으면 그 하위 클래스에서 직렬화를 지원하려 할 떄 부담이 늘어난다.

보통 이런 클래스를 역직렬화하려면 그 상위 클래스는 매개변수가 없는 생성자를 제공해야한다.

만약 지원하지 않는다면 하위 클래스는 어쩔 수 없이 직렬화 프록시 패턴(아이템90) 을 사용해야 한다.





## 내부 클래스 는 직렬화를 구현하지 말아야 한다.

내부 클래스에는 바깥 인스턴스의 참조와 유효 범위 안의 지역변수 값들을 저장하기 위해 컴파일러가 생성한 필드들이 자동으로 추가된다.

익명 클래스와 지역 클래스의 이름을 짓는 규칙이 언어 명세에 나와 있지 않듯, 이 필드들이 클래스 정의에 어떻게 추가되는지도 정의도지 않았다.

다시 말해 내부 클래스에 대한 기본 직렬화 형태는 분명하지 않다.

단 정적 맴버 클래스는 Serializable 을 구현해도 된다.