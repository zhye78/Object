# Chapter 12 - 다형성

10장에서 상속에 대해 보고 11장에서는 합성에 대해 보았는데, 이번 장에서는 상속의 관점에서 다형성이 구현되는 기술적인 메커니즘을 살펴본다.

# 다형성

**다형성(Polymorphism)**: 그리스어로 "많은 형태를 가질 수 있는 능력"을 의미한다.

1. **오버로딩 다형성**: 하나의 클래스 안에 동일한 이름의 메서드가 존재하는 경우
2. **강제 다형성**: 동일한 연산자를 다양한 타입에 사용할 수 있는 방식(ex. + 연산자)
3. **매개변수 다형성**: 클래스의 인스턴스 변수나 메서드의 매개변수 타입을 임의의 타입으로 선언한 후 사용하는 시점에 구체적인 타입으로 지정하는 방식(ex. T)
4. **포함 다형성**: 메시지가 동일하더라도 수신한 객체의 타입에 따라 실제로 수행되는 행동이 달라지는 능력(=서브타입 다형성), 이번 장에서 중점적으로 다룰 다형성.

# 상속의 양면성

- 상속의 목적은 코드 재사용이 아니고, 상속은 프로그램을 구성하는 개념들을 기반으로 다형성을 가능하게 하는 타입 계층을 구축하기 위한 것이다.

### 상속을 사용한 강의 평가

```java
Pass:3 Fail:2, A:1 B:1 C:1 D:0 F:2
```

 위같은 형식으로 전체 수강생들의 성적 통계를 출력할 예정이고, 이미 우리는 Pass와 Fail까지 표현하는 Lecture 클래스를 가지고 있다. 

```java
public class Lecture {
    private int pass;
    private String title;
    private List<Integer> scores = new ArrayList<>();

    public Lecture(String title, int pass, List<Integer> scores) {
        this.title = title;
        this.pass = pass;
        this.scores = scores;
    }

    public double average() {
        return scores.stream().mapToInt(Integer::intValue).average().orElse(0);
    }

    public List<Integer> getScores() {
        return Collections.unmodifiableList(scores);
    }

    public String evaluate() {
        return String.format("Pass:%d Fail:%d", passCount(), failCount());
    }

    private long passCount() {
        return scores.stream().filter(score -> score >= pass).count();
    }

    private long failCount() {
        return scores.size() - passCount();
    }
}
```

 이 Lecture 클래스를 상속받아 나머지 부분까지 출력하는 GradeLecture 클래스를 만든다.

```java
public class GradeLecture extends Lecture {
    private List<Grade> grades;

    public GradeLecture(String name, int pass, List<Grade> grades, List<Integer> scores) {
        super(name, pass, scores);
        this.grades = grades;
    }

    @Override
    public String evaluate() { //--->메서드 오버라이딩
        return super.evaluate() + ", " + gradesStatistics();
    }

    private String gradesStatistics() {
        return grades.stream().map(grade -> format(grade)).collect(joining(" "));
    }

    private String format(Grade grade) {
        return String.format("%s:%d", grade.getName(), gradeCount(grade));
    }

    private long gradeCount(Grade grade) {
        return getScores().stream().filter(grade::include).count();
    }

    public double average(String gradeName) { //--->메서드 오버로딩
        return grades.stream()
                .filter(each -> each.isName(gradeName))
                .findFirst()
                .map(this::gradeAverage)
                .orElse(0d);
    }

    private double gradeAverage(Grade grade) {
        return getScores().stream()
                .filter(grade::include)
                .mapToInt(Integer::intValue)
                .average()
                .orElse(0);
    }
}
```

```java
Lecture lecture = new GradeLecture("객체지향 프로그래밍",
																		70,
																		Arrays.asList(new Grade("A", 100, 95),
																									new Grade("B", 94, 80),
																									new Grade("C", 79, 70),
																									new Grade("D", 69, 50),
																									new Grade("E", 49, 0)),
																		Arrays.asList(81, 95, 75, 50, 45));

//결과 => Pass:3 Fail:2, A:1 B:1 C:1 D:1 F:1
lecture.evaluate();
```

### 데이터 관점의 상속

- 위의 Lecture클래스의 인스턴스를 생성하면 시스템은 인스턴스 변수를 저장할 수 있는 메모리 공간을 할당하고, GradeLecture의 인스턴스를 생성하면 GradeLecture가 정의한 인스턴스 변수뿐만 아니라 부모 클래스인 Lecture가 정의한 인스턴스 변수까지 함께 저장할 수 있는 메모리 공간이 할당된다.
- 데이터 관점에서 상속은 자식 클래스의 인스턴스 안에 부모 클래스의 인스턴스를 포함하는 것.

