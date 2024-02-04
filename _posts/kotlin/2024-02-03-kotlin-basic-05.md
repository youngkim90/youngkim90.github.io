---
title: 코틀린 기본5 - Class와 상속
date: 2024-02-03 00:00:00 +0900
categories: [ Programming, Kotlin ]
tags: [ kotlin, class, open class, 상속, getter, setter ]
image:
  path: https://blog.jetbrains.com/wp-content/uploads/2019/01/kotlin-2.svg
  content: false
---

## class 생성

코틀린 클래스 파일에 클래스를 생성할 때 **`class`** 키워드를 사용한다.  
그리고 자바와는 다르게 **생성자**를 선언할 때 **`constructor`** 키워드를 **class 명**과 함께 작성한다.

```kotlin
class Coffee constructor(
  var name: String = "아메리카노",
  var price: Int = 2000, // 후행콤마를 사용하여 여러 줄로 작성할 수 있다.
) {}
```

이렇게 **생성자**와 **멤버변수**를 동시에 선언할 수 있고, 멤버변수들은 **기본값**을 지정할 수 있다.  
**기본 생성자**는 컴파일 시점에 자동 생성되기 때문에, **constructor** 키워드는 생략할 수 있다.

```kotlin
// constructor 키워드 생략.
class Coffee(
  var name: String = "아메리카노",
  var price: Int = 2000,
) {}
```

> 코틀린 클래스 내부의 **var** 변수는 **getter**, **setter**를 자동 지원하고, **val** 변수는 **getter**만 지원한다.
>
{: .prompt-tip }

---

## class 선언과 사용

코틀린에서 class는 **new** 키워드를 사용하지 않고 객체 인스턴스를 생성할 수 있다.  
또한 위에서 언급한대로 class의 **var, val** 멤버변수들의 **getter, setter**는 자동으로 생성된다.  
아래는 class 생성자를 호출하는 다양한 방식과 멤버변수를 사용하는 방법이다.

```kotlin
// new 키워드 없이 객체 생성
val coffee1 = Coffee() // 기본 생성자로 객체 생성
val coffee2 = Coffee("라떼", 4000) // 생성자 매개변수 값을 지정하여 객체 생성
val coffee3 = Coffee(name = "라떼", price = 4000) // 생성자 매개변수의 이름과 값을 함께 지정하여 객체 생성 

// class의 멤버변수에 아래와 같이 값을 저장한다. 
// set~ 방식으로 저장하진 않지만, 내부적으로 setter 메소드가 호출된다.
coffee1.name = "콜드브루"
coffee1.price = 3000

// class의 멤버변수를 아래와 같이 값을 읽어온다.
// get~ 방식으로 불러오진 않지만, 내부적으로 getter 메소드가 호출된다.
printlns(coffee1.name) // 콜드브루
```

코틀린에서는 위에 처럼 get~, set~ 메소드를 호출하지 않고 변수를 직접 선언하여 값을 활용하는 방식으로 **getter, setter**를 사용하게 된다.
**val**로 선언된 변수는 **getter**만 선언되어 있다.  
물론 getter, setter 메소드를 직접 선언하여 **커스텀**하게 사용할 수도 있다.

```kotlin
class Coffee(
  var name: String = "",
  var price: Int = 0,
) {
  val brand: String
    get() = "이디야" // 커스텀 getter 선언

  var quantity: Int = 0
    set(value) {
      field = value // field는 해당 프로퍼티를 가리킨다.
      // quantity = value // 프로퍼티에 직접 저장 시 무한루프 발생
    }
}
```

위에처럼 get(), set() 커스텀 메소드를 변수와 함께 선언하여 커스텀 가능하고, getter/setter 메소드 내부에서 **field**는 해당 변수를 가리킨다.  
만약 **field**를 사용하지 않고, 변수에 직접 할당하면 **set()** 이 반복 호출되어 `무한루프`가 발생할 수 있으니 **주의**해야 한다.

---

## 상속

코틀린의 class는 기본적으로 **final**로 선언되기 때문에 **`상속`** 이 불가능하다.  
상속을 허용하려면 **class**에 **`open`** 키워드를 사용하여 선언해야 한다.

```kotlin
// open이 붙은 class는 하위클래스에서 상속이 가능하다.
open class Dog {
  open var age: Int = 0
  open fun bark() {
    println("멍멍")
  }
}
```

하위클래스에서 상속을 받기 위해서 클래스 뒤에 **`:`** 와 함께 상속받을 클래스를 지정한다.  
그리고 부모클래스의 메소드나 변수를 오버라이드 하려면 **`override`** 키워드를 사용한다.

```kotlin
// 변수는 생성자에서 override 키워드와 함께 선언 가능하다.
open class Bulldog(override var age: Int = 0) : Dog() {
  // final 키워드 사용 시 하위 클래스에서 오버라이드 불가능
  final override fun bark() {
    super.bark()
    println("컹컹")
  }
}
```

위처럼 변수나 메소드를 오버라이드하여 사용 가능하고, **final** 키워드를 함수에 사용하면 하위클래스에서 오버라이드가 불가능해진다.
부모클래스의 변수나 메소드를 호출할 때는 자바와 동일하게 **super** 키워드를 사용하면 된다.
