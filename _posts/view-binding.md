---
title : "View Binding"
excerpt: "View binding 을 사용해보자!" 

categories :
  - Android 
tags : 
  - View Binding 
last_modified_at: 2021-02-03
---
## 목표

- 안드로이드에서 View 에 접근하는 방법들을 보고 차이를 이해한다.
- View binding 의 사용법을 익힌다.

## View Binding 이란?

View binding 은 안드로이드에서 view 에 접근하는 방법 중 하나다. View binding 을 제공하기 이전에 이미 Data binding 이란 기능을 지원하고 있었지만, 많은 개발자들이 data binding 의 모든 기능을 사용하지 않고, view 에 접근하기 위해서만 사용하는 것을 보고 안드로이드 스튜디오 3.6 버전에서 추가된 기능이다. View 에 접근하는 방법은 몇 가지가 있는데, 이를 도표로 정리하면 아래와 같다. 

![View binding img1](https://github.com/thkim9373/thkim9373.github.io/blob/master/assets/images/view-binding/view-binding1.png)

해당 도표에 있는 3가지 측정 기준은 

- Elegrance : 코드를 간결하게 쓸 수 있는가
- Compile time safety : 컴파일 타임에 안전한가
- Build speed imact : 빌드 속도가 빠른가

이다. 그래서 어떤걸 선택해야 한다는건지...? 위의 기준을 토대로 기능들을 비교해보자. 

### findViewById

- 뷰의 갯수만큼 코드를 추가해줘야 하므로 간결함이 떨어진다.
- 잘못된 id 를 넘겼을 때, null pointer exception 이 발생한다. (null safety 하지 않음)

### Butter Knife

- findViewById 에 비해 간결하긴 하지만, 여전히 보일러 플레이트 코드가 발생한다.
- 2020년 3월에 deprecated 됐다.

### Kotlin Synthethic

- 같은 id 를 가진 다른 뷰에 접근하는 실수가 발생할 여지가 있다.
- kotlin 만 사용 가능하다.
- 안드로이드 스튜디오 4.1 버전 기준으로 deprecated 됐다.

### View Binding

- View 를 직접 참조하는 클래스를 생성하여 유효하지 않은 view id 를 참조할 수 없기 때문에 **null safety 하다**. 다만, 같은 레이아웃 중 일부에만 존재하는 view 는 @Nullable 로 표시된다.
- 바인딩 클래스 view 필드 유형이 xml 파일에서 참조하는 view 와 동일하기 때문에 **class cast exception 으로부터 안전하다.**
- **컴파일 속도가 빠르다.**

### Data Binding

- View 를 직접 참조하는 클래스를 생성하여 유효하지 않은 view id 를 참조할 수 없기 때문에 **null safety 하다**. 다만, 같은 레이아웃 중 일부에만 존재하는 view 는 @Nullable 로 표시된다.
- 바인딩 클래스 view 필드 유형이 xml 파일에서 참조하는 view 와 동일하기 때문에 **class cast exception 으로부터 안전하다.**
- View binding 에 비해 컴파일 속도가 느리다.
- 동적 UI 컨텐츠를 구성할 수 있다.
- Two-way data binding 을 지원한다.

**결론적으로, View binding 과 Data binding 의 차이를 비교해보고 상황에 맞도록 사용하면 된다.**
