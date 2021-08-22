# Ch.3 함수 정의와 호출

## 3.1 코틀린에서 컬렉션 만들기

```
val set = hashSetOf(1,7,53)
val list = arrayListOf(1,7,53)
val map = hashMapOf(1 to "one", ...)

val strings = listOf("first", ...)
val numbers = setOf(...)

val last = strings.last()
val max = numbers.max()
````

코틀린은 자체 컬렉션을 제공하지 않고 자바 컬렉션에 의존한다.  
코틀린의 메서드 익스텐션을 이용해서 자바 컬렉션보다 더 쉽게 컬렉션을 사용할 수 있다.

***

## 3.2 함수를 호출하기 쉽게 만들기

### 3.2.2 디폴트 파라미터값
```
fun <T> joinToString(
    collection: Collection<T>,
    sparator : String = ",",
    ...
)
```
파이썬처럼 디폴트 파라미터값을 사용할 수 있다.  
파라미터에 이름을 붙여 빌더 패턴처럼 사용할 수 있다.  
디폴트 파라미터는 함수 선언부에서 정의되므로 디폴트 파라미터 선언값을 바꾸고 다시 컴파일하면 호출쪽도 값이 바뀐다. 

***

### 3.2.3 최상위 함수와 프로퍼티
자바에서는 static method로만 이루어진 util 클래스가 있다.  
코틀린에서는 함수와 프로퍼티를 최상위에 정의할 수 있으므로 util 클래스가 필요 없다.

## 3.3 확장함수와 프로퍼티

기존 자바 API를 재작성하지 않고도 코틀린이 제공하는 여러 편리한 기능을 사용해보자!

확장함수 : 어떤 클래스의 멤버 메서드인 것처럼 호출할 수 있지만 사실 그 클래스 밖에 선언된 함수  

수신객체타입 : 확장함수를 적용할 객체의 타입
수신객체 : 확장 함수를 적용할 객체

```
fun String.lastChar() : Char
 = this.get(this.legth -1)

 println("Kotlin".lastChar())
```
여기서 수신객체타입은 String, 수신객체는 this이다.  
this는 생략 가능하나 객체지향을 깨지 않기 위해 private나 protected 멤버에는 접근할 수 없다.

확장함수도 임포트 할 수 있으며 as를 이용해 alias를 붙여 이름 충돌을 피할 수 있다.

확장함수는 자바 코드로는 수신객체를 파라미터로 받는 정적 메소드이다. 그러므로 오버라이딩 할 수없다. 

멤버함수와 확장함수의 시그니처가 같다면 멤버함수가 호출된다는 점에 유의하자.

***

### 3.3.5 확장 프로퍼티
확장 메서드와 마찬가지로 확장 프로퍼티 또한 정의할 수 있다.
```
var String.lastChar : Char
  get() = get(length - 1)
  set(value : Char)
   = this.setCharAt(length-1, value)
```

***

## 3.4 컬렉션 처리: 가변길이인자, 중위함수호출

### 3.4.1 자바 컬렉션 API 확장
```
fun <T> List<T>.last() : T {...}
fun Collection<Int>.max() Int {...}
```
코틀린 표준 라이브러리는 수많은 확장 함수를 지원한다.  
IDE의 코드 완성 기능을 이용해서 잘 써보자.

***

### 3.4.2 가변인자함수

```
fun listOf<T>(vararg values : T) :List<T>
```

가변인자: java의 ...과 동일하다  
...을 붙이는 대신 vararg를 앞에 써주자.  
배열을 가변인자로 넘길 때는 배열을 풀어주는 스프레드 연산자, *를 쓰자.

***
## 3.4.3 값의 쌍 다루기 : 중위호출과 구조분해선언
```
val map = mapOf(1 to "one", 7 to "seven", ...)
```
여기서 to는 코틀린 키워드가 아니라 메서드이다.
인자가 한개뿐인 메서드에 중위 호출을 사용할 수 있다.
```
// 일반적인 호출
1.to("one")
// 중위 호출
1 to "one"

infix fun Any.to(other : Any) = Pair(this,other)

val (number, name) = 1 to "one"
// 서로 동일한 함수
val (number, name) = Pair(1,"one")
```

Pair와 Map 같이 복합 값을 이루어진 객체는 위와 같이 분해할 수 있다.
***
## 3.5 문자열과 정규식 다루기

코틀린에서 문자열은 자바 문자열과 완전히 동일하다.

***
### 3.5.1 문자열 나누기
자바에서 String의 split 메서드의 구분 문자열은 정규식이다.
따라서 split(".")은 제대로 작동하지 않는다.
코틀린에서 제공하는 split은 Regrex 타입의 값을 받는다.
```
"12.345-6.A".split("\\.|-".toRegrex())
```
처럼 명시적으로 정규식 객체를 파라미터로 넘겨야한다.
***
### 3.5.2 정규식과 3중 따옴표

정규식은 가독성이 떨어지므로, 코틀린에서는 정규식을 사용하지 않고도 문자열을 파싱할 수 있다.  
parsePath 확장함수와 이스케이프 없는 문자열을 나타내는 삼중따옴표를 이용해서 쉽게 파싱할 수 있다.  
```
fun parsePath(path : String) {
    val regex """(.+)/(.+)\.(.+)""".toRegex()
    val matchResult = regex.matchEntire(path)
    if(matchResult != null){
        val (dir, name, extension) = matchResult.destructured
        println("Dir : $di, name: $name, ext: $extension")
    }
}
```
***
## 3.6 코드 다듬기 : 로컬 함수와 확장

일부 코드를 메서드로 만들면 긴 메서드를 부분 부분 나누어서 재활용할 수 있다.  (메서드 추출)  
하지만 메서드 추출은 작은 메서드가 많이 만들어져 코드를 이해하기 어렵게 만든다.  
코틀린에서는 추출한 메서드를 원래의 함수에 중첩시킬 수 있다.  
확장함수를 로컬함수로 정의할 수도 있으나 많이 중첩시키지는 말자.
```
fun saveUser(user : User){
    //파라미터로 받은 user는 로컬 메서드에서 사용할 수 있다.
    fun validate(value : String
                 fieldName : String
    ){
        //Validation Logic
    }
    validate(user.name, "Name")
    validate(user.address, "Address)
}
```

***
## 3.7 요약
- 코틀린은 자체 컬렉션 클래스를 정의하진 않지만 자바 클래스를 확장해서 더 풍부한 API를 제공한다.
- 함수 파라미터의 디폴트 값을 정의하면 빌더 패턴을 대신할 수 있다.
- 코틀린 파일에서 최상위 함수와 프로퍼티를 적접 선언할 수 있다.
- 확장 함수와 프로퍼티를 사용하면 API의 소스코드를 바꿀 필요 없이 확장할 수 있다. 성능 상 비용이 없다.
- 중위 호출을 이용해 더 깔끔한 구문을 만들 수 있다.
- 코틀린은 정규식과 일반 문자열을 처리할 때 유용한 함수를 제공한다.
- 이스케이프가 많이 필요할 경우 3중 따옴표를 쓰자.
- 로컬 함수를 써서 코드를 깔끔하게 유지하면서도 중복을 제거할 수 있다.













