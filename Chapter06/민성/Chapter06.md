# 6.1 널 가능성(nullability)

- 널 가능성은 NPE를 피할 수 있게 돕기 위한 코틀린 타입 시스템의 특성이다.
- 널이 될 수 있는지 여부를 타입 시스템에 추가함으로써 컴파일러가 여러 가지 오류를 컴파일 시 미리 감지해서 실행 시점에 발생할 수 있는 예외의 가능성을 줄일 수 있다.

## 6.1.1 널이 될 수 있는 타입

```kotlin
/* java */
int strLen(String s) {
	return s.length(); // NPE 가능성 존재
}

/* kotlin */
fun strLen(s: String) = s.length // s에 null 대입시 컴파일 오류

// null이 가능하게 하려면 (?) 명시
fun strLen(s: String?) = ...
```

- 어떤 타입이든 타입 이름 뒤에 물음표를 붙이면 그 타입의 변수나 프로퍼티에 null 참조를 저장할 수 있다는 뜻이다.
    - 물음표가 없는 타입은 그 변수가 null 참조를 저장할 수 없다는 뜻
- null이 가능한 타입인 변수에 대해 변수.메소드()처럼 메소드를 직접 호출할 수는 없다
    
    ```kotlin
    fun strLenSafe(s: String?) = s.length()
    ERROR: ...
    
    val x: String? = null
    val y: String = x 
    Error: ...
    ```
    
- null이 가능한 타입의 값은 null과 비교할 때 사용된다.
    - null과 비교하고 나면 컴파일러는 그 사실을 기억하고 null이 아님이 확실한 영역에서는 해당 값을 널이 될 수 없는 타입의 값처럼 사용할 수 있다.
    
    ```kotlin
    fun strLenSafe(s: String?) : Int = if (s != null) s.length else 0
    ```
    

## 6.1.2 타입의 의미

- 자바의 타입 시스템은 널을 제대로 다루지 못한다.
- 코틀린의 널이 될 수 있는 타입은 이런 문제에 대해 종합적인 해법을 제공한다.
    - 널이 될 수 있는 타입과 널이 될 수 없는 타입을 구분하면 각 타입의 값에 대해 어떤 연산이 가능할지 명확히 이해할 수 있고, 실행 시점에 예외를 발생시킬 수 있는 연산을 판단할 수 있다.
- 모든 검사는 컴파일 시점에 수행되어, 널이 가능한 타입을 처리하는 데 별도의 실행시점 부가 비용이 들지 않는다.

## 6.1.3 안전한 호출 연산자: ?.

?.은 null 검사와 메소드 호출을 한 번의 연산으로 수행한다.

- s?.toUpperCase()는 훨씬 더 복잡한 if (s != null) s.toUpperCase() else null과 같다.
- 메소드 호출뿐 아니라 프로퍼티를 읽거나 쓸 때도 안전한 호출을 사용할 수 있다.
- 안전한 호출 연쇄가 가능하다.
    
    ```kotlin
    val country = this.company?.address?.country // 여러 안전한 호출 연산자를 연쇄해 사용
    ```
    

## 6.1.4 엘비스 연산자: ?:

코틀린은 null 대신 사용할 디폴트 값을 지정할 때 편리하게 사용할 수 있는 엘비스 연산자를 제공한다.

```kotlin
fun foo(s: String?) {
	val t: String = s ?: ""
}
```

- 좌항 값이 null이 아니면 좌항 값을 결과로 하고, 좌항 값이 널이면 우항 값을 겨로가로 한다.
    
    ```kotlin
    fun strLenSafe(s: String?): Int = s?.length ?: 0
    
    >>> println(strLenSafe("abc"))
    3
    
    >>> println(strLenSafe(null))
    0
    ```
    
- 코틀린에서는 return이나 throw 등의 연산도 식이다. 따라서 엘비스 연산자의 우항에 return, throw 등의 연산을 넣을 수 있다.
    
    ```kotlin
    val address = person.company?.address
    	?: throw IllegalArgumentException("No address")
    with (address) { // 앞에서 검증했기 때문에 여기서 address는 null이 아니다.
    	println(...)
    }
    ```
    

