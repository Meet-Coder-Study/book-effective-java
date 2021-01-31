## 아이템 16 public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라
  - 예제 코드 및 설명

```java
// 이처럼 퇴보한 클래스는 public이어서는 안 된다!
// 이런 클래스는 데이터 필드에 직접 접근할 수 있으니 캡슐화의 이점을 제공하지 못한다.
// 캡슐화란 : 쉽게 말하면 중요한 데이터를 쉽게 바꾸지 못하도록 하는 것.
// API를 수정하지 않고는 내부 표현을 바꿀 수 없고, 불변식을 보장할 수 없으며, 외부에서 필드에 접근할 때 부수 작업을 수행할 수도 없다.

class Point {
    public double x;
    public double y;
}
```

```java
// 접근자와 변경자(mutator) 메서드를 활용해 데이터를 캡슐화한다.
// 패키지 바깥에서 접근할 수 있는 클래스라면 접근자를 제공함으로써 클래스 내부 표현 방식을 언제든 바꿀 수 있는 유연성을 얻을 수 있다.
class Point {
    private double x;
    private double y;
    
    public Point(double x, double y) {
        this.x = x;
        this.y = y;
    }

    public double getX() { return x; }
    public double getY() { return y; }

    public void setX(double x) { this.x = x; }
    public void setY(double y) { this.y = y; }


}
```

```java
// 롬복(lombok) 적용(롬복 플러그인을 설치하면 IDE에서 롬복 어노테이션 사용 가능)
// 패키지 바깥에서 접근할 수 있는 클래스라면 접근자를 제공함으로써 클래스 내부 표현 방식을 언제든 바꿀 수 있는 유연성을 얻을 수 있다.

@AllArgsConstructor
class Point {
    @Getter @Setter private double x;
    @Getter @Setter private double y;
}
```

```java
// 하지만 package-private 클래스 혹은 private 중첩 클래스라면 데이터 필드를 노출한다 해도 하등의 문제가 없다.
// 불변 필드를 노출한 public 클래스 - 과연 좋은가?
// 여전히 API를 변경하지 않고는 표현 방식을 바꿀 수 없고, 필드를 읽을 때 부수 작업을 수행할 수 없다는 단점은 여전하다.
// 단, 불변식은 보장할 수 있게 된다.(final)
public final class Time { 
    private static final int HOURS_PER_DAY = 24;
    private static final int MINUTES_PER_HOUR = 60;

    public final int hour;
    public final int minute;  

    public Time(int hour, int minute) {
        if(hour < 0 || hour >= HOURS_PER_DAY) 
            throw new IllegalArgumentException("시간: " + hour);
        if(hour < 0 || minute >= MINUTES_PER_HOUR)
            throw new IllegalArgumentException("분: " + minute);
        this.hour = hour;
        this.minute = minute; 
        // ... 나머지 코드 생략
    }
}
```

```java
// private 중첩 클래스 예제
public class TopPoint {

    private static class Point {
        public double x;
        public double y;
    }

    public Point getPoint() {
        Point point = new Point();  // TopPoint 외부에선
        point.x = 3.5;              // Point 클래스 내부 조작이
        point.y = 4.5;              // 불가능 하다.
        return point;
    }

}
```

```java
// public 클래스의 필드를 직접 노출시킨 사례
// 자바 플랫폼 라이브러리에서도 public 클래스의 필드를 직접 노출하지 말라는 규칙을 어긴 사례
// java.awt.Component 클래스 내부
   /**
     * getSize() 메서드를 호출 할 때마다
     * Dimension 인스턴스를 생성하고 있다.
     */

    public Dimension getSize() {
        return size();
    }

    // @Deprecated : 빌드할때 경고 메시지를 보여줌
    @Deprecated
    public Dimension size() {
        return new Dimension(width, height);
    }

```
```java


public class Dimension extends Test {
    public int width;
    public int height;

// Dimension 클래스의 필드는 가변으로 설계되어 getSize를 호출하는 모든 곳에서 방어적 복사를 위해 인스턴스를 새로 생성해야만 한다.
// JVM 힙 영역 메모리 계속 쌓이는 문제
```


```java
// 고정 되서 사용하는 변수 값은 static 으로 정적 메모리에 할당해두고 사용하는 것을 권장
public class messageConstants{
    public static final String MESSAGE_START = "시작";
    public static final String MESSAGE_END = "끝";
}

// 외부 클래스에서 messageConstants.MESSAGE_START 로 접근 가능.
``` 

## JVM 구성 요소
-메소드영역(클래스, 변수)  
-힙 영역(인스턴스, 객체)  
-스택영역(매개변수, 지역변수)  
-PC레지스터(스레드마다 하나씩존재, 현재 수행중인 JVM주소 저장)  
-Native 메소드 스택(JNI표준규약 제공)영역으로 이루어짐  

<img width="363" alt="JVM 구조" src="https://t1.daumcdn.net/cfile/tistory/216AE04C5654207F0A">



## 핵심정리
 - public 클래스는 절대 가변 필드를 직접 노출해서는 안 된다. 불변 필드라면 노출해도 덜 위험하지만 완전히 안심할 수는 없다.
 하지만 package-private 클래스나 private 중첩 클래스에서는 종종(불변이든 가변이든) 필드를 노출하는 편이 나을 때도 있다.

- 또한 롬복 플러그인을 설치 후 개발하면 가독성 및 생산성이 향상된다.(빌드할때 코드 자동 생성 해줌)
주의사항으로는 너무 남발 하지 말고 꼭 필요한 어노테이션만 지정하여 사용하는 것을 권장한다.




```
-접근 제어자(Access Modifier)
접근제어자는 클래스, 메소드, 인스턴스 및 클래스 변수를 선언할 때, 사용된다. 자바에서 사용하는 접근지시자는 public, protected, package-private(접근 제어자 없음), private로 총 네가지 이다.

-public
누구나 접근 가능하다.
-protected
같은 패키지에 있거나, 상속 받는 경우 사용할 수 있다.
-package-private
아무 접근제어자를 적어주지 않은 경우이며, package-private라 불린다. 같은 패키지 내에서 접근 가능하다.
-private
해당 클래스 내에서만 접근 가능하다.
```

<img width="500" alt="접근 제어자 관계도" src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FKb2tD%2FbtqRrPm3DAn%2F9Q28T3eXG4n3kukPuMiup0%2Fimg.png">
 
 
 ### Reference
 [-접근제어자 참고](https://kils-log-of-develop.tistory.com/430)    
 [-private 중첩 클래스 예제 참고](https://hyeon9mak.github.io/Effective-Java-item16/)  
 [-JVM 구성요소](https://hoonmaro.tistory.com/19)
 [-롬복 참고](https://www.daleseo.com/lombok-popular-annotations/)  


