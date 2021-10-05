# Chapter 12 - 다형성

10장에서 상속에 대해 보고 11장에서는 합성에 대해 보았는데, 이번 장에서는 상속의 관점에서 다형성이 구현되는 기술적인 메커니즘을 살펴본다.

# 다형성

**다형성(Polymorphism)**: 그리스어로 "많은 형태를 가질 수 있는 능력"을 의미한다.

1. **오버로딩 다형성**: 하나의 클래스 안에 동일한 이름의 메서드가 존재하는 경우
2. **강제 다형성**: 동일한 연산자를 다양한 타입에 사용할 수 있는 방식(ex. + 연산자)
3. **매개변수 다형성**: 클래스의 인스턴스 변수나 메서드의 매개변수 타입을 임의의 타입으로 선언한 후 사용하는 시점에 구체적인 타입으로 지정하는 방식(ex. T)
4. **포함 다형성**: 메시지가 동일하더라도 수신한 객체의 타입에 따라 실제로 수행되는 행동이 달라지는 능력(=서브타입 다형성), 이번 장에서 중점적으로 다룰 다형성.

# 상속의 양면성

- 상속의 목적은 코드 재사용이 아니고, 상속은 프로그램을 구성하는 개념들을 기바으로 다형성을 가능하게 하는 타입 계층을 구축하기 위한 것이다.

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

 (그림 12.5 참고) 인스턴스는 두 개가 생성되었지만 클래스는 단 하나만 생성되었다는 것을 확인할 수 있다.
 (그림 12.6 참고) 자식 클래스인 GradeLecture의 인스턴스를 생성했을 때, 부모 클래스인 Lecture의 인스턴스를 내부에 포함하고, class 포인터와 parent 포인터를 따라가면 부모 클래스의 메서드에도 접근할 수 있는 것을 확인할 수 있다. 물론 이 그림은 상속을 이해하기 쉽도록 표현한 개념적인 그림일 뿐.

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

- 모든 객체지향 언어는 명시적으로 타입을 변환하지 않고도 부모 클래스 타입의 참조 변수에 자식 클래스의 인스턴스를 대입할 수 있게 허용한다.

```java
Lecture lecture = new GradeLectrue(...);
```

- 반대로 부모 클래스의 인스턴스를 자식 클래스 타입으로 변환하기 위해 명시적인 타입 캐스팅을 하는 것을 다운 캐스팅이라고 한다.

```java
Lecture lecture = new GradeLectrue(...);
GradeLecture gradeLecture = (GradeLecture)lecture;
```

- 컴파일러의 관점에서 자식 클래스는 아무 제약 없이 부모 클래스를 대체할 수 있기 때문에 부모 클래스와 협력하는 클라이언트인 Professor는 다양한 자식 클래스의 인스턴스와도 협력이 가능하다는 무한한 확장 가능성을 지닌다.

### 동적 바인딩

-정적 바인딩: 함수를 호출하는 전통적인 언어들이 컴파일타임에 호출할 함수를 결정하는 방식
(= 초기 바인딩, 컴파일타임 바인딩)

-동적 바인딩: 객체지향 언어에서 실행될 메서드를 런타임에 결정하는 방식(= 지연 바인딩)

- 객체지향 언어가 제공하는 업캐스팅과 동적 바인딩을 이용하면 부모 클래스 참조에 대한 메시지 전송을 자식 클래스에 대한 메서드 호출로 변환할 수 있다.

# 동적 메서드 탐색과 다형성

- **self 참조**: 우리가 흔히 쓰는 this. 객체가 메시지를 수신하면 컴파일러는 self 참조라는 임시 변수를 생성한 후 메시지를 수신한 객체를 가리키도록 설정한다.
- (그림 12.9 참고) 객체지향 시스템이 메서드를 선택하는 규칙을 나타낸다. 메시지를 수신한 객체, self 참조가 가리키는 객체/클래스를 먼저 탐색하고, 찾지 못했다면 parent 포인터를 따라 부모 클래스로 이동해서 메서드를 탐색한다. 최상위 클래스에 이르러서도 메서드를 찾지 못했다면 예외를 발생시키고 종료한다.
- 메서드 탐색은 두 가지 원리로 구성된다.
 첫 번째는 자식 클래스가 이해할 수 없는 메시지를 전송받으면 부모 클래스에게 처리를 위임하는 **자동적인 메시지 위임.** 
 두 번째는 메시지 수신 시 실제로 어떤 메서드를 실행할지 컴파일 시점이 아닌 런타임 시점에 결정한다는 **동적인 문맥 사용**.
- 메시지가 처리되는 문맥 이해를 위해서는 정적인 코드 분석 외에도 런타임에 실제로 메시지를 수신한 객체가 어떤 타입인지 추적해야 하는데, 여기서 가장 중요한 역할을 하는 것이 self 참조.

### 자동적인 메시지 위임