## 6.1.5 안전한 캐스트: as?

as? 연산자는 어떤 값을 지정한 타입으로 캐스트한다. as?는 값을 대상 타입으로 변환할 수 없으면 null을 반환한다.

- foo as? Type
    - foo is Type → foo as Type
    - foo !is Type → null
- 안전한 연산자를 사용해 equals 구현
    
    ```kotlin
    override fun equals(o: Any?) : Boolean {
      // 타입이 서로 일치하지 않으면 flase
    	val otherPerson = o as? Person ?: return false
    
    	// 안전한 캐스트를 하고나면 otherPerson의 Person 타입으로 스마트캐스트된다.
    	val otherPerson.firstName == firstName && otherPerson.lastName == lastName
    		
    }
    ```
    
    - 파라미터로 받은 값이 원하는 타입인지 쉽게 검사하고 타입이 맞지 않으면 false를 반환할 수 있다.

## 6.1.6 널 아님 단언: !!(not-null assertion)

널 아님 단언은 느낌표를 이중(!!)으로 사용하면 어떤 값이든 널이 될 수 없는 타입으로 바꿀 수 없다.

- foo!!
    - foo ≠ null → foo
    - foo == null → NPE
- 예제)
    
    ```kotlin
    fun ignoreNulls(s: String?) {
    	val sNotNull: String = s!! // s가 null이면 NPE
    	println(sNotNull.length)
    }
    ```
    
- 근본적으로 !!는 컴파일러에게 "나는 이 값이 null이 아님을 잘 알고 있다. 내가 잘못 생각했다면 예외가 발생해도 감수하겠다"라고 말하는 것이다.
- 코틀린 설계자들은 컴파일러가 검증할 수 없는 단언을 사용하기보다는 더 나은 방법을 찾아보라는 의도를 넌지시 표현하려고 !!라는 못생긴 기호를 선택했다.
- 어떤 값이 널이었는지 확실히 하기 위해 여러 !! 단언문을 한 줄에 함께 쓰는 일을 피하라
    
    ```kotlin
    person.company!!.address!!.country // <- 이런 식으로 코드 작성 X
    ```
    

## 6.1.7 let 함수

- let 함수를 안전한 호출 연산자와 함께 사용하면 원하는 식을 평가해서 결과가 널인지 검사한 다음에 그 결과를 변수에 넣는 작업을 간단한 식을 사용해 한꺼번에 처리할 수 있다.
- let을 사용하는 가장 흔한 용례는 널이 될 수 있는 값을 널이 아닌 값만 인자로 받는 함수에 넘기는 경우다.
    
    ```kotlin
    fun sendEmailTo(email: String) { /*...*/ }
    
    if (email != null) sendEmailTo(email)
    ----------------------------------------------------------
    // 1. foo != null -> it은 람다 안에서 널이 아니다.
    // 2. foo == null -> 아무 일도 일어나지 않는다.
    foo?.let {
    	...it...
    }
    ----------------------------------------------------------
    email?.let { email -> sendEmilTo(email) }
    email?.let { sendEmailTo(it) }
    ```
    
- 아주 긴 식이 있고 그 값이 널이 아닐 때 수행해야 하는 로직이 있을 때 let을 쓰면 훨씬 더 편하다.
    
    ```kotlin
    val person: Person? = getTheBestPersonInTheWorld()
    if (person != null) sendEmailTo(person.email)
    
    getTheBestPersonInTheWorld()?.let { sendEmailTp(it.email) }
    ```
    
    - 위의 예제에서 getTheBestPersonInTheWorld()가 null일 수 있다
        - let을 중첩 처리할 수 있지만 그러면 코드가 복잡해져서 알아보기 어려워진다. 그런 경우 일반적으로 if를 사용해 모든 값을 한꺼번에 검사하는 편이 낫다.

## 6.1.8 나중에 초기화할 프로퍼티

