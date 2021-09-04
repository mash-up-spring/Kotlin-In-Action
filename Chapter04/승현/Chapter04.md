---
title: "Kotlin in Action 4장; 클래스, 객체, 인터페이스"
categories:
  - Language
tags:
  - kotlin
toc: true
toc_label: "Contents"
toc_sticky: true
---

### 4. 클래스, 객체, 인터페이스

* 클래스와 인터페이스
* 뻔하지 않은 생성자와 프로퍼티
* 데이터 클래스
* 클래스 위임
* `object` 키워드 사용



#### 4.1 클래스 계층 정의

##### 코틀린 인터페이스

* 자바 8 인터페이스와 비슷하다. 추상 메소드뿐 아니라 구현이 있는 메소드도 정의할 수 있다.
  * 다만 아무런 상태(필드)도 들어갈 수 없다.

```kotlin
interface Clickable {
  fun click()
}
```

* 인터페이스 구현은 아래와 같다.

```kotlin
class Button: Clickable {
  override fun click() = println("i was clicked")
}
```

* `extends` 나 `implements` 같은 키워드 없이 클래스 이름 뒤에 `:`을 붙이고 인터페이스와 클래스 이름을 적는 것으로 클래스 확장과 인터페이스 구현을 처리한다.
  * 인터페이스는 원하는 만큼 구현할 수 있고, 클래스는 하나만 확장할 수 있다.
* `override` 변경자는 상위 클래스나 상위 인터페이스에 있는 프로퍼티가 메소드를 오버라이드한다는 표시다.
  * **꼭** 사용해야 한다.
  * 상위 클래스에 있는 메소드와 시그니처가 같은 메소드를 우연히 하위 클래스에서 선언하는 경우 컴파일이 안 되기 때문에 `override` 를 붙이거나 메소드 이름을 바꿔야만 한다.
* 디폴트 구현은 별도의 키워드 없이 메소드 본문을 추가하면 된다.

```kotlin
interface Clickable {
  fun click()
  fun showOff() = println("i'm clickable")
}
```

* 같은 메소드를 정의하는 다른 인터페이스가 있다고 해 보자.

```kotlin
interface Focusable {
  fun setFocus(b: Boolean) = println("I ${if (b) "got" else "lost"} focus")
  fun showOff() = println("i'm focusable")
}
```

* 한 클래스에서 위 두 인터페이스를 함께 구현하는 경우 둘 중 어느 쪽의 `showOff` 도 선택되지 않는다. 구현을 대체할 오버라이딩 메소드를 직접 제공하지 않으면 컴파일러 오류가 발생한다. 
  * 코틀린 컴파일러는 두 메소드를 아우르는 구현을 하위 클래스에 직접 구현하게 강제한다.

```kotlin
class Button : Clickable, Focusable {
  override fun click() = println("i was clicked")
  override fun showOff() {
    super<Clickable>.showOff()
    super<Focusable>.showOff()
  }
}
```

* 상위 타입의 구현을 호출할 때는 `super` 을 사용하되 꺾쇠 괄호 안에 기반 타입 이름을 지정한다.

> 코틀린은 자바 6과 호환되게 설게되었으므로 인터페이스의 디폴트 메소드를 지원하지 않는다. 따라서 코틀린은 디폴트 메소드가 있는 인터페이스를 일반 인터페이스와 디폴트 메소드 구현이 정적 메소드로 들어 있는 클래스를 조합해 구현한다. 인터페이스에는 메소드 선언만 들어가며, 인터페이스와 함께 생성되는 클래스에는 모든 디폴트 메소드 구현이 정적 메소드로 들어간다. 그러므로 디폴트 인터페이스가 포함된 코틀린 인터페이스를 자바 클래스에서 상속해 구현하고 싶다면 코틀린에서 메소드 본문을 제공하는 메소드를 포함하는 모든 메소드에 대한 본문을 작성해야 한다.

##### open, final, abstract 변경자: 기본적으로 final

* 자바에서는 `final`로 명시적으로 상속을 금지하지 않는 모든 클래스를 다른 클래스가 상속할 수 있는데, 클래스에서 자신을 상속하는 방법에 대해 정확한 규칙을 제공하지 않으면 기반 클래스를 작성한 사람의 의도와 다른 방식으로 메소드를 오버라이드할 위험이 있다.
* 코틀린은 상속을 위한 설계와 문서를 갖추거나 그럴 수 없다면 상속을 금지하라는 철학을 따라 기본적으로 클래스와 메소드가 `final` 이다.
  * 상속을 허용하고자 하는 경우 `open` 변경자를 붙인다. 오버라이드를 허용하고 싶은 메소드나 프로퍼티의 앞에도 붙여야 한다.

```kotlin
open class RichButton : Clickable {
  
  fun disable()
  
  open fun animate()
  
  override fun click() {}
}
```

* 기반 클래스나 인터페이스의 멤버를 오버라이드하는 경우 그 메소드는 기본적으로 열려 있다. 오버라이드하는 메소드의 구현을 하위 클래스에서 오버라이드하지 못하게 하려면 오버라이드하는 메소드 앞에 `final`을 명시해야 한다.

```kotlin
open class RichButton : Clickable {
  final override fun click() {}
}
```

> 스마트 캐스트는 타입 검사 뒤에 변경될 수 없는 변수에만 적용 가능한데, 그러므로 클래스 프로퍼티는 `val` 이면서 커스텀 접근하가 없는 경우에만 스마트 캐스트가 가능해진다. 이는 프로퍼티가 `final` 이어야만 한다는 뜻이기도 하다. 프로퍼티는 기본적으로 `final`이기 때문에 따로 고민할 필요 없이 대부분의 프로퍼티를 스마트 캐스트에 활용 가능하다.

