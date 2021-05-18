# 아이템 11 equals를 재정의하려거든 hashCode도 재정의하라

- equals : 두 객체의 내용이 같은지, 동등성(equality)를 비교하는 메서드
- hashCode : 두 객체가 같은 객체인지, 동일성(identity)를 비교하는 메서드

---

> **equals 를 재정의**한 클래스 모두에서 **hashCode도 재정의**해야 한다.

- equals를 재정의 한 후 hashCode도 재정의 하지 않으면

  hashCode 일반 규약을 어기게 되는 것이며,

  **HashMap, HashSet**과 같은 컬렉션의 원소로 사용할 때 문제를 일으킬 수 있다.

- Object 명세 내용

  - equals 비교에 사용되는 정보가 변경되지 않았다면, 애플리케이션이 실행되는 동안 그 객체의 hashCode 메서드는 몇 번을 호출해도 일관되게 항상 같은 값을 반환해야 한다.

    단, 애플리 케이션을 다시 실행한다면 이 값이 달라져도 상관없다.

  - **equals(Object)가 두 객체를 같다고 판단했다면, 두 객체의 hashCode는 똑같은 값을 반환해야 한다.**
  - equals(Object)가 두 객체를 다르다고 판단했더라도, 두 객체의 hashCode가 서로 다른 값을 반환할 필요는 없다. 단, 다른 객체에 대해서는 다른 값을 반환해야 해시테이블의 성능이 좋아진다.

Object 명세내용의 두번째 항목을 보면, **논리적으로 같은 객체는 같은 해시코드를 반환**해야 한다.

### ❓hashCode()

- 객체의 hashCode를 리턴하는 메서드
- hashCode : **객체의 주소값을 변환하여 생성한 객체의 고유한 정수값**이다.
- hashCode 에 대한 testCode

  ```java
  class MemberTest {
      @DisplayName("hashCode 에 대해 알아보기")
      @Test
      void jessi의_hashCode(){
          Member jessi = new Member("Jessi", "woman", "22");
          Member john = new Member("John", "man", "22");
          Member jessi2 = jessi; // 같은 객체
          System.out.println(jessi.getName() + "의 hashCode" + jessi.hashCode());
          System.out.println(john.getName() + "의 hashCode" + john.hashCode());
          System.out.println(jessi.hashCode()+" "+jessi2.hashCode()+" 비교");
          assertEquals(jessi.hashCode(), jessi2.hashCode());
      }
  }
  ```

  jessi 와 jessi2는 논리적으로 같은 객체(동일한 주소값을 가진다) 이므로 hashCode 또한 같다.

### 🔎 Hash 관련 Colletions 에서

- HashSet, HashMap 과 같은 Hash 관련 Collection 에서 **Key는 hashCode 를 기준으로 정해진다**.
- Key 값으로 고유한 식별값인 해시값을 만드는데, 이 해시값은 `Bucket` 에 저장된다.
- 해시테이블의 크키는 한정적이기 때문에 서로 다는 객체라도 같은 해시값을 가질 수 있다.
- 이 때 `해시 충돌(Hash Collisions)` 이 발생하게 된다.
- 해시 충돌이 발생할 경우 해당 버킷에 LinkedList(TreeMap) 형태로 객체를 추가한다. (Seperate Chaining)
- 같은 해시값의 버킷 안에 **다른 객체가 있는 경우 eqauls() 메서드가 사용**된다.

- HashTable 에 put() 으로 객체를 추가하는 경우
  - 값이 같은 객체가 있다면(equals() == true) 기존 객체를 덮어쓴다.
  - 값이 같은 객체가 없다면(equals () == false) 해당 entry를 LinkedList에 추가한다.
- HashTable 에 get() 메서드로 객체를 조회하는 경우
  - equals () == true 면, 그 객체를 리턴한다.
  - equals () == false 면, null을 리턴한다.

