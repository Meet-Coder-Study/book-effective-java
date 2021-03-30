# 예외를 무시하지 말라.

`예외 처리란, 예외상황에 적절한 처리를 하기 위해 존재한다.`

## 부적절한 예외처리

 1. 예외 블랙홀 : 아무 처리도 하지 않음
```java
try {
    // 예외가 발생할 수 있는 코드
} catch(~Exception e) {
}
```

 2. 예외 블랙홀 : 로그만 출력함
```java
try {
    // 예외가 발생할 수 있는 코드
} catch(~Exception e) {
    e.printStackTrace();
}
```
 
 3. 무책임하게 예외 throws
 ```java
void method() throws Exception {
    // 예외가 발생할 만한 코드
}
 ```
 - 부적절하긴 하지만 그나마 낫다.
 - 이렇게라도 해두면 바깥의 메서드들중 한곳에서 디버깅 정보를 남기고 프로그램이 멈출 것이다.
 - 추상화 수준에 맞게 던져야서 처리할 것이면 좋다. (item73)
 
## 예외처리를 생략해도 되는 경우도 있긴 하다.
```java
public class FileInputStreamEx {

    private static final File defaultFile = new File("defaultFilePath");

    public static void main(String[] args) {
        FileInputStreamEx fx = new FileInputStreamEx();

        FileInputStream fileInputStream = fx.openFile();
        close(fileInputStream);
    }

    public FileInputStream openFile() {
        String filePath = (new Scanner(System.in)).nextLine();
        File file = new File(filePath);

        try {
            return new FileInputStream(file);
        } catch (FileNotFoundException e) {
            return openFile();
        }
    }

    public static void close(FileInputStream fileInputStream) {
        try {
            fileInputStream.close();
        } catch (IOException ignored) { 
            // 아래의 이유로 예외를 무시한다.
            ignored.printStackTrace(); // 예외가 주기적으로 발생한다면 조사하기 좋게 로그를 남긴다.
        }
    }
}
```
 1. FileInputStream 은 파일을 읽는 스트림이므로, 파일의 상태를 변경하지 않기에 복구할 것이 없다.
 1. close() 한다는 것은 정보는 다 읽었다는 뜻이므로 작업을 중단할 이유도 없다.
 
 
#### 어떤 이유로든, 예외를 무시하기로 했다면 주석으로 이유를 남기고, 예외 변수의 이름을 ignored 로 해두자.

#### 예외를 처리할 곳에서 예외를 처리하지 않는다면, 정말 이상한 곳에서 결국 에러가 터져 디버깅하기가 힘들 수 있다.