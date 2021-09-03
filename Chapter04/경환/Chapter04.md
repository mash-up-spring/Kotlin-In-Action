# 4. 클래스, 객체, 인터페이스



- 코틀린의 클래스와 인페이스는 자바와는 약간 다르다.
- 인터페이스도 프로퍼티를 가질 수 있으며, 기본적으로 final과 public이다.
- 중첩 클래스는 외부 참조를 가지지 않는다.
- 롬복 @Data를 네이티브하게 구현할 수 있다
- object는 클래스와 인스턴스를 선언과 동시에 생성하는 키워드이다.



## 4.1 클래스 계층 정의



### 4.1.1 인터페이스

- 코틀린 인터페이스는 자바 인터페이스와 비슷하다.

- 자바8처럼 구현체가 있는 메서드를 가질 수 있다.

- ```kotlin
  interface Clickable{
  	fun click()
    fun showOff() = println("I`m clickable!")
  }
  interface Focusable{
    fun showOff() = println("I`m Focusable!")
  }
  
  class Button : Clickable, Focusable{
    override fun click() = println("I was Clicked")
    //default 메서드가 여러개 있는경우 반드시 정의 필요하다. (ambiguous 문제)
    ovveride fun showOff() {
      super<Clickable>.showOff()
      super<Focusable>.showOff()
    }
  }
  ```

- 자바에서의 extend 와 implements 모두 콜론(:) 으로 대체한다.

- 다중 상속 불가, 다중 인터페이스 가능

- override 변경자는 필수
- 자바에서 코틀린 인터페이스를 쓴다면 디폴트 메서드의 구현 부분은 없는 셈 쳐야한다.(java6과 호환되므로)



### 4.1.2 open, final, abstract 변경자

- 모든 하위 클래스는 상속받은 기반(base) 클래스에 의존하므로, 변경에 취약하다.

- 그러므로 코틀린은 기본적으로 final 이다. non-final하게 만들고 싶다면 open 키워드를 사용한다.

- ```kotlin
  open class RichButton : Clickable{ // 다른 클래스가 RichButton을 상속할 수 있다.
  	fun disable(){} // final
  	open fun animate() // non final
  	override fun click() // override는 non final
    final override fun click() // open 상태인 override를 final 로 막을 수 있다.
  }
  ```

- 기본으로 final 을 쓰면 스마트캐스트를 사용할 수 있다.

-  추상클래스, 추상 멤버는 당연히 상속받아서 사용해야하기 때문에 open을 명시할 필요 없다.

- 추상클래스에 속했더라도 비추상함수는 기본적으로 final이다.(open 사용 가능)



### 4.1.3 가시성 변경자: 기본적으로 public

- 아무 변경자도 없는 경우 public

- 코틀린의 가시성 변경자(visibility modifier)는 패키지와는 무관하다.

- internal : 같은 모듈 안에서만 볼 수 있다

- protected: 하위 클래스에서만 볼 수 있다. (클래스 의존적이므로 최상위 선언은 불가)

- private: 같은 클래스에서만 볼 수 있거나 같은 파일 안에서만 볼 수 있다.

- ```kotlin
  internal open class TalkativeButton : Focusable{
    private fun yell() = ...
    protected fun whisper() = ...
  }
  
  fun TalkativeButton.giveSpeech(){// 오류: giveSpeech()는 public이나 TalkativeButton은 internal임
    yell() // private 접근 오류
    whisper() // protected 접근 오류
  }
  ```

- 확장함수는 protected에 접근할 수 없다는 것을 유의하자.



### 4.1.4 중첩 클래스는 기본적으로 static

- nested 클래스는 외부 참조가 없다.

- > 이펙티브 자바 Item24 멤버클래스는 static으로 만들어라
  >
  > 메모리문제, GC 문제 생길 수 있다.

- 바깥 참조를 포함시키고 싶다면 inner 변경자를 nested 클래스에 붙여야한다.
- `this@Outer`로 바깥 참조에 접근할 수 있다.



### 4.1.5 봉인된 클래스: 클래스 계층 확장 제한

- sealed 클래스는 상위 클래스를 상속한 하위 클래스 정의를 제한할 수 있다.

- ```kotlin
  sealed class Expr{
    class Num(val value : Int) : Expr() //기반 클래스의 모든 하위 클래스를 중첩 클래스로 나열한다.
    class Sum(val left: Expr, val right: Expr): Expr()
  }
  fun eval(e :Expr):Int = 
  	when(e){
      is Expr.Num -> e.value
      is Expr.Sum -> eval(e.right) + eval(e.left)
      //default 분기가 필요없다.
    }
  ```

- sealed 인터페이스는 자바와의 호환 문제로 만들 수 없다.

- sealed 클래스와 같은 파일의 아무데서나 sealed 클래스를 상속한 하위 클래스를 만들 수 있다.





## 4.2 생성자와 프로피티를 갖는 클래스



- 코틀린은 주 생성자와 부 생성자를 구분한다.
- 초기화 블록으로 초기화 로직을 추가할 수 있다.



### 4.2.1 클래스 초기화: 주 생성자와 초기화 블록

- 클래스 이름 뒤에 오는 괄호로 둘러싸인 코드를 주 생성자라고 부른다.

- 주 생성자는 파라미터를 지정하고 그 생성자에 의해 초기화되는 프로피터를 정의하는 두 목적으로 사용된다.

- ```kotlin
  class User constructor(_nickname: String){ // 파라미터 하나만 있는 주 생성자, 애노테이션이나 가시성 변경자 없다면 생략가능
  	val nickname = String
  	init {
  		nickname = _nickname // 초기화 블록
  	}
  }
  
  class User(val nickname: String,
             val isSubscribed: Boolean = true)
  
  val hyun = User("현석")
  val gye = User("계영", false)
  ```

- constructor: 주 생성자나 부 생성자 정의를 위해 사용한다.

- init: 인스턴스 생성시 실행되는 초기화 코드, 로직이 없는 주 생성자를 보완한다.

- 모든 생성자 파라미터에 디폴트 값을 지정하면 컴파일러가 자동으로 파라미터가 없는 생성자를 만들어준다.

- ```kotlin
  class TwitterUser(nickname : String) : User(nickname){...}
  open class Button // 인자가 없는 디폴트 생성자가 만들어진다.
  class RadioButton : Button(){} // 상속받으면서 디폴트 생성자 호출
  class Secretive private constructor(){} // 유일한 주 생성자는 private
  ```

- 주 생성자에서 기반 클래스의 생성자를 호출해야 한다.

- 상속 시 기반 클래스 뒤에는 괄호가 들어간다  (기반 클래스의 생성자) -> 인터페이스와 구별 가능



### 4.2.2 부 생성자 : 상위 클래스를 다른 방식으로 초기화

- ```kotlin
  class MyButton : View {
    constructor(ctx : Context)
      : this(ctx, MY_STYLE){
        ...
      }
    }
    constructor(ctx : Context, attr: AttributeSet)
      : super(ctx, attr){
        
    }
  }
  ```

- 클래스 바디 내부에서 constructor 키워드를 사용해서 주 생성자 없이 부생성자만 여러개 생성할 수 있다.

- this()를 이용해서 자신의 다른 생성자를 호출할 수 있다.
- 부생성자는 자바와의 상호운용성, 파라미터 목록이 다른 생성 방법이 필요할 때 사용한다.



### 4.2.3 인터페이스에 선언된 프로퍼티 구현

- ```kotlin
  interface User {
    val nickname: String
  }
  
  interface User2{
    val email: String // 반드시 구현해야 함
    val nickname: String
      get() = email.substringBefore('@') // 프로퍼티 뒷받침 필드가 없는 경우 게터와 세터가 있는 프로퍼티 선언 가능(상속가능)
  }
  
  class PrivateUser(override val nickname: String): User // 주 생성자에 있는 프로퍼티
  
  class SubscribingUser(val email: String): User{
    override val nickname: String
      get() = email.substringBefore('@') // 커스텀 게터(매번 계산)
  }
  
  class FacebookUser(val accountId: Int): User{
    override val nickname = getFacebookName(accountId) // 프로퍼티 초기화 식(한번만 계산)
  }
  ```

- User 인터페이스를 구현하는 클래스는 nickname의 값을 얻을 수 있는 방법을 제공해야한다.



### 4.2.4 게터와 세터에서 뒷받침하는 필드에 접근

- ```kotlin
  class User(val name: String) {
    var address: String = "unspecified"
     set(value: String){
       println(""":
         Address was changed for $name:
         "$field" -> $value".""".trimIndent())
       field = value
       )
     }
  }
  ```

- field라는 식별자를 이용해서 뒷받침 필드에 접근할 수 있다.

- 게터에서는 field 값을 읽을 수만 있고 세터에서는 읽거나 쓸 수 있다.

- field를 사용하지 않는 커스텀 접근자 구현을 정의하면 뒷받침하는 필드는 존재하지 않는다.(당연한 얘기)



### 4.2.5 접근자의 가시성 변경

- 접근자의 가시성은 기본적으로 프로퍼티의 가시성과 같다.

- get이나 set 앞에 변경자를 추가해서 가시성을 변경할 수 있다.

- ```kotlin
  class LengthCounter{
  	var counter: Int = 0
      private set // public getter는 컴파일러가 생성한다.
    fun addWord(word: String){
      counter += word.length
    }
  }
  ```

- lateinit 변경자 : 프로퍼티를 생성자가 호출된 다음 초기화한다.

- lazy initilized : 프로퍼티 초기화를 요청이 들어올 때까지 미룬다.





## 4.3 컴파일러가 생성한 메소드: 데이터 클래스와 클래스 위임

- Constructor나 getter/setter를 코틀린 컴파일러가 만들어준 것처럼 다른 필수 메소드도 컴파일러가 만들어준다.



### 4.3.1 모든 클래스가 정의해야 하는 메소드

- toString(), equals(), hashCode() 오버라이드 필요하다.
- 참고)JVM 언어 규칙으로 equals() true 이면 같은 hashCode()를 반환해야한다.
- 코틀린은 내부적으로 ==를 사용할 때 equals를 호출해서 객체를 비교한다.
- 참조 비교를 위해서는 ===를 사용한다.



### 4.3.2 데이터 클래스: 필요한 메서드 자동 완성 (＾▽＾) 

- ```kotlin
  data class Client(val name: String, val postalCode: Int)
  ```

- data 키워드를 클래스 앞에 붙여줌으로써 필요한 메서드를 기본적으로 만들어준다.

- 데이터 클래스는 불변 클래스가 권장된다.(다중 스레드, 해쉬맵 키로 사용 등)

- copy() 메서드: 객체를 복사하면서 일부 프로퍼티를 바꿀 수 있다.

- ```kotlin
  //copy를 직접 정의하는 경우
  fun copy(name: String = this.name,
          postalCode: Int = this.postalCode) = Client(name, postalCode)
  ```



### 4.3.3 클래스 위임: by 키워드 사용

- 상속을 우회하는 방법으로 데코레이터 패턴을 사용한다.

- ```kotlin
  //상속을 허용하지 않는 경우
  class Car{
    fun drive() = ...
  }
  
  class SuperCar(val car: Car){
    fun drive(){
      car.drive()
    }
    fun boost(){
      ...
    }
  }
  //상속할 수 있는 경우
  class DelegatingCollection<T>: Collection<T>{
    private val innerList = arrayListOf<T>()
    
    override fun size: Int get() = innerList.size()
    ...다른 모든 컬렉션 메서드들
  }
  ```

- 데코레이터를 만들 때 상위 클래스의 메서드를 구현해야하므로 코드가 번잡하다.

- 코틀린은 인터페이스를 구현할 때 by 키워드로 인터페이스에 대한 구현을 다른 객체에 위임 중이라는 것을 명시할 수 있다.

- ```kotlin
  class CountingSet<T>(
    innerSet: MutableCollection<T> = Hashset<T>()
  ): MutableCollection<T> by innerSet { // by 키워드로 위임을 명시한다.
    
    var objectsAdded = 0
    
    //특정 메서드를 오버라이드 할 경우
    //컴파일러가 생성한 메서드 대신 오버라이드 메서드가 쓰인다.
    override fun add(element: T): Boolean{
      objectsAdded++
      return innerSet.add(element)
    }
  }
  ```

- CountingSet 과 MutableCollection 구현 간 의존관계가 생기지 않는다.

- CountingSet은 오버라이드하더라도 Collection api 구현은 하지 않는다.



## 4.4 object 키워드: 클래스 선언과 인스턴스 생성

- object 키워드는 클래스를 정의하는 동시에 인스턴스를 생성한다.
- 싱글턴, 동반 객체(companion object), 무명 내부 클래스 대신 쓰인다.



### 4.4.1 객체 선언: 싱글턴을 쉽게 만들기

- 싱글턴을 객체 선언으로 구현할 수 있다.

- 객체 선언: 클래스 선언 + 그 클래스에 속한 단일 인스턴스 선언

- ```kotlin
  object Payroll{ // 객체 선언은 object 키워드로 시작한다.
    val allEmployees = arrayListOf<Person>()
    fun calculateSalary(){
      for(person in allEmployees){
        ...
      }
    }
  }
  ```

- 프로퍼티, 메소드, 초기화 블록을 사용할 수 있으나 생성자는 사용할 수 없다.

- 객체 선언도 클래스나 인터페이스를 상속할 수 있다.

- Stateless 객체 구현에 편리하다.

- ```kotlin
  object CaseInsensitiveFileComparator: Comparator<File> {
    override fun compare(file1: File, file2: File): Int{
      return file1.path.compareTo(file2.path, ignoreCase = true)
    }
  }
  
  >>println(CaseInsensitiveFileComparator.compare(file1, file2))
  ```

- 클래스 안에서 object를 선언할 수도 있다. (static한 특성이지, composite 관계가 아니다.)

- ```kotlin
  data class Person(val name: String){
    object NameComparator : Comparator<Person>{
      override fun compare(p1: Person, p2: Person): Int = 
       p1.name.compareTo(p2.name)
    }
  }
  ```



### 4.2.2 동반 객체: 팩토리 메소드와 정적 메소드가 들어갈 장소

- 코틀린은 static 대신 최상위 함수와 객체 선언을 사용한다.

- 클래스 안에 선언된 object 중 하나에 companion 이라는 키워드를 붙이면 그 객체는 동반 객체가 된다.

- 객체의 이름을 따로 지정할 필요 없이 클래스 이름을 사용한다 -> static이랑 같다.

- 따라서 static한 팩토리 대신 companion object 안에 팩토리 메서드를 선언하면 된다.

- ```kotlin
  class User private constructor(val nickname: String){
    companion object{
      fun newSubscribingUser(email: String) = User(email.substringBefore('@'))
      fun newFacebookUser(accountId: Int) = User(getFacebookName(accountId))
    }
  }
  ```

- 동반 객체 또한 일반 객체이다. 이름을 붙이거나, 상속 받거나, 동반 객체 안에 확장함수와 프로퍼티를 정의할 수 있다.

- 이름이 없을 경우 Companion이라는 이름으로 참조할 수 있다.

- 확장 static 메서드를 만들 때도 companion object를 쓰면 된다

- ```kotlin
  class Person(...){
    ...
  	companion object{
      //비어있는 companion object
    }
  }
  
  fun Person.Companion.fromJSON(json: String): Person{
    ...
  }
  
  val p = Person.fromJSON(json)
  ```



### 4.4.4 객체 식: 무명 내부 클래스를 다른 방식으로 작성

- 무명객체 : 특정 코드에서만 사용하는 상속받은 객체(ex. Runnable)

- ```kotlin
  window.addMouseListener{
    object: MouseAdapter(){
      override fun mouseClicked(e: MouseEvent){
        ...
      }
      override fun mouseEntered(e: MouseEvent){
        ...
      }
    }
  }
  
  val listener = object: MouseAdapter(){
      override fun mouseClicked(e: MouseEvent){
        ...
      }
      override fun mouseEntered(e: MouseEvent){
        ...
      }
    }
  ```

  

- 무명 객체는 싱글턴이 아니다.

- 로컬 변수가 effective final 일 필요는 없다.