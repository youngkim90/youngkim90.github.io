---
title: 코틀린 기본3 - 흐름제어(if, when, for, while)
date: 2024-02-01 00:00:00 +0900
categories: [ Programming, Kotlin ]
tags: [ kotlin, if문, when문, for문, while문 ]
image:
  path: https://blog.jetbrains.com/wp-content/uploads/2019/01/kotlin-2.svg
  content: false
---

## if문

코틀린의 `if문`은 **표현식**으로 사용할 수 있다. 즉, if문의 결과를 변수에 대입할 수 있다.

```kotlin
val age = 10
val str = if (age > 19) {
  "성인"
} else if (age > 7) {
  "어린이"
} else {
  "아기"
}
println(str) // 어린이

```

그리고 if문은 삼항연산자가 없고 대신 아래와 같이 사용할 수 있다.

```kotlin
val a = 1
val b = 2
// if문이 표현식이므로 삼항 연산자가 불필요
val c = if (b > a) b else a
```

---

## when문

코틀린의 **`when`** 은 자바의 **switch**를 대체한다.  
when문은 **표현식**으로도 사용할 수 있다. 즉, when문의 결과를 변수에 대입할 수 있다.

```kotlin
val day = 2
val result = when (day) {
  1 -> "월"
  2 -> "화"
  3 -> "수"
  4 -> "목"
  else -> "기타" // 값을 리턴하지 않는 경우, else는 생략 가능하다.
}
println(result) // 화
```

동일한 결과를 갖는 조건이 여러개인 경우 아래와 같이 **콤마(`,`)** 로 구분해서 표현할 수도 있다.

```kotlin
val day = 2
val result = when (day) {
  1, 2, 3, 4, 5 -> "주중"
  6, 7 -> "주말"
  else -> "기타"
}
println(result) // 주중
```

---

## for문

코틀린에서 **`for문`** 은 편리한 기능들을 제공한다. 아래는 다양한 for문 사용 방식을 보여준다.

#### 범위연산자(`..`)를 사용하여 반복 수행할 범위를 지정

```kotlin
// 1 <= i <= 10
for (i in 1..10) {
  println(i) // 1부터 10까지 출력
}
```

#### `until`을 사용하여 마지막 값을 제외한 범위를 지정

```kotlin
// 1 <= i < 10
for (i in 1 until 10) {
  println(i) // 1부터 9까지 출력
}
```

#### `step`을 사용하여 증가or감소 폭 설정

```kotlin
// 1 <= i <= 10, 증가폭 2
for (i in 1..10 step 2) {
  println(i) // 1, 3, 5, 7, 9
}
```

#### `downTo`를 사용하여 감소하는 범위 지정

```kotlin
// 10 >= i >= 1, 감소폭 2
for (i in 10 downTo 1 step 2) {
  println(i) // 10, 8, 6, 4, 2
}
```

#### 배열 생성 및 반복

```kotlin
val number = arrayOf(1, 2, 3)
for (i in number) {
  println(number[i])
}
```

--- 

## while문

코틀린의 **`while문`** 은 자바와 동일한 방식으로 사용된다.

```kotlin
var a = 5
while (a > 0) {
  println(a)
  a--
}
 ```