* 코틀린도 `abstract`으로 클래스를 선언할 수 있다. 이렇게 선언한 추상 클래스는 인스턴스화할 수 없다.
  * 추상 클래스에는 구현이 없는 추상 멤버가 있기 때문에 하위 클래스에서 추상 멤버를 오버라이드해야만 하는 게 보통이다. 추상 멤버는 항상 열려 있다. 따라서 `open`을 명시할 필요가 없다.

```kotlin
abstract class Animated {
  
  abstract fun animate()
  
  open fun stopAnimating() { // 추상 클래스에 속했더라도 비추상 함수는 기본적으로 파이널이지만 원한다면 open으로 오버라이드 허용 가능
  }
  
  fun animateTwice()
}
```

* 아래는 코틀린의 상속 제어 변경자이다. 
  * 클래스 멤버에 대해 적용 가능하다. 인터페이스 멤버는 `override` 외에 사용하지 않는다.

| 변경자   | 이 변경자가 붙은 멤버는?                               | 설명                                                         |
| -------- | ------------------------------------------------------ | ------------------------------------------------------------ |
| final    | 오버라이드할 수 없음                                   | 클래스 멤버의 기본 변경자                                    |
| open     | 오버라이드할 수 있음                                   | 반드시 명시해야 오버라이드 가능                              |
| abstract | 반드시 오버라이드해야 함                               | 추상 클래스의 멤버에만 이 변경자를 붙일 수 있으며 추상 멤버에는 구현이 있으면 X |
| override | 상위 클래스나 상위 인스턴스의 멤버를 오버라이드하는 중 | 오버라이드하는 멤버는 기본적으로 열려 있으며 하위 클래스의 오버라이드를 금지하려면 final 명시 |

##### 가시성 변경자: 기본적으로 공개

* 코드 기반에 있는 선언에 대한 클래스 외부 접근을 제어한다. 
* 기본적으로 코틀린 가시성 변경자는 자바와 비슷하다. 아무 변경자도 없는 경우 선언은 모두 공개된다.
* 자바의 기본 가시성인 패키지 전용은 코틀린에 없다. 코틀린은 패키지를 네임스페이스를 고나리하기 위한 용도로만 사용한다.
  * 패키지 전용 가시성은 `internal`이라는 새로운 가시성 변경자를 사용한다. 모듈(한 번에 한꺼번에 컴파일되는 코틀린 파일들) 내부에서만 볼 수 있다는 의미이다.
* 코틀린에서는 최상위 선언에 대해 `private` 가시성을 허용한다. 여기에는 클래스, 함수, 프로퍼티 등이 포함된다. 선언이 들어 있는 파일 내부에서만 사용할 수 있다.

| 변경자       | 클래스 멤버                      | 최상위 선언                    |
| ------------ | -------------------------------- | ------------------------------ |
| public(기본) | 모든 곳에서 볼 수 있다.          | 모든 곳에서 볼 수 있다.        |
| internal     | 같은 모듈 안에서만 볼 수 있다.   | 같은 모듈 안에서만 볼 수 있다. |
| protected    | 하위 클래스 안에서만 볼 수 있다. | (최상위 선언에 적용할 수 없음) |
| private      | 같은 클래스 안에서만 볼 수 있다. | 같은 파일 안에서만 볼 수 있다. |

> 예제

```kotlin
internal open class TalkativeButton : Focusable {
  private fun yell() = println("hey")
  protected fun whisper() = println("let's talk")
}

fun TalkativeButton.giveSpeech() { // public 멤버가 자신의 internal 수신 타입인 TalkativeButton을 노출하는 오류
  yell() // yell에 접근 불가: yell은 Talkative의 private 멤버
  
  whisper() // whisper에 접근 불가: Talkative의 protected 멤버
}
```

* 코틀린은 `public` 함수가 그보다 가시성이 낮은 타입을 참조하지 못하게 한다.
  * 어떤 클래스의 기반 타입 목록에 들어 있는 타입이나 제네릭 클래스의 타입 파라미터에 들어 있는 타입의 가시성은 그 클래스 자신의 가시성과 같거나 더 노파야 하고, 메소드의 시그니처에 사용된 모든 타입의 가시성은 그 메소드의 가시성과 같거나 더 높아야 하는 일반적인 규칙이다.
* 코틀린에서는 같은 패키지 안에서 `protect` 멤버에 접근할 수 없다는 점을 유의하자.
* 코틀린의 `public`, `protected`, `private` 변경자는 컴파일된 자바 바이트코드 안에서도 유지된다. 자바에서 똑같은 가시성을 사용해 선언한 경우와 같다. `private`의 경우 자바에서는 클래스를 `private`으로 만들 수 없으므로 내부적으로 코틀린은 `private` 클래스를 패키지 전용 클래스로 컴파일한다.
  * `internal`의 경우 자바에는 딱 맞는 가시성이 없어 바이트코드 상에는 `public`이 된다.
  * 둘의 차이로 인해 코틀린에서 접근할 수 없는 대상을 자바에서 접근할 수 있는 경우가 생긴다. 가령 다른 모듈에 정의된 `internal` 클래스나 `internal` 최상위 선언을 모듈 외부의 자바 코드에서 접근할 수 있다. 또한 코틀린에서 `protected`로 정의한 멤버를 코틀린 클래스와 같은 패키지에 속한 자바 코드에서는 접근할 수 있다.

