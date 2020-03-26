---
layout: post
title: "코틀린의 Companion Object (2) - 중첩클래스(Nested Class)와 내부클래스(Inner Class)"
date: 2020-02-23 20:00:00 +0900
categories: Kotlin(코틀린)
tags: [코틀린, kotlin, Comapanion object, Nested class, Inner class]
author: 지용호
image: assets/images/11.jpg
comments: true
featured: true
hidden: false

---
지난 글에서 **코틀린(Kotlin)**의 **Companion object**를 소개했습니다. 더불어 자바(Java)의 static 키워드와 차이점도 알게 되었습니다.

하지만 코틀린의 companion object에 대해 더 이해할 점이 아직 많이 남아 있습니다. 이번 글에서는 중첩클래스와 내부클래스를 이해해 보고 이와 Companion object의 관계에 대해서도 알아보겠습니다.

## 중첩클래스와 내부클래스란?

중첩클래스와 내부클래스는 각각 Nested Class와 Inner Class를 한글로 번역한 것입니다. 둘 다 클래스 내부에 또 다른 클래스를 정의하는 것을 의미합니다. 다음은 중첩클래스입니다.

```kotlin
class OuterClass{
	//중첩클래스
	class NestedClass
}
```

중첩클래스의 class 키워드 앞에 inner 키워드를 붙이면 내부클래스가 됩니다.

```kotlin
class OuterClass{
	//내부클래스
	inner class InnerClass
}
```

이 두 클래스에 대해서 공통점과 차이점을 간단히 요약해보면 다음과 같습니다.

- 공통점 : 클래스 내부에 다른 클래스로 정의됩니다.
- 외형적 차이점 : inner 키워드를 쓰면 내부클래스, 안쓰면 중첩클래스입니다.
- 기능적 차이점
	- 중첩클래스는 외부클래스(여기서는 OuterClass)의 참조를 가지지 않지만 내부클래스는 외부클래스의 참조를 가집니다.
	- 중첩클래스는 외부클래스 밖에서 생성이 가능하지만 내부클래스 외부클래스 안에서만 생성할 수 있습니다.

중첩클래스는 외형적으로는 외부클래스 소속인 것 같지만 사실은 큰 연관성이 거의 없습니다. 사실 바깥쪽에 정의해서 사용해도 되는 클래스입니다. 즉, 그냥 일반클래스입니다.

하지만 내부클래스는 외부클래스의 컨텍스트를 참조할 뿐더러 생성도 외부클래스 안에서만 가능합니다. 즉, 외부클래스의 인스턴스 참조를 물고 탄생하는 운명을 지닙니다.

## 중첩클래스와 내부클래스 기능적 차이점 이해

간단한 예시를 통해 중첩클래스와 내부클래스의 기능적 차이점을 이해해 보겠습니다.

먼저 중첩클래스를 예시를 만들어 보겠습니다.
```kotlin
class OuterClass1{
    private val o1 = 1
    val o2 = 2
    class NestedClass {
        private val n1 = 10
        private val n2 = 20
        fun method1() = n1 + n2

        //Unresolved reference: o1
        fun method2() = o1 + n1

		//Unresolved reference: o2
        fun method3() = o2 + n1
    }
}
```

외부클래스(OuterClass1)에 o1과 o2 속성을 정의하고 중첩클래스(OuterClassA1.NestedClass)에 n1과 n2 속성을 정의합니다. 중첩클래스의 메소드 method1()은 자신의 속성인 n1과 n2 연산으로 쓰는데 전혀 문제 없지만, method2()와 method2()는 외부클래스의 o1과 o2는 참조할 수 없다고 에러를 냅니다. 외부클래스와 중첩클래스는 원래 연관성이 없기 때문에 컨텍스트(context)를 공유하지 않는 것이지요.

그럼 위 예시를 조금 고쳐서 내부클래스로 바꿔보겠습니다.

