---
title : "코루틴 뿌시기 1편 - 베이직"
excerpt: "코루틴 가이드 중 베이직을 살펴보자"

categories :
  - Kotlin 
  - Coroutine  
tags : 
  - Coroutine  
last_modified_at: 2021-03-02
---
## 목표

- 코루틴의 기본적인 사용법을 코드를 보며 익힌다.

## 코루틴이란?

코루틴은 비동기 프로그래밍을 처리하는 방법 중 하나이다. 코루틴을 사용하면 작업의 foreground 및 background 전환을 쉽고 직관적으로 처리할 수 있으며, 작업의 실행, 지연 및 취소 또한 안드로이드 컴포넌트의 생명주기에 따라 처리해줄 수 있다. 해당 포스팅은 코루틴 [공식 가이드](https://kotlinlang.org/docs/coroutines-basics.html)의 내용을 기본적인 수준에서 부터 번역 및 해석해보려 한다. (같이 공부합시다 ㄱㄱ!) 

## Hello, World!

```kotlin
fun main() {
    GlobalScope.launch { // 백그라운드에서 새로운 코루틴을 실행한다. 이때, 메인 스레드는 계속 동작한다.
        delay(1000L) // non-blocking 하게 1초 동안 지연시킨다. (기본 시간단위는 ms 임)
        println("World!") // 1초 지연 후 출력
    }
    println("Hello,") // 코루틴이 딜레이 되는 동안 메인 스레드를 통해 해당 문구가 출력됨
    Thread.sleep(2000L) // 메인 스레드를 2초간 멈춰서 JVM 이 종료되는 것을 막는다. 
}
```

위 예제는 main 함수에서 GlobalScope.launch { } 블럭을 생성한 후, Hello, World! 를 출력하는 코드이다. 위의 코드가 실행되는 순서는 다음과 같다. 

1. main() 함수가 메인 스레드에서 실행 시작 
2. GlobalScope 코루틴 블럭이 생성되어 백그라운드에서 실행 시작 
3. 코루틴 블럭이 1초간 지연됨 
4. 메인 스레드에서 "Hello," 문구 출력
5. 메인 스레드가 2초간 지연됨 
