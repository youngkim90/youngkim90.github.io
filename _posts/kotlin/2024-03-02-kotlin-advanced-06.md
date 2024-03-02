---
title: 코틀린 심화6 - 지연초기화, 지연로딩
date: 2024-03-02 00:00:00 +0900
categories: [ Programming, Kotlin ]
tags: [ kotlin, lateinit, lazy, 지연초기화, 지연로딩 ]
image:
  path: https://blog.jetbrains.com/wp-content/uploads/2019/01/kotlin-2.svg
  content: false
---

## **lateinit(지연초기화)**

코틀린에서 **`lateinit`** 키워드는 클래스의 프로퍼티를 객체 생성시점이 아닌 그 이후에 초기화할 때 사용한다.
**lateinit**은 **var** 가변변수에만 선언 가능하며, **non-null** 타입으로 선언해야 한다.

```kotlin
class LateInit {
  // lateinit은 var로 선언해야 하고, non-null 타입으로 선언
  // Int, Float, Double, Boolean 와 같은 원시타입은 사용 불가
  lateinit var text: String

  fun printText() {
    // 지연변수 초기화 없이 호출 시에는 에러 발생
    if (this::text.isInitialized) println("초기화 완료")
    else text = "안녕하세요"

    println(text)
  }
}

fun main() {
  val test = LateInit()

  test.printText() // 안녕하세요
  test.text = "Hello" // 외부에서 지연변수에 값 할당
  test.printText() // 초기화 완료 \n Hello
}
```

위와 같이 **지연변수**로 선언된 **text**는 **LateInit** 클래스가 선언된 후에 외부에서 프로퍼티에 초기화하여 사용할 수 있다.
지연변수는 주로 **의존성 주입**이나 **프로퍼티 주입** 등 객체 생성시점 이후에 변수에 값을 할당할 때 사용된다.

## **lazy(지연로딩)**

코틀린에서 **`lazy`** 함수는 **변수 초기화 시점을 호출 시점으로 늦추어 성능을 향상**시키기 위해 사용된다.
**lazy** 함수는 **val** 불변변수에만 사용 가능하며, **`by lazy`** 키워드를 사용하여 **지연로딩**을 구현할 수 있다.

```kotlin
class HelloBot {
  // 지연로딩은 val로 선언하며, by lazy 키워드로 초기화 로직을 정의
  // 지연로딩은 스레드에 대한 동기화처리가 되어있어 멀티스레드 환경에서 안전하게 사용 가능
  val greeting: String by lazy {
    println("초기화 로직은 최초 호출 시 한번만 수행") // 최초 호출 시에만 출력
    getHello()
  }

  fun sayHello() = println(greeting)
}

fun getHello() = "안녕하세요"

fun main() {
  val helloBot = HelloBot()

  for (i in 1..3) {
    Thread {
      helloBot.sayHello()
      // 초기화 로직 수행
      // 안녕하세요
      // 안녕하세요
      // 안녕하세요
    }.start()
  }
}
```

위와 같이 **지연로딩**은 **`by lazy`** 이 후에 람다 표현식으로 **초기화 로직**을 정의하며,
**최초** 호출 시점에 **한번만** 초기화 로직이 수행되고 그 이후엔 변수에 **저장된 값**만 반환된다.  
**지연로딩**은 스레드에 대한 **동기화처리**를 지원하여 **한 스레드**에서만 초기화 로직이 한번 수행된다.
즉, 여러 스레드에서 접근하더라도 초기화는 한번만 수행되어 **스레드 환경으로 부터 안전하게 사용**할 수 있다.

