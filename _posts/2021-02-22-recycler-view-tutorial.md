---
title : "Recycler View 개요"
excerpt: "Recycler view 의 기본적인 사용법을 살펴보자."

categories :
  - Android 
tags : 
  - Recycler View 
last_modified_at: 2021-02-22
---

## 목표

- 리사이클러 뷰란 무엇인지 알아본다.
- 간단한 리사이클러 뷰를 샘플 코드를 보며 구현해본다.
- 리사이클러 뷰의 이벤트 처리법을 익힌다.

## 리사이클러 뷰(Recycler view) 란?

리사이클러 뷰는 리스트 형태의 UI 를 가진 뷰다. 정말정말 많은 곳에서 유용하게 사용되는 컴포넌트이기에 안드로이드 개발을 막 시작한 사람이라면 개념을 확실히 익혀두는 것을 추천한다. 이름에 리사이클이 들어가는 이유는 리스트를 표현하는데 사용하는 뷰를 재사용하기 때문이다. 아래 그림을 보면서 이해해보자. 

<p align="center">
  <img width="50%" height="50%" src="https://thkim9373.github.io/assets/images/recycler-view/recycler-view3.png">
</p>

위의 그림처럼 화면에 최대 6개의 아이템이 보일 수 있는 리스트가 있다고 가정해보자. (걸치면 6대 가능) 이 때, 유저가 스크롤을 계속해서 화면에 더 이상 노출되지 않는 뷰가 생긴다면 리사이클러 뷰는 이를 재사용하여 새로운 데이터에 맞게 갱신하여 보여줌으로써 메모리를 효율적으로 관리한다. 위의 그림에서는 데이터의 수가 몇 개가 됐던 단 6개의 뷰로 리스트 UI 를 보여줄 수 있다는 것이다. (눈속임 ㅇㅇ) 

## 구현

리사이클러 뷰를 커스텀할 수 있는 범위는 아주 많지만, 이번 포스팅에서는 아주 기본적인 기능만 구현해보자. (애초에 심화 내용 보려는 사람은 이거 안볼듯...?) 우리가 이번 시간에 만들어볼 리사이클러 뷰는 아래와 같다. (버튼은 따로 쓸 곳이 있다.) 

<p align="center">
  <img width="50%" height="50%" src="https://thkim9373.github.io/assets/images/recycler-view/recycler-view4.png">
