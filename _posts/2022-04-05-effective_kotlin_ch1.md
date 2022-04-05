---
layout: single
title: Effective Kotlin 1장, 안정성
---

## 아이템 1: 가변성을 제한하라
variable 프로퍼티나 mutable 객체를 사용하면 상태를 갖게 된다.  요소가 상태를 갖는 경우, 해당 요소의 동작은 사용 방법뿐만 아니라 그 이력에도 의존하게 된다.

1. 상태를 갖는 부분들의 관계를 이해해야 하며 상태 변경이 많아지면 추적하는 것이 힘들어 디버깅이 어려워진다.
2. 현재 어떤 값을 갖고 있는지를 예측해야 하기에 추론하기 어려워진다.
3. 멀티스레드 환경에서 적절한 동기화 처리가 필요하다.
4. 모든 상태를 테스트해야 하므로 테스트 코드 작성이 어려워진다.
5. 상태 변경 시 다른 부분에 알려야 하는 경우가 생길 수 있다. (ex: 정렬)

### 코틀린에서 가변성을 제한하는 방법

* 읽기 전용 프로퍼티(val)
* 가변 컬렉션과 읽기 전용 컬렉션 구분하기
* 데이터 클래스의 copy

### 읽기 전용 프로퍼티(val)
val의 값은 mutable 객체를 담는 경우, getter의 재정의 등으로 값이 변경될 수 있기는 하지만(불변, immutable은 아니다), 프로퍼티 레퍼런스 자체를 변경할 수 없으므로 동기화 문제를 줄일 수 있다. 최대한 var 보다는 val를 사용하는 것이 좋다.

### 가변 컬렉션과 읽기 전용 컬렉션 구분하기
읽기 전용 프로퍼티와 마찬가지로 레퍼런스 자체는 변경할 수 없지만 불변의 의미는 아니다. immutable하지 않은 컬렉션을 외부적으로 immutable하게 보이도록 만들어 다양한 안정성을 가질 수 있다.

❗️읽기 전용 컬렉션을 가변 컬렉션으로 다운캐스팅 해서는 안 된다.

### 데이터 클래스의 copy
immutable 객체는 변경할 수 없다는 단점을 가지고 있지만 data class의 copy를 활용하여 변경된 새로운 객체를 전달할 수 있다.

### 다른 종류의 변경 가능 지점
변경할 수 있는 리스트를 만들어야 할 경우 mutable 컬렉션을 만드는 방법과 mutable 프로퍼티를 만드는 방법 중 mutable 프로퍼티를 활용하는 경우가 더 좋다.

```kotlin
val list1: MutableList<Int> = mutableListOf()
var list2: List<Int> = listOf() //Win
```

1. 멀티스레드 처리의 안정성이 좀 더 높다.
2. 객체의 변경을 추적하기 용이하다 (setter의 정의, Delegates.observalble 활용)

### 변경 가능 지점 노출하지 말기
mutable 객체를 외부에 노출하면 외부에서 돌발적인 수정이 있을 수 있으므로 위험하다. 이에 대한 대처 방법은 `방어적 복제(defensive copying)`, `immutable 타입으로 업캐스트` 하는 방법을 활용한다.

---
## 아이템 2: 변수의 스코프를 최소화하라

### 스코프를 좁게 만드는 것이 좋은 이유

* 프로그램을 추적하고 관리하기 쉬워진다.
* 다른 개발자에 의해 변수가 잘못 사용될 여지가 줄어든다.

### 정리

* 프로퍼티보다는 지역 변수를 사용하는 것이 좋다.
* 최대한 좁은 스코프를 갖게 변수를 사용한다.

---
## 아이템 3: 최대한 플랫폼 타입을 사용하지 말라

> 플랫폼 타입 (Platform type) : 다른 프로그래밍 언어에서 전달되어서 nullable 인지 아닌지 알 수 없는 타입.

```
User! //플랫폼 타입
User User //Not Null
User User? //Nullable User
```

### 플랫폼 타입은 Null 허용 여부를 알 수 없어서 최대한 사용하지 않는 것이 좋다.
만약 플랫폼 타입의 어떤 함수를 사용하는 사람이 Null에 대한 가능성을 고려하지 않고 구현 할 경우 문제 발생의 여지가 있는 코드가 될 수 있다. 추후에 수정되면 영향이 발생 할 수 있기 때문이다.

```java
//Java
public class JavaClass {
 public String getValue() {
  return null
 }
}
```

```kotlin
fun statedType() {
 val value = JavaClass().value //String! (String platform Type)
 println(value.length) //NPE
}
```

### 정리

