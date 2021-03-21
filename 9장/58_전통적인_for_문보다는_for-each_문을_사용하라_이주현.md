## Item 58 전통적인 for 문보다는 for-each 문을 사용하라.

```java
List<String> fruits = List.of("Apple", "Orange", "Melon", "Lemon", "Banana");
int[] numbers = {1, 2, 3, 4, 5};

for (Iterator<String> iter = fruits.iterator(); iter.hasNext()) {
    // ...
}

for (int i = 0; i < numbers.length; i++) {
    // ...
}

```

  - 위 관용구들은 while 문보다는 낫지만 가장 좋은 방법은 아니다.
  - 반복자와 인덱스 변수는 모두 코드를 지저분하게 할 뿐 정말 필요한 것은 **원소들**뿐이다.
  - 더군다나 위와 같이 요소의 종류, 횟수가 늘어나면 오류가 생길 가능성이 높아진다.


<br>

```java
List<String> fruits = List.of("Apple", "Orange", "Melon", "Lemon", "Banana");
int[] numbers = {1, 2, 3, 4, 5};

for (String fruit : fruits) {
    // ...
}

for (int number : numbers) {
    // ...
}
```

  - 위의 문제는 **for-each문(향상된 for 문)** 을 사용하면 모두 해결된다.
  - 반복자와 인덱스변수를 사용하지 않으니 코드가 깔끔해지고, 오류가 날 일도 없다.
  - 하나의 관용구로 컬렉션과 배열을 모두 처리할 수 있어서 어떤 컨테이너를 다루는지 신경 쓰지 않아도 된다.

<br>

## 주사위를 두 번 굴렸을 때 나올 수 있는 모든 경우의 수 출력
```java
import java.util.Collection;
import java.util.EnumSet;
import java.util.Iterator;

public class ForEachEx {
    enum Face {ONE, TWO, THREE, FOUR, FIVE, SIX}

    public static void main(String[] args) {
        Collection<Face> faces = EnumSet.allOf(Face.class);

        for (Iterator<Face> i = faces.iterator(); i.hasNext(); ) {
            for (Iterator<Face> j = faces.iterator(); j.hasNext(); ) {
                System.out.print(i.next() + " " + j.next() + ", ");
            }
            System.out.println();
        }

        System.out.println();

        for (Face face : faces) {
            for (Face face1 : faces) {
                System.out.print(face + " " + face1 + ", ");
            }
            System.out.println();
        }
    }
}
```

```html
ONE ONE, TWO TWO, THREE THREE, FOUR FOUR, FIVE FIVE, SIX SIX, 

ONE ONE, ONE TWO, ONE THREE, ONE FOUR, ONE FIVE, ONE SIX, 
TWO ONE, TWO TWO, TWO THREE, TWO FOUR, TWO FIVE, TWO SIX, 
THREE ONE, THREE TWO, THREE THREE, THREE FOUR, THREE FIVE, THREE SIX, 
FOUR ONE, FOUR TWO, FOUR THREE, FOUR FOUR, FOUR FIVE, FOUR SIX, 
FIVE ONE, FIVE TWO, FIVE THREE, FIVE FOUR, FIVE FIVE, FIVE SIX, 
SIX ONE, SIX TWO, SIX THREE, SIX FOUR, SIX FIVE, SIX SIX, 
```

<br>

## for-each 문을 사용할 수 없는 상황
  - 파괴적인 필터링(destructive filtering)
    - 컬렉션을 순회하면서 선택된 원소를 제거해야 할 때
  - 변형(transforming)
    - 리스트나 배열을 순회하면서 그 원소의 값 일부 혹은 전체를 교체해야 할 때
  - 병렬 반복(parallel iteration)
    - 여러 컬렉션을 병렬로 순회해야 할 때

```java
List<String> list = List.of("A", "B", "C", "D");

// 파괴적인 필터링
for (Iterator<String> iterator = list.iterator(); iterator.hasNext(); ) {
    String element = iterator.next();
    if(element.equals("A")) {
        iterator.remove();
    }
}

list.removeIf(element -> element.equals("A"));

// 변형
String[] arr = new String[]{"A", "B", "C"};

for (int i = 0; i < arr.length; i++) {
    String s = arr[i];
    if ("a".equals(s)) {
        arr[i] = "d";
    }
}

// 병렬 반복
enum Suit { CLUB, DIAMOND, HEART, SPADE }
enum Rank { ACE, DEUCE, THREE, FOUR, ... }

static List<Suit> suits = Arrays.asList(Suit.values());
static List<Rank> ranks = Arrays.asList(Rank.values());

List<Card> deck = new ArrayList<>();

for (Iterator<Suit> i = suits.iterator(); i.hasNext(); ) {
    for (Iterator<Rank> j = ranks.iterator(); j.hasNext(); ) {
        // i.next() 메서드가 '숫자(suit) 하나당' 한 번씩만 불려야 하는데
        // 카드(Rank) 하나당 불리고 있다.
        deck.add(new Card(i.next(), j.next()));
    }
}
```
  
<br>

## 정리
  - 전통적인 for 문과 비교했을 때 for-each 문은 명료하고, 유연하고, 버그를 예방해준다.
  - 성능 저하도 없다.
  - 따라서 가능한 한 모든 곳에서 for 문이 아닌 for-each 문을 사용하자.
