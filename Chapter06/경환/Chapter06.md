# 6. 코틀린 타입 시스템



### Preview

```
- 널이 될 수 있는 타입과 그 문법
- 원시 타입
- 컬렉션 소개
```





## 6.1 널 가능성

- 객체는 동적으로 생성되는 경우가 많기 때문에 NPE는 가장 흔한 오류 중 하나.
- 그렇다면 Null 체크를 컴파일 타임에 강제해보자. 그리고 널 체크 문법을 좀 더 편하게 해보자.



### 6.1.1 널이 될 수 있는 타입

```java
int strLen(String s){
	return s.length();
}

strLen(null);
```

- strLen에는 null이 들어올 수 도 있고 아닐 수도 있다.
- 첫번째로 타입을 좀 더 명확히 구분짓자.
- null+Type(널을 담을 수 있음), Type(널을 담을 수 없다.)
- 전자: Nullable, Type?으로 표현한다.

- Nullable한 타입은 그대로 쓸 수 없게 하면된다.

### 6.1.3 안전한 호출 연산자: ?.

```kotlin
fun strLenNPESafe(s: String?) = s?.length()

val country: String? = this.company?.address?.country
```

- 함수의 파라미터를 null을 담을 수 있는 타입으로 바꿨다.
- java의 옵셔널과 같이 그대로 쓸 수 없고 컴파일타임에 null체크를 강제한다.
- ?.연산자는 null이 아닐경우 그대로 메서드를 실행하고, Null일 경우는 null을 반환한다.

- 체이닝해서 사용하면 편하다.

  

### 6.1.4 엘비스 연산자: ?:

```kotlin
val address  = person.company?.address?: throw Exception()
val address  = person.company?.address?: "Unknown"
```

- Null이 아니면 그대로 가져다 쓰고 아니면 뒤에 식을 가져다 쓴다.
- return이나 throw도 식이므로 사용 가능

- 위의 예시에서 address가 널이면 NPE 대신 다른 로직을 수행할 수 있다.



### 6.1.5 안전한 캐스트: as?

- 캐스팅 할 수 있으면 캐스팅하고 안되면 null 반환
- null을 반환하므로 ?. 나 ?: 쓰면 좋다.

```kotlin
val ohterPerson = o as? Person ?: return false
```



### 6.1.6 널 아님 단언: !!

- 널이 될 수 있는 타입을 널이 될 수 없는 타입으로 강제로 바꾼다.
- 만약 null에 대해 !!로 널이 될 수 없는 타입으로 바꾼다면 NPE 발생
- 하지만 메서드 호출 시가 아닌 단언 시에 NPE 나므로 디버깅하기 쉽다.
- 디버깅 편의상 여러 줄에 같이 쓰지 말자.

```kotlin
val s: String? = "hi"
val sNotNull: String = s!!
```



### 6.1.7 let 함수

- nullable 타입을 함수에 파라미터로 넘길 때
- null이 아닐때만 함수에 넘겨서 실행하고 null이면 아무 일도 일어나지 않는다.

```kotlin
email?.let{sendEmailTo(it)}
// email이 널일 경우: null.let -> 아무일도 X
// email이 널이 아닐 경우: email.let{sendEmail(it)} -> 메서드 실행
```



### 6.1.8 lateinit

- DI 사용하면 나중에 값이 설정되는데
- 그 잠깐 때문에 Nullabe Type으로 선언하기 싫다.
- 나중에 초기화된다고 선언하고, 널이 될 수 없는 타입으로 쓰자.
- 항상 var 이어야 한다.

```kotlin
private lateinit var myService: MyService
```



### 6.1.9 널이 될 수 있는 타입 확장

- 확장함수를 쓰면 수신객체가 널인지 아닌지 체크할 수 있다.(확장함수는 최상위함수이므로)
- 멤버함수로는 당연히 그럴 수 없다.(null값을 참조하므로)

```kotlin
fun String?.isNullOfBlank(): Boolean = this == null || this.isBlank()
```



### 6.1.10 타입 파라미터의 널 가능성

- 타입 파라미터: 제네릭
- T는 ?가 없더라도 널이 될 수 있는 타입니다.
- 널이 될 수 없는 타입으로 강제하려면 제네릭 정의시 <T: Any>로 써야한다.



### 6.1.11 널 가능성과 자바

- 자바 타입은 널이 될 수도 있다.
- Null관련 어노테이션이 있으면 그 어노테이션을 따르지만
- 그렇지 않다면 자바 타입을 코틀린에서 쓸 때 Type?으로 해야할 까? Type으로 해야할까?
- 플랫폼타입: 널관련 정보를 알 수 없는 타입, 모든 연산 수행 가능
- 자바 타입 쓸때는 널 검사 유의해야한다.

```kotlin
//Person: 자바 클래스
fun yellAtSafe(p: Person){
	println((person.name?: "Anyone").toUpperCase())
}
```

