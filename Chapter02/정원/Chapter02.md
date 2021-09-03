# 2 코틀린 기초
2장에서 다루는 내용
- 함수, 변수, 클래스, enum, 프로퍼티를 선언하는 방법
- 제어구조
- 스마트 캐스트
- 예외 던지기와 예외 잡기

---
## 2.1 기본 요소: 함수와 변수
<br>

### 2.1.1 Hello, World!
```kotlin
fun main(args: Array<String>) {
    println("Hello, world!")
}
```
- 함수를 선언할 때 fun 키워드를 사용한다.
- 파라미터 이름 뒤에 타입을 쓴다.
- 함수를 최상위 수준에 정의할 수 있다.
- 세미콜론을 붙이지 않아도 된다.
<br>

### 2.1.2 함수
```kotlin
fun max(a: Int, b: Int): Int {
    return if (a>b) a else b
}
```
코틀린의 if문은 식이다. 값을 만들어 내며 다른 식의 하위 요소로 계산에 참여할 수 있다. 코틀린에서는 루프를 제외한 대부분의 제어 구조가 식이다.

**식이 본문인 함수**<br>
```
fun max(a: Int, b: Int): Int = if (a > b) a else b
```
식이 본문인 함수의 경우 반환 타입을 적지 안항도 컴파일러가 함수 본문 식을 분석해 식의 결과 타입을 함수 반환 타입으로 정해준다. 
이렇게 컴파일러가 타입을 분석해 구성 요소의 타입을 정해주는 기능을 타입추론이라 부른다.
<br>

### 2.1.3 변수
**변경 가능한 변수와 변경 불가능한 변수**
- val : 변경불가능한
- var : 변경 가능한

기본적으로 모두 val로 선언한 다음 필요할때 var로 바꿔라! 변경 불가능한 참조와 변경 불가능한 객체를 부수 효과가 없는 함수와 조합해 사용하면 코드가 함수형 코드에 가까워진다.<br>
var를 사용하면 변수의 값을 바꿀순 있지만 타입은 고정돼 바뀌지 않는다!

### 2.1.4 문자열 템플릿
```kotlin
val name = ~~~
println("Hello, $name")
```

유니코드 주의! ${name} 처럼 중괄호로 감싸자.

## 2.2 클래스와 프로퍼티
**간단한 자바 클래스**<br>
```java
public class Person{
    private final String name;

    //생성자
    //게터
}
```

**코틀린으로 변환한 Person 클래스**<br>
```kotlin
class Person(val name: String)
```
이러한 유형의 클래스를 값객체 라 부른다. public이 사라졌음을 확인하라, 코틀린의 기본 가시성은 public이다.

### 2.2.1 프로퍼티
클래스라는 개념의 목적은 데이터를 캡슐화 하고 캡슐화한 데이터를 다루는 코드를 한 주체 아래 가두는 것.<br>
자바에서는 필드와 접근자를 한데 묶어 프로퍼티라고 부르며, 이 개념을 활용하는 프레임워크가 많다. 코틀린은 프로퍼티를 언어 기본 기능으로 제공하며, 코틀린 프로퍼티는 자바의 필드와 접근자 메소드를 완전히 대신한다.

<br>

**클래스 안에서 변경 가능한 프로퍼티 선언**<br>
```kotlin
class Person{
    val name: String, // 읽기전용 프로퍼티
    var isMarried: Boolean // 쓸 수 있는 프로퍼티
}
```

<br>  

**Person 클래스 사용하기**<br>
```kotlin
val person = Person("Bob", true)
print(person.name)
print(person.isMarried) // 자동으로 게터를 호출 해준다

person.isMarried = false
```

### 2.2.2 커스텀 접근자
```kotlin
class Rectangle(val height: Int, val width: Int) {
    val isSquare: Boolean
        get() {
            return height == width
        }
}
```
위처럼 사용하면 값을 저장 하지 않고도 그때 그때 계산 해서 필드처럼 사용 할 수 있다.
```
val rectangle = Rectangle(4, 3)
print(rectangle.isSquare)
```

### 2.2.3 코틀린 소스코드 구조: 디렉터리와 패키지
-생략- ide의 도움을 받자.

## 2.3 선택 표현과 처리: enum과 when
when은 자바의 Switch를 대치하되 훨씬 더 걍력하며, 더 자주 사용할 프로그래밍 요소다!

### 2.3.1 enum 클래스 정의
```kotlin
enum class Color {
    RED, ORANGE, YELLOW, GREEN, ...
}
```

**프로퍼티와 메소드가 있는 Enum 선언**<br>
```kotlin
enum class Color(
    val r: Int, val g: Int, val b: Int
) {
    RED(255, 0, 0), ...

    fun rgb() = (r * 256 + g) * 256 + b
}

print(Color.RED.rgb())
```


### 2.3.2 when으로 enum 클래스 다루기
```kotlin
fun getMnemonic(color: Color) =
    when (color) {
        Color.RED -> "Richard"
        Color.Orange -> "Of"
        Color.`` ->
        ...
    }
```
자바와 달리 각 분기 끝에 break를 넣지 않아도 된다. 콤마를 사용해 한 분기 안에서 여러 값을 매치 패턴으로 사용할 수 있다.

### 2.3.3 when과 임의의 객체를 함께 사용
```kotlin
fun min(c1: Color, c2: Color) =
    when (setOf(c1, c2)) {
        setOf(RED, YELLOW) -> ORANGE
        ...
        else -> throw Exception("Dirty color")
    }
```
간결하고 아름다운 코드!

### 2.3.4 인자 없는 when 사용
```kotlin
fun min(c1: Color, c2: Color) =
    when {
        (c1 == RED && c2== YELLOW) ||
        (c1 == YELLOW && c2 == RED) -> ORANGE
    }
```

### 2.3.5 스마트 캐스트
```kotlin
interface Expr
class Num(val value: Int): Expr
class Sum(val left: Expr, val right: Expr)

fun eval(e: Expr): Int =
    when(e) {
        is Num -> e.value
        is Sum -> eval(e.right) + eval(e.left)
        else -> throw Exception
    }
```

## 2.4 대상을 이터레이션
자바와 가장 비슷.

### 2.4.1 While 루프
문법이 자바와 다르지 않다...

### 2.4.2 수에 대한 이터레이션: 범위와 수열
```kotlin
val oneToTen = 1..10
```
..연산자로 으로 범위를 만든다.

<br>

```kotlin
for (i in 1..100) {
    print(i)
}
```

### 2.4.3 맵에 대한 이터레이션
```
val binaryReps = TreeMap<Char, String>()
for ((letter, binary) in binaryReps) {
    print(letter, binary)
}
```

### 2.4.4 in으로 컬렉션이나 범위의 원소 검사
```kotlin
fun isLetter(c: Char) = c in 'a'..'z' || 'A'..'Z'
```

..연산자로 만든 범위에 in 키워드를 통해 검사 가능.

## 2.5 코틀린의 예외 처리
자바와 비슷.
```kotlin
throw Exception("")
```

### 2.5.1 try, catch, finally
코틀린은 다른 JVM언어와 같이 체크드 익셉션과 언체크 익셉션을 구별하지 않는다. 코틀린 함수가 던지는 예외를 지정하지 않고 발생한 예외를 잡아내지 않아도 된다.

### 2.5.2 try를 식으로 사용
```kotlin
fun readNumber(reader: BufferedReader){
    val number = try {
        Integer.ParseInt(reader.readLine())
    } catch (e: NumberFormatException) {
        return
    }
    println(number)
}
```
