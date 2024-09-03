## 정수 열거 패턴

```java
public static final int APPLE_FUJI = 0;
public static final int APPLE_PIPPIN = 1;
public static final int APPLE_GRANNY_SMITH = 2;

public static final int ORANGE_NAVEL = 0;
public static final int ORANGE_TEMPLE = 1;
public static final int ORANGE_BLOOD = 2;
```

타입 안전을 보장할 방법이 없으며 표현력도 좋지 않다.

메서드에 인자를 잘못 보내거나 동등 연산자(==)로 비교하더라도 컴파일러는 아무런 경고 메세지를 출력하지 않는다.

상수의 값이 바뀌면 클라이언트도 반드시 다시 컴파일 해야한다.

정수 상수는 그 값을 출력하거나 디버거로 살펴보아도 의미가 아닌 단지 숫자로만 보여서 도움이 되지 않는다.

## 단순한 열거 타입

```java
public enum Apple { FUJI, PIPPIN, GRANNY_SMITH }
public enum Orange { NAVEL, TEMPLE, BLOOD }
```

자바의 열거 타입은 완전한 형태의 클래스라서 다른 언어의 열거 타입보다 훨씬 강력하다.

상수 하나당 자신의 인스턴스를 만들어 public static final 필드로 공개한다.

클라이언트가 인스턴스를 직접 생성하거나 확장할 수 없으니 열거 타입 선언으로 만들어진 인스턴스들은 딱 하나씩만 존재한다.

열거 타입은 컴파일타입 타입 안전성을 제공한다. Apple 열거 타입을 메개변수로 받는 메서드에 다른 타입의 값을 넘기려 하면 컴파일 오류가 난다.

열거 타입에는 각자의 이름공간이 있어서 이름이 같은 상수도 평화롭게 공존할 수 있다.

열거 타입에는 임의의 메서드나 필드를 추가할 수 있고 임의의 인터페이스를 구현할 수도 있다.

## 데이터와 메서드를 갖는 열거 타입

```java
public enum Planet {
    MERCURY(3.302e+23, 2.439e6),
    VENUS  (4.869e+24, 6.052e6),
    EARTH  (5.975e+24, 6.378e6),
    MARS   (6.419e+23, 3.393e6),
    JUPITER(1.899e+27, 7.149e7),
    SATURN (5.685e+26, 6.027e7),
    URANUS (8.683e+25, 2.556e7),
    NEPTUNE(1.024e+26, 2.477e7);

    private final double mass;           // 질량(단위: 킬로그램)
    private final double radius;         // 반지름(단위: 미터)
    private final double surfaceGravity; // 표면중력(단위: m / s^2)

    // 중력상수(단위: m^3 / kg s^2)
    private static final double G = 6.67300E-11;

    // 생성자
    Planet(double mass, double radius) {
        this.mass = mass;
        this.radius = radius;
        surfaceGravity = G * mass / (radius * radius);
    }

    public double mass()           { return mass; }
    public double radius()         { return radius; }
    public double surfaceGravity() { return surfaceGravity; }
    
    // 대상 객체의 질량을 입력받아, 그 객체가 행성 표면에 있을 떄의 무게를 반환
    public double surfaceWeight(double mass) {
        return mass * surfaceGravity;  // F = ma
    }
}
```

열거 타입 상수 각각을 특정 데이터와 연결지으려면 생성자에서 데이터를 받아 인스턴스 필드에 저장하면 된다.

열거 타입은 근본적으로 불변이라 모든 필드는 final이어야 한다.

필드는 private으로 두고 별도의 public 접근자 메서드를 두는 것이 낫다.

열거 타입은 자신 안에 정의된 상수들의 값을 배열에 담아 반환하는 정적 메서드인 values를 제공한다.

기본 toString이 제공하는 메서드는 상수 이름을 문자열로 반환한다.

### 열거 타입에서 상수를 하나 제거한다면?

제거한 상수를 참조하지 않는 클라이언트에는 아무 영향이 없다.

