# 람다로 프로그래밍

## 람다식(lambda expression) 또는 람다
다른 함수에 넘길 수 있는 작은 코드 조각

## 람다 식과 멤버 참조
함수를 값처럼 다루는 접근 방법

### 람다 소개: 코드 블록을 함수 인자로 넘기기
람다를 하나뿐인 무명 객체 대신 사용할 수 있다.
```java
// 무명 내부 클래스로 리스너 구현 (java < 8)
button.setOnClickListener(new OnClickListener() {
    @Override
    public void onClick(View view) {
        /* do something */
    }
});

// 무명 내부 클래스로 리스너 구현 (java >= 8)
button.setOnClickListener(view -> { /* do something */ });
```
```kotlin
// 람다로 리스너 구현 (kotlin)
button.setOnClickListener { /* do something */ }
```

### 람다와 컬렉션
람다나 멤버 참조를 인자로 받는 함수를 통해 개선한 코드는 더 짧고 이해하기 쉽다.  
```kotlin
// 컬렉션 직접 검색
fun findTheOldest(people: List<Person>) {
    var maxAge = 0
    var theOldest: Person? = null
    for (person in people) {
        if (person.age > maxAge) {
            maxAge = person.age
            theOldest = person
        }
    }
    println(theOldest) // 
}

val people = listOf(Person("Alice", 29), Person("Bob", 31))
findTheOldest(people) // Person(name=Bob, age=31)
```
```kotlin
// 람다를 사용해 컬렉션 검색
println(people.maxBy { it.age }) // Person(name=Bob, age=31)
```
```kotlin
// 멤버 참조를 사용해 컬렉션 검색
people.maxBy(Person::age)
```

### 람다 식의 문법
1. `{ x: Int, y: Int -> x + y }`
2. 변수에 저장 가능
    ```kotlin
    val sum = { x: Int, y: Int -> x + y }
    println(sum(1,2)) // 3
    ```
3. 람다식 직접 호출
    ```kotlin
    { println(42) }() // 42, 읽기 어렵고 쓸모없다. 
    ```
4. `run`은 인자로 받은 람다를 실행해주는 함수
    ```kotlin
    run { println(42) }
    ```
5. 함수 호출시 맨 뒤에 있는 인자가 람다 식이라면 그 람다를 괄호 밖으로 꺼낼 수 있음
    ```kotlin
    people.maxBy() { p: Person -> p.age }
    ```
6. 람다가 어떤 함수의 유일한 인자이고 괄호뒤에 인자를 썼다면 호출시 괄호 생략 가능
    ```kotlin
    people.maxBy10 { p: Person -> p.age }
    ```

### 현재 영역에 있는 변수에 접근
메서드의 로컬 변수를 무명 내부 클래스에서 사용할 수 있다.
람다를 함수 안에서 정의하면 함수의 파라미터뿐 아니라 람다 정의의 앞에 선언된 로컬변수까지 람다에서 모두 사용할 수 있다.

코틀린 람다 안에서는 파이널 변수가 아닌 변수에 접근할 수 있고, 그 변수를 변경할 수도 있다. 
람다 안에서 사용하는 외부 변수를 `람다가 포획(capture)한 변수`라고 부른다.

#### 변경 가능한 변수 포획하기
```kotlin
class Ref<T>(var value: T)
>>> var counter = Ref(0)
>>> var inc = { counter.value++ }
```
안에 들어있는 원소는 변경 가능하더라도 배열이나 클래스의 인스턴스에 대한 참조를 final 로 만들면 포획이 가능하다.
람다가 파이널 변수를 포획하면 자바와 마찬가지로 그 변수의 값이 복사된다.
람다가 변경 가능한 변수를 포획하면 변수를 Ref 클래스 인스턴스에 넣는다. 
그 Ref 인스턴스에 대한 참조를 파이널로 만들면 쉽게 람다로 포획할 수 있고, 람다 안에서는 Ref 인스턴스의 필드를 변경할 수 있다.

### 멤버 참조
코틀린에서는 자바8과 마찬가지로 함수를 값으로 바꿀 수 있다. 이 때 이중콜론(::) 을 사용한다.
TODO(p210) 

## 컬렉션 함수형 API
컬렉션을 다루는 코틀린 표준 라이브러리를 살펴보자! 
코틀린 설계자가 발명한 함수는 없으니 익숙한 독자는 필요한 설명만 찾아봐도 된다. 

### 고차함수(HOF, High Order Function)
함수형 프로그래밍에서 람다나 다른 함수를 인자로 받거나 함수를 반환하는 함수

### filter
컬렉션을 이터레이션하면서 주어진 람다에 각 원소를 넘겨서 람다가 true 를 반환하는 원소만 모은다. 

### map
주어진 람다를 컬렉션의 각 원소에 적용한 결과를 모아서 새 컬렉션을 만든다. 

### all, any
컬렉션의 모든/어떤 원소가 주어진 조건을 만족하는지 판단