##### 내부 클래스와 중첩된 클래스: 기본적으로 중첩 클래스

* 자바처럼 코틀린에서도 클래스 안에 다른 클래스를 선언할 수 있는데, 자바와의 차이는 코틀린의 중첩 클래스는 명시적으로 요청하지 않는 한 바깥쪽 클래스 인스턴스에 대한 접근 권한이 없다는 점이다.

```kotlin
interface State : Serializable

interface View {
  fun getCurrentState(): State
  fun restoreState(state: State) {}
}
```

```kotlin
/* 자바 */ 
public class Button implements View {
  @Override
  public State getCurrentState() {
    return new ButtonState();
  }
  
  @Override
  public void restoreState(State state) { /* ... */ }
  
  public class ButtonState implements State { /* ... */ }
}
```

* 위 예제에서 `State` 인터페이스를 구현한 `ButtonState` 클래스를 정의해서 `Button`에 대한 구체적인 정보를 저장한다. `getCurrentState` 메소드 안에서는 `ButtonState` 의 새 인스턴스를 만든다. 실제로는 `ButtonState` 안에 필요한 모든 정보를 추가해야 한다.
  * 선언한 버튼의 상태를 직렬화하면 오류가 발생하는데, 왜냐하면 자바에서 다른 클래스 안에 정의한 클래스는 자동으로 내부 클래스가 되기 때문이다. 이 예제의 `ButtonState` 클래스는 바깥쪽 클래스에 대한 참조를 묵시적으로 포함하므로 그 참조로 인해 `ButtonState`를 직렬화할 수 없다. `Button` 을 직렬화할 수 없으므로 버튼에 대한 참조가 `ButtonState`의 직렬화를 방해한다.
  * 이 문제를 해결하려면 버튼의 상태를 `Static` 클래스로 선언해야 한다. 정적 클래스로 선언하면 바깥쪽 클래스에 대한 묵시적 참조가 사라진다.
* 코틀린에서 중첩된 클래스가 기본적으로 동작하는 방식은 위와 반대이다.

```kotlin
class Button : View {
  override fun getCurrentState(): State = ButtonState()
  
  override fun restoreState(state: State) { /* ... */ }
  
  class ButtonState : State { /* ... */ } // 이 클래스는 자바의 중첩 클래스와 대응한다.
}
```

* 코틀린 중첩 클래스에 아무런 변경자가 붙지 않으면 자바 `static` 중첩 클래스와 같다. 
  * 내부 클래스로 변경해서 바깥쪽 클래스에 대한 참조를 포함하게 만들고 싶다면 `inner` 변경자를 붙여야 한다.

| 클래스 B 안에 정의된 클래스 A                          | 자바에서는     | 코틀린에서는  |
| ------------------------------------------------------ | -------------- | ------------- |
| 중첩 클래스(바깥쪽 클래스에 대한 참조를 저장하지 않음) | static class A | class A       |
| 내부 클래스(바깥쪽 클래스에 대한 참조를 저장함)        | class A        | inner class A |

* 코틀린의 내부 클래스 안에서 바깥쪽 클래스의 참조에 접근하려면 `this@<OuterClassName>`라고 써야 한다.

```kotlin
class Outer {
  inner class Inner {
    fun getOuterReference(): Outre = this@Outer
  }
}
```

##### 봉인된 클래스: 클래스 계층 정의 시 계층 확장 제한

* 코틀린 컴파일러는 `when` 사용 시 디폴트 분기를 덧붙이게 강제한다. 여러 하위 클래스를 분기 처리 할 때 디폴트 분기를 추가하는 게 편하지는 않다.
  * 디폴트 분기가 있으면 클래스 계층에 새로운 하위 클래스를 추가하더라도 `when`이 모든 경우를 처리하는지 제대로 검사할 수 없다. 새로운 클래스 처리를 잊었더라도 디폴트 분기가 선택되기 때문이다.
  * 코틀린에서는 이런 문제를 해결하기 위해 `sealed` 클래스를 제공한다.
* 상위 클래스에 `sealed` 변경자를 붙이면 그 상위 클래스를 상속한 하위 클래스 정의를 제한할 수 있다. 
  * 하위 클래스 정의 시 반드시 상위 클래스 안에 중첩시켜야 한다.

```kotlin
sealed class Expr {
  class Num(val value: Int) : Expr()
  class Sum(val left: Expr, val right: Expr) : Expr()
}

fun eval(e: Expr): Int =
    when(e) {
      is Expr.Num -> e.value
      is Expr.Sum -> eval(e.right) + eval(e.left)
    }
```

* `when` 식에서 `sealed` 클래스의 모든 하위 클래스를 처리한다면 디폴트 분기가 필요 없다.
  *  `sealed` 로 표시된 클래스는 자동으로 `open` 이므로 별도 변경자를 붙일 필요가 없다.
  * 나중에 `sealed` 클래스에 새로운 하위 클래스를 추가해도 `when` 식이 컴파일되지 않아 식을 고쳐야 한다는 사실을 알 수 있다.
* `sealed` 인터페이스를 정의할 수는 없다. 봉인된 인터페이스를 만들 수 있다면 그 인터페이스를 자바 쪽에서 구현하지 못하게 막을 수 있는 수단이 컴파일러에게 없기 때문이다.



#### 4.2 뻔하지 않은 생성자와 프로퍼티를 갖는 클래스 선언

* 코틀린은 주 생성자와 부 생성자를 구분하며 초기화 블록을 통해 초기화 로직을 추가할 수 있다.

##### 클래스 초기화: 주 생성자와 초기화 블록

