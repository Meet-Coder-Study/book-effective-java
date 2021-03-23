# item 73.추상화 수준에 맞는 예외를 던져라

---

## 🎯 아래 계층의 예외를 예방하거나 스스로 처리할 수 없고, 그 예외를 상위 계에 그대로 노출하기 곤란하다면 예외 번역을 사용하라. 이때 예외 연쇄를 이용하면 상위 계층에는 맥락에 고수준 예외를 던지면서 근본 원인도 함께 알려주어 오류를 분석하기에 좋다.

---

## 일반적인 예외 처리 방식

예외 복구, 예외 회피, 예외 전환

```java
		                          ---> Throwable <--- 
		                          |    (checked)     |
		                          |                  |
		                          |                  |
		                  ---> Exception           Error
		                  |    (checked)        (unchecked)
		                  |
		            RuntimeException
		              (unchecked)
```

[Exception Handling in Java: A Complete Guide with Best and Worst Practices](https://stackabuse.com/exception-handling-in-java-a-complete-guide-with-best-and-worst-practices/)

## 예외 번역(Exception Translation), 예외 전환

발생한 예외를 상황에 맞는 적절한 예외로 전환해 던지는 방법

> 목적

1. 내부에서 발생한 예외를 그대로 던지는 것이 그 예외 상황에 대한 적절한 의미를 부여하지 못하는 경우에

 의미를 분명하게 해줄 수 있는 예외로 바꿔주기 위해서

 2. 주로 예외 처리를 강제하는 체크 예외를 언체크 예외인 런타임 에러로 바꾸기 위해서

추상화 수준이 낮은 곳에서 높은 곳으로 예외를 발생시킬 경우, 높은 계층의 api가 오염된다.

메서드가 하는 일과 뚜렷한 관련성 없는 예외가 메서드에서 발생하기 때문이다

그래서 낮은 곳에서 받은 예외를 상위 계층의 예외로 번역하여 사용해야 한다.

```java
//예외 번역
try {
//낮은 수준의 추상화 계층 이용
} catch(LowerLevelException e) {
throw new HigherLevelException();
}
```

AbstractSequentialList 예제(의미 있는 예외로 변환하여 던지는 예시)

```java
public E get(int index) {
    ListIterator<E> i  = listIterator(index);
    try {
        return i.next();
    } catch (NoSuchElementException e) {
        throw new IndexOutOfBoundsException();
    }
}
```

SQLException 은 대표적인 체크 예외이다.

중요한 것은 SQLException은 대부분 복구가 불가능하다는 점이다.(예외를 잡아도 처리해 줄 수 없다)

그래서 아래의 예시와 같이 처리해 줄 수 있는 것은 처리하고, 처리할 수 없는 것은 런타임 에러로 포장하여 

호출하는 쪽에서 무차별 throws(→ 무책임한 예외처리 기법)를 선언하지 않도록 해주는 것이 좋다.

```java
try {

} catch (SQLException e) {
    if(e.getErrorCode() 등을 이용해서 처리할 수 있는 경우){
        // 처리 코드
    } else{
        // 로깅, 에러 내용 메일 전송 등의 로직
        throw new RuntimeException(e);
    }
}

출처: https://joont.tistory.com/157 [Toward the Developer]
```

## 예외 연쇄(Exception Chaining)

하위 계층에서 발생한 예외 정보가 상위 계층 예외를 발생시킨 문제를 디버깅하는데 유용할때

- 하위 계층 예외(원인 cause)는 상위 계층 예외로 전달된다, 상위 계층 예외의 접근자 메서드(Throwble.getCause)를 호출하면 해당 정보를 꺼낼 수 있다.

```java
//예외 연결
try {
    //낮은 수준의 추상화 계층 이용
} catch(LowerLevelException e) {
    throw new HigherLevelException(cause);
}
```

```java
public static void main(String[] args) {
        try {
            // 예외 생성
            NumberFormatException ex = new NumberFormatException("Exception");

            ex.initCause(new NullPointerException("근본 원인"));
            throw ex;
        } catch (NumberFormatException ex) {
            ex.getCause().printStackTrace();
        }

        //checked exception ->unchecked exception
        throw new RuntimeException(new Exception("런타임 예외로 변경"));
    }
/**
Caused by: java.lang.NullPointerException: 근본 원인

**/

```

예외 연결을 사용하면 프로그램 안에서 예외의 원인에 접근할 수 있을 뿐 아니라, 최초에 발생한 예외의 스택 추적 정보를 상위 계층 예외에 통합할 수 있다. 

---

## 💡주의 :  예외 변환 기법을 남용해선 안된다!

**제일 좋은 방법은 아예 에러없는 코딩을 하는 것 !**

**그리고 하위계층에서 발생한 에러는 하위계층에서 처리하고 어쩔수 없이 전달될경우에는 적절한 예외로 예외 변환해서 보내야한다.**