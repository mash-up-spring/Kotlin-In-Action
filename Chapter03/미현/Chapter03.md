## 3. 함수 정의와 호출 ##

### 3.1. 컬렉션 만들기 ###
코틀린에서는 표준 자바 컬렉션 클래스를 사용한다. 다만 자바보다 많은 기능을 제공한다.
```kotlin
//숫자로 이뤄진 집합
val set = setOf(1, 7, 54)

//숫자로 이뤄진 리스트
val list = listOf(1, 7, 54)

//map. 이때 to는 함수.
val map = hashMapOf(1 to "one", 7 to "seven", 54 to "fifty-four")
```

### 3.2. 함수를 호출하기 쉽게 만들기 ###
컬렉션의 디폴트 방식(toString)이 아닌 다른 방식으로 출력하는 함수를 정의해보자.
```kotlin
fun <T> joinToString(
    collection: Collection<T>,
    seperator: String,
    prefix: String,
    postfix: String
): String {
    val result = StringBuilder(prefix)
    for((index, element) in collection.withIndex()) {
        if(index > 0) result.append(seperator)
        result.append(element)
    }
    result.append(postfix)
    return result.toString()
}
```
#### 개선1. 이름을 붙인 인자 ####
코틀린에서는 작성한 함수를 호출할 때 함수에 전달하는 인자의 이름을 명시할 수 있다.   
인자 중 하나라도 이름을 명시한다면, 그 뒤에 오는 모든 인자에 이름을 명시해야한다.
```kotlin
joinToString(collection, seperator=";", prefix="[", postfix="]")
```

#### 개선2. 디폴트 파라미터 값 ####
함수 선언에서 파라미터의 디폴트 값은 지정할 수 있어 과한 오버로딩을 방지한다.
```kotlin
fun <T> joinToString(
    collection: Collection<T>,
    //디폴트 값이 지정된 파라미터
    seperator: String = ", ",
    prefix: String = "",
    postfix: String = ""
): String
```
일반 호출 문법을 사용할 땐, 선언할 때와 같은 순서로 인자 지정한다.
```kotlin
//prefix, postfix는 디폴트 값으로, seperator는 "; "로 바뀐 값으로.
joinToString(collection, ";")
```
인자에 이름을 붙인다면, 순서와 관계없이 원하는 인자의 디폴트 값만 바꿀 수 있다.
```kotlin
joinToString(collection, prefix="[")
```
함수의 디폴트 파라미터 값은 함수 선언 쪽에서 지정되기 때문에, 함수를 정의하는 곳에서 디폴트 값을 바꾸고
재컴파일하면 그 함수를 호출하는 코드 중 값을 지정하지 않는 인자들은 바뀐 값을 적용받는다.

`@JvmOverloads`   
자바에는 디폴트 파라미터의 개념이 없기 때문에, 자바에서 코틀린 함수를 호출할 때는 반드시 항상 모든 인자를
명시해야 한다. 이게 싫을때 `@JvmOverloads`를 붙이면 오버로딩한 함수가 만들어져 디폴트 파라미터를 사용하는 효과를
볼 수 있다.

#### 개선3. 정적인 유틸리티(Util) 클래스 없애기 : 최상위 함수, 최상위 프로퍼티 ####
1. 최상위 함수   
   함수를 직접 소스파일의 최상위 수준(다른 클래스들의 밖)에 위치시키면, 가장 앞의 패키지 멤버 함수가 된다.
    ```kotlin
    /*
    * strings 패키지에 직접 함수 정의
    * 파일 : join.kt
    */
    package strings

    fun <T> joinToString(...): String {...}
    ```
   코틀린 컴파일러가 생성하는 클래스 이름은 최상위 함수가 들어있던 코틀린 소스 파일의 이름과 대응한다.
    ```java
    /*
    * 위의 코틀린 코드를 자바로 변환
    */
    package strings;

    public class JoinKt {
        public static String jointToString(...) {...}
    }

    public static void main(String args[]) {
        JoinKt.joinToString(collection, "", "", "");   
    }
    ```

   `@JvmName`
   파일에 대응하는 클래스 이름을 변경하고 싶을 떈 `@file:JvmName( "원하는 이름" )`을
   파일의 맨 앞, 패키지 이름 선언 이전에 위치 시키면 된다.

2. 최상위 프로퍼티   
   데이터를 클래스 밖에 위치실 때 프로퍼티를 최상위 수준에 놓을 수 있다.   
   최상위 프로퍼티는 정적 필드에 저장이 되는데, 상수 추가도 가능하다.   
   최상위 프로퍼티도 접근자 메소드로 자바 코드에 노출(Val은 게터, var은 게터와 세터)되는데
   원시 타입과 String 타입 상수의 경우 `const` 변경자를 추가함으로써 자바의 public static final로 컴파일할 수 있다.

###3.3. 확장 함수와 확장 프로퍼티 ###
1. 확장 함수


확장 함수는 어떤 클래스의 멤버 메소드인 것처럼 호출할 수 있지만, 선언은 그 클래스의 밖에서 된 함수다.
```kotlin
package strings

/*
 * 확장함수
 * println("Kotlin".lastChar())의 결과는 n
 */
fun String.lastChar(): Char = this.get(this.length - 1)
```
위의 예시에서 String은 수신객체타입, this는 수신 객체라고 한다.   
this를 생략해서 사용할 수 있다.   
확장 함수 내부에서는 수신 객체의 메소드, 프로퍼티를 직접 사용할 수 있다.   
다만, 캡슐화를 깨지 않기 때문에 private멤버와 protected멤버를 직접 사용할 수 없다.

