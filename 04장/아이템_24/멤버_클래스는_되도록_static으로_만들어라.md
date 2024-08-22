## ❓ 중첩 클래스란?

- 다른 클래스 안에 정의된 클래스
- 자신을 감싼 바깥 클래스에서만 사용되어야 함  
  (그 외 쓰임새가 생기면 톱 레벨 클래스로 생성)

#### Inner class
inner class란 하나의 클래스 내부에 선언된 또 다른 클래스를 의미한다.<br>
보통 사용자 클래스 자료형이 필요하면, 메인 클래스 외부에 선언하거나, 따로 독립적인 클래스 파일을 만들어 불러와 사용해 왔다. 내부 클래스는 대신 클래스 내에 선언되어 사용되며, 내부에 정의된다는 점을 제외하고는 일반적인 클래스와 다르지 않다. 우리가 어느 클래스에 변수나 상수가 필요하다면 클래스 멤버로서 클래스 내에서 선언하여 사용해 왔듯이, 선언 주체를 변수에서 클래스로 바꾼다면 그것이 내부 클래스인 것이다.
<br>
이처럼 내부 클래스는 보통 두 클래스가 서로 긴밀한 관계가 있거나, 하나의 클래스또는 메소드에서만 사용되는 클래스일 때 이용되는 기법이라고 보면 된다.

* Inner class는 중첩(Nested) 클래스로 분류되기도 한다.

#### 내부 클래스의 장점

1. 클래스를 논리적으로 그룹화
* 클래스가 여러 클래스와 관계를 맺지 않고 하나의 특정 클래스와만 관계를 맺는다면, 외부에 클래스를 새로 작성하는 것이 아닌 내부 클래스로 작성할 수 있다.
  이런 경우 내부 클래스와 외부 클래스를 함께 관리하는 것이 가능해 유지보수 면에서나 코드 이해성 면에서 편리해진다.
* 내부 클래스로 인해 새로운 클래스를 생성하지 않아도 되므로 패키지를 간소화할 수 있다.

2. 더욱 타이트한 캡슐화의 적용
* 내부 클래스에 private 제어자를 적용해줌으로써, 캡슐화를 통해 클래스를 내부로 숨길 수 있다.
* 즉, 캡슐화를 통해 외부에서의 접근을 차단하면서도, 내부 클래스에서 외부 클래스의 멤버들을 제약 없이 쉽게 접근할 수 있어 구조적인 프로그래밍이 가능해 진다. 그리고 클래스 구조를 숨김으로써 코드의 복잡성도 줄일 수 있다.
```java
class Creature {
private int life = 50;

    // private class 로, 오로지 Creature 외부 클래스에서만 접근 가능한 내부 클래스로 설정
    private class Animal {
        private String name = "호랑이";

        int getOuter() {
            return life; // 외부 클래스의 private 멤버를 제약 없이 접근 가능
        }
    }

    public void method() {
        Animal animal = new Animal(); 

        // Getter 없이 내부 클래스의 private 멤버에 접근이 가능
        System.out.println(animal.name); // 호랑이

        // 내부 클래스에서 외부 클래스이 private 멤버를 출력
        System.out.println(animal.getOuter()); // 50
    }
}
```
3. 가독성이 좋고 유지 관리가 쉬운 코드
* 결국은 내부 클래스를 작성하는 경우 클래스를 따로 외부에 작성하는 경우보다, 물리적으로 논리적으로 외부 클래스에 더 가깝게 위치하게 된다. 따라서 시각적으로 읽기가 편해질 뿐 아니라 유지보수에 있어 이점을 가지게 된다.
* 한 클래스를 다른 클래스의 내부 클래스로 선언하면 두 클래스 멤버들 간에 서로 자유로이 접근할 수 있고, 그리고 외부에는 불필요한 클래스를 감춰서 클래스간의 연관 관계 따지는 것과 같은 코드의 복잡성을 줄일 수 있다는 장점이 있기 때문이다.
* 간단하게 말하자면 어차피 A 클래스안에서만 사용하기 위한 클래스이니 <span style="color:red">괜히 연관관계 생각없이 내부에 선언해 직관적으로 사용하자는 취지</span>인 것이다.