### 행동 관점의 상속

- 부모 클래스가 정의한 일부 메서드를 자식 클래스의 메서드로 포함시키는 것.
이것이 가능한 이유는 런타임에 시스템이 자식 클래스에 정의되지 않은 메서드가 있을 경우 이 메서드를 부모 클래스 안에서 탐색하기 때문이다.
- 객체의 경우 각 인스턴스별로 독립적인 메모리를 할당받아야 하지만, 메서드의 경우에는 동일한 클래스의 인스턴스끼리 공유가 가능하기 때문에 클래스는 한 번만 메모리에 로드하고 각 인스턴스별로 클래스를 가리키는 포인터를 갖게 하는 것이 경제적이다.

 (p.402 그림 12.5 참고) 인스턴스는 두 개가 생성되었지만 클래스는 단 하나만 생성되었다는 것을 확인할 수 있다.
 (p.403 그림 12.6 참고) 자식 클래스인 GradeLecture의 인스턴스를 생성했을 때, 부모 클래스인 Lecture의 인스턴스를 내부에 포함하고, class 포인터와 parent 포인터를 따라가면 부모 클래스의 메서드에도 접근할 수 있는 것을 확인할 수 있다. 물론 이 그림은 상속을 이해하기 쉽도록 표현한 개념적인 그림일 뿐.

# 업캐스팅과 동적 바인딩

### 같은 메시지, 다른 메서드

 실행 시점에 메서드를 탐색하는 과정을 살펴보기 위해 성적 계산 프로그램에 교수별 강의에 대한 성적 통계를 계산하는 기능을 추가한다.

```java
public class Professor {
    private String name;
    private Lecture lecture;

    public Professor(String name, Lecture lecture) {
        this.name = name;
        this.lecture = lecture;
    }

    public String compileStatistics() {
        return String.format("[%s] %s - Avg: %.1f", name,
                lecture.evaluate(), lecture.average());
    }
}
```

- Professor의 compileStatistics 메서드는 통계 정보를 생성하기 위해 Lecture의 evaluate() 와 average 메서드를 호출한다.
 (p. 404 코드 참고) Professor클래스 생성자의 두 번째 인자로 Lecture가 저달되든 GradeLecture가 전달되든 아무 문제 없이 실행된다는 것을 알 수 있다.
- 이처럼 코드 안에서 선언된 참조 타입과 무관하게 실제로 메시지를 수신하는 객체의 타입에 따라 실행되는 메서드가 달라질 수 있는 것은 업캐스팅과 동적 바인딩이라는 메커니즘이 작용하기 때문이다.

-**업캐스팅**: 부모 클래스 타입으로 선언된 변수에 자식 클래스의 인스턴스 할당 가능

-**동적 바인딩**: 선언된 변수 타입이 아니라 메시지 수신 객체의 타입에 따라 실행되는 메서드가 결정되는데, 이는 객체지향 시스템이 메시지를 처리할 적절한 메서드를 컴파일 시점이 아닌 실행 시점에 결정하기 때문이다.

### 업캐스팅

 모든 객체지향 언어는 명시적으로 타입을 변환하지 않고도 부모 클래스 타입의 참조 변수에 자식 클래스의 인스턴스를 대입할 수 있게 허용한다.

```java
Lecture lecture = new GradeLectrue(...);
```

 반대로 부모 클래스의 인스턴스를 자식 클래스 타입으로 변환하기 위해 명시적인 타입 캐스팅을 하는 것을 다운 캐스팅이라고 한다.

```java
Lecture lecture = new GradeLectrue(...);
GradeLecture gradeLecture = (GradeLecture)lecture;
```

 컴파일러의 관점에서 자식 클래스는 아무 제약 없이 부모 클래스를 대체할 수 있기 때문에 부모 클래스와 협력하는 클라이언트인 Professor는 다양한 자식 클래스의 인스턴스와도 협력이 가능하다는 무한한 확장 가능성을 지닌다.

### 동적 바인딩

-정적 바인딩: 함수를 호출하는 전통적인 언어들이 컴파일타임에 호출할 함수를 결정하는 방식
(= 초기 바인딩, 컴파일타임 바인딩)

-동적 바인딩: 객체지향 언어에서 실행될 메서드를 런타임에 결정하는 방식(= 지연 바인딩)

