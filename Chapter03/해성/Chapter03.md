# 3 함수 정의와 호출

## 다루는 내용
함수 정의와 호출 기능을 코틀린이 어떻게 개선했는지!

## 3.1 코틀린에서 컬렉션 만들기

코틀린에서 기존 자바 컬렉션을 사용할 수 있다
```text
>>> println(set.javaClass)
class java.util.HashSet
>>> println(list.javaClass)
class java.util.ArrayList
>>> println(map.javaClass)
class java.util.HashMap
```

코틀린에서는 자바보다 더 많은 기능을 쓸 수 있다
```
>>> val strings = listOf("first", "second", "fourteenth")
>>> println(strings.last())
fourteenth
>>> val numbers = set(1,14,2)
println(numbers.max())
14
```

## 3.2 함수를 호출하기 쉽게 만들기

함수를 호출할 때마다 매번 네 인자를 모두 전달해야해서 번거롭다
```kotlin
fun <T> joinToString(
        collection: Collection<T>,
        separator: String,
        prefix: String,
        postfix: String
): String {

    val result = StringBuilder(prefix)

    for ((index, element) in collection.withIndex()) {
        if (index > 0) result.append(separator)
        result.append(element)
    }

    result.append(postfix)
    return result.toString()
}
```
함수에 전달하는 인자 중 일부 (또는 전부)의 이름을 명시할 수 있다.
```java
// 자바
joinToString(collection, /* separator */ " ", /* prefix */ " ", 
        /* postfix */ ".");
```
```kotlin
// 코틀린
joinToString(collection, separator = " ", prefix = " ", postfix = ".")

```

함수 선언에서 파라미터의 디폴트 값을 지정할 수 있으므로 오버로드한 메서드를 줄일 수 있다
```kotlin
fun <T> joinToString(
        collection: Collection<T>,
        separator: String = ", ",
        prefix: String = "",
        postfix: String = ""
): String {
    // 생략
}
```

최상위 함수
코틀린 컴파일러가 생성하는 클래스의 이름은 최상위 함수가 들어있던 코틀린소스 파일의 이름과 대응된다.
```kotlin
package strings

fun <T> joinToString(
        collection: Collection<T>,
        separator: String = ", ",
        prefix: String = "",
        postfix: String = ""
): String {
    // 생략
}
```
```java
import strings.JoinKt;
...
JoinKt.joinToString(list, ", ", "", "");
```

함수와 마찬가지로 프로퍼티도 파일의 최상위 수준에 놓을 수 있다
```kotlin
var opCount = 0
fun performOperation() {
    opCount++
}
fun reportOperationCount() {
    println("Operation performed $opCount times")
}
```

## 3.3 메소드를 다른 클래스에 추가: 확장 함수와 확장 프로퍼티
확장함수는 어떤 클래스의 멤버 메소드인 것처럼 호출할 수 있지만 그 클래스의 밖에 선언된 함수이다.

확장함수 본문에도 this 를 쓰거나 생략 할 수 있다

확장함수 안에서는 클래스 안에서 정의한 메소드와 다르게 클래스 내부에서만 사용할 수 있는 private 멤버나 protected 멤버를 사용할 수 없다.


- 수신 객체 타입(receiver type) : 확장할 클래스의 이름 (String)
- 수신 객체(receiver object): 호출되는 대상이 되는 값 (this)
```kotlin
fun String.lastChar(): Char = this.get(this.length - 1)
```
```kotlin
// String 이 수신 객체 타입이고 "kotlin" 이 수신 객체이다. 
>>> println("Kotlin".lastChar())
n
```
```kotlin
TODO()
```

## 3.4 컬렉션 처리: 가변 길이 인자, 중위 함수 호출, 라이브러리 지원
```kotlin
TODO()
```

## 3.5 문자열과 정규식 다루기
```kotlin
TODO()
```

## 3.6 코드 다듬기: 로컬 함수와 확장
DRY 원칙 (DRY, Don't Repeat Yourself)
코드 중복을 로컬 함수를 통해 제거할 수 있다.

### 3.11 코드 중복을 보여주는 예제
```kotlin
class User(val id: Int, val name: String, val address: String)

fun saveUser(user: User) {
    if (user.name.isEmpty()) {
        throw IllegalArgumentException(
            "Can't save user ${user.id}: empty Name")
    }

    if (user.address.isEmpty()) {
        throw IllegalArgumentException(
            "Can't save user ${user.id}: empty Address")
    }

    // Save user to the database
}
```

### 3.12 로컬 함수를 통해 코드 중복 줄이기
```kotlin
class User(val id: Int, val name: String, val address: String)

fun saveUser(user: User) {

    fun validate(user: User,
                 value: String,
                 fieldName: String) {
        if (value.isEmpty()) {
            throw IllegalArgumentException(
                "Can't save user ${user.id}: empty $fieldName")
        }
    }

    validate(user, user.name, "Name")
    validate(user, user.address, "Address")

    // Save user to the database
}
```

### 3.13 로컬 함수에서 바깥 함수의 파라미터 접근하기
```kotlin
class User(val id: Int, val name: String, val address: String)

fun saveUser(user: User) {
    fun validate(value: String, fieldName: String) {
        if (value.isEmpty()) {
            throw IllegalArgumentException(
                "Can't save user ${user.id}: " +
                    "empty $fieldName")
        }
    }

    validate(user.name, "Name")
    validate(user.address, "Address")

    // Save user to the database
}
```
### 3.14 검증 로직을 확장 함수로 추출하기
```kotlin
class User(val id: Int, val name: String, val address: String)

fun User.validateBeforeSave() {
    fun validate(value: String, fieldName: String) {
        if (value.isEmpty()) {
            throw IllegalArgumentException(
               "Can't save user $id: empty $fieldName")
        }
    }

    validate(name, "Name")
    validate(address, "Address")
}

fun saveUser(user: User) {
    user.validateBeforeSave()

    // Save user to the database
}
```
