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
[작성중]

지난 글에서는 **코틀린(Kotlin)**의 **Companion object**를 소개했습니다. 자바(Java)의 static 키워드과 다르다 점도 알게 되었습니다.

코틀린의 companion object에 대해 더 이해할 점이 아직 많이 남아 있습니다. 이번 글에서는 중첩클래스와 내부클래스에서 Companion object에 대해 자세히 알아보기로 합니다.

## 중첩클래스에도 Companion object를 정의할 수 있습니다.

중첩클래스(Nested Class)도 companion object를 정의할 수 있습니다.

```kotlin
class OuterClass{
    class NestedClass{
        companion object{
            fun method() = "나는 중첩 클래스의 Companion object의 메소드다."
        }
    }
    fun method() = NestedClass.Companion.method()
}
fun main(args: Array<String>) {
    println(OuterClass().method())
    println(OuterClass.NestedClass.method())
    println(OuterClass.NestedClass.Companion.method())
}
```

위 코드에서 `main()` 함수 내에 3줄은 전부 같은 결과를 출력합니다.

첫 번째 줄에서 `OuterClass().method()`는 `OuterClass` 클래스의 객체를 생성하자마자 객체 메소드를 바로 호출했습니다.

두 번째와 세 번째 줄은 중첩클래스의 companion object에 정의된 메소드를 직접 접근해서 호출했습니다. 여기서도 `Companion` 이름은 붙이나 마나인 것을 알 수 있습니다. OuterClass안에 NestedClass가 정의되었을 뿐이지 NestedClass도 완전한 일반 클래스처럼 동작함을 확인할 수 있습니다. 즉, 중첩클래스도 일반클래스입니다.

