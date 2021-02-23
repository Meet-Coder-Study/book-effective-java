# 아이템 40. @Override 애너테이션을 일관되게 사용하라.

## @Override

- 자바에서 기본적으로 제공하는 애너테이션 !
- 상위 타입의 메서드를 재정의했음을 의미합니다.

@Override 애너테이션을 일관되게 사용하면 악명 높은 버그를 예방할 수 있습니다.

- 즉, 일관되게 사용하지 않으면 악명 높은 버그를 만들어 낼 수 있다는 것이죠.

## @Override를 사용하지 않은 경우

```java
public class Bigram {
    private final char first;
    private final char second;

    public Bigram(char first, char second) {
        this.first = first;
        this.second = second;
    }

    public boolean equals(Bigram bigram) {
        return bigram.first == this.first &&
                bigram.second == this.second;
    }

    public int hashCode() {
        return 31 * first + second;
    }

    public static void main(String[] args) {
        Set<Bigram> s = new HashSet<>();
        for (int i = 0; i < 10; i++) {
            for (char ch = 'a'; ch <= 'z'; ch++) {
                s.add(new Bigram(ch, ch));
            }
        }

        System.out.println(s.size()); //260
    }
}
```

26이 나와야할 것 같은데, 260이 출력됩니다 ⇒ 무언가 잘못되었다는 것을 인식 ! 어디인지 찾아다녀야 합니다.

- `equals` 는 `Object` 를 파라미터로 갖는데, `Bigram` 으로 받은 문제
    - Overriding이 아닌 Overloading이 된 것 !
- 이 문제를 통해 `Set` 에서 `==` 연산 시 `equals(Object)` 를 호출하게 됩니다.
    - `Bigram` 에서 같은 값을 가지는 값이더라도 다른 객체(참조)이므로 다른 값으로 인식해 의도한 목적대로 `Set` 이 수행되지 않는 문제가 발생합니다.

문제를 해결하기 위해선 `@Override` 를 사용해야 합니다.

```java
@Override
public boolean equals(Bigram bigram) {
    return bigram.first == this.first &&
        bigram.second == this.second;
}
```

- 시그니쳐가 다르므로 컴파일 단에서 Overriding이 잘못되었음을 알려주게 됩니다.
- 이걸 다시 수정하게 되면 다음과 같습니다.

```java
@Override
public boolean equals(Object bigram) {
    if(!(o instanceof Bigram)) {
        return false;
    }
    Bigram b = (Bigram) bigram;
    return b.first == this.first &&
        b.second == this.second;
}
```

---

앞선 문제를 방지하기 위해, **상위 클래스의 메서드를 재정의하는 모든 메서드에 `@Overriding` 애너테이션을 달아줍시다 !**

- 오타, 실수 등을 IDE 및 컴파일 단에서 잡아낼 수 있습니다. ⇒ 문제를 찾기 위한 시간을 단축시켜 줍니다.

구체 클래스에서 상위 추상 메서드를 재정의한 경우엔 **달지 않아도 되나, 단다고 해서 해로울 것도 없습니다.**

- `@Override`를 다는 습관을 들이면 시그니처가 올바른지 확인할 수 있습니다.
- [개인적인 생각] 팀의 룰을 정해놓고 일관성 있게 작성하는 것이 좋을 것 같습니다.