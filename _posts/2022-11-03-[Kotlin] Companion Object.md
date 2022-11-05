---
layout: post
title: "[Kotlin] Companion Object"
author: deBaeloper08
date: "2022-11-03 21:09:00 +0900"
categories: [Kotlin]
tags: [companion object]
---

#### Companion Object에 대해 글을 쓰기 전에 자바의 static에 대해 먼저 알아보자.

```java
public final class Student {
	static public final String name = "Bae";
    static public String say(String str) {
    	return "Hi~ " + str;
    }
    ...
}

System.out.println(Student.name);	// Bae
System.out.println(Student.say("Kim"));	// Hi~ Kim
```

static이 붙은 멤버는 클래스가 메모리에 적재될 때 자동으로 함께 생성되어 인스턴스 생성 없이도 클래스명 다음에 점(.)을 붙여 바로 참조할 수 있다.

```kotlin
class Student {
	companion object {
    	val name = "Bae"
        fun say(str: String) = "Hi ~$str"
    }
}

fun main(args: Array<String>) {
	println(Student.name)	// Bae
    println(Student.say("Kim"))	// Hi~ Kim
}
```

이렇게 보면 static과 companion object는 별 차이가 없어 보인다. 코틀린은 static을 버리고 companion object를 도입하였고 이제 companion object에 대해 알아보자.

---

```kotlin
class MyClass2 {
	companion object {
    	val prop = "나는 Companion object의 속성이다"
        fun method() = "나는 Companion object의 메소드다"
    }
}
fun main(args: Array<String>) {
	// MyClass2.멤버는 MyClass2.Companion.멤버의 축약표현이다.
    println(MyClass2.Companion.prop)
    println(MyClass2.Companion.method())
}
```

MyClass2 클래스에 companion object를 만들어 2개의 멤버를 정의했다. main() 함수에서 이 멤버에 접근하기 위해 **클래스명.Companion**형태로 쓴 것을 확인할 수 있다. companion object는 MyClass2 클래스가 메모리에 적재되면서 함께 생성되는 동반(companion)되는 객체고 이 동반 객체는 **클래스명.Companion**으로 접근할 수 있다.

## Companion object는 객체

Companion object에서 기억해야 할 중요한 점은 객체라는 것이다. 그래서 다음과 같은 코딩이 가능해진다.

```kotlin
class MyClass2 {
	companion object {
    	val prop = "나는 Companion object의 속성이다"
        fun method() = "나는 Companion object의 메소드다"
    }
}
fun main(args: Array<String>) {
    println(MyClass2.Companion.prop)
    println(MyClass2.Companion.method())

    val comp1 = MyClass2.Companion	// (1)
    println(comp1.prop)
    println(comp1.method())

    val comp2 = MyClass2	// (2)
    println(comp2.prop)
    println(comp2.method())
}
```

주석 (1)을 보다시피 companion obeject는 객체이므로 변수에 할당할 수 있다. 그리고 할당한 변수에서 점(.)으로 **MyClass2**에 정의된 companion object의 멤버에 접근할 수 있다. 이렇게 변수에 할당하는 것은 **static 키워드로는 불가능**한 방법이다.

주석 (2)는 **.Companion**을 빼고 직접 **MyClass2**로 할당한 것이다. 꼭 기억해야 할 것은 **클래스 내 정의된 companion object는 클래스 이름만으로도 참조 접근이 가능하다**는 것이다.

## Companion object에 이름을 지을 수 있다.

companion object의 기본 이름은 Companion이다. 이 이름을 바꿀 수가 있다.

```kotlin
class MyClass3 {
	companion object MyCompanion { // (1)
    	val prop = "나는 Companion object의 속성이다"
        fun method() = "나는 Companion object의 메소드다"
    }
}
fun main(args: Array<String>) {
    println(MyClass3.MyCompanion.prop)	// (2)
    println(MyClass3.MyCompanion.method())

    val comp1 = MyClass2.MyCompanion	// (3)
    println(comp1.prop)
    println(comp1.method())

    val comp2 = MyClass3	// (4)
    println(comp2.prop)
    println(comp2.method())

    val comp3 = MyClass3.Companion	// (5) 에러 발생!!
    println(comp3.prop)
    println(comp3.method())
}
```

주석 (1)에서 companion object 이름을 **MyCompanion**으로 설정했고 주석 (2), (3)을 보면 기본 이름인 **Companion** 대신 **MyCompanion**을 사용했다. 그리고 주석 (4)를 보면 이름을 생략할 수 있다.
하지만 주석 (5)처럼 기존 이름인 **Companion**을 쓰면 **Unresolved reference:Companion 에러**가 난다.

## 클래스 내 Companion object는 하나만 쓸 수 있습니다.

클래스 내에 2개 이상의 companion object를 쓰는 것은 안 된다. 코틀린은 클래스 명만으로 companion object 객체를 참조할 수 있기 때문에 한 번에 2개를 참조하는 것은 불가능하다.

```kotlin
class MyClass5 {
	companion object {
    	val prop1 = "나는 Companion object의 속성이다"
        fun method1() = "나는 Companion object의 메소드다"
    }
    companion object { // 에러 발생!!
    	val prop2 = "나는 Companion object의 속성이다"
        fun method2() = "나는 Companion object의 메소드다"
    }
}
```

위처럼 만들면 "**Only one companion object is allowed per class**" 에러가 발생한다.

## Interface 내에도 Companion object를 정의할 수 있다.

코틀린 인터페이스 내에 companion object를 정의할 수 있다. 이 특징을 잘 활용하면 설계하는 데 도움이 될 것이다.

```kotlin
interface MyInterface{
    companion object{
        val prop = "나는 인터페이스 내의 Companion object의 속성이다."
        fun method() = "나는 인터페이스 내의 Companion object의 메소드다."
    }
}
fun main(args: Array<String>) {
    println(MyInterface.prop)
    println(MyInterface.method())

    val comp1 = MyInterface.Companion
    println(comp1.prop)
    println(comp1.method())

    val comp2 = MyInterface
    println(comp2.prop)
    println(comp2.method())
}
```
