---
layout: post
title: "코틀린의 Companion Object는 클래스 static인가?"
date: 2020-02-20 20:00:00 +0900
categories: kotlin 코틀린
tags: [코틀린, kotlin, Comapanion object, static]
author: 지용호
image: assets/images/11.jpg
comments: true
featured: true
hidden: false

---

**코틀린(Kotlin)**의 ==Companion object==는 단순히 자바(Java)의 static 키워드를 대체하기 위해서 탄생했을까요? 이 갑작스런 질문은 왜 코틀린에서 static을 안쓰게 되었는지 이해하는데 큰 도움이 될 수 있습니다. 

자바의 static 키워드는 클래스 맴버(member)임을 지정하기 위해 사용합니다. static이 붙은 변수와 메소드를 각각 클래스 변수, 클래스 메소드라 부릅니다. 반면, static이 붙지 않은 클래스 내의 변수와 메소드는 각각 인스턴스 변수, 인스턴스 메소드라 합니다. static이 붙은 맴버는 클래스가 메모리에 적재될 때 자동으로 생성되므로 인스턴스 생성없이 클래스명 다음에 점(.)을 쓰면 바로 참조할 수 있습니다.  

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

자바(Java) 개발 경험이 있는 사람들에게는 코틀린(Kotin)에 static이 없다는 것에 당황할 수 있습니다. 대신 코틀린은 `companion object`라는 키워드와 함께 블록({})안에 맴버를 구성합니다. 

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

여기까지만 보면 단순히 static 키워드 대신 companion object 블록으로 대체한 느낌만 듭니다. 그러면 아래처럼 static 키워드를 써도 문제 없다고 판단할지도 모르겠습니다. 

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

코틀린은 static을 버리고 companion object를 도입했으며 더욱 명쾌하고 멋진 방법으로 문제를 해결하는데 도움을 주도록 했습니다. 이 글은 companion object와 static의 차이점을 이해하고 static의 한계를 파악하는 동시에 companion object의 장점을 학습하는데 도움을 주는데 목적이 있습니다.

## 코틀린의 class 키워드 기초 

compaion object를 다루기 전에 먼저 코틀린의 class와 object를 먼저 이해해야 할 것 같습니다. 사실 이 주제만으로도 엄청난 분량이지만 여기서는 간단하게 사용법만 소개합니다.

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

클래스 `WhoAmI`를 정의하면서 생성자의 인자로 `val name:String`을 받습니다. 자바에 익숙하나 코틀린을 처음 접하면 이게 무슨 생성자인가 생각이 들 수 있습니다. `WhoAmI`클래스를 자바 개발자가 이해하기 쉽게 풀어 쓰면 다음과 같습니다.

```kotlin
class WhoAmI{
    private val name:String
    constructor(name:String){
        this.name = name
    }
    fun myNameIs() = "나의 이름은 ${name}입니다."
}
```

위처럼 클래스 생성자는 `constructor` 키워드를 써서 정의합니다. 그리고 생성자에서 받은 인자 `name:String`의 값을 속성인 `this.name`을 통해 `private val name:String`에 할당하고 있습니다. 이 코드는 자세히 보면 너무 중복이 많습니다. 그래서 코틀린 언어 설계자는 이를 단순화했습니다. 즉, 클래스를 정의하자 마자 바로 생성자 및 속성까지 정의한 것입니다. 덕분에 코틀린으로 코드를 짜면 자바보다 훨씬 짧아집니다(이것 때문만은 아니지만 코틀린으로 코딩하면 경험상 코딩량이 최소 50%이상 줄어드는 것 같습니다. 아니면 그보다 더....).

각설하고, 여기서 중요한 것은 클래스로 부터 객체를 생성하기 위해 `val m1 = WhoAmiI("영수")`처럼 합니다. 자바처럼 객체 생성을 위해 ==`new` 키워드를 쓰지 않으며 클래스로 부터 인스턴스를 생성하기 위해 클래스 명 뒤에 괄호()를 붙힌다는 점==을 기억하세요 (코틀린에서 `new`를 쓰지 않는다는 점이 과연 무슨 의미인지 생각해보는 것도 나쁘지 않을 것 같습니다). 


## 코틀린의 object 키워드 기초 

코틀린에는 자바에 없는 독특한 ==싱글턴(singletion; 인스턴스가 하나만 있는 클래스)== 선언 방법이 있습니다. 아래처럼 `class` 키워드 대신 `object` 키워드를 사용하면 됩니다. 

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

