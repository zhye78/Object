# Chapter 6 - 메시지와 인터페이스

훌륭한 객체지향 코드를 얻기 위해서는 협력 안에서 객체가 수행하는 책임, 즉 메시지에 초점을 맞추어야 한다. 이 메시지들이 객체의 퍼블릭 인터페이스를 구성한다.

 이번 장에서는 유연하고 재사용 가능한 퍼블릭 인터페이스를 만드는 데 도움이 되는 설계 원칙과 기법을 익히고 적용하게 된다.

<br/>

# 협력과 메시지

- 클라이언트-서버 모델

 -협력 : 어떤 객체가 다른 객체에게 무언가를 요청할 때 시작

 -메시지 : 객체 사이의 협력을 가능하게 하는 매개체

 -**클라이언트-서버(client-server) 모델** : 두 객체 사이의 협력 관계를 설명하기 위해 사용하는 전통적인 방법으로, 메시지를 전송하는 객체를 클라이언트, 메시지를 수신하는 객체를 서버라고 부른다.

 (p.176 그림 6.1~6.3 참고) 보통 객체는 협력에 참여하는 동안 클라이언트와 서버의 역할을 동시에 수행하는 것이 일반적이다.

 따라서 협력에 적합한 객체를 설계하기 위해서는 객체가 수신하는 메시지의 집합 뿐만 아니라 외부에 전송하는 메시지의 집합도 함께 고려하는 것이 바람직하다.

<br/>

- 메시지와 메시지 전송

 -메시지 : 객체들이 협력하기 위해 사용할 수 있는 유일한 의사소통 수단.

 -메시지 전송(=메시지 패싱) : 한 객체가 다른 객체에게 도움을 요청하는 것.

 -메시지 전송자 : 메시지 전송하는 객체(=클라이언트)

 -메시지 수신자 : 메시지 수신하는 객체(=서버)

```java
condition.isSatisfiedBy(screening);
```

 여기서 isSatisfiedBy(screening)이 오퍼레이션명+인자 조합인 **메시지**이고, 

 메시지 수신자를 추가한 condition.isSatisfiedBy(screening) 이 **메시지 전송**이다.

<br/>

- 메시지와 메서드

 메시지를 수신했을 때 실제로 어떤 코드가 실행되는지는 메시지 수신자의 실제 타입이 무엇인가에 달려 있다.

 위 코드의 condition은 DiscountCondition이라는 인터페이스 타입으로 정의되어 있지만, 실제 실행되는 코드는 인터페이스를 실체화한 클래스의 종류에 따라 달라진다.

 이처럼 메시지를 수신할 때 실제로 실행되는 함수 또는 프로시저를 **메서드**라고 부르며, 코드 상에서 동일한 이름의 변수에 동일한 메시지를 전송하더라도 객체의 타입에 따라 실행되는 메서드가 달라질 수 있다. 

 → 이와 같은 메시지와 메서드의 구분은 메시지 전송자와 메시지 수신자가 느슨하게 결합될 수 있게 한다. 메시지 전송자는 자신이 어떤 메시지를 전송해야 하는지만 알면 되고, 메시지 수신자는 메시지가 도착했다는 사실만 알면 된다.

<br/>

- 퍼블릭 인터페이스와 오퍼레이션

 -퍼블릭 인터페이스 : 객체가 의사소통을 위해 외부에 공개하는 메시지의 집합

 -오퍼레이션 : 퍼블릭 인터페이스에 포함된 메시지(구현이 아닌 추상화)

 -메서드 : 메시지를 수신했을 때 실제로 실행되는 코드(실제 구현 포함)

<br/>

- 시그니처

 -시그니처 : 오퍼레이션 또는 메서드의 이름과 파라미터 목록

 -p.181 용어 정리 체크하고 다음으로 넘어가자

<br/>

# 인터페이스와 설계 품질

 좋은 인터페이스는 **최소한의 인터페이스**와 **추상적인 인터페이스**라는 조건을 만족해야 한다.

 → 이를 위해 책임 주도 설계 방법을 따르자

 책임 주도 설계 방법을 따르면서 훌륭한 인터페이스가 가지는 공통 특징을 아는 것은 분명 도움이 되므로, 퍼블릭 인터페이스의 품질에 영향을 미치는 원칙과 기법을 살펴본다.

<br/>

- 디미터 법칙

 4장에서 살펴본 절차적인 방식의 영화 예매 시스템 코드 중 할인 가능 여부를 체크하는 코드를 보자.