객체 인스턴스를 일단 생성한 다음에 나중에 초기화하는 프레임워크가 많다.

- 하지만 코틀린에서 클래스 안의 널이 될 수 없는 프로퍼티를 생성자 안에서 초기화하지 않고 특별한 메소드 안에서 초기화할 수는 없다. 코틀린에서는 일반적으로 생성자에서 모든 프로퍼티를 초기화해야 한다.
    - 게다가 프로퍼티 타입이 널이 될 수 없는 타입이라면 반드시 널이 아닌 값을 프로퍼티를 초기화해야 한다.
    - 그런 초기화 값을 제공할 수 없으면 널이 될 수 있는 타입을 사용하는데, 이 경우 모든 프로퍼티 접근에 널 검사를 넣거나 !! 연산자를 사용해야 한다.
    
    ```kotlin
    class MyTest {
    	private var myService: MyService? = null
    	
    	@Before fun sertUp() {
    		myService = MyService()
    	}
    
    	@Test
    	fun testAction() {
    		Assert.assertEquals("foo", myService!!.performAction()) // 반드시 널 체크
    	}
    }
    
    // late-initialized(나중에 초기화) 
    class MyTest {
    	private lateinit var myService: MyService
    	
    	@Before fun sertUp() {
    		myService = MyService()
    	}
    
    	@Test
    	fun testAction() {
    		Assert.assertEquals("foo", myService.performAction()) // 널 체크 없이 가능
    	}
    }
    ```
    
    - 나중에 초기화하는 프로퍼티는 항상 var여야 한다.
    - 생성자 안에서 초기화하는 경우는 val가능

## 6.1.9 널이 될 수 있는 타입 확장

어떤 메소드를 호출하기 전에 수신 객체 역할을 하는 변수가 널이 될 수 없다고 보장하는 대신, 직접 변수에 대해 메소드를 호출해도 확장 함수인 메소드가 알아서 널을 처리해준다.

```kotlin
fun verifyUserInput(input: String?)) {
	if (input.isNullOrBlank()) { // 안전한 호출을 하지 않아도 된다.
		println("test")
	}
}

fun String?.isNullOrBlank(): Boolean = this == null || this.isBlank()
```

## 6.1.10 타입 파라미터의 널 가능성

- 코틀린에서는 함수나 클래스의 모든 타입 파라미터는 기본적으로 널이 될 수 있다. 널이 될 수 있는 타입을 포함하는 어떤 타입이라도 타입 파리미터를 대신할 수 있다. 따라서 타입 파라미터 T를 클래스나 함수 안에서 타입 이름으로 사용하면 이름 끝에 물음표가 없더라도 T가 널이 될 수 있는 타입이다.

```kotlin
fun <T> printHashCode(t: T) {
	println(t?.hashCode()) // t가 null이 될 수 있으므로 안전한 호출을 써야한다.
}

printHashCode(null) // t와 타입은 Any?로 추론된다.

fun <T: Any> printHashCode(t: T) {
	println(t.hashCode()) // 이제 t는 널이 될 수 없는 타입이다.
}
```

## 6.1.11 널 가능성과 자바

### 플랫폼 타입

플랫폼 타입은 코틀린이 널 관련 정보를 알 수 없는 타입을 말한다.

- 널이 될 수 있는 타입을 처리해도 되고 널이 될 수 없는 타입으로 처리해도 된다.
- 자바 코드를 사용할 때 조심하자
    
    ```kotlin
    // java
    public class Person {
    	private final String name;
    
    	public Person(String name) {
    		this.name = name;
    	}
    
    	public String getName() {
    		return name;
    	}
    }
    
    // kotlin
    fun yellAt(person: Person) {
    	// toUpperCase()의 수신 객체 person.name이 널일 수 있다.
    	println(person.name.toUpperCase() + "!!!")
    }
    ```
    

### 상속

코틀린에서 자바 메소드를 오버라이드할 때 그 메소드의 파라미터와 반환 타입을 널이 될 수 있는 타입으로 선언할지 널이 될 수 없는 타입으로 선언할지 결정해야 한다.