```kotlin
class OuterClass2{
    private val o1 = 1
    val o2 = 2
    inner class InnerClass {
        private val n1 = 10
        private val n2 = 20
        fun method1() = n1 + n2
        fun method2() = o1 + n1
        fun method3() = o2 + n1
    }
}
```
이번에는 외부클래스(OuterClass2)에 내부클래스(InnerClass) 정의했고 속성과 메소드 시그니쳐는 전부 앞서 정의한 중첩클래스와 달라진 것은 없습니다. 하지만 이번에는 에러가 나지 않습니다. 내부클래스는 외부클래스 속성인 o1과 o2를 모두 참조할 수 있음을 확인하세요. 그런데 어째서 inner 키워드를 준 것만으로 외부 클래스의 속성을 참조할 수 있는 것일까요? 다음 예시를 통해서 이해해 보도록 하겠습니다.

```kotlin
fun main() {
	//중첩클래스 인스턴스 생성. 성공!
    val i1 = OuterClass1.NestedClass()
    println("i1.method1() = ${i1.method1()}") //결과 30

    //내부클래스 인스턴스 생성. 실패!
    //Constructor of inner class InnerClass 
    //can be called only with receiver of containing class
    val i2 = OuterClass2.InnerClass()

    //내부클래스 인스턴스 생성함
    val i3 = OuterClsss2().InnerClass()
}
```

위 코드에서 중첩클래스는 외부클래스에 점(.)구문을 통해서 참조해서 인스턴스를 생성했을 뿐 일반클래스처럼 사용할 수 있습니다. 하지만 내부클래스는 같은 방법(`val i2 = OuterClass2.InnerClass()`)으로 해도 인스턴스를 생성할 수 없습니다.  하지만 `val i3 = OuterClsss2().InnerClass()`를 보면 이것은 문제없이 내부클래스를 생성할 수 있습니다. 그 이유는 내부클래스는 외부클래스의 인스턴스의 참조를 물고 탄생하기 때문입니다. `OuterClass2.InnerClass()` 형태는 외부클래스의 인스턴스가 아니므로 내부클래스를 생성할 수 없는 것입니다. 이를 좀 더 이해하기 위해 아래 코드를 보겠습니다. 


```kotlin
class OuterClass2{
    private val o1 = 1
    val o2 = 2
    val inner = InnerClass()
    inner class InnerClass {
        private val n1 = 10
        private val n2 = 20
        fun method1() = n1 + n2
        fun method2() = o1 + n1
        fun method3() = o2 + n1
    }
}
```

위 코드에서 `val inner = InnerClass()` 를 사용한 부분을 보세요. 이처럼 내부클래스의 인스턴스는 이미 외부클래스 인스턴스의 참조 내에서 생성된 것으로 볼 수 있습니다.

```kotlin
fun main() {
    val i1 = OuterClass1.NestedClass()

	//결과 30
    println("i1.method1() = ${i1.method1()}")

    val i2 = OuterClass2()

    //결과 30
    println("i1.method1() = ${i2.inner.method1()}")

    //결과 11
    println("i1.method2() = ${i2.inner.method2()}")

	//결과 12
    println("i1.method3() = ${i2.inner.method3()}")
}
```


이처럼 내부클래스는 외부클래스와 운명을 같이 하므로 만약 내부클래스 인스턴스의 참조를 외부에 공개해서 다른 곳에서 계속 쓰게 되면 외부클래스도 메모리에서 삭제가 되지 않습니다. 이는 원치 않는 메모리 낭비를 초래할 수 있습니다. 그래서 `i2.inner` 처럼 외부에 공개하면 위험합니다.`val inner = InnerClass()`은 사실 private 이어야 안전합니다. 그보다는 나중에 다시 설명하겠지만 `inner class InnerClass`앞에 `private` 키워드를 붙이도록 해서 아에 인스턴스가 바깥쪽에 참조되지 않도록 원천 봉쇄할 수도 있습니다.

이처럼 내부클래스에 대한 이해를 부족하면 오용 소지가 있기 때문에 코틀린에서는 명시적으로 inner 키워드를 줘야 내부클래스로 인식하도록 만들어졌습니다.

참고로 자바에서는 중첩클래스는 static 키워드를 붙입니다. 안 붙이면 내부클래스죠. 그래서 자바는 기본이 내부클래스라 숙련이 되지 않은 개발자가 내부클래스를 아무렇지 않게 양산하여 문제가 되는 경우가 상당합니다. 코틀린 언어 개발자는 이를 개선한 것일겁니다.


