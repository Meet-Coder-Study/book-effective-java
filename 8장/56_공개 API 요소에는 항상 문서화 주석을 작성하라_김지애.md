# [Effective Java] item 56. 공개된 API 요소에는 항상 문서화 주석을 사용하라

## 1. 자바독(Javadoc)이란?
- API 문서화 유틸리티
- 소스코드 파일에서 문서화 주석(자바독 주석)이라는특수한 형태로 기술된 설명을 추려 API 문서로 변환해준다.

## 2. javadoc 사용법
#### javadoc 명령어 사용
```
$ javadoc -d docs {file_name}
```
##### 한글 사용시 UTF-8로 인코딩 필요
```
$ javadoc -d docs *.java -encoding UTF-8 -charset UTF-8 -docencoding UTF-8
```
#### 2.1. javadoc 명령어 실행
![item56_1](https://user-images.githubusercontent.com/37948906/110593923-85e44b80-81bf-11eb-9ca2-9fb0339cf6d7.PNG)

#### 2.2. javadoc에서 자동으로 웹페이지 생성
![item56_3](https://user-images.githubusercontent.com/37948906/110594657-6ef22900-81c0-11eb-8987-04de9d2f33ec.PNG)

#### 2.3. 주석에 작성한대로 api 문서 생성
![item56_2](https://user-images.githubusercontent.com/37948906/110594680-787b9100-81c0-11eb-954d-6e10e6448189.PNG)

## 3. javadoc 주석 유의 사항
#### 3.1 API를 올바르게 문서화하려면 공개된 모든 클래스, 인터페이스, 메서드, 필드 선언에 문서화 주석을 달아야 한다.
- 주석을달지 않으면 그저 공개된 API 요소들의 '선언'만을 나열한다.
- 공개 클래스는 기본 생성자에 주석을 달 수 있는 방법이 없으니 절대 기본 생성자를 사용해서는 안된다.
#### 3.2 메서드용 문서화 주석에는 해당 메서드와 클라이언트 사이의 규약을 명료하게 기술해야 한다.
- how가 아닌 what을 기술할 것. (상속용으로 설계된 API가 아닌 이상)
- 메서드를 성공적으로 호출하기 위한 전제조건을 나열할 것
- 메서드가 성공적으로 수행된 후에 만족해야 하는 사후조건을 나열할 것
- 부작용(사후조건으로 명확하게 나타나지는 않지만 시스템의 상태에 어떠한 변화를 가져오는 것)도 문서화할 것 (ex. 백그라운드 스레드를 시작하는 메서드)

## 4. javadoc 주석 태그
#### 4.1 `@param`
- 매개변수가 뜻하는 명사구
- 모든 매개변수에 설명을 달아야 한다.
#### 4.2 `@return`
- 반환 값을 설명하는 명사구
- 반환 타입이 void가 아니라면 달아야 한다.
#### 4.3 `@throws`
- if로 시작해 해당 예외를 던지는 조건을 설명하는 절
#### 4.4 `{@code}`
- 주석 내에 HTML 요소나 다른 자바독 태그를 무시한다.
- 주석에 여러 줄로 된 코드 예시를 넣으려면 `{@code}`를 `<pre>`태그로 감싸준다. `<pre>{@code ...코드... }</pre>`
#### 4.5 `{@literal}`
- 주석 내에 HTML 요소나 다른 자바독 태그를 무시한다.
- `{@code}`와 비슷하지만 코드 폰트로 렌더링하지 않는다.
#### 4.6 `@implSpec`
- 해당 메서드와 하위 클래스 사이의 계약을 설명한다.
- 하위 클래스들이 그 메서드를 상속하거나 super 키워드를 이용해 호출할 때 그 메서드가 어떻게 동작하는지 명확히 인지할 수 있도록 도와준다.
#### 4.7 `{@inheritDoc}`
- 상위 타입의 문서화 주석 일부를 상속할 수 있다.

##### 예시
```java
package chapter8.item56;

/**
 * @author jiae
 * 게임 캐릭터의 정보를 가지고 있는 클래스
 */
public class GameCharactor {
    private int level;
    private String name;
    private String weapon;
    private Job job;
    /**
     * 게임 캐릭터를 생성합니다.
     * <p>기본 무기는 목검, 기본 직업은 Beginner입니다.
     * @param name 캐릭터의 이름; 길이는 3자 이상 10자 이하이어야 합니다.
     * @throws IllegalArgumentException 캐릭터의 name 길이가 정해진 범위를 벗어나면, 즉 ({@code name < 3 || name > 10}) 이면 발생합니다.
     */
    public GameCharactor(String name) {
        this.level = 1;
        if (name.length() < 3 || name.length() > 10) throw new IllegalArgumentException("캐릭터의 이름은 3자 이상 10자 이하입니다.");
        this.name = name;
        this.weapon = "목검";
        this.job = Job.Beginner;
    }

    /**
     * 캐릭터의 레벨을 반환합니다.
     * @return 캐릭터의 레벨
     */
    public int getLevel(){
        return this.level;
    }

    /**
     * 캐릭터의 직업을 변경합니다.
     * @param job 캐릭터의 변경할 직업
     * @throws IllegalArgumentException 캐릭터의 레벨이 10이 넘지 않았다면 발생합니다.
     */
    public void setJob(Job job){
        if (this.level < 10) throw new IllegalArgumentException("캐릭터의 레벨이 10을 넘지 않습니다.");
        this.job = job;
    }

    /**
     * 캐릭터의 무기를 변경해주는 메서드입니다.
     * @param weapon 캐릭터가 착용할 무기
     * @param weaponLevel 무기의 레벨
     * @throws IllegalArgumentException 캐릭터의 레벨보다 무기의 레벨이 높으면 발생합니다.
     */
    public void setWeapon(String weapon, int weaponLevel){
        if (weaponLevel > this.level) throw new IllegalArgumentException("캐릭터의 레벨보다 무기의 레벨이 높습니다.");
        this.weapon = weapon;
    }

    /**
     * 캐릭터의 레벨을 올려주는 메서드입니다.
     */
    public void levelUp(){
        this.level++;
    }

    /**
     * 캐릭터의 status값을 보여주는 메서드입니다.
     * @return 직업, 레벨, 이름, 무기를 반환합니다.
     */
    public String getCharactorStatus() {
        return "GameCharactor [job=" + job + ", level=" + level + ", name=" + name + ", weapon=" + weapon + "]";
    }
    
    /**
     * 캐릭터의 직업을 나타냅니다.
     */
    public enum Job{
        /** 초보자 */
        Beginner, 
        /** 전사 */
        Warrior, 
        /** 마법사 */
        Wizard, 
        /** 궁수 */
        Archer, 
        /** 도적 */
        Thief
    }
}
```

![item56_4](https://user-images.githubusercontent.com/37948906/110622085-585aca80-81de-11eb-8a5c-462336ec3b88.PNG)
![item56_5](https://user-images.githubusercontent.com/37948906/110622106-5e50ab80-81de-11eb-9911-d4a97bf09861.PNG)
![item56_6](https://user-images.githubusercontent.com/37948906/110622110-5ee94200-81de-11eb-9025-e498369ef92e.PNG)
![item56_7](https://user-images.githubusercontent.com/37948906/110622116-601a6f00-81de-11eb-8fbb-c2a959c91091.PNG)


## 5. API 문서화에서 자주 누락되는 설명 두가지를 포함하자
- 1. 클래스 혹은 정적 메서드가 스레드 안전하든 그렇지 않든, `쓰레드 안전 수준`을 반드시 API 설명에 포함해야 한다.
- 2. 직렬화할 수 있는 클래스라면 직렬화 형태도 API 설명에 기술해야 한다.

## 핵심 정리
- 문서화 주석은 aPI를 문서화하는 가장 훌륭하고 효과적인 방법이다.
- 공개 API라면 빠짐없이 설명을 달아야 한다.
- 표준 규약을 일관되게 지키자.
- 문서화 주석에 임의의 HTML 태그를 사용할 수 있음을 기억하라. 단, HTML 메타문자는 특별하게 취급해야 한다.

### 참고 자료
- Effective Java 3/E