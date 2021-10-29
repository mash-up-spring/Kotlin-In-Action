# 7장 연산자 오버로딩과 기타 관례



이번 장에서 다룰 내용

```
- 연산자 오버로딩
- 관례: 특별한 이름이 붙은 메소드
- 위임 프로퍼티
```



관례: 어떤 언어의 기능과 소정의 이름을 가진 함수를 연결해주는 기법



## 7.1 산술 연잔자 오버로딩

- 연산자 오버로딩에서 우려먹을 Point 클래스

```kotlin
data class Point(val x: Int, val y: Int){
  operator fun plus(ohter: Point): Point{
    return Point(x+other.x, y+ohter.y)
  }
  operator fun times(cnt : Int): List<Person>{
    val list = mutableListOf()
    //cnt만큼 List에 person 추가
    return list
  }
}

println(Point(10,20) + Point(20,30))
```

- fun 앞에 operator를 붙여줌으로써 이 메서드는 언어 자체의 연산자 +와 연관된다는 것을 의미한다.
- 멤버함수로 만들 수도 있고 기존 클래스를 건들기 뭐 하면 확장함수로 만들면 된다.
- 어떤 함수가 어떤 연산자와 연관되는 지는 이름에 정해진 규칙을 따른다.



### 7.1.1 이항 연산자 오버로딩

| 식   | 함수이름      |
| ---- | ------------- |
| a*b  | times         |
| a/b  | div           |
| a%b  | mod(1.1~ rem) |
| a+b  | plus          |
| a-b  | minus         |

- 연산자 우선순위는 그대로 유지된다.
- 두 피연산자가 같은 타입일 필요는 없다(ex. Point 확대 예시) -> 오버로딩도 가능
- 어쨋든 함수호출이기 때문에 교환법칙이 성립하지는 않는다.
- **비트 연산자는 없으나** 중위표기 지원하는 함수를 이용해서 있는체 할 수있다.
- 중위 연산자는 fun 앞에 infix를 붙이면 됐던거 Remind~



### 7.1.2 복합 대입 연산자

- plus 만들면 += 도 만들어 준다.
- inPlace 연산으로 만들고 싶으면 위 함수 이름 표에 Assign을 붙여주자.
- 컴파일 오류가 생길 수 있으므로 plus와 plusAssign을 동시에 정의하지 말자.
- 코틀린의 철학답게, mutable 객체와 immuatable 객체를 분리하자.



### 7.1.3 단항 연산자 오버로딩

```kotlin
operator fun Point.unaryMinus(): Point {
    return Point(-x, -y)
}
```

- 마찬가지로 이름만 바꿔서 단항 연산자도 만들 수 있다.

| 식       | 함수이름   |
| -------- | ---------- |
| +a       | unaryPlus  |
| -a       | unaryMinus |
| !a       | not        |
| ++a, a++ | inc        |
| --a, a-- | dec        |

- inc나 dec를 만들어줌으로써 전위/후위 연산 기능 두가지를 함수 하나로 구현할 수 있다.



## 7.2 비교 연산자 오버로딩



### 7.2.1 동등성 연산자: equals

- == 은 사실 equals 호출이다. : 사실 연산자 오버로딩과 같은 원리였던것!
- 사실 equals는 data class 로 만들면 알아서 해준다.
  - 동등성 비교
  - 타입 비교
  - 값 비교
- equals는 Any 타입에서 정의된 함수이므로 override로 만들어준다.
- 확장함수로는 만들 수 없다. (상속된게 우선순위가 높기 때문)



### 7.2.2 순서 연산자: compareTo

- \> , \<, \<=,\>= : compareTo에 대응된다.
- obj1<obj2 일 때 obj1.compareTo(obj2) < 0
- 내부적으로 비교는 compareValuesBy(this, other, 비교함수1, 비교함수2)를 쓰면 된다.
- 비교함수는 람다나 프로퍼티/메소드일 수 있다.

```kotlin
class Person(val first:String, val last: String):Comparable<Person>{
  override fun compareTo(other: Person) = compareValuesBy(this,other, Person::lastName, Person::firstName)
}
```

- compareValuesBy보다 직접 구현이 더 빠르다.
- **처음에는 성능에 너무 신경쓰지 말고 후에 성능 개선을 하자. -> 모듈화를 해놓자**



