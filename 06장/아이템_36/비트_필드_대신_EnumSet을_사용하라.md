# item 36. 비트 필드 대신 EnumSet을 사용하라

열거 값들이 집합으로 사용되는 경우 비트 마스킹을 활용한 정수 열거 패턴을 사용하곤 했다.

```java
public class Text {
    public static final int BOLD = 1 << 0;
    public static final int ITALIC = 1 << 1;
    public static final int UNDERLINE = 1 << 2;
    public static final int STRIKETHROUGH = 1 << 3;

    public void applyStyles(int styles) {
        if ((styles & BOLD) != 0) {
            System.out.println("Apply bold style");
        }
        if ((styles & ITALIC) != 0) {
            System.out.println("Apply italic style");
        }
        if ((styles & UNDERLINE) != 0) {
            System.out.println("Apply underline style");
        }
        if ((styles & STRIKETHROUGH) != 0) {
            System.out.println("Apply strikethrough style");
        }
    }
}
```

굵은체 `BOLD`는 첫번째 비트, 기울임체 `ITALIC`는 두번째 비트, 밑줄 `UNDERLINE`은 세번째 비트, 취소선 `STRIKETHROUGH`는 네번째 비트로 구분하는 것이다. styles는 이들 비트를 조합한 int 값이 매개변수로 들어간다.

```java
text.applyStyles(BOLD | UNDERLINE); // BOLD | UNDERLINE은 3
text.applyStyles(3);
```

위와 같이 비트의 OR연산을 통해 여러 상수를 하나의 집합으로 모을 수 있다. 이렇게 만들어진 집합을 비트 필드라고 한다.

단, 비트 필드는 정수 열거 상수의 단점을 그대로 지닌다. 비트 필드 값이 그대로 출력이 되면 이를 해석하기 너무 어렵다.

위 예시에서도 `BOLD`와 `UNDERLINE`이 적용된 값이 3인데 3만 보고서는 이를 파악하기 매우 힘들다. 비트 필드 하나에 녹아있는 모든 원소 순회도 까다롭다. 마지막으로 필요한 최대 비트를 API 작성시 예측하여 `int`나 `long` 같은 적절한 타입을 선택해야한다.

### 비트 필드의 대안으로 나온 EnumSet
이에 대한 완벽한 대안이 바로 `EnumSet`이다. enum 상수 값으로 구성된 집합을 효과적으로 표현하며 `Set` 인터페이스를 완벽히 구현할 수 있다.

 `EnumSet`의 내부는 사실 비트 백터로 구성된다. 원소가 64개 이하라면 대부분의 경우 `long` 변수 하나로 표현하여 비트 필드와 비슷한 성능을 보여준다.

 `EnumSet`은 비트 연산을 사용하기 때문에 HashSet과 같은 일반적인 집합 구현체에 비해 메모리 사용량이 매우 적고, 집합 연산은 CPU 수준에서 비트 연산으로 매우 빠르게 수행된다.

 각 enum 상수는 정의된 순서에 따라 0부터 시작하는 인덱스를 가지며, 해당 인덱스의 위치의 비트가 1이면 그 상수가 포함된 것을 간주된다.

> 참고로 원소가 64개 이하라면 RegularEnumSet, 65개 이상이면 JumboEnumSet을 사용

```java
public static <E extends Enum<E>> EnumSet<E> noneOf(Class<E> elementType) {
    Enum<?>[] universe = getUniverse(elementType);
    if (universe == null)
        throw new ClassCastException(elementType + " not an enum");

    if (universe.length <= 64)
        return new RegularEnumSet<>(elementType, universe);
    else
        return new JumboEnumSet<>(elementType, universe);
}
```

RegularEnumSet의 내부를 보면 다음과 같다.

```java
class RegularEnumSet<E extends Enum<E>> extends EnumSet<E> {
    private static final long serialVersionUID = 3411599620347842686L;

    private long elements = 0L;

    void addAll() {
        if (universe.length != 0)
            elements = -1L >>> -universe.length;
    }

    void complement() {
        if (universe.length != 0) {
            elements = ~elements;
            elements &= -1L >>> -universe.length;  // Mask unused bits
        }
    }

    // ...
}
```
원소가 64개 이하에서 사용하는 `RegularEnumSet`은 64비트로 표현하는 `long` 타입의 `elements`를 통해 Set을 구현한다.

Enum과 EnumSet을 활용해서 위 정수 열거 패턴의 Text를 다음과 같이 변경할 수 있다.

```java
public class Text {
    public enum Style {
        BOLD, ITALIC, UNDERLINE, STRIKETHOUGH
    }

    private Set<Style> styles = new EnumSet.noneOf(Style.class);

    public void applyStyles(Set<Style> styles) {
        if (styles.contains(TextStyle.BOLD)) {
            System.out.println("Apply bold style");
        }
        if (styles.contains(TextStyle.ITALIC)) {
            System.out.println("Apply italic style");
        }
        if (styles.contains(TextStyle.UNDERLINE)) {
            System.out.println("Apply underline style");
        }
        if (styles.contains(TextStyle.STRIKETHROUGH)) {
            System.out.println("Apply strikethrough style");
        }
    }
}
```

다음과 같이 Set<Style>을 Text 객체의 styles 필드에 직접 할당하면, 정수 비트 패턴 방식의 applyStyles 메소드와 유사하게 스타일을 적용하고 관리할 수 있습니다.

> 대부분의 enum의 원소 갯수가 64개를 넘어갈일이 없다고 개인적으로 생각하므로 `RegularEnumSet`만 어느정도 알면 되지 않을까 생각합니다.