### count 
조건을 만족하는 원소의 개수
```kotlin
// 조건을 만족하는 모든 원소가 들어가는 중간 컬렉션이 생기므로 낭비. 
people.filter(canBeInClub27).size
```

### find (findOrNull)
조건을 만족하는 첫번째 원소

### groupBy
특성을 파라미터로 전달하면 컬렉션을 자동으로 구분해줌

### flatten
여러 리스트를 한 리스트로 모음

### flatMap
주어진 람다를 컬렉션의 모든 객체에 적용하고(map), 여러 리스트를 한 리스트로 한데 모음 (flatten)

## 지연 계산(lazy) 컬렉션 연산

### 컬렉션과 비교
컬렉션 함수는 결과 컬렉션을 즉시(eagerly) 생성한다. 
매 단계마다 계산 중간 결과를 새로운 컬렉션에 임시로 담는다. 

시퀀스를 사용하면 원소가 많은 경우 컬렉션을 사용하는 것 보다 성능이 눈에 띄게 좋아진다. 

### 시퀀스
시퀀스(Sequence)를 사용하면 중간 임시 컬렉션을 사용하지 않고도 컬렉션 연산을 연쇄할 수 있음

Sequence 안에는 iterator() 라는 단 하나의 메소드가 있다. 이 메소드를 통해 시퀀스로부터 원소 값을 얻을 수 있다.

시퀀스의 원소를 인덱스를 사용해 접근하는 등의 다른 API 메소드가 필요하다면 시퀀스를 리스트로 변환해야 한다.

```kotlin
people.asSequence() // 원본 컬렉션을 시퀀스로 변환
    .map(Person::name) // 시퀀스도 컬렉션과 동일한 API 제공
    .filter { it.startsWith("A") }
    .toList() // 결과 시퀀스를 다시 리스트로 반환
```

### 시퀀스 연산 실행: 중간 연산과 최종 연산
중간 연산(intermediate)과 최종 연산(terminal)으로 나뉜다.

중간 연산은 다른 시퀀스를 반환한다. 

최종 연산은 결과를 반환한다.

```kotlin
// 최종 연산이 없는 예제. 
// 아무것도 출력되지 않는다.  
listOf(1, 2, 3, 4).asSequence()
    .map { print("map($it) "); it * it }
    .filter { print("filter($it) "); it % 2 == 0}
```
```kotlin
// 최종 연산이 있는 예제.
// 연기됐던 모든 연산이 수행된다. 
listOf(1, 2, 3, 4).asSequence()
    .map { print("map($it) "); it * it }
    .filter { print("filter($it) "); it % 2 == 0}
    .toList()
```

#### 실행 순서 차이
##### 컬렉션
map(1), map(2), map(3), map(4), filter(1), filter(4), ...
##### 시퀀스 : 각 원소에 대해 순차적으로 적용
map(1), filter(1), map(2), filter(4), map(3), filter(9), ...

### 시퀀스 만들기 (generateSequence)
```kotlin
// 자연수의 시퀀스를 생성하고 사용하기
val naturalNumber = generateSequence(0) { it + 1 }
val numbersTo100 = naturalNumbers.takeWhile { it <= 100 }
println(numbersTo100.sun()) // 5050
```

```kotlin
// 객체의 조상으로 이뤄진 시퀀스
fun File.isInsideHiddenDirectory() = generateSequence(this) { it.parentFile }.any { it.isHidden } // any 를 find 로 바꾸면 원하는 디렉터리를 찾을 수도 있음
val file = File("/Users/svtk/.HiddenDir/a.txt")
println(file.isInsideHiddenDirectory()) // true
```

## 자바 함수형 인터페이스 활용
함수형 인터페이스(functional interface) 또는 SAM(Single Abstract Method) 인터페이스: 추상 메서드가 단 하나만 있는 인터페이스. ex) Runnable, Callable

코틀린에서 함수를 인자로 받을 필요가 있는 함수는 함수형 인터페이스가 아니라 함수 타입을 인자 타입으로 사용해야 한다. 

### 자바 메소드에 람다를 인자로 전달
```java
void postponeComputation(int delay, Runnable computation);
```
```kotlin
// 코틀린 컴파일러가 자동으로 람다를 Runnable 인스턴스로 변환해줌
postponeComputation(1000) { println(42) }
```

```kotlin
// Runnable 을 구현하는 무명 객체를 명시적으로 만들어서 사용할 수도 있다. 
postponeComputation(1000, object : Runnable { // 객체 식을 함수형 인터페이스 구현으로 넘긴다. 
    override fun run() {
        println(42)
    }
})
```

객체를 명시적으로 선언하는 경우 메소드를 호출할 때마다 새로운 객체가 생성된다. (무명객체)
정의가 들어있는 함수의 변수에 접근하지 않는 람다에 대응하는 무명 객체를 메소드를 호출할 때마다 반복 사용한다. (람다)

