> Hello, Swift (X)
> Hell, RxSwift(O)

Swift에 반응형 (Rx) 가 추가된 RxSwift가 궁금해서 오셨겠죠.

본격적으로 구성요소를 보기전에 개괄적인 설명을 시작하겠습니다.
<br/><br/>


## RxSwift에 대하여
> Rx는 Observable인터페이스를 통해 표현된 계산의 일반적인 추상화로, 이를 통해 Observable 스트림에서 값 및 기타 이벤트를 브로드캐스트하고 구독할 수 있습니다.
> RxSwift는 Reactive Extensions 표준의 Swift 전용 구현입니다.

공식문서에 적힌 첫 문단의 설명입니다.

- Rx는 “Obervable” 이라는 걸 통해서 이벤트를 알리고 구독시킬 수 있다.
- Rx를 적용한 라이브러리가 RxSwift이다.

좀 더 풀어서 설명해보면 다음과 같습니다.

> RxSwift는 라이브러리 입니다. 비동기 처리와 이벤트 기반의 코딩을 가능하게하는 라이브러리죠. 이를 옵저버블 시퀀스와 함수형 오퍼레이터를 통해서 구현합니다. 

![](https://images.velog.io/images/kipsong/post/16b8c0e4-b97f-4963-bd7a-fa5f79d274fa/url.jpg)

읽기만 해도 현기증 나시죠.

앞으로 작성될 글들에 있는 예시를 동작시켜보면서, 동작하는 모습을 보시면 이해가 좀 더 쉽습니다.

추가로 Appstore 에 “RxMarbles” 라는 앱을 통해서 구현되는 Sequence 를 조작해보며 익히시는 것도 강력 추천드립니다.
<br/><br/>



## 비동기 프로그래밍에 대하여
Rx를 사용한다는 것은 비동기 프로그래밍을 한다는 것으로 이야기하면 100%는 아니지만 어느정도 맞는 이야기입니다.

그러면 언제 비동기가 필요할까요?

- 버튼을 탭했을 때, 특정 Interaction
- 특정 View를 클릭했을 때, 키보드가 사라지는 애니메이션
- 인터넷에서 큰 용량의 사진을 다운로드 하는 것
- 데이터를 디스크에 저장하는 것
- 오디오를 재생하는 것

위 동작들을 하는데, 만약 “동기” 로 처리된다면 어떨까요?

아마 앱이 중간에 멈춰있을 겁니다.

그래서 위와 같은 처리는 사용자의 경험을 위해서 그리고 앱의 완성도를 위해서 “비동기” 처리하는 게 적절합니다.

이미 iOS에서는 해결책으로 API를 제공하고 있습니다.
(GCD, DispatchQueue, Operator … 등등 생각나시나요?)

이 문제에 대한 해결책으로 적절한 해결방안입니다. 다만, 코드가 복잡하고 고려해야할 사항이 많습니다.

이를 편리하게 도와주는 도구가 Rx입니다. (가독성은 덤)
<br/><br/>



## Cocoa와 UIKit에서 제공하는 비동기 API들
1. NotificationCenter
	- 사용자가 장치의 방향을 변경하거나 소프트웨어 키보드가 화면에 표시되거나 숨겨지는 등의 관심 이벤트가 발생할 때마다 코드를 실행합니다.
2. Delegate Pattern 
	- 다른 개체를 대신하거나 다른 개체와 협력하여 작동하는 개체를 정의할 수 있습니다.
3. Grand Central Dispatch (GCD)
	- 작업 실행을 추상화하는 데 도움이 됩니다. 순차적으로, 동시에 또는 지정된 지연 후에 실행되도록 코드 블록을 예약할 수 있습니다.
4. Closure
5. Combine

위 API를 사용해서 Rx와 동일한 결과를 만들 수도 있습니다.
(코드가독성, 조작편의성과 같은 요소를 무시한다면요.)
<br/><br/>



### 동기 (synchronous) vs 비동기 (Asynchronous)

> 메인 쓰레드가 다른 쓰레드에게 일을 맡기고 그 일이 끝날 때까지 기다리는 것

동기의 정의입니다.

Array에 있는 요소에 접근해서 특정 로직을 실행시키는 것은 동기적으로 로직이 실행될 것입니다. 그리고 해당 Collection 은 변하지 않을 것입니다.

이 말을 풀어보자면,
```swift
var array = [1, 2, 3] 
array.forEach { print($0) }
```

`array` 라는 요소에 접근하는데, 1 다음에 3인지 2 인지 확인할 필요가 없죠. 이건 무조건 1다음에 2 가옵니다. 

`1 -> 2 -> 3 ` 이런식으로 지금 “동기” 적으로 움직이고 있습니다.

1. 1에 접근
2. print 출력
3. 2에 접근
4. print 출력
5. 3에 접근
6. print 출력

이 순서입니다. 2에 접근했을 때, 만약 비동기로 처리한다면, 2에 접근하고 print 출력을 `기다리지 않고` 3에 접근하는 겁니다.

물론 비동기로 접근해도 print 출력 로직이 가벼운 로직이기에 순서대로 출력이 될겁니다.

> 만약 print 출력이 아니라 네트워킹이라면 어떨까요?

아마 3번의 출력이 먼저되고 2번의 네트워킹이 처리될 것입니다. 

이것이 동기와 비동기의 차이입니다.

<br/><br/>



### 명령형 프로그래밍 vs 선언형 프로그래밍

제가 작성한 글인
[Swift 함수형 프로그래밍이 나타난 배경](https://velog.io/@kipsong/Swift-%ED%95%A8%EC%88%98%ED%98%95-%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D%EC%9D%B4-%EB%82%98%ED%83%80%EB%82%9C-%EB%B0%B0%EA%B2%BD)
를 참조해주세요.

(전반적인 개념과 배경만 이해하시면 됩니다. 자세한 설명은 시리즈 중간에 이어가겠습니다.)
<br/><br/>



## Rxswift의 근간

반응형 프로그래밍 (Reactive programming) 자체는 새로 발명된 개념이 아닙니다.

기존 부터 있었던 오래된 개념입니다.

개념이 처음 나왔을 시기에는 비동기처리보다 중요한 이슈가 있었던 것 뿐입니다.

Rx에 대한 개념 자체에 좀 더 공부하고 싶다면 아래링크를 참조하시면 됩니다.
 [http://reactivex.io](http://reactivex.io/) 

Rx는 3 가지의 근간이되는 개념이 있습니다.
1. 옵져버블 (Obervables)
2. 오퍼레이터 (Operators)
3.  스케쥴러 (Schedulers)
<br/><br/>



### Obervables
(공식문서 : [RxSwift/GettingStarted.md at main · ReactiveX/RxSwift · GitHub](https://github.com/ReactiveX/RxSwift/blob/main/Documentation/GettingStarted.md#observables-aka-sequences)
같이 보시면 이해가 더 잘됩니다.)

`Observable<Element>` 은 Rx 코드의 시작점이나 다름없습니다.

이벤트의 시퀀스(Sequence)를 생성할 수 있는 주체입니다. 
(= 이벤트를 만들어서 그것에 순서를 부여할 수 있다.)

옵저버블이 신문을 발행하는 신문사입니다.
그리고 이를 구독하는 구독자가 있겠죠.

옵저버블은 이벤트(신문)을 발행할 수 있는 주체입니다.

신문 대신에 값이 될 수도, 이벤트가 될 수도 있는 겁니다.

여기서 좀더 알아봐야할 부분이 있습니다.

바로 “이벤트” 입니다. 이벤트에도 종류가 있습닏.

1. `next event` : 이벤트 중에서 가장 기본인 이벤트입니다. 이벤트를 구독자에게 방출합니다.( 보통 이벤트를 전달한다고 하기보다 “옵저버블이 이벤트를 방출한다.” 라고 칭하고 “구독자에게 이벤트를 전달한다.” 라고 칭합니다.)
옵저버블은 next event를 여러번 방출할 수도 있습니다. (complete event나 error event 가 호출되기 전까지는 말이죠.)

2. `completed event` : 이 이벤트는 이벤트의 시퀀스를 성공적으로 종료했음을 방출하는 이벤트입니다. 옵저버블의 생명주기가 성공적으로 완수되었고 더이상 추가적인 이벤트는 방출할 수 없습니다.

3. `error event`  : 이 이벤트는 옵저버블을 에러와 함께 종료합니다. completed event와 마찬가지로 더이상 추가적인 이벤트는 방출할 수 없습니다.


아래 그림을 보겠습니다.
![](https://images.velog.io/images/kipsong/post/e22816d7-2af9-4402-bc80-cb5456eba2e4/%E1%84%8B%E1%85%A9%E1%86%B8%E1%84%8C%E1%85%A5%E1%84%87%E1%85%A5%E1%84%87%E1%85%B3%E1%86%AF%E1%84%83%E1%85%A9%E1%84%89%E1%85%B5%E1%86%A8.jpeg)


검정 사각형에 있는 것이 옵저버블을 구성한 겁니다.

한 줄에 보면 12, 8, 16 그리고 62 라는 Int값을 방출하도록 시퀀스를 구성했습니다.

그리고 옵저버블을 구독하는 옵저버 1, 2, 3 에게 이벤트를 전달합니다.

여기서 커스터마이징에 따라서 오퍼레이션이 파생됩니다.

1. 옵저버블의 이벤트를 어떤 방식으로 방출하도록 구현할 것인가.
2. 옵저버에게 이벤트를 어떤 방식으로 전달할 것인가.

에 대한 물음에 답하는 것이 오퍼레이션이라고 이해합시다.
<br/><br/>



### 유한한 옵저버블 시퀀스

몇몇 옵저버블 시퀀스는 0이나 1 그리고 더 많은 값들 특정 값을 방출하기도하고
성공적으로 시퀀스를 종료하거나 에러를 방출하기도 합니다.

인터넷에서 파일을 다운로드하는 상황을 가정해봅시다.

- 먼저 다운로드를 시작하고 다운로드되는 데이터를 관찰합니다.
- 반복적으로 데이터의 덩어리를 받습니다.
- 인터넷이 끊기면, 다운로드는 종료될 것이고 타임아웃 에러가 발생하겠죠.
- 반면에 정상적으로 모두 다운했다면 완료와함께 성공메세지를 받을 것입니다.


코드를 보겠습니다.
```swift
API.download(file: "http://www...")
   .subscribe(
     onNext: { data in
      // Append data to temporary file
     },
     onError: { error in
       // Display error to user
     },
     onCompleted: {
       // Use downloaded file
     }
   )
```

API는 옵저버블입니다. (옵저버블은 moda? 이벤트를 방출할 것을 정의하고 구독자에게 이벤트를 전달하는 주체이다.)

그 바로 아래 “subscribe” 라는 메소드와 함께 파라미터가 3 개 있습니다. (onNext, onError 그리고 onCompleted)

이 코드에서는 API가 어떻게 이벤트를 줄지는 모릅니다.

다만, 구독자가 어떻게 이벤트에 대해서 대응할 지를 구현하는 겁니다.
(자세한 설명은 다음 그리고 다다음 글에서 추가 설명되니 지금은 맥락에 집중해서 보세요.)

`onNext` 파라미터는 next event로 왔을 시, 처리할 로직을 클로저를 통해 작성합니다.

`onError` 파라미터는 에러 발생시 처리할 로직을 클로저를 통해 작성합니다.

`onCompleted` 파라미터는 완료 발생시 처리할 로직을 클로저를 통해 작성합니다.

직관적으로 이름만 봐도 느낌이 오죠.

인터넷에서 데이터를 다운로드하는 로직은 끝이 있는 로직입니다.

다운로드 시작이 시작이고 다운로드 종료 혹은 실패가 끝입니다.

그러면 이제 종료되지 않는 옵저버블 시퀀스를 보겠습니다.
<br/><br/>



### 무한한 옵저버블 시퀀스

파일 다운로드와 다르게 UI 이벤트는 무한한 시퀀스가 있습니다.

예를들면, 디바이스에서 “화면전환” 이벤트가 있겠죠.
(중간에 화면전환이 안되도록 컴플리트가 호출되면 안되겠죠? 로직상 막는게 아니라면요.)

- NotificationCenter로 부터 UIDeviceOrientationDidChange 알림을 옵저버에게 추가했습니다.
- 오리엔테이션 변화를 다루기 위해서 콜백함수가 필요하겠죠. UIDevice 객체로부터 현재 오리엔테이션을 확인할 것입니다. 그리고 여러번의 알림이 오더라도 `제일 마지막(가장 최근)의 오리엔테이션` 으로 화면방향을 결정할 것입니다.

옵저버블을 마블다이어그램으로 표현하면 다음과 같습니다.

![](https://images.velog.io/images/kipsong/post/344942a8-1e47-4da9-9f6c-d23b1929f5e4/%E1%84%8B%E1%85%A9%E1%84%85%E1%85%B5%E1%84%8B%E1%85%A6%E1%86%AB%E1%84%90%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%89%E1%85%A7%E1%86%AB%E1%84%86%E1%85%A1%E1%84%87%E1%85%B3%E1%86%AF%E1%84%83%E1%85%A1%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%A5%E1%84%80%E1%85%B3%E1%84%85%E1%85%A2%E1%86%B7.jpeg)

“Obervable<Orientation>” 시퀀스는 종료되선 안됩니다.

그러면 화면전환을 못하게 되겠죠.

그리고 짧은시간에 화면방향을 바꿨을 때, 액션을 그때그때 모두 처리해도 되지만, 그런 경우 가장 마지막에 들어온 이벤트만 처리하면 사용자가 원하는 화면으로 출력이 가능하죠. (이부분은 오퍼레이션을 설명하면서 상세히 설명하겠습니다.) 그렇게 처리하도록 도와주는 오퍼레이션들이 있습니다.

코드로 보면다음과 같습니다.
```swift
UIDevice.rx.orientation
  .subscribe(onNext: { current in
    switch current {
    case .landscape:
      // Re-arrange UI for landscape
    case .portrait:
      // Re-arrange UI for portrait
    }
  })
```

이전과 크게 다르진 않죠?

다만 onNext의 클로저에 파라미터를 받아소 그 파라미터를 enum으로 정의한 후 해당 case에 맞는 값으로 switch 분기문처리를 해주고 있을 뿐입니다. 

그리고 onCompleted 와 onError 이벤트에 대한 처리가 없죠.
(왜냐하면 해당 시퀀스는 종료되선 안되니까요. 종료되면 화면전환이 안됩니다~!)
<br/><br/>


## 오퍼레이터 (Operators)
`ObservableType` 과 `Observable` 클래스는 수많은 비동기관련 작업과 이벤트에 대한 조작이 가능한 메소드들이 있습니다.

이들을 잘 조합하면 복잡한 로직도 쉽게 구현이 가능합니다.

왜냐하면 이 메소드들은 잘게 쪼개고 잘 조합해서 사용하도록 구현되어 있습니다. 이 메소드들을 오퍼레이터 `Operator` 라고 칭합니다.

이러한 오퍼레이터는 비동기적으로 인풋을 넣고 아웃풋에는 사이드 이펙트 (Side-effect) 가 없도록 처리됩니다. ( cf. 함수형 프로그래밍 )

구체적인 코드로 보겠습니다.
```swift
UIDevice.rx.orientation
  .filter { $0 != .landscape }
  .map { _ in "Portrait is the best!" }
  .subscribe(onNext: { string in
    showAlert(text: string)
  })
```

바로 전에 봤던, 화면방향에 따른 옵저버입니다.

옵저버에서 보면 
 `.filter`  -> `map`  으로 이어지고 있죠. 이들은 Swift에서 고차함수로 불려지는 함수입니다.

이를 도식화 해보면 다음과 같습니다.

  ![](https://images.velog.io/images/kipsong/post/4d942a80-2875-4afe-ba4c-0916d0460940/%E1%84%8B%E1%85%A9%E1%84%91%E1%85%A5%E1%84%85%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%90%E1%85%A5%E1%84%83%E1%85%A9%E1%84%89%E1%85%B5%E1%86%A8%E1%84%92%E1%85%AA.jpeg)

옵저버블에서 filter 메소드로 전달되죠.

`Orientation` 에서 전달할 수 있는 것들은 `.portrait` 이나 `.landscape` 입니다.

그 중에서 filter 조건에 맞는 것들만 통과되겠죠.

`$0 != .landscape` 

.portrait 만 통과되겠죠.

그러면 map에 전달될 값은 `.portrait` 뿐입니다.

그리고 map 에서는 파라미터로 값을 받지만 사용하진 않습니다. 그 대신 String으로 리턴해주고 있죠.
`"Portrait is the best!"` 

여기까지 작성하면, 오퍼레이터의 동작은 끝납니다.

```swift
.subscribe(onNext: { string in
    showAlert(text: string)
  })
```

이렇게 조작(?) 혹은 편집(?) 된 데이터는 구독자에게 다음과 같은 onNext 이벤트가 전달 됩니다.(위코드처럼요.)

그러면 showAlert 메소드가 호출되면서 이벤트하나가 전달된겁니다.

`UIDevice.rx.orientation` 가 옵저버블이 되는 것이고

그걸 구독하고 있는 옵저버의 처리 로직입니다.

## 스케줄러 (Schedulers)

스케줄러는 Rx에서는 “dispatch Queue & operation Queue” 와 유사한 역할을 합니다.
(물론 Rx에서 스케줄러가 더 사용하기 쉽습니다.)

RxSwift에서 수많은 스케줄러들은 모두 해당되는데, 개발자가 직접 구성해야할 일은 거의 없을 겁니다.

아래 그림을 먼저 보겠습니다. (이미지 출처 : Raywenderlich, “Reactive programming with swift - 1. Hello, RxSwift!” 에서 Schedulers 중)

  
![](https://images.velog.io/images/kipsong/post/a95f5710-dbd7-4bf3-a280-bed7c5af6704/original.png)


왼쪽에 있는 것들이 개발자가 처리하고 싶은 로직들이고 RxSwift를 거쳐서 하나의 스케쥴러로 변환되고 있습니다. 

그 중에서 파란색 상자만 보면,

(1) fetch JSON 은 “Custom NSOperation Scheduler” 에 위치해 있습니다. 오퍼레이션 큐 기반 스케줄러 입니다.

(2) process JSON 은 “Background Concurrent Scheduler” 에 위치해 있습니다. GCD 큐 기반 스케줄러에 등록됩니다.

(3) display to UI는 “Main Thread Serial Scheduler” 에 위치해 있습니다. UI 작업이므로 메인스레드에서 처리하는 것이죠.

이에 대한 자세한 내용은 해당 챕터에서 추가 설명하겠습니다.
지금은 이런 것이 있다는 느낌으로만 봐두세요.

## 앱 아키텍처 (App architectrue)

`RxSwift + MVVM`  이라는 문구를 자주 보셨을 겁니다.
(Rx에 관심이 있으셨다면 상당히 자주 나오는 조합의 명사입니다.)

RxSwift를 활용한다고 해서 반드시 MVVM 패턴으로 앱 아키텍쳐를 구성해야하는 것은 아닙니다.
MVC와 MVP 로 해도 구현이 가능하죠.

다만 MVVM에 적용하기에 상당히 적합하다는 것 뿐입니다. 왜냐하면, `ViewModel(뷰모델)` 이 옵저버블 프로퍼티를 UI와 연동시키기 상당히 유용합니다. (글로 쓰니 이해가 잘 안되실 것 같아 그림을 좀 그렸습니다.)

![](https://images.velog.io/images/kipsong/post/e60ca360-4a49-4dd1-a636-9e4fd1e6b8a5/IMG_1CA49F001B29-1.jpeg)

해당 부분도 따로 목차로 구성되어 있으니 그 때, 예제와 함께 구현해보겠습니다.

## RxCocoa
RxSwift는 Rx 요구사항에 맞게 구성된 Swift 라이브러리 이죠.

그러므로 Cocoa 레벨 혹은 UIKit 레벨의 구체적인 class 에 대해서 연동되어있진 않을 겁니다.

RxCocoa는 RxSwift와 함께 사용하는 라이브러리 입니다. 왜 사용하느냐?
바로 위에서 말했던 Cocoa와 UIKit과 Rx를 연동시키기 위해서입니다.

코드를 먼저 보겠습니다.

```swift
toggleSwitch.rx.isOn 
	.subscribe(onNext: { isOn in
		print(isOn ? "It's On" : "It's Off")
	})
```

코드를 보시면, 
toogleSwitch는 `UISwitch` 객체 입니다. 그리고 `.rx.isOn` 이 붙어있죠? 이부분이 RxCocoa에서 재공하는 프로퍼티입니다. 이를 사용함으로서 Observable 시퀀스를 구독할 수 있게 되는 것이죠.

<br/><br/>



## 정리

여기까지 아주 개괄적으로 RxSwift에 대해서 살펴봤습니다.
각 단원에서 상세하게 다루겠습니다.

