---
layout: post
title: "코틀린의 Companion Object는 자바의 static과 같은 것인가? (1)"
date: 2020-02-20 20:00:00 +0900
categories: kotlin 코틀린
tags: [코틀린, kotlin, Comapanion object, static]
author: 지용호
image: assets/images/11.jpg
comments: true
featured: true
hidden: false

---

**코틀린(Kotlin)**의 **Companion object**는 단순히 자바(Java)의 static 키워드를 대체하기 위해서 탄생했을까요? 이 갑작스러운 질문은 왜 코틀린에서 static을 안 쓰게 되었는지 이해하는 데 큰 도움이 될 수 있습니다.

자바의 static 키워드는 클래스 멤버(member)임을 지정하기 위해 사용합니다. static이 붙은 변수와 메소드를 각각 클래스 변수, 클래스 메소드라 부릅니다. 반면, static이 붙지 않은 클래스 내의 변수와 메소드는 각각 인스턴스 변수, 인스턴스 메소드라 합니다. static이 붙은 멤버는 클래스가 메모리에 적재될 때 자동으로 함께 생성되므로 인스턴스 생성 없이도 클래스명 다음에 점(.)을 쓰면 바로 참조할 수 있습니다.

```java
public final class MyClass{
  static public final String TEST = "test";  //클래스 변수
  static public method(int i):int{     //클래스 메소드
    return i + 10
  }
}

System.out.println(MyClass.TEST);		//test
System.out.println(MyClass.method(1)); 	//11
```

자바(Java) 개발 경험이 있는 사람들에게는 코틀린(Kotin)에 static이 없다는 사실에 당황할 수 있습니다. 대신 코틀린은 `companion object`라는 키워드와 함께 블록({}) 안에 멤버를 구성합니다.

```kotlin
class MyClass{
    companion object{
        val TEST = "test"
        fun method(i:Int) = i + 10
    }
}
fun main(args: Array<String>){
    println(MyClass.TEST);		//test
    println(MyClass.method(1)); 	//11
}
```

여기까지만 보면 단순히 static 키워드 대신 companion object 블록으로 대체한 느낌만 듭니다. 그러면 아래처럼 static 키워드를 써도 문제없다고 판단할지도 모르겠습니다.

```kotlin
class MyClass{
    static val TEST = "test"
    static fun method(i:int) = i + 10
}
fun main(args: Array<String>){
    println(MyClass.TEST);		//test
    println(MyClass.method(1)); 	//11
}
```

코틀린은 static을 버리고 companion object를 도입했으며 더욱 명쾌하고 멋진 방법으로 문제를 해결하는 데 도움을 주도록 했습니다. 이 글은 companion object와 static의 차이점을 이해하고 static의 한계를 파악하는 동시에 companion object의 장점을 학습에 도움을 주는 데 목적이 있습니다.

## 코틀린의 class 키워드 기초

companion object를 다루기 전에 먼저 코틀린의 class와 object를 먼저 이해해야 할 것 같습니다. 사실 이 주제만으로도 엄청난 분량이지만 여기서는 간단하게 사용법만 소개합니다.

```kotlin
class WhoAmI(private val name:String){
    fun myNameIs() = "나의 이름은 ${name}입니다."
}
fun main(args: Array<String>){
    val m1 = WhoAmI("영수")
    val m2 = WhoAmI("미라")
    println(m1.myNameIs()) //나의 이름은 영수입니다.
    println(m2.myNameIs()) //나의 이름은 미라입니다.
}
```

클래스 `WhoAmI`를 정의하면서 생성자의 인자로 `val name:String`을 받습니다. 자바에 익숙하나 코틀린을 처음 접한 분에게는 이게 무슨 생성자인가 생각이 들 수 있습니다. `WhoAmI`클래스를 자바 개발자가 이해하기 쉽게 풀어 쓰면 다음과 같습니다.

```kotlin
class WhoAmI{
    private val name:String
    constructor(name:String){
        this.name = name
    }
    fun myNameIs() = "나의 이름은 ${name}입니다."
}
```

