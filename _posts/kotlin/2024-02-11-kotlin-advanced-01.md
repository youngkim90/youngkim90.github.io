---
title: 코틀린 심화1 - Collection
date: 2024-02-11 00:00:00 +0900
categories: [ Programming, Kotlin ]
tags: [ kotlin, collection, list, set, map, mutablem, immutable ]
image:
  path: https://blog.jetbrains.com/wp-content/uploads/2019/01/kotlin-2.svg
  content: false
---

## Collection

코틀린에서 컬렉션 자료구조는 크게 **`List`**, **`Set`**, **`Map`** 세 가지로 구분된다.
각 자료구조 모두 **불변(immutable)** 과 **가변(mutable)** 형식으로 나누어 사용할 수 있다.
불변 형식의 자료구조는 구조의 변경이 불가능하며, 가변 형식의 자료구조는 구조를 변경할 수 있다.  
아래 컬렉션 계층 다이어그램을 통해 각 컬렉션의 상속관계를 확인할 수 있다.  
![컬렉션 계층 다이어그램](https://kotlinlang.org/docs/images/collections-diagram.png)

코틀린 표준 라이브러리에 등록된 **Colletion 함수**들을 사용하여 자료구조에 대한 선언과 다양한 연산을 수행할 수 있다.

---

## List

**`List`** 자료구조는 순서가 있는 컬렉션으로, 중복된 원소를 가질 수 있다.
불변 형태의 **List**와 가변 형태의 **MutableList**
인터페이스로 나누어 간단하게 선언이 가능하며 선언 방식은 다음과 같다.

```kotlin
// Immutable list
// 불변이므로 초기화 시 자료를 선언하여 할당한다.
val animalList = listOf<String>("개", "고양이", "쥐")

// Mutable list
// 가변이므로 초기화 시 or 초기화 후 자료를 추가하거나 삭제할 수 있다.
val mutableAnimalList = mutableListOf<String>().apply {
  this.addAll(animalList) // 리스트 전체 추가
  this.add("독수리") // 원소 추가
}
mutableAnimalList.remove("개") // 원소 삭제
```

위와 같이 **`listOf()`** 와 **`mutableListOf()`** 컬렉션 메소드를 사용하여 리스트를 선언할 수 있다.  
**ArrayList**와 **LinkedList** 자료구조는, **List** 인터페이스를 상속받은 클래스이며 가변 형태의 리스트로 사용된다.

```kotlin
// linked list
val linkedList = LinkedList<Int>().apply {
  this.addFirst(1)
  this.add(2)
  this.addLast(3)
}

// array list
val arrayList = ArrayList<Int>().apply {
  this.add(1)
  this.add(2)
}
arrayList.add(3)
```

---

## Set

**`Set`** 자료구조는 순서를 보장하지 않는 컬렉션이다. 또한 중복된 원소를 가질 수 없다.
**Set** 자료구조는 불변 형태의 **Set**과 가변 형태의 **MutableSet**
인터페이스로 나누어 간단하게 선언이 가능하며 선언 방식은 다음과 같다.

```kotlin
// Immutable set
val alphabetSet = setOf("a", "b", "c")

// Mutable set
val mutableAlphabetSet = mutableSetOf<String>().apply {
  add(1)
  add(2)
  add(3)
  add(3) // 중복된 원소는 추가되지 않는다.
}
```

위와 같이 **`setOf()`** 와 **`mutableSetOf()`** 컬렉션 메소드를 사용하여 선언할 수 있다.

---

## Map

**`Map`** 자료구조는 **Key-Value** 쌍으로 이루어진 컬렉션이다. **Key**는 중복될 수 없지만 **Value**는 중복될 수 있다.
List, Set과 마찬가지로 **Map** 자료구조도 불변 형태의 **Map**과 가변 형태의 **MutableMap**
인터페이스로 나누어 간단하게 선언이 가능하며 선언 방식은 다음과 같다.

```kotlin
// Immutable map
// key to value
val numberMap = mapOf("one" to 1, "two" to 2, "three" to 3)

// Mutable map
val mutableNumberMap = mutableMapOf<String, Int>()
mutableNumberMap["one"] = 1 // 리터럴 문법
mutableNumberMap.put("two", 2) // 함수 문법
mutableNumberMap["three"] = 3
```

위와 같이 **`mapOf()`** 와 **`mutableMapOf()`** 컬렉션 메소드를 사용하여 선언할 수 있다.