## 접근제한자를 중첩클래스와 내부클래스에 붙힌다면?

중첩클래스나 내부클래스 앞에 접근제한자를 붙힐 수 있습니다.

```kotlin
class OuterClass{
    //중첩클래스
    private class NestedClass{
        val a = 1
    }
    fun method1() = NestedClass().a

    //에러 : 'public' function exposes
    //       its 'private' return type NestedClass
    fun method2() = NestedClass()
}
fun main(args: Array<String>) {
	//에러 : Cannot access 'NestedClass': it is private in 'OuterClass'
    println(OuterClass.NestedClass)

    //결과 : 1
    println(OuterClass().method1())
}
```

위 코드처럼 중첩클래스 앞에 private 접근제한자를 붙이면 더 이상 OuterClass.NestedClass 형태로 접근할 수 없고 게다가 그 인스턴스 조차 외부에 공유가 불가해 집니다.

이렇게 접근제한자를 활용해 중첩클래스를 쓰게 되면 외부에 노출하지 않아도 되는 코드를 클래스 단위로 묶어서 관리하는데 도움이 됩니다. 기본적으로는 기본 태생이 일반클래스와 같지만 이런 차이점을 인지하고 있으면 코드 관리에 큰 도움이 될 것 입니다.

```kotlin
class OuterClass{
    //내부클래스
    private inner class InnerClass{
        val a = 1
    }
    fun method1() = InnerClass().a

    //에러 : 'public' function exposes
    //       its 'private' return type NestedClass
    fun method2() = InnerClass()
}
```

내부 클래스도 private 접근제한자를 쓰면 중첩클래스와 동일하게 내부클래스의 인스턴스를 외부에 공개되지 않게 됩니다. 내부클래스는 private 접근제한자를 쓰는게 메모리 관리 측면에서 더욱 안전할 수 있습니다. 

하지만 내부클래스에 특성을 조금 더 생각해보면 private 접근제한자를 붙힌 내부클래스는 효용가치가 많이 떨어집니다. 이유를 간단하게 살펴봅니다.


```kotlin
class MySQL{
   val transactionManager = TransactionManager()
   inner class Query{
      fun excute()
   }
}
```

위와 같은 코드가 있다고 가정한다면 `MySQL().Query()`로 내부클래스의 인스턴스를 생성할 수 있습니다. 이것은 즉, 쿼리를 실행하는 환경이 이미 정해졌다고 볼 수 있습니다. 쿼리를 만드는 시점부터 자연스럽게 외부클래스인 MySQL의 자원을 공유하게 됩니다. 여기서 중요한 점은 MySQL인스턴스에 Query 인스턴스를 주입(injection)하여 참조하는 과정이 전혀 없다는 것입니다. 이 과정이 전부 양자화 되어 중간에 이 관계를 맺는데 로직이 개입될 수도 없습니다.

내부클래스를 쓰는 가장 큰 목적과 이유는 바로 여기에 있습니다. 이에 대한 자세한 이야기는 내부클래스의 활용에 대한 글을 적을 때 다시 언급해 보겠습니다.

## 중첩클래스에도 Companion object를 정의할 수 있습니다.

중첩클래스도 companion object를 정의할 수 있습니다. 중첩클래스는 위치만 다를 뿐 일반클래스와 다를게 없기 때문입니다.

```kotlin
class OuterClass{
    class NestedClass{
        companion object{
            fun method() = 
             "나는 중첩 클래스의 Companion object의 메소드다."
        }
    }
    fun method() = NestedClass.Companion.method()
}
fun main(args: Array<String>) {
	//나는 중첩 클래스의 Companion object의 메소드다.
    println(OuterClass().method())

    //나는 중첩 클래스의 Companion object의 메소드다.
    println(OuterClass.NestedClass.method())

    //나는 중첩 클래스의 Companion object의 메소드다.
    println(OuterClass.NestedClass.Companion.method())
}
```

위 코드에서 `main()` 함수 내에 3줄은 전부 같은 결과를 출력합니다.