#### 확장함수 import ####
확장함수를 정의하고 Import를 해야지만 사용할 수 있다.   
`as`를 사용하면 이름을 바꿀 수 있는데, 코틀린 문법상 확장 함수의 이름은 짧아야한다.
```kotlin
import strings.lastChar
import strings.*
import strings.lastChar as last
```

#### 자바에서 확장 함수 호출 ####
내부적으로 확장함수는 수신 객체를 첫번째 인자로 받는 정적 메소드이다.   
따라서 호출, 실행시 부가비용이 들지 않는다.

```kotlin
//확장 함수로 정의
fun <T> Collection<T>.joinToString(
    seperator: String = ", ",
    prefix: String = "",
    postfix: String = ""
): String {
    val result = StringBuilder(prefix)
    
    for((index, element) in this.withIndex()) {
        if(index > 0) result.append(seperator)
        result.append(element)
    }
    result.append(postfix)
    return result.toString()
}
```

#### 확장 함수는 오버라이드 불가 ####
확장 함수는 정적 메소드와 같은 특징을 가지기 때문에 오버라이드 할 수 없다.

2. 확장 프로퍼티

게터를 꼭 정의해 주어야한다.   
초기화 코드를 쓸 수 없다(계산한 값을 담을 곳이 없다.)
```kotlin
val String.lastChar: Char
    get() = get(length - 1)
```
StringBuilder는 마지막 문자가 변경가능해 var로 프로퍼티를 만들 수 있다.
```kotlin
var StringBuilder.lastChar: Char
    get() = get(length - 1)
    set(value: Char) {
        this.setCharAt(length-1, value)
    }
```

###3.4. 컬렉션 처리 ###
코틀린에서 제공하는 컬렉션의 추가 기능들은 확장 함수다.

#### 가변 인자 함수 ####
자바의 가변 길이 인자와 유사하지만, 코틀린에서는 파라미터 앞에 vararg 변경자를 붙인다.
```kotlin
fun listOf<T>(vararg values: T): List<T> { ... }
```
배열에 들어있는 원소를 가변 길이 인자로 넘길 때는 `스프레드 연산자`로 배열을 풀어서 각 원소가 인자로 전달되게 해야한다.
```kotlin
fun main(args: Array<String>) {
    val list = listOf("args: ", *args)  //스프레드 연산자
}
```

#### 중위 호출과 구조 분해 선언 ####
```kotlin
val map = hashMapOf(1 to "one", 7 to("seven"))  //두 방법 모두 결과 동일
```
map을 만들때 쓰는 mapOf함수에서 사용한 `to`는 일반 메소드로 중위 호출에 쓰인다.   
중위 호출는 인자가 하나인 메소드에 사용가능하다.   
메소드를 중위 호출에 사용하고 싶다면 `infix`변경자를 메소드 선언 앞에 추가한다.
```kotlin
infix fun Any.to(other: Any) = Pair(this, other)
```
ㅅ`to`는 확장 함수로 수신 객체가 제네릭하다.

###3.5. 문자열과 정규식 다루기 ###
자바의 문자열과 코틀린의 문자열은 같다. 코틀린에서는 확장 함수로 더 많은 기능을 제공한다.

#### 문자열 나누기 ####
자바의 split은 String타입의 정규식으로 구분 문자열을 표현해 불편하다.   
코틀린은 여러 방법으로 파라미터를 받도록 확장함수를 제공한다.
1. Regex타입의 정규식을 파라미터로 받는 경우
```kotlin
println("12.345-6.A".split("\\.|-".toRegex()))  //toRegex로 정규식으로 반환
```
2. 구분 문자열 지정
```kotlin
println("12.345-6.A".split(".", "-"))
```

#### 정규식과 3중 따옴표로 묶은 문자열 ####
정규식은 강력하지만 가독성이 떨어진다. 코틀린에서 제공하는 확장함수를 활용하면 정규식을 대체할 수 있다.
- substringBeforeLast()
- substringAfterLast()

3중 따옴표 문자열을 사용해 정규식을 표현하면 이스케이프가 불필요하다.
```kotlin
val regex = """(.+)/(.+)\.(.+)""".toRegex()
```

#### 여러 줄 3중 따옴표 문자열 ####
3중 따옴표를 사용하면 여러 줄 문자열을 코드 상에서 보기 좋게 표현할 수 있다.

###3.6. 로컬 함수와 확장 ###
코드의 중복을 피하기 위해 메소드 내부 로직을 개별 메소드로 떼어 놓는데, 이떄 더욱 깔끔할 수 있도록
코틀린에서는 `로컬 함수`를 지원한다.   
로컬 함수는 함수에서 추룰한 함수를 원 함수 내부에 중첩시키는 방법이다.   
로컬 함수는 자신이 속한 바깥 함수의 모든 파라미터, 변수를 사용할 수 있다.
```kotlin
class User(
    val id: Int,
    val name: String,
    val address: String
) {
    fun saveUser(user: User) {
        fun validate(value: String, fieldName: String) {
            if(value.isEmtpy()) {
                throw IllegalArgumentException(${user.id} empty $fieldName")
            }
        }
        
        validate(user.name, "Name")
        validate(user.address, "Address")
        
        //저장
    }
}
```
위의 로컬 함수를 확장 함수로 추출할 수 있다.
```kotlin
class User(
    val id: Int,
    val name: String,
    val address: String
) {
    fun User.validateBeforeSave() {
        fun validate(value: String, fieldName: String) {
            if(value.isEmtpy()) {
                throw IllegalArgumentException("$id empty $fieldName")
            }
        }
        
        validate(user.name, "Name")
        validate(user.address, "Address")
    }
    
    fun saveUser(User: user) {
        user.validateBeforeSave()
        //저장
    }
}
```
중첩된 함수 깊이기 깊어지면 가독성이 떨어지기 때문에 일반적으로 한 단계만 중첩시기키를 권장한다.