```java
public class ReservationAgency {
	public Reservation reserve(Screening screening, Customer customer, int audienceCount){
		Movie movie = screening.getMovie();

		boolean discountable = false;
		for(DiscountCondition condition : movie.getDiscountConditions()){
			if(condition.getType() == DiscountConditionType.PERIOD){
				discountable = screening.getWhenScreened().getDayOfWeek().equals(condition.getDayOfWeek()) &&
					condition.getStartTime().compareTo(screening.getWhenScreened().toLocalTime()) <= 0 &&
					condition.getEndTime().compareTo(screening.getWhenScreened().toLocalTime()) >= 0;
			}else{
				discountable = condition.getSequence() == screening.getSequence();
			}

			if(discountable) break;
			...
		}
	}
}
```

 이 코드의 가장 큰 단점은 ReservationAgency와 인자로 전달된 Screening의 결합도가 너무 높기 때문에 Screening의 내부 구현을 변경할 때마다 ReservationAgency도 함께 변경된다는 것이다.

 → 이처럼 협력하는 객체의 내부 구조에 대한 결합으로 인해 발생하는 설꼐 문제를 해결하기 위해 제안된 원칙이 바로 **디미터 법칙(Law of Demeter)**이다. 

 디미터 법칙을 따르기 위해서는 클래스가 특정한 조건을 만족하는 대상에게만 메시지를 전송하도록 프로그래밍해야 한다.

1. this 객체
2. 메서드의 매개변수
3. this의 속성
4. this의 속성인 컬렉션의 요소
5. 메서드 내에서 생성된 지역 객체

```java
public class ReservationAgency {
	public Reservation reserve(Screening screening, Customer customer, int audienceCount){
		Money fee = screening.calculateFee(audieneCount);
		return new Reservation(customer, screening, fee, audienceCount);
	}
}
```

 바로 이전의 코드에서 결합도 문제를 해결한 최종 코드이다.

 이 코드에서 ReservationAgency는 메서드의 인자로 전달된 Screening 인스턴스에게 만 메시지를 전송하며, 두 객체가 결합돼 있지 않기 때문에 Screening의 변경이 ReservationAgency에 영향을 끼치지 않는다.

 이렇게 불필요한 어떤 것도 다른 객체에게 보여주지 않으며,  다른 객체의 구현에 의존하지 않는 코드를 **부끄럼타는 코드(shy code)**라고 부른다.

```java
screening.getMovie().getDiscountCondition();

screening.calculateFee(audienceCount);
```

 위의 첫 번째 코드가 디미터 법칙을 위반하는 대표적인 사례이다. 메시지 전송자가 수신자의 내부 구조에 대해 물어보고, 반환받은 요소에 대해 연쇄적으로 메시지를 전송하는 **기차 충돌(train wreck)** 코드이다.

 반면 개선된 아래 코드는 단지 원하는 것이 무엇인지 명시하고 단순히 수행하도록 요청만 한다.

<br/>

