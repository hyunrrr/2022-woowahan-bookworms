# 이펙티브 자바

아이템 15) 클래스와 멤버의 접근 권한을 최소화하라 (p96-101)


## 아이템15) 클래스와 멤버의 접근 권한을 최소화하라

## 핵심 정리
- 프로그램 요소의 접근성은 가능한 최소한으로
- 꼭 필요한 것만 골라 최소한의 public API를 설계
- public 클래스는 상수용 public static final 필드 외에는 어떠한 public 필드도 가져선 안됨
- public static final 필드가 참조하는 객체가 불변인지 확인해라

### 정보은닉의 장점
- 시스템 개발 속도를 높임
- 시스템 관리 비용을 낮춤
- 정보 은닉 자체가 성능을 높이진 않지만, 성능 최적화에 도움을 줌
- 소프트웨어 재사용성을 높임
- 큰 시스템을 제작하는 난이도를 낮춤

-> 접근 제한자를 제대로 활용하는 것이 정보 은닉의 핵심
### 모든 클래스와 멤버의 접근성을 가능한 한 좁히자
- 클래스의 공개 api를 세심히 설계한 후, 그 외의 모든 멤버는 private으로 만듦
- 그 후 오직 같은 패키지의 다른 클래스가 접근해야 하는 멤버에 한해 package-private으로 풀어줌
- 권한을 풀어주는 일이 빈번하면 시스템에서 컴포넌트를 더 분해해야 하는 것은 아닌지 고민
- 단, **Serializable**을 구현한 클래스에서는 그 필드들도 의도치 않게 공개 api가 될 수도 있음(아이템 86, 87)

### 멤버 접근성을 좁히지 못하게 방해하는 제약
- 상위 클래스의 메서드를 재정의할 때는 그 접근 수준을 상위 클래스에서 보다 좁게 설정 할 수 없음. 리스코프 치환 원칙(아이템 10)을 지키기 위해 필요

### 코드 테스트 목적으로 클래스, 인터페이스, 멤버의 접근 범위를 적당한 수준까지 넓히는 것은 괜찮다
- 예를들어 public클래스의 private멤버를 package-private까지 풀어주는 것까진 괜찮음
- 하지만 테스트만을 위해 클래스, 인터페이스, 멤버를 공개 api로 만들어선 안됨

### public 클래스의 인스턴스 필드는 되도록 public이 아니여야 한다(아이템 16)
- 필드가 가변 객체를 참조하거나, final이 아닌 인스턴스 필드를 public으로 선언하면 해당 필드와 관련된 모든 것은 불변식을 보장할 수 없게 됨
<br>_(외부에서 자유롭게 접근해 바꿀 수 있기 떄문인거 같음)_
- 필드가 수정될 때 (lock 획득 같은) 다른 작업을 할 수 없게 되므로 public 가변 필드를 갖는 클래스는 일반적으로 스레드 세이프 하지 않음
<br>_(lock과 관련된 필드가 공개되어 외부에서 변경되면 lock이 걸릴 수 있다는 말인듯?)_
- 필드가 final이면서 불변 객체를 참조하더라도 public이면 리펙터링시 해당 필드를 삭제하는 방식으로 내부 구현을 바꿀 수 없음
<br>_(외부에서 사용하고 있기때문에 맘대로 삭제 못한다는 말인듯..?)_
- 꼭 필요한 구성요소로써 상수라면 public static final 필드로 공개해도 됨.
<br>이런 필드는 반드시 기본 타입 값이나 불변 객체를 참조해야 함.
<br>가변 객체를 참조하면 다른 객체를 참조하진 못하지만 참조된 객체 자체가 수정될 수 있음
<br>_(가변 객체의 필드가 final이 아니라면 해당 필드가 수정될 수 있음)_

### 길이가 0이 아닌 배열은 모두 변경 가능하니 주의하자
- 클래스에서 public static final 배열필드를 두거나
<br>이 필드를 반환하는 접근자 메서드를 제공하면 안됨
- 해결책<br>1. 배열을 private으로 만들고 public 불변리스트를 추가
<br>2. 배열을 private으로 만들고 그 복사본을 반환

```java
private static final Thing[] PRIVATE_VALUES = {...};
public static final List<Thing> VALUES =
        Collections.unmodifiableList(Arrays.asList(PRIVATE_VALUES));
```

```java
private static final Thing[] PRIVATE_VALUES = {...};
public static final List<Thing> values() {
        return PRIVATE_VALUES.clone();
}
```

### 자바 9에선 "모듈 시스템"이라는 것이 도임됨
- 모듈 개념이 받아질지 미지수라 당분간은 사용하지 않는게 좋을것 같다