#### 중첩 클래스의 종류
- 정적 멤버 클래스
- 비정적 멤버 클래스
- 익명 클래스
- 지역 클래스

정적 멤버 클래스를 제외한 클래스들은 내부 클래스(inner class)라고 한다.

### 📕 정적 멤버 클래스

- static 키워드가 붙은 내부 클래스
- 단, 일반적인 static 필드 변수나 static 메서드와 똑같이 생각하면 안된다.
- static 클래스 내부에는 instance 멤버와 static 멤버 모두 선언 할 수 있다.
- 그러나 일반적인 static 메서드와 동일하게 외부 클래스의 인스턴스 멤버에는 접근이 불가하고, 정적(static) 멤버에만 접근할 수 있다. 
- 바깥 클래스의 private static 멤버 접근 가능, private 인스턴스 멤버의 경우 간접 접근 가능(생성후 접근 가능)
- private로 선언 시 바깥 클래스에서만 접근 가능


```java
public class Animal {
    private String name = "cat";

    // 열거 타입도 암시적 static
    public enum Kinds {
        MAMMALS, BIRDS, FISH, REPTILES, INSECT
    }

    private static class PrivateSample {
        private int temp;

        public void method() {
            Animal outerClass = new Animal();
            System.out.println("private" + outerClass.name); // 바깥 클래스인 Animal의 private 멤버 접근
        }
    }

    public static class PublicSample {
        private int temp;

        public void method() {
            Animal outerClass = new Animal();
            System.out.println("public" + outerClass.name); // 바깥 클래스인 Animal의 private 멤버 접근
        }
    }
}
```

#### 정적 클래스(static 클래스)에 대한 오해
가장 많은 사람들이 실수하는 오해가 static 클래스가 그 외의 static 필드 변수나 static 메소드와 같이, 'static 이니까 메모리에 하나만 올라가는 인스턴스'로 착각 한다는 점이다.
우선 결론부터 말하면 static 클래스는, 키워드가 static이 들어갔을 뿐이지, 우리가 알던 static 멤버와는 전혀 다른 녀석이다.

```java
public class Main {
// 스태틱 필드 변수
static Integer num = new Integer(0);

    // 내부 인스턴스 클래스
    class InnerClass{
    }

    // 내부 스태틱 클래스
    static class InnerStaticClass{
    }
    
    public static void main(String[] args) {

        // 스태틱 필드 변수는 유일해서 서로 같다
        Integer num1 = Main.num;
        Integer num2 = Main.num;
        System.out.println(num1 == num2); // true


        // 생성된 내부 클래스 인스턴스는 서로 다르다
        Main.InnerClass inner1 = new Main().new InnerClass();
        Main.InnerClass inner2 = new Main().new InnerClass();
        System.out.println(inner1 == inner2); // false


        // 생성된 내부 스태틱 클래스 인스턴스는 서로 다르다
        Main.InnerStaticClass static1 = new InnerStaticClass();
        Main.InnerStaticClass static2 = new InnerStaticClass();
        System.out.println(static1 == static2); // false
    }
}
```
복잡하게 생각할 필요없이 static 클래스는 static 이라고 해서 메모리에 한번만 로드되는 객체 개념이 아닌 것이다.
즉, 내부 인스턴스 클래스 처럼 외부 인스턴스를 먼저 선언하고 초기화해야 하는 선수 작업 필요 없이, 내부 클래스의 인스턴스를 바로 생성할 수 있다는 차이점이 존재 할 뿐이다.


#### 언제 사용할까?

- 👉 바깥 클래스가 표현하는 객체의 한 부분(구성요소)일 때 사용
- example: Map의 Entry
    - Map과 연관
    - Entry의 getKey(), getValue() 등의 메소드를 직접 사용 X