</p>

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
    fun setItem(item: Item) {
        binding.apply {
            title.text = item.title
            description.text = item.description
        }
    }
}
```

### Adapter

어탭터는 데이터와 리사이클러 뷰를 연결해주는 역할을 한다. 너무 대충 말한 것 같은데... 구현할 수 있는 핵심적인 기능을 조금 더 자세히 말하면 아래와 같다. 

- 데이터에 따라 뷰 타입을 지정하여 상황에 맞는 뷰 홀더를 생성할 수 있다. (해당 포스팅에선 구현 ㄴㄴ)
- 뷰 홀더가 바인드 될 때, 데이터에 맞게 UI 를 갱신할 수 있다.
- 데이터가 변경되었을 때, 리사이클러 뷰를 갱신할 수 있다.

어탭터에도 종류가 몇 있는데, 그 중 오늘 사용할 어탭터는 [List Adapter](https://developer.android.com/reference/androidx/recyclerview/widget/ListAdapter) 이다. 리스트 어댑터는 데이터가 변했을 때, 리사이클러 뷰의 갱신을 아주 편리하게 할 수 있도록 구현되어 있는 친구다. 전통적으로 쓰이던 RecyclerView.Adapter 를 사용할 시절에는 데이터가 변했을 때, 추가, 제거 및 갱신을 일일히 알려줘야 했었지만, 지금은 시절이 좋아져서 Diff.ItemCallback 의 메서드만 잘 정의해놓으면 알아서 리사이클러 뷰를 갱신해준다. (시절 참 좋아졌네...) 

```kotlin
class MyAdapter : ListAdapter<Item, MyViewHolder>(
    object : DiffUtil.ItemCallback<Item>() {
				// id 가 같으면 같은 아이템이다. 
        override fun areItemsTheSame(oldItem: Item, newItem: Item): Boolean =
            oldItem.id == newItem.id
				
				// title 과 description 이 모두 같으면 내용도 같다. 
        override fun areContentsTheSame(oldItem: Item, newItem: Item): Boolean =
            oldItem.title == newItem.title &&
                    oldItem.description == newItem.description
    }
) {
    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): MyViewHolder =
        MyViewHolder(
						// 이렇게 뷰 바인딩 객체를 생성하여 넘겨준다. 
            ItemBinding.inflate(
                LayoutInflater.from(parent.context),
                parent,
                false
            )
        )

    override fun onBindViewHolder(holder: MyViewHolder, position: Int) {
				// 뷰 홀더가 바인딩 될 때마다 해당 포지션에 맞는 아이템을 넘겨줘서 뷰를 갱신한다. 
        holder.setItem(getItem(position))
    }
}
```

위에 언급한 것 처럼 리스트 어댑터를 쓸 때는 데이터의 변화를 일일히 알려주지 않아도 되고, 변경하고 싶은 땐 submitList() 라는 메서드로 갱신하고 싶은 데이터 리스트를 가져다 주기만 하면 된다. 새로운 리스트를 받으면 원래 가지고 있던 리스트의 아이템들과 새로운 아이템들을 구현해놓은 DiffUtil.ItemCallback 에 정의한대로 비교하여 결과에 따라 추가, 변경 및 제거 등을 수행해준다. 심지어 아이템의 비교는 백그라운드에서 처리해주기 때문에 성능까지도 보장해주는 아주 듬직한 친구다. 

DiffUtil.ItemCallback 를 구현할 때, 꼭 구현해야 하는 메서드는 위의 areItemsTheSame 과 areContentsTheSame 이다. areItemsTheSame 은 같은 아이템인지 여부를 리턴해주면 되며, 위의 코드에서는 id 를 비교하여 같으면 true, 다르면 false 를 리턴하도록 구현했다. 해당 메서드의 리턴 값이 false 이면 그 아이템이 표시되고 있는 뷰 홀더가 갱신된다. 만약 true 를 반환하게 되면 다른 메서드인 areContentsTheSame 이 호출되는데, 여기서는 두 아이템의 내용이 같은지 여부를 리턴해주면 된다. 해당 메서드는 리턴 값이 true 이면 뷰 홀더를 갱신하지 않게 되고, false 이면 해당 아이템이 표시되고 있는 포지션의 뷰 홀더가 갱신된다. 

DiffUtil.ItemCallback 의 메서드들을 작성할 때 유의해야 할 점이 있는데, areItemsTheSame 의 리턴 값으로 객체를 비교한 값으로 내보내지 않는 것이 좋다. (해시 코드 등 사용 ㄴㄴ) 객체가 같은지 여부를 리턴하게 되면 뒤의 areContentsTheSame 또한 같은 객체를 비교하게 되므로 같은 항목을 검사하면 모두 true 가 나오기 때문이다. 따라서 리스트 어탭터에 submitList() 를 할 때는 되도록 ***비교할 수 있는 고유값을 가진 다른 객체로 이루어진 리스트***를 넣는 것이 바람직한 사용 방법인 것 같다. (적다보니 submitList() 가 먼저 나와버렸는데 뒤의 액티비티 코드에서 사용한다.)

### Activity

리스트를 이것 저것 바꿀 수 있는 버튼이 2개 그리고 오늘의 주인공인 리사이클러 뷰가 하나 있는 레이아웃을 만들자. 

```xml
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <Button
        android:id="@+id/list1Button"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="submit list1"
        app:layout_constraintEnd_toStartOf="@+id/list2Button"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <Button
        android:id="@+id/list2Button"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="submit list2"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toEndOf="@id/list1Button"
        app:layout_constraintTop_toTopOf="parent" />

    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/recyclerView"
        android:layout_width="0dp"
        android:layout_height="0dp"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@id/list1Button" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

다음은 코드! 

