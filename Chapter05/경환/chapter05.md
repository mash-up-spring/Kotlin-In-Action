# 5. 람다로 프로그래밍

- 람다는 코드 조각이다.
- 코틀린 표준 라이브러리는 람다를 많이 사용한다.
- Lambda with Receiver: 람다 선언을 둘러싸고 있는 환경과는 다른 환경에서 람다 본문을 실행할 수 있다.



## 5.1 람다 식과 멤버 참조



### 5.1.1 람다 소개: 코드 블록을 함수 인자로 넘기기

- 객체 간 데이터뿐만 아니라 "무엇을 할 지"를 전달하기 위해 람다가 필요하다.

- 람다는 Functional Interface 구현의 다른 형태이다.

- ```java
  button.setOnclickListner{/* 클릭 시 수행할 동작*/}
  ```



### 5.1.2 람다와 컬렉션

- 람다를 이용해서 외부 반복을 내부 반복으로 바꿀 수 있다.

- ```kotlin
  people.maxBy{it.age} // 컬렉션에 무엇을 기준으로 찾을지를 제공하는 프로듀서를 넘긴다.
  people.maxBy{Person::age} // 
  ```



### 5.1.3 람다식의 문법

- ```kotlin
  val sum = {x: Int, y: Int -> x+y} // 람다는 값
  println(sum(1,2)) // 람다 뒤에 괄호로 실행할 수 있다.
  run{println(42)} // run: 람다식을 실행하는 함수
  ```

- x, y : 파라미터, x+y : 본문

- 람다식은 중괄호로 둘러쌓여 있다.

- 람다는 인라인되어 함수 호출에 필요한 비용이 들지 않는다.

- 람다식 줄여쓰기

- ```kotlin
  people.maxBy({p:Person -> p.age}) // maxBy는 funtion 타입 람다식을 인자로 받는다.
  people.maxBy(){p:Person -> p.age} // 맨 뒤 인자가 람다식이면 괄호 밖으로 꺼낼 수 있다.
  people.maxBy{p->p.age} // 빈괄호, 타입 생략
  peol.maxBy{it.age} // 파라미터가 하나뿐이고 추론 가능하다면 it를 바로 쓸 수 있다.
  ```

- 필요에 따라 일부 타입만 표시해주면 된다.

- it는 관습적으로 사용하는 파라미터 이름이다.

- 람다가 중첩된 경우 명시적으로 파라미터를 써주자.

- 람다를 변수에 저장할 경우 컬렉션과는 다르게 문맥이 없으므로 타입을 명시해야 한다.

- ```kotlin
  val getAge = {p:Person -> p.age}
  ```

- 본문이 여러 줄로 되어 있는 경우 본문의 맨 마지막 식이 람다의 결과이다.

- ```kotlin
  val sum = {x: Int, y: Int ->
  	println("Computing the sum of $x and $y...")
  	x+y // 람다식의 결과
  }
  ```



### 5.1.4 람다식에서 로컬 변수에 접근하기



- 람다를 함수 안에서 정의하면 파라미터와 로컬 변수를 사용할 수 있다.
- 람다는 별도 쓰레드에서 실행 가능하다.
- 로컬 변수는 스택에 저장되고, 쓰레드는 각자 다른 스택을 사용하기 때문에 람다와 로컬 변수는 사용하는 스택이 다를 수 있다.
- 따라서 람다가 사용하는 로컬 변수는 final 한 성격이어야한다.
- 코틀린은 람다가 사용하는 로컬 변수인 포획 변수를 final 객체로 래핑해서 사용한다.
- 래퍼 객체의 참조를 람다와 함께 저장한다.



- 람다가 비동기로 실행되는 경우 문제점

- ```kotlin
  fun tryAsync(button: Button): Int {
    val clickCount = 0
    button.onClick{clickCount++}
    return clickCount
  }
  ```

- clickCount는 비동기적으로 더해지므로 tryAsync의 결과는 0이다.



### 5.1.5 멤버 참조

- 멤버 참조: 메서드를 값으로 나타내기 (함수 포인터?)

- ```kotlin
  val getAge = Person::age
  					 = {person: Person -> person.age}
  ```

- 콜론 두개 :: 를 사용한다.

