# item 40. @Override 애너테이션을 일관되게 사용하라

# 한 줄 정리

추상메소드를 구현할 때를 제외하고는 `@Override`를 사용하라!

# 왜 그래야 할까?

```java
public class Bigram {

  private final char first;
  private final char second;

  public Bigram(char first, char second) {
    this.first = first;
    this.second = second;
  }

  public boolean equals(Bigram b) {
    return b.first == first && b.second == second;
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
    
    System.out.println(s.size());
  }
}
```

이 코드를 실행하면 뭐가 나올까?

> Set을 사용했으니까 aa ~ zz까지 26이 나오겠군!

![Untitled](https://user-images.githubusercontent.com/42836576/108953573-c8c9fd80-76ae-11eb-98d9-03e260109123.png)

## 왜 260이 나올까?

```java
// 뭔가 이상한 equals
public boolean equals(Bigram b) {
  return b.first == first && b.second == second;
}
```

사실은 equlas는 오버라이딩이 아닌, 오버로딩을 하고 있었다! (인자가 Object가 아닌, Bigram임)

![Untitled 1](https://user-images.githubusercontent.com/42836576/108953576-c9fb2a80-76ae-11eb-8201-def48937c34b.png)

만약 `@Override`를 추가했으면 컴파일 단계에서부터 오류를 잡을 수 있었다!

```java
// 정상적인 equals
@Override
public boolean equals(Object o) {
    if (!(o instanceof Bigram))
        return false;
    Bigram b = (Bigram) o;
    return b.first == first && b.second == second;
}
```

⇒ 즉, 이런 실수를 방지하기 위해서는 항상 `@Override`를 다는 것이 좋다.

# 예외 사항

**추상메소드를 구현할 때를 제외**하고는 `@Override`를 사용하라!

```java
public abstract class Animal {
  public abstract void eat();
}
```

```java
public class Dog extends Animal {

  // 이번에도 오버로딩을 해보자!
  public void eat(String food) {

  }
}
```

![Untitled 2](https://user-images.githubusercontent.com/42836576/108953578-cc5d8480-76ae-11eb-8f8b-b42ee6924897.png)

추상메서드를 구현하지 않으면 오류가 뜨기 때문이다.

그런데 사실 달아도 된다. 굳이 안 달 이유가 없으니 같이 달자 (사실 IDE에서 알아서 잘 달아준다.)

# 다시 한 번 한 줄 정리

IDE에서 잘 달아주는 `@Override`를 굳이 지우지 말자!