* 클래스 이름 뒤에 오는 괄호로 둘러싸인 코드를 주 생성자라고 부른다.
  * 주 생성자는 생성자 파라미터를 지정하고 그 생성자 파라미터에 의해 초기화되는 프로퍼티를 정의하는 두 가지 목적에 쓰인다. 

```kotlin
class User constructor(_nickname: String) {
  val nickname: String
  init {
    nickname = _nickname
  }
}
```

* `constructor` 는 주 생성자나 부 생성자 정의를 시작할 때 사용한다.
* `init` 은 초기화 블록을 시작한다.
  * 객체가 만들어질 때 실행될 초기화 코드가 들어간다.
  * 초기화 블록은 주 생성자와 함께 사용된다. 주 생성자는 제한적이기 때문에 별도의 코드를 포함할 수 없으므로 초기화 블록이 필요하다.
* 생성자 파라미터에서 맨 앞의 밑줄은 프로퍼티와 생성자 파라미터를 구분해 준다.
  * 자바처럼 `this`를 사용할 수도 있다.
  * 위 예제에서는 `nickname` 프로퍼티를 초기화하는 코드를 `nickname` 프로퍼티 선언에 포함시킬 수 있어 초기화 코드를 초기화 블록에 넣을 필요가 없다.
  * 주 생성자 앞에 별다른 애노테이션이나 가시성 변경자가 없다면 `constructor`를 생략해도 된다.

```kotlin
class User(_nickname: String) {
  val nickname = _nickname
}
```

* 프로퍼티를 초기화하는 식이나 초기화 블록 안에서만 주 생성자의 파라미터를 참조할 수 있다.
* 주 생성자의 파라미터로 프로퍼티를 초기화한다면 주 생성자 파라미터 이름 앞에 `val`을 추가하는 방식으로 프로퍼티 정의와 초기화를 간략히 쓸 수 있다.

```kotlin
class User(val nickname: String,
           val isSubcribed: Boolean = true)
```

* 생성자 파라미터에도 디폴트 값을 정의할 수 있다.
* 인스턴스를 만들려면 생성자를 직접 호출하면 된다.

```kotlin
val hyun = User("승현")
val min = User("nim", false)
val won = User("money", isSubcribed = false)
```

> 모든 생성자 파라미터에 디폴트 값을 지정하면 컴파일러가 자동으로 파라미터가 없는 생성자를 만들어 준다. 자동으로 만들어진 파라미터 없는 생성자는 디폴트 값을 사용해 클래스를 초기화한다.

* 클래스에 기반 클래스가 있다면 주 생성자에서 기반 클래스의 생성자를 호출해야 할 필요가 있다.
  * 기반 클래스를 초기화하려면 기반 클래스 이름 뒤에 괄호를 치고 생성자 인자를 넘긴다.

```kotlin
open class User(val nickname: String) { ... }
class TwitterUser(nickname: String) : User(nickname) { ... }
```

* 클래스를 정의할 때 별도로 생성자를 정의하지 않으면 컴파일러가 자동으로 인자가 없는 디폴트 생성자를 만든다.

```kotlin
open class Button
```

* `Button` 클래스를 상속한 하위 클래스는 만드시 `Button` 클래스의 생성자를 호출해야 한다.
* 이로 인해 기반 클래스의 이름 뒤에는 꼭 빈 괄호가 들어간다.
  * 반면 인터페이스는 생성자가 없어 괄호를 붙일 필요가 없다.
* 어떤 클래스를 클래스 외부에서 인스턴스화하지 못하게 막고 싶다면 모든 생성자를 `private`으로 만들면 된다. 주 생성자에 `private`을 붙일 수 있다.

```kotlin
class Secretive private constructor() {}
```

##### 부 생성자: 상위 클래스를 다른 방식으로 초기화

* 코틀린은 디폴트 파라미터 값과 이름 붙인 인자 문법을 사용해 생성자의 오버로드가 필요한 일이 별로 없다.
* 그래도 생성자가 여럿 필요한 경우에는 아래와 같이 여러 생성자를 정의할 수 있다.

```kotlin
open class View {
  constructor(ctx: Context) {
    
  }
  constructor(ctx: Context, attr: AttributeSet) {
    
  }
}
```

* 부 생성자는 `constructor` 키워드로 시작한다. 필요에 따라 생성 가능하다.

```kotlin
class MyButton: View {
  constructor(ctx: Context)
    : super(ctx) {
      
  }
      
  constructor(ctx: Context, attr: AttributeSet)
    : super(ctx, attr) {
      
  }
}
```

* 여기서 두 부 생성자는 `super` 키워드로 자신에 대응하는 상위 클래스 생성자를 호출한다. 상위 클래스 생성자에게 객체 생성을 위임한다.

```kotlin
class MyButton: View {
  constructor(ctx: Context): this(ctx, MY_STYLE) {
    // ...
  }
  constructor(ctx: Context, attr: AttributeSet): super(ctx, attr) {
    // ...
  }
}
```

* 생성자에서 `this` 를 통해 자신의 다른 생성자를 호출할 수 있다.
* 클래스에 주 생성자가 없다면 모든 부 생서자는 반드시 상위 클래스를 초기화하거나 다른 생성자에게 생성을 위임해야 한다.
* 부 생성자는 자바와의 상호 운용성, 클래스 인스턴스 생성 시 파라미터 목록이 다른 생성 방법이 여럿 존재하는 경우 등을 위해 사용된다.

##### 인터페이스에 선언된 프로퍼티 구현

* 코틀린에서는 인터페이스에 추상 프로퍼티 선언을 넣을 수 있다.