- 묻지 말고 시켜라

 디미터 법칙은 훌륭한 메시지는 객체의 상태에 관해 묻지 말고 원하는 것을 시켜야 한다는 사실을 강조한다. → **묻지 말고 시켜라(Tell, Don't Ask):** 이런 스타일의 메시지 작성을 장려하는 원칙을 가리키는 용어

 이 원칙을 따르면 객체의 정보를 이용하는 행동은 그 객체의 내부에 위치시키기 때문에 자연스럽게 정보와 행동을 동일한 클래스에 두게 된다.

 따라서 묻지 말고 시켜라 원칙대로 메시지를 결정하다 보면 정보 전문가에게 책임을 할당하게 되고, 높은 응집도를 가진 클래스를 얻을 확률이 높아진다.

<br/>

- 의도를 드러내는 인터페이스

 켄트 백의 메서드 명명 두 가지 방법

1. 메서드가 작업을 어떻게 수행하는지 나타내도록 명명 - 이 경우 메서드의 이름은 내부의 구현 방법을 드러낸다.

    ```java
    public class PeriodCondition {
    	public boolean isSatisfiedByPeriod(Screening screening){ ... }
    }

    public class SequenceCondition {
    	public boolean isSatisfiedBySequence(Screening screening){ ... }
    }
    ```

     이 방법은,
     -두 메서드의 내부 구현을 정확히 이해하지 못한다면 클라이언트 입장에서 두 메서드가 동일한 작업을 수행한다는 사실을 알아채기 어렵고, 
     -메서드 수준에서 캡슐화를 위반한다는 문제점이 있다.

2. '어떻게'가 아니라 '무엇'을 하는지 드러내도록 명명

    ```java
    public class PeriodCondition implements DiscountCondition {
    	public boolean isSatisfiedBy(Screening screening){ ... }
    }

    public class SequenceCondition {
    	public boolean isSatisfiedBy(Screening screening){ ... }
    }
    ```

    ```java
    public interface DiscountCondition implements DiscountCondition {
    	boolean isSatisfiedBy(Screening screening);
    }
    ```


 변경된 코드는 두 메서드가 동일한 목적을 가진다는 것을 메서드명을 통해 명확히 표현했고, 두 메서드를 가진 PeriodCondition, SequenceCondition 클래스를 동일한 타입으로 간주할 수 있도록 DiscountCondition 인터페이스를 실체화했다.


 이처럼 어떻게 하느냐가 아니라 무엇을 하느냐에 따라 메서드의 이름을 짓는 패턴을 **의도를 드러내는 선택자(Intention Revealing Selector)**라고 부르며, 이를 인터페이스 레벨로 확장한 것이 **의도를 드러내는 인터페이스(Intention Revealing Interface)**이다.

<br/>

- 함께 모으기

위의 원칙들을 이해하는 좋은 방법 중 하나는 원칙을 위반하는 코드를 살펴보는 것이다.

1. 디미터 법칙을 위반하는 티켓 판매 도메인 (p.191 Theater 코드 참고)
```java
Ticket ticket = ticketSeller.getTicketOffice().getTicket();
audience.getBat().minusAmount(ticket.getFee());
```
 Theater 클래스의 위 부분은 Theater가 ticketSeller와 audience의 내부에 포함된 객체에도 접근한다는 문제를 보여준다.
 이런 디미터 법칙을 위반한 코드를 수정하는 일반적인 방법은 Audience와 TicketSeller의 내부 구조를 묻는 대신 각각의 객체가 직접 자신의 책임을 수행하도록 시키는 것이다.

2. 묻지 말고 시켜라 (p.193 ~ 195 코드 참고)
Theater는 TicketSeller와 Audience 의 내부 구조에 관해 묻지 말고 원하는 작업을 시켜야 한다.
코드를 살펴보면, 최종적으로 디미터 법칙을 준수하는 TicketSeller와 Audience를 얻은 것을 확인할 수 있다.

3. 인터페이스에 의도를 드러내자
코드를 리팩터링했지만 아직 현재의 인터페이스는 클라이언트의 의도를 명확히 드러내지 못한다. 따라서 클라이언트의 의도가 분명히 드러나도록 객체의 퍼블릭 인터페이스를 개선해야 한다.
-TicketSeller의 setTicket() → sellTo()
-Audience의 setTicket() → buy()
-Bag의 setTicket() → hold()
오퍼레이션은 객체 자신이 아닌 클라이언트의 의도를 표현하는 이름을 가져야 한다.

<br/>

# 원칙의 함정

원칙이 현재 상황에 부적합하다고 판단된다면 과감하게 원칙을 무시해도 된다. 지금까지 설명한 원칙들을 적용할 때 고려해볼 만한 이슈 몇 가지를 살펴보자.

<br/>

- 디미터 법칙은 하나의 도트(.)를 강제하는 규칙이 아니다.

앞에서 살펴본 디미터 법칙은 "오직 하나의 도트만을 사용하라"는 말이 되기도 하는데, 하나 이상의 도트를 사용하는 모든 케이스가 디미터 법칙 위반인 것은 아니고, 객체의 내부에 대한 어떤 내용도 묻지 않는다면 상관 없다.

<br/>

- 결합도와 응집도의 충돌

모든 상황에서 맹목적으로 묻지 말고 시켜라와 디미터 법칙을 준수하는 것은 같은 퍼블릭 인터페이스 안에 어울리지 않는 오퍼레이션들이 공존하게 할 수 있고, 결과적으로 객체는 상관 없는 책임들을 한꺼번에 떠안게 되기 때문에 응집도가 낮아진다.

p.200의 PeriodCondition 코드를 보면 Screening의 내부 상태를 가져와 사용하기 때문에 캡슐화를 위반한 것처럼 보이지만, 이를 수정한 p.201의 Screening, PeriodCondition 코드는 각 객체가 본질적인 책임 외에 다른 책임까지 떠안게 되어 객체의 응집도가 낮아진 것을 확인할 수 있다.

따라서 이 경우에는 Screening의 캡슐화를 향상시키는 것보다 Screening의 응집도를 높이고, Screening과 PeriodCondition의 결합도를 낮추는 것이 더 좋은 방법이다.

<br/>

# 명령-쿼리 분리 원칙

-**프로시저** : 내부의 상태를 변경하는 루틴의 한 종류(=**명령, 반환값 X, 상태변경 O**)

-**함수** : 필요한 값을 계산해서 반환하는 루틴의 한 종류(=**쿼리, 반환값 O, 상태변경 X**)

-**명령-쿼리 인터페이스** : 명령-쿼리 원칙에 따라 작성된 객체의 인터페이스

<br/>

- 반복 일정의 명령과 쿼리 분리하기

이벤트와 반복 일정을 예시로 살펴보자.

```java
public class Event {
private String subject;
private LocalDateTime from;
private Duration duration;

public Event(String subject, LocalDateTme from, Duration duration) {
	this.subject = subject;
	this.from = from;
	this.duration = duration;
}

public boolean isSatisfied(RecurringSchedule schedule) {
	if(from.getDayOfWeek() != schedule.getDayOfWeek() ||
			!from.toLocalTime().equals(schedule.getFrom()) ||
			!duration.equals(schedule.getDuration())) {

		reschedule(schedule); //이 부분이 문제!
		return false;
	}
	return true;
	}

	private void reschedule(RecurringSchedule schedule) {
		return schedule.getDayOfWeek().getValue() - from.getDayOfWeek().getValue();
	}

	private long daysDistance(RecurringSchedule schedule) {
		return schedule.getDayOfWeek().getValue() - from.getDayOfWeek().getValue();
	}
}
```

```java
public class RecurringSchedule {
	private String subject;
	private DayOfWeek dayOfWeek;
	private LocalTime from;
	private Duration duration;

	public RecurringSchedule(String subject, DayOfWeek dayOfWeek,
			LocalTime from, Duration duration) {
		this.subject = subject;
		this.dayOfWeek = dayOfWeek;
		this.from = from;
		this.duration = duration;
	}

	//getter...
}
```

Event와 RecurringSchedule 클래스를 만들고, 이벤트가 반복 일정 조건을 만족시키는지 체크하는 코드를 작성해서 체크해보면,

```java
RecurringSchedule schedule = new RecurringSchedule("회의", DayOfWeek.WEDNESDAY,
	LocalTime.of(10, 30), Duration.ofMinutes(30));
Event meeting = new Event("회의", LocalDateTime.of(2019, 5,9,10, 30), 
	Duration.ofMinus(30));

assert meeting.isSatisfied(schedule) == false;
assert meeting.isSatisfied(schedule) == true;
```

처음에는 false로 잘 나오지만, 같은 코드를 두 번째 실행하면 true가 나온다.

이는 Event의 isSatisfied 메서드에서 인자로 전달된 값들이 현재 이벤트의 값과 동일하지 않을 때, false 반환 직전 reschedule 메서드를 호출하여 Event 객체의 상태를 수정하기 때문이다.

그리고, 이 버그를 찾기 어려운 이유는 isSatisfied가 상태를 변경하는 명령과, 상태를 반환하는 쿼리의 두 가지 역할을 동시에 수행하고 있었기 때문이다.

따라서 명령과 쿼리를 뒤섞지 않고, 명확하게 분리하는 것이 좋다.

```java
public class Event {
	public boolean isSatisfied(RecurringSchedule schedule){ ... }
	public void reschedule(RecurringSchedule schedule){ ... }
}
```

```java
if(!event.isSatisfied(schedule)) {
	event.reschedule(schedule);
}
```

<br/>

- 명령-쿼리 분리와 참조 투명성

-**참조 투명성** : 어떤 표현식 e가 있을 때 모든 e를 e의 값으로 바꾸더라도 결과가 달라지지 않는 특성

-**부수효과** : 동일한 입력값을 사용하더라도 결괏값이 달라질 수 있도록 하는 것

-**불변성** : 어떤 값이 변하지 않는 성질, 부수효과가 발생하지 않는다는 말과 동일

수학에서의 함수는 어떤 값도 변경하지 않기 때문에 부수효과가 존재하지 않아 항상 불변성을 만족시키고, 이는 모든 로직이 참조 투명성을 만족시킨다는 말이 된다.

반면 객체지향 프로그래밍은 객체의 상태 변경이라는 부수효과를 기반으로 하기 때문에 참조 투명성은 예외에 가깝다. 명령-쿼리 분리 원칙을 사용하면 부수효과를 가지는 명령으로부터 부수효과를 가지지 않는 쿼리를 명확하게 분리함으로써 제한적이나마 참조 투명성의 혜택을 누릴 수 있게 한다.

<br/>

- 책임에 초점을 맞춰라

디미터 법칙, 묻지 말고 시켜라를 따르면서 의도를 드러내는 인터페이스를 설계하는 쉬운 방법은, 지금까지 계속 강조해왔던 메시지를 먼저 선택하고 그 후에 메시지를 처리할 객체를 선택하는 것이다.

메시지를 먼저 선택하는 방식은 디미터 법칙, 묻지 말고 시켜라, 의도를 드러내는 인터페이스, 명령-쿼리 분리 원칙에 긍정적인 형향을 끼친다. (p.214 참고)