## 7.3 컬렉션과 범위에 대해 쓸 수 있는 관례



### 7.3.1 인덱스로 원소에 접근: get과 set

```kotlin
operator fun Point.get(index: Int): Int {
    return when(index) {
        0 -> x
        1 -> y
        else ->
            throw kotlin.IndexOutOfBoundsException("Invalid coordinate $index")
    }
}
```

- 배열에 [idx] 를 이용해 원소에 접근할 수 있는 것처럼 객체에 대해서도 마찬가지로 쓸 수 있다.
- operator + get / set으로 구현하면 된다.
- 인덱스는 맵이 String을 쓰는 것처럼 다양한 타입으로 구현 가능, **[a,b]** 처럼 여러 인덱스 사용도 가능하다



### 7.3.2 in 관례

- in에 대응하는 함수는 contains 이다.

```kotlin
operator fun Rectangle.contains(p : Point) : Boolean{
  return p.x in upperLeft.x until lowerRight.x && p.y in upperLeft.y until lowerRight.y
}
```

- a in c -> a가 파라미터 c가 수신객체 ( c.contains(a) 로 컴파일된다.)
- 10 until 20  : 10~19
- 10 .. 20: 10~20



### 7.3.3 rangeTo 관례

- .. 연산자는 rangeTo를 간략하게 표기한 방법이다.

```kotlin
operator fun <T: Comparable<T>> T.rangeTo(that: T): ClosedRange<T>

val now = LocalDate.now()
val vacation = now..now.plusDays(10)
println(now.plusWeeks(1) in vacation) 
```

- Comparable 인터페이스를 구현하면 rangeTo가 이미 정의되어 있다.
- Range 객체를 반환하므로 in을 통해 해당 범위에 속하는지 검사할 수 있다.
- ..을 쓸땐 연산자 우선순위 때문에 괄호로 잘 감싸주자.
- (0..(n+1))..forEach~



### 7.3.4 for 루프를 위한 iterator 관례

- for 루프 내에서 사용되는 in 은 iterator 이름을 가진 함수를 호출한다.
- iterator 함수는 iterator를 반환하며, hasNext와 next로 순회한다.



## 7.4 구조 분해 선언과 component 함수

- 구조 분해 선언은 componentN함수를 호출한다.

```kotlin
val (a,b) = p   -> val a = p.component1(), val b = p.component2()
```

- 코틀린 표준 라이브러리는 component1~5 까지 제공한다.
- 6이상 쓰면 컴파일 오류, n 이상 없는데 n 쓰면 IndexOutOfBound 익셉션 발생



## 7.5 프로퍼티 접근자 로직 재활용: 위임 프로퍼티

- 위임 프로퍼티란 복잡한 get / set의 로직을 객체로 캡슐화해서 그 객체에 위임하는 프로퍼티를 의미한다.
- 따라서 복잡한 방식으로 작동하는 프로퍼티 접근자 로직을 재구현 필요 X



### 7.5.1 위임 프로퍼티 소개

```kotlin
class Foo {
    val p: type by Delegate()
}
```

- 프로퍼티 p에 접근 시 get 과 set을 Delegate() 객체에 위임한다.
- p의 get -> Delegate().getValue()
- p의 set -> Delegate().setValue(newValue)
- Delegate 객체는 앞에서 설명한 관례를 따른다.
  - get: operator fun getValue(...)
  - set: operator fun setValue(...)
- by 키워드: 프로퍼티와 프로퍼티의 get/set을 담당할 위임 객체를 연결하는 키워드



### 7.5.2 위임 프로퍼티 사용: by lazy()

- 사용 예시 1 지연초기화를 사용해야 할 경우
  - 뒷받침 필드를 null로 선언한다.
  - get 시 뒷받침 필드가 null 일경우 Loading하고 아니면 그냥 가져온다.
  - 스레드 세이프하지 않다.
- lazy를 쓰면 문제가 깔끔히 해결된다.
  - val email by lazy{ loadingFuction(this) }
  - 위 로직을 줄여서 표현한 것이 lazy
  - 스레드 세이프하다.

```kotlin
class Person(val name: String) {
    val emails by lazy { loadEmails(this) }
}
```



### 7.5.3 위임 프로퍼티 구현

