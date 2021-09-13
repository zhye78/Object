# Chapter 9 - 유연한 설계

이전 8장에서 유연하고 재사용 가능한 설꼐를 만들기 위해 적용할 수 있는 다양한 의존성 관리 기법들을 소개했다면, 이번 장에서는 이 기법들을 원칙이라는 관점에서 정리했다.

# 개방-폐쇄 원칙(Open-Closed Principle, OCP)

 소프트웨어 개체(클래스, 모듈, 함수 등)는 확장에 대해 열려 있어야 하고, 수정에 대해 닫혀 있어야 한다.

> 확장에 대해 열려있다 : 애플리케이션의 요구사항이 변경될 때 이 변경에 맞게 새로운 **동작**을 추가해서 애플리케이션의 기능을 확장할 수 있다. (동작의 관점)

> 수정에 대해 닫혀있다 : 기존의 **코드**를 수정하지 않고도 애플리케이션의 동작을 추가하거나 변경할 수 있다. (코드의 관점)

 개방-폐쇄 원칙은 유연한 설계란 기존의 코드를 수정하지 않고도 애플리케이션의 동작을 확장할 수 있는 설계라고 이야기한다.

### 컴파일타임 의존성을 고정시키고 런타임 의존성을 변경하라

- 개방-폐쇄 원칙은 런타임 의존성과 컴파일타임 의존성에 관한 이야기다. 앞 장에서 본 것처럼 유연하고 재사용 가능한 설계에서 런타임 의존성과 컴파일타임 의존성은 서로 다른 구조를 가진다.

- (p. 183 그림 9.1 참고) 영화 예매 시스템을 살펴보면, Movie의 관점에서 DiscountPolicy에 대한 컴파일타임 의존성과 런타임 의존성이 동일하지 않다는 것을 알 수 있다.</br>
 여기서 중복 할인 정책인 OverlappedDiscountPolicy 클래스를 DiscountPolicy의 자식 클래스로 추가했는데, 이 때 기존의 클래스들 중 어떤 코드도 수정하지 않았다.</br></br>
 이 설계는 새 할인 정책을 추가해서 기능을 확장할 수 있도록 허용하고, 기존 코드를 수정할 필요 없이 확장이 가능하다는 점이 '개방-폐쇄 원칙'이 의미하는 것과 같다.</br>

- 개방-폐쇄 원칙을 수용하는 코드는 컴파일타임 의존성을 수정하지 않고도 런타임 의존성을 쉽게 변경할 수 있다. </br>
 (p.184 그림 9.2 참고) OverlappeddiscountPolicy 클래스를 추가하더라도 Movie클래스는 여전히 DiscountPolicy 클래스에만 의존하지만 런타임시에 OverlappeddiscountPolicy 클래스로 의존성이 변경되는 것을 확인할 수 있다.</br></br>
 이렇게 의존성 관점에서 개방-폐쇄 원칙을 따르는 설꼐란 컴파일타임 의존성은 유지하면서 런타임 의존성의 가능성을 확장하고 수정할 수 있는 구조이다.</br>

### 추상화가 핵심이다.

- 개방-폐쇄 원칙의 핵심은 추상화에 의존하는 것이다.</br>
추상화란 핵심적인 부분만 남기고 불필요한 부분은 생략함으로써 복잡성을 극복하는 기법이다.</br></br>
 추상화로 생략되지 않고 남겨지는 부분은 다양한 상황에서의 공통점을 반영한 추상화의 결과물이므로, 이 공통부분은 문맥이 바뀌더라도 수정할 필요가 없어야 한다. 따라서 추상화 부분은 수정에 대해 닫혀 있고, 추상화로 생략된 부분은 확장의 여지를 남긴다. 이것이 추상화가 개방-폐쇄 원칙을 가능하게 만드는 이유다.</br></br>
 (p.285 코드 참고) DiscountPolicy 클래스에서 변하지 않는 부분은 할인 여부를 판단하는 로직이고, 변하는 부분은 할인된 요금을 계산하는 방법이다. 여기서 변하지 않는 부분을 고정하고 변하는 부분을 생략하는 추상화 메커니즘이 개방-폐쇄 원칙의 기반이 된다.</br>

