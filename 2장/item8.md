# finalizer와 cleaner 사용을 피하라.

## 자바의 객체 소멸자 finalizer와 cleaner

- finalizer : 예측할 수 없고, 상황에 따라 위험할 수 있어 일반적으로 불필요하다.
- cleaner : finalizer보다 덜 위험하지만, 여전히 예측할 수 없고, 느리고 일반적으로 불필요하다.

> 왜 cleaner가 finalizer보다 덜 위험할까? finalizer에서는 스레드를 통제하지 않기 때문에 경고를 출력하지 않고, 종료해버리지만, cleaner를 사용하는 라이브러리는 자신의 스레드를 통제하기 때문이다.

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
      
    public class Room implements AutoCloseable {  
      
	     private static final Cleaner cleaner = Cleaner.create();  
	      
	     private static class State implements Runnable {  
	     	int numJunkpiles;  
	      
	     	State(int numJunkpiles) {  
			this.numJunkpiles = numJunkpiles;  
	     	}  
	      
	     	@Override  
	     	public void run() {  
			System.out.println("방 청소하자! 방번호는 [" +numJunkpiles +"] 시간은 [" + LocalDateTime.now()+"]"); 
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

-------

    public class CleanerExample {
	    public static void main(String[] args) throws Exception {

		//룸 청소 1~ 10까지는 clearner를 사용
		for (int i = 1; i <= 10; i++) {
		    System.out.println("방 청소 시작! : " + i);
		    new Room(i);
		}


		//룸 청소 11~ 20까지는 try-with-resource를 이용하여 사용하여 AutoCloseable 활성화
		for (int i = 11; i <= 20; i++) {
		    System.out.println("방 청소 시작! : " + i);
		    try (Room myRoom = new Room(i)) {
		    }
		}

		for (int i = 1; i <= 10000; i++) {
		    int[] a = new int[10000];
		    try {
			Thread.sleep(1);
		    } catch (InterruptedException e) {
		    }
		}
	    }
    }


결과 : 

	방 청소 시작! : 0
	방 청소 시작! : 1
	방 청소 시작! : 2
	방 청소 시작! : 3
	방 청소 시작! : 4
	방 청소 시작! : 5
	방 청소 시작! : 6
	방 청소 시작! : 7
	방 청소 시작! : 8
	방 청소 시작! : 9
	방 청소 시작! : 10
	방 청소 시작! : 11
	방 청소했다! 방번호는 [11] 시간은 [2021-05-16T22:46:29.666385]
	방 청소 시작! : 12
	방 청소했다! 방번호는 [12] 시간은 [2021-05-16T22:46:29.674797]
	방 청소 시작! : 13
	방 청소했다! 방번호는 [13] 시간은 [2021-05-16T22:46:29.674909]
	방 청소 시작! : 14
	방 청소했다! 방번호는 [14] 시간은 [2021-05-16T22:46:29.675019]
	방 청소 시작! : 15
	방 청소했다! 방번호는 [15] 시간은 [2021-05-16T22:46:29.675124]
	방 청소 시작! : 16
	방 청소했다! 방번호는 [16] 시간은 [2021-05-16T22:46:29.675224]
	방 청소 시작! : 17
	방 청소했다! 방번호는 [17] 시간은 [2021-05-16T22:46:29.675416]
	방 청소 시작! : 18
	방 청소했다! 방번호는 [18] 시간은 [2021-05-16T22:46:29.675480]
	방 청소 시작! : 19
	방 청소했다! 방번호는 [19] 시간은 [2021-05-16T22:46:29.675549]
	방 청소 시작! : 20
	방 청소했다! 방번호는 [20] 시간은 [2021-05-16T22:46:29.675611]
	방 청소 시작! : 21
	방 청소했다! 방번호는 [21] 시간은 [2021-05-16T22:46:29.675674]
	방 청소했다! 방번호는 [2] 시간은 [2021-05-16T22:46:30.337386]
	방 청소했다! 방번호는 [0] 시간은 [2021-05-16T22:46:30.337745]
	방 청소했다! 방번호는 [1] 시간은 [2021-05-16T22:46:30.337849]
	방 청소했다! 방번호는 [3] 시간은 [2021-05-16T22:46:30.337915]
	방 청소했다! 방번호는 [4] 시간은 [2021-05-16T22:46:30.337968]
	방 청소했다! 방번호는 [5] 시간은 [2021-05-16T22:46:30.338021]
	방 청소했다! 방번호는 [6] 시간은 [2021-05-16T22:46:30.338081]
	방 청소했다! 방번호는 [7] 시간은 [2021-05-16T22:46:30.338133]
	방 청소했다! 방번호는 [8] 시간은 [2021-05-16T22:46:30.338216]
	방 청소했다! 방번호는 [9] 시간은 [2021-05-16T22:46:30.338266]
	방 청소했다! 방번호는 [10] 시간은 [2021-05-16T22:46:30.338321]
      
   


    






