item 8. finalizer와 cleaner 사용을 피하라
- 책 내용:
    - finalizer나 cleaner를 얼마나 신속하게 수행할지는 전적으로 가비지 컬렉터 알고리즘에 달렸으며, 이는 가비지 컬렉터 구현마다 천차만별이다.
    
- 질문 
    - finalizer나 cleaner를 효과적으로 사용하는 GC 알고리즘이 있는지 궁금합니다.
    
---


item 9. try-finally 보다는 try-with-resourece를 사용하라
- 질문
    - finalizer 스레드는 다른 애플리케이션 스레드보다 우선 순위가 낮아서 실행될 기회를 제대로 얻지 못하는 문제점이 있다.
    - 그렇다면 try-with-resource 에서 close 메소드가 호출될 때 스레드 우선 순위는 대략적으로 얼마나 될까요 ? 
    