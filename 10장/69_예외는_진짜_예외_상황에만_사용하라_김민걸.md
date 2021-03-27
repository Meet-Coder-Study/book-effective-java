# 예외는 진짜 예외 상황에만 사용하라.

```java
import java.awt.event.MouseAdapter;import java.util.Arrays;class Mountain {
    public void climb() throws InterruptedException {
        Thread.sleep(1000L);
        System.out.println("한걸음~");
    }
}

public static void main(String[] args) {
    List<Mountain> range = Arrays.asList(new Mountain(), new Mountain(), new Mountain());
    for (Mountain m : range) {
        m.climb();
    }
}
```

이 순회를 최적화 하는 방법은 ?

<br>
<br>
<br>
<br>
<br>
<br>

## for-each 문은 내부적으로 iterator 를 통해 반복을 실행한다.
 - JVM 은 배열에 접근할 때 마다 경계를 넘는지 안넘는지 반복한다.
 - 일반적인 반복문도 배열 경계에 도달하면 종료한다.
 - JVM 에서 한번, Iterator 의 hasNext() 이렇게 두번 검사하는 것은 반복이다 ?
 
```java
int i = 0;
try {
    while (true) {
        range[i++].climb();
    }
} catch (Exception e) {
}
```
 - 이 코드는 계속 순회하다가 범위를 벗어나 ArrayIndexOutOfBoundsException 이 일어나면 반복을 중지하는 방법이다.
 
 <br>
 
## 이 최적화 방법은 잘못되었다!
 1. 예외는 예외 상황에 쓸 용도로 설계되었다.
 1. 코드를 try-catch 블록 안에 넣으면 JVM 이 적용할 수 있는 최적화가 제한된다.
 1. 배열을 순회하는 표준 관용구는 앞서의 중복적인 검사를 수행하지 않는다. JVM 이 알아서 최적화해준다.
 
 `실제로 뒷 방법이 훨씬 느리다.`
 ```java
Mountain[] range = new Mountain[1000000];
System.currentTimeMillis();
기준

일반적 for-each : 10715
예외를 이용한 반복 : 10075

제 컴퓨터에서는 비슷하지만, 죠슈아님의 컴퓨터에서는 예외를 사용한 방법이 2배 느리다고 합니다.
```

<br>

### 예외를 사용한 반복문의 다른 단점
 1. 코드를 더럽고 성능을 떨어뜨린다.
 1. 반복문이 돌때 ArrayIndexOutOfBoundsException 가 아닌 다른 예외가 발생해도 정상적으로 배열의 반복이 끝난줄 알 것이다.
 
 > 예외는 오직 예외 상황에서만 써야한다. 절대 일상적인 제어 흐름용으로 쓰여선 안된다.

 > 표준적이고 쉽게 이해되는 관용구를 사용해라. 자신의 머리가 뛰어나다고 착각하지 말라.

<br>

## 잘 설계된 API 라면 클라이언트가 정상적인 제어 흐름에서 예외를 사용할 일이 없게 해야 한다.
상태 의존적 메서드와 상태 검사 메서드를 통해 정상적 제어 흐름에서만 사용할 수 있도록 도와야한다.

 - 상태 의존적 메서드 : 특정 상태에서만 호출할 수 있는 메서드
 - 상태 검사 메서드 : 상태를 검사하는 메서드
 
 위의 두 메서드는 함께 제공해야한다.
 
 > 예를 들어, Iterator 인터페이스에서 상태 의존적 메서드는 next(), 상태 검사 메서드는 hasNext()

#### 정상 흐름을 위해 올바르지 않은 상태일때, 빈 옵셔널이나 null 같은 특수한 값을 넘길수도 있다.


<br>

## 상태 검사 메서드, 옵셔널, 특정 값 중 선택을 하는 팁
 1. 여러 스레드가 동시에 접근가능하거나 외부에서 상태를 바꿀 수 있다  
  -> 옵셔널이나 특정 값을 사용한다.  
  -> 상태 검사 메서드와 상태 의존적 메서드의 동작 사이에 값이 변해버릴 수 있다.
 1. 성능이 중요한 상황에서 상태 검사 메서드가 상태 의존적 메서드의 일과 중복되는 일을 한다.  
  -> 옵셔널이나 특정 값을 사용한다.
 1. 다른 모든 경우에는 상태 검사 메서드, 상태 의존적 메서드를 제공하자.
 
 
<br>
<br>

## 결론
 - 예외를 정상 흐름에서 제어를 위해 사용하면 안된다.
 - 또, 정상 흐름에서 제어하기 위해 예외를 사용해야하는 API 를 만들면 안된다.
    - 그러기 위해서는 상태 의존적 메서드, 옵셔널, 특정값 반환등의 방법을 제공할 수 있다.