## equals 메서드를 재정의하지 않는 것이 좋은 경우

아래의 상황 중 하나라도 해당한다면 재정의하지 않는 것이 좋다.

### 각 인스턴스가 본질적으로 고유하다.

값을 표현하는 것이 아니라 동작하는 개체를 표현하는 클래스 (Thread)

```java
@Override
public boolean equals(Object obj) {
    if (obj == this)
        return true;

    if (obj instanceof WeakClassKey) {
        Object referent = get();
        return (referent != null) && (referent == ((WeakClassKey) obj).get());
    } else {
        return false;
    }
}
```

### 인스턴스의 ‘논리적 동치성’을 검사할 일이 없다.

Pattern의 equals는 인스턴스가 같은 정규표현식을 나타내는지 검사하기 위해 equals를 재정의했다.

### 상위 클래스에서 재정의한 equals가 하위 클래스에도 들어맞는다.

Set, List, Map의 구현체들은 모두 각각 AbstractSet, AbstractList, AbstractMap이 구현한 equals를 상속받아 쓴다.

서로 같은 Set, List, Map인지 구분하기 위해 사이즈, 내부 값들을 비교하면 되기 때문이다.

### 클래스가 private이거나 package-private이고 equals 메서드를 호출할 일이 없다.

```java
@Override 
public boolean equals(Object o){
    throw new AssertionError(); // 호출 금지
}
```

## equals를 재정의해야 하는 경우

논리적 동치성을 확인해야 하는데, 상위 클래스의 equals가 논리적 동치성을 비교하도록 재정의되지 않았을 때

주로 Integer나 String처럼 값을 표현하는 값 클래스들이 여기 해당한다.

Enum과 같이 논리적으로 같은 인스턴스가 2개 이상 만들어지지 않음을 보장하므로 eqals를 재정의 하지 않아도 된다.

## equals의 일반 규약

모든 참조 값 x, y, z는 null이 아니라고 가정한다.

> 반사성: x.equals(x)는 true이다. <br>대칭성: x.equals(y)가 true이면, y.equals(x)도 true이다. <br>추이성: x.equals(y)와 y.equals(z)가 true이면, x.equals(z)도 true이다. <br>일관성: x.equals(y)를 반복해서 호출하면 항상 true를 반환하거나 항상 false를 반환한다. <br>null-아님: x.equals(null)은 false이다.

## 반사성

> x.equals(x)는 true이다.

객체는 자기 자신과 같아야 한다.

일부러 어기는 경우가 아니면 만족시키지 못하기가 더 어렵다.

이 요건을 어긴 클래스의 인스턴스를 컬렉션에 넣은 다음 contains 메서드를 호출하면 존재하지 않는다고 할 것이다.

## 대칭성

> x.equals(y)가 true이면, y.equals(x)도 true이다.

서로에 대한 동치 여부가 같아야 한다.

대소문자를 구별하지 않는 문자열을 구현한 클래스 CaseInsensitiveString

```java
public final class CaseInsensitiveString {

    private final String str;
    
    public CaseInsensitiveString(String str) {
        this.str = Objects.requireNonNull(str);
    }
    
    @Override
    public boolean equals(Object o) {
        if (o instanceof CaseInsensitiveString) {
            return str.equalsIgnoreCase(((CaseInsensitiveString) o).str);
        }
        if (o instanceof String) { // 한 방향으로만 작동한다.
            return str.equalsIgnoreCase((String) o);
        }
        return false;
    }
}
```

CaseInsensitiveString의 equals는 CaseInsensitiveString, String 타입에 대해 대소문자 구별을 하지않고 검증한다.

하지만 String의 equals는 대소문자 구별을 하고 검증한다.

```java
void symmetryTest() {
    CaseInsensitiveString caseInsensitiveString = new CaseInsensitiveString("Test");
    String test = "test";
    System.out.println(caseInsensitiveString.equals(test)); // true
    System.out.println(test.equals(caseInsensitiveString)); // false
}
```