```kotlin
interface User {
  val nickname: String
}
```

* 이런 경우 인터페이스를 구현하는 클래스가 추상 프로퍼티의 값을 얻을 수 있는 방법을 제공해야 한다.

```kotlin
class PrivateUser(override val nickname: String) : User

class SubscribingUser(val email: String) : User {
  override val nickname: String
     get() = email.substringBefore('@')
}

class FacebookUser(val accountId: Int) : User {
  override val nickcname = getFacebookName(accountId)
}
```

* 추상 프로퍼티를 구현하는 방식은 여러 가지가 있다.
  * 주 생성자 안에 프로퍼티를 직접 선언하는 간결한 구문을 사용한다. 인터페이스의 추상 프로퍼티를 구현하므로 `override`를 표시해야 한다.
  * 커스텀 게터로 설정한다. 위의 예에서는 뒷받침하는 필드에 값을 저장하지 않고 매번 별명을 계산해 반환한다.
  * 초기화 식으로 초기화한다.
* 인터페이스에는 추상 프로퍼티뿐 아니라 게터와 세터가 있는 프로퍼티를 선언할 수 있다.
  * 게터와 세터는 뒷받침하는 필드를 참조할 수 없다. 
  * 아래 예에서 게터가 있는 필드는 오버라이드하지 않고 상속할 수 있다.

```kotlin
interface User {
  val email: String
  val nickname: String
    get() = email.substringBefore('@')
}
```

* 클래스에 구현된 프로퍼티는 뒷받침하는 필드를 원하는 대로 사용할 수 있다.

##### 게터와 세터에서 뒷받침하는 필드에 접근

```kotlin
class User(val: name: String) {
  var address: String = "x"
    set(value: String) {
      println("""
     	  Address was changed:
     	  "$field" -> "$value".""".trimIndent())
      field = value
    }
}
```

* 코틀린에서 프로퍼티 값을 바꿀 때는 필드 설정 구문을 사용한다.
  * 이 구문은 내부적으로는 필드의 세터를 호출한다. 위 예제에서는 커스텀 세터를 정의해 추가 로직을 실행한다.
* 접근자의 본문에서는 `field`라는 특별한 식별자를 통해 뒷받침하는 필드에 접근할 수 있다. 게터에서는 `field` 값을 읽을 수만 있고 세터에서는 읽거나 쓸 수 있다.
* 뒷받침하는 필드가 있는 프로퍼티와 없는 프로퍼티의 차이는  뭘까?
  * 클래스의 프로퍼티를 사용하는 쪽에서 프로퍼티는 읽는 방법이나 쓰는 방법은 뒷받침하는 필드의 유무와는 관계가 없다.
  * 컴파일러는 게터나 세터에서 `field` 를 사용하는 프로퍼티에 대해 뒷받침하는 필드를 생성해 준다.
  * 다만 `field`를 사용하지 않는 커스텀 접근자 구현을 정의한다면 뒷받침하는 필드는 존재하지 않는다.

##### 접근자의 가시성 변경

* 기본적으로 접근자의 가시성은 프로퍼티의 가시성과 같다.
  * 변경자르르 추가해 변경 가능하다.

```kotlin
class LengthCounter {
  var counter: Int = 0
    private set
  fun addWord(word: String) {
    counter += word.length
  }
}
```

* 위 예처럼 세터의 가시성을 `private`으로 지정하면 클래스 밖에서 프로퍼티의 값을 바꿀 수 없다.



#### 4.3 컴파일러가 생성한 메소드: 데이터 클래스와 클래스 위임

* 코틀린 컴파일러는 `equals`, `hashCode` 같은 메소드를 기계적으로 생성하는 작업을 보이지 않는 곳에서 해 준다.

##### 모든 클래스가 정의해야 하는 메소드

###### 문자열 표현: toString()

* 기본 제공되는 표현을 유용하게 변경하려면 메소드를 오버라이드해야 한다.

````kotlin
class Client(val name: String, val postalCode: Int) {
  override fun toString() = "Client(name=$name, postalCode=$postalCode)"
}
````

###### 객체의 동등성: equals()

* 내용이 같은 경우 동등한 객체로 간주할 수 있게 한다.

  > 코틀린에서 `==` 연산자는 참조 동일성을 검사하지 않고 객체의 동등성을 검사한다. 따라서  `==` 연산은 `equals`를 호출하는 식으로 컴파일된다. `===` 연산자는 자바에서 객체의 참조를 비교할 때 사용하는 `==` 연산자와 같다.

```kotlin
class Client(val name: String, val postalCode: Int) {
  override fun equals(other: Any?): Boolean {
    if (other == null || other !is Client) return false
    return name == other.name && postalCode == other.postalCode
  }
}
```

* 코틀린의 `is` 는 자바의 `instanceof` 와 같다.
* 코틀린에서는 `override` 변경자가 필수여서 실수로 `override fun equals(other: Any?)` 대신 `override fun equals(other: Client)` 를 작성할 수는 없다.
* 위와 같이 변경하면 내용이 같은 경우 두 객체를 같은 것으로 취급하지만 더 복잡한 작업을 수행해 보면 제대로 작동하지 않는 경우가 있다.
  * `hashCode` 정의를 빠뜨려서 그렇다고 할 수 있지만 이 경우에는 실제 `hashCode`가 없다는 점이 원인이다.

###### 해시 컨테이너: hashCode()

