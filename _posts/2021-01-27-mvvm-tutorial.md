---
title : "MVVM 개요"
excerpt: "MVVM 에 대하여 알아보자"

categories :
  - Android 
tags : 
  - MVVM 
last_modified_at: 2021-01-26
---

# 목표

- MVVM 이란 무엇인지 학습한다.
- MVVM 구조를 간단한 예제 코드로 구현한다.

# MVVM 이란?

MVVM은 안드로이드 앱 아키텍쳐 패턴 중 하나로 Model View ViewModel 의 약자이다. 아키텍쳐 패턴은 코드를 기능별로 나누는 기준을 두는 것으로, 안드로이드에서 주로 쓰이는 구조는 MVVM 외에도 MVC 및 MVP 패턴이 있다. MVVM 구조는 UI 로직과 비즈니스 로직의 완전한 분리를 할 수 있도록 설계할 수 있는 구조이며, 따라서 

- 테스트를 용이하게 할 수 있으며,
- 유지보수 비용을 낮출 수 있고,
- Configuration 변화에도 데이터를 보장할 수 있다

는 장점이 있다. 단점으로는 

- 러닝 커브가 높고,
- 해당 구조를 만들기 위한 코드가 늘어나기 때문에 개발 시간이 늘어날 수 있다.

# 컴포넌트

MVVM 은 위에 기술한대로 View ViewModel Model 3가지로 컴포넌트로 나뉘며, 각각의 특징은 아래와 같다. 

## View

View 는 유저 액션에 따라 ViewModel 에 비즈니스 로직 실행을 요청하며, 그에 따른 결과를 반영하여 UI 를 업데이트하는 컴포넌트이다. Activity 및 Fragment 와 같이 유저에게 노출되는 컴포넌트들이 View 에 속한다. 

## ViewModel

View 에서 발생하는 액션에 따라 비즈니스 로직을 실행하여 Model 의 데이터를 업데이트하고, 이를 View 에서 UI 를 업데이트 할 수 있도록 데이터를 변경하는 컴포넌트이다. 또한 ViewModel 은 View 의 lifecycle 을 알고있고, 이에 따른 독자적인 lifecycle 을 가지며, 이를 이용하여 효과적인 비즈니스 로직을 짜는 것이 가능하다. 

## Model

로컬 데이터베이스나 네트워크 등의 데이터를 가지고 있는 친구이다. ViewModel 의 요청에 따라 데이터를 변경하는 역할을 수행한다. 

{% include image.html url="https://thkim9373.github.io/assets/images/mvvm_image1.png" description="MVVM 모식도([출처](https://docs.microsoft.com/en-us/xamarin/xamarin-forms/enterprise-application-patterns/mvvm))" %}