- set이 복잡한 사용 예시를 들어보자.
- set시 나를 listen하는 객체에게 message를 전달해야 하는 상황을 가정하자.
  - 옵저버 패턴을 구현하기 위해서는 listener 목록, 메세지 발송 함수가 필요하다.
  - 이를 위해 자바 API로 PropertyChangeSupport 클래스를 사용한다.
- PropertyChangeAware 클래스
  - 앞으로 들 예시 클래스에 매번 PropertyChangeSupport를 필드를 다 추가하긴 싫으니
  - PropertyChangeSupport 객체를 감싸는 프록시 객체를 만들어서 super Class로 만들어 상속받자
  - 그냥 리스너 추가 삭제 해주는 클래스이다.
- 복잡한 get / set을 직접 구현하는 예제

```kotlin
class Person(val name:String, age:Int, salary:Int):PropertyAware(){
  var age: Int = age
    set(newValue){
      val old = field
      field = new
      changeSupport.firePropertyChange("age", old, new)
    }
  var salary: Int 
  // set이 위와 같은 로직
}
val p = Person("Dmitry", 20, 2000)
p.addPropertyChangeListener(PropertyChangeListener{evnet -> ,,,})
```



- set의 로직이 프로퍼티마다 중복된다.
- 로직을 객체로 캡슐화 해보자
- 로직을 캡슐화하는 ObservableProperty 객체를 선언해서 뒷받침 필드를 이 인스턴스로 만들고
- 기존 get/set은 ObservableProperty 의 get set 을 호출하는 식으로 바꿔보자.



```kotlin
class ObservableProperty(
	val propName: String,
  val propValue: Int,
  val changeSupport: PropertyChangeSupport
){
  fun getValue() = propValue
  fun setValue(newValue :Int){
    val old = propValue
    propValue = newValue
    changeSupport.firePropertyChange(propName, oldValue, newValue)
  }
}

class Person(
	val name: String, age: Int, salary: Int
): PropertyChangeAware(){
  val _age = ObservableProperty("age", age, changeSupport)
  var age: Int
    get() = _age.getValue()
    set(value) {_age.setValue(value)}
}
```



- by 키워드랑 같이 쓰기 위해서 ObsevablleProperty를 관례에 맞게 만들어보자



```kotlin
class ObservableProperty(
	//val propName: String, 삭제
  val propValue: Int,
  val changeSupport: PropertyChangeSupport
){
  operator fun getValue(p:Person, prop: KProperty<*>) = propValue
  operator fun setValue(p:Person, prop: KProperty<*>,newValue :Int){
    val old = propValue
    propValue = newValue
    changeSupport.firePropertyChange(prop.name, oldValue, newValue)
  }
}
```

- getValue와 setValue는 관례에 따르는 함수 이름이다.
- operator 키워드 + 관례 함수 이름 으로 by 키워드랑 같이 쓸 수 있다.
- 즉 by 키워드에 저 객체 넣어주면 프로퍼티의 get/set 시 알아서 저 관례 함수가 호출된다.
- 프로퍼티가 포함된 객체와 프로퍼티를 인자로 받는다. -> propName 사라짐
- person을 다시 써보면

```kotlin
class Person(
	val name: String, age: Int, salary: Int
): PropertyChangeAware(){
  val age: Int by ObservableProperty(age, changeSupport)
  val salary: Int by ObservableProperty(salary, changeSupport)
}
```



- 이렇게 observable은 많이 사용되므로 Delegates.observable(age, lambda)로 이미 구현되어 있다.



### 7.5.4 위임 프로퍼티 컴파일 규칙

- \<delegate\> 와 같은 걸로 편환된다고 한다.



### 7.5.5 프로퍼티 값을 맵에 저장

- 위임 프로퍼티를 사용하면 프로퍼티를 동적으로 만들 수 있다?!?!?!

```kotlin
class Person {
    private val _attribute = hashMapOf<String, String>()
    fun setAttribute(attrName: String, value: String) {
        _attributes[attrName] = value
    }
    //필수 정보는 프로퍼티로 만들어둔다.
    val name: String by _attribute
}
```

- Map은 getValue와 setValue가 구현되어 있다. 따라서 자연스럽게 by로 사용할 수 있는 것!
- 필수 정보는 API로 만들어서 제공하고
- 추가적인 정보는 역직렬화로 얻어올 수 있다.



