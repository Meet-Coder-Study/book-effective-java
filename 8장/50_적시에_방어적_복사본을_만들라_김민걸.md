# *ITEM 50)* 적시에 방어적 복사본을 만들라

## *클라이언트가 우리의 불변식을 깨뜨리려 혈안이 되어 있다고 가정하고 방어적으로 프로그래밍해야 한다 !*

불변식 (불변속성) : 어떤 객체의 상태가 프로그래머의 의도에 맞게 잘 정의되어 있다고 판단할 수 있는 기준을 제공하는 속성

<br>
<br>

### 불변식을 깨뜨리는 경우 1)  객체의 허락없이 외부에서 생성자로 객체 내부의 값을 변경하는 경우

```java
public final class Period {
    private final Date start;
    private final Date end;

    public Period(Date start, Date end) {
        if (start.compareTo(end) > 0) {
            throw new IllegalStateException(start + "가 " + end + " 보다 늦을 수 없습니다.");
        }
        this.start = start;
        this.end = end;
    }

    public Date getStart() {
        return start;
    }

    public Date getEnd() {
        return end;
    }
}
```
 - 불변 클래스를 위해 final 클래스로 만들고,
 - 모든 필드를 final 로 만들었다.
 - 그리고 시작 시간이 끝 시간보다 최근인 경우는 예외로 막는다.


```java
SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy/MM/dd");

Date start = new Date();
Date end = new Date();
Period p = new Period(start, end);
end.setTime(1000000000);

System.out.println("시작시간 : " + dateFormat.format(p.getStart()) + "\n끝시간 : " + dateFormat.format(p.getEnd()));
```
 
![이미지](period.png)
 - 하지만 Date 클래스가 가변클래스라는 이유로, 위의 불변식은 쉽게 깨진다.
 - 날짜를 표현할 떄 JAVA 8 버전에 생긴 Instant 불변 클래스를 사용하면 해결할 수 있다.
 - 외에는 LocalDateTime, ZoneDateTime 을 사용할 수 있다.
 
 > *Date 클래스는 낡은 API 이니 사용하지 말자.*
 
 ```java
 public class Date {...}
 
 public final class Instant {...}
 public final class LocalDateTime {...}
 public final class ZonedDateTime {...}
```

<br>
<br>
<br>

### 불변식을 깨뜨리는 경우 2)  객체의 허락없이 외부에서 setter 로 객체 내부의 값을 변경하는 경우
```java
        Date start = new Date();
        Date end = new Date();
        Period p1 = new Period(start, end);
        p1.end().setYear(78);
```


### 이미 개발된 구현에 Date 같은 클래스를 쓰고 있다면? - 매개변수에서 받은 가변 매개변수 각각을 *방어적 복사*하라.

```java
    // 생성자
    public DefensiveCopiedPeriod(Date start, Date end) {
        this.start = new Date(start.getTime());
        this.end = new Date(end.getTime());

        if (start.compareTo(end) > 0) {
            throw new IllegalStateException(start + "가 " + end + " 보다 늦을 수 없습니다.");
        }
    }
```

```java
Date start = new Date();
Date end = new Date();

Period p1 = new Period(start, end);
DefensiveCopiedPeriod p2 = new DefensiveCopiedPeriod(start, end);
end.setTime(1000000000);

System.out.println("<방어적 복사 안함>\n" + "시작시간 : " + dateFormat.format(p1.getStart()) + "\n끝시간 : " + dateFormat.format(p1.getEnd()));
System.out.println("<방어적 복사 함>\n" + "시작시간 : " + dateFormat.format(p2.getStart()) + "\n끝시간 : " + dateFormat.format(p2.getEnd()));
```
![compare](compare.png)

 - 방어적 복사를 하면 생성자로 넘기는 가변객체를 사용하여 인스턴스를 만들지 않고 새로운 객체를 만드므로 위의 공격에서 안전하다.
 
<br>
 
#### 생성자에서 복사와 유효성 검사의 순서 ?
 - 멀티 스레드 환경에서, 원본 객체의 유효성을 검사하는 찰나에 다른 스레드가 원본 객체를 변경할 위험이 있다.
 - **반드시 방어적 복사 후, 유효성 검사하는 순서로 작성해야 한다.**
 - Time-Of-Check/Time-Of-Use 공격 (TOCTOU 공격)

<br>

#### clone() 메서드

1) 생성자에서 새로운 객체로 복사가 아닌 clone() 을 씁니다.
```java
    public DefensiveCopiedPeriod(Date start, Date end) {
        this.start = (Date) start.clone();
        this.end = (Date) end.clone();

        if (start.compareTo(end) > 0) {
            throw new IllegalStateException(start + "가 " + end + " 보다 늦을 수 없습니다.");
        }
    }
```

<br>

2) Date 는 가변 클래스기 때문에, 이를 상속하는 클래스를 만들 수 있습니다.
 - 악의적인 리스트를 만들고, clone 메서드를 오버라이딩하여 객체 참조를 보관합니다.
```java
public class SubDate extends Date implements Cloneable {

    private List<Date> hackersList = new ArrayList<>();

    public List<Date> getHackersList() {
        return hackersList;
    }

    @Override
    public Object clone() {
        Date badDate = (Date) super.clone();
        hackersList.add(badDate);
        return badDate;
    }
}
```

<br>

3) 아래와 같이 악의적으로 만든 하위 클래스를 SubDate 를 넣어서 객체 참조를 얻어올 수 있습니다.
 - 이렇게 되면 얻어온 참조로 불변식을 깨뜨릴 수 있습니다.
 ```java
        Date start2 = new SubDate();
        Date end2 = new SubDate();
        DefensiveCopiedPeriod period = new DefensiveCopiedPeriod(start2, end2);
        List<Date> hackersList = ((SubDate) (period.getEnd())).getHackersList();
        hackersList.get(0).setTime(100000);

        System.out.println("<방어적 복사 함>\n" + "시작시간 : " + dateFormat.format(period.getStart()) + "\n끝시간 : " + dateFormat.format(period.getEnd()));
```

![clone](clone.png)



<br>

##### **매개변수 (이번 예에서는 Date) 가 다른 사람에 의해 확장될 수 있는 타입이면, clone() 으로 방어적 복사를 하면 안된다 !**

##### 생성자와 달리 접근자 메서드에는 clone() 을 사용해도 된다.
 - Period 가 가진 Date 가 생성자에서 들어와서 new Date() 로 만든다.
 - 따라서 확장된 타입이 아닌 Date 임이 확실하기 떄문이다.
 
 
 
 
<br>
<br>

## 방어적 복사의 다른 목적 - 객체가 이후 변경되었을 때, 프로그램이 문제가 생길 수 있는지 검토하라 !

 - 예를 들어 객체가 다른 곳에서 Map 의 Key 값으로 쓰이고 있었다면,
 - 객체가 변경되었을때 그 Map 의 불변식이 깨지고, 문제가 생긴다.


## 복사 비용이 너무 크거나, 클라이언트가 그 요소를 수정할 일이 없다는 것이 확실하다면 ?
 - 방어적 복사를 생략한다.
 - 문서화시에 책임이 클라이언트에 있음을 명시한다.