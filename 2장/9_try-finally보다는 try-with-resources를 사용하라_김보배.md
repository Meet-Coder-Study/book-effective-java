# [이펙자바] 아이템9. try-finally 보다는 try-with-resources를 사용하라.

자바 라이브러리에서 close 메서드를 호출해 직접 닫아줘야하는 자원들이 있다.

- Ex)
    - InputStream
    - OutputStream
    - java.sql.Connection
- 이러한 자원들을 닫아주는 것은 클라이언트가 놓치기 쉬워 예측할 수 없는 성능문제로 이어지기도 한다.
- 상당수가 안전망으로 finalizer를 활용하고는 있지만 아이템8에서 이야기하듯 사용하는 것이 좋지 않다.

이전부터 close를 하기 위해 `try-finally` 가 사용됬다. 

Ex)

- 자원이 하나인 경우

```java
static String firstLineOfFile(String path) throws IOException {
	BufferedReader br = new BufferedReader(new FileReader(path));
	try {
		return br.readLine();
	} finally {
		br.close();
	}
}
```

- 자원이 2개인 경우

```java
static void copy(String src, String dst) throws IOException {
	InputStream in = new FileInputStream(src);
	try {
		OutputStream out = new FileOutputStream(dst);
		try {
			byte[] buf = new byte[BUFFER_SIZE];
			int n;
			while ((n = in.read(buf)) >= 0)
				out.write(buf, 0, n);
		} finally {
			out.close();
	} finally {
		in.close();
	}
}
```

위 둘 예제에 실수가 있다. 

- 예외는 try 블록과 finally 블록 모두에서 발생할 수 있다. 기기의 물리적인 문제가 발생했다면 이로 인해서 `readLine` 메서드가 예외를 던지고, 같은 이유로 `close` 메서드도 실패할 것이다.
- 그러면 스택추적 내역에서 **첫 번째 예외의 정보는 남지 않게 되어 실제 시스템에서의 디버깅을 어렵게한다.**
  

Ex)
- 직접 만들어 본 예제

```java
public class Main {
    public static void main(String[] args) {
        try {
            writeData("Write this !");
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    public static void writeData(String writeString) throws IOException {
        TestFileOutputStream ts = new TestFileOutputStream();
        try {
            ts.write(writeString, true);
        } finally {
            ts.close();
        }
    }

    public static void writeData2(String writeString) throws IOException {
        TestFileOutputStream ts = new TestFileOutputStream();
        try {
            TestFileOutputStream ts2 = new TestFileOutputStream();
            try {
                ts.write(writeString, true);
                ts2.write(writeString, true);
            } finally {
                ts2.close();
            }
        } finally {
            ts.close();
        }
    }
}

-------------------------------------------------------

public class TestFileOutputStream {
    public void write(String writeData, boolean throwException) throws IOException {
        System.out.println("WRITING .... : " + writeData);
        if (throwException)
            throw new IOException("[TestFileOutputStream readLine method] EXCEPTION !");
    }

    public void close() throws IOException {
        throw new IOException("[TestFileOutputStream close method] EXCEPTION");
    }
}
```