- 멤버 참조 뒤에는 괄호를 넣으면 안된다.

- 멤버 참조는 그 멤버를 호출하는 람다와 같은 타입이다.

- 확장 함수도 같은 방식, ::로 참조하면 된다.

- 최상위 함수 참조하기

- ```kotlin
  fun salute() = println("Salute!")
  run(::salute)
  ```

- 단순히 위임만 하는 람다의 경우 람다를 만들지 말고 위임 함수에대한 참조를 제공하면 된다.

- 생성자를 참조하는 객체를 만들어서 클래스 생성 작업을 저장하거나 연기할 수 있다.

- 바운드 멤버 참조: 멤버 참조를 생성할 때 인스턴스를 함께 저장한다.

- ```kotlin
  val p = Person(...)
  val bound = p::age
  println(bound())
  >> 인스턴스 p의 age 출력
  ```



## 5.2 컬렉션 함수형 API



### 5.2.1 filter와 map

- filter

- ```kotlin
  people.filter{it.age > 30} // predicate를 넣는다.
  ```

- map

- ```kotlin
  list.map{it*it}
  people.map(Person::name)
  people.filter{it.age>30}.map(Person::age)
  ```

- 자료구조 Map에 필터와 맵을 사용할 때 filterKeys 나 mapValues 등을 이용해서 특정 엔트리를 걸러내거나 변환할 수 있다.



### 5.2.2 all, any, count, find

- all: 모든 원소가 인자로 받은 predicate를 만족하는지

- any: 인자로 받은 predicate를 하나라도 만족하는 원소가 있는지

- count: predicate를 만족하는 원소의 개수 반환

- ```kotlin
  val canBeInClub27 = {p: Person -> p.age <= 27}
  println(people.any(canBeInClub27))
  >> true or false
  ```

- find: predicate를 만족하는 원소 중 첫번째꺼, 없으면 null

- filter + size를 쓰면 임시 컬렉션이 필요하지만 count는 그렇지 않다.



### 5.2.3 groupBy

- SQL의 groupBy로 동일하다.

- 무엇을 기준으로 묶을지를 Function 타입으로 넘겨준다.

- ```kotlin
  val list = listOf("a", "ab", "b")
  println(list.groupBy(String::first))
  >>{a=[a,b], b=[b]}
  ```

- 결과는 map 타입이다.



### 5.2.4 flatMap, flatten

- flatmap

- ```kotlin
  books.flatMap{it.authors}.toSet()
  ```

- books: list, authours: list -> 여러 리스트를 하나의 리스트로 합친다.

- flatten: book -> book.authors 와 같이 따로 매핑이 필요하지 않는 경우



## 5.3 지연 계산(lazy) 컬렉션 연산



- 코틀린 시퀀스 == 자바 스트림
- 시퀀스를 사용하면 중간 임시 컬렉션을 만들지 않고 컬렉션 연산을 수행할 수 있다.
- 시퀀스를 안쓸때: 연산 하나하나마다 전체 원소에 대해 전부 수행
- 시퀀스 쓰면: 원소 하나하나 모든 연산 수행
- 모든 연산을 한번에 하므로 중간 연산자는 최종 연산자까지 지연된다.



### 5.3.1 중간 연산자, 최종 연산자

- map, filter, .. : 중간 연산자.

- toList, ... : 최종 연산자.

- 컬렉션의 각 원소는 각 원소마다 계산이 한번에 이루어지므로 중간 연산자는 최종 연산자 전까지 미뤄진다.

- 즉시계산은 모든 원소에 대한 중간계산 결과를 저장할 버퍼가 필요하므로 메모리 사용이 더 많다.

- filter 다음 map을 쓰면 map 다음 filter 쓰는 것보다 연산량이 줄어든다.

- ```kotlin
  people.asSequence().filter{it.name.length < 4}.map(Person::name).toList()
  ```



### 5.3.2 시퀀스 만들기

- generateSequence(): 이전의 원소를 받아서 다음 원소를 계산하는 시퀀스

- 첫째항을 첫번째 인자로, 점화식을 람다식으로 건넨다.

