---
title: 코틀린 기본6 - 추상클래스와 인터페이스
date: 2024-02-06 00:00:00 +0900
categories: [ Programming, Kotlin ]
tags: [ kotlin, abstract class, interface, 추상클래스, 인터페이스 ]
image:
  path: https://blog.jetbrains.com/wp-content/uploads/2019/01/kotlin-2.svg
  content: false
---

## 추상클래스와 상속

코틀린에서 **추상클래스**는 자바와 동일하게 **`abstract`** 키워드를 사용하여 선언한다.
추상클래스의 목적은 상속받은 클래스에서 기능을 **오버라이드** 하여 구현하도록 강제하는데 있다.  
추상클래스 내부에 **`abstract`** 키워드를 사용하여 변수, 함수를 선언하면 상속받는 클래스에서는
반드시 **오버라이드** 하여 구현해야 한다.

```kotlin
// 추상클래스
abstract class Developer {
  abstract var age: Int
  abstract fun code(language: String)
}

class BackendDeveloper(override var age: Int) : Developer() {
  // 추상클래스의 abstract 변수, 함수는 모두 오버라이드 하여 할당해야 한다.
  override fun code(language: String) {
    println("BackendDeveloper code is $language")
  }
}
```

추상클래스도 클래스이기 때문에 **다중상속**은 **불가능**하다.
자바와 코틀린에서 다중상속을 막은 이유는 부모 클래스에 동일한 이름의 함수가 있으면 컴파일 에러가 발생하기 때문이다.
다중상속을 구현해야 한다면 **인터페이스**를 사용하여야 한다.

---

## 인터페이스

**인터페이스**는 추상클래스와 유사하게 상속받은 클래스에서 구현을 강제하기 위한 목적으로 사용되지만,
추상클래스와는 다르게 **다중상속**을 지원한다. 그 이유는 인터페이스 내부적인 구현 없이 오직 **추상클래스or추상변수**만을 정의하고 있기 때문이다.  
그렇기 때문에 자바에서도 **extends**가 아닌 **implements**(구현, 실행) 키워드를 사용하여 인터페이스를 상속받는다.
코틀린에서는 상속 시 인터페이스도 클래스 상속과 동일한 방식으로 표현하며 **override** 키워드를 사용하여 구현한다.

```kotlin
interface Developer {
  var age: Int
  fun code(language: String)
}

class BackendDeveloper(override var age: Int) : Developer() {
  override fun code(language: String) {
    println("BackendDeveloper code is $language")
  }
}
```

그렇다면 인터페이스 내부에서는 구현이 불가능한걸까? 자바는 JDK1.8부터 **default** 키워드를 사용하여 인터페이스 내부에서 구현을 지원하고 있다.  
코틀린도 마찬가지로 내부 구현이 가능하다. 함수를 선언 후 기능을 구현하면 자동으로 자바의 **default** 키워드가 붙어있는 함수처럼 인식된다.

```kotlin
interface Developer {
  var age: Int
  fun code(language: String)

  // 메소드의 기능을 구현하면 자동으로 default 함수로 인식된다.
  fun review() {
    println("review")
  }
}

class BackendDeveloper(override var age: Int) : Developer() {
  override fun code(language: String) {
    println("BackendDeveloper code is $language")
  }
  // 인터페이스의 default 함수는 오버라이드하지 않아도 컴파일 에러가 발생하지 않는다.
}
```

그러나 만약 여러 개의 인터페이스에서 동일한 이름의 함수로 기능이 **구현**되어 있다면, 다중 상속시 컴파일 에러가 발생하니 이 부분은 `주의`해야 한다.  
상속받은 인터페이스의 **구현 함수**를 호출할 때는 **super** 키워드를 사용하여 호출할 수 있다.

```kotlin
interface Developer {
  var age: Int
  fun code(language: String)

  fun review() {
    println("review")
  }
}

class BackendDeveloper(override var age: Int) : Developer() {
  override fun code(language: String) {
    println("BackendDeveloper code is $language")
    super<Developer>.review() // 상속받은 인터페이스의 구현 함수 호출  
  }
}
```

