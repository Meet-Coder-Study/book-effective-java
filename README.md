# book-effective-java

## 참고 링크
- [스터디 운영방안](https://www.notion.so/5eb841d5c4b8401bac02ab8ffd71d1a9)
- [책 정보 링크](https://www.aladin.co.kr/shop/wproduct.aspx?ItemId=171196410)
- [이펙티브자바 공식 Github](https://github.com/WegraLee/effective-java-3e-source-code)
- [이펙티브자바 3판 번역 용어 설명](https://docs.google.com/document/d/1Nw-_FJKre9x7Uy6DZ0NuAFyYUCjBPCpINxqrP0JFuXk/edit)

## 2장. 객체 생성과 파괴
| 아이템 | 발표자료
:---: | :---:
[아이템1. 생성자 대신 정적 팩터리 메서드를 고려하라](https://github.com/Blog-Posting/book-effective-java/issues/1) | [김민걸](https://github.com/Blog-Posting/book-effective-java/blob/main/2%EC%9E%A5/1_%EC%83%9D%EC%84%B1%EC%9E%90_%EB%8C%80%EC%8B%A0_%EC%A0%95%EC%A0%81%20%ED%8C%A9%ED%84%B0%EB%A6%AC_%EB%A9%94%EC%84%9C%EB%93%9C%EB%A5%BC_%EA%B3%A0%EB%A0%A4%ED%95%98%EB%9D%BC_%EA%B9%80%EB%AF%BC%EA%B1%B8.md) / [최락준](https://github.com/Blog-Posting/book-effective-java/blob/main/2%EC%9E%A5/1_%EC%83%9D%EC%84%B1%EC%9E%90_%EB%8C%80%EC%8B%A0_%EC%A0%95%EC%A0%81%20%ED%8C%A9%ED%84%B0%EB%A6%AC_%EB%A9%94%EC%84%9C%EB%93%9C%EB%A5%BC_%EA%B3%A0%EB%A0%A4%ED%95%98%EB%9D%BC_%EC%B5%9C%EB%9D%BD%EC%A4%80.md)
[아이템2. 생성자에 매개변수가 많다면 빌더를 고려하라](https://github.com/Blog-Posting/book-effective-java/issues/2) | [유효정](https://github.com/Blog-Posting/book-effective-java/blob/main/2%EC%9E%A5/2_%EC%83%9D%EC%84%B1%EC%9E%90%EC%97%90_%EB%A7%A4%EA%B0%9C%EB%B3%80%EC%88%98%EA%B0%80_%EB%A7%8E%EB%8B%A4%EB%A9%B4_%EB%B9%8C%EB%8D%94%EB%A5%BC_%EA%B3%A0%EB%A0%A4%ED%95%98%EB%9D%BC_%EC%9C%A0%ED%9A%A8%EC%A0%95.pdf) / [박창원](https://github.com/Blog-Posting/book-effective-java/blob/main/2%EC%9E%A5/2_%EC%83%9D%EC%84%B1%EC%9E%90%EC%97%90%20%EB%A7%A4%EA%B0%9C%EB%B3%80%EC%88%98%EA%B0%80%20%EB%A7%8E%EB%8B%A4%EB%A9%B4%20%EB%B9%8C%EB%8D%94%EB%A5%BC%20%EA%B3%A0%EB%A0%A4%ED%95%98%EB%9D%BC_%EB%B0%95%EC%B0%BD%EC%9B%90.md)  
[아이템3. private 생성자나 열거 타입으로 싱글턴임을 보증하라](https://github.com/Blog-Posting/book-effective-java/issues/3) | [김보배](https://github.com/Blog-Posting/book-effective-java/blob/main/2%EC%9E%A5/3_private%20%EC%83%9D%EC%84%B1%EC%9E%90%EB%82%98%20%EC%97%B4%EA%B1%B0%20%ED%83%80%EC%9E%85%EC%9C%BC%EB%A1%9C%20%EC%8B%B1%EA%B8%80%ED%84%B4%EC%9E%84%EC%9D%84%20%EB%B3%B4%EC%A6%9D%ED%95%98%EB%9D%BC_%EA%B9%80%EB%B3%B4%EB%B0%B0.md) 
[아이템4. 인스턴스화를 막으려거든 private 생성자를 사용하라](https://github.com/Blog-Posting/book-effective-java/issues/4) | [이호빈](https://github.com/Blog-Posting/book-effective-java/blob/main/2%EC%9E%A5/4_%EC%9D%B8%EC%8A%A4%ED%84%B4%EC%8A%A4%ED%99%94%EB%A5%BC_%EB%A7%89%EC%9C%BC%EB%A0%A4%EA%B1%B0%EB%93%A0_private_%EC%83%9D%EC%84%B1%EC%9E%90%EB%A5%BC_%EC%82%AC%EC%9A%A9%ED%95%98%EB%9D%BC_%EC%9D%B4%ED%98%B8%EB%B9%88.md) / [박지은](https://github.com/Blog-Posting/book-effective-java/blob/main/2%EC%9E%A5/4_%EC%9D%B8%EC%8A%A4%ED%84%B4%EC%8A%A4%ED%99%94%EB%A5%BC_%EB%A7%89%EC%9C%BC%EB%A0%A4%EA%B1%B0%EB%93%A0_private_%EC%83%9D%EC%84%B1%EC%9E%90%EB%A5%BC_%EC%82%AC%EC%9A%A9%ED%95%98%EB%9D%BC_%EB%B0%95%EC%A7%80%EC%9D%80.dㅛ)
[아이템5. 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라](https://github.com/Blog-Posting/book-effective-java/issues/5) | [황준호](https://github.com/Blog-Posting/book-effective-java/blob/main/2%EC%9E%A5/5_%EC%9E%90%EC%9B%90%EC%9D%84_%EC%A7%81%EC%A0%91_%EB%AA%85%EC%8B%9C%ED%95%98%EC%A7%80_%EB%A7%90%EA%B3%A0_%EC%9D%98%EC%A1%B4_%EA%B0%9D%EC%B2%B4_%EC%A3%BC%EC%9E%85%EC%9D%84_%EC%82%AC%EC%9A%A9%ED%95%98%EB%9D%BC_%ED%99%A9%EC%A4%80%ED%98%B8.md) / [김지애](https://github.com/Blog-Posting/book-effective-java/blob/main/2%EC%9E%A5/5_%EC%9E%90%EC%9B%90%EC%9D%84%20%EC%A7%81%EC%A0%91%20%EB%AA%85%EC%8B%9C%ED%95%98%EC%A7%80%20%EB%A7%90%EA%B3%A0%20%EC%9D%98%EC%A1%B4%20%EA%B0%9D%EC%B2%B4%20%EC%A3%BC%EC%9E%85%EC%9D%84%20%EC%82%AC%EC%9A%A9%ED%95%98%EB%9D%BC_jiae.md)   
[아이템6. 불필요한 객체 생성을 피하라](https://github.com/Blog-Posting/book-effective-java/issues/6) | [신선영](https://github.com/Blog-Posting/book-effective-java/blob/main/2%EC%9E%A5/6_%EB%B6%88%ED%95%84%EC%9A%94%ED%95%9C_%EA%B0%9D%EC%B2%B4_%EC%83%9D%EC%84%B1%EC%9D%84_%ED%94%BC%ED%95%98%EB%9D%BC_%EC%8B%A0%EC%84%A0%EC%98%81.md) / [박경철](https://github.com/Blog-Posting/book-effective-java/blob/main/2%EC%9E%A5/6_%EB%B6%88%ED%95%84%EC%9A%94%ED%95%9C_%EA%B0%9D%EC%B2%B4_%EC%83%9D%EC%84%B1%EC%9D%84_%ED%94%BC%ED%95%98%EB%9D%BC_%EB%B0%95%EA%B2%BD%EC%B2%A0.md)
[아이템7. 다 쓴 객체 참조를 해제하라](https://github.com/Blog-Posting/book-effective-java/issues/7) |  [김세윤](https://github.com/Blog-Posting/book-effective-java/blob/main/2%EC%9E%A5/7_%EB%8B%A4%20%EC%93%B4%20%EA%B0%9D%EC%B2%B4%20%EC%B0%B8%EC%A1%B0%EB%A5%BC%20%ED%95%B4%EC%A0%9C%ED%95%98%EB%9D%BC_%EA%B9%80%EC%84%B8%EC%9C%A4.md) / [이주현](https://github.com/Blog-Posting/book-effective-java/blob/main/2%EC%9E%A5/7_%EB%8B%A4%20%EC%93%B4%20%EA%B0%9D%EC%B2%B4%20%EC%B0%B8%EC%A1%B0%EB%A5%BC%20%ED%95%B4%EC%A0%9C%ED%95%98%EB%9D%BC_%EC%9D%B4%EC%A3%BC%ED%98%84.md)
[아이템8. finalizer와 cleaner 사용을 피하라](https://github.com/Blog-Posting/book-effective-java/issues/8) |  [신선영](https://github.com/Blog-Posting/book-effective-java/blob/main/2%EC%9E%A5/8_finalizer%EC%99%80_cleaner_%EC%82%AC%EC%9A%A9%EC%9D%84_%ED%94%BC%ED%95%98%EB%9D%BC_%EC%8B%A0%EC%84%A0%EC%98%81.md)
[아이템9. try-finally보다는 try-with-resources를 사용하라](https://github.com/Blog-Posting/book-effective-java/issues/9) | [황준호](https://github.com/Blog-Posting/book-effective-java/blob/main/2%EC%9E%A5/9_try_finally%EB%B3%B4%EB%8B%A4%EB%8A%94_try_with_resources%EB%A5%BC_%EC%82%AC%EC%9A%A9%ED%95%98%EB%9D%BC_%ED%99%A9%EC%A4%80%ED%98%B8.md) / [김보배](https://github.com/Blog-Posting/book-effective-java/blob/main/2%EC%9E%A5/9_try-finally%EB%B3%B4%EB%8B%A4%EB%8A%94%20try-with-resources%EB%A5%BC%20%EC%82%AC%EC%9A%A9%ED%95%98%EB%9D%BC_%EA%B9%80%EB%B3%B4%EB%B0%B0.md)

## 3장. 모든 객체의 공통 메서드
| 아이템 | 발표자료
:---: | :---:
[아이템10. equals는 일반 규약을 지켜 재정의하라](https://github.com/Blog-Posting/book-effective-java/issues/10) | [김세윤](https://github.com/Blog-Posting/book-effective-java/blob/main/3%EC%9E%A5/10_equals%EB%8A%94%20%EC%9D%BC%EB%B0%98%20%EA%B7%9C%EC%95%BD%EC%9D%84%20%EC%A7%80%EC%BC%9C%20%EC%9E%AC%EC%A0%95%EC%9D%98%ED%95%98%EB%9D%BC_%EA%B9%80%EC%84%B8%EC%9C%A4.md) / [최락준](https://github.com/Blog-Posting/book-effective-java/blob/main/3%EC%9E%A5/10_equals%EB%8A%94%20%EC%9D%BC%EB%B0%98%20%EA%B7%9C%EC%95%BD%EC%9D%84%20%EC%A7%80%EC%BC%9C%20%EC%9E%AC%EC%A0%95%EC%9D%98%ED%95%98%EB%9D%BC_%EC%B5%9C%EB%9D%BD%EC%A4%80.md)
[아이템11. equals를 재정의하려거든 hashCode도 재정의하라](https://github.com/Blog-Posting/book-effective-java/issues/11) | [박경철](https://github.com/Blog-Posting/book-effective-java/blob/main/3%EC%9E%A5/11_equals%EB%A5%BC_%EC%9E%AC%EC%A0%95%EC%9D%98%ED%95%98%EB%A0%A4%EA%B1%B0%EB%93%A0_hashCode%EB%8F%84_%EC%9E%AC%EC%A0%95%EC%9D%98%ED%95%98%EB%9D%BC_%EB%B0%95%EA%B2%BD%EC%B2%A0.md)
[아이템12. toString을 항상 재정의하라](https://github.com/Blog-Posting/book-effective-java/issues/12) | [이호빈](https://github.com/Blog-Posting/book-effective-java/blob/main/3%EC%9E%A5/12_toString%EC%9D%84_%ED%95%AD%EC%83%81_%EC%9E%AC%EC%A0%95%EC%9D%98%ED%95%98%EB%9D%BC_%EC%9D%B4%ED%98%B8%EB%B9%88.md)
[아이템13. clone 재정의는 주의해서 진행하라](https://github.com/Blog-Posting/book-effective-java/issues/13) | [김민걸](https://github.com/Blog-Posting/book-effective-java/blob/main/3%EC%9E%A5/13_clone_%EC%9E%AC%EC%A0%95%EC%9D%98%EB%8A%94_%EC%A3%BC%EC%9D%98%ED%95%B4%EC%84%9C_%EC%A7%84%ED%96%89%ED%95%98%EB%9D%BC_%EA%B9%80%EB%AF%BC%EA%B1%B8.md) / [박창원](https://github.com/Blog-Posting/book-effective-java/blob/main/3%EC%9E%A5/13_clone_%EC%9E%AC%EC%A0%95%EC%9D%98%EB%8A%94_%EC%A3%BC%EC%9D%98%ED%95%B4%EC%84%9C_%EC%A7%84%ED%96%89%ED%95%98%EB%9D%BC_%EB%B0%95%EC%B0%BD%EC%9B%90.md)
[아이템14. Comparable을 구현할지 고려하라](https://github.com/Blog-Posting/book-effective-java/issues/14) | [이주현](https://github.com/Blog-Posting/book-effective-java/blob/main/3%EC%9E%A5/14_Comparable%EC%9D%84_%EA%B5%AC%ED%98%84%ED%95%A0%EC%A7%80_%EA%B3%A0%EB%A0%A4%ED%95%98%EB%9D%BC_%EC%9D%B4%EC%A3%BC%ED%98%84.md)

## 4장. 클래와 인터페이스
| 아이템 | 발표자료
:---: | :---:
[아이템 15. 클래스와 멤버의 접근 권한을 최소화하라](https://github.com/Meet-Coder-Study/book-effective-java/issues/15) | 
[아이템 16. public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라](https://github.com/Meet-Coder-Study/book-effective-java/issues/16) |
[아이템 17. 변경 가능성을 최소화하라](https://github.com/Meet-Coder-Study/book-effective-java/issues/17) |
[아이템 18. 상속보다는 컴포지션을 사용하라](https://github.com/Meet-Coder-Study/book-effective-java/issues/18) |
[아이템 19 상속을 고려해 설계하고 문서화하라. 그러지 않았다면 상속을 금지하라.](https://github.com/Meet-Coder-Study/book-effective-java/issues/19) |
[아이템 20. 추상 클래스보다는 인터페이스를 우선하라](https://github.com/Meet-Coder-Study/book-effective-java/issues/20) |
[아이템 21. 인터페이스는 구현하는 쪽을 생각해 설계하라](https://github.com/Meet-Coder-Study/book-effective-java/issues/21) |
[아이템 22. 인터페이스는 타입을 정의하는 용도로만 사용하라](https://github.com/Meet-Coder-Study/book-effective-java/issues/22) |
[아이템 23. 태그 달린 클래스보다는 클래스 계층구조를 활용하라](https://github.com/Meet-Coder-Study/book-effective-java/issues/23) |
[아이템 24. 멤버 클래스는 되도록 static으로 만들라](https://github.com/Meet-Coder-Study/book-effective-java/issues/24) |
[아이템 25. 톱레벨 클래스는 한 파일에 하나만 담으라](https://github.com/Meet-Coder-Study/book-effective-java/issues/25) | 