위처럼 클래스 생성자는 `constructor` 키워드를 써서 정의합니다. 그리고 생성자에서 받은 인자 `name:String`의 값을 속성인 `this.name`을 통해 `private val name:String`에 할당하고 있습니다. 이 코드는 자세히 보면 너무 중복이 많습니다. 그래서 코틀린 언어 설계자는 이를 단순화했습니다. 즉, 클래스를 정의하자마자 바로 생성자 및 속성까지 정의한 것입니다. 덕분에 코틀린으로 코드를 짜면 자바보다 훨씬 짧아집니다(이것 때문만은 아니지만 코틀린으로 코딩하면 경험상 코딩량이 최소 50% 이상 줄어드는 것 같습니다. 아니면 그보다 더....).

각설하고, 여기서 중요한 것은 클래스로부터 객체를 생성하기 위해 `val m1 = WhoAmiI("영수")`처럼 한다는 점입니다. 자바는 객체 생성을 위해 `new` 키워드를 쓰지만 **코틀린에서는 `new` 키워드 없이 클래스 명 뒤에 괄호()를 붙인다는 점**을 기억하세요(코틀린에서 `new`를 쓰지 않는다는 점이 과연 무슨 의미인지 생각해보는 것도 나쁘지 않을 것 같습니다). 마치 함수 호출처럼요.


## 코틀린의 object 키워드 기초

코틀린에는 자바에 없는 독특한 싱글턴(singletion; 인스턴스가 하나만 있는 클래스) 선언 방법이 있습니다. 아래처럼 `class` 키워드 대신 `object` 키워드를 사용하면 됩니다.

```kotlin
object MySingleton{
    val prop = "나는 MySingleton의 속성이다."
    fun method() = "나는 MySingleton의 메소드다."
}
fun main(args: Array<String>){
    println(MySingleton.prop);		//나는 MySingleton의 속성이다.
    println(MySingleton.method()); 	//나는 MySingleton의 메소드다.
}
```

object는 특정 클래스나 인터페이스를 확장(`var obj = object:MyClass(){}`또는 `var obj = object:MyInterface{}`)해 만들 수 있으며 위처럼 선언문이 아닌 표현식(`var obj = object{}`)으로 생성할 수 있습니다. 싱글톤이기 때문에 시스템 전체에서 쓸 기능(메소드로 정의)을 수행하는 데는 큰 도움이 될 수 있지만, 전역 상태를 유지하는 데 쓰면 스레드 경합 등으로 위험할 수 있으니 주의해서 사용해야 합니다.

언어 수준에서 안전한 싱글턴을 만들어 준다는 점에서 object는 매우 유용합니다.

이 글은 object가 아닌 companion object를 다룰 것이므로 이 정도만 소개하는 것으로 마무리 짓겠습니다. 앞서 미리 말씀드린 companion object는 클래스 내부에 정의되는 object의 특수한 형태입니다. 이제 companion object를 자세히 알아봅시다. 


## Companion object는 static이 아닙니다.

사실 코틀린 companion object는 static이 아니며 사용하는 입장에서 static으로 동작하는 것처럼 보일 뿐입니다. 다음 코드를 보세요. 

```kotlin
class MyClass2{
    companion object{
        val prop = "나는 Companion object의 속성이다."
        fun method() = "나는 Companion object의 메소드다."
    }
}
fun main(args: Array<String>) {
    //사실은 MyClass2.맴버는 MyClass2.Companion.맴버의 축약표현이다.
    println(MyClass2.Companion.prop)
    println(MyClass2.Companion.method())
}
```

`MyClass2` 클래스에 companion object를 만들어 2개의 멤버를 정의했습니다. 이를 사용하는 main() 함수를 보면 이 멤버에 접근하기 위해 `클래스명.Companion` 형태로 쓴 것을 확인할 수 있습니다. 이로써 유추할 수 있는 것은 `companion object{}`는 `MyClass2` 클래스가 메모리에 적재되면서 함께 생성되는 동반(companion)되는 객체이고 이 동반 객체는 `클래스명.Companion`으로 접근할 수 있다는 점입니다(클래스와 동반자라고 하면 정감이 가려나요?).

```kotlin
fun main(args: Array<String>) {
    //사실은 MyClass2.맴버는 MyClass2.Companion.맴버의 축약표현이다.
    println(MyClass2.prop)
    println(MyClass2.method())
}
```

