### ordinal 메서드 대신 인스턴스 필드를 사용하라
열거 타입은 해당 상수가 그 열거 타입에서 몇 번째 위치인지를 반환하는 ordinal이라는 메서드를 반환한다. 그래서 이 ordinal을 쓰고 싶을 수 있다.

**나쁜 예시**
``` java
public enum Ensemble {
    SOLO, DUET, TRIO, QUARTET, QUINTET,
    SEXTET, SEPTET, OCTET, NONET, DECTET

    public int numberOfMusicians(){ return ordinal() + 1 };
}
```

유지보수하기 힘든 코드다. 상수 선언 순서가 바뀌면 오작동한다. 또한 이미 사용중인 정수와 값이 같은 상수는 추가할 방법이 없다.
또 값을 비워둘 수도 없다.

**열거 타입 상수에 연결된 값은 ordinal 메서드로 얻지 말고, 인스턴스 필드에 저장하자.**

애초에 `Enum`의 API 문서에도 이렇게 쓰여있다. "대부분 프로그래머는 이 메서드를 쓸 일이 없다. 이 메서드는 EnumSet과 EnumMap 같이 열거 타입 기반의 범용 자료구조에 쓸 목적으로 설계되었다."

![](https://images.velog.io/images/injoon2019/post/76f1241c-2521-47a2-be88-9c97b82a3f2c/image.png)

**올바른 예시**
``` java
public enum Ensemble {
    SOLO(1), DUET(2), TRIO(3), QUARTET(4), QUINTET(5),
    SEXTET(6), SEPTET(7), OCTET(8), DOUBLE_QUARTET(8),
    NONET(9), DECTET(10), TRIPLE_QUARTET(12);

    private final int numberOfMusicians;
    Ensemble(int size){ this.numberOfMusicians = size;}
    public int numboerOfMusicians() { return numberOfMusicians; }
}
```
위의 코드는 ordinal 메서드를 사용하지 않고, 인스턴스 필드에 인원 수를 저장하여 중복된 값(8)를 저장할 수 있게 되었다.