- 플랫폼 타입을 도입한 이유: 위험하지만 실용성을 위해서
- 자바 메서드 오버라이드 할 때 널이 될수 있는 타입으로 오버라이딩할지 말지를 잘 고려하자.



## 6.2 코틀린의 원시 타입



### 6.2.1 원시타입: Int, Boolean 등

- 컴파일시 원시타입으로 쓸지 말지는 코틀린이 알아서 해준다.
- 코틀린은 원시 타입과 래퍼 타입을 구분하지 않는다.

### 6.2.2 널이 될 수 있는 타입

- 자바 원시타입에는 null이 들어갈 수 없으므로 래퍼 클래스로 컴파일된다.
- 제네릭은 원시타입 허용하지 않는다.(제네릭은 타입캐스팅으로 구현되므로)

### 6.2.3 숫자변환

- 숫자 변환 (Int <-> Long)은 명시적으로 변환하자.

```kotlin
Int.toLong(), Long.toInt()
```

- 박스타입간 equals는 그 안의 값이 아닌 박스 객체를 비교한다.
- (new Long(42)).equals(new Integer(42)) -> false
- 1L, 1f, 0x,0b 등을 써서 명시적으로 숫자의 타입을 지정할 수 있다.

### 6.2.4 Any, Any?: 최상위 타입

- Any: Object와 같은 최상위 타입, Int의 최상위 타입도 Any 이다.

### 6.2.5 Unit 타입

- void에 해당. void는 키워드이나 Unit은 타입
- 따라서 제네릭으로 쓸 수 있음

```kotlin
interface Processor<T> {
	fun Process(): T
}
class ~ : Processer<Unit>{
  ~
}
```

### 6.2.6 Nothing 타입

- 이 함수는 절대 정상적으로 끝나지 않는단는 타입
- 반환 타입에만 사용한다.
- 무한루프나 throw하는 함수에 사용





## 6.3 컬렉션과 배열



### 6.3.1 널 가능성과 컬렉션

- 컬렉션을 널이 될 수 있는 타입으로 선언할 수도 있다.
- 컬렉션의 원소를 널이 될 수 있는 타입으로 선언할 수 있다.

```
List<Int?> : 컬렉션은 널이 아니고 원소가 널일 수 있음
List<Int>? : 컬렉션은 널일 수 있고 원소는 널이 아니다.
```



### 6.3.2 읽기 전용과 변경 가능한 컬렉션

- 코틀린에서 컬렉션 인터페이스는 Read only이다.
- 변경 가능한 컬렉션 인터페이스인 MutableCollections는 컬렉션을 상속받는다.
- 변경 불가능 컬렉션 타입의 참조가 변경 가능한 컬렉션 타입을 가르키고 있을 수 있으므로
- 참조가 Read only라고 스레드 세이프하지는 않다.



### 6.3.3 코틀린 컬렉션과 자바

- 자바 컬렉션 구조를 그대로 사용했다.

- 컬렉션 생성 함수
  - 타입   변경x       변경o
  - List:   [listOf],     [mutableListOf, arrayListsOf]
  - Set:   [setOf],      [mutableSetOf, hashSetOf, linkedSetOf, sortedSetOf]
  - Map: [mapOf],   [mutableMapOf, hashMapOf, linkedMapOf, sortedMapOf]
- 자바는 읽기 전용 구분하지 않으므로 자바와 같이 쓸 때는 read only라도 변경될 수 있다.



### 6.3.4 컬렉션을 플랫폼 타입으로 다루기

- 자바에 컬렉션 파라미터가 있는 인터페이스를 코틀린으로 구현할 때
- 코드의 의미를 파악해서 Nullable하게 할지 안할지를 신중하게 구현해야 한다.

```java
/* 자바 */
interface FileContentProcessor {
	void processContents(File path, byte[] binaryContents, List<String> textContents);
}
```



### 6.3.5 객체의 배열과 원시 타입의 배열

- 코틀린 배열은 타입 파라미터를 받는 클래스
- arrayOf로 생성 가능
- arrayOfNulls(size)로 size 만큼 null로 채워진 배열을 만들어준다.
- Array 생성자는 배열 크기와 람다로 배열 원소를 초기화해준다.
- toTypedArray로 컬렉션을 배열로 만들 수 있다.
- 원시 타입 배열은 IntArray처럼 별도 클래스를 제공해준다.

```kotlin
val squares = IntArray(5){i -> (i+1)*(i+1)}
//1,4,9,16,25
```





## 6.4 요약

- NPE를 컴파일 타입에 방지할 수 있다.
- ?. ?: !! as? let 등을 잘 사용하자.
- 자바 타입은 플랫폼 타입으로 취급된다.
- 원시 타입을 따로 구분하지 않고 알아서 잘 해준다.
- Any, Nothing ,Unit
- 컬렉션은 읽기전용과 변경 가능으로 나뉜다.
- 자바와 같이 쓸때는 특히 null 관련해서 조심하자
- 배열은 타입 파라미터를 받는 클래스이다.