equals() 는 재정의 하고 hashCode()를 재정의 하지 않을 때 → 같은 객체인데 hashCode 가 다르기 때문에 해당 객체가 저장된 버킷을 찾을 수 없다.  
hashCode()는 재정의하고 equals()를 재정의 하지 않을 때 → hashCode가 같은데 값이 같은 객체가 없으므로(equals()가 false) 원하는 객체를 찾을 수 없다.

---

EffectiveJava 에서 사용된 예를 보면

```java
public static void main(String[] args) {
	Map<PhoneNumber, String> m = new HashMap<>();
	m.put(**new PhoneNumber**(707, 867, 5309), "Jenny"); // 1

	System.out.println(**m.get(new PhoneNumber(707, 867, 5309)**)); // 2
	// "Jenny" 가 나와야 할 것 같지만 null을 반환한다.
}
```

왜 null 을 반환할까 ?

- 2개의 PhoneNumber 인스턴스가 사용되었는데
  1. HashMap에 "Jenny"를 넣을 때 사용됐고,
  2. map에서 Jenny를 get()할 때 사용됐다.
- PhoneNumber 클래스는 hashCode를 재정의하지 않아 **논리적 동치(equals)**인 두 객체가 서로 다른 해시코드를 반환한다. (두번째 규약 위배)
- 두 인스턴스 **같은 버킷(hashCode()%M)**에 있더라도 HashMap은 **hashCode 가 다른 entry끼리**는 동치성 비교를 시도초자 하지 않도록 최적화되어 있기 때문에 null이 반환된다.

---

### 🙆🏼 올바른 hashCode 구현 방법

- 절대! 사용해서는 안되는 방법

  ```java
  @Override public int hashCode() { return 42; }
  ```

  - 모든 객체가 해시테이블의 버킷 하나에 담겨 LinkedList 처럼 동작하게 된다. → 평균 수행 시간이 O(n)으로 느려져서 객체가 많아지면 쓸 수 없게 된다.

- 좋은 해시 함수라면 서로 다른 인스턴스에 다른 해시코드를 반환해야 한다. (세번째 규약)

  - 좋은 hashCode 를 작성하는 간단한 요령

    1. int 변수 reult 를 선언 한 후 값 `c`로 초기화 한다. 이때 c는 해당 객체의 첫번째 핵심 필드를 (2.a)단계 방식으로 계산한 hashCode 다.
    2. 해당 객체의 나머지 핵심필드 `f` 각각에 대해 다음 작업을 수행한다.

       a. 해당 필드의 해시코드 c를 계산한다.

       1. 기본 타입 필드라면, Type.hashCode(f)를 수행한다. (Type: 해당 기본 타입의 박싱 클래스)

       2. 참조 타입 필드면서 이 클래스의 equals 메서드가 이 필드의 equals를 재귀적으로 호출해 비교한다면, 이 필드의 hashCode 를 재귀적으로 호출한다. 계산이 더 복잡해질 것 같으면, 이 필드의 표준형을 만들어 그 표준형의 hashCode를 호출한다. 필드의 값이 null이면 0을 사용한다. (보통 0을 사용)

       3. 필드가 배열이라면, 핵심 원소 각각을 별도 필드처럼 다룬다. 이상의 규칙을 재귀적으로 적용해 각 핵심 원소의 해시코드를 계산한 다음, (2.b)단계 방식으로 갱신한다. 배열에 핵심 원소가 하나도 없다면 단순히 상수(0을 추천)를 사용한다. 모든 원소가 핵심 원소라면 Arrays.hashCode를 사용한다.

       b. (2.a) 단계에서 계산한 해시코드 c 로 result 를 갱신한다.

       result = **31** \* result + c;

       - 31 을 곱하는 이유 : 31은 홀수이며 소수이기 때문이다.
         - 곱할 숫자가 짝수고 오버플로가 발생하면→ 2를 곱하면 시프트 연산과 같은 결과를 주기 때문에 정보를 잃게 된다.
         - 소수를 곱하는것은 전통적으로 해온 방식(나머지연산에서 충돌을 줄이기 위한방법으로 주로 사용)이다.
         - \*31은 시프트연산과 뺄셈으로 대체해 최적화 할 수 있다.
           - 31\*i == (i<<5)-i

    3. result 를 반환한다.

  - 주의사항
    - 파생 필드는 해시코드 계산에서 제외해도 된다.
    - eqauls 비교에 사용되지 않은 필드는 반드시 제외해야 한다. (두번째 규약내용)
    - hashCode가 반환하는 값의 생성 규칙을 API 사용자에게 자세히 공표하지 말자.
      - 클라이언트에서 이 값에 의존하지 않고 추후에 계산 방식을 바꿀 수 있게 한다.
  - 예제 코드

    ```java
    @Override public int hashCode() {
    	int result = Short.hashCode(areaCode);
    	result = 31 * result + Short.hashCode(prefix);
    	result = 31 * result + Short.hashCode(lineNum);
    	return result;
    }

    public static void main(String[] args) {
    	Map<PhoneNumber, String> m = new HashMap<>();
    	m.put(**new PhoneNumber**(707, 867, 5309), "Jenny");

    	System.out.println(**m.get(new PhoneNumber(707, 867, 5309)**));
    	// hashCode를 재정의 했으므로 "Jenny" 가 나온다
    }
    ```

  - 다른 예제 코드

    ```java
    @Override
    public int hashCode() {
    	// 1
    	int result = name.hashCode(); // String 타입의 hashCode
    	// 2
    	result = 31 * result + gender.hashCode();
    	result = 31 * result + age.hashCode();
    	System.out.println("좋은 해시코드 구현 방법 : "+result+"  Object의 hash : "+ Objects.hash(name, gender, age));
    	return result;
    }
    ```

    ```java
    		@DisplayName("hashCode 만들기")
        @Test
        void hashCode_테스트(){
            Member jessi = new Member("Jessi", "woman", "22");
            Member jessiCopy = new Member("Jessi", "woman", "22");
            assertEquals(jessi.hashCode(), jessiCopy.hashCode());
        }
    ```

