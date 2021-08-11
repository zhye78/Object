# Chapter 5 - 책임 할당하기

지난 4장에서 데이터 중심의 접근법을 취할 경우 직면하게 되는 다양한 문제점들을 살펴보았다.

 이번 5장에서는 2장에서 구현한 책임 중심 설계 코드를 따라가보며 객체에 책임을 할당하는 기본적인 원리를 알아간다.
<br/>
<br/>

# 책임 주도 설계를 향해

 데이터 중심 설계에서 책임 주도 설계로 전환하기 위한 두 가지 원칙

- 데이터보다 행동을 먼저 결정해라

  : 데이터는 객체가 책임을 수행하는 데 필요한 재료를 제공할 뿐이라는 것을 알고, 책임을 먼저 결정한 후에 객체의 상태를 결정하도록 한다.

- 협력이라는 문맥 안에서 책임을 결정해라

  : 객체의 책임은 객체의 입장이 아닌, 객체가 참여하는 협력에 적합하게 결정하도록 한다. 협력을 시작하는 주체는 메시지이기 때문에 메시지를 결정한 후에 적합한 객체를 선택해야 한다.
 
<br/>

✔책임 주도 설계의 핵심은 **책임을 결정한 후**에 책임을 수행할 객체를 결정하는 것

<br/>

# 책임 할당을 위한 GRASP 패턴

> GRASP(General Responsibility Assignment Software Pattern) 
: 일반적인 책임 할당을 위한 소프트웨어 패턴

<br/>

- 도메인 개념에서 출발

 -객체에 어떤 책임을 할당해야 할 때 가장 먼저 고민해야 하는 유력한 후보는 바로 도메인 개념(p.137 그림 5.1 참고)

 -BUT, 설계를 시작하기 위한 출발점일 뿐, 도메인 개념을 완벽하게 정리할 필요는 X

<br/>

- 정보 전문가에게 책임 할당

 -"메시지를 전송할 객체는 무엇을 원하는가?"와 "메시지를 수신할 적합한 객체는 누구인가?" 의 순서로 메시지와 객체를 선택한다.

 -**정보 전문가 패턴** 사용 : 책임 수행에 필요한 정보를 가장 많이 알고 있는 개체에게 메시지를 처리할 책임을 할당하도록 한다. (p.139 ~ p.141 영화 예매 예시 참고)

<br/>

- 높은 응집도와 낮은 결합도

 -설계는 트레이드오프 활동 : 책임 할당에 다양한 대안들이 존재한다면 응집도와 결합도의 측면에서 정보 전문가 패턴 말고도 더 나은 대안을 선택할 수 있다.

 -**LOW COUPLING(낮은 결합도) 패턴과 HIGH COHESION(높은 응집도) 패턴** 을 사용하여 더욱 유연한 설계를 얻을 수도 있다. (p.142 ~ p.144 참고)

<br/>

- 창조자에게 객체 생성 책임 할당

 -**CREATOR 패턴** : 객체를 생성할 책임을 어떤 객체에게 할당할지의 지침 제공하며, 이미 존재하는 객체 사이의 관계를 이용하기 때문에 설계가 낮은 결합도를 유지할 수 있게 한다. (p.145 참고)

<br/>

# 구현을 통한 검증

- 영화 예매 시스템을 예로 들고, "예매해라" 메시지에 응답할 Screening 구현으로 시작한다.

```java
public class Screening {
	private Movie movie;
	private int sequence;
	private LocalDateTime whenScreened;

	public Reservation reserve(Customer customer, int audienceCount){
		return new Reservation(customer, this, calculateFee(audienceCount), audienceCount);
	}

	public Money calculateFee(int audienceCount){
		return movie.calculateMovieFee(this).times(audienceCount);
	}

	public LocalDateTime getWhenScreened(){
		return whenScreened;
	}

	public int getSequence(){
		return sequence;
	}
}
```

 → Screening은 Movie와 협력하기 위해 calculateMovieFee 메시지를 전송한다.

```java
public class Movie {
	private String title;
	private Duration runningTime;
	private Money fee;
	private List<DiscountCondition> discountConditions;

	private MovieType movieType;
	private Money discountAmount;
	private double discountPercent;

	public Money calculateMovieFee(Screening screening){
		if(isDiscountable(screening)){ return fee.minus(calculateDiscountAmount(); }
		return fee;
	}

	private boolean isDiscountable(Screening screening) {
		return discountConditions.stream().anyMatch(condition -> condition.isSatisfiedBy(screening));
	}

	private Money calculateDiscountAmount() {
		switch(movieType) {
			case AMOUNT_DISCOUNT: //금액 할인 정책
				return calculateAmountDiscountAmount();
			case PERCENT_DISCOUNT: //비율 할인 정책
				return calculatePercentDiscountAmount();
			case NONE_DISCOUNT: //미적용
				return calculateNoneDiscountAmount();
		}

		throw new IllegalStateException();
	}

	private Money calculateAmountdiscountAmount() {
		return discountAmount;
	}

	private Money calculatePercentdiscountAmount() {
		return fee.times(discountPercent);
	}

	private Money calculateNonediscountAmount() {
		return Money.ZERO;
	}
}
```

 → Movie는 각 DiscountCondition에 "할인 여부를 판단하라"는 메시지를 전송한다.