object는 특정 클래스나 인터페이스를 확장(`var obj = object:MyClass(){}`또는 `var obj = object:MyInterface{}`)해 만들 수 있으며 위처럼 선언문이 아닌 표현식(`var obj = object{}`)으로 생성할 수 있습니다. 싱글톤이기 때문에 시스템 전체에서 수행해야 하는 기능(메소드로 정의)을 수행하는데는 큰 도움이 될 수 있지만 전역 상태를 유지하는데 쓰면 스레드 경합등으로 위험할 수 있으니 주의해서 사용해야 합니다. 이 글은 object가 아닌 companion object를 다룰 것이므로 이 정도만 소개하는 것으로 마무리 짓겠습니다. 

앞서 미리 말씀드린 companion object는 클래스 내부에 정의되는 object의 특수한 형태입니다. 이제 companion object를 자세히 알아봅시다. 


## 코틀린의 Companion Object는 static이 아닙니다.

사실 코틀린 companion object는 static이 아니며 사용하는 입장에서 static처럼 동작하는 것처럼 보일 뿐입니다. 다음 코드를 보세요. 

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

`MyClass2` 클래스에 companion object를 만들어 2개의 맴버를 정의했습니다. 이를 사용하는 main()함수를 보면 이 맴버에 접근하기 위해 ==클래스명.Companion== 형태로 쓴 것을 확인할 수 있습니다. 이로써 유추할 수 있는 것은 `companion object{}`는 `MyClass2` 클래스가 메모리에 적재되면서 함께 생성되어 동반(companion)되는 객체이고 이 동반 객체는 `클래스명.Companion`으로 접근할 수 있다는 점입니다. (클래스와 동반자라고 하면 정감이 가려나요?)

```kotlin
fun main(args: Array<String>) {
	//사실은 MyClass2.맴버는 MyClass2.Companion.맴버의 축약표현이다.
    println(MyClass2.prop)
    println(MyClass2.method())
}
```

그래서 위 코드에서 `MyClass2.prop`와 `MyClass2.method()`는 `MyClass2.Companion.prop`과 `MyClass2.Companion.method()` 대신 쓰는 ==축약 표현==일 뿐이라는 점을 이해해야 합니다. 언어적으로 지원하는 축약 표현 때문에 companion object가 static으로 착각이 드는 것입니다. 

## 코틀린의 Companion Object는 객체입니다.

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

위 코드에서 주석 (1)을 확인해 주세요. companion object는 객체이므로 변수에 할당할 수 있습니다. 그리고 할당한 변수에서 점(.)으로 `MyClass2`에 정의된 companion object의 맴버에 접근할 수 있습니다. 이렇게 변수에 할당하는 것은 자바의 클래스에서 static 키워드로 정의된 맴버엔 불가능한 방법입니다. 

위 코드에서 주석 (2)는 `.Companion`을 빼고 직접 `MyClass2`로 할당한 것입니다. 이것도 또한 `MyClass2`에 정의된 companion object입니다. 위에서 `MyClass2.Companion.prop` 대신 `MyClass2.prop` 로 해도 동일하다는 점을 생각해보면 쉽게 가능함을 유추할 수 있습니다. 즉, 클래스의 companion object는 그 클래스 이름만으로도 참조 접근이 가능합니다.

`static` 키워드를 쓰면 클래스 맴버를 companion object처럼 하나의 독립된 객체로 여겨 질 수 없겠죠? 이것도 또한 `static`과 큰 차이점이기도 합니다.

## 코틀린 Companion object 이름 짓기 

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

하지만 주석 (5)처럼 기존 이름인 `Companion`을 쓰면 ==Unresolved reference: Companion== 에러가 납니다. 

```kotlin
    val comp4:MyClass3.MyCompanion = MyClass3
    println(comp4.prop)
    println(comp4.method())

```

위 코드처럼 타입 추론(코틀린은 타입 추론 때문에 코드량이 급격이 줄어든다)을 사용하지 않는다면 MyClass3의 결과는 MyClass3.MyCompanion이기 때문에 이렇게 사용해도 컴파일 에러없이 정상 동작할 수 있음을 유추할 수 있다.

## 코틀린 Companion Object 활용 


## 정리하며 

1. companion object는 클래스 내에서 딱 한번만 사용할 수 있음
2. 인스턴스에서는 companion object에 정의된 맴버에 접근할 수 없음 
3. companion object에 별명을 지을 수 있다. 별명을 지으면 외부에서 Compaion으로 더이상 접근할 수 없으며 별명으로만 접근이 가능하다.
4. companion object는 인터페이스를 상속해서 만들 수 있다. 
5. 자바에서 compaion object를 사용하려면 Compaion 키워드를 붙여서 사용해야 한다.
  - 이를 회피하려면 @JvmStatic(메소드)이나 @JvmField(속성)을 변수나 메소드에 붙여야 한다. 
  - 이외에도 const 키워드를 쓰면 get/set이 만들어지지 않아서 속성에 바로 접근이 가능합니다.(프리미티브 타입과 String만 가능)
6. 내부클래스에도 Companion object를 정의할 수 있나?
7. inner 클래스에도 CO를 정의할 수 있나?