- 객체지향 언어가 제공하는 업캐스팅과 동적 바인딩을 이용하면 부모 클래스 참조에 대한 메시지 전송을 자식 클래스에 대한 메서드 호출로 변환할 수 있다.

# 동적 메서드 탐색과 다형성

- **self 참조**: 우리가 흔히 쓰는 this. 객체가 메시지를 수신하면 컴파일러는 self 참조라는 임시 변수를 생성한 후 메시지를 수신한 객체를 가리키도록 설정한다.
- (그림 12.9 참고) 객체지향 시스템이 메서드를 선택하는 규칙을 나타낸다. 메시지를 수신한 객체, self 참조가 가리키는 객체/클래스를 먼저 탐색하고, 찾지 못했다면 parent 포인터를 따라 부모 클래스로 이동해서 메서드를 탐색한다. 최상위 클래스에 이르러서도 메서드를 찾지 못했다면 예외를 발생시키고 종료한다.
- 메서드 탐색은 두 가지 원리로 구성된다.
 첫 번째는 **자동적인 메시지 위임,** 두 번째는 **동적인 문맥 사용**.

### 자동적인 메시지 위임

 상속 계층 안의 클래스는 메시지를 처리할 방법을 알지 못할 경우 메시지에 대한 처리를 부모 클래스에게 위임하고, 적절한 메서드를 찾을 떄까지 상속 계층을 따라 부모 클래스로 처리가 위임된다.

- **메서드 오버라이딩**
 자식 클래스가 부모 클래스에 존재하는 메서드와 동일한 시그니처를 가진 메서드를 재정의해서 부모 클래스의 메서드를 감추는 현상으로, 동적 메서드 탐색은 self 참조가 가리키는 객체의 클래스(자식)부터 시작되고, 먼저 발견된 메서드가 실행된다.
- **메서드 오버로딩**
 이름은 같지만 시그니처가 다르기 때문에 동일한 이름의 메서드가 공존하는 경우
(C++은 상속에서 지원하지 않음 → 동적 메서드 탐색 규칙은 언어마다 다를 수 있으므로 사용하는 언어의 규칙을 주의깊게 살펴보자)

### 동적인 문맥

- 메시지 수신 시 실제로 어떤 메서드를 실행할지 컴파일 시점이 아닌 런타임 시점에 결정한다.
 메시지를 수신한 객체가 무엇이냐에 따라 메서드 탐색을 위한 문맥이 동적으로 바뀌며, 이 동적인 문맥을 결정하는 것은 메시지를 수신한 객체를 가리키는 self 참조이다.

- **self 전송**
self 참조가 가리키는 자기 자신에게 메시지를 전송하는 것.

```java
public class Lecture {
		public String stats() {
        return String.format("Title: %s, Evaluation Method: %s",
                title, getEvaluationMethod());
    }

    public String getEvaluationMethod() {
        return "Pass or Fail";
    }
}
```

 이 경우 Lecture가 stats 메시지를 수신하면, self 참조는 메시지를 수신한 Lecture 인스턴스를 가리키게 되고, stats() 메서드를 실행하던 중에 self 참조가 가리키는 현재 객체에게 getEvaluationMethod 메시지를 전송하게 된다.

```java
public class GradeLecture extends Lecture {
		@Override
    public String getEvaluationMethod() {
        return "Grade";
    }
}
```

 하지만 이런 상속이 끼어든 경우에는 GradeLecture가 stats 메시지를 수신해서 self 참조는 GradeLecture 인스턴스를 가리키게 되고, 부모 클래스 Lecture에서 stats()를 발견하여 실행, getEvaluationMethod 탐색은 다시 self 참조가 가리키는 GradeLecture에서부터 시작된다.

 이처럼 self 전송은 자식 클래스에서 부모 클래스 방향으로 진행되는 동적 메서드 탐색 경로를 다시 self 참조가 가리키는 원래의 자식 클래스로 이동시켜, 극단적으로 이해하기 어려운 코드가 만들어질 수 있다.

### 이해할 수 없는 메시지

- 정적 타입 언어
 코드 컴파일 시 상속 계층 안의 클래스들 중 메시지를 처리할 메서드가 없다면 컴파일 에러를 발생시킨다. (ex. 자바)
- 동적 타입 언어
 컴파일 단계가 존재하지 않아서 실제로 코드를 실행해야 메시지 처리 가능 여부를 판단하고 예외를 던진다. (ex. 스몰토크, 루비)
 추가로, 예외 메시지에 응답할 수 있는 메서드 구현이 가능하다.

