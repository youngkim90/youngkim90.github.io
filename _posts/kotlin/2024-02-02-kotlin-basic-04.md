---
title: 코틀린 기본4 - Null and Exception
date: 2024-02-02 00:00:00 +0900
categories: [ Programming, Kotlin ]
tags: [ kotlin, null, exception, nullsafety, tryCatch ]
image:
  path: https://blog.jetbrains.com/wp-content/uploads/2019/01/kotlin-2.svg
  content: false
---

## Null Safety

코틀린은 컴파일러에서 **`Null Safety`** 기능을 지원한다.  
자바에서는 기본형 변수에 **null**을 대입할 수 있지만, 코틀린에서는 기본형 변수에 **null**을 바로 대입할 수 없다.

```kotlin
// val a : String = null // 컴파일러 에러
```

기본형 변수에 null을 대입하기 위해서는 변수를 **nullable** 타입으로 선언해야 한다.

```kotlin
// 타입 뒤에 ?를 붙이면 nullable 타입의 변수로 선언된다.
val a: String? = null
// nullable 타입의 변수는 뒤에 안전연산자(?.)를 사용하여 접근해야 한다.
println(a?.length)

val b = null // nullable 타입으로 자동 추론된다.
```

변수 값이 **non-null**로 추론될 경우 안전연산자를 사용하지 않아도 된다.

```kotlin
val a: String? = null
val b = a ?: "a" // non-null 타입으로 자동 추론된다.
println(b.length) // non-null 타입으로 추론된 경우 안전연산자(?.)가 없어도 컴파일 에러가 발생하지 않는다.
```

> **?:** 엘비스 연산자로 **null**이 아닌 경우 **좌변**의 값을 반환하고, **null**인 경우 **우변**의 값을 반환한다.
>
{: .prompt-tip }

이처럼 코틀린 컴파일러에서 **null**을 사전에 체크하여 **NullPointerException**이 발생하는 것을 사전에 방지할 수 있다.

---

## 예외처리

코틀린은 자바와 다르게 **Checked Exception**을 강제하지 않는다.
그 이유는 **Checked Exception**이 코드의 가독성을 떨어뜨리고, 예외처리를 강제하는 것이 오히려 코드의 복잡성을 증가시키기 때문이다.  
따라서 아래처럼 작성하여도 컴파일 에러가 발생하지 않는다.

```kotlin
// tryCatch문이 없어도 컴파일 에러가 발생하지 않는다.
Thread.sleep(1)
```

필요한 경우 **tryCatch**문을 사용하여 예외를 처리할 수도 있다.

```kotlin
try {
  Thread.sleep(1)
} catch (e: Exception) {
  println("에러 발생")
} finally {
  println("finally")
}
```

그리고 코틀린에서 tryCatch문은 **표현식**으로 사용할 수 있으며, tryCatch문의 결과값은 **try 블록의 결과값**이 된다.

```kotlin
// tryCatch를 표현식으로 사용 가능
val a = try {
  "123".toInt()
} catch (e: Exception) {
  println("예외 발생 !")
}
println(a) // "123"
```

또한 **Exception**을 함수로 던질 수도 있다.

```kotlin
// 함수가 정상적으로 끝나지 않는 경우 Nothing 타입을 반환(생략가능)
fun fail(message: String): Nothing {
  throw IllegalArgumentException(message)
}

val a: String? = null
val b = a ?: fail("a is Null")
// 예외 발생
// Exception in thread "main" java.lang.IllegalArgumentException: a is Null
```