중첩클래스를 조금 더 이해하려면 자바의 중첩클래스를 보면 더 편합니다. 자바에서 중첩클래스는 `class` 키워드 앞에 `static`을 붙여야 합니다. 자바에서 키워드 `static`를 클래스 앞에 붙이면 중첩클래스가 되고 `OuterClass`의 `static` 키워드를 붙힌 클래스 멤버를 참조할 수 있게 됩니다(private static 멤버 포함). 비슷하게 생각해보면 코틀린의 경우는 `OuterClass`의 companion object의 멤버에는 접근할 수 있습니다.

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
            //fun getB() = b  // 에러 : Unresolved reference: b
            //fun getC() = c  // 에러 : Unresolved reference: c
            fun getD1() = d
            fun getD2() = OuterClass.d
            fun getD3() = OuterClass.Companion.d
        }
        fun getA1() = a
        fun getA2() = OuterClass.a
        fun getA3() = OuterClass.Companion.a
        //fun getB() = b // 에러 : Unresolved reference: b
        fun getC() = c
        fun getD1() = d
        fun getD2() = OuterClass.d
        fun getD3() = OuterClass.Companion.d
    }
}
fun main(args: Array<String>) {
    println(OuterClass.NestedClass.getA1()) // -- (1)
    println(OuterClass.NestedClass.getA2()) // -- (2)
    println(OuterClass.NestedClass.getA3()) // -- (3)
    println(OuterClass.NestedClass.getD1()) // -- (4)
    println(OuterClass.NestedClass.getD2()) // -- (5)
    println(OuterClass.NestedClass.getD3()) // -- (6)

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

이제 중첩클래스를 쓰는 이유가 명확해졌습니다. `OuterClass`에 정의된 companion object의 private 속성에 제약 없이 접근이 가능한 다른 클래스가 필요한 경우 중첩클래스를 쓰면 됩니다. 이렇게 되면 클래스간 공유하는 필요한 로직이나 값이 외부에 노출되지 않고 내부에서만 처리하기 때문에 효율적인 정적 데이타 은닉과 기능의 캡슐화가 가능해집니다.

## 내부클래스에는 Companion object를 쓸 수 없습니다.

반면에 중첩 클래스에 `inner` 키워드를 붙이면 companion object를 정의할 수 없습니다. 이때는 내부클래스(Inner Class)라 부릅니다.

```kotlin
class OuterClass{
    inner class InnerClass{
        companion object{  // 에러 발생 - Companion object is not allowed here
            fun method() = "앗! 난 존재할 수 없었다! 내부 클래스에 Companion object는 있을 수 없다!"
        }
    }
}
```

내부클래스를 이해해야 그 이유를 알 수 있습니다. 일단 내부클래스가 어떤 특성을 가지고 있는지 확인해 봅니다.

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
        fun getA() = a // -- (1)
        fun getB() = b // -- (2)
        fun getC() = c // -- (3)
        fun getD() = d // -- (4)
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
    //val inner = OuterClass.InnerClass() //에러! 내부클래스는 외부에서 생성 불가!
    OuterClass().print()
}
```
위 코드에서 `InnerClass`에 정의된 인스턴스 메소드인 주석 (1)~(4)까지 정의된 메소드를 보면 a, b, c, d 속성의 값을 반환합니다. `OuterClass`의 `print()` 함수의 결과는 순서대로 1, 20, 300, 400 입니다. 왜 그런 결과가 나왔는지 자세히 보겠습니다.

주석(1)에서는 a의 값을 반환하는데, a는 `InnerClass`가 아닌 `OuterClass`의 companion object에 정의되어 있습니다. 그래서 1이 출력됩니다.

주석(2)에서는 b값을 반환하는데, b는 `InnerClass`에는 없지만 `OuterClass`의 멤버 b와 companion object에 b가 정의되어 있습니다. 결과값은 20으로 나온 것으로 보아 companion object의 b는 섀도잉 되고 `OuterClass`의 멤버 b의 값이 출력되었습니다. `InnerClass`의 `getB()`가 인스턴스 멤버니 인스턴스가 우선이 되는 것으로 해석하면 편할 것 같습니다.

주석(3)은 c값을 반환합니다. c는 `InnerClass`의 멤버이고 `OuterClass`의 companion object의 멤버입니다. 가장 가까운 자신의 속성 값인 300이 출력됩니다.

주석(4)는 d값을 반환합니다. d는 `InnerClass`, `OuterClass`, 그리고 `OuterClass` companion object의 멤버입니다. 이것도 역시 가장 가까운 자신의 멤버 값인 400이 출력됩니다.

결과를 보면 중첩클래스와 달리 `OuterClass` companion object 멤버 뿐 아니라 `OuterClass`의 멤버 모두 접근이 가능하다는 것을 확인할 수 있습니다.

독특한 점은 main() 함수를 보면 내부클래스는 외부에서 객체 생성이 불가능합니다. 어떻게 이런 일이 가능한 것일까요?

지금까지 알게 된 내부클래스의 특성은 외부에서 객체 생성할 수 없고, 내부클래스의 외부 클래스 멤버와 companion object 멤버에 접근할 수 있다는 점입니다. 그리고 내부클래스에는 companion object를 정의할 수 없습니다.

이 특성의 의미를 더욱 짚고 넘어가기 위해 다음 코드를 보세요.

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
        fun getA() = a // -- (1)
        fun getB() = b // -- (2)
        fun getC() = this.c // -- (3)
        fun getD() = this.d // -- (4)
        fun getOuterB() = this@OuterClass.b // -- (5)
        fun getOuterD() = this@OuterClass.d // -- (6)
        fun getOuterCompA() = OuterClass.a // -- (7)
        fun getOuterCompC() = OuterClass.c // -- (8)
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

주석 (3)과 (4)는 `this`를 붙였습니다. 이 `this`는 `InnerClass` 객체 컨텍스트(context)를 지칭합니다. this를 빼도 무방하죠. 하지만 (1)과 (2)에는 this를 붙힐 수 없습니다. a와 b는 `InnerClass`의 멤버가 아니기 때문니다.

주석 (5)와 (6)은 조금 톡특한 `this@OuterClass.(멤버참조)` 형태로 쓰고 있습니다. 결과가 각각 20, 40이 나오는 것으로 보아 여기서 쓰인 b와 d는 `OuterClass`의 멤버를 지칭합니다. 즉, `this@OuterClass`는 `InnerClass` 객체 컨텍스트를 감싸는 `OuterClass`의 객체 컨텍스트를 가리킵니다. 이 점이 매우 중요한데, 외부클래스 객체가 내부클래스의 객체를 참조함과 동시에 내부클래스 객체가 외부클래스의 객체를 참조합니다(순환참조). 내부클래스 객체가 어딘가에 참조되어 있다면 외부클래스 객체는 결코 메모리에서 삭제될 수 없다는 것도 알 수 있습니다. 이 점때문에 내부클래스를 잘못쓰면 메모리 누수가 발생하거나 가비지 컬렉팅(GC)의 성능을 나쁘게 하는 문제가 생겨나기도 합니다. 반면 이전에 언급한 중첩클래스는 외부클래스의 companion object만 참조할 수 있으므로 원천적으로 이 문제가 발생하지 않습니다.

주석 (7), (8) 부분은 `OuterClass`의 companion object의 멤버에 접근했으므로 쉽게 값 1, 3을 출력함을 알 수 있습니다.


내부클래스에서 companion object를 못쓰는 이유는 그 의미가 모호하기 때문입니다.


## Companion Object를 진정으로 활용해보자.

코틀린이 static을 버리고 Companion Object를 쓴다는 점에서 발견할 수 있는 장점은 무엇이 있을까요? 위에서 언급한 특징만으로 코틀린이 정말 매력적인 언어라는 것을 알기는 힘들지 모릅니다. static보다 장점이라기 보다 static 역할을 하면서 companion object 객체를 대신해서 쓴다는 점 외에는 나아 보일 것이 없어 보일 수 있습니다.

하지만 companion object의 특징과 여러가지 다른 특징을 잘 활용하면 개발시 유용하게 쓸 수 있습니다. 제가 경험한 것 중 하나만 예시를 들어보겠습니다. 저는 companion object와 속성 delegate를 사용해 리플렉션(reflection)을 쓰지 않고 동적 자원을 정적 자원으로 쓰는 방법을 소개하겠습니다.




## 아직 정리할 사항

2. 인스턴스에서는 companion object에 정의된 맴버에 접근할 수 없음. 클래스 내부에서만 가능 
3. companion object에 접근제한자를 쓰면? 본인 클래스에서 쓸 수 있는가? 자식에서는?? 
4. companion object는 인터페이스를 상속해서 만들 수 있다. 
5. 자바에서 compaion object를 사용하려면 Compaion 키워드를 붙여서 사용해야 한다.
  - 이를 회피하려면 @JvmStatic(메소드)이나 @JvmField(속성)을 변수나 메소드에 붙여야 한다. 
  - 이외에도 const 키워드를 쓰면 get/set이 만들어지지 않아서 속성에 바로 접근이 가능합니다.(프리미티브 타입과 String만 가능)


