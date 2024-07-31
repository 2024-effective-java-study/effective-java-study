> hashCode: 객체의 주소값을 변환하여 생성한 객체의 고유한 정수값

```java

public final class PhoneNumber {

    private final short areaCode, prefix, lineNum;

    public PhoneNumber(int areaCode, int prefix, int lineNum) {
        this.areaCode = ranceCheck(areaCode, 999, "지역코드");
        this.prefix = ranceCheck(prefix, 999, "프리픽스");
        this.lineNum = ranceCheck(lineNum, 999, "가입자 번호");
    }

    private static rangeCheck(int val, int max, String arg) {
        if (val < 0 || val > max) {
            throw new IllegalArgumentException(arg + ": " + val);
        }
        return (short) val;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) {
            return true;
        }
        if (!(o instanceof PhoneNumber)) {
            return false;
        }

        PhoneNumber pn = (PhoneNumber) o;

        return pn.lineNum == lineNum && pn.prefix == prefix && pn.areaCode == areaCode;
    }
}
```

```java
public class Main {

    public static void main(String[] args) {

        PhoneNumber phone1 = new PhoneNumber(123, 456, 7890);
        PhoneNumber phone2 = new PhoneNumber(123, 456, 7890);

        System.out.println(phone1.hashCode());
        System.out.println(phone2.hashCode());
        System.out.println(phone1.eqauls(phone2));
    }
}

//    1456208737 
//    288665596
//    true
```

`위와 같이 equals를 통해 물리적으로 다른 두 객체를 논리적으로 같다고 할 수 있다 .하지만 Object의 기본 hashcode 메서드는 이 둘을 전혀 다르다고 판단하여, 규약과 달리(무작위 처럼 보이는) 서로 다른 값을 반환한다.`

```java
public class Main {

    public static void main(String[] args) {
        Map<PhoneNumber, Integer> 전화_번호부 = new HashMap<>();
        전화_번호부.put(phone1, 16);
        System.out.println(전화_번호부.get(phone1));
        System.out.println(전화_번호부.get(phone2));
    }
}

// 16
// null
```

hashMap, hashSet 등 hash 를 이용하는 컬렉션의 원소로 사용할 때 문제를 일으킨다.

Object 명세에서 hashcode 과 관련된 언급

- equals 비교에 사용되는 정보가 변경되지 않았다면 hashcode 의 값은 항상 같은 값을 반환한다.
- equals 가 두 객체를 같다고 판단했으면 두 객체의 hashcode 는 같은 값을 반환한다.
- equals 가 두 객체를 다르다고 판단했더라도 두 객체의 hashcode 가 다를 필요는 없다.

---

`그냥 아무 값이나 리턴하면 안돼?`

```
@Override
public int hashCode() {
    return 32;
    }
```

1. 해당 타입을 가지는 모든 인스턴스가 같은 인스턴스로 인식된다.
2. 수행 시간이 O(1) 에서 O(N) 으로 느려진다.

```
올바른 해시코드 작성 예시
@Override
    public int hashCode() {
        int result = Short.hashcode(areaCode);
        result = 31 * result + Short.hashcode(prefix);
        result = 31 * result + Short.hashcode(lineNum);
        return result;
    }
```

```
intellij default
@Override
    public int hashCode() {
        return category != null ? category.hashCode() : 0;
    }
    
java 7+
@Override
    public int hashCode() {
        return Objects.hash(category);
    }
```

---

해시코드 작성 요령

1. int result = c
    - c 는 아래의 2단계때!
2. c 의 계산 방식
    1. 기본 타입 필드(String, Integer 등)라면 Type.hashCode(필드)
    2. 참조 타입 필드(기본 타입 이외의 타입)라면 재귀적으로 호출, 계산이 복잡해진다면 표준형을 만들어 hashCode 호출
        - ex) getLottoMachine().getLottoTicket().getLotto().getLottoNumber()
    3. 배열 필드라면 핵심 원소에 대해 위의 1, 2 방식으로 해시코드 계산하여 비교, 모든 원소가 핵심 원소라면 Arrays.hashCode 이용
3. result = 31 * result + c
    - ??

```java

public class Address {

    private String 도시;
    private String 구;
    private String 동;

    @Override
    public int hashCode() {
        int result = String.hashCode(도시);
        result = 31 * result + String.hashCode(구);
        result = 31 * result + String.hashCode(동);
        return result;
    }
}
```

31인 이유!
* 31이 홀수이면서 소수이기 떄문이라고 한다.(마약 이 숫자가 짝수이고 오버플로가 발생한다면 정보를 잃게 된다. 2를 곱하는 것은 시프트 연산과 같은 결과를 내기 떄문이다.)
* 31 * i = (i << 5) - i

```
java 7+
@Override
public int hashCode() {
    return Objects.hash(도시, 구, 동);
```

---

클래스가 불변이면서 해시코드를 계산하는 비용이 크고, 주로 해시의 키로 사용되는 경우

```

private int hashCode;

@Override public int hashCode() {
    int result = hashCode;
    if (result == 0) {
    // 해시코드 계산
    hashCode = result;
    }
    return result;
}
```

```
String a = "Z@S.ME";
String b = "Z@RN.E";
String c = "ZZZ";
System.out.println(a.hashCode());
System.out.println(b.hashCode());
System.out.println(c.hashCode());

출력 결과
-1656719047
-1656719047
89370

Map<String, Integer> map = new HashMap<>();
map.put(a, 1);
System.out.println(map.get(a));
System.out.println(map.get(b));

출력 결과
1
null
```

## 주의할 점

- 성능을 위해 해시코드 계산할 때 핵심 필드를 생략해서는 안된다. (해시 품질이 나빠져 해시테이블의 성능을 심각하게 떨어뜨릴 수 있다.)
- hashcode 값의 생성 규칙을 API 사용자에게 자세히 공표해서는 안된다.