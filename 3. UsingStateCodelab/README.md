# Using state in Jetpack Compose
https://developer.android.com/codelabs/jetpack-compose-state

<img width="40%" src="https://user-images.githubusercontent.com/46019755/142735200-a0ab3463-928e-4246-8882-01285185359c.gif"/>


## state
- 애플리케이션에서 변경될 수 있는 값
- event의 결과로 업데이트 
### A few examples of state in Android applications
1. 네트워크 연결이 안 될때 보여주는 스낵바
2. 블로그 포스트와 관련된 댓글
3. 사용자가 클릭할때 재생되는 버튼의 Ripple 애니메이션
4. 사용자가 이미지 위에 그릴 수 있는 스티커


## event
- 애플리케이션의 외부로부터 생기는 input의 변화
- For Example, 사용자가 버튼을 누르면 `OnClickListener` call back 


## UI가 업데이트 되는 과정

![image](https://user-images.githubusercontent.com/46019755/142725315-6a64daf3-b0f7-4832-a3c5-51224da4da3b.png)

Display State - 업데이트된 값이 보여지는 UI로 업데이트


## Unstructured state의 문제점
```Kotlin
class HelloCodelabActivity : AppCompatActivity() {
   private lateinit var binding: ActivityHelloCodelabBinding
   var name = "" // state
   override fun onCreate(savedInstanceState: Bundle?) {
       /* ... */
       binding.textInput.doAfterTextChanged {text ->
           name = text.toString() // state 업데이트
           updateHello() // UI 업데이트
       }
   }

   private fun updateHello() {
       binding.helloText.text = "Hello, $name" 
   }
}
```
1. Test하기 어렵다.
2. 이벤트가 많아질수록 변경해야할 state가 많아져서 업데이트할 state를 빠트릴 수 있다.
3. 변하는 state가 많아질수록 변경될 UI가 많아지면서 업데이트할 UI를 빠트릴 수 있다.
4. 코드가 복잡해진다.


## Unidirectional Data Flow
- Android Architecture Components with `ViewModel` and `LiveData`
- ViewModel에서 UI에서 사용하는 state와 state를 업데이트할 event를 정의한다.
- Unidirectional data flow is a design where state flows down and events flow up

![image](https://user-images.githubusercontent.com/46019755/142725718-6b950dcb-3085-41bb-8aa9-805b436605e3.png)

1. `events flow up` - Activity에서 event가 발생하면 ViewModel로 event를 올려보낸다.
2. `state flows down` - ViewModel에서 event를 처리하고 state를 변경해서 Activity로 내려보낸다.

```Kotlin
class HelloCodelabViewModel: ViewModel() {

   // LiveData holds state which is observed by the UI
   // (state flows down from ViewModel)
   private val _name = MutableLiveData("")
   val name: LiveData<String> = _name

   // onNameChanged is an event we're defining that the UI can invoke
   // (events flow up from UI)
   fun onNameChanged(newName: String) {
       _name.value = newName // Update State
   }
}

class HelloCodeLabActivityWithViewModel : AppCompatActivity() {
   private val helloViewModel by viewModels<HelloCodelabViewModel>()

   override fun onCreate(savedInstanceState: Bundle?) {
       /* ... */

       binding.textInput.doAfterTextChanged {
           helloViewModel.onNameChanged(it.toString()) // Event
       }

       helloViewModel.name.observe(this) { name ->
           binding.helloText.text = "Hello, $name" // to update the UI whenever the state changes. (Display State)
       }
   }
}
```
### ## Unidirectional Data Flow 장점
1. UI와 state를 분리함으로써 Activity와 ViewModel 테스트 용이
2. state는 ViewModel에서만 변경되므로 UI가 많아져도 state 변경 버그 감소
3. observable을 통해 state를 관찰하므로 state 변경이 즉시 UI 변경으로 반영


## todo package 구조
`Data.kt` – TodoItem을 보여줄 데이터 구조    
`TodoComponents.kt` – Todo 앱을 만들 재사용가능한 Composable     
`TodoActivity.kt` – Todo 앱을 만들기 위한 Compose 사용 Actvitiy       
`TodoViewModel.kt` – Todo 앱의 View와 연결되는 ViewModel    
`TodoScreen.kt` – state와 직접적으로 상호작용하는 Todo앱의 화면 구현    


## stateless composable
- state를 직접적으로 변경할 수 없는 composable
- UI와 관련된 코드 작성
### 장점
1. 테스트 용이
2. 버그 감소
3. 재사용성 증가


## state hoisting
- stateless composable을 만들기 위해 state를 위로 이동하는 패턴
### 필요한 파라미터
`value: T` - 현재 값
`onValueChange: (T) -> Unit` – 새로운 값 T로 value를 변경하는 이벤트


## observeAsState
- LiveData를 관찰하고 변경될떄마다 LiveData를 업데이트해서 State 객체로 반환


## stateful composable
- 시간이 지나면서 변경될 수 있는 state를 소유하는 composable
- UI와 관련없는 코드 작성


## Recomposition
- 새로운 입력으로 data가 변경되었을 때, compose tree를 업데이트하기 위해 동일한 composable을 다시 호출하는 프로세스
- must be `idempotent` : 같은 입력에대해 같은 결과 (항상)

## remember
- composition tree에 값을 저장할 수 있는 키워드
          
      remember(key) { calculation() }
1. key arguments - remember이 사용할 key
2. calculation - 새로운 값으로 계산하는 람다식


## state hoisting을 어디까지 올려야하는지에 대한 3가지 규칙
1. state를 사용하는 모든 composable의 가장 최소 공통 조상으로
2. 최소한 수정할 수 있는 최고 수준으로
3. 같은 event에 의해 2가지 state가 변하면 2가지 모두 hoising


## slot
- 호출자가 화면의 section을 설명할 수 있는 composable에 대한 매개변수
- 선언 방식 : `@Composable () -> Unit`
- 파라미터 수 감소 및 재사용성 증가




