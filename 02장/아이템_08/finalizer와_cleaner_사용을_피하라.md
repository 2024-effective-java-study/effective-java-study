## finalize란?

Object 클래스에 있는 finalize 메서드를 오버라이드하여 gc가 일어날 때 수행된다.

자바 9부터는 deprecated 되었으며 cleaner 사용을 권장한다.

```java
@Override
protected void finalize() throws Throwable {
		super.finalize();
}
```

## cleaner란?

gc가 일어날 경우 등록한 스레드에서 정의된 클린 작업이 수행된다.

자원 반납용 안전망으로 사용한다.

## 사용을 피해야 하는 이유

### 즉시 수행된다는 보장이 없다.

1. 두 경우 모두 gc의 대상이 되지만 실행되기까지 얼마나 걸릴지 알 수 없다. 즉시 실행되어야 하는 작업은 절대 할 수 없다.
2. 이를 얼마나 신속히 수행할지는 gc 알고리즘에 달렸으며, finalizer 스레드는 우선순위가 낮은 편이라 실행될 기회를 제대로 얻지 못할 것이다.
   → 결국 인스턴스가 gc되지 않고 쌓이다가 OutOfMemoryError가 발생할 수 있다.
3. 파일 닫기를 finalizer나 cleaner에 맡기면 시스템이 동시에 열 수 있는 파일 개수에 제한이 있기 때문에 중대한 오류를 일으킬 수 있다.

### 수행 시점뿐 아니라 수행 여부조차 보장되지 않는다.

1. 프로그램 생애주기와 상관없는, 상태를 영구적으로 수정하는 작업에서는 절대 의존하면 안된다.
2. 데이터베이스 같은 공유 자원의 영구 락 해제를 finalizer나 cleaner에게 맡기면 분산 시스템 전체가 서서히 멈출 것이다.

### finalizer 동작 중 발생한 예외는 무시된다.

1. 처리할 작업이 남았더라도 그 순간 종료된다.
2. 잡지 못한 예외 때문에 해당 객체는 마무리가 덜 된 상태로 남을 수 있다.
3. 그나마 cleaner를 사용하는 라이브러리는 자신의 스레드를 통제하기 때문에 이러한 문제가 발생하지 않는다.

### 성능 문제

1. AutoCloseable, try-with-resources를 통해 객체를 생성하고 gc가 수거하기 까지 12ns가 걸린다.
2. finalizer를 사용하면 550ns, cleaner를 사용하면 500ns 등으로 50배가량 느려진다.
3. 하지만 cleaner를 안전망 형태로 사용하면 66ns 정도로 훨씬 빨라진다.

### 공격에 노출될 수 있다.

1. 객체의 생성자나 직렬화 과정에서 예외가 발생하면, 생성되다 만 객체에서 악의적인 하위 클래스의 finalizer가 수행될 수 있다.

```java
public class KakaoBank {

    private int money;

    public KakaoBank(final int money) {
        if (money < 1000) {
            throw new RuntimeException("1000원 이하로 생성이 불가능해요.");
        }
        this.money = money;
    }

    void transfer(final int money) {
        this.money -= money;
        System.out.println(MessageFormat.format("{0}원 입금 완료!!", money));
    }
}
```

```java
public class BankAttack extends KakaoBank {

    public BankAttack(final int money) {
        super(money);
    }

    @Override
    protected void finalize() throws Throwable {
        this.transfer(1000000000);
    }
}
```

```java
public class Main {

    public static void main(final String[] args) throws InterruptedException {
        KakaoBank bank = null;
        try {
            bank = new BankAttack(500);
            bank.transfer(1000);
        } catch (Exception e) {
            System.out.println("예외 터짐");
        }
        System.gc();
        sleep(3000);
    }
}
```

2. final 클래스는 하위 클래스를 만들 수 없으니 안전하지만, final이 아닌 클래스라면 아무 일도 하지 않는 finalize 메서드를 만들고 final로 선언해야 한다.

```java
@Override
protected final void finalize() throws Throwable {
}
```

## AutoCloseable 구현

1. 클라이언트에서 인스턴스를 다 사용하고 나면 close 메서드를 호출하면 된다.

```java
public class Sample implements AutoCloseable {
    @Override
    public void close() {
        System.out.println("close");
    }
}
```

2. 예외가 발생해도 제대로 종료되도록 try-with-resources 를 사용한다.

## finalizer와 cleaner의 적절한 사용법

1. AutoCloseable의 close 메서드를 호출하지 않는 것에 대비해 cleaner를 사용해 안전망 역할을 수행할 수 있다.
   (그렇다 해도 즉시 호출되리라는 보장은 없지만 클라이언트가 하지 않은 자원 회수를 늦게라도 해주는 편이 낫다.)

```java
public class Room implements AutoCloseable {
    private static final Cleaner cleaner = Cleaner.create();

    // Room을 참조하지 말것!!! 순환 참조
    private static class State implements Runnable { 
        int numJunkPiles;

        State(int numJunkPiles) {
            this.numJunkPiles = numJunkPiles;
        }

				// **colse가 호출되거나, GC가 Room을 수거해갈 때 run() 호출**
        @Override
        public void run() {
            System.out.println("Room Clean");
            numJunkPiles = 0;
        }
    }

    private final State state;
    private final Cleaner.Cleanable cleanable;

    public Room(int numJunkPiles) {
        state = new State(numJunkPiles);
        cleanable = cleaner.register(this, state);
    }

    @Override
    public void close() {
        cleanable.clean();
    }
}
```

위 코드는 안전망을 만들었을 뿐이고 클라이언트가 try-with-resources 블록으로 감싼다면 clean을 정상적으로 수행한다.

```java
public static void main(final String[] args) {
    try (Room myRoom = new Room(8)) {
        System.out.println("방 쓰레기 생성");
    }
}
```

아래와 같은 경우는 clean을 항상 기대할 수는 없다.

```java
public static void main(final String[] args) {
    new Room(8);
    System.out.println("방 쓰레기 생성");
}
```

2. 네이티브 피어란 일반 자바 객체가 네이티브 메서드를 통해 기능을 위임한 네이티브 객체를 말한다. 그 결과 자바 피어를 회수할 때 네이티브 객체까지 회수하지 못 한다. 성능 저하를 감당할 수 없거나 자원을 즉시 회수해야 한다면 close 메서드를 사용해야 한다.