- But, 단순히 어떤 개념을 추상화했다고 해서 수정에 대해 닫혀 있는 설계가 되는 것은 아니다. 개방-폐쇄 원칙에서 폐쇄를 가능하게 하는 것은 의존성의 방향이다.</br></br>
 (p.286 코드 참고) Movie는 DiscountPolicy에만 의존하고, 이 DiscountPolicy는 변하지 않는 추상화이다. Movie가 안정되어 있는 추상화인 DiscountPolicy에 의존하기 때문에 DiscountPolicy의 자식 클래스를 추가하더라도 영향을 받지 않는다. </br></br>
 올바른 추상화를 설계하고 추상화에 대해서만 의존하도록 관계를 제한함으로써 설계를 유연하게 확장할 수 있다. </br>
 주의할 점은 추상화를 했다고 해서 모든 수정에 대해 설꼐가 폐쇄되는 것이 아니고, 변경에 의한 파급효과를 최대한 피하기 위해서는 변하는 것과 변하지 않는 것이 무엇인지 이해하고 이를 추상화의 목적으로 삼아야 한다는 것이다.</br>

# 생성 사용 분리

```java
public class Movie {
	...
	private DiscountPolicy discountPolicy;
	
	public Movie(String title, Duration runningTime, Money fee){
		...
		this.discountPolicy = new AmountDiscountPolicy(...);
	}

	public Money calculateMovieFee(Screening screening){
		return fee.minus(discoutPolicy.calculateDiscountAmount(screening));
	}
}
```

- Movie가 오직 DiscountPolicy에만 의존하기 위해서는 Movie 내부에서 AmountDiscountPolicy같은 구체 클래스의 인스턴스를 생성하면 안된다. </br></br>
 위 코드는 부적절한 곳에서 객체를 생성하고 있다는 문제가 있다. 생성자 안에서는 DiscountPolicy 인스턴스를 생성하고, calculateMovieFee 메서드에서는 이 객체에게 메시지를 전송하고 있다. 
 객체만 생성하거나 메시지만 전송했다면 문제가 없었겠지만 동일한 클래스 안에서 객체 생성과 사용이라는 두 가지 이질적인 목적을 가진 코드가 공존하는 것이 문제이다.</br>

- 유연하고 재사용 가능한 설계를 원한다면 객체와 관련된 두 가지 책임을 서로 다른 객체로 분리해야 한다. 한 마디로 객체에 대한 **생성과 사용을 분리**해야 한다.</br></br>
 사용으로부터 생성을 분리하는 데 사용되는 가장 보편적인 바업ㅂ은 객체를 생성할 책임을 클라이언트로 옮기는 것이다.</br>

    ```java
    public class Client {
    	public Money getAvaterFee(){
    		Movie avatar = new Movie("아바타", 
    					Duration.ofMinus(120), 
    					Money.wons(10000), 
    					new AmountDiscountPolicy(...));
    		return avatar.getFee();
    	}
    }
    ```

     p.289 그림 9.4와 9.5를 비교해 보면, 생성에 관한 책임이 Movie의 클라이언트로 옮겨졌고, Movie의 의존성을 추상화인 DiscountPolicy로만 제한하기 때문에 확장에 대해서는 열려 있으면서 수정에 대해서는 닫혀 있는 코드를 만들 수 있다는 것을 알 수 있다.</br>

### FACTORY 추가하기

- 위에서 생성 책임을 Client로 옮긴 배경에는 Movie는 특정 컨텍스트에 묶이면 안 되지만 Client는 묶여도 상관 없다는 전제가 깔려 있는데, 이 전제도 없다고 가정해본다. </br></br>
 Client 코드를 보면 Movie 인스턴스 생성과 동시에 getFee 메시지를 함게 보내면서 역시 생성과 사용의 책임이 함께 존재한다.</br>
 이 경우, 객체 생성과 관련된 책임만 전담하는 별도의 객체를 추가하고 Client는 이 객체를 사용하도록 만들 수 있는데, 이처럼 생성과 사용을 분리하기 위해 객체 생성에 특화된 객체를 FACTORY라고 부른다.</br>

    ```java
    public class Factory{
    	public Movie createAvatarMovie(){
    		return new Movie("아바타", 
    				Duration.ofMinus(120), 
    				Money.wons(10000), 
    				new AmountDiscountPolicy(...));
    	}
    }
    ```

    ```java
    public class Client {
    	private Factory factory;

    	public Client(Factory factory){
    		this.factory = factory;
    	}

    	public Money getAvaterFee(){
    		Movie Avatar = factory.createAvatarMovie();
    		return avatar.getFee();
    	}
    }
    ```

