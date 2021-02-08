# 요약
#### public 클래스의 가변 필드를 직접 노출하지 않도록, 필드를 모두 private으로 바꾸고 public 접근자(getter)를 추가하자.

# 예시0
#### 인스턴스 필드를 모아둔 것 외에는 목적이 없는 퇴보한 클래스
~~~java
class Point {
    public double x;
    public double y;
}
~~~

# 예시1
~~~java
class Point {
    private double x;
    private double y;
    
    public Point(double x, double y) {
        this.x = x;
        this.y = y;
    }
    
    public double getX() {return x;}
    public double getY() {return y;}
}
~~~


# 예시2
~~~java
import lombok.Getter;
import lombok.NoArgsConstructor;

import javax.persistence.*;

@Getter //접근
@NoArgsConstructor
@Entity
public class Product extends Timestamped{

    @GeneratedValue(strategy = GenerationType.AUTO)
    @Id
    private Long id;

    @Column(nullable = false)
    private String title;

    @Column(nullable = false)
    private String image;

    @Column(nullable = false)
    private String link;

    @Column(nullable = false)
    private int lprice;

    @Column(nullable = false)
    private int myprice;

    public Product(ProductRequestDto requestDto) {
        this.title = requestDto.getTitle();
        this.link = requestDto.getLink();
        this.lprice = requestDto.getLprice();
        this.image = requestDto.getImage();
        this.myprice = 0;
    } 

    public void updateByItemDto (ItemDto itemDto) {
        this.lprice = itemDto.getLprice();
    }

    public void update(ProductMypriceRequestDto requestDto) {
        this.myprice = requestDto.getMyprice();
    }
}
~~~

# 문제점
- 필드에 직접 접근 가능 (캡슐화X)
- API를 수정하지 않고는 내부표현 변경 불가
- 불변식 보장 불가
- 외부에서 필드 접근 시 부수 작업 수행 불가

# 예외
- packge-private
- private 중첩 클래스
#### 각각 패키지/클래스 내부에서만 동작하는 코드이기 때문에 데이터 필드가 노출되어도 무관 (오히려 접근자 방식보다 훨씬 깔끔)


<code>private</code> 멤버를 선언한 클래스 내부에서만 접근 가능

<code>package-private</code> 멤버가 소속된 패키지 내부의 모든 클래스에서 접근 가능

<code>protected</code> package-private의 접근 범위 포함. 멤버를 선언한 클래스의 하위 클래스에서도 접근 가능

<code>public</code> 모든 곳에서 접근 가능

# 기존 사례
#### java.awt.package의 Point와 Dimension 클래스 (아이템67)
- 필드를 외부로 직접 노출
- 불변성을 보장할 수 없음

# 불변식 예시

#### 문제점
- API를 변경하지 않고는 내부표현 변경 불가
- 외부에서 필드 접근 시 부수 작업 수행 불가
~~~java
//각 인스턴스가 유효한 시간을 표현함을 보장 (final)
public final class Time {
    private static final int HOURS_PER_DAY = 24;
    private static final int MINUTES_PER_HOURS = 60;

    public final int hour;
    public final int minute;

    public Time(int hour, int minute) {
        if (hour < 0 || hour >= HOURS_PER_DAY)
            throw new IllegalArgumentException("시간: " + hour);
        if (minute < 0 || minute >= MINUTES_PER_HOURS)
            throw new IllegalArgumentException(": " + minute);
        this.hour = hour;
        this.minute = minute;
    }
    //..생략
}
~~~

>#핵심정리
>#### public 클래스는 절대 가변 필드를 직접 노출하지 않도록 한다. 
>#### 불변 필드라면 노출해도 위험이 줄지언정 완전 안심은 할 수 없다.
>#### 예외적으로 package-private, private 중첩 클래스에서는 불변/가변 여부와 관계없이 필드를 노출하는 편이 나은 경우가 있다.



출처: 이펙티브 자바 3판, https://velog.io/@yebink/이펙티브-자바-4장.-클래스와-인터페이스