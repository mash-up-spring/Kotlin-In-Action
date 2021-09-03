# 4. 클래스, 객체, 인터페이스
4장에서 다루는 내용
- 클래스와 인터페이스
- 뻔하지 않은 생성자와 프로퍼티
- 데이터 클래스
- 클래스 위임
- object 키워드 사용 

코틀린의 클래스와 인터페이스는 자바 클래스, 인터페이스와는 약간 다르다. 예를 들어 인터페이스에 프로퍼티 선언이 들어갈 수 있다. 자바와 달리 코틀린 선언은 기본적으로 final이며 public이다. 중첩 클래스는 기본적으로 내부 클래스가 아니다.

## 4.1 클래스 계층 정의

### 4.1.1 코틀린 인터페이스
코틀린 인터페이스는 자바 8 인터페이스와 비슷하다. 코틀린 인터페이스 안에는 추상 메소드뿐 아니라 구현이 있는 메소드도 정의할 수 있다. 다만 인터페이스에는 아무런 필드도 들어갈 수 없다.
```kotlin
interface Clickable {
    fun click()
}

class Button : Clickable {
    override fun click() = println(" I was clicked")
}
```
자바에서는 extends와 implements 키워드를 사용하지만 코틀린에서는 클래스 이름 뒤에 : 을 붙이고 인터페이스와 클래스 이름을 적는다.<br>
코틀린에서는 override 변경자를 꼭 사용해야 한다.

<br>

```kotlin
interface Clickable {
    fun showOff() = println(" ~~~") // 디폴트 구현이 있는 메소드.
}

interface Focusable {
    fun showOff() = println("~~~")
}

class Button : Clickable, Focusable {
    override fun showOff() {
        super<Clickable>.showOff()
        super<Focusable>.showOff()
    } // 상위 타입의 이름을 꺾쇠 괄호 사이에 넣어 super를 지정하면 어떤 상위 타입의 메소드를 사용할지 지정 가능
}
```

### 4.1.2 open, final, abstract 변경자: 기본적으로 final
상속을 이용하는 어떤 정확한 규칙을 제공해야 한다.(취약한 기반 클래스 문제) Effective Java에서는 상속을 위한 설계와 문서를 갖추거나, 그럴 수 없다면 상속을 금지하라 라고 한다. 이는 오버라이드하게 의도된 클래스와 메서드가 아니라면 모두 final로 만들라는 뜻이다. <br>
코틀린의 클래스와 메소드는 기본적으로 final이다. 클래스의 상속을 허용하려면 open, 오버라이드 하고싶은 메소드나 프로퍼티 앞에도 open을 붙여야 한다.
```kotlin
open class RichButton : Clickable {
    fun disable() {}
    open fun animate() {}
    override fun click() {} // 오버라이한 메소드는 기본적으로 열려있다.
    // override 한것을 상속 금지하려면 final을 앞에 명시적으로 붙여주자.
}
```

### 4.1.3 가시성 변경자: 기본적으로 공개
코틀린에서는 아무 변경자가 없는경우 선언은 모두 public이다.<br>
코틀린에는 패키지 전용(package-private)이 없다. 대안으로 internal이라는 새로운 가시성 변경자를 도입했다. (같은 모듈에서만 볼 수 있다.)

```kotlin
internal open class TalkativeButton : Focusable {
    private fun yell() = println("Hey!")
    protected fun whisper() = println("Let's talk!")
}

fun TalkativeButton.giveSpeech() { // public 멤버가 자신의 interanl 타입인 TalkativeButton을 노출함
    yell()
    whisper()
}
```
어떤 클래스의 기반 타입 목록에 들어있는 타입이나 제네릭 클래스의 타입 파라미터에 들어있는 타입의 가시성은 그 클래스 자신의 가시성과 같거나 더 높아야 하고, 메소드의 시그니처에 사용된 모든 타입의 가시성은 그 메소드의 가시성과 같거나 더 높아야 한다.

### 4.1.4 내부 클래스와 중첩된 클래스: 기본적으로 중첩 클래스
코틀린 중첩클래스는 명시적으로 요청하지 않는 한 바깥쪽 클래스 인스턴스에 대한 접근 권한이 없다.<br>