## 추이성

> x.equals(y)와 y.equals(z)가 true이면, x.equals(z)도 true이다.

```java
public class Point {

    private final int x;
    private final int y;
    
    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }
    
    @Override
    public boolean equals(Object o) {
        if (!(o instanceof Point)) {
            return false;
        }
        Point p = (Point) o;
        return this.x == p.x && this.y == p.y;
    }
}
```

```java
public class ColorPoint extends Point {

    private final Color color;
    
    public ColorPoint(int x, int y, Color color) {
		    super(x, y);
		    this.color = color;
		}

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof Point)) {
            return false;
        }

        // o가 일반 Point이면 색상을 무시햐고 x,y 정보만 비교한다.
        if (!(o instanceof ColorPoint)) {
            return o.equals(this);
        }

        // o가 ColorPoint이면 색상까지 비교한다.
        return super.equals(o) && this.color == ((ColorPoint) o).color;
    }
}
```

Point의 equals는 x, y값만을 검증한다.

ColorPoint의 equlas는 Point라면 x, y값만을 검증하고, ColorPoint라면 추가로 color까지 검증한다.

```java
void transitivityTest() {
    ColorPoint a = new ColorPoint(2, 3, Color.RED);
    Point b = new Point(2, 3);
    ColorPoint c = new ColorPoint(2, 3, Color.BLUE);

    System.out.println(a.equals(b)); // true
    System.out.println(b.equals(c)); // true
    System.out.println(a.equals(c)); // false
}
```

위 예제에서 추이성은 명백하게 위배된다.

### 무한 재귀의 위험성

Point를 상속받은 SmellPoint 클래스

```java
public class SmellPoint extends Point {

    private final Smell smell;
    
    public SmellPoint(int x, int y, Smell smell) {
		    super(x, y);
		    this.smell = smell;
		}

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof Point)) {
            return false;
        }

        // o가 일반 Point이면 냄새를 무시햐고 x,y 정보만 비교한다.
        if (!(o instanceof SmellPoint)) {
            return o.equals(this);
        }

        // o가 SmellPoint이면 냄새까지 비교한다.
        return super.equals(o) && this.smell == ((SmellPoint) o).smell;
    }
}
```

```java
void infinityTest() {
    Point cp = new ColorPoint(2, 3, Color.RED);
    Point sp = new SmellPoint(2, 3, Smell.SWEET);

    System.out.println(cp.equals(sp));
}
```

위와 같은 예제는 ColorPoint와 SmellPoint의 equals 메서드가 무한 재귀 형태로 실행되기 때문에 StackOverflowError가 발생된다.

### 리스코프 치환 원칙

구체 클래스를 확장해 새로운 값을 추가하면서 equals 규약을 만족시킬 방법은 존재하지 않는다.

추이성을 지키기 위해 instanceof이 아닌 getClass를 통해 규약도 지키고 값도 추가하며 구체 클래스를 상속할 수 있을까?

```java
@Override
public boolean equals(Object o) {
    // getClass
    if (o == null || o.getClass() != this.getClass()) {
        return false;
    }
    Point p = (Point) o;
    return this.x == p.x && this.y = p.y;
}
```

위의 예제는 같은 구현 클래스의 객체와 비교할 때만 수행되지만 실제로 활용할 수는 없다.

Point의 하위 클래스는 정의상 여전히 Point이므로 어디서든 Point로 활용될 수 있어야 한다. (리스코프 치환 원칙 위배)

Point를 상속받은 여러 구체 클래스들은 어디서든 Point로 활용될 수 있어야 하지만 그렇지 못하다.

### 상속대신 컴포지션을 사용해라

Point를 상속받지 않고 private 필드로 두고, Point를 반환하는 뷰 메서드를 public으로 추가한다.

