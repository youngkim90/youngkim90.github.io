---
title: 코틀린 기본2 - 변수와 함수
date: 2024-01-31 00:00:00 +0900
categories: [ Programming, Kotlin ]
tags: [ kotlin, var, val, fun, 변수, 함수 ]
image:
  path: https://blog.jetbrains.com/wp-content/uploads/2019/01/kotlin-2.svg
  content: false
---

## 변수 선언

코틀린은 변수 선언 시 **`val`** 또는 **`var`** 키워드를 사용한다.  
아래 코드를 보면 val, var 키워드와 함께 변수를 선언하였다.
그리고 변수명 뒤에 콜론(`:`)을 붙이고 자료형을 명시한다.  
자료형을 생략하면 컴파일 시점에 변수에 초기화 값으로 자료형을 추론하여 자동으로 할당한다.

```kotlin
// 자바와 달리 세미콜론(;)을 붙이지 않아도 된다.
val a: Int = 1
var b: String = "abc"
```

## val, var 차이점

> **val** : value의 약자로 **변경 불가능한 변수**를 선언할 때 사용한다.  
> **var** : variable의 약자로 **변경 가능한 변수**를 선언할 때 사용한다.
>
{: .prompt-tip }

즉, **val**로 선언한 변수는 한번 값을 지정하면 다른 값을 대입할 수 없다. 반면 **var**로 선언한 변수는 값을 다시 대입할 수 있다.  
**val**로 선언한 변수에 값을 다시 할당하려 하는 경우 컴파일 에러가 발생한다.


---

## 함수

기본적으로 함수(Method)는 **`fun`** 키워드를 사용하여 선언한다. 반환 타입을 지정할 수 있으며 표현식을 다양하게 사용할 수 있다.

## 다양한 함수 선언

아래는 다양한 함수 선언 방법을 보여준다.

#### 함수명() 뒤에 콜론(`:`)을 붙여 반환타입 지정

```kotlin
fun sum(a: Int, b: Int): Int {
  return a + b
}
```

#### 표현식과 반환타입 생략

```kotlin
fun sum(a: Int, b: Int) = a + b
```

#### 반환값이 있는 경우 반환 타입이 지정이 안되면 `컴파일 에러 발생`

```kotlin
fun sum(a: Int, b: Int) {
  return a + b // 컴파일 에러. 반환 타입 지정 필요.
}
```

#### 반환값이 없는 함수는 반환타입 Unit을 지정(생략 가능)

```kotlin
fun printSum(a: Int, b: Int): Unit {
  println("sum of $a and $b is ${a + b}")
}
```

#### 함수 파라미터에 디폴트값 설정이 가능하다.

```kotlin
fun greeting(message: String = "Hello") = println(message)

fun main() {
  // 함수 호출 시 디폴트 파라미터는 생략 가능하다.
  greeting() // "Hello"
  greeting("Hi") // "Hi"
}
```