제거된 상수를 참조하는 클라이언트에서는 클라이언트 프로그램을 다시 컴파일하면 디버깅에 유용한 메세지를 담은 컴파일 오류가 발생할 것이다.

### 올바른 사용방법

열거 타입을 선언한 클래스 혹은 그 패키지에서만 유용한 기능은 private이나 package-private 메서드로 구현한다.

일반 클래스와 마찬가지로, 그 기능을 클라이언트에 노출해야 할 합당한 이유가 없다면 private으로, 혹은 package-private으로 선언해야 한다.

## 값에 따라 분기하는 열거 타입

```java
public enum Operation {
    PLUS("+") {
        public double apply(double x, double y) { return x + y; }
    },
    MINUS("-") {
        public double apply(double x, double y) { return x - y; }
    },
    TIMES("*") {
        public double apply(double x, double y) { return x * y; }
    },
    DIVIDE("/") {
        public double apply(double x, double y) { return x / y; }
    };

    private final String symbol;

    Operation(String symbol) { this.symbol = symbol; }
    
    @Override public String toString() { return symbol; }
    public abstract double apply(double x, double y);    
}
```

**상수별 메서드 구현** : 열거 타입에 추상 메서드를 선언하고 각 상수별 클래스 몸체, 즉 각 상수에서 자신에 맞게 재정의하는 방법이다.

apply는 추상 메서드이므로 재정의하지 않아도 컴파일 오류로 알려준다.

열거 타입에는 상수 이름을 입력받아 그 이름에 해당하는 상수를 반환해주는 valueOf(String) 메서드가 자동 생성된다.

열거 타입의 toString 메서드를 재정의 하려거든, toString이 반환하는 문자열을 해당 열거 타입 상수로 변환해주는 fromString 메서드도 제공하는 것을 고려해보자.

## 전략 열거 타입 패턴

```java
enum PayrollDay {
    MONDAY(WEEKDAY), TUESDAY(WEEKDAY), WEDNESDAY(WEEKDAY),
    THURSDAY(WEEKDAY), FRIDAY(WEEKDAY),
    SATURDAY(WEEKEND), SUNDAY(WEEKEND);

    private final PayType payType;

    PayrollDay(PayType payType) { this.payType = payType; }

    int pay(int minutesWorked, int payRate) {
        return payType.pay(minutesWorked, payRate);
    }

    enum PayType {
        WEEKDAY {
            int overtimePay(int minsWorked, int payRate) {
                return minsWorked <= MINS_PER_SHIFT ? 0 :
                        (minsWorked - MINS_PER_SHIFT) * payRate / 2;
            }
        },
        WEEKEND {
            int overtimePay(int minsWorked, int payRate) {
                return minsWorked * payRate / 2;
            }
        };

        abstract int overtimePay(int mins, int payRate);
        private static final int MINS_PER_SHIFT = 8 * 60;

        int pay(int minsWorked, int payRate) {
            int basePay = minsWorked * payRate;
            return basePay + overtimePay(minsWorked, payRate);
        }
    }

    public static void main(String[] args) {
        for (PayrollDay day : values())
            System.out.printf("%-10s%d%n", day, day.pay(8 * 60, 1));
    }
}
```

private 중첩 열거 타입을 선언해 각 상수별로 중첩 열거 타입의 상수를 사용할 수 있다.

switch 문을 사용하는 것보다 복잡하지만 더 안전하고 유연하다.

## 결론

열거 타입은 필요한 원소를 컴파일타임에 다 알 수 있는 상수 집합일 때 사용하자.

열거 타입에 정의된 상수 개수가 영원히 고정 불변일 필요는 없다.

대다수의 열거 타입이 명시적 생성자나 메서드 없이 쓰이지만, 각 상수를 특정 데이터와 연결짓거나 상수마다 다르게 동작하게 할 때는 필요하다.

드물게, 하나의 메서드가 상수별로 다르게 동작해야 할 때도 있는데, 이 열거 타입에서는 switch문 대신 상수별 메서드 구현을 사용하자.

열거 타입의 상수 일부가 같은 동작을 공유한다면 전략 열거 타입 패턴을 사용하자.