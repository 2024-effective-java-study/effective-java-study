# Effective Java 3/E Study

## 일정
24.07.04. ~ 24.12.01

## 목록
1. [생성자 대신 정적 팩터리 메서드를 고려하라](02장/아이템_01/생성자_대신_정적_팩터리_메서드를_고려하라.md)
2. 생성자에 매개변수가 많다면 빌더를 고려하라
3. private 생성자나 열거 타입으로 싱글턴임을 보증하라
4. 인스턴스화를 막으려거든 private 생성자를 사용하라
5. 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라
6. 불필요한 객체 생성을 피하라
7. 다 쓴 객체 참조를 해제하라
8. finalizer와 cleaner 사용을 피하라
9. try-finally보다는 try-with-resources를 사용하라
10. equals는 일반 규약을 지켜 재정의하라
11. equals를 재정의하려거든 hashCode도 재정의하라
12. toString을 항상 재정의하라
13. clone 재정의는 주의해서 진행하라

## 학습 목적
* 이펙티브 자바는 자바 개발자라면 읽어야 하는 필독서 중 하나라고 생각합니다. 효율적인 자바 코드를 짜기 위한 솔루션들을 배울 수 있는 책입니다. 
하지만 책의 모든 아이템을 정독하면서 테스트 해보고, 책의 예제 외에 적용할 수 있는 코드를 작성해 보는 것을 혼자서 마치기란 쉽지 않습니다. 
따라서 여러 사람이 함께 스터디하여 학습 효율을 높여 같이 시니어 개발자로의 성장을 위해 나아가고자 합니다.
* 개인적으로 효과적인 학습 방식 중 하나가 본인이 배운 내용을 남에게 설명하거나 가르쳐주는 것이라고 생각합니다. 
자신이 맡은 아이템에 대해 발표함으로써 해당 아이템에 대해 효과적으로 학습할 수 있습니다.
* 또한 맡은 아이템이 아니더라도 책을 읽으면서 이해가 가지 않는 부분에 대해 의문점을 가져보고, 다른 크루들과의 토론으로 의문을 풀어나가는 과정을 통해 효율적인 학습을 추구하고자 합니다.

## 진행 방식

매주 화, 목 20시에 스터디를 진행합니다.
매 스터디마다 1인당 이펙티브 자바의 1개 아이템에 대한 10분 이내의 발표를 진행합니다.
발표자료의 형식은 자유입니다.
매 스터디 전날 23:59까지 발표자료를 Pull Requests로 올립니다.
조원들이 모두 발표를 마치면 질문을 받고 답변합니다.
질문 사항에 대해 발표자가 잘 모르겠다면, 스터디 진행 중 답변도 가능합니다.
본인이 발표하지 않는 아이템도 반드시 읽고 이해가 되지 않는 부분에 대해 질문을 남겨주시면 됩니다.