---
title : "Recycler View 개요"
excerpt: "Recycler view 의 기본적인 사용법을 살펴보자."

categories :
  - Android 
tags : 
  - Recycler View 
last_modified_at: 2021-01-26
---

## 목표

- 리사이클러 뷰의 기본적인 사용법을 익힌다.

## 리사이클러 뷰(Recycler view) 란?

리사이클러 뷰는 리스트 형태의 UI 를 가진 뷰다. 정말정말 많은 곳에서 유용하게 사용되는 컴포넌트이기에 안드로이드 개발을 막 시작한 사람이라면 개념을 정립해 두는 것을 추천한다. 이름에 리사이클이 들어가는 이유는 리스트를 표현하는데 사용하는 뷰를 재사용하기 때문이다. 아래 그림을 보면서 이해해보자. 

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c32d24c9-5878-45ee-8e03-5c413e75cc7f/recycler-view1.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c32d24c9-5878-45ee-8e03-5c413e75cc7f/recycler-view1.png)

위의 그림처럼 화면에 최대 6개의 아이템이 보일 수 있는 리스트가 있다고 가정해보자. (걸치면 6대 가능) 이 때, 유저가 스크롤을 계속해서 화면에 더 이상 노출되지 않는 뷰가 생긴다면 리사이클러 뷰는 이를 재사용하여 새로운 데이터에 맞게 갱신하여 보여줌으로써 메모리를 효율적으로 관리한다. 위의 그림에서는 데이터의 수가 몇 개가 됐던 단 6개의 뷰로 리스트 UI 를 보여줄 수 있다는 것이다. 

## 구현

리사이클러 뷰를 커스텀할 수 있는 범위는 아주 많지만, 이번 포스팅에서는 아주 기본적인 기능만 구현해보자. (애초에 심화 내용 보려는 사람은 이거 안볼듯...?) 우리가 이번 시간에 만들어볼 리사이클러 뷰는 아래와 같다. 

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/bf4c5c3d-ab1f-40ca-9a14-492cc794acaf/recycler-view2.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/bf4c5c3d-ab1f-40ca-9a14-492cc794acaf/recycler-view2.png)

2개의 텍스트 뷰에 각각 텍스트를 표시할 수 있고, 세로 방향으로 스크롤이 가능한 리사이클러 뷰다. 위와 같은 리사이클러 뷰를 구현하는데 필수적으로 필요한 것은 

- Item - 리스트를 구성하는 데이터
- View Holder - 아이템 뷰를 담을 수 있는 홀더
- Adapter - 데이터 리스트를 리사이클러 뷰와 연결해주는 역할
- Layout Manager - 리사이클러 뷰의 모양과 동작을 관리

이며, 추가적으로 Item decoration 이라는 컴포넌트를 추가해서 아이템 사이에 구분선을 만든다거나, 아이템이 보여지는 시점 및 위치를 조정하여 다양한 UI/UX 를 제공할 수 있다. 위의 리사이클러 뷰에는 구분선을 만드는 아이템 데코레이션이 적용되어 있는 상태이다. 

### Item

먼저 리스트에 넣을 데이터를 만들자. id 는 유저에게 보여지는 데이터는 아니지만, 나중에 어탭터를 구현할 때 필요하니 넣어주자. 

```kotlin
data class Item(
    // 고유값
    val id: Int,
    val title: String = "",
    var description: String = "",
)
```

### View Holder

뷰 홀더는 리사이클을 하기 위한 아이템 뷰를 가지고 있는 홀더이다. 클래스를 만들기에 앞서 타이틀과 디스크립션을 표시해줄 레이아웃을 생성하자. 

```xml
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/container"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:padding="16dp">

    <TextView
        android:id="@+id/title"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:textSize="36sp"
        android:textStyle="bold"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        tools:text="Title" />

    <TextView
        android:id="@+id/description"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:textSize="24sp"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@id/title"
        tools:text="Description" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

그리고 해당 뷰를 가지고 있을 뷰 홀더를 만든다. 뷰 홀더를 만들 때는 View 인스턴스를 생성자에 넘겨줘야 한다. 해당 포스팅에서는 뷰 바인딩을 사용하도록 하자. (아직 View Binding 사용법을 모른다고? 그럼 그것부터 보고 오자. 다 피가되고 살이 된다. [포스팅 보러가기](https://thkim9373.github.io/android/view-binding/))

```kotlin
class MyViewHolder(
    private val binding: ItemBinding
) : RecyclerView.ViewHolder(binding.root) {
    fun setUi(item: Item) {
        binding.apply {
            title.text = item.title
            description.text = item.description
        }
    }
}
```

## Adapter

어탭터는 데이터와
