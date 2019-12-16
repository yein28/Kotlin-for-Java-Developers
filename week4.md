# Week 4 Properties, OOP, Conventions

## Properties

### [Properties](https://kotlinlang.org/docs/reference/properties.html#properties-and-fields)

Java에서 field와  getter, setter를 통해 property 컨셉을 구현할 수 있음

kotlin의 property는 완벽히 똑같은 아이디어와 컨셉이지만 문법이 다름

```kotlin
class KotlinClass {
  var foo = 0
}
```

Define the property iteself rather than separately fields and accessors.

property = field + accessor(s) -> 필드와 접근자를 통칭

val = field + getter

var = fiend + getter + setter

코틀린에서 property를 사용할땐 getter나 setter를 호출할 필요없으며 변수에 접근하는 것처럼 바로 property를 사용하면 됨



Property를 field 없이 정의할 수 있음

```kotlin
class Rectangle(val height: Int, val width: Int) {
  val isSquare: Boolean
  	// field가 없음 
  	get() { // custom getter, 접근할 때마다 계산
      return height == width
    }
}
```



**Fields**

코틀린에서는 field를 직접적으로 사용할 필요 없이 properties를 사용하면 됨

하지만 필요하다면, property accessor 내에서 `field` 키워드를 통해 접근할 수 있으며 이는 클래스내 다른 메서드에게는 보이지 않음

```kotlin
class StateLogger {
  var state = false
  	set(value) {
      println("$field -> $value")
      field = value
    }
}
```



클래스 외부, 내부에서는 필드의 getter와 setter에는 직접적으로 접근할 수 없으며, 프로퍼티를 통해서만 사용할 수 있음

`println(lengthCounter.count)`는 내부적으로는 `lengtCounter.getCounter`를 호출



만약 var mutable 프로퍼티를 클래스 외부에서는 오직 read-only 프로퍼티로만 접근되도록 하려는 경우에는 setter를 private으로 만들 수 있음

```kotlin
class LengthCounter { 
	var counter: Int = 0
		// setter는 default implementation이며 private임
  	private set // getter는 어디서든지 접근할 수 있지만 setter는 private
  // 동일 클래스 내에서만 해당 프로퍼티를 수정할 수 있음
  fun addWord(word: String) {
    counter += word.length
  }
}
```



**Constructor**

`class Person(val name: String, var isMarried: Boolean)`

- 생성자에 선언된 데이터는 private
- 생성자에 `val` 또는 `var`이 있는 경우 멤버 변수로 변환 됨

- `val` 또는 `var`이 없는 경우에는 `init{...}`또는 프로퍼티 초기화 식에서만 사용 가능



