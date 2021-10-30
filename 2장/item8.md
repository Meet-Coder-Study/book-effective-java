# finalizer와 cleaner 사용을 피하라.

## 자바의 객체 소멸자 finalizer와 cleaner

- finalizer : 예측할 수 없고, 상황에 따라 위험할 수 있어 일반적으로 불필요하다.
- cleaner : finalizer보다 덜 위험하지만, 여전히 예측할 수 없고, 느리고 일반적으로 불필요하다.

> 왜 cleaner보다 finalizer가 덜 위험할까? finalizer에서는 스레드를 통제하지 않기 때문에 경고를 출력하지 않고, 종료해버리지만, cleaner를 사용하는 라이브러리는 자신의 스레드를 통제하기 때문이다.

## 왜 지양해야 하는가? 

1. 즉시 수행된다는 보장이 없다.
-  finalizer나 cleaner를 얼마나 신속히 수행할지는 전적으로 가비지 컬렉션에 달렸다.
2. 수행 여부도 보장하지 않는다.
- 접근 할 수 없는 일부 객체에 딸린 종료 작업을 전혀 수행하지 못한 채 프로그램이 중단 될 수도 있다. 
- 즉, 상태를 영구적으로 수정하는 작업에서는 절대 finalizer나 cleaner에 의존해서는 안 된다.
3. 심각한 성능 문제도 동반한다.
-  AutoCloseable 사용하여 처리했을때보다 약 50배 느리다.
4. finalizer를 사용한 클래스는 finalizer 공격에 노출되어 심각한 보안 문제를 일으킬 수도 있다.
- 생성자나 직렬화 과정에서 예외가 발생하면, 이 생성되다 만 객체에서 악의적인 하위 클래스의 finalizer가 수행될 수 있기 때문에 가비지 컬렉터가 수집하지 못하게 막을 수 있다.

## 이렇게 지양하는데 왜 finalizer와 cleaner가 있는건데?

1. 자원의 소유자가 close 메서드를 호출하지 않는 것에 대비한 안전망 역할
2. 네이티브 피어를 회수하기 위해서이다. 왜냐하면 네이티브 피어는 자바 객체가 아니기 때문에 가비지 컬렉터에서 회수하지 못하기 때문이다.
-  즉시 자원을 회수해야 한다면 close 메서드를 사용해야한다.

## finalizer나 cleaner를 대신해줄 묘안은 무엇일까?

1. **AutoCloseable을 구현해주고, 클라이언트에서 인스턴스를 다 쓰고 나면 close 메서드를 호출해주면 된다.**

2. **AutoCloseable에서 Cleaner를 사용하여 해주는 경우, try-with-resources을 사용하자.**

  
  ## 책 예제

    import java.lang.ref.Cleaner;  
      
    public class Room implements AutoCloseable {  
      
	      private static final Cleaner cleaner = Cleaner.create();  
	      
	      private static class State implements Runnable {  
	      int numJunkpiles;  
	      
	      State(int numJunkpiles) {  
		      this.numJunkpiles = numJunkpiles;  
	      }  
	      
	      @Override  
	      public void run() {  
		      System.out.println("방 청소");  
		      numJunkpiles = 0;  
	      }  
	     }  
	     private final State state;  
	     private final Cleaner.Cleanable cleanable;  
	      
	     public Room(int numJunkpiles) {  
	      state = new State(numJunkpiles);  
	      cleanable = cleaner.register(this, state);  
	      }  
	      
	      @Override  
	      public void close() throws Exception {  
	      cleanable.clean();  
	      }  
    }

    public class main {  
      public static void main(String[] args) throws Exception {  
	      try (Room myRoom = new Room(7)) {
		      System.out.println("안녕~");
		  }
		  new Room(99);
		  System.out.println("아무렴~");
	  }

결과 : 

    안녕~
    방청소
    아무렴
      
   


    






