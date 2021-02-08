## Item 15 클래스와 멤버의 접근 권한을 최소화하라
  - 어설프게 설계된 컴포넌트와 잘 설계된 컴포넌트의 가장 큰 차이는 바로 **클래스 내부 데이터와 내부 구현 정보를 외부 컴포넌트로부터 얼마나 잘 숨겼느냐다.**
  - 잘 설계된 컴포넌트는 모든 내부 구현을 완벽히 숨겨, 구현과 API를 깔끔히 분리한다.
  - 오직 API를 통해서만 다른 컴포넌트와 소통하며 서로의 내부 동작 방식에는 전혀 개의치 않는다.
  - 정보 은닉, 캡슐화 라고 하는 이 개념은 **소프트웨어 설계의 근간이 되는 원리다.**
  

### ① 잘못된 설계
```java
class Man {
  public String jumin;
}
```

### ② 잘못된 설계
```java
public class Man {
    private String jumin;

    public String getJumin() {
        return jumin;
    }

    public void setJumin(String jumin) {
        this.jumin = jumin;
    }
}
```

### 올바른 설계
```java
public class Man {
    private final String jumin;

    public String getJumin() {
        return jumin;
    }

    public Man(String jumin) {
        this.jumin = jumin;
    }
}

```

### 정보 은닉의 장점
  - 시스템 개발 속도 ↑
  - 시스템 관리 비용 ↓
  - 성능 최적화에 도움
    - 다른 컴포넌트에 영향을 주지 않고 해당 컴포넌트만 최적화 할 수 있기 때문
  - 소프트웨어의 재사용성 ↑

<br>

### 자바는 정보 은닉을 위한 다양한 장치를 제공한다.
  - 클래스, 인터페이스, 멤버의 접근성을 명시함
  - 각 요소의 접근성은 접근 제한자(private, default, protected, public)로 정해진다.
  - 기본 원칙은 **모든 클래스와 멤버의 접근성을 가능한 한 좁혀야 한다.**
  - public일 필요가 없는 클래스의 접근 수준을 default 클래스로 좁혀야 한다.
  - *private*: 멤버를 선언한 톱레벨 클래스에만 접근 가능
  - *package-private(default)*: 패키지 안의 모든 클래스에서 접근 가능
  - *protected*: 동일 패키지 및 상속받은 하위 클래스에서 접근 가능
  - *public*: 모든 곳에서 접근 가능
  - **public 클래스의 인스턴스 필드는 되도록 public이 아니어야 한다.(아이템 16)**
  - 필요에 따라 public static final 상수 필드는 공개해도 좋다.
  
  ![캡쳐](https://user-images.githubusercontent.com/50076031/105838013-db4f0980-6012-11eb-9132-0986cd2c2a30.PNG)

  
```java
class Point {
  private double x;
  private double y;
  ...
}
```

<br>

### 객체 모델링 방법
  - 초기에는 객체 모델을 수정이 불가능 하도록 Immutable 상태로 시작
  - 객체 모델에 필요한 행위가 무엇인지 고민하고, 점진적으로 Immutable 상태를 제거
  - (Item 10~12)Object의 equals, hashcode, toString 등의 메서드 구현은 필요에 따라 꼼꼼하게 할 것
  - (Item 11)equals가 참이라면 hashcode의 결과도 참이어야 함
  - 불필요한 생성자는 노출 X
  - (Item 2)생성 인자 갯수가 많아질 경우 -> 빌더 패턴 고려
  
```java
public class Post {
  private final Long seq;               // PK(불변)
  private final Id<User, Long> userId;  // USER(불변)
  private final String contents;        // 제목(변함)
  private final int likes;              // 좋아요 갯수(변함)
  private final boolean likesOfMe;      // 좋아요 여부(변함)
  private final int comments;           // 댓글 수(변함)
  private final Writer writer;          // 글쓴이(불변)
  private final LocalDateTime createAt; // 작성일자(불변)
```

<br>

```java
public class Post {
  private final Long seq;
  private final Id<User, Long> userId;
  private String contents;
  private int likes;
  private boolean likesOfMe;
  private int comments;
  private final Writer writer;
  private final LocalDateTime createAt;
  
  @Override
  public boolean equals(Object o) {
    if (this == o) return true;
    if (o == null || getClass() != o.getClass()) return false;
    Post post = (Post) o;
    return Objects.equals(seq, post.seq);
  }

  @Override
  public int hashCode() {
    return Objects.hash(seq);
  }

  @Override
  public String toString() {
    return new ToStringBuilder(this, ToStringStyle.SHORT_PREFIX_STYLE)
      .append("seq", seq)
      .append("userId", userId)
      .append("contents", contents)
      .append("likes", likes)
      .append("likesOfMe", likesOfMe)
      .append("comments", comments)
      .append("writer", writer)
      .append("createAt", createAt)
      .toString();
  }
```
  
<br>

### 핵심 정리
  - 프로그램 요소의 접근성은 가능한 한 **최소한**으로 하라.
  - 꼭 필요한 것만 골라 최소한의 public API를 설계하자.
  - public 클래스는 상수용 public static final 필드 외에는 어떠한 public 필드도 가져서는 안된다!
  