- 위 예제에서 의도적으로 `write`와 `close`에서 `IOException`를 발생시켰다.
    - `write` 와 `close`에서 둘 다 `IOException` 이 발생하는 것이지만, 두 번째 예외(`close`에 대한 예외) 만 표시되게 된다.
        ![Untitled](https://user-images.githubusercontent.com/37873745/105031564-d0d3c380-5a98-11eb-8ec7-2130f7726ca7.png)

----
### **이러한 문제들을 Java 7에서 나온 try-with-resources가 해결했다.**

- try-with-resources 구조를 사용하기 위해선 `AutoCloseable` 인터페이스를 구현해야 한다.
    - [AutoCloseable (Java Platform SE 7 )](https://docs.oracle.com/javase/7/docs/api/java/lang/AutoCloseable.html)

    - 단순히 void를 반환하는 close 메서드 하나만 정의한 인터페이스
    - try-with-resoureces와 `AutoCloseable` 을 구현해놓은 클래스를 사용하면 try 문이 종료될 때 **자동으로 close를 호출**해준다.
- 닫아야 하는 자원을 가지는 클래스를 작성한다면 `AutoCloseable` 을 반드시 구현해야한다.

Ex) 앞의 예제를 수정해보자.

- 자원이 하나인 경우

```java
static String firstLineOfFile(String path) throws Exception {
	try (BufferedReader br = new BufferedReader (
		new FileReader(path))) {
			return br.readLine();
	}
}
```

- 자원이 둘인 경우

```java
static void copy(String src, String det) throws IOException {
	try (InputStream in = new FileInputStream(src);
		OutputStream out = new FileOutputStream(dst)) {
		byte[] buf = new byte[BUFFER_SIZE];
		int n;
		while ((n == in.read(buf)) > = 0)
			out.write(buf, 0, n);
}
```

위의 예제를 보면 알겠지만, try-with-resources 버전이 

- **짧고 읽기도 수월하다.**
- **문제를 진단하기도 좋다.**

위 예제에서도 `readLine` 과 `close` 에서 둘 다 예외가 발생한다고 하자.

- `close` 에서 발생한 예외는 숨겨지고 `readLine` 에서 발생한 예외가 기록된다.
    - `try-finally` 예제와 달리 `close` 예외에 덮어지지 않는다.
- 버려지지는 않고, **스택 추적 내역에 'suppressed (숨겨짐)'이 붙어 출력**된다.
- 또한, Java 7에서 `Throwable` - `getSuppressed` 메서드를 사용하면 프로그램 코드에서 가져올 수 있다.

Ex)

- 직접 만든 예제

```java
public class Main {
    public static void main(String[] args) {
        try {
            writeData("Write this !");
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    public static void writeData(String writeString) throws IOException {
        try(TestFileOutputStream ts = new TestFileOutputStream()){
            ts.write(writeString, true);
        }
    }
    public static void writeData2(String writeString) throws IOException {
        try (TestFileOutputStream ts = new TestFileOutputStream();
            TestFileOutputStream ts2 = new TestFileOutputStream()) {
            ts.write(writeString, true);
            ts2.write(writeString, true);
        }
    }
}

-------------------------------------------------------

public class TestFileOutputStream implements AutoCloseable{
    public void write(String writeData, boolean throwException) throws IOException {
        System.out.println("WRITING .... : " + writeData);
        if (throwException)
            throw new IOException("[TestFileOutputStream readLine method] EXCEPTION !");
    }

    @Override
    public void close() throws IOException {
        throw new IOException("[TestFileOutputStream close method] EXCEPTION");
    }
}
```

- 이전 직접만들었던 예제와 유사하나 바뀐 부분은 다음과 같다.
    - `try-finally` 대신 `try-with-resources` 를 사용했다.
    - `TestFileOutputStream` 에서 `AutoCloseable`를 구현했다. (`close`가 `Override` 되었다)
    - 나머지는 같다.
- 결과는 앞서 언급되어있듯 Suppressed로 꼬리표를 달고 숨겨진 예외들이 보여지게 된다.
- `writeData` 인 경우
    ![Untitled 1](https://user-images.githubusercontent.com/37873745/105031556-ce716980-5a98-11eb-93c6-955f3e15e2e6.png)

- `writeData2` 인 경우
    ![Untitled 2](https://user-images.githubusercontent.com/37873745/105031560-d03b2d00-5a98-11eb-8845-ee67a5569a5e.png)


**try-with-resources에서도 `catch` 절을 사용할 수 있다.**

- try 문을 더 중첩하지 않고도 다수의 예외를 처리할 수 있다.

```java
static String firstLineOfFile(String path, String defaultVal) {
	try (BufferedReader br = new BufferedReader(
			new FileReader(path))) {
		return br.readLine();
	} catch (IOException e) {
		return defaultVal;
	}
}
```

---

## 핵심정리

- **회수해야 할 자원을 다룰 땐, try-with-resources를 사용하자. → 예외는 없다. 무조건.**
- **코드는 더 짧아지고 분명해진다. 또한, 예외정보도 훨씬 유용하다.**
- **try-finally 보다 정확하고 쉽게 자원을 회수할 수 있다.**