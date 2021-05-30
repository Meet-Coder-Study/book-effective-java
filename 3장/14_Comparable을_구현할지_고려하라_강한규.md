# Item 14. Comparable을 구현할지 고려하라 

[백기선님 강의](https://www.youtube.com/watch?v=0yUxPUXS1pM&list=PLfI752FpVCS8e5ACdi5dpwLdlVkn0QgJJ&index=6)

[code1](https://github.com/keesun/study/blob/master/effective-java/item6.md)

[code2](https://github.com/WegraLee/effective-java-3e-source-code/tree/master/src/effectivejava/chapter2/item6) 

## 1. 결론 

자연적인 순서를 고려해야 한다면 Comparable 구현을 추가하라 

즉, 정렬과 비교기능을 넣어야 한다면 Comparable을 고려하라는 것 

구현 방법으로는 2가지를 고려한다

1. (단순비교) Comparable Interface를 사용하면 compareTo 매서드를 사용하고 박싱된 기본 타입 클래스가 제공하는 정적 compare 매서드 사용한다
2. (복잡한 비교) Comparator Interface가 제공하는 비교자 생성 매서드를 사용한다 

## 2. Equal 과 비교

- compareTo는 단순 동치성 비교에 추가로 순서까지 비교할 수 있다 
- 제네릭하게 특정 타입을 정의해서 사용할 수 있다 
- 자연적인 순서가 있다는 것은 정렬 기능을 추가로 사용할 수 있다 
  - 자연적인 순서라하면 알파벳, 숫자, 연대 와 같은 순서들을 얘기한다

- 순서를 비교할 때 -1, 0, 1로 비교 할 수 있는데 객체가 주어진 객체보다 작으면(<) -1을, 주어진 객체와 같으면(=) 0을, 주어진 객체보다 크면(>) 1을 반환한다
  - 0을 반환한다면 Equal의 결과도 true가 나오도록 고려해라 
- 타입이 다른 객체가 주어지면 이때는 ClassCastException 을 던져라 

## 3. Compareble 을 지원하는 Java 기본 클래스들

- TreeSet, TreeMap: 비교를 활용하는 자료구조 클래스
- Collections, Arrays: 검색과 정렬 알고리즘을 활용하는 유틸리티 클래스

## 4. 확장에 있어 유의 사항 

기존 클래스를 확장한 확장 클래스에서 새로운 컴포넌트가 추가되면 단순 비교에 의한 Compareble 규칙(-1,0,1)이 일관되지 않을 수 있다 

이 경우는 확장에 대한 클래스를 독립적으로 구현하고 이 클래스의 인스턴스를 가리키는 필드 클래스를 원래 클래스에 둔다

동치성의 결과를 가져오는 equals 와 compareTo 의 결과가 같도록 유지해야 한다 이 경우 확장된 부분에도 일관성 있는 결과를 가져올 수 있다 

## 5. Comparator를 고려해야 하는 경우 

Comparable을 구현하지 않는 필드나 표준이 아닌 순서로 비교해야 한다면 비교자(Comparator)를 대신 사용한다 

비교자는 직접 만들거나 자바가 제공하는 것 중에 골라 쓸 수 있다

과거에는 compareTo 매서드에서 관계 연산자인 <, >, = 을 사용하는 부분이 있었으나 java7 이 후에는 기본 타입비교에는 박싱된 기본타입에 제공하는 정적 매서드를 권장한다 

아래 예제는 자바가 제공하는 비교자를 사용하고 있다 
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

```java
Comparator<Student> comparator = new Comparator<Student>() {
    @Override
    public int compare(Student s1, Student s2) {
        return Double.compare(s1.score, s2.score);
    }
};
```

## 6. 객체 참조용 비교자를 활용하는 경우 

단순히 필드하나에 대해 compare로서 비교를 판단하기 어려운 경우가 있다 즉, 객체 비교가 복잡해지는 경우에 Comparetor 비교자를 활용할 수 있고 자바에서는 이를 사용하는 정적 매소드도 제공한다 

순차적으로 필드를 비교하는 경우에 아래처럼 할 수 있다 

```java
import java.util.Comparator;

public class Student implements Comparable<Student> {
    private static final Comparator<Student> COMPARATOR =
            Comparator.comparingInt((Student student) -> student.grade)
                    .thenComparing((Student student) -> student.name)
                    .thenComparingInt((Student student) -> student.age);

    private int grade;
    private String name;
    private int age;

    @Override
    public int compareTo(Student o) {
        return COMPARATOR.compare(this, o);
    }
}
```

Comparator의 정적 매소드를 체이닝해서 사용할 수 있다 

## 7. 마지막 주의사항 

해시코드의 값의 차를 비교하는 경우는 
- 기본타입의 compare 정적 매소드를 사용해라
- Comparator의 정적 타입 메소드를 사용해라 

이렇게 쓰지말고
```java
private static final Comparator<Student> HASHCODE_COMPARATOR = new Comparator<Student>() {
    @Override
    public int compare(Student o1, Student o2) {
        return o1.hashCode() - o2.hashCode();
    }
};
```

이렇게 써라
```java
private static final Comparator<Student> HASHCODE_COMPARATOR = new Comparator<Student>() {
    @Override
    public int compare(Student o1, Student o2) {
        return Integer.compare(o1.hashCode(), o2.hashCode());
    }
};

private static final Comparator<Student> HASHCODE_COMPARATOR = Comparator.comparingInt(
            Object::hashCode);
```

## 8. 다시 결론

결론적으로 보면 자바에서 제공하는 정적 매소드 사용을 가이드로 따르는 식으로 유도하고 있다

커스텀하게 비교를 하려면 비즈니스는 개발자가 짜고 비교에 대한 기능은 만들어진 자바 클래스의 정적 매소드를 권장하고 있다 