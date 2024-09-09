## ordinal대신 EnumMap

```java
public class Plant {

    public enum Lifecycle { ANNUAL, PERENNIAL, BIENNIAL }

    final String name;
    final Lifecycle lifecycle;

    Plant(String name, Lifecycle lifecycle) {
        this.name = name;
        this.lifecycle = lifecycle;
    }

    @Override
    public String toString() {
        return name;
    }
}
```

LifeCycle별로 묶어서 관리하고 싶을 때,

### ordinal()을 배열 인덱스로 사용

```java
public static void withOrdinal(Plant[] garden) {
    Set<Plant>[] plantsByLifeCycle =
            (Set<Plant>[]) new Set[Plant.Lifecycle.values().length];
    for (int i = 0; i < plantsByLifeCycle.length; i++)
        plantsByLifeCycle[i] = new HashSet<>();
    for (Plant p : garden) {
        plantsByLifeCycle[p.lifecycle.ordinal()].add(p);
    }
    for (int i = 0; i < plantsByLifeCycle.length; i++) {
        System.out.printf("%s: %s%n",
                Plant.Lifecycle.values()[i], plantsByLifeCycle[i]);
    }
}
```

배열은 제네릭과 호환되지 않으니 비검사 형변환을 수행해야 하고 깔끔히 컴파일되지 않을 것이다.

배열은 인덱스의 의미를 모르니, 출력 결과에 직접 레이블을 달아야 한다.

정확한 정수값을 사용한다는 것을 클라이언트에서 직접 보증해야 한다.

### EnumMap을 사용

```java
public static void withEnumMap(Plant[] garden) {
    Map<Plant.Lifecycle, Set<Plant>> plantByLifeCycle =
            new EnumMap<>(Plant.Lifecycle.class);
    for (Plant.Lifecycle lc: Plant.Lifecycle.values())
        plantByLifeCycle.put(lc, new HashSet<>());
    for (Plant p: garden)
        plantByLifeCycle.get(p.lifecycle).add(p);
    System.out.println(plantByLifeCycle);
}
```

안전하지 않은 형변환은 쓰지 않고, 맵의 키인 열거 타입이 그 자체로 출력용 문자열을 제공하니 출력 결과에 레이블을 달 필요도 없다.

EnumMap의 성능이 HashMap보다 빠르고 ordinal을 쓴 배열에 비견되는 이유는 내부에서 배열을 사용하기 때문이다.

내부 구현 방식을 안으로 숨겨서 Map의 타입 안전성과 배열의 성능을 얻어낸 것이다. (ordinal을 내부 구현에서 사용)

### 스트림을 사용

```java
public static void withStream1(Plant[] garden) {
    System.out.println(Arrays.stream(garden)
            .collect(groupingBy(p -> p.lifecycle)));
}
```

EnumMap이 아닌 고유한 맴 구현체를 사용하기 때문에 EnumMap을 써서 얻은 공간과 성능 이점이 사라진다.

```java
public static void withStream2(Plant[] garden) {
    System.out.println(Arrays.stream(garden)
            .collect(groupingBy(p -> p.lifecycle,
                    () -> new EnumMap<>(Plant.Lifecycle.class), toSet())));
}
```

`p -> p.lifecycle`: `Plant` 객체의 `lifecycle` 필드를 기준으로 그룹화한다. 즉, `Plant` 객체의 `lifecycle` 열거형 값(`Lifecycle.ANNUAL`, `Lifecycle.PERENNIAL`, `Lifecycle.BIENNIAL` 등)에 따라 식물들을 그룹으로 나누게 된다.

`() -> new EnumMap<>(Plant.Lifecycle.class)`: 이 부분은 그룹화된 결과를 저장할 맵을 생성한다. 여기서는 `EnumMap`을 사용하여 `Plant.Lifecycle` 열거형을 키로 하는 맵을 만든다. `EnumMap`은 열거형 타입을 키로 사용하는 특별한 맵으로, 성능이 뛰어나며 메모리 사용이 효율적이다.

`toSet()`: 그룹화된 각 생애주기(`lifecycle`)에 속하는 `Plant` 객체들을 **Set**으로 수집한다. 즉, 각 `lifecycle` 값에 대해 그에 해당하는 `Plant` 객체들을 중복 없이 수집하는 것입니다.

### EnumMap을 직접 사용할 때와 스트림을 통해 사용할 때의 차이

EnumMap 버전은 언제나 식물의 생애주기당 하나씩의 중첩 맵을 만든다.

스트림 버전에서는 해당 생애주기에 속하는 식물이 있을 때만 만든다.

## 중첩 열거 타입과 EnumMap

```java
public enum Phase {

    SOLID, LIQUID, GAS;

    public enum Transition {
        MELT(SOLID, LIQUID), FREEZE(LIQUID, SOLID),
        BOIL(LIQUID, GAS), CONDENSE(GAS, LIQUID),
        SUBLIME(SOLID, GAS), DEPOSIT(GAS, SOLID);

        private final Phase from;
        private final Phase to;

        Transition(Phase from, Phase to) {
            this.from = from;
            this.to = to;
        }

        // Map 초기화
        private static final Map<Phase, Map<Phase, Transition>> m =
                Stream.of(values()).collect(groupingBy(t -> t.from,
                        () -> new EnumMap<>(Phase.class),
                        toMap(t -> t.to, t -> t,
                                (x, y) -> y, () -> new EnumMap<>(Phase.class))));

        public static Transition from(Phase from, Phase to) {
            return m.get(from).get(to);
        }
    }
}
```

`toMap(t -> t.to, t -> t, (x, y) -> y, () -> new EnumMap<>(Phase.class))`

`toMap()`은 내부적으로 그룹화된 각 `Transition` 객체를 다시 `Map`으로 변환하는 역할을 한다. 즉, 하나의 `Phase`에서 다른 `Phase`로 전이될 때의 정보가 담긴 맵을 생성한다.

- **`t -> t.to`**: `Transition` 객체의 `to` 필드를 키로 사용한다. 즉, 전이에서의 도착지 `Phase`를 키로 설정한다.
- **`t -> t`**: `Transition` 객체 자체를 값으로 사용한다. 즉, 전이 정보를 그대로 값으로 설정한다.
- **`(x, y) -> y`**: 만약 동일한 키(`to`)에 대해 두 개의 전이 정보가 있을 경우, 충돌이 발생하면 둘 중 하나를 선택하는 로직이다. 여기서는 나중에 들어온 값을 선택한다.
- **`() -> new EnumMap<>(Phase.class)`**: `toMap()`이 생성할 맵의 구현체를 명시한다. 여기서는 `EnumMap`을 사용하여 `Phase` 열거형을 키로 가지는 맵을 생성한다.

새로운 상태가 들어온다 하더라도, 상태 목록과 전이 목록에 값을 추가해주기만 하면 끝이다.

## 결론

배열의 인덱스를 얻기 위해 ordinal을 쓰는 것은 일반적으로 좋지 않으니, EnumMap을 사용하라.

다차원 관계는 `EnumMap<…, EnumMap<…>>`으로 표현하라.