- 해시 충돌이 적은 방법을 써야 한다면 [com.google.common.hash.Hashing](https://guava.dev/releases/21.0/api/docs/com/google/common/hash/Hashing.html) 를 참고하면 된다.
- Objects 클래스의 hash

  ```java
  @Override
  public int hashCode() {
  	return Objects.hash(name, gender, age);
  }
  //
  public static int hash(Object... values) {
  	return Arrays.hashCode(values);
  }
  //
  public static int hashCode(Object a[]) {
  	if (a == null)
  		return 0;

  	int result = 1;

  	for (Object element : a)
  		result = 31 * result + (element == null ? 0 : element.hashCode());

   return result;
  }
  ```

  - 한 줄로 hashCode를 구현할 수 있지만, 입력인수를 담기 위한 배열 생성과 기본 타입에 대한 박싱/언박싱도 거쳐야 하기 때문에 속도가 느리다. → 성능에 민감하지 않은 상황에서만 사용해야 한다.

- 클래스가 불변이고 hashCode를 계산하는 비용이 크다면 → 캐싱 을 해보자.

  - 클래스의 객체가 주로 해시의 키로 사용된다면 → 인스턴스가 생성될 때 해시코드 계산
  - 해시 키로 사용되지 않으면 → hashCode가 처음 불릴 때 계산 (지연초기화 전략)

    ```java
    private int hashCode; // Automatically initialized to 0

    @Override public int hashCode() {
    	int result = hashCode;
    	if(result == 0){
    		result = Short.hashCode(areaCode);
    		result = 31 * result + Short.hashCode(prefix);
    		result = 31 * result + Short.hashCode(lineNum);
    		hashCode = result;
    	}
    	return result;
    }
    ```

### 핵심 정리

- equals를 재정의할 때는 반드시 hashCode도 재정의 해야한다.
- 재정의한 hashCode는 Object 의 API 문서에 기술된 일반 규약을 따라야 하며, 서로 다른 인스턴스라면 되도록 해시코드도 서로 다르게 구현해야 한다.
- 아이템10의 AutoValue 프레임 워크를 사용하면 자동으로 생성해주기도 한다.