```java
public class ColorPoint {

    private Point point;
    private Color color;

    public ColorPoint(int x, int y, Color color) {
        this.point = new Point(x, y);
        this.color = Objects.requireNonNull(color);
    }

    public Point asPoint() {
        return this.point;
    }

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof ColorPoint)) {
            return false;
        }
        ColorPoint cp = (ColorPoint) o;
        return this.point.equals(cp) && this.color.equals(cp.color);
    }
}
```

### 추상 클래스

추상 클래스의 하위 클래스에서는 equals 규약을 지키면서 값을 추가할 수 있다.

상위 클래스를 직접 인스턴스로 만드는게 불가능하므로 지금까지 이야기한 문제들은 발생하지 않는다.

## 일관성

> x.equals(y)를 반복해서 호출하면 항상 true를 반환하거나 항상 false를 반환한다.
>

```java
@Test
void consistencyTest() throws MalformedURLException {
    URL url1 = new URL("www.xxx.com");
    URL url2 = new URL("www.xxx.com");

    System.out.println(url1.equals(url2)); // 항상 같지 않다.
}
```

java.net.URL 클래스는 URL과 매핑된 host의 IP주소를 이용해 비교한다.

하지만 같은 도메인 주소라도 나오는 IP정보가 달라질 수 있기 때문에 반복적으로 호출할 경우 결과가 달라질 수 있다.

이러한 문제를 피하려면 equals는 항상 메모리에 존재하는 객체만을 사용한 결정적 계산을 수행해야 한다.

## null-아님

> x.equals(null)은 false이다.
>

실수로 NullPointerException을 던지는 코드는 흔하다.

일반 규약은 이런 경우도 허용하지 않기 때문에 null인지 확인하고 false를 반환해야 한다.

```java
@Override 
public boolean equals(Object o) {
    // 흔히 인텔리제이를 통해서 생성하는 equals는 다음과 같다.
    if (o == null || getClass() != o.getClass()) {
        return false;
    }
    
    // 책에서 추천하는 내용은 null 검사를 할 필요 없이 instanceof를 이용하라는 것이다.
    // instanceof는 두번째 피연산자(Point)와 무관하게 첫번째 피연산자(o)가 null이면 false를 반환하기 때문이다. 
    if (!(o instanceof Point)) {
        return false;
    }
}
```

## equals의 좋은 재정의 방법

```java
@Override
public boolean equals(final Object o) {
    // 1. == 연산자를 사용해 입력이 자기 자신의 참조인지 확인한다.
    if (this == o) {
        return true;
    }

    // 2. instanceof 연산자로 입력이 올바른 타입인지 확인한다.
    if (!(o instanceof Point)) {
        return false;
    }

    // 3. 입력을 올바른 타입으로 형변환 한다.
    final Piece piece = (Piece) o;

    // 4. 입력 개체와 자기 자신의 대응되는 핵심 필드들이 모두 일치하는지 하나씩 검사한다.
    return piece.x == x && piece.y == y;
}
```

```java
// float와 double을 제외한 기본 타입 필드는 ==를 사용한다.
// float과 double은 compare을 사용해 비교한다. (특수한 부동소수 값을 다뤄야 하기 때문)
return this.x == p.x && this.y == p.y;

// 필드가 참조 타입이라면 equals를 사용한다.
return str1.equals(str2);

// null 값을 정상 값으로 취급한다면 Objects.equals로 NullPointException을 예방하자.
return Objects.equals(Object, Object);
```

## 결론

꼭 필요한 경우가 아니면 equals를 재정의하지 말자.

Object의 equals가 클라이언트가 원하는 비교를 정확히 수행해준다.

재정의해야 할 때는 클래스의 핵심 필드를 빠짐없이 다섯가지 규약을 확실히 지켜 비교해야 한다.

어떤 필드를 먼저 비교하느냐가 equals의 성능을 좌우하기도 한다. 최상의 성능을 바란다면 다를 가능성이 더 크거나 비교하는 비용이 더 싼 필드를 먼저 비교하자.

객체의 논리적 상태와 관련 없는 필드는 비교하면 안된다.