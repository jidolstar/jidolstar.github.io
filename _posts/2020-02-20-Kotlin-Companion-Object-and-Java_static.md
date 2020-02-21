---
layout: post
title: "코틀린의 Companion Object는 클래스 Static인가?"
date: 2020-02-20 20:00:00 +0900
categories: kotlin 코틀린
tags: [코틀린, kotlin, Comapanion object, static]
author: 지용호
image: assets/images/11.jpg
comments: true
featured: true
hidden: false
---

자바(Java) 개발 경험이 있는 사람들에게는 코틀린(Kotin)에 static이 없다는 것에 당황한다. 자바에서 static은 클래스의 변수나 메소드 앞에 static 키워드를 붙여서 사용한다. 보통 static은 클래스를 설계시 모든 인스턴스에서 공통적으로 사용해야하는 변수나 메소드에 붙힌다. static이 붙은 변수나 메소드는 클래스가 메모리에 적재될 때 자동으로 생성되므로 인스턴스 생성없이 클래스명 다음에 점(.)을 쓰면 바로 참조할 수 있다. static이 붙은 변수와 메소드를 각각 클래스 변수, 클래스 메소드라 부른다. 반면, static이 붙지 않은 클래스내의 변수와 메소드는 각각 인스턴스 변수, 인스턴스 메소드라 한다.

```java
class MyClass{
  static public String TEST = "test";  //클래스 변수
  static public method(int i):int{     //클래스 메소드 
    return i + 10
  }
}
```

## 코틀린의 Companion Object 사용 예제 

## 자바의 static과 차이점 

## 코틀린 Companion Object 활용 


## 정리하며 


## 

```kotlin
fun main(args:Array<String>){
  var basic:VeryBasic = VeryBasic()
}

```