```Java
public class Button implements View {
    //
    public class ButtonState implements State {}
}
```
ButtonState를 직렬화 할 수 없는 문제가 발생!

```kotlin
class Button: View {
    //
    class ButtonState: State {} // 이 클래스는 자바의 정적 중첩 클래스와 대응한다.
}
```
코틀린 중첩 클래스에 아무런 변경자가 붙지 않으면 자바 static 중첩 클래스와 같다. 이를 내부 클래스로 변경해서 바깥쪽 클래스에 대한 참조를 포함하게 만들고 싶다면 inner 변경자를 붙여야 한다.<br>
코틀린에서 바깥쪽 클래스의 인스턴스를 가리키는 참조를 표기하는 방법은 this@Outer 라고 쓰면 된다.

### 4.1.5 봉인된 클래스: 클래스 계층 정의 시 계층 확장 제한
```kotlin
sealed Class Expr {
    class Num(): Expr()
    class Sum(): Expr()
}

fun eval: Int = 
    when(e) {
        is Expr.Num ->
        is Expr.Sum ->
    }
```
sealed 클래스를 통해 확장을 제한 할 수 있다.

## 4.2 뻔하지 않은 생성자와 프로퍼티를 갖는 클래스 선언
코틀린은 주 생성자와 부 생성자를 구분 한다. 또한 코틀린에서는 초기화 블록을 통해 초기화 로직을 추가할 수 있다.

### 4.2.1 클래스 초기화: 주 생성자와 초기화 블록
```kotlin
class User(val nickname: String) // 간단한 클래스 선언

// 위 클래스 선언을 풀어서 보면
class User constructor(_nickname: String) {
    val nickname: String
    init {
        nickname = _nickname
    }
}
```
init 키워드는 초기화 블록을 시작하며, 클래스 객체가 만들어질 때 실행될 코드가 들어간다.

```kotlin
class User(val nickname: String,
           val isSubscribed: Boolean = true) // 생성자 파라미터에 대한 디폴트 값을 제공 한다.
```

### 4.2.2 부 생성자: 상위 클래스를 다른 방식으로 초기화
일반적으로 코틀린에서 생성자가 여럿 있는 경우는 자바보다 훨씬 적다. 그래도 생성자가 여럿 필요한 경우가 가끔 있다.<br>
프레임워크 클래스를 확장해야 하는데 여러 가지 방법으로 인스턴스를 초기화할 수 있게 다양한 생성자를 지원해야 하는 경우다.

```kotlin
open class View {
    constructor(ctx: Context) {
        //
    }

    constructor(ctx: Context, attr: AttributeSet) {
        //
    }
}
```
위 클래스를 확장 하기 위해서는 super() 키워드를 통해 자신에 대응하는 상위 생성자를 호출 해야 한다.
```kotlin
class MyButton : View {
    constructor(ctx: Context): this(ctx, MY_STYLE) { // this() 를 위해 다른 생성자에 위임 가능!

    }
    constructor(ctx: Context, attr: AttributeSet) : super(ctx, attr) {

    }
}
```

### 4.2.3 인터페이스에 선언된 프로퍼티 구현
```kotlin
interface User {
    val nickname: String
}
```

```kotlin
class PrivateUser(override val nickname: String) : User
class SubscribingUser(val email: String) : User {
    override val nickname: String
        get() = email.substringBefore('@') // 호출 할 때 마다 매번 계산
}
class FacebookUser(val accountId: Int): User {
    override val nickname = getFacebookName(accountId) // 초기화될 때 계산 해서 저장
}
```

### 4.2.4 게터와 세터에서 뒷받침하는 필드에 접근
어떤 값을 저장하되 그 값을 변경하거나 읽을 때마다 정해진 로직을 실행하는 유형의 프로퍼티를 만드는 방법을 살펴보자.<br>
값을 저장하는 동시에 로직을 실행할 수 있게 하기 위해서는 접근자 안에서 프로퍼티를 뒷받침하는 필드에 접근할 수 있어야 한다.
```kotlin
class User(val name: String) {
    var address: String = "unspecified"
        set(value: String) {
            println("""
                Address was changed for $name:
                "$field" -> "$value".""".trimIndent()) // 뒷받침 필드 읽기
                field = value // 뒷받침 필드 변경
            )
        }
}
```

