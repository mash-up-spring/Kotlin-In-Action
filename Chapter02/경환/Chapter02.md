# Ch2. 코틀린 기초

# 2.1 기본 요소 : 함수와 변수

## 2.1.1 Hello World

```
fun main(args: Array<String>) {
  
  println("Hello World!")
}
```

- 함수를 선언할 때는 fun 키워드를 사용한다.
- 변수명 뒤에 타입을 선언한다.
- 함수를 최상위 수준에 정의할 수 있다.
- 배열 문법이 따로 존재하지 않는다.
- 간결성을 위해 자바 메서드를 감싼 Wrapper 메서드가 있다.
- 세미콜론으르 생략 가능하다.
***

## 2.1.2 함수

```
fun max(a: Int, b: Int): Int = if(a>b) a else b
````
- 식(Expression): 값으로 치환될 수 있는 코드
- 문장(statement): 실행될 수 있는 코드 단위, 값을 만들어내지 않는다. 
- 코틀린은 루프를 제외한 **모든 제어 구조가 식**이다.-> 값을 만들어낸다.

### 식이 본문인 함수
- 반환값 생략 가능(타입 추론 지원)

### 블록이 본문인 함수
- 반환 타입 생략 불가

## 2.1.3 변수
```
val question = "삶,우주, 어쩌구"
val answer : Int = 42
var year = 23

val hi : String
hi = "hi"
````
 - 소수점은 double로 변환된다.
 - 초기화 없이 변수 선언 시 타입 선언 필요
 - val : value, immutable
 - var : variable, mutable
 - 대입에 의한 **타입 추론은 초기 타입 추론을 따라간다**.
 

 **유의) 꼭 필요할때만 var를 쓰자, val의 내부 참조값은 변경될 수 있다.(Ex.컬렉션)**

***

 ## 2.1.4 문자열 템플릿
 ```
 val name = "경환"
 println("Hello, $name!")
 ```
- $를 이용해서 문자열 내에 변수를 사용할 수 있다.
- ${}로 식도 문자열 템플릿과 같이 사용할 수 있다.
- 가독성을 위해 가급적 변수도 {}로 감싸자.

# 2.2 클래스와 프로퍼티
## 2.2.1 프로퍼티

프로퍼티 선언
```
class Person(
    val name: String,
    val isMarried : Boolean
){}
```

- 프로퍼티란 필드 + 접근자
- val : 읽기 전용 프로퍼티 (only getter) -> getXXX
- var : 변경 가능 프로퍼티 (getter + setter) -> getXXX, setXXX
- is 로 시작하는 프로퍼티는 이름 그대로 간다.
- Backing field: 프로퍼티 값을 저장하기 위한 필드
- 커스텀 게터로 Backing field 없이 그때그때 값을 계산해주는 프로퍼티를 만들 수 있다.
***
## 2.2.2 커스텀 접근자  
```
class Nemo(val h : Int, val w : Int){
    val isSquare : Boolean
      get(){
          return h == w
      }
}
```
- 프로퍼티는 getH, getW, isSquare
- isSquare는 Backing Field가 없다.
- 파라미터가 없는 함수와 Backing Field가 없는 프로피터의 **차이는 그저 가독성**뿐, 객체의 특성을 보여주고 싶을 땐 후자를 쓰자.
***
## 2.2.3 디렉토리와 패키지

- 최상위에 선언된 함수도 import 할 수 있다.
- 패키지 구조와 디렉토리 구조가 일치할 필요는 없으나 **가급적 맞추자.**
- 여러 클래스 한 파일에 넣어도 된다.
***
# 2.3 Enum 과 when
## 2.3.1 Enum 클래스 정의
```
enum class Color{
    RED, ORANGE, YELLOW
}
```
- enum은 class 앞에 붙을 때만 키워드이다.
- 클래스와 마찬가지로 프로퍼티와 메소드를 정의할 수 있다.
- **메서드 정의시 열거 끝에 세미콜론 필수!**

***
## 2.3.2 when 과 enum
```
fun getMnemonic(c : Color) = 
  when(c){
      Color.RED -> "Richard"
      Color.ORANGE -> "Of"
      Color.GREEN, Color.BLUE -> ...
      ...
  }
```

- **when 도 식이다!**
- break문 필요 없고 매치 조건은 쉼표로 더해준다.
- when은 분기 조건에 임의의 객체를 허용한다.
- when의 인자로 받은 객체와 분기 조건에 있는 객체와 **동등성 비교**를 한다. (set 예시)
- 인자 없이 비교조건을 직접 boolean으로 만들어서 객체 생성을 줄일 수 있으나 가독성이 떨어진다.
***
## 2.3.5 스마트 캐스트
```
fun eval(e: Expr) : Int{
    if(e is Num){
        val n = e as Num // 스마트 캐스트를 쓰지 않을때
        return n.value
    }
    if(e is Sum){
        return eval(e.right) + eval(e.left)
    }
}
```

- 컴파일러가 타입을 알 수 있을때 명시적 타입변환 없이도 해당 타입처럼 사용할 수 있다.
- var이거나 커스텀 접근자인 경우는 불가능, as로 명시적 캐스팅이 필요하다.
- if나 when에 블록을 사용할 경우 **블록 마지막이 그 분기의 결과이다.**

***
# 2.4 이터레이션

## 2.4.1 while 루프
- while과 do while은 자바와 동일하다.
***
## 2.4.2 범위와 수열
코틀린은 반복을 위해 범위를 사용한다.
```
val oneToTen = 1..10 // 폐구간

100 downTo 1 step 2 // 99, 94, ..., 2

0 until n // 0, 1, ..., n-1
```
- Comparable을 구현했다면 범위를 만들 수 있다.
- in 으로 어떤 값이 범위에 속하는 지 알 수 있다.

***
## 2.4.3 맵에 대한 이터레이션

```
for((key, value) in map){
    ...
} 
for((index, element) in list.withIndex()){
    ...
}
```
- map과 같은 복합 객체를 풀어서 사용할 수 있다.

***
# 2.5 코틀린의 예외 처리
- 코틀린의 예외처리는 다른 언어의 예외처리와 비슷하다. 
- 그러나 체크 예외와 언체크 예외를 구분하지 않는다.
- 즉 throws 키워드가 필요 없다.
*** 

## 2.5.2 try를 식으로 사용

```
val number = try{
    Integer.parseInt(readNumber)//식의 값
} catch(e: Exception){
    null//식의 값
}
```
- **try도 마찬가지로 식** -> 값을 만들어낸다.
- 예외가 발생하지 않았다면 number는 정수,
- 예외가 발생했다면 number의 값은  null이다.













