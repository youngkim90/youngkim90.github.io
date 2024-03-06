---
title: 코틀린 심화5 - 제네릭스
date: 2024-02-25 00:00:00 +0900
categories: [ Programming, Kotlin ]
tags: [ kotlin, 제네릭스, generics ]
image:
  path: https://blog.jetbrains.com/wp-content/uploads/2019/01/kotlin-2.svg
  content: false
---

## **Generics**

코틀린에서는 자바와 마찬가지로 **`Generics`** 를 지원한다.
> 제네릭스란 클래스나 함수를 정의할 때 타입을 파라미터로 받아서 사용할 수 있도록 하는 기능이다.
>
{: .prompt-tip }

```kotlin
class MyGenerics<T>(val t: T)

fun main() {
  // 반환타입은 생략가능
  val generics = MyGenerics("Hello") // MyGenerics<String>
  // 변수 타입에 제네릭을 사용한 경우
  val list1: MutableList<String> = mutableListOf()
  // 타입 아규먼트를 생성자에서 추가
  val list2 = mutableListOf<String>()
}
```

위와 같이 **MyGenerics** 클래스 생성자 호출 시 **"Hello"** 라는 **String** 문자열을 넣어주면
`var t: String`이 되어 **`T`** 의 타입이 **String**으로 결정된다.  
또한 list1, list2와 같이 **타입 아규먼트**를 **반환타입**, **생성자**에 추가하여 사용할 수 있다.

## **공변성과 반공변성**

코틀린에서는 제네릭을 사용할 때 **`공변성`** 과 **`반공변성`** 을 지원한다.
> **`공변성`** : **상위** 클래스 타입의 참조가 **하위** 클래스 타입을 참조 가능하게 하는 것 <-> **`반공변성`**  
> 자바 제네릭의 **extends** 코틀린에선 **out**  
> 자바 제네릭의 **super** 코틀린에선 **in**
>
{: .prompt-tip }

아래 예시에서 **공변성**과 **반공변성**을 어떻게 지원하는지 알아보자.

```kotlin
class MyGenerics<out T>(val t: T)

class Bag<T> {
  fun saveAll(
    to: MutableList<in T>,
    from: MutableList<T>
  ) {
    to.addAll(from)
  }
}

fun main() {
  val generics: MyGenerics<String> = MyGenerics("Hello")

  // MyGenerics 클래스가 CharSequence 타입 아규먼트로 선언되었어도, out T 이기 때문에 상속받은 String 타입도 참조 가능(공변성)
  val charGenerics: MyGenerics<CharSequence> = generics

  // Bag 클래스가 String 타입 아규먼트로 선언되었어도, 'to' 변수 타입 아규먼트가 in T 이기 때문에 부모인 CharSequence 타입 참조 가능(반공변성)
  val bag = Bag<String>()
  bag.saveAll(mutableListOf<CharSequence>("1", "2"), mutableListOf<String>("3", "4"))
}
```

**MyGenerics** 클래스의 **`<out CharSequence>`** 는 **CharSequence**의 **`공변성`** 을 지원한다는 의미이다.
즉, **String** 클래스는 **CharSequence**를 **상속** 받고 있기 때문에 **공변성**이 지원되어 **String** 타입도 **참조 가능**해진다.  
**Bag** 클래스는 **String** 타입 아규먼트로 제네릭을 사용하여 선언하였고,
**to** 변수의 **`<in CharSequence>`** 는 **CharSequence**의 **`반공변성`** 을 지원한다는 의미이다.
즉, **CharSequence** 클래스는 **String**의 **부모** 클래스이기 때문에 **반공변성**이 지원되어 **CharSequence** 타입도 **참조 가능**해진다.

공변성과 반공변성의 개념은 **제네릭**을 사용할 때 **타입**을 **유연**하게 사용할 수 있도록 도와주기 때문에 중요하다.