- 이렇게 FACTORY를 사용하면 Movie와 DiscountAmountPolicy를 생성하는 책임 모두를 FACTORY로 이동할 수 있고, Client는 오직 사용과 관련된 책임만 지고 생성과 관련된 어떤 지식도 가지지 않을 수 있다.</br>

### 순수한 가공물에게 책임 할당하기

- 책임 할당의 가장 기본이 되는 원칙은 책임을 수행하는 데 필요한 정보를 가장 많이 알고 있는 정보 전문가(INFORMATION EXPERT)에게 책임을 할당하는 것이고, 도메인 모델은 정보 전문가를 찾기 위해 참조할 수 있는 일차적인 재료다.</br></br>
 방금 전 추가한 FACTORY 클래스는 도메인 모델이 아니고, 단지 전체적인 결합도를 낮추고 재사용성을 높이기 위해 도메인 개념에 할당되어 있던 책임을 이동시킨 것이다.</br>

- 시스템을 객체로 분해하는 데는 크게 표현적 분해, 행위적 분해가 존재한다.</br></br>
 표현적 분해는 도메인에 존재하는 사물 또는 개념을 표현하는 개체들을 이용해 시스템을 분해하는 것이며 객체지향 설계를 위한 가장 기본적인 접근법이다.</br></br>
 하지만 종종 도메인 개념을 표현하는 객체에게 책임을 할당하는 것만으로는 부족한 경우가 발생한다. 이런 경우 도메인 객체가 아닌 설계자가 편의를 위해 임의로 만들어낸 가공의 객체에게 책임을 할당해서 문제를 해결해야 한다. </br>
 이런 객체를 **PURE FABRICATION(순수한 가공물)** 이라고 부른다. 위의 FACTORY 클래스가 이 PURE FABRICATION에 해당한다. </br></br>
 이렇게 추가된 PURE FABRICATION은 보통 특정한 행동을 표현하는 것이 일반적이다. 따라서 PURE FABRICATION은 표현적 분해보다는 행위적 분해에 의해 생성되는 것이 일반적이다.</br>

- 먼저 도메인의 본질적인 개념을 표현하는 추상화를 이용해 애플리케이션을 구축하기 시작하고, 도메인 개념이 만족스럽지 못하다면 주저하지 말고 인공적인 객체인 PURE FABRICATION을 창조해야 한다.</br>

# 의존성 주입

- **의존성 주입(Dependency Injection)** : 사용하는 객체가 아닌 외부의 독립적인 객체가 인스턴스를 생성한 후 이를 전달해서 의존성을 해결하는 방법.
- 의존성을 해결하는 세 가지 방법 : 생성자 주입, setter 주입, 메서드 주입 (이전 8장 참고)

### 숨겨진 의존성은 나쁘다

- **SERVICE LOCATOR** 패턴 : 외부에서 객체에게 의존성을 전달하는 의존성 주입과 달리 객체가 직접 SERVICE LOCATOR에게 의존성 해결을 요청하는 패턴
- SERVICE LOCATOR 역할을 하는 ServiceLocator 클래스를 사용하면 Movie는 직접 ServiceLocator의 메서드를 호출해서 DiscountPolicy에 대한 의존성을 해결한다.

    ```java
    public class Movie{
    	...
    	private DiscountPolicy discountPolicy;

    	public Movie(String title, Duration runningTime, Money fee){
    		this.title = title;
    		this.runningTime = runningTime;
    		this.fee = fee;
    		this.discountPolicy = ServiceLocator.discountPolicy;
    	}
    }
    ```

    ```java
    public class ServiceLocator{
    	private static ServiceLocator soleInstance = new ServiceLocator();
    	private DiscountPolicy discountPolicy;

    	public static DiscountPolicy discountPolicy(){
    		return soleInstance.discountPolicy;
    	}

    	public static void provide(DiscountPolicy discountPolicy){
    		soleInstance.discountPolicy = discountPolicy;
    	}

    	private ServiceLocator(){
    	}
    }
    ```

    ```java
    ServiceLocator.provide(new AmountDiscountPolicy(...));
    Movie avatar = new Movie("아바타", 
    				Duration.ofMinus(120), 
    				Money.wons(10000));
    ```

