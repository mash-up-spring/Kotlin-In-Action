# 코틀린이란 무엇이며, 왜 필요한가?

> 코틀린은 자바 플랫폼에서 돌아가는 새로운 프로그래밍 언어다. 코틀린은 간결하고 실용적이며, 자바 코드와의 상호운용성을 중시한다.

<br>

## 코틀린 맛보기

```kotlin
// 데이터 클래스
data class Person (val name: String,
                   val age: Int? = null) // Null이 될 수 있는 타입과 파라미터 디폴트 값

fun main (args: Array<String>) {         // 최상위 함수
    val persons = listOf(Person("영희"), Person("쳘수", age = 29))

    val oldest = persons.maxByOrNull { it.age ?: 0 }  // 람다 식과 엘비스 연산자
    println("나이가 가장 많은 사람: $oldest");   // 문자열 템플릿
}
```

위의 코드는 나이가 가장 많은 사람을 찾는 코드입니다. 

<br>

## 코틀린의 주요 특성

### 대상 플랫폼: 서버, 안드로이드 등 자바가 실행되는 모든 곳

코틀린의 주목적은 현재 자바가 사용되고 있는 모든 용도에 적합하면서도더 간결하고 생선적이며 안전한 대체 언어를 제공하는 것!

코틀린을 활용할 수 있는 가장 일반적인 영역은 다음과 같습니다.

- 서버상의 코드

- 안드로이드 디바이스에서 실행되는 모바일 애플리케이션

- 인텔의 멀티 OS 엔진을 사용하면 코틀린을 iOS 디바이스에서 실행 가능!

- 자바뿐 아니라 자바스크립트로도 코틀린을 컴파일 가능! 따라서 브라우저나 노드에서 실행할 수 있다.

<br>

## 정적 타입 지정 언어

자바와 마찬가지로 코틀린도 `정적 타입(statically typed)` 지정 언어입니다. 

```
정적 타입 지정

모든 프로그램 구성 요소의 타입을 컴파일 시점에 알 수 있고 프로그램 안에서 객체의 필드나 메소드를 사용할 때마다 컴파일러가 타입을 검증해준다는 뜻
```` 

한편 자바와 달리 코틀린에서는 모든 변수의 타입을 프로그래머가 직접 명시할 필요가 없습니다. 대부분의 경우 코틀린 컴파일러가 문맥으로부터 변수 타입을 자동으로 유추할 수 있기 때문에 프로그래머는 타입 선언을 생략해도 됩니다. 

```kotlin
var x = 1
```

여기서 위와 같이 변수를 정수로 초기화하면 코틀린은 이 변수의 타입이 Int임을 자동으로 알아냅니다. 컴파일러가 문맥을 고려해 변수 타입을 결정하는 이런 기능을 `타입 추론`이라고 부릅니다.

- 코틀린은 `타입 추론`을 지원하므로 정적 타입 지정 언어에서 프로그래머가 직접 타입을 선언해야 함에 따라 생기는 불편함이 대부분 사라집니다.

- 코틀린은 `클래스`, `인터페이스`, `제네릭` 모두 자바와 비슷하지만! 가장 큰 특성은 `코틀린이 널이 될 수 있는 타입을 지원한다는 점`입니다.

- 널이 될 수 있는 타입을 지원함에 따라 컴파일 시점에 널 포인터 예외가 발생할 수 있는지 여부를 검사할 수 있어서 좀 더 프로그램의 신뢰성을 높일 수 있습니다.

- 코틀린의 타입 시스템에 있는 다른 새로운 내용으로는 `함수 타입`에 대한 지원을 들 수 있습니다. 

<br>

## 함수형 프로그래밍과 객체지향 프로그래밍 

- 일급 시민인 함수(일급 객체): 함수를 일반 값처럼 다룬다.
    - 간결성: 함수를 값처럼 활용할 수 있으면 더 강력한 추상화를 할 수 있고 강력한 추상화를 상ㅇ해 코드 중복을 막을 수 있습니다.

- 불변성
    - 다중 스레드를 사용해도 안전하다는 사실!

- 부수 효과 없음: 입력이 같으면 항상 같은 출력을 내놓는다.
    - 테스트하기 쉽습니다. 그 함수를 실행할 때 필요한 전체 환경이 없다보니!

<br>

함수형 프로그래밍의 핵심은 위와 같습니다. 

<br>

## 코틀린 철학

- 실용성

- 간결성
  - getter, setter, 생성자 파라미터 등등..

- 안정성
  - 코틀린도 JVM 에서 실행함 !
  - 실행 시점에 오류를 발생시키는 대신 컴파일 시점 검사를 통해 오류를 더 많이 방지해줍니다! 대표적으로 `NullPointerException`을 없애기 위해 노력합니다!
  - ```kotlin
    val s: String? = null   // 널이 될 수 있음
    val s3: String = ""     // 널이 될 수 없음
    ```
  - 코틀린이 방지해주는 다른 예외로는 ClassCastException이 있습니다. 코틀린에서는 타입 검사와 캐스트가 한 연산자에 의해 이뤄지기 떄문에 예외를 방지하기에 좀 더 수월합니다.

- 상호운용성
  - 기존 라이브러리를 그대로 사용할 수 있는가? 에 대한 질문에서 코틀린은 `그렇다` 라고 대답할 수 있습니다.
  - 자바 코드에서 코틀린 코드로 바꾸는데 많은 노력이 필요하지 않는다는 장점!

<br>

## 요약

- 코틀린은 타입 추론을 지원하는 정적 타입 지정 언어!

- 코틀린은 객체지향과 함수형 프로그래밍 스타일을 모두 지원

- 코틀린은 서버 개발, 안드로이드에 잘 활용 가능!

- 코틀린은 무료며 오픈 소스!

- 코틀린은 실용적이며 안전하고, 간결하며 상호운용성이 좋습니다. 이는 NullPointerException과 같이 흔히 발생하는 오류를 방지하고, 읽기 쉽고 간결한 코드를 지원하면서 자바와 아무런 제약 없이 통합될 수 있는 언어를 만드는데 초점을 맞췄다는 뜻입니다.
