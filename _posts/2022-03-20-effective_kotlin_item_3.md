---
layout: single
title: 3장, 최대한 플랫폼 타입을 사용하지 말라
---

[이펙티브 코틀린](https://search.naver.com/search.naver?where=nexearch&sm=tab_etc&mra=bktE&pkid=476&os=25673819&qvt=0&query=이펙티브%20코틀린%20기본정보) 을 읽고 정리한 내용입니다.


_플랫폼 타입 (Platform type) : 다른 프로그래밍 언어에서 전달되어서 nullable 인지 아닌지 알 수 없는 타입._

```kotlin
User! //플랫폼 타입 User
User //Not Null User
User? //Nullable User
```

## 플랫폼 타입은 Null 허용 여부를 알 수 없어서 최대한 사용하지 않는 것이 좋다.

외부에서 생성되어 nullable 여부를 알 수 없는 타입을 플랫폼 타입으로 부른다. (ex: Java에서 생성된 메소드나 객체) 이는 절대 Null이 아니라고 보장 할 수 없으므로 Null에 대한 고려가 필요 할 수 있다.
```java
//Java
public class JavaClass {
  public String getValue() {
    return null
  }
}
```
```kotlin
//Kotlin
fun statedType() {
  val value = JavaClass().value //String! (String platform Type)
  println(value.length) //NPE
}
```
만약 플랫폼 타입의 어떤 함수를 사용하는 사람이 Null에 대한 가능성을 고려하지 않고 구현 할 경우 문제 발생의 여지가 있는 코드가 될 수 있다.

## 정리
* 플랫폼 타입의 사용 자체를 최대한 피하는 것이 좋고 사용하더라도 Null에 대한 가능성을 항상 생각해야 한다.
* 자바 코드에는 @Nullable or @NonNull Annotation을 붙여 명시하도록 하는 것이 자바 코드를 구현하는 입장에서도 유용하고, 사용하는 코틀린 개발자에게도 명확한 타입을 전달 할 수 있다.

```java
public class JavaClass {
  public @Nullable String getValue() {
    return null
  }
}
```
