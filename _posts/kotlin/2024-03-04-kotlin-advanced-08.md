---
title: 코틀린 심화8 - 스코프 함수
date: 2024-03-04 00:00:00 +0900
categories: [ Programming, Kotlin ]
tags: [ kotlin, 스코프 함수, let, run, with, apply, also ]
image:
  path: https://blog.jetbrains.com/wp-content/uploads/2019/01/kotlin-2.svg
  content: false
---

## **let**

코틀린 스코프 함수 중 **`let()`** 함수는 **`null`** 이 아닌 객체에 대해 **블록 내부**에서
**`it`**을 사용하여 **객체의 속성**을 사용하고 값을 반환할 때 사용한다.

```kotlin
val str: String? = "Hello"

// let은 null이 아닌 경우에만 실행됨
val result: Int? = str?.let {
  println(it) // Hello

  123 // let 함수의 마지막 줄은 반환값
}
println(result) // 123
```

## **run**

코틀린 스코프 함수 중 **`run()`** 함수는 객체의 함수를 수행할 때 주로 사용하며,
**let()** 함수와 비슷하지만 **it** 을 사용하지 않아도 객체의 속성을 사용할 수 있어 더 편리하다.

```kotlin
class DatabaseClient {
  var url: String? = null
  var username: String? = null
  var password: String? = null

  fun connect(): Boolean {
    println("Connecting...")
    Thread.sleep(1000)
    println("Connected")
    return true
  }
}

// let으로도 가능하지만 it을 사용해야 하기 때문에 run이 더 편리하다.
val connected: Boolean = DatabaseClient().run {
  // this가 생략되어 있음
  url = "localhost:3306"
  username = "root"
  password = "1234"

  connect() // run 함수의 마지막 줄은 반환값
}
println(connected) // true
```

## **with**

코틀린 스코프 함수 중 **`with()`** 함수는 **run()** 함수와 비슷하게 사용되지만
**run()** 함수는 **확장함수**로 사용되고, **`with()`** 함수는 **일반함수**로 사용된다.

```kotlin
// with는 확장 함수가 아니라는 점에서 run과 다르다.
val result: Boolean = with(DatabaseClient()) {
  url = "localhost:3306"
  username = "root"
  password = "1234"

  connect()
}
println(result)
```

## **apply**

코틀린 스코프 함수 중 **`apply()`** 함수는 반환 값이 객체가 되며,
**객체 생성** 시에 **객체의 속성**을 초기화할 때 주로 사용된다.

```kotlin
// apply는 반환 값이 객체 자신
val client: DatabaseClient = DatabaseClient().apply {
  // 값 초기화
  url = "localhost:3306"
  username = "root"
  password = "1234"
}

client.connect().run {
  println(this)
}
```

## **also**

코틀린 스코프 함수 중 **`also()`** 함수는 객체 생성 시 객체에 대한 **초기화**, **검증** 등의 **사이드 이펙트**를 수행할 때 주로 사용한다.
**let** 함수처럼 **it**을 활용할 수 있으며 **apply** 함수처럼 객체 **자신을 반환**한다.

```kotlin
class User(val name: String, val password: String) {
  fun validate() {
    if (name.isNotEmpty() && password.isNotEmpty()) {
      println("유효하지 않은 사용자입니다.")
    } else {
      println("유효한 사용자입니다.")
    }
  }

  fun printName() = println(name)
}

// also 함수를 통해 객체의 속성에 대한 검증을 진행
val user: User = User(name = "wani", password = "1234").also {
  it.validate() // 유효한 사용자입니다.
  it.printName() // wani
}

// also 함수를 통해 리스트를 수정
val addList = mutableListOf(1, 2, 3).also {
  it.add(4) // 객체 수정
}
println(addList) // [1, 2, 3, 4]
```


