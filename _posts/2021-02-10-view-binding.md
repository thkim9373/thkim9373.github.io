---
title : "View Binding"
excerpt: "View binding 을 사용해보자!" 

categories :
  - Android 
tags : 
  - View Binding 
last_modified_at: 2021-02-10
---
## 목표

- 안드로이드에서 View 에 접근하는 방법들을 보고 차이를 이해한다.
- View binding 의 사용법을 익힌다.

## View Binding 이란?

View binding 은 안드로이드에서 view 에 접근하는 방법 중 하나다. View binding 을 제공하기 이전에 이미 Data binding 이란 기능을 지원하고 있었지만, 많은 개발자들이 data binding 의 모든 기능을 사용하지 않고, view 에 접근하기 위해서만 사용하는 것을 보고 안드로이드 스튜디오 3.6 버전에서 추가된 기능이다. View 에 접근하는 방법은 몇 가지가 있는데, 이를 도표로 정리하면 아래와 같다. 

![View binding img1](https://thkim9373.github.io/assets/images/view-binding/view-binding3.png)

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

**결론적으로, View binding 과 Data binding 의 차이를 비교해보고 상황에 맞도록 사용하면 된다. ~~답정너~~**

### View Binding vs Data Binding

간단하게 둘의 관계를 정리하면 Data binding = View binding + Binding data to views 이다. 

- View Binding : 빌드 속도 우세, 앱 용량 크기가 상대적으로 작음, xml 에 layout 태그를 감싸줄 필요가 없어서 사용이 용이함, 데이터 바인딩 기능 사용 못함
- Data Binding : 빌드 속도 열세, 앱 크기가 View binding 을 이용하는 것 보다 커짐, xml 의 최상단에 layout 태그로 감싸줘야함, 데이터 바인딩 기능을 이용한 동적 UI 설계 가능

## 사용법

### Setup

앱의 build.gradle 에서 View binding 을 사용할 수 있도록 설정해주자. 

```groovy
// 안드로이드 스튜디오 4.0 이상 
android {
    buildFeatures {
        viewBinding = true
    }
}
```

안드로이드 스튜디오 4.0 부터 param 의 위치가 buildFeatures 로 변경됐다. 그보다 미만 버전에서는 아래와 같이 설정하면 된다. 

```groovy
// 안드로이드 스튜디오 4.0 미만 
android {
    viewBinding {
        enabled = true
    }
}
```

위와 같이 설정을 완료하면 레이아웃 파일에 해당하는 바인딩 클래스가 생성된다. 바인딩 클래스의 네이밍 규칙은 xml 의 이름을 upper camel case 로 변환한 뒤 Binding 이 붙는 형태이다. 

> main_activity.xml → MainActivityBinding 
first_fragment.xml → FirstFragmentBinding 
item_content.xml → ItemContentBinding

### Activity

```kotlin
class MainActivity : AppCompatActivity() {

    private lateinit var binding: ActivityMainBinding

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        // 바인딩 클래스의 inflate 메서드를 호출하여 인스턴스 생성
        binding = ActivityMainBinding.inflate(layoutInflater)
        // 바인딩 인스턴스에서 getRoot 메서드를 호출하여 해당 뷰를 스트린에 띄움 
        setContentView(binding.root)
				
        // 바인딩 클래스에 id 가 textView 인 Text view 에 다음과 같이 접근할 수 있다. 
        binding.textView.text = "Hello View Binding!"
    }
}
```

### Fragment

```kotlin
class FirstFragment : Fragment() {

    private var _binding: FragmentFirstBinding? = null

    private val binding get() = _binding!!

    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        _binding = FragmentFirstBinding.inflate(inflater, container, false)
        return binding.root
    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)

        binding.textView.text = "Hello View Binding!"
    }

    override fun onDestroyView() {
        super.onDestroyView()
        _binding = null
    }
}

```

프래그먼트에서 뷰 바인딩을 사용할 때, Fragment 는 View 보다 오래 존재하기 때문에 onDestroyView() 에서 바인딩 클래스 인스턴스의 레퍼런스를 null 로 정리해줘야 합니다. (안해주면 메모리릭 남. 님 앱 펑! ㅇㅇ) 그렇기 때문에 프래그먼트에서 뷰 바인딩을 사용할 때는 위와 같이 추가적인 코드가 들어가게 됩니다. 

## 결론

뷰에 접근하기 위해서는 View binding 과 Data binding 의 차이를 숙지하고, 상황에 맞는 방법을 사용하는 것이 좋으며, 속도와 간결함을 중시한다면 View binding 을 고려해보는 것이 좋습니다.
