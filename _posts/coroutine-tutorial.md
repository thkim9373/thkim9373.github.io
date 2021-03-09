---
title : "코루틴 뿌시기 1편 - 베이직"
excerpt: "코루틴 가이드 중 베이직을 살펴보자"

categories :
  - Android 
tags : 
  - Coroutine  
last_modified_at: 2021-03-02
---

## 목표

- 코루틴의 기본적인 사용법을 코드를 보며 익힌다.

## 코루틴이란?

코루틴은 비동기 프로그래밍을 처리하는 방법 중 하나이다. 코루틴을 사용하면 작업의 foreground 및 background 전환을 쉽고 직관적으로 처리할 수 있으며, 작업의 실행, 지연 및 취소 또한 안드로이드 컴포넌트의 생명주기에 따라 처리해줄 수 있다. 해당 포스팅은 코루틴 [공식 가이드](https://kotlinlang.org/docs/coroutines-basics.html#your-first-coroutine)의 내용을 기본적인 수준에서 부터 번역 및 해석해보려 한다. (같이 공부합시다 ㄱㄱ!) 

## Hello, World!

```kotlin
import kotlinx.coroutines.*

fun main() {
    GlobalScope.launch {
        delay(1000L)
        println("World!")
    }
    println("Hello,")
    Thread.sleep(2000L)
}
```

위 예제는 main 함수에서 GlobalScope.launch { } 블럭을 사용하여 Hello, World! 를 출력하는 코드이다.
