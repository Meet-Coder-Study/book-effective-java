# [Effective Java] Item 5. 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라

---

### 예시

```java
public class AccountService implements UserDetailsService {

    private final AccountRepository accountRepository;
    private final EmailService emailService;
    private final PasswordEncoder passwordEncoder;
    private final AppProperties appProperties;

    public AccountService(AccountRepository accountRepository, EmailService emailService, PasswordEncoder passwordEncoder, AppProperties appProperties) {
        this.accountRepository = Objects.requireNonNull(accountRepository);
        this.emailService = Objects.requireNonNull(emailService);
        this.passwordEncoder = Objects.requireNonNull(passwordEncoder);
        this.appProperties = Objects.requireNonNull(appProperties);
    }
    ...
}
```

위와 같이 유저의 회원가입이나 로그인, 로그아웃 등의 처리를 담당하는 AccountService가 있을 때, AccountService가 사용하는 서비스나 자원들을 의존 객체를 통해 주입받을 수 있다.
1) AccountRepository: Account에서 사용하는 데이터에 접근하기 위한 클래스
2) EmailService: 유저 가입/패스워드 변경 시 이메일을 전송하는 서비스
3) PasswordEncoder: 유저가 가입/로그인 시 패스워드의 인코딩을 확인하여 올바른 패스워드를 입력했는지 확인하는 서비스
4) AppProperties: 어플리케이션의 속성 정보를 들고 있는 클래스

서비스의 사이즈가 커지면 의존하는 객체들이 늘어나기 쉽다. 따라서 해당 서비스 내에서 다른 서비스나 자원을 사용할 경우 직접  객체들을 생성하여 사용하는 것이 아니라, 의존을 주입받아 사용하는 것이 좋다. 위 예시와 같이 AccountService를 사용할 경우, 사용하는 곳에서 의존 대상 객체들을 넣어준다. 대신 스프링 프레임워크와 같은 의존 객체 주입 프레임워크를 사용한다면 이와 같은 의존 객체 주입을 프레임워크단에서 대신 해주게 된다.

---

## **의존 사용의 잘못된 예시**

많은 클래스가 하나 이상의 자원에 의존한다. 가령 맞춤법 검사기는 사전(dictionary)에 의존하는데, 이런 클래스를 정적 유틸리티 클래스로 구현한 모습을 드물지 않게 볼 수 있다.

### **정적 유틸리티를 잘못 사용한 예 (유연하지 않고 테스트하기 어렵다.)**

```java
public class SpellChecker {
    private static final Lexicon dictionary = ...;

    private SpellChecker() {} // 객체 생성 방지

    public static boolean isValid(String word) { ... }
    public static List<String> suggestions (String typo) { ... }
}
```

### **싱글턴을 잘못 사용한 예 (유연하지 않고 테스트하기 어렵다.)**

```java
public class SpellChecker {
    private final Lexicon dictionary = ...;

    private SpellChecker() {}
    public static SpellChecker INSTANCE = new SpellChecker(...);

    public boolean isValid(String word) { ... }
    public List<String> suggestions (String typo) { ... }
}
```

두 방식 모두 사전을 단 하나만 사용한다고 가정한다는 점에서 그리 훌륭해 보이지 않다. 실전에서는 사전이 언어별로 있을 수도 있고, 특수 어휘용 사전을 별도로 두기도 한다. 심지어 테스트용 사전이 필요할 수 있다. 사전 하나로 이 모든 쓰임에 대응할 수 있기를 바라는 것은 너무 순진한 생각이다.

SpellChecker가 여러 사전을 사용할 수 있도록 만들어보자. 간단히 dictionary 필드에서 final 한정자를 제거하고 다른 사전으로 교체하는 메서드를 추가할 수 있지만, 아쉽게도 이 방식은 어색하고 오류를 내기 쉬우며 멀티쓰레드 환경에서는 쓸 수 없다. `사용하는 자원에 따라 동작이 달라지는 클래스에는 정적 유틸리티 클래스나 싱글턴 방식이 적합하지 않다.`

대신 클래스(SpellChecker)가 여러 자원 인스턴스를 지원해야 하며, 클라이언트가 원하는 자원(dictionary)을 사용해야 한다. 이 조건을 만족하는 간단한 패턴이 있으니, 바로 인스턴스를 생성할 때 생성자에 필요한 자원을 넣어주는 방식이다. 이는 `의존 객체 주입의 한 형태로, 맞춤법 검사기를 생성할 때 의존 객체인 사전을 주입해주면 된다.`

## **의존 사용의 올바른 예시**

### **의존 객체 주입을 통한 자원 사용 (유연하고 테스트 용이성을 높여준다.)**
```java
public class SpellChecker {
    private final Lexicon dictionary;

    public SpellChecker(Lexicon dictionary) {
        this.dictionary = Objects.requireNonNull(dictionary);
    }

    public boolean isValid(String word) { ... }
    public List<String> suggestions(String typo) { ... }
}
```

