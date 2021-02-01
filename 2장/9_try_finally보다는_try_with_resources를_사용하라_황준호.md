## 아이템9. try-finally보다는 try-with-resources를 사용하라

`close()`를 통해서 직접 닫아줘야 하는 자원이 많이 있다. (`InputStream`, `OutputStream`, `java.sql.Connection` 등)

자원을 제대로 닫기 위해서 아래와 같이 try-finally 방식으로 구현해왔다.

```java
//try-finally방식
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
        }
    } finally {
        in.close();
    }
}
```

다음은 위 코드를 자바7에 등장한 try-with-resources방식으로 바꾼 코드다.

```java
//try-with-resources방식
static void copy(String src, String dst) throws IOException {
    try (InputStream in = new FileInputStream(src);
         OutputStream out = new FileOutputStream(dst)) {
        byte[] buf = new byte[BUFFER_SIZE];
        int n;
        while ((n = in.read(buf)) >= 0)
            out.write(buf, 0, n);
    }
}
```

단, 이 구조를 사용하려면 해당 자원이 `AutoClosable` 인터페이스를 구현해야 한다.

try-with-resources방식의 장점은

1. 가독성이 좋다

   - try-finally방식은 자원이 2개 이상인 경우 try문을 중첩으로 사용하지만, try-with-resources방식은 그렇지 않다

2. 자원을 닫지 않는 실수를 막을 수 있다

3. try-finally방식보다 문제를 진단하기에 좋다

   - 예를 들어 아래의 코드의 try-finally방식에서, 물리적인 문제 때문에 `br.readLine()` `br.close()` 두 곳에서 예외가 발생한다고 가정했을 때,

   - ```java
     //try-finally방식
     static String firstLineOfFile(String path) throws IOException {
         BufferedReader br = new BufferedReader(new FileReader(path));
         try {
             return br.readLine();//실패
         } finally {
             br.close();//실패
         }
     }
     
     //try-with-resources방식
     static String firstLineOfFile(String path) throws IOException {
         try (BufferedReader br = new BufferedReader(new FileReader(path))) {
             return br.readLine();
         }
     }
     ```

   - try-finally방식은 `br.readLine()`에 대한 스택 추적 내역은 남지 않고 `br.close()`의 스택 추적 내역만 남는다

   - try-with-resources방식은 `br.readLine()`은 기록되고, `br.close()`는 '숨겨졌다'(Suppressed)는 꼬리표를 달고 출력된다.

     <img width="905" alt="suppressed" src="https://user-images.githubusercontent.com/42054054/105153029-4ac48500-5b4b-11eb-8cf8-ba1f3830e2b9.png">

     -> [여기](https://github.com/hwanghe159/lab/tree/master/trywithresources)에서 예제 코드를 볼 수 있습니다

### 결론

회수해야 하는 자원을 다룰 때는 예외없이 try-finally 보다는 try-with-resources를 쓰자. 가독성도 좋아지고 만들어지는 예외 정보도 훨씬 유용하다.