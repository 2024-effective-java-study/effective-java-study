## **정적 메서드와 정적 필드만을 담은 클래스를 만들고 싶을 때**

### 1. 기본 타입 값이나 배열 관련 메서드를 모은 클래스

java.util.Arrays

```java
public class Arrays {

    private Arrays() {}
    
    // ...
    
    public static boolean isArray(Object o) { ...}

    public static Object[] asObjectArray(Object array) { ...}

    public static List<Object> asList(Object array) { ...}

    public static <T> boolean isNullOrEmpty(T[] array) { ...}

    // ...
}
```

java.lang.Math

```java
public final class Math {

    private Math() {}

    public static final double E = 2.7182818284590452354;

    public static final double PI = 3.14159265358979323846;
    
    // ...
    
    @IntrinsicCandidate
    public static int abs(int a) {
        return (a < 0) ? -a : a;
    }
    
    // ...
}
```

- 내부 변수와 메서드가 모두 static으로 선언되어 있다.
- 생성자가 private으로 선언되어 있다.

### 2. 특정 인터페이스를 구현하는 객체를 생성해주는 정적 메서드(혹은 팩토리)를 모은 클래스

java.util.Collections

```java
public class Collections {

    private Collections() {}
    
    private static final int BINARYSEARCH_THRESHOLD   = 5000;
    
    // ...
    
    public static <T extends Comparable<? super T>> void sort(List<T> list) {
        list.sort(null);
    }
    
    // ...
}
```

- 객체를 구현한다는 목적만 다를 뿐, static으로 선언된 메서드와 private으로 선언된 생성자의 형태는 1번과 같다.

### 3. final 클래스와 관련된 메서드들을 모아놓을 때

final 클래스는 보안 상의 이유로 상속을 금지

(final 클래스의 하위 클래스를 만드는 것은 시스템의 파괴를 야기할 수 있음)

```java
public final class String implements java.io.Serializable, Comparable<String>, CharSequence, Constable, ConstantDesc { ... }
```

final 클래스와 관련된 메서드들을 모아놓은 Util 클래스

```java
@API(status = INTERNAL, since = "1.0")
public final class StringUtils {
    private static final Pattern ISO_CONTROL_PATTERN = compileIsoControlPattern();
    private static final Pattern WHITESPACE_PATTERN = Pattern.compile("\\s");

    static Pattern compileIsoControlPattern() { ... }

    private StringUtils() {
        /* no-op */
    }

    public static boolean isBlank(String str) { ... }

    public static boolean isNotBlank(String str) { ... }

    public static boolean containsWhitespace(String str) { ... }

    // ...
}
```

- 모든 변수와 메서드가 static으로 선언되어 있다.

### Util 클래스

- static 멤버를 사용하는 유틸리티 클래스는 인스턴스로 만들어 사용하려고 설계한 것이 아니다.
- 생성자를 private으로 선언하는 이유
  → 생성자를 선언하지 않아도 컴파일러가 자동으로 기본 생성자(매개변수를 받지 않는)를 public으로 만든다.
- abstract 클래스는 인스턴스화를 막지 못하는 이유
  → 클래스를 상속받으면 인스턴스화를 할 수 있다.
  → 오히려 상속을 해야한다고 생각할 수 있다.
- 결론
  → 인스턴스화를 막으려면 private 생성자 선언
  → private 생성자를 사용하면, 상위 클래스의 생성자로 접근이 불가능하므로 상속이 불가능
  → private 생성자 내부에서 Exception을 일으켜 내부에서도 선언하지 못하게 할 수 있다.