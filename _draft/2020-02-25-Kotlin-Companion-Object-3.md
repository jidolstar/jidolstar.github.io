---
layout: post
title: "코틀린의 Companion Object (3) - JVM과 관계"
date: 2020-02-23 20:00:00 +0900
categories: Kotlin(코틀린)
tags: [코틀린, kotlin, Comapanion object]
author: 지용호
image: assets/images/11.jpg
comments: true
featured: true
hidden: false

---
[작성중]

지난 글에서는 **코틀린(Kotlin)**의 **Companion object**를 소개했습니다. 자바(Java)의 static 키워드과 다르다 점도 알게 되었습니다. 

하지만 코틀린의 companion object에 대해 더 이해할 점이 아직 많이 남아 있습니다. 그래서 이 글에서는 컴파일러 관점에서 companion object를 바라보면서 companion object를 더욱 이해하고 나아가 어떻게 활용될 수 있는지 학습하는 시간을 가져보겠습니다.



## Companion Object를 진정으로 활용해보자. 

코틀린이 static을 버리고 Companion Object를 쓴다는 점에서 발견할 수 있는 장점은 무엇이 있을까요? 위에서 언급한 특징만으로 코틀린이 정말 매력적인 언어라는 것을 알기는 힘들지 모릅니다. static보다 장점이라기 보다 static 역할을 하면서 companion object 객체를 대신해서 쓴다는 점 외에는 나아 보일 것이 없어 보일 수 있습니다. 

하지만 companion object의 특징과 여러가지 다른 특징을 잘 활용하면 개발시 유용하게 쓸 수 있습니다. 제가 경험한 것 중 하나만 예시를 들어보겠습니다. 저는 companion object와 속성 delegate를 사용해 리플렉션(reflection)을 쓰지 않고 동적 자원을 정적 자원으로 쓰는 방법을 소개하겠습니다. 


조금 난도를 높여보겠습니다. 아래 코드를 보겠습니다.

[java]
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
[/java]

클래스 정의를 먼저 자세히 보면 `Parent`의 `method()` 메소드가 open이고 `Child`에서 이를 override했습니다. `main()` 함수에서 첫 번째와 두 번째 실행 결과는 쉽게 예측이 됩니다. 하지만 주석(1)의 결과는 무엇일까요? 분명 `Child` 클래스의 객체를 생성했으나 `Parent` 부모로 형 변환 했습니다. 그럼 이때 호출한 method() 결과는 대체 무엇인가요? 부모 형이니 "나는 부모"라고 출력할까요? 아니면 태생이 자식이니 "나는 자식"일까요?

이 문제는 compaion object문제가 아닙니다. 게다가 코틀린 문제도 아닙니다. 사실 이는 객체지향 언어에서 약속한 **다형성(Polymorphism)**에 대한 질문입니다. 다형성은 두 가지를 만족해야 합니다.

1. **대체 가능성(substitution)** – 어떤 형을 요구한다면 그 형의 자식형으로 그 자리를 대신할 수 있다.
2. **내적 동질성(internal identity)** – 객체는 그 객체를 참조하는 방식에 따라 변화하지 않는다. 즉 업다운캐스팅해도 여전히 최초 생성한 그 객체라는 것입니다.

이 두 가지 조건을 만족하면 다형성을 만족한다고 볼 수 있고 거꾸로 다형성을 만족하면 객체지향 언어라고 볼 수 있습니다. 다형성의 1번 조건인 대체 가능성 때문에 `var c:Parent = Child()`를 할 수 있었던 것입니다. 그리고 2번 조건인 내적 동질성으로 인해 아무리 자식을 부모 형으로 대체해도 자식은 자식이죠. 태어날 때의 본질은 어디 가지 않습니다! 그래서 `p.method()`의 결과는 `Child().method()`결과와 같습니다. 그러므로 결과는 "나는 자식" 입니다.

갑자기 왠~ 객체지향의 다형성... 딴 길로 흘렀습니다.


## 아직 정리할 사항

2. 인스턴스에서는 companion object에 정의된 맴버에 접근할 수 없음. 클래스 내부에서만 가능 
3. companion object에 접근제한자를 쓰면? 본인 클래스에서 쓸 수 있는가? 자식에서는?? 
4. companion object는 인터페이스를 상속해서 만들 수 있다. 
5. 자바에서 compaion object를 사용하려면 Compaion 키워드를 붙여서 사용해야 한다.
  - 이를 회피하려면 @JvmStatic(메소드)이나 @JvmField(속성)을 변수나 메소드에 붙여야 한다. 
  - 이외에도 const 키워드를 쓰면 get/set이 만들어지지 않아서 속성에 바로 접근이 가능합니다.(프리미티브 타입과 String만 가능)


