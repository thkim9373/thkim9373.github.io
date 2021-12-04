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

- MVVM 의 개념을 이해한다.
- View model 과 Live data 가 사용된 구조를 예제 코드로 구현한다.

# MVVM 이란?

MVVM은 안드로이드 앱 아키텍쳐 패턴 중 하나로 **M**odel, **V**iew, **V**iew **M**odel 의 약자입니다. 아키텍쳐 패턴이란 코드를 기능별로 나누는 기준을 두는 것으로, 안드로이드에서 주로 쓰이는 구조는 MVVM 외에도 MVC 및 MVP 패턴이 있어요. 그 중 MVVM 은 UI 로직, 비즈니스 로직 및 데이터 레이어의 완전한 분리를 할 수 있도록 설계 가능한 구조이며, Room, Corountine 등의 요소들을 사용하는 프로젝트에서 엄청난 시너지를 낼 수 있습니다. 

요약하자면 MVVM 은 아래의 장점을 가져요. 

- UI 로직, 비즈니스 로직 및 데이터 레이어의 완전한 분리를 통해 유지보수 비용을 낮출 수 있습니다.
- 뷰 모델에 데이터를 두는 경우 액티비티와 다른 라이프사이클을 가지기 때문에 Configuration 변화(화면 방향 전환, 언어 설정 변경 등)에도 데이터를 유지할 수 있습니다.

단점으로는

- 제대로 설계하기 위해서는 높은 숙련도가 필요합니다. (러닝 커브가 높다!)
- 코드의 양이 늘기 때문에 개발 시간이 늘어날 수 있다. 라고 하는 사람들도 있지만 유지 보수 비용을 줄인하는 장점이 이를 쌈싸먹고도 남는 것 같습니다.

# 컴포넌트

MVVM 은 위에 기술한대로 View, View model, Model 의 컴포넌트로 나뉘며, 각각의 특징은 아래와 같습니다. 

## View(뷰)

뷰는 UI 를 노출하고 유저 액션(클릭 이벤트 등)에 따라 뷰 모델에 비즈니스 로직 실행을 요청하며, 그 결과를 반영하여 UI 를 갱신하는 컴포넌트입니다. 액티비티, 프래그먼트 등이 뷰에 속합니다. 

## ViewModel(뷰 모델)

뷰 모델은 뷰와 모델 사이의 매개체 역할을 하는 컴포넌트입니다. 조금 더 구체적으로 말하면, 모델에서 제공받은 데이터를 Live data, Flow 등을 사용하여 뷰에서 관찰 가능한 형태의 데이터로 제공하며, 뷰에서 요청하는 비즈니스 로직을 실행하여 모델의 데이터를 업데이트하는 역할을 수행해요. 뷰 모델은 뷰의 라이프사이클에 종속적인 라이프사이클을 가지기 때문에 화면 방향 변경, 언어 변경 등의 설정 변화에서도 데이터를 유지할 수 있습니다. 

## Model(모델)

모델은 데이터 레이어 컴포넌트입니다. 리포지토리, 데이터 소스(room, retrofit, preference 등)가 모델에 속합니다. 뷰 모델에 데이터를 제공하며, 요청에 따라 데이터 CRUD 를 수행합니다.

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