- SERVICE LOCATOR의 가장 큰 단점은 의존성을 감춘다는 것이다. Movie는 DiscountPolicy에 의존하고 있지만 Movie의 퍼블릭 인터페이스 어디에도 이 의존성에 대한 정보가 표시되어있지 않다.</br></br>
 이 경우, 의존성과 고나련된 문제가 컴파일타임이 아닌 런타임에 가서야 발견이 되고, 단위 테스트 작성 또한 어렵다는 문제가 있다. 더 큰 문제는 의존성을 이해하기 위해 코드의 내부 구현을 이해할 것을 강요한다는 것인데, 이는 캡슐화를 위반하게 된다.</br></br>
 반면에 의존성 주입은 이 문제들을 깔끔하게 해결한다. 따라서 명시적인 의존성이 숨겨진 의존성보다 좋으며, 책에서는 가급적 의존성을 객체의 퍼블릭 인터페이스에 노출하라는 말을 하고 있다.</br>

# 의존성 역전 원칙

### 추상화와 의존성 역전

```java
public class Movie{
	private AmountDiscountPolicy discountPolicy;
}
```

- 위 코드는 구체 클래스에 대한 의존성으로 인해 결합도가 높아지고 재사용성과 유연성이 저해된다. </br>
 이 설계가 변경에 취약한 이유는 상위 수준 클래스인 Movie가 하위 수준 클래스인 AmountDiscountPolicy에 의존하기 때문이다.</br>
- 객체 사이의 협력이 존재할 때 그 협력의 본질을 담고 있는 것은 상위 수준 클래스이다. </br>
상위 수준 클래스는 하위 수준의 변경에 영향을 받으면 안 되며, **상위 수준 클래스는 어떤 식으로든 하위 수준 클래스에 의존해서는 안 되는 것**이다.
- 이 경우에도 추상화로 해결이 가능하다.</br>
(p.301 그림 9.8 참고) Movie와 AmountDiscountPolicy 모두 **추상화에 의존하도록** 수정하면 하위 수준 클래스의 변경을 인해 상위 수준의 클래스가 영향을 받는 것을 방지할 수 있고, 상위 수준을 재사용할 때 하위 수준의 클래스에 얽매이지 않을 수 있다.</br>
- 이를 **의존성 역전 원칙(Dependency Inversion Principle, DIP)** 이라고 한다.</br>

### 의존성 역전 원칙과 패키지

- 역전은 의존성의 방향 뿐만 아니라 인터페이스의 소유권에도 적용된다.</br></br>
(p.303 그림 9.9 참고) Movie를 정상적으로 컴파일하기 위해서는 DiscountPolicy 클래스가 필요한데, 문제는 DiscountPolicy가 포함돼 있는 패키지 안에 구체 클래스들도 포함돼 있다는 것이다.</br></br>
이 경우, DiscountPolicy가 포함된 패키지 안의 어떤 클래스가 수정되더라도 패키지 전체가 재배포되어야 한다. 따라서 불필요한 클래스들을 같은 패키지에 두는 것은 전체적인 빌드 시간을 가파르게 상승시킨다.</br></br>
(p.304 그림 9.10 참고) 그렇기 때문에 추상화를 별도의 독립적인 패키지가 아니라 클라이언트가 속한 패키지에 포함시켜야 한다. 그림과 같이 Movie와 DiscountPolicy를 하나의 패키지로 모으는 것은 Movie를 특정 컨텍스트로부터 완벽하게 독립시킨다.</br>
- 잘 설계된 객체지향 애플리케이션에서는 인터페이스의 소유권을 서버가 아닌 클라이언트에 위치시키고, 이것은 객체지향 프레임워크의 모듈 구조를 설계하는 데 가장 중요한 핵심 원칙이다.
훌륭한 객체지향 설계를 위해서는 의존성을 역전시켜야 한다.</br>

# 유연성에 대한 조언

### 유연한 설계는 유연성이 필요할 때만 옳다

- 유연한 설계라는 말의 이면에는 복잡한 설계라는 의미가 숨어 있고, 유연성은 항상 복잡성을 수반하며, 유연하지 않은 설계가 단순하고 명확하다.
불필요한 유연성은 불필요한 복잡성을 낳으므로, 단순하고 명확한 해법이 그런대로 만족스럽다면 유연성을 제거하는 것이 좋다.

### 협력과 책임이 중요하다

- 객체를 생성하는 방법에 대한 결정은 모든 책임이 자리를 잡은 후 가장 마지막 시점에 내리는 것이 적절하다. 
의존성을 관리해야 하는 이유는 역할, 책임, 협력의 관점에서 설계가 유연하고 재사용 가능해야 하기 때문이다. 따라서 역할, 책임, 협력에 먼저 집중하는 것이 좋다.