```kotlin
// java
interface StringProcessor {
	 void process(String value)
}

// kotlin - 아래 둘 다 가능하다
class StringPrinter : StringProcessor {
	override fun process(value: String) {
		println(value)
	}
}

class StringPrinter : StringProcessor {
	override fun process(value: String) {
		if (value != null) {
			println(value)
		}
	}
}
```

- 자바 클래스나 인터페이스를 코틀린에서 구현할 경우 널 가능성을 제대로 처리하는 일이 중요하다

# 6.2 코틀린의 원시 타입

코틀린은 원시 타입과 래퍼 타입을 구분하지 않는다.

## 6.2.1 원시 타입: Int, Boolean 등

대부분의 경우 코틀린의 Int 타입은 자바 int 타입으로 컴파일된다. 이런 컴파일이 불가능한 경우는 컬렉션과 같은 제네릭 클래스를 사용하는 경우뿐이다.

- Int 타입을 컬렉션의 타입 파라미터로 넘기면 그 컬렉션에는 Int의 래퍼 타입에 해당하는 Integer 객체가 들어간다.
- Int와 같은 코틀린 타입에는 널 참조가 들어갈 수 없기 때문에 쉽게 그에 상응하는 자바 원시 타입으로 컴파일할 수 있다.

## 6.2.2 널이 될 수 있는 원시 타입: Int?, Boolean? 등

- 널이 될 수 있는 코틀린 타입은 자바 원시 타입으로 표현할 수 없다. 따라서 코틀린에서 널이 될 수 있는 원시 타입을 사용하면 그 타입은 자바의 래퍼 타입으로 컴파일된다.
- 제네릭 클래스의 경우 래퍼 타입을 사용한다.
    
    ```kotlin
    val listOfInts = listOf(1, 2, 3) // Integer 타입이다.
    ```
    
    - JVM에서 제네릭을 구현하는 방법 때문
        - JVM은 타입 인자로 원시 타입을 허용하지 않는다. 따라서 자바나 코틀린 모두에서 제네릭 클래스는 항상 박스 타입을 사용해야 한다.
        - 원시 타입으로 이뤄진 컬렉션을 효율적으로 저장해야 한다면 서드파티 라이브러리를 사용하거나 배열을 사용해야 한다.

## 6.2.3 숫자 변환

코틀린은 한 타입의 숫자를 다른 타입의 숫자로 자동 변환하지 않는다. 결과 타입이 허용하는 숫자의 범위가 원래 타입의 범위보다 넓은 경우조차도 자동 변환은 불가능하다.

```kotlin
val i = 1
val l: Long = i // "Error: type mismatch"
```

- 코틀린은 모든 원시 타입(Boolean 제외)에 대한 변환 함수를 제공한다.
    - toByte(), toShort() 등
    - 코틀린은 개발자의 혼란을 피하기 위해 타입 변환을 명시하기로 결정했다.
    - equals 메소드는 그 안에 들어있는 값이 아니라 박스 타입 객체를 비교한다.
        - 따라서 자바에서 new Integer(42).equals(new Long(42))는 false다.
            
            ```kotlin
            // 코틀린
            var x = 1
            val list = listOf(1L, 2L, 3L)
            x in list // false
            
            x.toLong() in list // true
            ```
            
            타입을 명시적으로 변환해서 같은 타입의 값으로 만든 후 비교해야 한다.
            
- 숫자 리터럴을 사용할 때는 보통 변환 함수를 호출할 필요가 없다.
    - 추가로 산술 연산자는 적당한 타입의 값을 받아들일 수 있게 이미 오버로드돼 있다.
        
        ```kotlin
        fun foo(l: Long) = println(1)
        val b: Byte = 1
        val 1 = b + 1L
        foo(42)
        ```
        
    - 코틀린 산술 연산자에서도 overlfow가 발생할 수 있다.

## 6.2.4 Any, Any?: 최상위 타입