[코틀린 커스텀 접근자](https://effectivesquid.tistory.com/entry/커스텀-접근자)

프로퍼티를 위해 커스텀 접근자를 정의할 수 있음

custom getter를 정의하는 경우, 해당 프로퍼티에 접근할때 마다 호출됨

-> 이를 통해 computed property를 구현할 수 있음

custom setter를 정의하는경우, 해당 프로퍼티에 값(value)을 assign할 때 마다 호출됨

kotlin 1.1부터 getter를 통해 프로퍼티의 타입을 유추할 수 있는 경우에는 생략 할 수 있음

-> `val isEmpty get() = this.size == 0 // 타입은 Boolean  `

```kotlin
class StateLogger {
  var state = false // 프로퍼티 이름
  	// By convention, setter 파라미터의 이름은 value
  	set(value) { // custom setter를 정의?
      print("$field -> $value")
      field = value
    }
}
```

만약 접근자들을 재정의하면 backing field는 컴파일러에 의해서 만들어지지 않음!

내부에서 `field ` 키워드를 사용하지 않는 경우 backing field가 필요없다는 의미이므로 알아서 안만들어짐



[Backing Fields](https://kotlinlang.org/docs/reference/properties.html#backing-fields)

코틀린 클래스에서 필드는 직접 선언할 수 없음

하지만 property가 backing field가 필요한경우, 코틀린은 이를 자동으로 제공함

이 backing field는 `field` 식별자를 통해 접근자에서 참조 할 수 있음

`field` 식별자는 오직 프로퍼티의 접근자에서만 사용할 수 있음

적어도 하나의 접근자에서 default implementation을 사용하는 경우 baking field는 프로퍼티를 위해 자동으로 생성 됨 

또는  custom 접근자가 `field` 식별자를 통해 이를 참조하는 경우

```kotlin
val isEmpty: Boolean
	get() = this.size == 0 // custom getter, no backing field
```



### More about Properties

**Property in interface**

인터페이스에서 프로퍼티를 정의할 수 있음

```kotlin
// under the hood, 이는 단순히 getter
interface User { 
	val nickname: String
}
// subclass에서 원하는 대로 재정의 가능
class FacebookUser(val sccountId: Int) : User { 
  // field에 값을 저장
	override val nickname = getFacebookName(accountId)
}

class SubscribingUser(val email: String) : User {
  // custom getter를 가지는 property는 상응하는 field를 가지지 않음
  // custom getter를 가지는 프로퍼티는 매 접근마다 값을 계산함
  override val nickname: String
  	get() = email.substringBefore('@')
}
```

인터페이스에 있는 모든 프로퍼티들은 **open**(redefine, not final), can't be used in smart cast

프로퍼티가 custom getter를 가지는 경우 smart cast에서 쓰는 것은 안전하지 않음 

-> 매 접근시마다 값을 다시 계산하기 때문,  mutable value가 smart cast가 안되는 거랑 동일



**Extension properties**

코틀린에서는 extension 프로퍼티를 정의할 수 있으며 syntax는 확장 함수와 유사

```kotlin
// 프로퍼티를 정의시 receiver 타입을 먼저 명시
// receiver에는 this ref를 통해 접근(생략 가능)
val String.lastIndex: Int
	get() = this.length - 1
val String.indices: IntRange 
	get() = 0..lastIndex
"abc".lastIndex // 2
"abc".indices	// 0..2
```



**Mutable extension properties**

```kotlin
val StringBuilder.lastChar: Char
	get() = get(length - 1)
	set(value: Char) {
    this.setCharAt(length - 1, value)
  }
val sb = StringBuilder("Kotlin?")
sb.lastChar = '!'
print(sb) // Kotlin!
```



### Lazy or late initialization

**Lazy property**

- value are coputed onlay on the first success
- won't do anything unless the result is really needed
- `lazy` 키워드 사용, 람다를 인자로 받는 함수 
- 제일 처음 접근시에만 계산되고 저장, 이후에는 저장된 값이 리턴됨



**lateinit**

- unit test의 activity initializing이나 dependency ingection에서 유용
- 때때로 생성자 외에서 프로퍼티를 초기화 하고 싶을 수 있음

```kotlin
class KotlinActivity: Activity() {
	// onCreate에서 초기값을 얻으므로 null을 초기값으로, 그러므로 non-null value 안됨
  var myData: MyData? = null 
  // 이럴때 lateinit 사용
  lateinit var myData: MyData
  
  onCreate() {
    myData = ~~~
  }
  
  myData?.foo // null이 아님에도 매번 safe Access해야함
  myData.foo // lateinit
}
```

그렇다면 lateinit으로 선언한 프로퍼티가 적절하게 초기화 되지 않았다면 어떻게 될까

non-null 로 가정하고 사용했으므로 runtime exception이 날 수 있음, but NPE with detailed message 

-> `lateinit property myData has note been initialized`



`lateinit`을 사용하기 위한 몇 가지 규약

- `lateinit` 사용시 `val`일 수 없음
  - val은 final의 특성을 가지기 때문
- 해당 프로퍼티의 타입은 non-nullable
- primitive type은 lateinit 안됨
  - 오직 레퍼런스 타입만 null로 초기화 될 수 있음
- `isInitialized` 를 통해 lateInit var가 초기화 되었는지 확인할 수 있음



## Object-oriented Programming

### OOP in Kotlin

- 기본적으로 `public`에 `final`임 

- non-final로 만들고 싶다면 `open` 명시

- 코틀린에서 package visibility는 없음 

  대신에 `internal` -> visible inside this same module

  `protected`는 자바에서는 visible inside the same package였지만 코틀린에서는 visible in subclasses



kotlin의 internal(동일한 모듈에서만 visible)

-> JVM에서는 public & name mangling 으로 변환

```kotlin
class MyClass {
  internal fun foo() {} // 라는 함수가 있다면
  // 내부에서는 굉장히~~~긴 이름으로 변경됨. 아래는 예시
  // public final void foo$production_sources_for_module_example_main()
}
```



패키지 구조에서도 차이점이 있음 

자바에서는 클래스가 작아도 각각 별개의 파일로 두었어야 하지만

코틀린에서는 여러개의 클래스들을 하나의 파일에 넣을 수 있음 

또한 최상위 함수나 프로퍼티들의 선언도 하나의 파일에 넣을 수 있음(연관된 것들을 모두 하나의 파일에)

또, 패키지 이름이 디렉토리 구조와 일치할 필요 없음 



###  Constructors, Inheritance syntax

**Constructors**

코틀린에서는 new 키워드 불필요 , 일반 함수처럼 호출, 첫글자가 대문자이면 클래스

초기화에 복잡한 로직이 필요한경우 `init { } `  내에서 수행

생성자 파라미터에 `val` 혹은 `var` 를 사용할 경우 자동으로 프로퍼티를 만듦

`val` 또는 `var`가 없을 경우 이는 오직 생성자 파라미터임



생성자의 visibililty를 변경할 수 있음

```kotlin
class InternalComponent 
// internal 혹은 private keyword사용 가능
internal constructor(name: String) {
  // ...
}
```



**Secondary constructor**

```kotlin
class Rectangle(val height: Int, val width: Int) {
  // this 키워드를 통해 동일한 클래스의 다른 생성자를 호출할 수 있음
  // 반드시 다른 세컨더리 생성자나 주요 생성자를 호출해야 함
  constructor(side: Int) : this(side, side) { ... }
}
```



**Inheritance**

상속에서 조금 다른 문법을 사용

```kotlin
interface Base
class BaseImpl : Base

open class Parent
class Child : Prent() // ()로 생성자 호출
```



**Overriding a property**

프로퍼티를 오버라이딩하는 것은 getter를 오버라이딩 하는것과 동일(필드 x)



### Class modifiers - 1

enum과 data class 에 대해 

`==` 는 내부에서는 `equals` 를 호출하는 것, 즉 content equality 확인

kotlin에서 reference equality를 확인할 때에는 `===` 사용



Data class는 `equals` 를 자동으로 만들어 주기때문에 `==` 를 사용해 content 비교할 수 있음

Data class에서 자동으로 생성되는 함수를 만들때 오직 primary constructor에 정의된 프로퍼티들만 사용함(생성자 괄호안에 들어가는 것들)

-> 제외하고 싶은 경우에는 class body에 선언하면 됨 



### Class modifiers - 2

**sealed class** 

제한된 클래스 계층 구조를 나타낼 때 사용

모든 서브 클래스들은 동일한 파일내에 위치해야 함 

값이 제한된 셋트에서 하나의 타입을 가지나, 다른 타입을 가질 수는 없는 경우

enum type의 value set이 제한된 확장된 enum 클래스로도 볼 수 있음

enum constant가 오직 single instance인 반면에 sealed 클래스는 상태를 포함 할 수 있는 인스턴스들을 여러개 가질 수 있음



**Inner and nested classes**

class 는 다른 클래스 안에 중첩될 수 있음 -> nested class

`inner`로 표기된 nested 클래스는 outer클래스의 멤버에 접근할 수 있음

inner 클래스는 outer 클래스에 대한 참조를 가지고 있음



**class delegation**

[클래스 위임에 대한 글](https://medium.com/til-kotlin-ko/kotlin의-클래스-위임은-어떻게-동작하는가-c14dcbbb08ad)

하나의 클래스를 다른 클래스에 위임하도록 선언, 위임된 클래스가 가지는 인터페이스 메소드를 참조 없이 호출할 수 있도록 생성해줌

```kotlin
interface A
class B : A {}
val b = B() 
// C생성, A에서 정의하는 B의 모든 메서드 C에 위임
// b는 A 타입의 private 필드로 C에 저장, B에 구현된 모든 메서드는 이를 참조하는 형태의 정적 메소드로 생성
class C : A by b
```



`by` means by delagating to the following instance



### Objects, object experessions & companion objects

**object declarataion**

코틀린에서 `object` 는 singleton임(오직 하나의 instance를 가지는)

java에서 싱글턴이 필요할때 해줘야했던 것들 불필요 

- 생성자를 private으로 만들거나, 단 하나의 static instance field를 만드는 등의 작업

```kotlin
public class UsingKotlinObjectFromJava {
  public static void main(String[] args) {
    KSingleton.INSTANCE.foo(); // java처럼 인스턴스 필드에 접근하면 됨
    // 코틀린 코드에서는 그냥 KSingleton.foo()
  }
}
```



**object expressions**

`object `표현식은  java의 익명 클래스를 대신함

object 표현식의 instance는 매 호출마다 새로 생성됨



**companion object** (example of nested object)

Java같은 static method를 대체할 수 있음

-> top-level 위치나 object 혹은 companion objects안에 "static" 멤버들 선언

그렇다면 static method대신에 companion object를 사용하는 이유는?

첫째, companion object는 interface implement할 수 있음 

```kotlin
interface Factory<T> {
  fun create(): T
}

class A {
  private constructor()
  
  // static은 불가능함
  companion object : Factory<A> {
    override fun create(): A {
      return A()
    }
  }
}
```

둘째, can be a receiver of extension function(확장 함수의 receiver가 될 수 있음)

-> 뭐가 장점인지 잘 모르겠음

```kotlin
class Person(val first: String, val last: String) {
  companion object {...}
}

fun Person.Companion.fromJson(json: String): Person {
  ...
}

val p = Person.fromJson(json)
```



**@JvmStatic**

companion object안에 정의된 함수나 object는 static member로 컴파일 되지 않음

Java에서는 static member로 호출 할 수 없음

-> 필요하다면 `@JvmStatic` 어노테이션을 추가해야함 

또는 Companion 인스턴스에 접근해서 호출해야함 



### Constants

string이나 primitive type의 constant를 정의할 때에는 `const` modifier 이용

-> 컴파일 타임 상수로 만들어짐, 컴파일 시간에 실제 값으로 대치 됨 



reference type을 위해 몇가지 이유때문에 getter를 만들고 싶지 않을 경우에는

-> `@JvmField` 어노테이션 적용, 컴파일러에게 field만 만들게함

-> getter가 없고, property가 mutable이라면 setter도 없음



**@JvmField**

코틀린 프로퍼티를 필드로 노출하고 싶은 경우, 필드는 프로퍼티와 동일한 가시성을 가지게 됨

top-level 또는 object class안에서는 property를 static으로 만들어줌

```kotlin
object A {
  @JvmField
  val prop = MyClass() // static fidle가 생성됨
}

class B {
  @JvmField
  val prop = MyClass() // 일반 field가 생성됨
}
```

backing field가 있고(custom getter가 없고?) private이 아니고, `open`, `override` 또는 `const` 한정자를 가지지 않으며, delegated property가 아니면 프로퍼티를 `@JvmField`로 표시할 수 있음

```kotlin
@JvmField
val prop = MyClass()
// 위는 아래 자바코드와 동일함
public static final MyClass prop = new MyClass();
```

지연 초기화(late-initialized) 프로퍼티 또한 field로 노출되며, 해당 필드의 가시성은 `lateinit` 프로퍼티 setter의 가시성과 동일(뭔솔?)



object안에 property를 선언하는 경우 Java에서는 오직 getter를 통해서만 접근할 수 있음

-> 기본적으로 field는 private이기 때문에, as the getter is only available as a member of instance field.

```kotlin
object SuperCompyter {
  val answer = 42
}
// Java
SuperComputer.INSTANCE.getAnswer(); // only via a getter
```

static member로 사용할 수 있도록 하기 위해서

if `@JvmStatic` 애너테이션 적용한다면

이때는 field가 아니라 getter를 static member로서 접근 할 수 있음

`SuperCompyter.getAnswer()`

field를 드러내고 싶은 경우에는 `@JvmField` 어노테이션 적용

`SuperComputer.answer`

primitive 타입이나 string 인 경우에는 `const` modifier를 사용할 수 있음



그렇다면 @JvmField와 const의 차이

`@JvmStatic`이 붙으면 컴파일러는 내부적으로는 getter를 호출함

-> `SuperComputer.getAnswer()`

`@JvmField`가 붙으면 컴파일러는 field를 호출함

-> `superCoputer.answer` // field 호출 

`const`가 붙으면 실제 값으로 대치 됨 (**inline the value**) 

-> `superComputer.answer` // answer는 실제 값으로 대치



### Generics

타입을 파라미터로 받는 클래스와 인터페이스

자바와 다르게 코틀린은 반드시 타입을 정의하고 사용해야함



**Non-nullable upper bound**

generic argument가 nullable이 아님을 제한 하고 싶을 경우

`fun <T: Any> foo(list: List<T>) { ... }` 로 non-null upper bound 명시 



Jvm 플랫폼 제약때문에 동일한 Jvm 시그니쳐를 가지는 함수를 정의할 수 없음

-> 시그니쳐는 제너릭 타입 파라미터를 지우기때문에

아래 두 함수의 시그니쳐는 `average(Ljava/util/List;)D`

```kotlin 
fun List<Int>.average(): Double { ... }
// @JvmName("averageOfDouble")
// 위 어노테이션을 통해 바이트 코드 레벨에서 함수이름을 변경할 수 있음
fun List<Double>.average(): Double { ... }
```



## Conventions

### Operator Overloading

파라미터 타입에 제한은 없음 , receiver 타입과 동일하지 않아도 됨

```kotlin
operator fun Point.times(scale: Int): Point {
  return Point(x * scale, y * scale)
}
```



```kotlin
var list = listOf(1,2,3) // 읽기 전용 리스트를 var로 선언하고 사용할 경우
list += 4 // new list is created

// 위와 같은 경우에는 아래의 mutable list 를 사용하는것이 나음
val list1 = mutableListOf(1,2,3)
```



### Coventions

원소에 인덱스를 가지고 접근 -> `[]`

내부에서는 get, set 메서드가 호출됨

커스텀 클래스의 멤버나 확장함수로 정의하여 사용할 수 있음

``` kotlin
// x[a, b] -> x.get(a, b)
operator fun Board.get(x: Int, y: Int): Char { ... }

// x[a, b] = c -> x.set(a, b, c)
operator fun Board.set(x: Any, y : Any, value: Char) { ... }
```

set operator를 정의하는 경우 추가적인 get operator는 필요하지않음



`in` -> contains 호출

`1..2 ` -> rangeTo 호출



Java에서는 String에 iterable를 구현하지 않아서 순회할 수 없었지만

Kotllin에서는 iterator operator를 확장함수로 구현했기때문에 가능함



### (Not)using operator overloading

Q. 연산자 오버로딩을 사용해야하는 경우와 그렇지 않은 경우에 대한 유즈케이스

A. 연산자 오버로딩은 매우 강력한 메커니즘, 강한 힘에는 강한 책임이.. 

 코틀린에서도 매우 제한된 버전의 연산자 오버로딩만 제공하고 있음

ex. 나만의 연산자를 만들거나 기존 연산자의 우선순위를 변경할 수 없음

해당 연산자에 자연스러운 의미로?만 오버로딩 하라는듯...
