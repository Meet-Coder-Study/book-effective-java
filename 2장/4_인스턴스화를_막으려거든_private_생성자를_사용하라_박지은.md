># 요약
### 정적 멤버(정적 메서드, 정적 필드)만 담은 Utility Class 생성 시 '인스턴스화' 방지를 위해 private 생성자를 사용하기

># 상태*
```java
public class DateUtility {
    private static String FULL_DATE_FORMAT = "yyyy-MM-dd HH:mm:ss.SSS";

    // 생성자 없음

    public static String convertDateToString(Date date) {
        return new SimpleDateFormat(FULL_DATE_FORMAT).format(date);
    }
}
```


># 기대결과*
```java
public void someMethod() {
    DateUtility.convertDateToString(new Date());
}
```

># 실제결과*
```java
    DateUtility dateUtility = new DateUtility();
    String formattedToday = dateUtility.convertDateToString(new Date());
}
```

># 해결방법*
```java
class DateUtility {
    private DateUtility() {
        throw new AssertionError();
    }
}
```


># 추상화로는 해결할 수 없을까?*
```java
abstract class DateUtility {
    // ...생략
}

class SubDateUtility extends DateUtility {
    // ...생략
}

public class PrivateConstructorTest {
    public static void main(String[] args) {
        // abstract 클래스는 인스턴스화 불가
        // DateUtility dateUtility = new DateUtility();

        // Okay!
        SubDateUtility subDateUtility = new SubDateUtility();
    }
}
```

># 예시**
```java
public class UtilClass {

    // 유틸성 클래스로 인해 인스턴스화 방지
    private UtilClass() {
        throw new AssertionError();
    }

    // 나머지 코드는 생략
}
```

># 출처
*https://madplay.github.io/post/enforce-noninstantiability-with-private-constructor    
**이펙티브 자바 3판   