```kotlin
class MainActivity : AppCompatActivity() {

    private lateinit var binding: ActivityMainBinding
		
		// 리사이클러 뷰에 할당할 어댑터 
    private val myAdapter = MyAdapter()

		// 리사이클러 뷰에 할당할 리스트 1
    private val list1 = listOf(
        Item(1, "Item 1", "Description 1",),
        Item(2, "Item 2", "Description 2",),
        Item(3, "Item 3", "Description 3",),
        Item(4, "Item 4", "Description 4",),
        Item(5, "Item 5", "Description 5",),
        Item(6, "Item 6", "Description 6",),
        Item(7, "Item 7", "Description 7",),
        Item(8, "Item 8", "Description 8",),
        Item(9, "Item 9", "Description 9",),
        Item(10, "Item 10", "Description 10",),
    )
		// 22
    private val list2 = listOf(
        Item(0, "Odd List", "First Item!",),
        Item(1, "Item 1", "Description 1",),
        Item(3, "Item 3", "Description 3",),
        Item(5, "Item 5", "Description 5",),
        Item(7, "Item 7", "Description 7",),
        Item(9, "Item 9", "Description 9",),
        Item(10, "Item 10", "Last Item!",),
    )

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)

        binding.apply {
            // 리사이클러 뷰의 속성 정의 ㄱㄱ
            recyclerView.apply {

                // 해당 리스트를 세로 스크롤이 가능한 리스트로 사용할 수 있도록 해준다.
                layoutManager = LinearLayoutManager(this@MainActivity)

                // 아이템 사이의 구분선을 만들어 준다.
                addItemDecoration(
                    DividerItemDecoration(
                        this@MainActivity,
                        DividerItemDecoration.VERTICAL
                    )
                )

                // 어댑터를 할당한다.
                adapter = this@MainActivity.myAdapter.apply {
                    submitList(
                        list1
                    )
                }
            }

            list1Button.setOnClickListener {
                // 리스트를 갖다주면 아까 구현한 어댑터에서 리스트끼리 비교하여 리사이클러 뷰를 갱신해준다.
                myAdapter.submitList(list1)
            }
            list2Button.setOnClickListener {
                // 얘도 ㅇㅇ
                myAdapter.submitList(list2)
            }
        }
    }
}
```

주석에 다 설명을 다 써놔서 적을 내용이 별로 없다. (코드가 간단하기도 하구) 아 하나 있구나! 

리스트 어댑터를 사용할 때 리사이클러 뷰에 노출할 데이터를 변경하고 싶으면 어댑터 코드에서 살짝 등장한 submitList() 메서드를 사용하면 된다. 기본적인 애니메이션도 구현이 되어있어서 생각보다 꽤나 그럴싸하게 리사이클러 뷰가 갱신되는걸 볼 수 있다. 

여기까지 기본적인 리사이클러 뷰 구현이 끝났다. 이제 리사이클러 뷰에 이벤트(클릭 등)를 줬을 때, 처리하는 방법을 알아보자. 

## 이벤트 처리

리사이클러 뷰에는 리스트 뷰의 onItemClickListener 같이 이벤트를 처리할 수 있도록 제공되는 인터페이스가 없다. 따라서 직접 클릭, 롱클릭 등의 이벤트를 처리할 수 있도록 구현해줘야 한다. 한 번에 다 하지 왜 2번 보게 만드냐고 할 수 있지만 2번 보면 2배 더 좋은 내용이니 복습의 느낌으로 한 번 더 보도록 하자. 

### Adapter

이벤트를 전달할 리스너를 하나 만들고, 액티비티에서 어댑터를 생성할 때 받을 수 있도록 하자. 마지막으로 해당 리스너를 뷰 홀더가 바인딩 될 때 건네주자. 

```kotlin
class MyAdapter(
    // 어탭터를 생성할 때 리스너도 받아오자  
    private val listener: MyAdapterListener
) : ListAdapter<Item, MyViewHolder>(
    ...
) {
		// 이벤트를 전달해 줄 리스너를 정의한다. 
    interface MyAdapterListener {
        fun onItemClick(position: Int)
        fun onItemLongClick(position: Int)
    }

    ...

    override fun onBindViewHolder(holder: MyViewHolder, position: Int) {
        holder.apply {
            setItem(getItem(position))
            // 여기서 홀더에 리스너를 전달해주자 
            setListener(listener)
        }
    }
}
```