- ```kotlin
  val naturalNumbers = generateSequence(0){it+1} // 0,1,2,3,4,.... (lazy)
  val numberTo100 = naturalNumbers.takeWhile{it<=100} // takeWhile: filter 조건
  println(nummberTo100.sum()) // 최종연산자
  ```



## 5.4 자바 함수형 인터페이스 활용



- 코틀린 람다는 자바 Functional Interface와 아주 잘 호환된다.



### 5.4.1 자바 메소드에 람다를 인자로 전달하기

- 람다를 인자로 전달하면 실행 시 싱글턴 객체가 만들어진다.
- 외부에서 val로 만들어서 참조를 전달하는 것과 동일하다.
- 익명 객체를 만들어서 전달하면 매번 객체가 생성된다.
- 람다가 포획변수가 있다면 매번 새로운 객체를 생성한다.
- inline이 붙어있는 함수에 람다를 넘기면 객체가 생성되지 않고 인라인된다.



### 5.4.2 SAM 생성자 : 람다를 객체로 만들기

- SAM: Single Abstract Method

- SAM 생성자는 람다를 함수형 인터페이스의 인스턴스로 변환할 수 있게 컴파일러가 자동으로 생성한 함수다.

- ```kotlin
  fun createAllDoneRunnable() : Runnable{
  		return Runnable{println("All Done!")} // Runnalbe 객체 생성
  }
  ```

- 람다를 인스턴스 변수에 저장하거나 재사용 될 때 등에 사용한다.

- 람다가 자기 자신을 가리키기 위해서 this를 사용할 수 없으므로 this를 사용할 수 있는 익명 객체를 사용하기 위해 SAM을 쓴다.

- SAM을 파라미터로 받는 람다는 메서드를 호출할때 SAM으로 자동 변환되지만 잘 안되면 명시적으로 생성자로 변경해주자.



## 5.5 수신객체지정람다(Lambda with receiver) : with 와 apply



- 수신 객체를 명시하지 않고 람다 본문에서 다른 객체의 메서드를 사용할 수 있다.

- with와 apply는 둘 다 라이브러리 함수다.

- ```kotlin
  fun alphabet() = with(StringBuilder()){ // 객체의 참조를 넘길 수 있다.
    for(letter in 'A'..'Z'){
      this.append(letter)//this는 생략 가능
    }
    append("...")
    toString()
  }
  ```

- 리뷰: 람다는 마지막줄이 리턴값이다.

- apply: 수신 객체를 받아서 처리하고 수신 객체를 그대로 반환한다.

- ```kotlin
  fun alphabet() = StringBuilder().apply{
  	for(letter in 'A'..'Z'){
  		append(letter)
  	}
  }.toString() // apply의 리턴값은 StringBuilder 객체(수신객체)
  
  fun alphabet = buildString{ // buildString: Stringbuilder + toString
  	for(letter in 'A'..'Z'){
  		append(letter)
  	}
  }
  ```





# 요약

- 람다를 사용하면 코드 조각을 다른 함수에게 인자로 넘길 수 있다.
-  코틀린에서는 람다가 함수 인자인 경우 괄호 밖으로 빼낼 수 있고다
- 람다의 인자가 하나인 경우 이름을 지정하지 않고 it라는 디폴트 이름으로 부를 수 있다.
- 람다 안에 있는 코드는 람다가 들어있는 바깥 함수의 변수를 읽거나 쓸 수 있다.
- 메서드, 생성자, 프로퍼티의 앞에 ::를 붙이면 각각에 대한 참조를 만들 수 있다. 그런 참조를 람다 대신 다른 함수에게 넘길 수 있다.
- filter, map, all, any로 컬렉션에 대해 내부 반복을 수행할 수 있다.
- 시퀀스를 이용해 중간 결과를 담는 컬렉션을 생성하지 않고 컬렉션에 대한 여러 연산을 조합할 수 있다.
- 함수형 인터페이스를 인자로 받는 자바 함수를 호출할 경우 람다를 그대로 넘길 수 있다.
- 수신객체지정람다를 사용하면 람다 안에서 미리 정해둔 수신 객체의 메소드를 직접 호출할 수 있다.
- with나 apply를 쓰면 빌더처럼 초기화하는데도 사용할 수 있다.
- 아무튼 람다 좋음 끝

