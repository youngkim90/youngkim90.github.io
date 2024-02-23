---
title: 코틀린 심화4 - 확장 함수
date: 2024-02-24 00:00:00 +0900
categories: [ Programming, Kotlin ]
tags: [ kotlin, 확장 함수, extension ]
image:
  path: https://blog.jetbrains.com/wp-content/uploads/2019/01/kotlin-2.svg
  content: false
---

## **확장 함수**

코틀린에서는 **`확장 함수`** 기능을 사용하여 기존 클래스에 새로운 함수를 자유롭게 **추가**할 수 있다.  
다만 확장함수는 코틀린 버전이 올라가면 **동일한 함수**가 추가될 수 있기 때문에 **주의**해야 한다.

```kotlin
fun String.first(): Char {
  return this[0]
}

fun String.addFirst(char: Char): String {
  return char + this.substring(0)
}

fun main() {
  println("ABC".first()) // A
  println("ABC".addFirst('Z')) // ZABC
}
```

위와 같이 **`String`** 클래스에 **`.first()`** 와 **`.addFirst()`** 확장함수를 외부에서 추가하여 사용 가능하다.

## 확장 함수의 사용

확장 함수는 주로 오브젝트에서 **공통**으로 사용하는 기능을 추가하거나 **오버로딩**하여 사용해야하는 경우에 사용된다.

```kotlin
class MyClass {
  fun printMessage() = println("Hello")
}

// 외부에서 printMessage 오버로딩 확장함수 추가
fun MyClass.printMessage(message: String) = println(message)

// 외부에서 null 체크 공통 확장함수 추가
fun MyClass?.printNullOrNotNull() {
  if (this == null) println("null")
  else println("not null")
}

fun main() {
  MyClass().printMessage() // 멤버 함수 출력
  MyClass().printMessage("Goodbye") // 확장 함수 출력

  var myClass: MyClass? = null
  MyClass.printNullOrNotNull() // null

  myClass = MyClass()
  MyClass.printNullOrNotNull() // not null
}
```
