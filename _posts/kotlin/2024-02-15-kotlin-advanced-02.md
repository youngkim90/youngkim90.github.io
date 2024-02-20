---
title: 코틀린 심화2 - Data Class와 Sealed Class
date: 2024-02-15 00:00:00 +0900
categories: [ Programming, Kotlin ]
tags: [ kotlin, data class, sealed class ]
image:
  path: https://blog.jetbrains.com/wp-content/uploads/2019/01/kotlin-2.svg
  content: false
---

## data class

코틀린에서 **`data class`** 는 요청/응답 데이터와 같이 데이터를 다루고 전달하기 위한 **DTO** 같은 용도로 사용하기 위한 클래스이다.  
**`data class`** 는 **equals()**, **hashCode()**, **toString()**, **copy()** 등의 메소드를 자동으로 생성해주며,
**componentN()** 함수도 자동으로 생성해준다. **componentN()** 함수는 data class의 **N번째** 멤버변수 값을 호출하는 함수이다.

```kotlin
// class 앞에 data 키워드를 사용
// 데이터 클래스는 내부적으로 equals(), hashCode(), toString(), copy() 등의 메소드를 포함하고 있다.
data class Person(val name: String, val age: Int)

fun main() {
  val person1 = Person(name = "ryan", age = 30)
  val person2 = Person(name = "ryan", age = 30)

  // data class는 toString() 메소드를 지원한다.(생략 가능)
  println(person1.toString())

  // data class는 == 비교 시 equals() 메소드가 자동 지원된다.
  println(person1 == person2) // true

  // data class는 hashCode를 자동으로 생성하기 때문에 동일한 객체로 인식한다.
  val set = hashSetOf(person1)
  println(set.contains(person2)) // true

  // data class는 copy() 메소드를 통해 객체를 복사할 수 있도록 지원한다.
  val person3 = person1.copy(name = "rory")
  println(person3)

  // data class는 componentN() 함수를 통해 객체의 N번째 멤버변수 값을 호출할 수 있다.
  println("이름=${person1.component1()}, 나이=${person1.component2()}") // 이름=ryan, 나이=30
}

```

위와 같이 **`data class`** 인스턴스를 생성하여 기본으로 제공되는 data class 메소드들을 사용할 수 있다.
또한 **data class**로 명시할 경우 해당 클래스는 데이터를 다루기 위한 클래스임을 명시적으로 표현하게 되는 효과도 있다.

---

## sealed class

코틀린에서 **`sealed class`** 는 상속받은 하위클래스를 **제한 조건**에 따라 정의하기 위한 클래스이다.  
**`sealed class`** 는 다른 패키지나 모듈에서는 상속 받을 수 없으며, 같은 파일 내에서만 상속 받을 수 있다.
아래에서 **sealed class**가 하위클래스를 제한하는 이유와 예시를 확인한다.

```kotlin
// sealed class는 다른 패키지나 모듈에서는 상속이 불가능하다.
sealed class Developer

// sealed class 상속
data class BackendDeveloper : Developer()
data class FrontendDeveloper : Developer()

object DeveloperPool {
  val pool = mutableMapOf<String, Developer>()

  fun add(developer: Developer) = when (developer) {
    is BackendDeveloper -> pool[developer.name] = developer
    is FrontendDeveloper -> pool[developer.name] = developer
//    else -> println("알 수 없는 개발자입니다.")

    // 추상클래스나 일반클래스를 상속받았다면 else를 반드시 정의해야 한다. 안그러면 컴파일에러가 발생한다.
    // 그러나 sealed class를 상속받았다면 같은 파일 내에서만 상속이 되는 제한 조건 때문에 else를 정의하지 않아도 컴파일 에러가 발생하지 않는다.
  }
}
```

위의 예제와 같이 when문에서 **sealed class**를 사용하면 **else**를 정의하지 않아도 컴파일 에러가 발생하지 않는다.  
그 이유는 **sealed class**가 같은 파일 내에서만 클래스를 상속할 수 있다는 **제한 조건**에 따라 정의하기 때문이다.
즉, **Developer** 클래스를 상속받은 클래스는 **BackendDeveloper**와 **FrontendDeveloper** 클래스 밖에 없다는 것을
컴파일러가 인지하고 있기 때문에 **else**를 정의하지 않아도 컴파일 에러가 발생하지 않는다.  
만약 **sealed class**를 상속받은 클래스가 추가되면 어떻게 될까?

```kotlin
// ...
data class AndroidDeveloper : Developer()

object DeveloperPool {
  val pool = mutableMapOf<String, Developer>()

  fun add(developer: Developer) = when (developer) {
    is BackendDeveloper -> pool[developer.name] = developer
    is FrontendDeveloper -> pool[developer.name] = developer

    // AndroidDeveloper 케이스를 추가해주지 않으면 컴파일 에러가 발생한다.
    is AndroidDeveloper -> pool[developer.name] = developer
  }
}
```

위와 같이 **sealed class**를 상속받은 클래스가 추가되는 경우 when문 내에서 컴파일러가 **AndroidDeveloper**를
sealed class를 상속받은 클래스로 인지하고 있기 때문에 케이스로 추가해주지 않으면 컴파일 에러가 발생하게 된다.  
이와 같이 sealed class는 같은 파일 내에서만 상속한다는 **제한 조건**을 가지고 있는 대신,
sealed class를 상속받은 클래스를 컴파일러가 인지하게 되어 when문 내에서 명확하게 케이스 정의가 가능해진다.
단순히 else로 정의해버릴 때 발생할 수 있는 예기치 못한 에러가 발생할 확률을 줄일 수 있는 효과가 있다.