1. 플랫폼 타입의 사용 자체를 최대한 피하는 것이 좋고, 사용하더라도 Null에 대한 가능성을 항상 생각해야 한다.
2. 자바 코드에는 @Nullable 이나 @NonNull Annotation을 활용하여 Null의 여부를 명시하도록 하는 것이 좋다.

---
## 아이템 4: inferred 타입으로 리턴하지 말라
코틀린의 타입 추론은 코틀린의 가장 큰 특징 중 하나이다. 하지만 타입 추론에는 몇 가지 위험한 부분이 존재한다.

### Inferred 타입은 정확히 오른쪽에 있는 피연산자에 맞게 설정된다.
생성된 인스턴스에 따라 타입이 추론되기 때문에 절대로 슈퍼클래스(or 인터페이스) 로 설정되지 않는다.

```kotlin
open class Animal
class Zebra: Animal()

fun main() {
 var animal = Zebra()
 animal = Animal() // Error: Type mismatch
}
```

일반적으로는 타입을 명시하는 것으로 해결 할 수 있다.

```kotlin
var animal: Animal = Zebra()
```

### 정리

1. 타입을 확실하게 지정해야 하는 경우에는 명시적으로 타입을 지정해줘야 한다.
2. 외부에 공개되는 라이브러리(or 모듈) 의 경우 반드시 타입을 지정하도록 하고, 특별한 이유나 확신 없이 지우지 않도록 한다.
3. inferred 타입은 프로젝트가 진행 되면서 너무 많은 제약사항이거나 예측하지 못한 결과를 낼 수 있다는 점을 기억해야 한다.

---
## 아이템 5: 예외를 활용해 코드에 제한을 걸어라
확실한 조건에서 동작해야 하는 코드가 있다면 예외를 활용해 제한(검사)을 걸어주는 것이 좋다.

* require 블록 : 아큐먼트를 제한할 수 있다. (IllegalArgumentException 발생)
* check 블록 : 상태와 관련된 동작을 제한할 수 있다. (IlleqgalStateException 발생)
* assert 블록 : 어떤 것이 true 인지 확인할 수 있다. assert 블록은 테스트 모드에서만 동작한다.
* return 또는 throw와 함께 활용하는 Elvis 연산자 (?:)

### 정리
제한을 걸어주면 다양한 장점이 발생한다.

* 제한을 훨씬 더 쉽게 확인할 수 있다.
* 문제가 있을 경우 함수가 예상하지 못한 동작을 하지 않고 예외를 던지게 되므로 애플리케이션을 더 안정적으로 지킬 수 있다. (예상하지 못한 동작은 예외를 throw하는 것보다 훨씬 위험할 수 있다.)
* 코드가 어느 정도 자체적으로 검사된다. 따라서 이와 관련된 단위 테스트를 줄일 수 있다.
* 스마트 캐스트 기능을 활용할 수 있게 되므로, 캐스트(타입 변환)를 적게 할 수 있다.

---
## 아이템 6: 사용자 정의 오류보다는 표준 오류를 사용하라
JSONParsingException과 같이 적절한 오류가 없어 사용자 정의 오류를 사용하는 경우도 있지만 가능하다면 최대한 표준 라이브러리의 오류를 사용하는 것이 좋다.

### 표준 라이브러리 오류는 많은 개발자가 알고 있으므로 재사용한다면 다른 사람들이 훨씬 이해하기 쉽다.

* IllegalArgumentException, IllegalStateException : require, check를 사용한 예외.
* IndexOutOfBoundsException : 인덱스 범위를 벗어났다는 예외.
* ConcurrentModificationException : 동시 수정(concurrent modification) 문제.
* UnsupportedOperationException : 사용하려는 메소드가 현재 객체에서는 사용할 수 없을 때.
* NoSuchElementException : 사용하려고 했던 요소가 존재하지 않음.

---
## 아이템 7: 결과 부족이 발생할 경우 null과 Failure를 사용하라

### 예측할 수 있는 범위의 오류는 null과 Failure를 사용한다.

* 예외의 상황을 명시할 수 있어, 예외에 대한 처리를 놓치지 않을 수 있고 효율적이다.
* 전체 애플리케이션을 중지시키지 않는다.

### null 처리
안전 호출(safe call) 또는 Elvis 연산자 같은 다양한 Null-safety 기능을 활용한다.

```kotlin
val age = person?.age ?: -1

person?.let { ... }
```

### Failure 처리
Result와 같은 sealed class를 리턴하는 경우, when 표현식을 사용해서 처리할 수 있다.

```kotlin
val age = when(person) {
 is Success -> preson.age
 is Failure -> -1
}
```

## *추가적인 정보를 전달해야 한다면 Failure, 그렇지 않으면 null을 사용하는 것이 일반적이다.*

