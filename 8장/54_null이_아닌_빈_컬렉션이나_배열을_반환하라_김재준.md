# [ITEM 54] null 이 아닌, 빈 컬렉션이나 배열을 반환하라





## 컬렉션이 비었으면 null 을 반환하는 경우 발생하는 문제점

```java
public class Store {

    private final Map<String, List<String>> ItemMap = new HashMap<>(){{
       put("음료",List.of("포카리","이프로","파워에이드"));
       put("과자",List.of());
    }};

    public List<String> getItem(String category){
        List<String> items = ItemMap.get(category);
        return items.isEmpty() ? null : new ArrayList<>(items);
    }

    public static void main(String[] args){
        Store store = new Store();
        List<String> items = store.getItem("과자"); // 참조 값이 null 일 수도 있음

        if(items == null){ // null 체크 로직이 반드시 들어가야함
            System.out.println("텅텅 비어있어요");
        }else{
            items.forEach(System.out::println);
        }
    }
}
```

`null` 을 반환하게된다면 API를 사용하는 클라이언트는 <u>null 체크 로직을 추가로 작성해야 한다</u>.

이러한 추가로직은 서비스를 복잡하게 만들며 실수를 발생시킬 수 있다.



## 따라서 null 이 아닌 빈 배열이나 컬렉션을 반환하자



**컬렉션**

```java
public List<String> getItem(String category){
  	List<String> items = ItemMap.get(category);
  	return items.isEmpty() ? Collections.emptyList() : new ArrayList<>(items);
}
```

빈 컬렉션을 매번 생성하는 것이 성능상 문제가 된다면 
앞에 있는 예제 처럼 `Collections.emptyList()` 을 이용하여 매번 똑같은 <u>빈 불변 컬렉션</u>을 반환하게 하자. 



**배열**

```java
public String[] getItem(String category){
  	List<String> items = ItemMap.get(category);
  	return items.toArray(new String[0]);
}
```

여기 또한 만약 배열을 계속 생성하는것이 성능에 부담이 된다면 미리 길이 0짜리 배열을 선언하여 매번 생성한 배열을 반환하자.

`private static final String[] EMPTY_STRING_ARRAY = new String[0];`



**여기서 주의할점은 배열의 용량을 미리 할당하는 것은 추천하지 않는다(성능이 오히려 떨어진다).**

```java
public String[] getItem(String category){
  	List<String> items = ItemMap.get(category);
  	return items.toArray(new String[items.size()]); // 이처럼 item.size() 를 지정하여 미리 할당하지 말라는 이야기입니다.
}
```

왜 이러한 행위가 성능을 오히려 떨어트릴까요?.. 저도 자세히는 모르지만

![image-20210626234413508](https://tva1.sinaimg.cn/large/008i3skNgy1grw1v87u7ej30jw05naan.jpg)

`List.toArray()` 메서드 같은경우 주어진 배열의 크기가 적으면 새로운 배열을 알아서 만들어서 원소를 할당하고 배열크기가 충분하다면  주어진 배열에 원소를 담아 반환합니다.

따라서 메서드 내부에서 최적의 배열을 만들기 때문에 굳이 특정 용량을 지정하여 사용하지말라는 이야기 같습니다. 



## 결론

null 을 반환한다해서 성능이 좋아지거나 하지않고 서비스 로직을 복잡하게 만든다.

따라서 빈 배열이나 컬렉션을 반환하자.