## Comparable을 구현할지 고려하라
### Comparable: 인터페이스를 구현한 객체 스스로에게 부여하는 한 가지 기본 정렬 규칙을 설정
### Comparator: 인터페이스를 구현한 클래스는 정렬 규칙 그 자체를 의미함, 기본 정렬 규칙과는 다르게 원하는대로 정렬순서를 지정할 때 사용

<br>

  - Comparable을 구현했다는 것은 그 클래스의 인스턴스들에는 자연적인 순서(natural order)가 있음을 뜻함
    - 자연적인 순서(natural order): 일반적으로 받아들이는 대소 비교
    - ex) 1보다는 2가 큼
  - Arrays.sort() 메서드는 자동으로 Comparable에 구현되어 있는 compareTo() 메서드를 호출해서 사용함
  - 아래 프로그램은 명령줄 인수들을 알파벳순으로 출력함
    - String 클래스가 Comparable을 구현했기 때문
  
  ```java
  public class WordList {
    public static void main(String[] args) {
      Set<String> set = new TreeSet<>();
      Collections.addAll(set, args);
      System.out.println(set);
    }
  }
  ```
  
  
  - HashSet
    - 데이터를 중복 저장할 수 없고 **순서를 보장하지 않음**
  - TreeSet
    - 중복된 데이터를 저장할 수 없고 입력한 순서대로 값을 저장하지 않음
    - TreeSet은 기본적으로 **오름차순으로 데이터를 정렬**함
  - LinkedHashSet
    - 중복된 데이터를 저장할 수 없고 **입력된 순서대로 데이터를 관리**함

<br>

### compareTo 메서드의 일반 규약은 equals의 규약과 비슷하다.
  - 객체가 주어진 객체보다 작으면 음의 정수, 같으면 0, 크면 양의 정수를 반환
  - 비교할 수 없는 객체가 주어지면 ClassCastException을 던짐
  

```java
class Student {
    String studentName; // 이름
    int studentId;      // 학번
    double score;       // 학점

    public Student(String studentName, int studentId, double score) {
        this.studentName = studentName;
        this.studentId = studentId;
        this.score = score;
    }

    @Override
    public String toString() {
        return "이름: " + studentName + ", 학번: " + studentId + ", 학점: " + score;
    }
}
```

  - Student 클래스는 정렬 기준이 없으므로 정렬을 할 수 없음
  - Comparable 구현
    
```java
class Student implements Comparable<Student> {
    String studentName; // 이름
    int studentId;      // 학번
    double score;       // 학점

    public Student(String studentName, int studentId, double score) {
        this.studentName = studentName;
        this.studentId = studentId;
        this.score = score;
    }

    @Override
    public String toString() {
        return "이름: " + studentName + ", 학번: " + studentId + ", 학점: " + score;
    }

    // 학번(studentId) 기준으로 오름차순 정렬
    @Override
    public int compareTo(Student o) {
        return Integer.compare(studentId, o.studentId);
    }
}
```

<br>
   
```java
public class ComparableExample {
    public static void main(String[] args) {
        Student[] student = getStudents();
        print(student);

        Arrays.sort(student);

        System.out.println("========================");
        System.out.println("========================");

        print(student);
    }

    private static Student[] getStudents() {
        Student[] student = new Student[5];

        student[0] = new Student("Dave", 20120001, 4.2);
        student[1] = new Student("Amie", 20150001, 4.5);
        student[2] = new Student("Emma", 20110001, 3.5);
        student[3] = new Student("Brad", 20130001, 2.8);
        student[4] = new Student("Cara", 20140001, 4.2);

        return student;
    }

    private static void print(final Student[] student) {
        for (Student s : student) {
            System.out.println(s.toString());
        }
    }
}
```

<img width="460" alt="Comparable" src="https://user-images.githubusercontent.com/50076031/104839698-2461d800-5906-11eb-8a01-b8993cbf2a2d.PNG">

<br>

  - 위에서 Comparable로 정의한 학번순이 아닌 **학점** 순으로 정렬하고자 할 때? -> Comparator 사용할 수 있음

```java
...

Comparator<Student> comparator = new Comparator<Student>() {
    @Override
    public int compare(Student s1, Student s2) {
        return Double.compare(s1.score, s2.score);
    }
};
          
Arrays.sort(student, comparator);
        
...

```

  - 위 Comparator의 익명 클래스는 다음과 같은 람다식으로도 가능함
  ```java
  Arrays.sort(student, (s1, s2) -> Double.compare(s1.score, s2.score));
  
  Arrays.sort(student, Comparator.comparingDouble(s -> s.score));
  ```
  
  <img width="460" alt="Comparator" src="https://user-images.githubusercontent.com/50076031/104843057-1a42d800-590c-11eb-844a-e7e89e67e2c5.PNG">

    
  <br><br>
    
  ### References
  [`자바 Set - HashSet, TreeSet, LinkedHashSet`]
  [`객체 정렬하기 1부 - Comparable vs Comparator`]
  
  
  
  [`자바 Set - HashSet, TreeSet, LinkedHashSet`]: https://m.blog.naver.com/PostView.nhn?blogId=heartflow89&logNo=220994601249&proxyReferer=https:%2F%2Fwww.google.com%2F
  [`객체 정렬하기 1부 - Comparable vs Comparator`]: https://www.daleseo.com/java-comparable-comparator/