코틀린에서는 Any 타입이 모든 널이 될 수 없는 타입의 조상 타입이다.

- 코틀린에서는 Any가 Int 등의 원시 타입을 포함한 모든 타입의 조상 타입이다.
    - 자바는 참조 타입만 Object를 정점으로 하는 타입 계층에 포함.
- 내부에서 Any 타입은 java.lang.Object에 대응한다.
    - 코틀린 함수가 Any를 사용하면 자바 바이트코드의 Object로 컴파일된다.
    - 일부 메소드(wait, notify 등)는 Any에서 사용할 수 없다.

## 6.2.5 Unit 타입: 코틀린의 void

코틀린 Unit 타입은 자바 void와 같은 기능을 한다.

```kotlin
fun f(): Unit { ... }
fun f() { ... }
```

- Unit은 모든 기능을 갖는 일반적인 타입이며, void와 달리 Unit을 타입 인자로 쓸 수 있다.
    
    ```kotlin
    interface Processor<T> {
    	fun process() : T
    }
    
    class NoResultProcessor: Proccessor<Unit> {
    	override fun process() {
    		// ...
    	}
    }
    ```
    

## 6.2.6 Nothing 타입: 이 함수는 결코 정상적으로 끝나지 않는다

코틀린에는 결코 성공적으로 값을 돌려주는 일이 없으므로 '반환 값'이라는 개념 자체가 의미 없는 함수가 일부 존재한다.

그런 함수를 호출하는 코드를 분석하는 경우 함수가 정상적으로 끝나지 않는다는 사실을 표현하기 위해 Nothing이라는 특별한 반환 타입이 있다.

```kotlin
fun fail(message: String) : Nothing {
	throw IllegalStateException(message)
}
```

- Nothing 타입은 아무 값도 포함하지 않는다. 따라서 함수의 반환 타입이나 반환 타입으로 쓰일 타입 파라미터로만 쓸 수 있다.
- Nothing을 반환하는 함수를 엛리스 연산자의 우항에 사용해서 전제 조건을 검사할 수 있다.
    
    ```kotlin
    val address = company.address ?: fail("No address")
    ```
    

# 6.3 컬렉션과 배열

## 6.3.1 널 가능성과 컬렉션

- List<Int?>
    
    리스트 자체는 항상 널이 아니다. 하지만 원소는 널이 될 수 있다.
    
- List<Int>?
    
    리스트를 가르키는 변수에는 널이 들어갈 수 있지만 리스트 안에는 널이 아닌 값만 들어간다.
    

### filterNotNull

널이 될 수 있는 값으로 이뤄진 컬렉션으로 널 값을 걸러내는 경우 사용

```kotlin
fun addValidNumbers(numbers: List<Int?>) {
	val validNumbers = numbers.filterNotNull() // List<Int> 타입 리턴
	...
}
```

## 6.3.2 읽기 전용과 변경 가능한 컬렉션

코틀린 컬렉션은 자바 컬렉션과 달리 컬렉션 안의 데이터에 접근하는 인터페이스와 컬렉션 안의 데이터를 변경하는 인터페이스를 분리했다.

### kotlin.collections.Collection

- 컬렉션 안의 원소에 대해 이터레이션하고, 컬렉션의 크기를 얻고, 어떤 값이 컬렉션 안에 들어있는지 검사하고, 컬렉션에서 데이터를 읽는 여러 다른 연산 수행 가능
- 원소 추가/제거 메소드는 없다
- 읽기 전용

### kotlin.collections.MutableCollection

- 원소를 추가/삭제 등의 메소드를 더 제공한다.

### 읽기 전용 컬렉션이라고해서 꼭 변경 불가능한 컬렉션일 필요는 없다

