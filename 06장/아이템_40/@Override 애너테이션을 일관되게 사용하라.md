# @Override 애너테이션을 일관되게 사용하라

## @Override의 특징
- 메서드 선언에만 달 수 있다.
- 이 애너테이션이 달렸다는 것은 상위 타입의 메서드를 재정의했음을 뜻한다.
- 이 애너테이션을 일관되게 사용하면 여러 가지 악명 높은 버그들을 예방해준다.

---

### 영어 알파벳 2개로 구성된 문자열을 표현하는 클래스 - 버그를 찾아보자.
```java
public class Bigram {
    private final char first;
    private final char second;

    public Bigram(char first, char second) {
        this.first = first;
        this.second = second;
    }
    
    public boolean equals(Bigram b) {
        return b.first == first && b.second == second;
    }
    
    public int hashcode(){
        return 31 * first + second;
    }
    
    public static void main(String[] args){
        Set<Bigram> s = new HashSet<>();
        for (int i=0;i<10;i++){
            for(char ch = 'a'; ch <='z';ch++){
                s.add(new Bigram(ch,ch));
            }
        }
      System.out.println(s.size());
    }
}
```
main 메서드를 보면 똑같은 소문자 2개로 구성된 바이그램 26개를 10번 반복해 집합에 추가한 다음, 그 집합의 크기를 출력하고 있다.<br>
Set은 중복을 허용하지 않으므로, 26이 출력될 것 같지만, 실제로는 260이 출력된다.

### 왜 그럴까 ⁉️
겉으로 보이기에는 Bigram은 equals, hashcode를 모두 재정의한것처럼 보인다.<br>
그런데 안타깝게도 equals를 재정의(overriding)한게 아니라 <span style="color:red">다중정의</span>(overloading)해버렸다.<br> 
Object의 equals를 재정의하려면 매개변수 타입을 Object로 해야만 하는데, 그렇게 하지 않은 것이다.<br>
그래서 Object에서 상속한 equals와는 별개인 equals를 새로 정의한 꼴이 되었다.<br>
Object의 equals는 == 연산자와 똑같이 객체 식별성(identity)만을 확인한다.<br>
따라서 같은 소문자를 소유한 바이그램 10개가 각각이 서로 다른 객체로 인식되어, 260을 출력한 것이다.

---
### 결론
상위 클래스의 메서드를 재정의할 때는 반드시 @Override 애너테이션을 사용하자.<br>
단, 예외가 한 가지 있다. 구체 클래스에서 상위 클래스의 추상 메서드를 재정의할 때는 @Override를 사용하지 않아도 된다.<br>
구체 클래스에서 아직 구현되지 않은 추상 메서드가 있을 경우, 컴파일러가 이를 바로 알려주기 때문이다.
