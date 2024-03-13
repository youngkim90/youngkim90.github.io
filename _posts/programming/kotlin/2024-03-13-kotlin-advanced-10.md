---
title: 코틀린 심화10 - 함수형 프로그래밍
date: 2024-03-06 00:00:00 +0900
categories: [ Programming, Kotlin ]
tags: [ kotlin, functional programming ]
image:
  path: https://blog.jetbrains.com/wp-content/uploads/2019/01/kotlin-2.svg
  content: false
---

## **1급 객체**

코틀린에서 **함수형 프로그래밍**을 활용할 때, 함수를 주로 **`1급 객체`** 로 다루게 된다.
**1급 객체**란 함수를 **변수에 저장**하거나, **매개변수로 전달**하거나, **반환값으로 사용**할 수 있는 객체를 말한다.  
코틀린에서 함수를 **1급 객체**로 다루기 위해서는 **람다식**을 사용하여 함수를 정의해야한다.

```kotlin
val printHello: () -> Unit = { println("Hello") }

fun main() {
  printHello() // Hello
}
```

위에서 **printHello** 변수에 람다식을 사용하여 함수를 정의하면 **1급 객체**로 사용할 수 있다.
**1급 객체**의 함수를 호출할 때는 `변수명()`으로 호출하면 된다. 단순히 **fun** 키워드로 선언된 함수는 1급 객체로 사용할 수 없다.

```kotlin
val printHello: () -> Unit = { println("Hello") }

fun call(block: () -> Unit): () -> Unit {
  block()
  return printBye
}

fun main() {
  val bye = call(printHello) // Hello
  bye() // Bye
}
```

위와 같이 **1급 객체**를 활용하면 **매개변수**로 **함수**를 전달할 수 있다.
**call()** 함수의 매개변수 타입을 **람다식**으로 정의하면 **1급 객체**를 전달할 수 있게 되는 것이다.
**fun** 키워드로 선언된 함수는 함수를 매개변수로 전달할 수 없다.  
또한 **call()** 함수에서 **printBye** 변수(1급 객체)를 반환하는 것을 확인할 수 있다.
이처럼 1급 객체는 **반환 값**으로도 사용 가능하다.

## **고차 함수**

함수형 프로그래밍에서 **`고차 함수`** 란 함수를 **매개변수**로 받거나 **반환**하는 함수를 말한다.

```kotlin
val printBye: () -> Unit = { println("Bye") }

fun forEachStr(collection: Collection<String>, action: (String) -> Unit): () -> Unit {
  for (item in collection) {
    action(item)
  }
  return printBye
}

fun main() {
  val list = listOf("A", "B", "C")
  val printStr: (String) -> Unit = { println(it) }

  forEachStr(list, printStr)
  // 람다식이 마지막 인자로 전달되는 경우 괄호 밖에 작성 가능(후행람다)
  val bye = forEachStr(list) { println(it) }
  bye() // Bye
}
```

위와 같이 코틀린에서는 함수를 **매개변수**로 전달하거나 **반환**하는 고차함수를 사용할 수 있다.
그리고 함수의 마지막 매개변수가 **람다식**인 경우 **후행람다** 방식으로 작성할 수 있다.

## **람다 함수**

람다 함수를 정의하는 방식을 알아보자.

```kotlin
val printMessage: (String) -> Unit = { message -> println(message) }
val printMessage2: (String) -> Unit = { println(it) } // 인자를 it으로 대체 가능

val sum: (Int, Int) -> Int = { x: Int, y: Int -> x + y }
val sum2 = { x: Int, y: Int -> x + y }

fun outerFunc(): () -> Unit = { return fun() { println("익명함수!") } }
fun outerFunc2(): () -> Unit = { println("익명함수!") }

fun main() {
  printMessage("Hello") // Hello
  printMessage2("Hello") // Hello

  println(sum(1, 2)) // 3
  println(sum2(1, 2)) // 3

  outerFunc()() // 익명함수!
  outerFunc2()() // 익명함수!
}
```

위의 코드에서 **printMessage**와 **printMessage2**는 같은 기능을 수행하는 1급 객체지만 람다식을 생략하여 표현할 수 있다.
람다식 내부에서 간단하게 생략 가능하며 단일 인자는 **it**로 대체하여 사용 가능하다.  
**sum**과 **sum2**도 같은 기능을 수행하지만 인자의 **타입추론**을 통해 **sum2**처럼 **타입**을 생략할 수 있다.  
**outerFunc()** 함수는 이름없는 함수(fun())를 반환하는 **익명함수**이다.
리턴 함수를 람다 함수로 변환하면 **outerFunc2()** 와 같이 간결하게 표현할 수 있다.
함수 내부의 **익명함수**를 호출할 경우 내부적으로 한 번더 호출해야 하기 때문에 **()()** 형태로 호출한다.

---



추가적으로 **코틀린**에서 함수형 프로그래밍을 좀 더 깊이있게 사용하기 원한다면
[Arrow 라이브러리](https://arrow-kt.io/) 를 적용하여 사용해보길 추천한다.
**Arrow** 라이브러리는 코틀린에서 **함수형 프로그래밍**을 좀 더 다양하게 사용할 수 있도록 기능을 제공해주는 라이브러리이다.
