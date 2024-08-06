## 캡슐화의 이점을 제공하지 못하는 클래스

```java
class Point {
	public int x;
	public int y;
	
	public Point(int x, int y) {
		this.x = x;
		this.y = y;
	}
}
```

- API를 수정하지 않고는 내부 표현을 바꿀 수 없다.
- 불변식을 보장할 수 없다. (클라이언트가 직접 값을 변경할 수 있음)
- 외부에서 필드에 접근할 때 부수 작업을 수행할 수도 없다.

### 올바른 객체 지향 프로그래밍

1. 필드들을 모두 private으로 변경
2. public 접근자(getter), 변경자(setter) 추가

→ 패키지 바깥에서 접근할 수 있는 클래스라면 접근자를 제공함으로써 클래스 내부 표현 방식을 언제든 바꿀 수 있는 유연성을 얻을 수 있다.

## package-private, private 중첩 클래스

```java
public class TopPoint {
	private static class Point {
		public double x;
		public double y;
	}
	
	public Point getPoint() {
		Point point = new Point();
		point.x = 3.5;
		point.y = 4.5;
		return point;
	}
}
```

- 패키니 내부(or TopPoint 클래스)에서는 얼마든지 Point 클래스의 필드를 조작할 수 있음
- 패키지 외부(or 외부 클래스)에서는 Point 클래스의 필드에 직접 접근할 수 없음
- 클래스를 통해 표현하려는 추상 개념만 상세하고 올바르게 표현하면 됨

## 결론

public 클래스는 절대 가변 필드를 직접 노출해서는 안 된다.

불변 필드라면 노출해도 덜 위험하지만 완전히 안심할 수는 없다.

하지만 package-private, private 중첩 클래스에서는 종종 (불변이든 가변이든) 필드를 노출하는 편이 나을 때도 있다.