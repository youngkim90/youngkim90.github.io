---
title: 코틀린 심화9 - 고급 예외처리
date: 2024-03-06 00:00:00 +0900
categories: [ Programming, Kotlin ]
tags: [ kotlin, 예외처리, runCatching ]
image:
  path: https://blog.jetbrains.com/wp-content/uploads/2019/01/kotlin-2.svg
  content: false
---

## **try-catch**

코틀린에서 **try-catch** 문을 사용 시에 **catch**문의 마지막 줄을 **반환 값**으로 사용할 수 있다.

```kotlin
fun getStr(): Nothing = throw Exception("예외 발생 기본 값으로 초기화")

fun main() {
  val result = try {
    getStr() // 예외 발생
  } catch (e: Exception) {
    println(e.message)
    "기본 값" // catch문의 마지막 줄은 반환값
  }

  println(result) // 기본 값
}
```

## **runCatching**

코틀린에서 **runCatching** 함수 내에서 로직을 실행하면 예외발생 시,
**try-catch** 문을 사용하지 않아도 원하는 **반환 값**을 지정해 줄 수 있다.

```kotlin
fun getStr(): Nothing = throw Exception("예외 발생 기본 값으로 초기화")

fun main() {
  // 예외 발생 시 null 반환
  var result: Throwable? = runCatching { getStr() }
    .getOrNull()

  // 예외 발생 시 함수의 결과를 반환
  result = runCatching { getStr() }
    .getOrElse {
      println(it.message)
      "기본 값2"
    }

  // 예외 발생 시 정해진 값 반환
  result = runCatching { getStr() }
    .getOrDefault("기본 값3")

  // 바로 예외를 발생
  result = runCatching { getStr() }
    .getOrThrow()

  // 예외 발생 시 null 반환
  result = runCatching { getStr() }
    .exceptionOrNull()

  // map 함수 내부에서 예외 발생 시 다음 함수가 실행되지 않고 예외발생.
  // mapCatching을 사용하면 예외를 반환하거나 다음 함수를 실행.
  result = runCatching { "안녕하세요" }
    //  .map { throw Exception("예외 발생") } // 예외발생 시 다음 함수 실행X. 예외 발생
    .mapCatching { throw Exception("예외 발생") }
    .getOrDefault("기본 값")

  // recover는 runCatching 함수에서 예외가 발생했을 때 대체할 값을 지정
  // recoverCatching을 사용하면 예외를 반환하거나 다음 함수를 실행.
  result = runCatching { getStr() }
    //  .recover { "복구" } // 예외발생 시 다음 함수 실행X. 예외 반환
    .recoverCatching { throw Exception("예외") }
    .getOrNull()
}
```

위와 같이 코틀린에서는 **runCatching** 함수 내에서 예외발생 시 필요한 **반환 값**을 지정할 수 있다.
`map`, `recover` 함수의 경우 예외발생 시 다음 함수가 실행되지 않으므로, `mapCatching`, `recoverCatching`을 사용하여 처리해줘야 한다.