위 코드에서 `MyClass2.prop`와 `MyClass2.method()`는 `MyClass2.Companion.prop`과 `MyClass2.Companion.method()` 대신 쓰는 **축약 표현**일 뿐이라는 점을 이해해야 합니다. 언어적으로 지원하는 축약 표현 때문에 companion object가 static으로 착각이 드는 것입니다.

## Companion Object는 객체입니다.

Companion object에서 기억해야 할 중요한 점은 객체라는 것입니다. 그래서 다음과 같은 코딩이 가능해 집니다.

```kotlin
class MyClass2{
    companion object{
        val prop = "나는 Companion object의 속성이다."
        fun method() = "나는 Companion object의 메소드다."
    }
}
fun main(args: Array<String>) {
    println(MyClass2.Companion.prop)
    println(MyClass2.Companion.method())

    val comp1 = MyClass2.Companion  //--(1)
    println(comp1.prop)
    println(comp1.method())

    val comp2 = MyClass2  //--(2)
    println(comp2.prop)
    println(comp2.method())
}
```

위 코드에서 주석 (1)을 확인해 주세요. companion object는 객체이므로 변수에 할당할 수 있습니다. 그리고 할당한 변수에서 점(.)으로 `MyClass2`에 정의된 companion object의 맴버에 접근할 수 있습니다. 이렇게 변수에 할당하는 것은 자바의 클래스에서 static 키워드로 정의된 멤버로는 불가능한 방법입니다.

위 코드에서 주석 (2)는 `.Companion`을 빼고 직접 `MyClass2`로 할당한 것입니다. 이것도 또한 `MyClass2`에 정의된 companion object입니다. 위에서 `MyClass2.Companion.prop` 대신 `MyClass2.prop` 로 해도 같다는 점을 생각해보면 쉽게 가능함을 유추할 수 있습니다. 꼭 기억해야 할 것은 **클래스 내 정의된 companion object는 클래스 이름만으로도 참조 접근이 가능**합니다. 이 특징은 코틀린 설계자의 신의 한 수인지도 모릅니다(왜일까요? ^^).

여기서 알 수 있듯이 `static` 키워드만으로는 클래스 멤버를 companion object처럼 하나의 독립된 객체로 여겨질 수 없겠죠? 이것도 또한 `static`과 큰 차이점이기도 합니다.

## Companion object에 이름을 지을 수 있습니다.

companion object의 기본 이름은 `Companion`입니다. 앞서 `MyClass2.Companion.prop`처럼 사용할 수 있음을 기억해 보시면 이해가 됩니다. 이 이름은 바꿀 수 있습니다.

```kotlin
class MyClass3{
    companion object MyCompanion{  // -- (1)
        val prop = "나는 Companion object의 속성이다."
        fun method() = "나는 Companion object의 메소드다."
    }
}
fun main(args: Array<String>) {
    println(MyClass3.MyCompanion.prop) // -- (2)
    println(MyClass3.MyCompanion.method())

    val comp1 = MyClass3.MyCompanion // -- (3)
    println(comp1.prop)
    println(comp1.method())

    val comp2 = MyClass3 // -- (4)
    println(comp2.prop)
    println(comp2.method())
    
    val comp3 = MyClass3.Companion // -- (5) 에러발생!!!
    println(comp3.prop)
    println(comp3.method())
}

```
위 코드에서 주석 (1)을 확인해 주세요. companion object 이름을 `MyCompanion`으로 지었음을 확인하세요. 주석 (2), (3)을 보시면 이제 기본 이름인 `Companion` 대신 `MyCompanion`을 사용할 수 있습니다.

그러나 여전히 주석 (4)를 보시면 생략할 수 있습니다.

하지만 주석 (5) 처럼 기존 이름인 `Companion`을 쓰면 **Unresolved reference: Companion** 에러가 납니다.

```kotlin
val comp4:MyClass3.MyCompanion = MyClass3
println(comp4.prop)
println(comp4.method())
```

