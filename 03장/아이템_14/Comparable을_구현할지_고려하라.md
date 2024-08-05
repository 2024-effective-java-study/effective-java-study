## Comparable이란?

단순 동치성 비교뿐 아니라 순서까지 비교할 수 있는 제네릭 메서드 compareTo를 가진 인터페이스

Comparable을 구현했다는 것은 그 클래스의 인스턴스들에는 자연적인 순서가 있음을 뜻한다.

Comparable을 구현한 객체의 배열과 컬렉션은 Arrays.sort(), Collections.sort()를 사용해 정렬할 수 있다.

자바 플랫폼 라이브러리의 모든 값 클래스와 열거 타입은 Comparable을 구현해 놓았다.

알파벳, 숫자, 연대 같이 순서가 명확한 값 클래스를 작성한다면 반드시 Comparable 인터페이스를 구현하자.

## compareTo의 일반 규약

> 객체와 주어진 객체의 순서를 비교한다. 객체가 주어진 객체보다 작으면 음의 정수를, 같으면 0을, 크면 양의 정수를 반환한다. 객체와 비교할 수 없는 타입의 객체가 주어지면 ClassCastException을 던진다.
다음 설명에서 sgn(표현식) 표기는 수학에서 말하는 부호 함수(signum function)를 뜻하며, 표현식의 값이 음수, 0, 양수일 때 -1, 0, 1을 반환하도록 정의했다.
<br><br>
Comparable을 구현한 클래스는 모든 x, y에 대해 sgn(x.compareTo(y)) == -sgn(y.compareTo(x))여야 한다. (예외도 동일하게 한쪽이 예외를 던진다면 반대쪽도 예외를 던져야 한다.)
<br><br>
Comparable을 구현한 클래스는 추이성을 보장해야 한다.
(x.compareTo(y) > 0 && y.compareTo(z) > 0)이면 x.compareTo(z) > 0이다.
<br><br>
Comparable을 구현한 클래스는 모든 z에 대해 x.compareTo(y) == 0이면 sgn(x.compareTo(z)) == sgn(y.compareTo(z))다.
<br><br>
이번 권고가 필수는 아니지만 꼭 지키는게 좋다.
(x.compareTo(y) == 0 == (x.equals(y))여야 한다. Comparable을 구현하고 이 권고를 지키지 않는 모든 클래스는 그 사실을 명시해야 한다. (”주의: 이 클래스의 순서는 equals 메서드와 일관되지 않다.”)

## equals와 compareTo

모든 객체에 대해 전역 동치관계를 부여하는 equals 메서드와 달리, compareTo는 타입이 다른 객체를 신경 쓰지 않아도 된다.

hashCode 규약을 지키지 못하면 해시를 사용하는 클래스와 어울리지 못하듯, compareTo 규약을 지키지 못하면 비교를 활용하는 클래스(TreeSet, TreeMap, Collections, Arrays)와 어울리지 못한다.

compareTo 메서드의 동치성 검사도 equals 규약과 비슷하기에 주의사항도 똑같다. 확장한 구체 클래스에서 새로운 값 컴포넌트를 추가했다면 규약을 지킬 방법이 없다. 그러므로 컴포지션을 사용해야 한다.

## compareTo의 사용법

compareTo 메서드는 각 필드가 동치인지를 비교하는게 아니라 순서를 비교한다.

Comparable을 구현하지 않은 필드나 표준이 아닌 순서로 비교해야 한다면 비교자(Comparator)를 대신 사용한다.

compareTo 메서드 내부에서 정수 기본 타입 필드를 비교할 때 (<, >) 연산자가 아닌, 박싱된 기본 타입 클래스들의 정적 메서드인 compare를 이용하면 된다.

클래스의 핵심 필드가 여러 개라면, 가장 핵심적인 필드부터 비교해나가야 한다. 비교 결과가 0이 아니라면, 즉 순서가 결정되면 그곳에서 return 한다. 똑같지 않은 필드를 찾을 때까지 그 다음으로 중요한 필드를 비교해나간다.

```java
public int compareTo(PhoneNumber pn) {
    int result = Short.compare(areaCode, pn.areaCode);
    if (result == 0) {
        result = Short.compare(prefix, pn.prefix);
        if (result == 0) {
            result = Short.compare(lineNum, pn.lineNum);
        }
    }
    return result;
}
```

### Comparator과의 활용

Comparator 인터페이스의 정적 비교자 생성 메서드들을 compareTo 메서드를 구현하는데 활용할 수 있다.

```java
private static final Comparator<PhoneNumber> COMPARATOR = 
        comparingInt((PhoneNumber pn) -> pn.areaCode)
                .thenComparingInt(pn -> pn.prefix)
                .thenComparingInt(pn -> pn.lineNum);

public int compareTo(PhoneNumber pn) {
    return COMPARATOR.compare(this, pn);
}
```

comparingInt는 람다를 인수로 받으며, 이 람다는 PhoneNumber에서 추출한 지역 코드를 기준으로 전화번호의 순서를 정하는 Comparator<PhoneNumber>를 반환한다.

thenComparingInt는 comparingInt와는 다르게 첫번째 비교자 이후에 비교를 수행할 때 사용된다. 추가로 원하는 만큼 연달아 호출할 수 있다.

## 결론

순서를 고려해야 하는 값 클래스를 작성한다면 꼭 Comparable 인터페이스를 구현하여, 그 인스턴스들을 쉽게 정렬하고, 검색하고, 비교 기능을 제공하는 컬렉션과 어우러지도록 해야 한다.

compareTo 메서드에서 필드의 값을 비교할 때 <와 > 연산자는 쓰지 말아야 한다. 그 대신 박싱된 기본 타입 클래스가 제공하는 정적 compare 메서드나 Comparator 인터페이스가 제공하는 비교자 생성 메서드를 사용하자.