```java
public class DiscountCondition {
	private DiscountConditionType type;
	private int sequence;
	private DayOfWeek dayOfWeek;
	private LocalTime startTime;
	private LocalTime endTime;

	public boolean isSatisfiedBy(Screening screening) {
		if(type == DiscountConditionType.PERIOD){ return isSatisfiedByPeriod(screening); }
		return isSatisfiedBySequence(screening);
	}
	
	private boolean isSatisfiedByPeriod(Screening screening) {
		return dayOfWeek.equals(screening.getWhenScreened().getDayOfWeek()) &&
			startTime.compareTo(screening.getWhenScreened().toLocalTime()) <= 0 &&
			endTime.isAfter(screening.getWhenScreened().toLocalTime()) >= 0;
	}

	private boolean isSatisfiedBySequence(Screening screening) {
		return sequence == screening.getSequence();
	}
}
```

<br/>

- DiscountCondition 개선

 위 코드의 문제점은 변경에 취약한 클래스 DiscountCondition을 포함하고 있다는 것이다. 하나 이상의 변경 이유를 가지기 때문에 응집도가 낮고, 따라서 변경의 이유에 따라 클래스를 분리해야 한다.

1. **인스턴스 변수가 초기화되는 시점** 살펴보기 
: DiscountCondition이 순번 조건을 표현하는 경우 sequence 변수만이 초기화되고 나머지 변수들은 초기화되지 않는다. → 함께 초기화되는 속성을 기준으로 코드를 분리한다.
2. **메서드들이 인스턴스 변수를 사용하는 방식** 살펴보기 
: isSatisfiedByPeriod 메서드는 sequence를 뺀 나머지 변수들만을 사용한다. → 속성 그룹과 해당 그룹에 접근하는 메서드 그룹을 기준으로 코드를 분리한다.

<br/>

- 타입 분리

  DiscountCondition의 가장 큰 문제는 순번 조건과 기간 조건이라는 두 개의 독립적인 타입이 하나의 클래스 안에 공존학 있다는 점이다.

 → SequenceCondition, PeriodCondition 클래스로 분리한다.

```java
public class PeriodCondition {
	private DayOfWeek dayOfWeek;
	private LocalTime startTime;
	private LocalTime endTime;

	public PeriodCondition(DayOfWeek dayOfWeek, LocalTime startTime, LocalTime endTime){
		this.dayOfWeek = dayOfWeek;
		this.startTime = startTime;
		this.endTime = endTime;
	}

	public boolean isSatisfiedBy(Screening screening){
		return dayOfWeek.equals(screening.getWhenScreened().getDayOfWeek()) &&
			startTime.compareTo(screening.getWhenScreened().toLocalTime()) <= 0 &&
			endTime.isAfter(screening.getWhenScreened().toLocalTime()) >= 0;
	}
}
```

```java
public class SequenceCondition {
	private int sequence;

	public SequenceCondition(int sequence){
		this.sequence = sequence;
	}

	private boolean isSatisfiedBy(Screening screening) {
		return sequence == screening.getSequence();
	}
}
```

 클래스를 분리하여 위에서 언급했던 두 가지 문제들이 해결되었다.

 BUT, 수정 후에 Movie 클래스는 SequenceCondition, PeriodCondition 두 가지와 모두 협력해야 한다. → Movie 클래스 안에서 SequenceCondition, PeriodCondition 목록을 따로 유지하는 방법은 Movie 클래스가 양쪽 모두에게 결합된다는 문제가 있다. 결합도가 높아진다.

 또한, 새 할인 조건 추가가 더 어려워졌다. DiscountCondition 입장에서는 응집도가 높아졌지만, 전체적으로는 변경과 캡슐화 관점에서 품질이 더 나빠졌다.

<br/>

- 다형성을 통해 분리

 역할의 개념을 가져와 DiscountCondition 객체의 타입을 추상화한다.

 -**다형성(POLYMORPHISM)** 패턴

```java
public class DiscountCondition {
	boolean isSatisfiedBy(Screening screening);
}
```

```java
public class PeriodCondition implements DiscountCondition { ... }
public class SequenceCondition implements DiscountCondition { ... }
```

 → 이렇게 하면 Movie는 협력 객체의 구체적인 타입은 몰라도 된다.

<br/>

- 변경으로부터 보호하기

 이제 새로운 할인 조건을 추가하는 경우에도 DiscountCondition 클래스가 그 구체적인 타입을 캡슐화하고, Movie에 대한 어떤 수정도 필요가 없다.

 -**변경 보호(PROTECTED VARIATIONS)** 패턴 : 변경을 캡슐화하도록 책임을 할당

<br/>

- Movie 클래스 개선하기

 Movie 역시 이전의 DiscountCondition 처럼 금액할인 영화와 비율할인 영화를 모두 가지고 있어 응집도가 낮다. 

 → DiscountCondition과 같은 방법인 **다형성** 패턴과 **변경 보호** 패턴으로 해결하면 된다.

<br/>

- 변경과 유연성

 코드를 수정하지 않고도 변경을 수용할 수 있도록 더 유연한 코드를 만들어야 한다.

 → 상속 대신 **합성**을 사용하는 것도 좋은 방법(p.164 그림 5.8 참고)

<br/>

# 책임 주도 설계의 대안

> **리팩터링(Refactoring)** : 이해하기 쉽고 수정하기 쉬운 소프트웨어로 개선하기 위해 겉으로 보이는 동작은 바꾸지 않은 채 내부 구조를 변경하는 것

<br/>

- 메서드 응집도

 -몬스터 메서드 : 응집도가 낮아서 이해, 재사용, 변경이 어려운 긴 메서드

 이런 메서드는 작게 분해해서 각 메서드의 응집도를 높여야 한다.

-객체를 자율적으로

 각 객체는 자신이 소유하고 있는 데이터를 자기 스스로 처리하도록 만든다.