첫 번째 줄에서 `OuterClass().method()`는 `OuterClass` 클래스의 객체를 생성하자마자 객체 메소드를 바로 호출했습니다.

두 번째와 세 번째 줄은 중첩클래스의 companion object에 정의된 메소드를 직접 접근해서 호출했습니다. 여기서도 `Companion` 이름은 붙이나 마나인 것을 알 수 있습니다.

OuterClass안에 NestedClass가 정의되었을 뿐이지 NestedClass도 완전한 일반 클래스처럼 동작함을 확인할 수 있습니다. 즉, 중첩클래스도 일반클래스입니다.

```kotlin
class OuterClass{
    companion object{
        private val a = 1
        private val d = 2
    }
    private val b = 2
    class NestedClass{
        private val c = 3
        companion object{
            private val d = 4
            fun getA1() = a
            fun getA2() = OuterClass.a
            fun getA3() = OuterClass.Companion.a

			// 에러 : Unresolved reference: b
            fun getB() = b
            // 에러 : Unresolved reference: c
            fun getC() = c

            fun getD1() = d
            fun getD2() = OuterClass.d
            fun getD3() = OuterClass.Companion.d
        }
        fun getA1() = a
        fun getA2() = OuterClass.a
        fun getA3() = OuterClass.Companion.a

        // 에러 : Unresolved reference: b
        fun getB() = b

        fun getC() = c
        fun getD1() = d
        fun getD2() = OuterClass.d
        fun getD3() = OuterClass.Companion.d
    }
}
fun main(args: Array<String>) {
	// -- (1)
    println(OuterClass.NestedClass.getA1())

    // -- (2)
    println(OuterClass.NestedClass.getA2())

    // -- (3)
    println(OuterClass.NestedClass.getA3())

    // -- (4)
    println(OuterClass.NestedClass.getD1())

    // -- (5)
    println(OuterClass.NestedClass.getD2())

    // -- (6)
    println(OuterClass.NestedClass.getD3())

    val i = OuterClass.NestedClass()
    println(i.getA1()) // -- (7)
    println(i.getA2()) // -- (8)
    println(i.getA3()) // -- (9)
    println(i.getC()) // -- (10)
    println(i.getD1()) // -- (11)
    println(i.getD2()) // -- (12)
    println(i.getD3()) // -- (13)
}
```

위 코드를 보면 `OuterClass`에 companion object의 private 멤버 a, d 속성이 정의되어 있습니다. `OuterClass` 자신에는 b 속성이 정의되어 있습니다. `NestedClass`를 보면 자신의 속성 c가 있고 companion object에는 d가 정의되어 있습니다. 이때 `NestedClass`의 companion object에 정의된 getA(), getD() 메소드는 각각 `OuterClass`와 `NestedClass`의 companion object 속성에 접근하기 때문에 문제없으나 나머지 getB(), getC() 메소드는 에러가 납니다.

main() 함수에 보면 주석 (2), (3) 의 결과는 1임을 금방 알 수 있습니다. 하지만 (1)의 결과도 1입니다. `NestedClass`의 companion object에는 a가 없으므로 자동으로 `OuterClass`의 companion object의 a를 참조하게 됩니다.

그럼 (4)의 결과는 2일까요? 4일까요? 답은 4입니다. 왜냐하면 자신의 companion object의 속성 d가 이미 있기 때문에 (1)의 결과와 다르게 `OuterClass` companion object의 d는 섀도잉(Shadowing, 가려짐) 되기 때문입니다.

주석 (5), (6)는 `NestedClass`의 companion object의 d가 아닌 섀도잉된 `OuterClass`의 companion object의 d에 직접 접근하기 위해 `OuterClass.d` 또는 `OuterClass.Companion.d`를 썼습니다. 결과는 2입니다.

이제 companion object와 함께 중첩클래스를 쓰는 이유가 명확해졌습니다. `OuterClass`에 정의된 companion object의 private 속성에 제약 없이 접근이 가능한 다른 클래스가 필요한 경우 중첩클래스를 쓰면 됩니다. 이렇게 되면 클래스간 공유하는 필요한 로직이나 값이 외부에 노출되지 않고 내부에서만 처리하기 때문에 효율적인 정적 데이타 은닉과 기능의 캡슐화가 가능해집니다.