### 4.2.5 접근자의 가시성 변경
```kotlin
class LengthCounter {
    var counter: Int = 0
        private set      // 이 클래스 밖에서 이 프로퍼티의 값을 바꿀 수 없다.

    fun addWord(word: String) {
        counter += word.length
    }
}
```

## 4.3 컴파일러가 생성한 메소드: 데이터 클래스와 클래스 위임
자바 플랫폼에서는 클래스가 equals, hashCode, toString 등의 메소드를 구현해야 한다. 코틀린 컴파일러는 이런 메소드를 기계적으로 생성하는 작업을 보이지 않는곳에서 해준다.

### 4.3.1 모든 클래스가 정의해야 하는 메소드
**toString()**<br>
```kotlin
class Client(val name: String, val postalCode: Int) {
    override fun toString() = "Client(name=$name, postalCode=$postalCode)"
}
```

<br>

**equals()**<br>
코틀린에서는 객체 비교를 ==로 할 수 있다.<br>
```kotlin
class Client() {
    override fun equals(other: Any?): Boolean {
        // 구현
    }
}
```

### 4.3.2 데이터 클래스: 모든 클래스가 정의해야 하는 메소드 자동 생성
```kotlin
data class Client(val name: String, val postalCode: Int)
```
equals, hashCode, toString 메소드가 자동으로 생성됐다!<br>
data 클래스는 몇 가지 유용한 메소드를 더 생성 해준다.<br><br>

**copy() 메소드**<br>
주로 불변객체를 사용하므로 copy 메소드를 통해 복사 해 일부 프로퍼티를 바꿀 수 있게 해준다.

### 4.3.3 클래스 위임: by 키워드 사용
대규모 객체지향 시스템을 설계할 때 시스템을 취약하게 만드는 문제는 보통 구현 상속에 의해 발생한다. 하위 클래스가 상위 클래스의 메소드 중 일부를 오버라이드하면 하위 클래스는 상위 클래스의 세부 구현 사항에 의존하게 된다.<br>
코틀린을 설계하며 이런 문제를 인식하고 기본적으로 클래스를 final로 취급하기로 결정 했다.<br>
하지만 종종 상속을 허용하지 않는 클래스에 새로운 동작을 추가해야 할 때가 있다. 이럴 때 사용하는 일반적인 방법이 데코레이터 패턴이다.
```kotlin
class DelegatingCollection<T>: Collection<T> {
    private val innerList = arrayListOf<T>()
    override val size: Int get() = innerList.size
    override fun isEmpty(): Boolean = innerList.isEmpty()
    ...
}
```

```kotlin
class DelegatingCollection<T>(
    innerList: Collection<T> = ArrayList<T>()
) : Collection<T> by innerList ()
```

## 4.4 object 키워드 : 클래스 선언과 인스턴스 생성
- 객체 선언(object declaration)은 싱글턴을 정의하는 방법 중 하나다.
- 동반 객체(companion object)는 인스턴스 메소드는 아니지만 어떤 클래스와 관련 있는 메소드와 팩토리 메소드를 담을 때 쓰인다.
- 객체 식은 자바의 무명 내부 클래스 대신 쓰인다.

### 4.4.1 객체 선언: 싱글턴을 쉽게 만들기
코틀린은 객체 선언 기능을 통해 싱글턴을 언어에서 기본 지원한다.
```kotlin
object Payroll {
    val allEmployees = arrayListOf<Person>()
    fun calculateSalary() {
        for (person in allEmployees) {
            ...
        }
    }
}
```
object 키워드를 사용 하면 프로퍼티, 메소드, 초기화 블록 등이 들어갈 수 있지만 생성자는 쓸 수 없다. 일반 클래스 인스턴스와 달리 싱글턴 객체는 객체 선언문이 있는 위치에서 생성자 호출 없이 즉시 만들어진다.
<br>
Comparator 구현하기 예제
```kotlin
object CaseInsensitiveFileComparator : Comparator<File> {
    override fun compare(file1: File, file2: File): Int {
        return file1.path.compareTo(file2.path, ignoreCase = true)
    }
}
>> CaseInsensitiveFileComparator.compare(file1, file2)
```