위 코드처럼 타입 추론(참고로 코틀린은 타입 추론 때문에 코드량이 급격히 줄어듭니다)을 사용하지 않고 명시적으로 타입을 정하면 MyClass3의 결과는 MyClass3.MyCompanion이기 때문에 이렇게 사용해도 컴파일 에러 없이 정상 동작할 수 있음을 유추할 수 있습니다.

## 클래스내 Companion Object는 딱 하나만 쓸 수 있습니다.

클래스 내에 2개 이상 companion object를 쓰는 것은 안 됩니다. 지금까지 학습한 것을 유추해 보면 당연한 건데 코틀린은 클래스 명만으로 companion object 객체를 참조할 수 있기 때문에 한 번에 2개를 참조하는 것은 애초부터 불가능한 것이지요. 

```kotlin
class MyClass5{
    companion object{
        val prop1 = "나는 Companion object의 속성이다."
        fun method1() = "나는 Companion object의 메소드다."
    }
    companion object{ // -- (1) 에러발생!! Only one companion object is allowed per class
        val prop2 = "나는 Companion object의 속성이다."
        fun method2() = "나는 Companion object의 메소드다."
    }
}
```

위처럼 만들면 `Only one companion object is allowed per class` 에러가 발생할 것입니다.

```kotlin
class MyClass5{
    companion object MyCompanion1{
        val prop1 = "나는 Companion object의 속성이다."
        fun method1() = "나는 Companion object의 메소드다."
    }
    companion object MyCompanion2{
        val prop2 = "나는 Companion object의 속성이다."
        fun method2() = "나는 Companion object의 메소드다."
    }
}
```
위 코드처럼 companion object 이름을 별도로 부여해도 마찬가지입니다.

덕분에 자바에서 static 멤버를 클래스에 아무 데나 쓰는 게 제약이 없었다면 코틀린에서는 자동으로 한곳에 모이게 됩니다. 물론 이 목적으로 companion object를 한 개만 있도록 코틀린이 설계된 것은 아닙니다.

## 인터페이스 내에도 Companion Object를 정의할 수 있습니다.

코틀린 인터페이스 내에 companion object를 정의할 수 있습니다. 덕분에 인터페이스 수준에서 상수항을 정의할 수 있고, 관련된 중요 로직을 이곳에 기술할 수 있습니다. 이 특징을 잘 활용하면 설계하는 데 도움이 될 것입니다.

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

이제 위 코드가 이해되시죠? 완벽히 이해가 안 되면 다시 위에서부터 차근차근 읽으세요.

뇌피셜이지만 이렇게 보면 인터페이스의 해석 및 동작도 클래스와 별반 차이가 없어 보입니다. `interface` 키워드와 `class` 키워드만 다를 뿐이지 기본 내부 동작은 일정 부분 똑같다는 생각이 듭니다. 결국, 언어 수준에서 살펴보면 인터페이스의 기본기능에 클래스로 만들고 확장한 것 아닐까요? 클래스나 인터페이스 모두 객체일 뿐이며 특수한 역할을 위해 조금씩 변형했다는 느낌이 듭니다. 코틀린 설계자는 무엇인가 언어 동작의 일관성을 만들기 위해 많은 연구를 했음을 짐작하게 합니다. 코틀린에 대해 더 깊은 세계를 알고 싶게 하는 흥미로운 사실입니다. 언어 설계 수준에서 학습해 보고 싶네요.

## 내부 클래스에도 Companion object를 정의할 수 있습니다.

물론 클래스 내부에 있는 클래스도 companion object를 정의할 수 있습니다.
```kotlin
class MyClass4{
    class MyInnerClass{
        companion object{
            fun method() = "나는 내부 클래스의 Companion object의 메소드다."
        }
    }
    fun method() = MyInnerClass.method() //MyInnerClass.Companion.method()와 같음
}
fun main(args: Array<String>) {
    println(MyClass4().method()) //나는 내부 클래스의 Companion object의 메소드다.
    println(MyClass4.MyInnerClass.method()) //나는 내부 클래스의 Companion object의 메소드다.
    println(MyClass4.MyInnerClass.Companion.method()) //나는 내부 클래스의 Companion object의 메소드다.
}
```