> Map 안의 Entry는 interface다.  
> Map을 구현하는 구현체인 HashMap에서 해당 interface를 implements하여 새로운 클래스를 정의한다.  
> 자세한 로직이 궁금하다면 `static class Node<K,V> implements Map.Entry<K,V>`를 찾아보아라.

### 📒 비정적 멤버 클래스

- static이 붙지 않은 멤버 클래스
- 비정적 멤버 클래스의 인스턴스 메서드에서 `클래스명.this`를 사용해 바깥 인스턴스의 메서드나 참조 가져올 수 있음
- 바깥 인스턴스 없이는 생성 불가  
  (👉 비정적 멤버 클래스의 인스턴스는 바깥 클래스의 인스턴스와 암묵적인 연결)

```java
public class TestClass {
    private String name = "yeonlog";

    public class PublicSample {
        public void printName() {
            // 바깥 클래스의 private 멤버 가져오기
            System.out.println(name);
        }

        public void callTestClassMethod() {
            // 바깥 클래스의 메소드 호출하기
            TestClass.this.testMethod();
        }
    }

    public PublicSample createPublicSample() {
        return new PublicSample();
    }

    public void testMethod() {
        System.out.println("hello world");
    }
}
```

- 바깥 클래스의 인스턴스 메서드에서 비정적 멤버 클래스의 생성자 호출
- `바깥 클래스 인스턴스.new 멤버클래스()` 호출
    - 위 두 방법으로 바깥 클래스 - 비정적 멤버 클래스의 관계 생성됨
    - 관계 정보는 비정적 멤버 클래스의 인스턴스 내에서 메모리 공간을 차지

```java
TestClass test = new TestClass();
PublicSample publicSample1 = test.createPublicSample(); // 바깥 클래스의 인스턴스 메서드에서 생성자 호출
PublicSample publicSample2 = test.new PublicSample(); // 바깥 인스턴스 클래스.new 멤버클래스()
```

#### 멤버 클래스에서 바깥 인스턴스에 접근할 일이 없다면 static을 붙이자.

- 바깥 인스턴스 - 멤버 클래스 관계를 위한 시간과 공간 소모
- Garbage Collection이 바깥 클래스의 인스턴스 수거 불가 -> 메모리 누수 발생
- 참조가 눈에 보이지 않아 개발이 어려움

#### 그럼 비정적 멤버 클래스는 언제 쓰는가?

👉 어댑터 정의 시 자주 쓰임.  
= 어떤 클래스의 인스턴스를 감싸서 다른 클래스의 인스턴스처럼 보이게 하는 뷰로 사용