의존 객체 주입 패턴은 아주 단순하여 수많은 프로그래머가 이 방식에 이름이 있다는 사실도 모른 채 사용해왔다. 예시에서는 dictionary라는 딱 하나의 자원을 사용하지만, `자원이 몇 개든 의존 관계가 어떻든 상관 없이 잘 작동한다.` 또한 `불변`을 보장하여 (같은 자원을 사용하려는) 여러 클라이언트가 의존 객체들을 안심하고 공유할 수 있기도 하다. 의존 객체 주입은 생성자, `정적 팩터리`, `빌더` 모두에 똑같이 응용할 수 있다.

이 패턴의 쓸만한 변형으로, `생성자에 자원 팩터리를 넘겨주는 방식`이 있다. `팩터리란 호출할 때마다 특정 타입의 인스턴스를 반복해서 만들어주는 객체`를 말한다. 즉, 팩터리 메서드 패턴(Factory Method Pattern)을 구현한 것이다. 자바 8에서 소개한 Supplier<T> 인터페이스가 팩터리를 표현한 완벽한 예이다. Supplier<T>를 입력으로 받는 메서드는 일반적으로 `한정적 와일드카드 타입 (bounded wildcard type)`을 사용해 팩터리의 타입 매개변수를 제한해야 한다. `이 방식을 사용해 클라이언트는 자신이 명시한 타입의 하위 타입이라면 무엇이든 생성할 수 있는 팩터리를 넘길 수 있다.` 예컨대 다음 코드는 클라이언트가 제공한 팩터리가 생성한 타일(Tile)들로 구성된 모자이크(Mosaic)를 만드는 메서드다.

```java
Mosaic create(Supplier<? extends Tile> tileFactory) { ... }
```

<details>
<summary>팩터리 메서드 패턴 예시</summary>
<div markdown="1">

```java
abstract class Product {
    public abstract void use();
}
```

```java
class IDCard extends Product {
    private String owner;

    public IDCard(String owner) {
        System.out.println(owner + "의 카드를 만듭니다.");
        this.owner = owner;
    }

    @Override
    public void use() {
        System.out.println(owner + "의 카드를 사용합니다.");
    }

    public String getOwner() {
        return owner;
    }
}
```

```java
abstract class Factory {
    public final Product create(String owner) {
        Product p = createProduct(owner);
        registerProduct(p);
        return p;
    }
    protected abstract Product createProduct(String owner);
    protected abstract void registerProduct(Product p);
}
```

```java
class IDCardFactory extends Factory {
    private List<String> owners = new ArrayList<>();

    @Override
    protected Product createProduct(String owner) {
        return new IDCard(owner);
    }

    @Override
    protected void registerProduct(Product p) {
        owners.add(((IDCard) p).getOwner());
    }

    public List<String> getOwners() {
        return owners;
    }
}
```

```java
Factory factory = new IDCardFactory();
Product card1 = factory.create("홍길동");
Product card2 = factory.create("이순신");
Product card3 = factory.create("강감찬");
card1.use();
card2.use();
card3.use();
```

```
홍길동의 카드를 만듭니다.
이순신의 카드를 만듭니다.
강감찬의 카드를 만듭니다.
홍길동의 카드를 사용합니다.
이순신의 카드를 사용합니다.
강감찬의 카드를 사용합니다.
```
</div>
</details>

의존 객체 주입이 유연성과 테스트 용이성을 개선해주긴 하지만, 의존성이 수천개나 되는 큰 프로젝트에서는 코드를 어지럽게 만들기도 한다. 대거(Dagger), 주스(Guice), 스프링(Spring) 같은 의존 객체 주입 프레임워크를 사용하면 이런 어질러짐을 해소할 수 있다. 프레임워크 활용법은 이 책에서 다를 주제는 아니지만, 이들 프레임워크는 의존 객체를 직접 주입하도록 설계된 API를 알맞게 응용해 사용하고 있음을 언급해둔다.

---

### 핵심정리

클래스가 내부적으로 하나 이상의 자원에 의존하고, 그 자원이 클래스 동작에 영향을 준다면 싱글턴과 정적 유틸리티 클래스는 사용하지 않는 것이 좋다. 이 자원들을 클래스가 직접 만들게 해서도 안된다. 대신 필요한 자원을 (혹은 그 자원을 만들어주는 팩터리를) 생성자에 (혹은 정적 팩터리나 빌더에) 넘겨주자. 의존 객체 주입이라 하는 이 기법은 클래스의 유연성, 재사용성, 테스트 용이성을 기막히게 개선해준다.

---

### 관련 자료

1. 불변 - item 17
2. 정적 팩터리 - item 1
3. 빌더 - item 2
4. 팩터리 메서드 패턴(Factory Method Pattern) - gramma 95
5. 한정적 와일드카드 타입 (bounded wildcard type) - item 31

### 참고 자료
- Effective Java 3/E