메인 함수에서 첫 번째 줄은 실행 시 `MyClass4().method()`는 `MyClass4` 클래스의 객체를 생성하자마자 객체의 메소드를 바로 호출한 것입니다. 두 번째와 세 번째 줄은 내부 클래스의 companion object에 정의된 메소드를 직접 접근해서 호출했습니다. 여기서도 `Companion` 이름은 붙이나 마나인 것을 알 수 있습니다.


반면에 아래처럼 내부 클래스에 `inner` 키워드를 붙이면 companion object를 정의할 수 없습니다.

```kotlin
class MyClass4{
    inner class MyInnerClass{
        companion object{  // 에러 발생 - Companion object is not allowed here
            fun method() = "나는 내부 클래스의 Companion object의 메소드다."
        }
    }
}
```

왜 그럴까요????? `inner class`를 정확히 공부하면 이해가 될 순간이 올 것 같습니다. 나중에 정확히 알게 되면 글을 정정하겠습니다.


## 부모 클래스의 Companion object는 가려집니다(쉐도잉, shadowing).

부모 클래스를 상속한 자식 클래스가 있다면 부모 클래스에 선언한 companion object는 자식 클래스에 선언한 companion object와 관계가 없는 것처럼 동작합니다. 그 이유를 알아가 봅시다.

```kotlin
open class Parent{
    companion object{
        val prop = "나는 부모"
    }
    fun method0() = prop //Companion.prop과 동일
}
class Child:Parent(){
    companion object{
        val prop = "나는 자식"
    }
    fun method1() = prop //Companion.prop 과 동일
}
fun main(args: Array<String>) {
    println(Parent().method0()) //나는 부모
    println(Child().method0()) //나는 부모
    println(Child().method1()) //나는 자식

    println(Parent.prop) //나는 부모
    println(Child.prop) //나는 자식

    println(Parent.Companion.prop) //나는 부모
    println(Child.Companion.prop) //나는 자식
}
```

위 코드를 자세히 보면 부모/자식 둘 다 자신의 companion object의 prop의 값을 반환하고 있습니다. main() 함수에서 예상대로 결과가 나왔습니다. 다만 조금 헷갈릴 수 있는 부분은 `Child().method0()`의 결과일 것입니다. `method0()` 메소드는 `Parent`클래스 것입니다. 그래서 부모의 companion object의 prop값인 "나는 부모"가 출력되었습니다.

조금 난도를 높여보겠습니다. 아래 코드를 보겠습니다.

```kotlin
open class Parent{
    companion object{
        val prop = "나는 부모"
    }
    open fun method() = prop //Companion.prop과 동일
}
class Child:Parent(){
    companion object{
        val prop = "나는 자식"
    }
    override fun method() = prop //Companion.prop 과 동일
}
fun main(args: Array<String>) {
    println(Parent().method()) //나는 부모
    println(Child().method()) //나는 자식

    val p:Parent = Child()
    println(p.method()) // -- (1)
}
```

클래스 정의를 먼저 자세히 보면 `Parent`의 `method()` 메소드가 open이고 `Child`에서 이를 override했습니다. `main()` 함수에서 첫 번째와 두 번째 실행 결과는 쉽게 예측이 됩니다. 하지만 주석(1)의 결과는 무엇일까요? 분명 `Child` 클래스의 객체를 생성했으나 `Parent` 부모로 형 변환 했습니다. 그럼 이때 호출한 method() 결과는 대체 무엇인가요? 부모 형이니 "나는 부모"라고 출력할까요? 아니면 태생이 자식이니 "나는 자식"일까요?

이 문제는 compaion object문제가 아닙니다. 게다가 코틀린 문제도 아닙니다. 사실 이는 객체지향 언어에서 약속한 **다형성(Polymorphism)**에 대한 질문입니다. 다형성은 두 가지를 만족해야 합니다.

1. **대체 가능성(substitution)** – 어떤 형을 요구한다면 그 형의 자식형으로 그 자리를 대신할 수 있다.
2. **내적 동질성(internal identity)** – 객체는 그 객체를 참조하는 방식에 따라 변화하지 않는다. 즉 업다운캐스팅해도 여전히 최초 생성한 그 객체라는 것입니다.