## 내부클래스에는 Companion object를 쓸 수 없습니다.

반면에 내부클래스는 companion object를 정의할 수 없습니다.

```kotlin
class OuterClass{
    inner class InnerClass{
    	// 에러 발생 - Companion object is not allowed here
        companion object{
            fun method() = 
            	"앗! 난 존재할 수 없었다! 내부 클래스에 Companion object는 있을 수 없다!"
        }
    }
}
```

내부클래스에서 companion object를 못쓰는 이유는 그 의미가 모호하기 때문입니다.

일반클래스와 중첩클래스에서 companion object는 가상 머신 안에서 companion object가 딱 하나만 존재하는 것을 의미합니다.

하지만 내부클래스는 외부클래스의 참조를 가지므로 내부클래스에 companion object를 정의할 수 있다면 이것이 외부클래스 단위로 하나만 있는 것인지 아니면 일반클래스와 마찬가지로 가상 머신 안에서 하나만 있는 것인지 판단하기 힘들어 집니다. 내부클래스의 의미를 볼 때, 둘 다 맞는 생각이거든요. 그래서 내부클래스에는 이 모호함으로 인해 companion object를 사용을 허가하지 않았습니다.

참고로 자바에서도 같은 이유로 내부클래스에 static 키워드를 쓰는 것이 금지되어 있습니다.


## 외부클래스에 companion object를 정의하고 내부클래스 참조 관계를 이해하자.

내부클래스의 인스턴스가 생성되면 외부클래스 인스턴스 참조를 가지기 때문에 내부클래스 내에서 외부클래스의 companion object 속성에 어떤 관계로 참조가 가능한지 확실히 알아볼 필요가 있습니다.

```kotlin
class OuterClass{
    companion object{
        private val a = 1
        private val b = 2
        private val c = 3
        private val d = 4
    }
    private val b = 20
    private val d = 40
    inner class InnerClass{
        private val c = 300
        private val d = 400
        fun getA() = a //--(1)
        fun getB() = b //--(2)
        fun getC() = c //--(3)
        fun getD() = d //--(4)
    }
    fun print() {
        val inner = InnerClass()
        println(inner.getA()) //1
        println(inner.getB()) //20
        println(inner.getC()) //300
        println(inner.getD()) //400
    }
}
fun main(args: Array<String>) {
	//에러! 내부클래스는 외부에서 생성 불가!
    val inner = OuterClass.InnerClass()

    OuterClass().print()
}
```
위 코드에서 `InnerClass`에 정의된 인스턴스 메소드인 주석 (1)~(4)까지 정의된 메소드를 보면 a, b, c, d 속성의 값을 반환합니다. `OuterClass`의 `print()` 함수의 결과는 순서대로 1, 20, 300, 400 입니다. 왜 그런 결과가 나왔는지 자세히 보겠습니다.

주석(1)에서는 a의 값을 반환하는데, a는 `InnerClass`가 아닌 `OuterClass`의 companion object에 정의되어 있습니다. 그래서 1이 출력됩니다.

주석(2)에서는 b값을 반환하는데, b는 `InnerClass`에는 없지만 `OuterClass`의 멤버 b와 companion object에 b가 정의되어 있습니다. 결과값은 20으로 나온 것으로 보아 companion object의 b는 섀도잉 되고 `OuterClass`의 멤버 b의 값이 출력되었습니다. `InnerClass`의 `getB()`가 인스턴스 멤버니 인스턴스가 우선이 되는 것으로 해석하면 편할 것 같습니다.

주석(3)은 c값을 반환합니다. c는 `InnerClass`의 멤버이고 `OuterClass`의 companion object의 멤버입니다. 가장 가까운 자신의 속성 값인 300이 출력됩니다.

주석(4)는 d값을 반환합니다. d는 `InnerClass`, `OuterClass`, 그리고 `OuterClass` companion object의 멤버입니다. 이것도 역시 가장 가까운 자신의 멤버 값인 400이 출력됩니다.

결과를 보면 중첩클래스와 달리 `OuterClass` companion object 멤버 뿐 아니라 `OuterClass`의 멤버 모두 접근이 가능하다는 것을 확인할 수 있습니다.