## 아이템 8: 적절하게 null을 처리하라
null은 값이 부족하다(lack of value)는 것을 나타내며 함수에 따라 여러 의미를 가질 수 있으나 최대한 명확한 의미를 가져야 한다.

### 방어적 프로그래밍
null인 경우에 대한 적절한 처리가 반드시 필요하다. (안전 호출, 스마트캐스팅, Elvis 연산자 등을 활용할 수 있다. )
사용자의 관점에서 가장 안전한 방법이지만 모든 상황을 안전하게 처리하는 것은 불가능하다. 안전 호출을 통해 문제되는 부분을 넘어갈 수 있겠지만, 어떻게 해당 부분에 null이 들어가게 되었으며 해당 부분에서 개발자가 원하는 처리가 되고 있지 않음을 알아차리기 힘들 수 있기 때문이다.

### 공격적 프로그래밍
어떤 부분이 null이 될 수 있다는 예상을 하지 못하고 있었다면 원하는 로직이 수행되지 않는 원인을 찾기 어려울 것이다. 선입견처럼 당연히 그럴 것이다 라고 인지할 수 있는 위험성이 있기에 이런 경우에는 오히려 개발자에게 throw, requireNotNull, checkNotNull 등을 활용하여 오류를 강제로 발생시켜 주는 것이 좋다.

*방어적 프로그래밍과 공격적 프로그래밍은 코드의 안전을 위해 모두 필요한 개념이며 적절하게 사용될 필요가 있다.*

### not-null assertion(!!)과 관련된 문제
!! 은 nullable 타입이지만 확실이 null이 아니다는 확신이 있을 때 사용할 수 있다. 하지만 지금 확실하다고 하여 영원히 null이 아니라는 것을 확신할 수는 없을 것이다.  
일반적으로 !! 연산자가 의미 있는 경우는 드물다. nullability가 제대로 표현되지 않는 라이브러리를 사용할 때만 사용해야 하고 그 외에는 lateinit 또는 Delegates.notNull을 사용하도록 한다.  
!! 연산자를 통해 엄격한 예외 처리를 강조하기 위한 용도로 쓰일 수 있겠지만 예외는 예상하지 못한 잘못된 부분을 알려 주기 위해 발생하는 것으로 명시적 오류 처리를 통해 훨씬 더 많은 정보를 제공하는 것이 좋다.

### 의미 없는 nullability 피하기
null의 존재가 반드시 필요한 경우가 아니라면 nullability 자체를 피하는 것이 좋다. 별다른 의미가 없는 null을 처리하기 위해 불필요한 예외 처리가 추가되어야 하기 때문이다.

## 아이템 9: use를 사용하여 리소스를 닫아라
코틀린/JVM에서 사용하는 자바 표준 라이브러리에는 더 이상 사용하지 않는 리소스를 close 메소드를 사용하여 명시적으로 닫아야 하는 경우가 많다.

* InputStream, OutputStream
* java.sql.Connection
* java.io.Reader
* java.new.Socket

이러한 리소스들은 Closeable 인터페이스를 구현하고 있다.
전통적으로 이러한 리소스는 try-finally 블록을 사용해서 처리하였으나 코드가 복잡하다. 따라서 `use` 함수를 활용하여 처리한다.

```kotlin
fun countCharactersInFile(path: String): Int {
 BufferedReader(FileReader(path)).use { lines ->
  return lines.sumBy { it.length }
 }
}
```

### 정리

* use를 사용하면 Closeable/AutoCloseable을 구현한 객체를 쉽고 안전하게 처리할 수 있다.
* 파일을 처리할 때는 파일을 한 줄씩 읽어 들이는 useLines를 사용하는 것이 좋다.

---
## 아이템 10: 단위 테스트를 만들어라

* 단위 테스트는 개발자가 만드록 있는 요소가 제대로 동작하는지를 빠르게 피드백 해주므로 개발하는 동안에 큰 도움이 된다.
* 테스트는 계속해서 축적되므로 회귀 테스트도 쉬우며 수동으로 테스트하기 어려운 것들도 확인할 수 있다.

### 단위 테스트에서 확인할 내용

* 일반적인 사용(happy path) : 요소가 사용될 일반적인 방법을 테스트한다.
* 일반적인 오류 케이스와 잠재적인 문제 : 제대로 동작하지 않을 거라고 예상되는 일반적인 부분과 과거에 문제가 발생했던 부분 등을 테스트한다.
* 엣지 케이스와 잘못된 argument : Int의 경우 `Int.MAX_VALUE` 를 사용하는 경우, nullable의 경우 null 값으로 채워진 객체를 사용하는 경우 등을 테스트한다.

---