이 두 가지 조건을 만족하면 다형성을 만족한다고 볼 수 있고 거꾸로 다형성을 만족하면 객체지향 언어라고 볼 수 있습니다. 다형성의 1번 조건인 대체 가능성 때문에 `var c:Parent = Child()`를 할 수 있었던 것입니다. 그리고 2번 조건인 내적 동질성으로 인해 아무리 자식을 부모 형으로 대체해도 자식은 자식이죠. 태어날 때의 본질은 어디 가지 않습니다! 그래서 `p.method()`의 결과는 `Child().method()`결과와 같습니다. 그러므로 결과는 "나는 자식" 입니다.

갑자기 왠~ 객체지향의 다형성... 딴 길로 흘렀습니다.

다른 재미있는(?) 실험을 해봅니다. 여기서 진짜 알려드리고 싶은 내용입니다. 일단 자식의 companion object에 이름을 부여합니다.

```kotlin
open class Parent{
    companion object{
        val prop = "나는 부모"
    }
    fun method0() = prop
}
class Child:Parent(){
    companion object ChildCompanion{ // -- (1) ChildCompanion로 이름을 부여했어요.
        val prop = "나는 자식"
    }
    fun method1() = prop
    fun method2() = ChildCompanion.prop
    fun method3() = Companion.prop
}
fun main(args: Array<String>) {
    val child = Child()
    println(child.method0()) //나는 부모
    println(child.method1()) //나는 자식
    println(child.method2()) //나는 자식
    println(child.method3()) // -- (2)
}
```

위 코드에서 주석 (1)에 자식 클래스의 companion object에 `ChildCompanion`로 이름을 부여했습니다. 그리고 자식 클래스에 3개의 메소드를 정의했습니다. `child.method0()`은 부모의 method이므로 어렵지 않게 "나는 부모"가 출력된 것을 예상할 수 있습니다. 그리고 `child.method1()`과 `child.method2()`도 역시 자식의 companion object의 속성을 가리킨다는 것을 알 수 있습니다.

이제 진짜 문제인데, 주석(2)는 어떤 결과가 나올까요? 답부터 말씀드리면 "나는 부모"입니다. 놀랍지 않나요? 자식의 companion object를 뛰어넘어서 부모의 companion object에 직접 접근이 가능하게 되는 순간입니다. 그도 그럴 것이 `fun method3() = Companion.prop`에서 Companion.prop를 하면 이때 `Companion`은 자식 것이 아닙니다. 왜냐하면 자식은 `ChildCompanion`로 이름을 바꿨으니깐요. 그래서 여기서  `Companion`은 부모가 됩니다.

아래 코드에서는 부모의 companion object마저 이름을 붙여봅니다.

```kotlin
open class Parent{
    companion object ParentCompanion{ // -- (1) ParentCompanion로 이름을 부여했어요.
        val prop = "나는 부모"
    }
    fun method0() = prop
}
class Child:Parent(){
    companion object ChildCompanion{
        val prop = "나는 자식"
    }
    fun method1() = prop
    fun method2() = ChildCompanion.prop
    fun method3() = Companion.prop // -- (2) Unresolved reference: Companion 에러!!
}
```

위 코드에서 주석(1)을 보시면 부모 companion object의 이름을 ParentCompanion로 했습니다. 그러자마자 주석(2) 부분은 컴파일 에러가 납니다. 에러가 안 날려면 `fun method3() = Companion.prop`이 아닌 `fun method3() = ParentCompanion.prop` 바꿔야 합니다.

이 실험을 통해 우리는 부모와 자식의 companion object에 이름을 붙이지 않으면 부모의 companion object는 자식 입장에서 섀도잉(shadowing) 되어 감춰질 뿐임을 우리는 알 수 있었습니다.


## 정리하며 
조금 코틀린 companion object에 대해서 이해가 되셨나요? 자바의 static과는 다르며 더 많은 일을 할 수 있다는 것도 느껴지시나요? 하지만 이제 맛보기만 했을 뿐입니다. companion object에 대해서 더 학습이 필요하지만, 글이 많이 길어지는 관계로 이 정도로 마무리 짓고 다음에 2번째 글을 올리도록 하겠습니다. 긴 글 읽어주셔서 감사하며, 부족한 점 있거나 잘못된 내용이 있다면 메일 부탁드리겠습니다. 