### self 대 super

- 대부분의 객체지향 언어들은 자식 클래스에서 부모 클래스의 인스턴스 변수나 메서드에 접근하기 위해 사용할 수 있는 **super 참조**를 제공한다.
- super 참조의 정확한 의도는 "부모 클래스의 메서드를 실행하세요"가 아닌, "지금 이 클래스의 부모 클래스부터 메서드 탐색을 시작하세요"이다. 만약 부모 클래스에서도 찾지 못하면 더 상위의 부모 클래스를 검사한다.
- 이처럼 super 참조를 통해 메시지를 전송하는 것을 **super 전송**이라고 부른다.

 

# 상속 대 위임

### 위임과 self 참조

- (p.426 그림 12.20, 12.21 참고)메서드 탐색 중에는 자식 클래스의 인스턴스와 부모 클래스의 인스턴스가 동일한 self 참조를 공유하는 것으로 봐도 무방하다.
- (p.427 코드) GradeLecture에서 Lecture로 self 참조가 공유되는 과정

```ruby
class Lecture
  def initialize(name, scores)
    @name = name
    @scores = scores
  end

  def stats(this)
    "Name: #{@name}, Evaluation Method: #{this.evaluationMethod(this)}"
  end

  def getEvaluationMethod()
    "Pass or Fail"
  end
end
```

```ruby
class GradeLecture
  def initialize(name, canceled, scores)
    @parent = Lecture.new(name, scores)
    @canceled = canceled
  end

  def stats(this)
    @parent.stats(this) //직접 처리하지 않고 부모에게 요청 전달
  end

  def getEvaluationMethod()
    "Grade"
  end
end
```

 위 코드에서는 @parent에 부모 클래스 Lecture의 인스턴스를 할당하는 것으로 상속 관계를 흉내낸다.

 주목해야 할 부분은 GradeLecture의 stats 메서드가 메시지를 직접 처리하지 않고 부모 클래스인 Lecture의 stats 메서드에게 요청을 전달한다는 것이다.

- 이처럼 자신이 수신한 메시지를 다른 객체에게 동일하게 전달해서 처리를 요청하는 것을 **위임**이라고 부른다.
 그리고 이 위임이 객체 사이의 동적인 연결 관계를 이용해 상속을 구현하는 방법이며, 상속을 구현하면 실행 시에 인스턴스들 사이에서 self 참조가 자동으로 전달된다는 것을 알 수 있다.

 

### 프로토타입 기반의 객체지향 언어

- 클래스가 존재하지 않고 오직 객체만 존재하는 프로토타입 기반의 객체지향 언어(ex. 자바스크립트)에서 상속을 구현하는 유일한 방법은 객체 사이의 위임을 이용하는 것이다.

```jsx
function Lecture(name, scores) {
    this.name = name;
    this.scores = scores;
}

Lecture.prototype.stats = function() {
    return "Name: "+ this.name + ", Evaluation Method: "+ this.getEvaluationMethod();
}

Lecture.prototype.getEvaluationMethod = function() {
    return "Pass or Fail"
}
```

```jsx
function GradeLecture(name, canceled, scores) {
    Lecture.call(this, name, scores);
    this.canceled = canceled;
}

GradeLecture.prototype = new Lecture();

GradeLecture.prototype.constructor = GradeLecture;

GradeLecture.prototype.getEvaluationMethod = function() {
    return "Grade"
}
```

```jsx
var grade_lecture = new GradeLecture("OOP", false, [1, 2, 3]);
grade_lecture.stats();
```

 위 자바스크립트 코드의 prototype은 앞의 Ruby 코드에서 부모 객체를 가리키기 위해 사용했던 @parent와 동일한 것으로 봐도 무방하다. prototype 체인으로 연결된 객체 사이에 메시지를 위임함으로써 상속을 구현할 수 있다.

- 코드에서 알 수 있듯이, 메서드를 탐색하는 과정은 클래스 기반 언어의 상속과 거의 동일하며, 차이점이 있다면 정적인 클래스 간의 관계가 아니라 동적인 객체 사이의 위임을 통해 상속을 구현한다는 것이다.
 (p.433 그림 12.22 참고) 클래스가 존재하지 않기 때문에 오직 객체들 사이의 메시지 위임만을 이용해 다형성을 구현할 수 있다는 사실을 알 수 있다.
- 클래스 기반의 상속과 객체 기반의 위임 사이에 기본 개념과 메커니즘이 공유된다!