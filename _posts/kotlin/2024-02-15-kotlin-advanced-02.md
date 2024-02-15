---
title: 코틀린 심화1 - Data Class, Sealed Class
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