* JVM 언어에서는 `hashCode`가 지켜야 하는 `equals`가 `true`를 반환하는 두 객체는 반드시 같은 `hashCode`를 반환해야 한다는 제약이 있다.
* `HashSet`에서 원소를 비교할 때 비용을 줄이기 위해 객체의 해시 코드를 비교하고 해시 코드가 같은 경우에만 실제 값을 비교한다. 원소 객체들이 해시 코드에 대한 규칙을 지키지 않으면 `HashSet`은 제대로 작동할 수 없다.

```kotlin
class Client(val name: String, val: postalCode: Int) {
  ...
  override fun hashCode(): Int = name.hashCode() * 31 + postalCode
}
```

##### 데이터 클래스: 모든 클래스가 정의해야 하는 메소드 자동 생성

* 어떤 클래스가 데이터를 저장하는 역할만을 수행해야 한다면 `toString`, `equals`, `hashCode`를 반드시 오버라이드해야 한다.
* 코틀린은 `data`라는 변경자를 클래스 앞에 붙이면 필요한 메소드를 컴파일러가 자동으로 생성해 준다. 이 변경자가 붙은 클래스를 데이터 클래스라고 한다.
* `equals`, `hashCode`는 주 생성자에 나열된 모든 프로퍼티를 고려해 만들어진다.
  * 프로퍼티의 동등성을 확인하고 모든 프로퍼티의 해시 값을 바탕으로 계산한 해시 값을 반환한다.
  * 주 생성자 밖에 정의된 프로퍼티는 고려 대상이 아니다.
* 코틀린 컴파일러는 데이터 클래스에 세 메소드뿐 아니라 몇 가지 유용한 메소드를 더 생성해 준다.

###### 데이터 클래스와 불변성: copy() 메소드

* 데이터 클래스의 프로퍼티가 꼭 `val`일 필요는 없지만 읽기 전용으로 만들어 데이터 클래스를 불변 클래슬 만드는 게 권장된다.
  * `HashMap` 등의 컨테이너에 데이터 클래스 객체를 담는 경우 불변성이 필수적이다. 객체를 키로 하는 경우 객체의 프로퍼티가 변경되면 컨테이너 상태가 잘못될 ㅜㅅ 있다.
* 불변 객체를 사용하면 프로그램에 대해 쉽게 추론할 수 있다.
* 데이터 클래스 인스턴스를 불변 객체로 더 쉽게 활용할 수 있게 코틀린 컴파일러는 편의 메소드를 제공한다.
  * 객체를 복사하면서 일부 프로퍼티를 바꿀 수 있게 해 주는 `copy` 메소드이다.
  * 객체를 직접 바꾸는 대신 복사본을 만들고 복사본은 원본과 다른 생명 주기를 가진다.
  * 프로퍼티 값을 바꾸거나 복사본을 제거해도 원본을 참조하는 부분에 영향을 끼치지 않는다.

###### 클래스 위임: by 키워드 사용

* 하위 클래스가 상위 클래스의 메소드 중 일부를 오버라이드하면 하위 클래스는 상위 클래스의 세부 구현 사항에 의존하게 된다.
  * 코틀린은 이런 문제를 인식하고 기본적으로 클래스를 `final`로 취급한다.
* 하지만 종종 상속을 허용하지 않는 클래스에 새로운 동작을 추가해야 할 때가 있다.
  * 이럴 때 사용하는 일반적인 방법이 데코레이터 패턴이다.
  * 상속을 허용하지 않는 클래스 대신 사용할 수 있는 클래스를 만들되 기존 클래스와 같은 인터페이스를 데코레이터가 제공하게 만들고, 기존 클래스를 데코레이더 내부에 필드로 유지한다.
  * 새로 정의해야 하는 기능은 데코레이터의 메소드에 새로 정의하고 기존 기능이 그대로 필요한 부분은 데코레이터의 메소드가 기존 클래스의 메소드에게 요청을 전달한다.
* 이 접근 방법의 문제는 준비 코드가 많이 필요하다는 것이다.
* 이런 위임을 코틀린은 언어가 제공하는 일급 시민 기능으로 지원한다.
* 인터페이스를 구현 시 `by` 키워드를 통해 그 인터페이스에 대한 구현을 다른 객체에 위임 중이라는 사실을 명시할 수 있다.

```kotlin
class DelegatingCollection<T> : Collection<T> {
  private val innerList = arrayListOf<T>()
  
  override val size: Int get() = innerList.size
  override fun isEmpty(): Boolean = innerList.isEmpty()
  // ...
}
```

```kotlin
class DelegatingCollection<T>(
	 innerList: Collection<T> = ArrayList<T>()
) : Collection<T> by innerList {}
```

* 클래스 안에 있던 메소드는 컴파일러가 자동으로 생성해 준다.
  * 동작을 변경하고 싶은 경우 메소드를 오버라이드하면 된다.

```kotlin
class CountingSet<T>(
	 val innerSet: MutableCollection<T> = HashSet<T>()
) : MutableCollection<T> by innerSet { // MutableCollection의 구현을 innerSet에게 위임한다.
  
  var objectsAdded = 0
  
  override fun add(element: T): Boolean {
    objectsAdded++
    return innerSet.add(element)
  }
  
  override fun addAll(c: Collection<T>): Boolean {
    objectsAdded += c.size
    return innerSet.addAll(c)
  }
}
```

* 이때 `CountingSet`에 `MutableCollection`의 구현 방식에 대핸 의존 관계가 생기지 않는다.
  * `CountingSet`의 코드를 호출할 때 발생하는 일은 `CountingSet` 안에서 제어할 수 있지만 `CountingSet` 코드는 위임 대상 내부 클래스인 `MutableCollection`의 문서화된 API를 활용한다. 그러므로 내부 클래스가 문서화된 API를 변경하지 않는 한 잘 작동할 것임을 확신할 수 있다.



