# 코틀린 기초
## 함수와 변수
### 2.1.1 Hello World
* 파라미터, 변수 이름 뒤에 타입을 쓴다.(`Array<String>`)
* 함수를 최상위 수준에 정의할 수 있다(꼭 클래스 안에 함수를 넣을 필요 X)
* 배열 처리를 위한 문법이 따로 존재하지 않음  Q. 이게 구체적으로 무슨말...?
* sout 대신 println이라고 쓴다. 그런데 sout이 더 짧긴한데..

### 2.1.2 함수
* fun 함수이름(파라미터이름:타입 ...): 반환 타입
* Kotlin: if(a>b) a else b, 자바나 파이썬..?: a if(a>b) else b
* 식이 본문인 함수
`fun max(a: Int, b: Int): Int = if (a > b) a else b`
코틀린은 static type binding 언어이므로 컴파일 시점에 모든 식의 타입을 지정해야 함.
식이 본문인 경우에는 type inference를 통해서 구성 요소의 타입을 정해준다.

### 2.1.3 변수
- 초기화 식이 있는 경우: 타입 생략 가능
- 초기화 식이 없는 경우: 타입 생략 불가능

- 변경 가능한 변수와 변경 불가능한 변수
val - 변경 불가능한 참조(자바의 final)
var - 변경 가능한 참조
- immutable을 이용하여 side effect가 없는 함수와 조합해 사용하여 함수형 코드에 가깝게
- val 참조 자체는 불변이지만 참조가 가리키는 객체의 내부 값은 변경될 수 있음

### 2.1.4 문자열 템플릿
`println("Hello, $name!")`
`println("Hello, ${args[0]}!")`
`println("Hello, ${if (args.size > 0) args[0] else "someone"}!"`

## 클래스와 프로퍼티
- 자바의 기본 가시성은 private인데 코틀린의 기본 가시성은 public이다.
- 클래스의 목적은 encapsulation이다. 그래서 보통 자바에서는 멤버 필드를 private으로 하고, getter를 제공한다. 
- 코틀린에서는 val: 읽기 전용, var: 변경 가능
- 코틀린에서는 getter를 호출하는 대신 프로퍼티를 직접 사용한다.

### 2.2.3 디렉터리와 패키지
- 자바는 한 파일 안에 여러 클래스를 만들 수 없지만 코틀린은 가능하다 그런데 분리하는 편이 낫다

## 선택 표현과 처리: enum과 when
1) 상수의 프로퍼티를 정의한다 - val r: Int, val g: Int, val b: Int
2) 각 상수를 생성할 때 그에 대한 프로퍼티 값을 지정 - RED(255, 0, 0)
3) enum 클래스 안에서 메소드를 정의 - fun rgb() = (r*256+g)*256+b


* if와 마찬가지로 when도 값을 만들어내는 식이므로 본문에 when을 바로 사용할 수 있음

`
when (color) {
    Color.RED -> "Richard"
    Color.ORANGE -> "Of"
}
`
- Java와 다르게 break을 넣지 않아도 되며, 한 when 분기 안에 여러 값을 사용하기 위해서는 쉼표를 사용한다. 아마 C언어에서 
case:2 case:3 case:4
blah blah
break;
case 5
blah
break;
같이 쓰는 걸 보고 이렇게 설계하지 않았을까..?

### 2.3.3 when과 임의의 객체를 함께 사용
- C언어나 java는 분기조건 switch절에 상수만을 사용할 수 있는데, 코틀린은 임의의 객체를 허용한다.
- 이때는 equility를 사용한다. 찾다가 없으면 else분기를 사용한다.
### 2.3.4 인자 없는 when 사용
- 매번 함수가 호출될 때마다 인스턴스를 사용하는게 비효율적일 수 있다. Garbage 객체. 그래서 코드가 좀 더러워지더라도 인자가 없는 when절(조건)을 이용할 수 있다.
### 2.3.5 스마트 캐스트
- is로 변수에 든 값의 타입을 검사한다음에 타입캐스팅을 자동으로 해줌
- 명시적으로 캐스팅을 하려면 as를 사용

## 대상을 이터레이션: while과 for 루프
- while, do while은 자바와 같다
- for은 range를 사용하며 ..은 폐구간이다. for(i in 1..100)
- 기타 키워드: downTo, step, until
- Map의 경우에는 for ((key, val) in Map)으로 구조 분해 문법을 사용한다. 파이썬같아서 편해보인다.

### 2.4.4 in으로 컬렉션이나 범위의 원소 검사
- 파이썬과 같이 c in 'a' .. 'z' 같이 사용 가능
- try를 식으로 사용할 수 있다.