example - Map 인터페이스의 구현체  
👉 `keySet()`, `entrySet()`, `values()`가 반환하는
자신의 [컬렉션 뷰](https://stackoverflow.com/questions/18902484/what-is-a-view-of-a-collection) 를 구현할 때 활용

```java
public class HashMap<K, V> extends AbstractMap<K, V>
        implements Map<K, V>, Cloneable, Serializable {

    final class EntrySet extends AbstractSet<Map.Entry<K, V>> {
        // size(), clear(), contains(), remove(), ...
    }

    final class KeySet extends AbstractSet<K> {
        // size(), clear(), contains(), remove(), ...
    }

    final class Values extends AbstractCollection<V> {
        // size(), clear(), contains(), remove(), ...
    }
}
```

### 📗 익명 클래스

- 이름이 없는 클래스
- 바깥 클래스의 멤버가 아님
- 쓰이는 시점에 선언 + 인스턴스 생성

#### 익명 클래스의 제약 사항

- 비정적인 문맥에서 사용될 때만 바깥 클래스의 인스턴스 참조 가능
    - static으로 선언된 메소드에서는 static만 접근이 가능하다.
- 정적 문맥에서 static final 상수 외의 정적 멤버 갖기 불가능
- 선언 지점에서만 인스턴스 생성 가능
- instanceof 검사 및 클래스 이름이 필요한 작업 수행 불가
    - 선언과 동시에 인스턴스 생성하고 더이상 사용하지 않음. 클래스 이름이 없음.
- 인터페이스 구현 및 다른 클래스의 상속 X
- 짧지 않으면 가독성 ↓

```java
public class Calculator {
    private int x;
    private int y;

    public Calculator(int x, int y) {
        this.x = x;
        this.y = y;
    }

    public int plus() {
        Operator operator = new Operator() {
            private static final String COMMENT = "더하기"; // 상수
            // private static int num = 10; // 상수 외의 정적 멤버는 불가능
          
            @Override
            public int plus() {
                // Calculator.plus()가 static이면 x, y 참조 불가
                return x + y;
            }
        };
        return operator.plus();
    }
}

interface Operator {
    int plus();
}
```

#### 언제 사용하는가?

- 즉석에서 작은 함수 객체나 처리 객체를 만드는 데 주로 사용  
  👉 람다 등장 이후로 람다가 이 역할을 대체

```java
List<Integer> list = Arrays.asList(10, 5, 6, 7, 1, 3, 4);

// 익명 클래스 사용
Collections.sort(list, new Comparator<Integer>() {
    @Override
    public int compare(Integer o1, Integer o2) {
        return Integer.compare(o1, o2);
    }
});

// 람다 도입 후
Collections.sort(list, Comparator.comparingInt(o -> o));
```

- 정적 팩터리 메소드 구현 시 사용

```java
static List<Integer> intArrayAsList(int[] a) {
    Objects.requiredNonNull(a);
    
    return new AbstracktList<>() {
        @Override public Integer get(int i) {
            return a[i];
        }
    }
}
```

### 📘 지역 클래스

- 지역 변수를 선언할 수 있는 곳이면 어디서든 선언 가능
- 유효 범위는 지역 변수와 동일

#### 각 클래스와의 공통점

- `멤버 클래스`: 이름이 있고 반복해서 사용 가능
- `익명 클래스`: 비정적 문맥에서 사용될 때만 바깥 인스턴스 참조 가능
- `익명 클래스`: 정적 멤버를 가질 수 없음
- `익명 클래스`: 가독성을 위해 짧게 작성해야 함

```java
public class TestClass {
    private int number = 10;

    public TestClass() {
    }

    public void foo() {
        // 지역변수처럼 선언
        class LocalClass {
            private String name;
            // private static int staticNumber; // 정적 멤버 가질 수 없음

            public LocalClass(String name) {
                this.name = name;
            }

            public void print() {
                // 비정적 문맥에선 바깥 인스턴스를 참조 가능
                // foo()가 static이면 number에서 컴파일 에러
                System.out.println(number + name);
            }
        }
        LocalClass localClass1 = new LocalClass("local1"); // 이름이 있고
        LocalClass localClass2 = new LocalClass("local2"); // 반복해서 사용 가능
    }
}
```

## 💡 총정리

- 멤버 클래스: 메소드 밖에서 사용해야 하거나 너무 긴 경우
    - 정적: 바깥 인스턴스 참조 X
    - 비정적: 바깥 인스턴스 참조 O
- 익명 클래스: 한 메서드 안에서만 사용 + 인스턴스 생성 시점이 단 한 곳 + 해당 타입으로 쓰기 적합한 클래스나 인터페이스 이미 존재
- 지역 클래스: 그 외. (잘 쓰이지 않음)
***

### 참고
- Effective Java / 조슈아 블로크 / Chapter 4 - item 24
- [Infa 블로그](https://inpa.tistory.com/entry/JAVA-%E2%98%95-%EB%82%B4%EB%B6%80-%ED%81%B4%EB%9E%98%EC%8A%A4Inner-Class-%EC%9E%A5%EC%A0%90-%EC%A2%85%EB%A5%98)
- 우아한테크 코스 2022-effective-java