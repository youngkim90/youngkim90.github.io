---
title: 코틀린 심화3 - 싱글톤과 동반객체
date: 2024-02-21 00:00:00 +0900
categories: [ Programming, Kotlin ]
tags: [ kotlin, singleton, companion object ]
image:
  path: https://blog.jetbrains.com/wp-content/uploads/2019/01/kotlin-2.svg
  content: false
---

## **singleton**

코틀린에서는 **`object`** 키워드를 사용하여 **`싱글톤`** 객체를 생성할 수 있다.
> `싱글톤`은 인스턴스 하나를 공유하여 사용하는 방식을 의미한다.(자바의 static과 유사하다.)
>
{: .prompt-tip }

```kotlin
// object로 객체 생성 시 싱글톤으로 생성
object Singleton {
  val a = 123
  fun printA() = println("A")
}

fun main() {
  println(Singleton.a)
  Singleton.printA()
}
```

코틀린에서 싱글톤 객체는 하나의 인스턴스를 **공유**하여 사용하기 때문에 자바의 **static** 클래스처럼 사용되며, 주로 **유틸성** 클래스나 **상태 공유** 클래스에 사용된다.
아래에서 간단한 날짜 유틸 싱글톤 객체를 생성하여 활용해보자.

```kotlin
object DateTimeUtils {
  val now: LocalDateTime
    get() = LocalDateTime.now()

  // const는 자바의 상수 개념
  const val DEFAULT_FORMAT = "yyyy-MM-dd"

  fun same(a: LocalDateTime, b: LocalDateTime): Boolean {
    return a == b
  }
}

fun main() {
  println(DateTimeUtils.now)
  println(DateTimeUtils.DEFAULT_FORMAT)

  val now = LocalDateTime.now()
  println(DateTimeUtils.same(now, now))
}
```

---

## **companion object**

코틀린에서는 클래스 내부에서 **`companion object`** 키워드를 사용하여 **동반객체**를 생성할 수 있다.
동반객체 내부에 선언된 변수와 메소드는 클래스의 정적 멤버로 사용되며, 클래스에서 직접 접근이 가능해진다.

```kotlin
class MyClass {
  // 외부에서 생성하지 못하도록 private 생성자로 선언
  private constructor()

  // companion object로 동반객체 생성(이름은 생략가능)
  companion object MyCompanion {
    val a = 123
    fun newInstance() = MyClass()
  }
}

fun main() {
  // MyClass에서 동반객체(MyCompanion)의 변수와 메소드에 직접 접근할 수 있다.
  println(MyClass.a) // = MyClass.MyCompanion.a
  println(MyClass.newInstance()) // = MyClass.MyCompanion.newInstance()
}
```

위와 같이 클래스 내부에서 **`companion object`** 키워드로 **동반객체**를 생성하면, 클래스를 생성하지 않아도 클래스에서 동반객체의 변수와 메소드에 **직접 접근**이 가능하다. 
