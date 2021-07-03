# Item.58 전통적인 for 문보다는 for-each문을 사용하라
## 1. 핵심 정리
- 전통적인 for 문과 비교했을 때 for-each 문은 명료, 유연하고 버그를 예방해준다.
- 성능 저하도 없기 때문에 가능한 모든 곳에서 for 문이 아닌 for-each 문을 사용하자.

<br>

## 2. for-each 를 사용하는 하는 이유
#### 전통적인 for문 
- 예제 코드
    ```java
    for (Iterator<String> iter = names.iterator(); iter.hasNext()) {
        String name = iter.next();
    }
        
    for (int i = 0; i < names.length; i++) {
        // ...
    }
    ```

- 전통적인 for 문은 Iterator와 인덱스 변수는 코드르 지져분하게 만들 뿐이다.

<br>

#### for-each 문
- 정식이름은 enhanced for statement 이다.
- 반복 대상이 컬렉션이든 배열이든, for-each 문을 사용해도 속도는 그대로다.
- 왜냐하면 for-each 문이 만들어내는 코드는 사람이 손으로 최적화한 것과 같기 때문이다.


- 예시
    ```java
    for (String name : names) {
        // ...
    }
    ```

<br>  

## 3. for-each 를 사용하지 못하는 상황 
#### (1) 파괴적인 필터링 (destructive filtering)
- 컬렉션을 순회하면서 선택된 원소를 제거하고 싶다면 Iterator의 remove() 를 호출해야한다.
    ```java
    List<String> names = new ArrayList<>();
    names.add("A");
    names.add("B");
    names.add("C");
    names.add("D");
          
    Iterator<String> i = names.iterator();
          
    while(i.hasNext()) {
        String e = i.next();
        if (e.startsWith("A")) {
            i.remove();
        }
    }
    ```
  

- 자바 8 이상부터는 Collection의 removeIf() 를 사용할 수 있다.
    ```java
    List<String> names = new ArrayList<>();
    names.add("A");
    names.add("B");
    names.add("C");
    names.add("D");
  
    names.removeIf(e -> e.startsWith("A"));
    ```


#### (2) 변형 (transforming)
- 리스트나 배열을 순회하면서 그 원소의 값 일부 혹은 전체를 교체해야 한다면 Iterator 나 index 를 사용해야 한다.
    ```java
    // index 를 컨트롤 할 수 없다. 
    for (String name : names) {
        // ...
    }
  
    // index 변수를 선언한다.
    int index = 0;
    for (String name : names) {
        // ...
        index ++;
    }
    ```

#### (3) 병렬 반복 (parallel iteration)
- 여러 컬렉션을 병렬로 순회해야 한다면 각각의 Iterator와 Index 변수를 사용해 엄격하고 명시적으로 제어해야 한다.
    ```java
    for (Iterator<Suit> i = suits.iterator(); i.hasNext(); ) {
        for (Iterator<Rank> j = ranks.iterator(); j.hasNext(); ) {
            deck.add(new Card(i.next(), j.next()));  // i가 j 변수 for문 영역 내에서 계속 호출되고 있다.
        }
    }
    ```
  
- 아래 코드 처럼 Iterator 를 제어 해야한다.
    ```java
    for (Iterator<Suit> i = suits.iterator(); i.hasNext(); ) {
        Suit suit = i.next(); // i 변수 for 문 영역 내에서만 next() 호출
        for (Iterator<Rank> j = ranks.iterator(); j.hasNext(); ) {
            deck.add(new Card(suit, j.next()));  
        }
    }
    ```