---
title: 코틀린 심화7 - Tuples
date: 2024-03-03 00:00:00 +0900
categories: [ Programming, Kotlin ]
tags: [ kotlin, tuples, pair, triple ]
image:
  path: https://blog.jetbrains.com/wp-content/uploads/2019/01/kotlin-2.svg
  content: false
---

## **Pair**

코틀린에서 **`Pair`** 클래스는 **두 개의 값**을 담는 컨테이너로 사용된다.  
**data 클래스** 기반으로 구현되었으며, **first**, **second** 프로퍼티를 사용하여 값을 가져올 수 있다.
**Pair** 클래스는 data 클래스에서 제공하는 함수 외에도 **to**, **toList** 함수를 제공한다.

```kotlin
val pair = Pair("A", "B")
println(pair.first) // A
println(pair.second) // B

// pair 객체 복사
val newPair = pair.copy(first = "C")
println(newPair) // (C, B)

// to 함수를 사용하여 Pair 객체 생성
val toPair = "C" to "D"
println(toPair) // (C, D)

// toList 함수를 사용하여 Pair 객체를 List로 변환
val toList = toPair.toList()
println(toList) // [C, D]

// Map을 생성시에도 key, value를 pair로 생성
val map = mutableMapOf("나" to "개발자")
for ((name, job) in map) {
  println("${name}의 직업은 $job") // 나의 직업은 개발자
}
```

위와 같이 **Pair** 클래스는 **first**, **second** 프로퍼티를 사용하여 두 값을 활용할 수 있다.
주로 **두 값**을 한번에 저장하거나 반환하기 위한 목적으로 사용된다.  
또한 **Map**을 생성할 때에도 **key, value**를 **Pair**로 생성하여 사용할 수 있다.

## **Triple**

코틀린에서 **`Triple`** 클래스는 **세 개의 값**을 담는 컨테이너로 사용된다.  
**Pair** 클래스와 마찬가지로 **data 클래스** 기반으로 구현되었으며,
**first**, **second**, **third** 프로퍼티를 사용하여 값을 가져올 수 있다.

```kotlin
val triple = Triple("A", "B", "C")
println(triple) // (A, B, C)

// Triple 객체 복사
val newTriple = triple.copy(third = "D")
println(newTriple) // (A, B, D)

// Triple 객체를 List로 변환
val list = newTriple.toList()
println(list) // [A, B, D]

// 구조분해 할당으로 변수 선언 가능
val (a: String, b: String, c: String) = newTriple
println("$a, $b, $c") // A, B, D
```

위와 같이 **Triple** 클래스는 **first**, **second**, **third** 프로퍼티를 사용하여 세 값을 활용할 수 있다.
주로 **세 값**을 한번에 저장하거나 반환하기 위한 목적으로 사용된다.  
또한 **Pair**, **Triple** 클래스 모두 **data** 클래스를 기반이므로 프로퍼티를 **`구조분해 할당`** 방식으로 선언 가능하다.