#### 4.4 object 키워드: 클래스 선언과 인스턴스 생성

* `object` 키워드를 사용하는 여러 상황
  * 객체 선언은 싱글턴을 정의하는 방법 중 하나다.
  * 동반 객체는 인스턴스 메소드는 아니지만 어떤 클래스와 관련 있는 메소드와 팩토리 메소드를 담을 때 쓰인다.
  * 동반 객체 메소드에 접근할 때는 동반 객체가 포함된 클래스의 이름을 사용할 수 있다.
  * 객체 식은 자바의 무명 내부 클래스 대신 쓰인다.

##### 객체 선언: 싱글턴을 쉽게 만들기

* 코틀린은 객체 선언 기능을 통해 싱글턴을 언어에서 기본 지원한다.
* 객체 선언은 클래스 선언과 그 클래스에 속한 단일 인스턴스의 선언을 합친 선언이다.

```kotlin
object Payroll {
  val allEmployees = arrayListOf<Person>()
  
  fun calculateSalary() {
    for (person in allEmployess)
  }
}
```

* 객체 선언은 `object` 키워드로 시작한다. 클래스를 정의하고 그 클래스의 인스턴스를 만들어서 변수에 저장하는 모든 작업을 한 문장으로 처리한다.
  * 프로퍼티, 메소드, 초기화 블록 등이 들어갈 수 있다.
  * 생성자는 객체 선언에 쓸 수 없다. 싱글턴 객체는 객체 선언문이 있는 위치에서 생성자 호출 없이 즉시 만들어진다.
  * 객체 선언에 사용한 이름 뒤 마침표를 붙이면 객체에 속한 메소드나 프로퍼티에 접근할 수 있다.
* 객체 선언도 클래스나 인터페이스를 상속할 수 있다.

```kotlin
object CaseInsensitiveFileComparator : Comparator<File> {
  override fun compare(file1: file, file2: file): Int {
    return file1.path.compareTo(file2.path, ignoreCase = true)
  }
}
```

* 일반 객체(클래스 인스턴스)를 사용할 수 있는 곳에서는 항상 싱글턴 객체를 사용할 수 있다.
  * 가령, 위의 객체를 `Compartor`를 인자로 받는 함수에게ㅔ 인자로 넘길 수 있다.

> 의존 관계가 많지 않은 소규모 소프트웨어에서는 싱글턴이나 객체 선언이 유용하지만 시스템을 구현하는 다양한 구성 요소와 상호 작용하는 대규모 컴포넌트에는 싱글턴이 적합하지 않다. 객체 생성을 제어할 방법이 없고 생성자 파라미터를 지정할 수 없어서다. 시스템의 설정이 달라질 때 객체를 대체하거나 객체의 의존 관계를 바꿀 수 없으므로 그런 기능이 필요하다면 의존 관계 주입 프레임워크와 코틀린 클래스를 함께 사용해야 한다.

* 코틀린 객체 선언은 유일한 인스턴스에 대한 정적인 필드가 있는 자바 클래스로 컴파일된다. 이 인스턴스의 이름은 항상 `INSTANCE`다. 

##### 동반 객체: 팩토리 메소드와 정적 멤버가 들어갈 장소

* 코틀린 클래스 안에는 정적인 멤버가 없고 `static` 키워드를 지원하지 않는다.
* 대신 코틀린에서는 패키지 수준의 최상위 함수(정적 메소드 역할을 거의 대신)와 객체 선언(최상위 함수가 대신할 수 없는 역할이나 정적 필드를 대신함)을 활용한다.
  * 최상위 함수는 `private` 인 클래스 비공개 멤버에 접근할 수 없다. 그래서 클래스의 인스턴스와 관계없이 호출해야 하지만 클래스 내부 정보에 접근해야 하는 함수가 필요할 때는 클래스에 중첩된 객체 선언의 멤버 함수로 정의해야 한다. 대표적인 예로 팩토리 메소드를 들 수 있다.
* 클래스 안에 정의된 객체 중 하나에 `companion` 이라는 키워드를 붙이면 그 클래스의 동반 객체로 만들 수 있다.
  * 동반 객체의 프로퍼티나 메소드에 접근하려면 그 동반 객체가 정의된 클래스 이름을 사용한다.
  * 이때 객체의 이름을 따로 지정할 필요가 없다. 그 결과 동반 객체의 멤버를 사용하는 구문은 자바의 정적 메소드 호출이나 정적 필드 사용 구문과 같아진다.

```kotlin
class A {
  companion object {
    fun bar() {
      // ...
    }
  }
}

A.bar()
```

* 동반 객체는 자신을 둘러싼 클래스의 모든 `private` 멤버에 접근할 수 있다. 따라서 팩토리 패턴을 구현하기 가장 적합한 위치다.

```kotlin
class User {
  val nickname: String
  
  constructor(email: String) { // 부 생성자
    nickname = email.substringBefore('@')
  }
  
  constructor(facebookAccountId: Int) { // 부 생성자
    nickname = getFacebookName(facebookAccountId)
  }
}
```

* 이런 로직을 표현하는 더 유용한 방법으로 클래스의 인스턴스를 생성하는 팩토리 메소드가 있다.

```kotlin
class User private constructor(val nickname) { // 주 생성자를 비공개로 만든다.
  companion object { // 동반 객체를 선언한다.
    fun newSubscribingUser(email: String) = User(email.substringBefore('@'))
    fun newFacebookUser(accountId: Int) = User(getFacebookName(accountId))
  }
}
```