### View Holder

눌렀을 때 아무 반응이 없으면 재미가 덜하니 레이아웃 단에서 간단한 처리를 하나 하자. 

```xml
<androidx.constraintlayout.widget.ConstraintLayout 
		...
    android:background="?attr/selectableItemBackground"\>

    <TextView ... />

    <TextView ... />

</androidx.constraintlayout.widget.ConstraintLayout>
```

해당 속성을 추가하면 조금 더 역동적이게 된다. ([간단하게 리플 이펙트 사용하는 법](https://thkim9373.github.io/android/tips/#%EA%B0%84%EB%8B%A8%ED%95%98%EA%B2%8C-ripple-%EC%9D%B4%ED%8E%99%ED%8A%B8-%EC%82%AC%EC%9A%A9%ED%95%98%EA%B8%B0)) 

다음은 코드에 리스너를  수정하자. 

```kotlin
class MyViewHolder(
    private val binding: ItemBinding,
) : RecyclerView.ViewHolder(binding.root) {

    ...
		
		// 리스너를 할당하는 메서드
    fun setListener(listener: MyAdapter.MyAdapterListener) {
        binding.container.apply {
            setOnClickListener {
                if (adapterPosition != RecyclerView.NO_POSITION) {
                    listener.onItemClick(adapterPosition)
                }
            }
            setOnLongClickListener {
                if (adapterPosition != RecyclerView.NO_POSITION) {
                    listener.onItemLongClick(adapterPosition)
                }
                return@setOnLongClickListener true
            }
        }
    }
}
```

뷰 홀더는 리사이클러 뷰에서 본인의 위치를 adapterPosition 으로 가져올 수 있다. 뷰에 이벤트가 발생했을 때, 구현해 둔 리스너에 뷰 홀더의 포지션을 넘겨주도록 하자. 이 때 주의할 점은 반드시 RecyclerView.NO_POSITION 인지 체크한 후 사용해야 한다. RecyclerView.NO_POSITION 은 리사이클러 뷰가 갱신 중일 때 어댑터 포지션을 가져오면 리턴되는 값이다. 그러므로 해당 값이 아닐 때만 리스너를 트리거 시켜주도록 하자. 

### Activity

```kotlin
// 리스너를 추가하자 
class MainActivity : AppCompatActivity(), MyAdapter.MyAdapterListener {
		
		// 어댑터를 생성할 때, 리스너를 넘겨주자. 
		private val myAdapter = MyAdapter(this)

		...

		// 아이템을 클릭 및 롱클릭 하게되면 토스트를 보여주도록 구현해놓았다. 
    override fun onItemClick(position: Int) {
        Toast.makeText(this, "$position click!", Toast.LENGTH_SHORT).show()
    }

    override fun onItemLongClick(position: Int) {
        Toast.makeText(this, "$position long click!", Toast.LENGTH_SHORT).show()
    }
}
```

여기까지 리사이클러 뷰 이벤트를 처리하는 방법 중 하나를 알아봤다. 위의 방법 뿐만 아니라 여러 방법이 존재하겠지만, 해당 방법을 소개한 이유는 위의 방법이 뷰 홀더, 어탭터 및 액티비티가 가지는 본연의 역할을 지킬 수 있는 방법이라 생각하여 추천하기 때문이다. 

## 마치며

리사이클러 뷰의 기본적인 사용 방법과 이벤트를 처리하는 방법을 알아보는 시간을 가져봤다. 리사이클러 뷰는 안드로이드 개발을 하는데 있어서 필수적으로 익혀둬야 하는 컴포넌트라 기초부터 확실한 개념을 가지는 것을 추천한다. 아울러 해당 포스팅의 내용 외에도 봐야할 내용이 많으니 후에 다른 내용들도 포스팅 해야겠다. To be continued...
