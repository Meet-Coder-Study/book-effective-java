# 아이템10. equals는 일반 규약을 지켜 재정의하라

## 핵심정리
```
꼭 필요한 경우가 아니면 equals를 재정의하지 말자.

재정의해야 할 때는 그 클래스의 핵심 필드 모두를 빠짐없이, 
다섯 가지 규약을 확실히 지켜가며 비교해야 한다.
```

---

## 재정의해야 할 때

* 두 객체가 물리적으로 같은가가 아닌 `논리적 동치성`을 확인해야 하는데,
* 상위 클래스의에서 이를 수행하지 않을 때


> 물리적으로 같은가   

두 객체가 메모리의 같은 주소를 가르키고 있는가를 뜻한다.

> 논리적 동치성
```java
public class Student {

    // 학생 번호
    private int studentId;

    public Student(int studentId) {
        this.studentId = studentId;
    }

    // equals 재정의 : 학생 번호가 같으면 true
    @Override
    public boolean equals(Object o) {
        if(o instanceof Student){
            Student student = (Student) o;
            return studentId == student.studentId;
        }
        return false;
    }
}
```
위의 Student 객체는 생성될 때마다 물리적으로는 다르겠지만, 학생 번호만 같다면 **논리적으로 같다.**


---

# equals 재정의의 5가지 일반 규약

## 1. 반사성 (reflexivity)

*객체는 자기 자신과 같아야 한다.*

## 2. 대칭성 (symmetry)

*x.equals(y)가 참이면 그 반대(y.equals(x))도 참이어야 한다.*

다음과 같이 대소문자를 구분하지 않는 CaseInsensitiveString 클래스가 있다.    
```java
public class CaseInsensitiveString {
    private final String s;

    public CaseInsensitiveString(String s) {
        this.s = s;
    }

    @Override
    public boolean equals(Object o) {
        // ...생략
        if (o instanceof String) {  //String과의 비교 연산 시도
            return s.equalsIgnoreCase((String) o);
        }
        // ...생략
        return false;
    }
}
```
이 클래스의 equals에서는 일반 문자열(String)과 비교하려 하고 있다.
```java
CaseInsensitiveString cis = new CaseInsensitiveString("AAA");
String s = "aaa";

cis.equals(s); //true
s.euqals(cis); //false
```
CaseInsensitiveString의 equals 자체는 잘 작동하지만 **그 반대는 작동하지 않는다.**  
String의 equals는 CaseInsensitiveString의 존재를 알지 못하기 때문이다.    

이를 해결하기 위해서는 equals에서 **일반 String과 비교하는 부분을 없애야 한다.**

## 3. 추이성(transitivity)
*첫 번째 객체와 두 번째 객체가 같고, 두 번째 객체와 세 번째 객체가 같다면, 첫 번째 객체와 세 번째 객체도 같아야 한다.*

추이성 문제는 주로 상위 클래스에 없는 새로운 필드를 하위 클래스에 추가할 때 발생한다.

다음과 같이 스마트폰을 표현하는 GalaxyS 클래스와 그 하위의 GalaxyNote 클래스가 있다.
```java
public final String name;

    public GalaxyS(String name) {
        this.name = name;
    }
    @Override
    public boolean equals(Object o) {
        if (o instanceof GalaxyS) {
            return name.equals(((GalaxyS) o).name);
        }
        return false;
    }
```
GalaxyNote 클래스에서는 S펜 기능을 위한 pen 필드가 추가되었다.
```java
public class GalaxyNote extends GalaxyS {
    public final String pen;

    public GalaxyNote(String name, String pen) {
        super(name);
        this.pen = pen;
    }

    @Override
    public boolean equals(Object o) {
        if(!(o instanceof GalaxyS)) return false;
        if (o instanceof GalaxyNote) { //GalaxyNote 일 경우 이름과 펜 모두 비교
            return super.equals(o) && pen.equals(((GalaxyNote) o).pen);
        }
        return name.equals(((GalaxyS) o).name); //GalaxyS 일 경우 이름만 비교
    }
}
```
대칭성을 지키기 위해 *GalaxyS일 경우와, GalaxNote일 경우를 나누어 equals를 정의*하였다.   
하지만 이는 또다른 문제가 발생한다. 추이성을 위반하는 것이다.

```java
@Test
    void equalsTest(){
        GalaxyNote g1 = new GalaxyNote("s20", "blue pen");
        GalaxyS g2 = new GalaxyS("s20");
        GalaxyNote g3 = new GalaxyNote("s20", "red pen");

        assertThat(g1).isEqualTo(g2);   //pass
        assertThat(g2).isEqualTo(g3);   //pass
        assertThat(g1).isEqualTo(g3);   //fail
    }
```
이처럼 g1, g2는 같고 g2,g3도 같지만 g1,g3는 서로 같지 않다.(추이성 위반)    
이를 해결하기 위해 equals를 instanceof 검사 대신 getClass 검사로 바꾸면 추이성을 지킬 수는 있다. 하지만...
```java
if (o == null || getClass() != o.getClass()) return false;
```
이는 `리스코프 치환 원칙의 위배`한다는 문제가 있다.  
즉 상위 클래스인 GalaxyS는 하위클래스인 GalaxyNote와 getClass가 다르기 때문에 equals는 항상

>리스코프 치환 원칙 : 상위 타입의 자료형은 하위 타입으로 변환되어도 문제 없이 작동해야 한다.

아쉽게도, `클래스를 확장해 새로운 값을 추가하면서 equals 규약을 만족시킬 방법은 존재하지 않는다.`

다만 컴포지션 패턴을 통해 이를 우회할 수 있다.
```java
public class GalaxyNoteComposition {
    private final GalaxyS galaxyS;
    private final String pen;

    public GalaxyNoteComposition(String name, String pen) {
        this.galaxyS = new GalaxyS(name);
        this.pen = pen;
    }
    
//    GalaxyS의 뷰를 반환
    public GalaxyS asGalaxyS(){
        return galaxyS;
    }

    @Override
    public boolean equals(Object o) {
        if(!(o instanceof GalaxyNoteComposition)) return false;
        GalaxyNoteComposition gn = (GalaxyNoteComposition) o;
        //컴포지션 내부의 멤버들을 비교
        return galaxyS.equals(gn.galaxyS) && pen.equals(gn.pen);    
    }
}
```
컴포지션 패턴을 통해 GalaxyS를 **상속 받는 대신 private 멤버로** 두고 필요할 경우 GalaxyS의 뷰를 반환하는 메소드를 만든다.   
이후 equals를 재정의할 때는 상위/하위 클래스를 따질 필요 없이 **컴포지션 클래스 내부의 멤버들을 비교**하기만 하면 된다.

## 4. 일관성(consistency)

*두 객체가 같다면 (수정되지 않는한) 앞으로 영원히 같아야 한다.*     
equals를 판단하는 기준이 신뢰할 수 없는 자원이 되어서는 안된다.

## 5. null-아님(non-null)

*모든 객체가 null이 아니어야 한다. (x.equals(null)은 항상 false)*