- 읽기 전용 인터페이스 타입인 변수를 사용할 때 그 인터페이스는 실제로는 어떤 컬렉션 인스턴스를 가리키는 수많은 참조 중 하나일 수 있다.
    
    ![스크린샷 2021-10-14 오전 1.33.11.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c79f1044-73ea-4c06-968a-8560cde5bbf0/스크린샷_2021-10-14_오전_1.33.11.png)
    
    - 이런 상황에서 이 컬렉션을 참조하는 다른 코드를 호출하거나 병렬 실행한다면 컬렉션을 사용하는 도중에 다른 컬렉션이 그 컬렉션의 내용을 변경하는 상황이 생길수 있다. 따라서 읽기 전용 컬랙션이 항상 스레드 세잎하지 않다.

## 6.3.3 코틀린 컬렉션과 자바

모든 코틀린 컬렉션은 그에 상응하는 자바 컬렉션 인터페이스의 인스턴스라는 점은 사실이다. 따라서 코틀린과 자바 사이를 오갈 때 아무 변환도 필요 없다.

코틀린은 모든 자바 컬렉션 인터페이스마다 읽기 전용 인터페이스와 변경 가능한 인터페이스라는 두가지 표현을 제공한다.

![스크린샷 2021-10-14 오전 1.39.54.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/cf4a7da8-c508-4e3d-8f0c-19b8325272e9/스크린샷_2021-10-14_오전_1.39.54.png)