* 클래스 이름을 사용해 그 클래스에 속한 동반 객체의 메소드를 호출할 수 있다.
  * 팩토리 메소드는 그 팩토리 메소드가 선언된 클래스의 하위 클래스 객체를 반환할 수도 있다.
  * 하지만 클래스를 확장해야만 하는 경우에는 동반 객체 멤버를 하위 클래스에서 오버라이드할 수 없으므로 여러 생성자를 사용하는 것이 낫다.

##### 동반 객체를 일반 객체처럼 사용

* 동반 객체는 클래스 안에 정의된 일반 객체다. 따라서 동반 객체에 이름을 붙이거나 동반 객체가 인터페이스를 상속하거나 확장 함수와 프로퍼티를 정의할 수 있다.

```kotlin
class Person(val name: String) {
  companion object Loader { // 동반 객체에 이름을 붙인다.
    fun fromJSON(jsonText: String): Persion = ... 
  }
}
```

###### 동반 객체에서 인터페이스 구현

* 인터페이스를 구현하는 동반 객체를 참조할 때 객체를 둘러싼 클래스의 이름을 바로 사용할 수 있다.

```kotlin
interface JSONFactory<T> {
  fun fromJSON(jsonText: String): T
}

class Person(val name: String) {
  companion object : JSONFactory<Person> {
    override fun fromJSON(jsonText: String): Person = .. // 동반 객체가 인터페이스 구현
  }
}
```

* 위의 예에서 이제 JSON으로부터 각 원소를 다시 만들어내는 추상 팩토리가 있다면  `Person` 객체를 그 팩토리에게 넘길 수 있다.

````kotlin
fun loadFromJSON<T>(factory: JSONFactory<T>): T {
  ...
}

loadFromJSON(Person)
````

> 동반 객체는 일반 객체와 비슷한 방식으로 클래스에 정의된 인스턴스를 가리키는 정적 필드로 컴파일된다. 이름을 붙이지 않았다면 자바에서 `Companion` 이라는 이름으로 그 참조에 접근할 수 있다. 이름을 붙였다면 이름이 쓰인다.
>
> 자바에서 사용하기 위해 코틀린 클래스의 멤버를 정적인 멤버로 만들어야 할 필요가 있다면 `@JvmStatic` 어노테이션을 코틀린 멤버에 붙이면 된다. 정적 필드가 필요하다면 `@JvmField` 어노테이션을 최상위 프로퍼티나 객체에서 선언된 프로퍼티 앞에 붙인다. 
>
> 코틀린에서도 자바의 정적 필드가 메소드를 사용할 수 있다. 그런 경우 자바와 똑같은 구문을 사용한다.

###### 동반 객체 확장

* 클래스에 동반 객체가 있으면 그 객체 안에 함수를 정의함으로써 클래스에 대해 호출할 수 있는 확장 함수를 만들 수 있다.

```kotlin
class Person(val firstName: String, val lastName: String) {
  companion object {
    // 비어 있는 동반 객체를 선언한다.
  }
}

fun Person.Companion.fromJSON(json: String): Person { // 확장 함수를 선언한다.
  //...
}


val p = Person.fromJSON(json)
```

* 클래스 밖에서 정의한 확장 함수임에도 불구하고 동반 객체 안에서 `fromJSON` 함수를 정의한 것처럼 `fromJSON`을 호출할 수 있다.
  * 동반 객체에 대한 확장 함수를 작성할 수 있으려면 원래 클래스에 빈 동반 객체라도 동반 객체가 있어야 한다.

##### 객체 식: 무명 내부 클래스를 다른 방식으로 작성

* 무명 객체를 정의할 때도 `object` 키워드를 쓴다.
* 무명 객체는 자바의 무명 내부 클래스를 대신한다.

```kotlin
window.addMouseListener(
	  object : MouseAdapter() { // MouseAdapter를 확장하는 무명 객체를 선언한다.
      override fun mouseClicked(e: MouseEvent) {
        // ...
      }
      
      override fun mouseEntered(e: MouseEvent) {
        // ...
      }
    }
)
```

* 사용한 구문은 객체 선언과 같다. 한 가지 차이는 객체 이름이 빠졌다는 점이다.
* 객체 식은 클래스를 정의하고 그 클래스에 속한 인스턴스를 생성하지만 그 클래스나 인스턴스에 이름을 붙이지 않는다.
  * 보통 함수를 호출하면서 인자로 무명 객체를 넘기기 때문에 클래스와 인스턴스 모두 이름이 필요하지 않다.
  * 객체에 이름을 붙여야 한다면 변수에 무명 객체를 대입하면 된다.
* 코틀린 무명 클래스는 여러 인터페이스를 구현하거나 클래스를 확장하면서 인터페이스를 구현할 수 있다.

> 무명 객체는 싱글턴이 아니다.

* 객체 식 안의 코드는 그 식이 포함된 함수의 변수에 접근할 수 있다.
  * `final`이 닌 변수도 객체 식 안에서 사용할 수 있다.

```kotlin
/* 무명 객체 안에서 로컬 변수 사용하기 */
fun countClicks(window: Window) {
  var clickCount = 0
  window.addMouseListener(object : MouseAdapter) {
    override fun mouseClicked(e: MouseEvent) {
      clickCount++
    }
  }
}
```

> 객체 식은 무명 객체 안에서 여러 메소드를 오버라이드해야 하는 경우에 유용하다. 메소드가 하나뿐인 인터페이스를 구현해야 한다면 SAM 변환 지원을 활용하는 게 낫다.