추가적으로 main() 함수를 보면 내부클래스는 외부에서 객체 생성이 불가능하죠.

지금까지 알게 된 내부클래스의 특성은 외부에서 객체 생성할 수 없고, 내부클래스의 외부 클래스 멤버와 companion object 멤버에 접근할 수 있다는 점입니다. 그리고 내부클래스에는 companion object를 정의할 수 없습니다.

이 특성의 의미를 더욱 짚고 넘어가기 위해 다음 코드를 보시죠.

```kotlin
class OuterClass{
    companion object{
        private val a = 1
        private val b = 2
        private val c = 3
        private val d = 4
    }
    private val b = 20
    private val d = 40
    inner class InnerClass{
        private val c = 300
        private val d = 400

		// -- (1)
        fun getA() = a

        // -- (2)
        fun getB() = b

        // -- (3)
        fun getC() = this.c

        // -- (4)
        fun getD() = this.d

        // -- (5)
        fun getOuterB() = this@OuterClass.b

        // -- (6)
        fun getOuterD() = this@OuterClass.d

        // -- (7)
        fun getOuterCompA() = OuterClass.a

        // -- (8)
        fun getOuterCompC() = OuterClass.c
    }
    fun print() {
        val inner = InnerClass()
        println(inner.getA()) //1
        println(inner.getB()) //20
        println(inner.getC()) //300
        println(inner.getD()) //400
        println(inner.getOuterB()) //20
        println(inner.getOuterD()) //40
        println(inner.getOuterCompA()) //1
        println(inner.getOuterCompC()) //3
    }
}
fun main(args: Array<String>) {
    OuterClass().print()
}
```

이전에 코드를 약간 더 수정 또는 추가한 것입니다. 주석 (1), (2) 는 1, 20입니다. 앞서 설명한 코드와 똑같습니다.

주석 (3)과 (4)는 `this`를 붙였습니다. 이 `this`는 `InnerClass`의 인스턴스 컨텍스트(context)를 지칭합니다. this를 빼도 무방하죠. 하지만 (1)과 (2)에는 this를 붙힐 수 없습니다. a와 b는 `InnerClass`의 속성이 아니기 때문입니다.

주석 (5)와 (6)은 조금 톡특한 `this@OuterClass.(멤버참조)` 형태로 쓰고 있습니다. 결과가 각각 20, 40이 나오는 것으로 보아 여기서 쓰인 b와 d는 `OuterClass`의 멤버를 지칭합니다. 즉, `this@OuterClass`는 `InnerClass` 컨텍스트를 감싸는 `OuterClass`의  컨텍스트를 가리킵니다. 이 점이 매우 중요한데, 외부클래스 인스턴스가 내부클래스의 인스턴스를 참조함과 동시에 내부클래스 인스턴스가 외부클래스의 인스턴스를 참조합니다(순환참조). 내부클래스 인스턴스가 어딘가에 참조되어 있다면 외부클래스 객체는 결코 메모리에서 삭제될 수 없다는 것도 알 수 있습니다. 이 점때문에 내부클래스를 잘못쓰면 메모리 누수가 발생하거나 가비지 컬렉팅(GC)의 성능을 나쁘게 하는 문제가 생겨나기도 합니다. 반면 이전에 언급한 중첩클래스는 외부클래스의 companion object만 참조할 수 있으므로 원천적으로 이 문제가 발생하지 않습니다.

주석 (7), (8) 부분은 `OuterClass`의 companion object의 멤버에 접근했으므로 쉽게 값 1, 3을 출력함을 알 수 있습니다.

## 정리하며 

코틀린 중첩클래스와 내부클래스에 대해서 알아보고 이들 클래스에서 Companion object의 관계도 함께 살펴보았습니다. 공부해 보면서 느끼시겠지만 무엇하나 쉬운게 없습니다. 글이 좀 길지만 이해가 확실히 안되면 다시 한번 읽어보시고, 관련된 자료도 함께 찾아보면서 지식을 공고히 하시기를 바랍니다. 다음에는 내부클래스의 활용에 대해서 다뤄보겠습니다.