```
But there’s a difference. 
When you explicitly declare an object, a new instance is created on each invocation. 
With a lambda, the situation is different: 
if the lambda doesn’t access any variables from the function where it’s defined, 
the corresponding anonymous class instance is reused between calls
```

```kotlin
// 프로그램 전체에서 Runnable 의 인스턴스는 단 하나만 만들어진다. 
postponeComputation(1000) { println(42) }
```

```kotlin
// 명시적인 object 선언을 사용하면서 람다와 동일한 코드
val runnable = Runnable { println(42) }
fun handleComputation() {
    postponeComputation(1000, runnable)
}
```

```kotlin
// 람다가 주변 변수를 포획하면 같은 인스턴스를 사용할 수 없으므로, 컴파일러가 매번 새로운 인스턴스 생성
fun handleComputation(id: String) { // 람다 안에서 id capture
    postponeComputation(1000) { println(id) } // 호출할 때마다 새로 Runnable 인스턴스를 만든다
}
```

### SAM 생성자: 람다를 함수형 인터페이스로 명시적으로 변경
SAM 생성자는 람다를 `함수형 인터페이스의 인스턴스`로 변환할 수 있게 컴파일러가 자동으로 생성한 함수다.
#### SAM 생성자를 사용하기
1. 컴파일러가 자동으로 바꾸지 못하는 경우
2. 람다로 생성한 함수형 인터페이스 인스턴스를 변수에 저장해야 하는 경우
3. 오버로드 한 메소드중에서 어떤 타입의 메소드를 선택해 람다를 변환해 넘겨줘야 할지 모호한 때
```kotlin
// SAM 생성자를 사용해 값 반환
fun createdAllDoneRunnable(): Runnable {
    return Runnable { println("All done!") } 
}
createdAllDoneRunnable().run() // All done!
```
```kotlin
// SAM 생성자를 사용해 listener 인스턴스 재사용 
val listener = OnClickListener { view -> 
    val text = when (view.id) {
        R.id.button1 -> "First button"
        R.id.button2 -> "Second button"
        else -> "Unknown button"
    }
    toast(text)
}
button1.setOnClickListener(listener)
button2.setOnClickListener(listener)
```

## 수신 객체 지정 람다
### 수신 객체 지정 람다(lambda with receiver)
수신 객체를 명시하지 않고 람다의 본문 안에서 다른 객체의 메소드를 호출할 수 있게 하는 것

### with
어떤 객체의 이름을 반복하지 않고도 그 객체에 대해 다양한 연산을 수행하면 좋겠다.

람다를 괄호 밖으로 빼내는 관계를 사용함에 따라 전체 함수 호출이 언어가 제공하는 특별 구문처럼 보인다.

with 는 첫 번째 인자로 받은 객체를 두 번째 인자로 받은 람다의 수신 객체로 만든다.

with 의 반환값은 람다 코드를 실행한 결과값이다. 

확장함수의 this 가 확장하는 타입의 인스턴스를 가리키는 것과 비슷하다. 

```kotlin
// without 'with'
fun alphabet(): String {
    val result = StringBuilder()
    for (letter in 'A'..'Z') {
        result.append(letter)
    }
    result.append("\nNow I Know the alphabet!")
    return result.toString()
}
```
```kotlin
// with 'with'
fun alphabet(): String {
    val stringBuilder = StringBuilder()
    return with(stringBuilder) {
        for (letter in 'A'..'Z') {
            this.append(letter)
        }
        append("\nNow I Know the alphabet!")
        this.toString()
    }
}
```
```kotlin
fun alphabet() = with(StringBuilder()) {
    for (letter in 'A'..'Z') {
        append(letter)
    }
    append("\nNow I Know the alphabet!")
}
```
```kotlin
// this 참조에 레이블을 붙여서 특정 메서드 지정함으로써 이름 충돌 해결 
this@OuterClass.toString()
```

### apply
람다의 결과 대신 수신 객체가 필요한 경우 사용한다.

with 와 거의 같지만, 유일한 차이는 apply 는 항상 자신에게 전달된 객체(수신객체) 를 반환한다는 것 뿐이다. 

객체의 인스턴스를 만들면서 즉시 프로퍼티 중 일부를 초기화해야 하는 경우 유용하다.

```kotlin
// with apply
fun alphabet(): StringBuilder().apply {
    for (letter in 'A'..'Z') {
        append(letter)
    }
    append("\nNow I Know the alphabet!")
}.toString()
```
```kotlin
// apply 를 텍스트뷰 초기화에 사용하기
fun createViewWithCustomAttributes(context: Context) = 
    TextView(context).apply() {
        text = "Sample Text"
        textSize = 20.0
        setPadding(10, 0, 0, 0)
    }
```
```kotlin
// buildString : StringBuilder 객체를 만들고, toString 을 호출하는 일을 알아서해준다. 
fun alphabet() = buildString {
    for (letter in 'A'..'Z') {
        append(letter)
    }
    append("\nNow I Know the alphabet!")
}
```
https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.text/build-string.html

