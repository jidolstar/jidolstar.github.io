---
layout: post
title: "코틀린의 Companion Object (2) - 작성중"
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




## 아직 정리할 사항

2. 인스턴스에서는 companion object에 정의된 맴버에 접근할 수 없음. 클래스 내부에서만 가능 
3. companion object에 접근제한자를 쓰면? 본인 클래스에서 쓸 수 있는가? 자식에서는?? 
4. companion object는 인터페이스를 상속해서 만들 수 있다. 
5. 자바에서 compaion object를 사용하려면 Compaion 키워드를 붙여서 사용해야 한다.
  - 이를 회피하려면 @JvmStatic(메소드)이나 @JvmField(속성)을 변수나 메소드에 붙여야 한다. 
  - 이외에도 const 키워드를 쓰면 get/set이 만들어지지 않아서 속성에 바로 접근이 가능합니다.(프리미티브 타입과 String만 가능)