[컬렉션 생성 함수](https://www.notion.so/983cac7e91794a3dbd6dd7e79ab9147a)

- 컬렉션을 메소드 인자로 넘겨야 한다면 따로 변환/복사 등의 추가 작업 없이 직접 컬렉션을 넘기면 된다.
    - 이런 성질로 인해 컬렉션의 변경 가능성과 관련해 중요한 문제가 생긴다.
        
        ```kotlin
        // java
        public class CollectionUtils {
        	public static List<String> uppercaseAll(List<String> items) [
        		for (int i = 0; i < items.size(); i++) {
        			items.set(i, items.get(i).toUpperCase());
        		}
        		return items;
        	}
        }
        
        // kotlin
        fun printInUppercase(list: List<String>) { // 읽기 전용 파라미터 선언
        	println(CollectionUtils.uppercaseAll(list)) // 컬렉션을 변경하는 자바 함수 호출
        }
        ```
        
        따라서 자바로 넘기는 코틀린 프로그램을 작성한다면 호출하려는 자바 코드가 컬렉션을 변경할지 여부에 따라 올바라른 파라미터 타입을 사용할 책임은 개발자에게 있다.
        
    - 널이 아닌 원소로 이뤄진 컬렉션 타입에도 발생 가능한 문제.
        - 널이 아닌 원소로 이뤄진 컬렉션을 자바 메소드로 넘겼는데 자바 메소드가 널을 컬렉션에 넣을 수도 있다.

## 6.3.4 컬렉션을 플랫폼 타입으로 다루기

자바 코드에서 정의한 타입을 코틀린에서는 플랫폼 타입으로 본다.

- 플랫폼 타입의 경우 코틀린 쪽에는 널 관련 정보가 없다.
- 보통은 원하는 동작이 그냥 잘 수행될 가능성이 높으므로 실제 문제가 되지는 않는다.

컬렉션 타입이 시그니처에 들어간 자바 메소드 구현을 오버라이드하려는 경우 읽기 전용 컬렉션과 변경 가능 컬렉션의 차이가 문제가 된다. 이런 상황에서는 여러 가지를 선택해야 한다.

- 컬렉션이 널이 될 수 있는가?
- 컬렉션의 원소가 널이 될 수 있는가?
- 오버라이드하는 메소드가 컬렉션을 변경할 수 있는가?

```kotlin
interface FileContentProcessor {
	void processContents(File path, byte[] binaryCOntents, List<String> textContents);
}
```

이 인터페이스를 코틀린으로 구현하려면 다음을 선택해야 한다.

- 일부 파일은 이진 파일이며 이진 파일 안의 내용은 텍스트로 표현할 수 ㅇ없는 경우가 있으므로 리스트는 널이 될 수 있다.
- 파일의 각 줄은 널일 수 없으므로 이 리스트의 원소는 널이 될 수 없다.
- 이 리스트는 파일의 내용을 표현하며 그 내용을 바꿀 필요가 없으므로 읽기 전용이다.

이를 코틀린으로 구현한 모습을 보여준다.

```kotlin
// FileContentProcessor를 코틀린으로 구현한 모습
class FileIndexer : FileContentProcessor {
	override fun processContents(path: File, 
		binaryContents: ByteArray?, 
		textContents: List<String>?) {
			...
	}
}
```

## 6.3.5 객체의 배열과 원시 타입의 배열

```kotlin
// kotlin 배열
fun main(args: Array<String>) {
	for (i in args.indices) { // 배열의 인덱스 값의 범위에 대해 이터레이션하기 위해 array.indices 확장 함수 사용
		println("Argument $i is: ${args[i]}") // array[index]로 인덱스를 사용해 배열 원소 접근
	}
}
```

### 코틀린에서 배열을 만드는 법

코틀린 배열은 타입 파라미터를 받는 클래스다. 배열의 원소 타입은 바로 그 타입 파라미터에 의해 정해진다.

1. arrayOf 함수에 원소를 넘기면 배열을 만들 수 있다.
2. arrayOfNulls 함수에 정수 값을 인자로 넘기면 모든 원소가 null이고 인자로 넘긴 값과 크기가 같은 배열을 만들 수 있다.
    - 물론 원소 타입이 널이 될 수 있는 타입인 경우에만 쓸 수 있다.
3. Array 생성자는 배열 크기와 람다를 인자로 받아서 람다를 호출해서 각 배열 원소를 초기화해준다.
    - arrayOf를 쓰지 않고 각 원소가 널이 아닌 배열을 만들어야 하는 경우 이 생성자를 사용한다.
    
    ```kotlin
    val letters = Array<String>(26) { i -> ('a' + i).toString() } // 타입 인자 생략 가능
    ```
    
    - 코틀린은 배열을 인자로 받는 자바 함수를 호출하거나 vararg 파라미터를 받는 코틀린 함수를 호출하기 위해 배열을 자주 선언. 이 때 toTypedArray 메소드를 사용하면 컬렉션을 배열로 변경 가능
        
        ```kotlin
        val strs = listOf("a", "b", "c")
        println("%s/%s/%s".format(*strs.toTypedArray())) // vararg 인자를 넘기기 위해 스프레드 연산자(*)를 써야 한다.
        ```
        

### 코틀린 배열 특징

- 배열 타입의 타입 인자도 항상 객체 타입이 된다.
    - Array<Int> 선언시 그 배열은 박싱된 정수의 배열이다.
    - 박싱하지 않은 원시 타입의 배열이 필요하다면 그런 타입을 위한 특별한 배열 클래스를 사용해야 한다.
        - IntArray, ByteArray, CharArray, BooleanArray 등
        - 생성방법
            1. 생성자는 size 인자를 받아서 해당 원시 타입의 디폴트 값(보통 0)으로 초기화된 size 크기의 배열을 반환한다.
                
                ```kotlin
                val fiveZeros = IntArray(5)
                ```
                
            2. 팩토리 함수(IntArray, intArrayOf 등)는 여러 값을 가변 인자로 받아서 그런 값이 들어간 배열을 반환한다.
                
                ```kotlin
                val fiveZerosToo = intArrayOf(0, 0, 0, 0, 0)
                ```
                
            3. 크기와 람다를 인자로 받는 생성자를 사용한다.
                
                ```kotlin
                val squares = IntArray(5) { i -> (i + 1) * (i + 1) }
                ```
                
        - 박싱된 값이 들어있는 컬렉션이나 배열은 toIntArray 등의 변환 함수를 사용해 원시 타입의 배열로 변환 가능하다.
- 코틀린 표준 라이브러리는 배열 기본 연산에 더해 컬렉션에 사용할 수 있는 모든 확장 함수를 배열에도 제공한다.
    - 다만, 그런 확장 함수를 사용할 경우 반환 값이 리스트일 수 있다.
