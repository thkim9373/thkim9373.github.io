---
title : "MVVM 개요"
excerpt: "MVVM 에 대하여 알아보자"

categories :
  - Android 
tags : 
  - MVVM 
last_modified_at: 2021-01-31
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

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/7ff147ec-cf6f-4e65-8a07-6e1f418a9ce5/mvvm_image1.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/7ff147ec-cf6f-4e65-8a07-6e1f418a9ce5/mvvm_image1.png)

MVVM 모식도([출처](https://docs.microsoft.com/en-us/xamarin/xamarin-forms/enterprise-application-patterns/mvvm))

## 구현

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0bad37ea-64f4-4ec4-abc8-fb6711a2f12a/Screenshot_1612276524.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0bad37ea-64f4-4ec4-abc8-fb6711a2f12a/Screenshot_1612276524.png)

+, - 버튼을 만들어서 숫자를 카운트하는 간단한 앱을 MVVM 구조를 적용하기 전과 후로 구분하여 만들어 보고 차이를 살펴보자. 

### 레이아웃

```xml
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <TextView
        android:id="@+id/count"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textSize="48sp"
        android:textStyle="bold"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        tools:text="0" />

    <androidx.appcompat.widget.AppCompatImageView
        android:id="@+id/minus"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:background="?attr/selectableItemBackgroundBorderless"
        android:padding="12dp"
        app:layout_constraintBottom_toBottomOf="@id/count"
        app:layout_constraintEnd_toStartOf="@id/count"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="@id/count"
        app:srcCompat="@drawable/ic_baseline_minus_48" />

    <androidx.appcompat.widget.AppCompatImageView
        android:id="@+id/plus"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:background="?attr/selectableItemBackgroundBorderless"
        android:padding="12dp"
        app:layout_constraintBottom_toBottomOf="@id/count"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toEndOf="@id/count"
        app:layout_constraintTop_toTopOf="@id/count"
        app:srcCompat="@drawable/ic_baseline_plus_48" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

이미지 리소스가 필요한데, 안드로이드 스튜디오에서 지원하는 기능을 사용하면 편리하게 Material design icon 을 사용할 수 있다. ([꿀팁 보러가기](https://thkim9373.github.io/android/tips/#material-design-icon-vector-image-%EC%B6%94%EA%B0%80%ED%95%98%EA%B8%B0)) 

### View

```kotlin
class MainActivity : AppCompatActivity() {

    private lateinit var binding: ActivityMainBinding
    private var count: Int = 0

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)

        binding.count.text = count.toString()

        setListener()
    }

    private fun setListener() {
        binding.apply {
            minus.setOnClickListener {
                count.text = (--this@MainActivity.count).toString()
            }
            plus.setOnClickListener {
                count.text = (++this@MainActivity.count).toString()
            }
        }
    }
}
```

위의 코드는 MVVM 에서 view 에 해당하는 액티비티 코드다. 일단 view model 을 작성하기 전 이 상태로 앱을 실행시켜 보자. 겉으로 보기엔 이상이 없는 것 처럼 보이지만, 카운팅 하다가 기기를 가로 또는 세로로 회전하여 orientation 을 변경하면 숫자가 유지되지 않고, 0으로 초기화 된다. 이는 위에 언급한대로 configuration 이 변하게 되면 activity 를 재생성하기 때문에 발생하는 문제이다. 이제 view model 을 생성하여 데이터를 보존하고 비즈니스 로직도 옮겨보도록 하자. 

```kotlin
class MainActivity : AppCompatActivity() {

    private lateinit var binding: ActivityMainBinding
    private lateinit var viewModel: MainViewModel

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)

        viewModel = ViewModelProvider(this).get(MainViewModel::class.java)

        viewModel.countLiveData.observe(
            this,
            Observer {
                binding.count.text = it.toString()
            }
        )

        binding.apply {
            plus.setOnClickListener {
                viewModel.plus()
            }
            minus.setOnClickListener {
                viewModel.minus()
            }
        }
    }
}
```

## View model code

```kotlin
class MainViewModel : ViewModel() {

    private val _countLiveData = MutableLiveData(0)
    val countLiveData get() = _countLiveData

    fun plus() {
        _countLiveData.value = (_countLiveData.value ?: 0) + 1
    }

    fun minus() {
        _countLiveData.value = (_countLiveData.value ?: 0) - 1
    }
}
```