### 4.4.2 동반 객체: 팩토리 메소드와 정적 멤버가 들어갈 장소
코틀린 클래스 안에는 정적인 멤버가 없다. 코틀린 언어는 자바 static 키워드를 지원 하지 않는다. 그 대신 코틀린에서는 패키지 수준의 최상위 함수와 객체 선언을 활용한다.<br>
최상위 함수는 private으로 표시된 클래스 비공개 멤버에 접근할 수 없다. 그래서 클래스의 인스턴스와 관계 없이 호출해야 하지만, 클래스 내부 정보에 접근해야 하는 함수가 필요할 때는 클래스에 중첩된 객체 선언의 멤버 함수로 정의해야 한다.<br>

```kotlin
class A {
    companion object {
        fun bar() {
            println("Companion")
        }
    }
}
```

```kotlin
class User private constructor(val nickname: String) {
    companion object {
        fun newSubscribingUser(email: String) = 
            User(email.substringBefore('@'))
        fun newFacebookUser(accountId: Int) =
            User(getFacebookName(accountId))
    }
}
```

### 4.4.3 동반 객체를 일반 객체처럼 사용
동반 객체에 이름 붙이기
```kotlin
class Person(val name: String) {
    companion object Loader {
        fun fromJSON(jsonText: String) : Person = ...
    }
}

// 두 방법 모두 가능
person = Person.Loader.fromJSON("{name: 'garden'}")
person2 = Person.fromJSON()
```

<br>

인터페이스 구현
```kotlin
interface JSONFactory<T> {
    fun fromJSON(jsonText: String): T
}

class Person(val name: String) {
    companion object : JSONFactory<Person> {
        override fun fromJSON(jsonText: String): Person = ...
    }
}
```

<br>

동반 객체 확장
```kotlin
class Person(val firstName: String, val lastName: String) {
    companion object {
        // 비어있는 동반 객체를 선언 한다.
    }
}

fun Person.Companion.fromJSON(json: String): Person { // 확장 함수를 선언한다.
    ...
}
```

### 4.4.4 객체 식: 무명 내부 클래스를 다른 방식으로 작성
```kotlin
window.addMouseListener(
    object : MouseAdapter() { // MouseAdapter를 확장하는 무명 객체를 선언한다.
        override fun mouseClicked(e: MouseEvent) {

        }
        override fun mouseEntered(e: MouseEvent) {

        }
    }
)
```
<br>
무명 객체 안에서 로컬 변수 사용하기

```kotlin
fun countClick(window: Window) {
    var clickCount = 0

    window.addMouseListener(object: MouseAdapter() {
        override fun mouseClicked(e: MouseEvent) {
            clickCount++
        }
    })
}
```

## 4.5 요약
- 코틀린 인터페이스는 디폴트 구현을 포함할 수 있다.
- 모든 선언은 기본적으로 final이며 public이다.
- 선언이 final이 되지 않게 만들려면 앞에 open을 붙여야 한다.
- 중첩 클래스는 기본적으로 내부 클래스가 아니다. 바깥쪽 클래스에 대한 참조를 중첩 클래스 안에 포함시키려면 inner 키워드를 중첩 클래스 선언 앞에 붙여서 내부 클래스로 만들어야 한다.
- sealed 클래스
- 초기화 블록과 부 생성자 활용
- field 식별자를 통해 프로퍼티 접근자 안에서 데이터를 저장하는데 쓰이는 뒷받침 필드를 참조할 수 있다.
- 데이터 클래스
- 객체 선언 / object 키워드 / 싱글턴
- companion은 자바의 정적 메소드와 필드 정의를 대신한다.
- companion도 인터페이스를 구현할 수 있다. 확장도 가능 하다.