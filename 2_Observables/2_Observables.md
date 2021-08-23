## 옵저버블 (Observables) 에 대하여
옵저버블 (Observable) 은 Rx의 핵심입니다.

이전 글에서 아마 옵저버블, 옵저버블 시퀀스 그리고 시퀀스 라는 용어들을 보셨을 겁니다.

위 단어들은  하나의 같은 의미를 가지고 있습니다.

간혹 스트림(Stream) 이라는 용어도 보실겁니다. 다른 반응형 프로그래밍에서는 그렇게 칭하기도 합니다.

스트림 또한 Rxswift에서 말하는 옵저버블과 같은 의미입니다.

> 옵저버블은 시퀀스이다.

옵저버블의 가장 핵심적인 기능은 “비동기 (Asynchronuse)” 입니다. 옵저버블은 이벤트를 일정한 기간동안 생성하고 방출합니다. 그 이벤트들은 값이되기도하고 특정 타입이 되기도하며 제스쳐나 탭(tap)이 되기도 합니다. 

마블 다이어그램으로 보면 다음과 같습니다.
(아래 다이어그램은 Int 값을 이벤트로 방출하는 예시입니다.)
![](https://images.velog.io/images/kipsong/post/c2d7c58d-fdd9-4847-806e-a13e5601f386/IMG_EC628D82F836-1.jpeg)

시퀀스를 읽는 방향은 왼쪽에서 오른쪽으로 순차적으로 읽습니다.

“1” 이 방출되고 “2”가 방출되며 마지막으로 “3”이 방출됩니다.

간단하죠?

## 옵저버블의 생명주기

위 마블 다이어그램에서 옵저버블은 3 개의 element를 방출합니다. 옵저버블이 이벤트를 방출할 때, 그것들은 `next` 이벤트를 방출한 겁니다.

아까 말했죠. 방출하는 이벤트는 다른 것들이 될 수도 있다고요.
그래서 “탭(tap)” 도 가능합니다.
![](https://images.velog.io/images/kipsong/post/283a4018-c2d7-454c-8a44-0571d24bc241/IMG_A604661E9B8A-1.jpeg)


이 옵저버블은 3 개의 탭 이벤트를 방출하고 있고 그 이벤트는 `next` 이벤트입니다.

그리고 위 마블다이어그램에는 표시되어있지 않지만, `completed` 이벤트를 호출할 수도 있습니다.

이 이벤트는 옵저버블이 더이상의 이벤트를 호출하지 않고 “종료” 를 알리는 이벤트입니다.

그래서 이후에 next 이벤트를 추가하더라도 이벤트가 방출되지 않습니다. (자연스럽게 구독자들도 이벤트를 수신하지 못하겠죠.)

이렇게 정상적으로 성공하고 종료되는 경우가 `completed` 이벤트라면

비정상적으로 종료되는 경우도 있겠죠.

그때 방출하는 이벤트가 `error` 이벤트입니다.

error 이벤트도 마찬가지로 더이상의 이벤트를 방출하지 않습니다.

> 정상적으로 완료하고 종료하는 경우 -> completed Event
> 비정상적으로 종료하는 경우 -> error Event

이벤트의 정의를 보겠습니다.

```swift
public enum Event<Element> {

	case next(Element)
	case error(Swift.Error)
	case completed
}
```

위에서 설명한 그대로 enum 으로 정의되어 있죠.

이제 직접 작성해보겠습니다.

## 옵저버블 생성

정리하자면, 옵저버블을 생성한다는 건 무엇을 생성하는거다?

네, 맞습니다 시퀀스를 생성하는 것이다.

시퀀스에는 무엇이 있다?

이벤트들이 있다.

코드를 보겠습니다.
(Playground에서 보통 실행하는 예제인데, Playground 라이브러리 인덱싱 에러때문에 불쾌지수가 상승해서 그냥 빈 ViewController로 저는 진행했습니다. 참고하세요.)

```swift
import UIKit
import RxSwift

class ViewController: UIViewController {

    override func viewDidLoad() {
        super.viewDidLoad()
        
        example(of: "just, of, from") {
            
            // 1 전달하고 싶은 인스턴스를 생성한다.
            let one = 1
            let two = 2
            let three = 3
            
            // 2 옵저버블을 생성하고 옵저버블 타입을 지정한다. 
			   // 그리고 just 오퍼레이터를 호출한다.
            let observable = Observable<Int>.just(one)
        }   
    }
    
    func example(of: String, _ completion: () -> ()) {
        print("===",of,"===")
        completion()
    }
}
```

전체 코드이고 보고 싶은 부분은 여기입니다.
```swift
example(of: "just, of, from") {
            
	// 1 전달하고 싶은 인스턴스를 생성한다.
	let one = 1
	let two = 2
	let three = 3
            
	// 2 옵저버블을 생성하고 옵저버블 타입을 지정한다. 
	// 그리고 just 오퍼레이터를 호출한다.
	let observable = Observable<Int>.just(one)
}   

```

먼저 그냥 인스턴스를 생성했습니다. 이건 Rx의 개념이랑은 무관한 부분이죠?

그리고 2 번보시면, 이제부터 시작입니다.

`Observable<Int>` 라는 객체를 생성했습니다.

옵저버블 생성해야하니 옵저버블을 했고, 제네릭타입으로 정의된 객체이므로 타입을 직접 타이핑해줬죠.

> 옵저버블을 생성하는데 옵저버블 타입은 Int야
라고 해석할 수 있습니다.

그리고 의문의 `.just`

just == 그냥 == 걍 == 그까이꺼 ==.. 뭐 이런 느낌의 단어죠?

그 느낌대로입니다.

> 옵저버블이고 Int로 생성해 근데 뭐 “이거 하나만 방출해” 

`just` 오퍼레이터는 단 하나의 element만 시퀀스에 가질 수 있습니다.

지금 위 코드로 따지면 Int 값 하나만 넣을 수 있다는 뜻입니다.

그러면,
> 여러개의 element를 옵저버블에 전달해주고 싶으면 어떻게 할까요?

이 떄 “of” 오퍼레이터를 사용하면 됩니다.

```swift
let observable2 = Observable.of(one, two, three)
```

`just`랑 `of`랑 차이점 보이시나요?

of 의 경우 타입추론을 하고 있죠.

![](https://images.velog.io/images/kipsong/post/eb3879bc-4e3d-4ddd-8f88-dea74f976e29/8569BDBD-74E4-413D-B579-24E4F3510D15.png)

타입을 선언해주지 않았습니다. 그럼에도 옵저버블의 타입을 Int로 추론하고 있습니다.

of 의 정의를 볼까요?
![](https://images.velog.io/images/kipsong/post/651f9aea-67ad-422d-a539-072fc170bee6/2590C886-3F12-487A-A670-B73E6CF873DE.png)

element를 array로 받을 수 있군요.

그러면 아래 코드로도 옵저버블을 선언할 수 있을 겁니다.

```swift
let observable3 = Observable.of([one, two, three])
```


마지막으로 알아볼 오퍼레이터는 `from` 입니다.
```swift
let observable4 = Observable.from([one, two, three])
```

from 의 경우는 array 만 받을 수 있군요.

![](https://images.velog.io/images/kipsong/post/ceb71a3a-ca92-4f5c-8c49-790be03da8ed/3D04140B-C383-462E-9042-BAFA48B47AB4.png)

여기까지 설명은 “옵저버블의 생성” 에 관한 글입니다.

아직 실행하더라도 아무일도 발생하지 않습니다.

왜냐면…

Subscriber (구독자) 가 없으니까요!
~~구독 알람 필수…~~

## 옵저버블 구독
iOS 개발을 하시면서, `NotificationCenter` 에 대해 들어보셨거나 이미 능숙하게 사용하시고 계신 분들이 많을 것이라 생각됩니다.

알림에 대해서 관찰자에게 알려주는 객체죠.

기존 알림이 RxSwift에서는 옵저버블이 담당하죠.

먼저 키보드의 frame을 변경하는 노티피케이션의 예제를 보여드리겠습니다.

```swift
let observer = NotificationCenter.default.addObserver(
  forName: UIResponder.keyboardDidChangeFrameNotification,
  object: nil,
  queue: nil) { notification in
  // Handle receiving notification
}
```

옵저버를 생성하는 코듭니다.

노티피케이션 센터에 관찰자로 등록하는데 그 알림 이름을 파라미터로 전달합니다. 그리고 알림을 받았을 때, 어떻게 처리할지를 클로저를 통해 표현하고 있군요.

RxSwift도 유사합니다.

`addObserver()` -> `subscribe()`  로 변한 것으로 생각하시면 편합니다 ^^

다시 한번 강조하자면, 옵저버블 그 자체는 이벤트를 보낼 수도 없고 어던 작업을 할 수도 없습니다.

구독자가 생겨야 그 이벤트를 전달하는 겁니다. (유튜브에서도 영상을 열심히 만든다고 영상이 바로 송출되는 것은 아니죠. 구독자나 시청자가 있어야 그 영상이 실행되는 것처럼요.)

정리하자면,

> 옵저버블은 시퀀스를 정의한다.
> “옵저버블을 구독하는 것”은 next() 이벤트를 Iterator처럼 호출하는 것이다.

설명은 그만하고 코드로 한번 해볼까요?

```swift
example(of: "subscribe") {
            let one = 1
            let two = 2
            let three = 3
            
            // 옵저버블을 생성하는대 of 오퍼레이터로 연산한다.
            let observable = Observable.of(one, two, three)
            
            // 옵저버블에 구독자를 추가한다.
            observable.subscribe { event in
                // 이벤트를 받아서 클로저 로직으로 처리한다.
                print(event)
            }
        }
```

이전 코드에서 아래 구독자만 추가했습니다.

결과는 다음과 같습니다.
```
=== of ===
next(1)
next(2)
next(3)
completed
```

저는 `next(IntValue)` 라고 작성한 적이 없이 Int 만 작성했는데 next 라는 명칭이 붙어있네요?
그리고 completed 도 호출한 적이 없습니다.

왜냐하면 클로저 부분을 보면.

우리가 event에서 전달받은 “값” 을 콘솔에 출력한 것이 아니라 
event 그 자체를 출력했기 때문이죠.

of에 있는 element들은 event로 전달된겁니다.
그래서 값만 출력하고 싶다면, 코드를 이렇게 바꾸면 됩니다.

```swift
observable.subscribe { event in
  if let element = event.element {
    print(element)
  }
}
```

코드를 다시 아래와 같이 변경해보겠습니다.
```swift
observable.subscribe(onNext: { element in
  print(element)
})
```

`onNext` 가 추가되었죠. 이번에는 이벤트의 종류에 따라서 어떻게 처리하겠다는 걸, 명시적으로 작성한 경우입니다.

`empty` 라는 오퍼레이터가 있습니다. 

단어 뜻 그대로 비어있는 걸 출력합니다.
```swift
observable.subscribe(
  // 1
  onNext: { element in
    print(element)
  },

  // 2
  onCompleted: {
    print("Completed")
  }
)
```

이전에 봤던 코드랑 유사하죠.

다만 이전에는 `subscribe` 에 onNext 파라미터만 있었다면 이번에는 onCompleted 파라미터에도 표현식을 넣어준 것이 차이점입니다.

콘솔에는 다음과 같이 출력됩니다.
```
--- Example of: empty ---
Completed
```

`empty` 와 반대로 동작하는  `never` 라는 오퍼레이터가 있습니다.

never 오퍼레이터는 어떤 이벤트도 방출하지 않습니다. 

completed 조차 방출하지 않습니다.

```swift
example(of: "never") {
  let observable = Observable<Void>.never()

  observable.subscribe(
    onNext: { element in
      print(element)
    },
    onCompleted: {
      print("Completed")
    }
  )
}
```


옵저버블을 생성하다보면 Iterator 처럼 1부터 쭈우욱 무언가 처리해주고 싶을 수 있겠죠.

그럴 때 사용하는 오퍼레이터가 `.range` 오퍼레이터입니다.

피보나치 수열을 처리하는 옵저버블과 구독자를 생성한 코드입니다.
```swift
example(of: "range") {
  // 1
  let observable = Observable<Int>.range(start: 1, count: 10)

  observable
    .subscribe(onNext: { i in  
      // 2
      let n = Double(i)

      let fibonacci = Int(
        ((pow(1.61803, n) - pow(0.61803, n)) /
          2.23606).rounded()
      )

      print(fibonacci)
  })
}
```


1. 옵저버블을 생성했습니다. 그리고 오퍼레이터로 range를 활용했습니다. 파라미터로 시작하는 값과 생성할 int value를 결정하는 count 파라미터에 값을 할당했습니다.
2. onNext 파라미터에 클로저표현식을 통해 구독자가 이벤트에 대해 처리할 로직을 추가했습니다.
(지금은 피보나치를 통해 했지만, 실제로 구현하실 때는 원하시는 표현을 넣으면 되겠죠?)


## Disposing 와 종료

지금까지 개념을 살펴보면 다음과 같습니다.
- 옵저버블을 생성한다.
- 옵저버블의 세부 구현을 오퍼레이터를 통해서 한다.
- 구독자가 이벤트별로 어떻게 처리할 지 로직을 구현한다.

이렇게 3 단계죠.

그런데 subscribe 의 정의를 보면 다음과 같습니다.

![](https://images.velog.io/images/kipsong/post/c01ee6c4-e9df-4dfa-b8dd-e4520547d731/FA7A3F25-A341-45E6-9CFE-FE5A9B69C9AE.png)

return 값이 있었죠. “Disposable” 이라는 친구요.

우리가 직접 처리해주지 않아도 처리가 되고 있긴 합니다.

다만, 공식문서 (Rx 문서) 에서는 이를 명시적으로 처리하길 강력권고하고 있습니다.

그러므로 앞으로 구독 이후에 dispose를 다음과 같이 처리해줍시다.

```swift
        example(of: "dispose") {
            
            let bag = DisposeBag()
            
            let observable = Observable.of("A", "B", "C")
            
            let subscription = observable.subscribe { event in
                print(event)
            }
            .disposed(by: bag)
        }
```


아직 왜 사용해야하는지에 대해 말씀드리지 않았죠.

사용하는 이유는

> Dispose를 통해 옵저버블을 제거하는 타이밍일 컨트롤할 수 있고, 메모리 관리가 용이하다.

입니다.

바로 위 코드에서 `bag` 라는 변수가 메모리에서 해제될 때, `.disposed(by:)` 메소드에 해당 변수를 설정한 옵저버블도 함께 해제됩니다.

이렇게 언제 메모리에서 해제될 지 확실하게 알 수 있죠.
(이부분은 메모리 관리에 어려움을 느껴본 사람은 감탄할 것입니다…)

## Create
이전 예제까지는 옵저버블을 next 이벤트의 요소에 대해서만 생성했었죠.

이번에는 `create` 라는 오퍼레이터를 사용해서 다른 방식으로 이벤트를 생성하고 그 이벤트를 구독자에게 전달해보겠습니다.

(이전까지는 옵저버블에 어떤 element를 전달할지에 따라서 오퍼레이터를 사용했고, 해당 element를 next로 구독자에게 전달했습니다. 그리고 그 element에 대한 처리는 구독하는 시점에 결정했죠. 이번에는 구독하는 시점이 아니라 옵저버블을 생성하는 시점에 어떻게 처리할지를 정의하는 겁니다.)

코드를 보겠습니다.
```swift
example(of: "create") {
  let disposeBag = DisposeBag() // 바로 이전에 공부했던 개념이죠.
	
	Observable<String>.create { observer in 
	
	}
}
```

이번에 보면 옵저버블을 생성할 때, `create` 를 사용했고, 파라미터로 옵저버를 전달받아서 옵저버를 표현식 내에서 로직을 처리할겁니다.

그리고 create 의 문서를 볼까요?

![](https://images.velog.io/images/kipsong/post/d7bd0c0a-37ce-465d-a13c-5c18d9aefc34/FF8E59F9-4C7C-42D1-A290-B30DDF2AF68B.png)

return 으로 `Disposable` 를 클로저에서 리턴하고 메소드 전체에서는 옵저버블자체를 리턴하고 있습니다.

(아마 위 코드를 작성하고 컴파일에러가 떠 있을 겁니다. 왜냐하면 return type이 불일치하니까요.)

이제 위 코드를 마저 완성해보겠습니다.


```swift
example(of: "dispose") {
    let disposeBag = DisposeBag()
    Observable<String>.create { observer in
        // 구독자에게 전달할 이벤트를 정의할 수 있게하는 오퍼레이터 "create"
        // 1 next 이벤트를 옵저버에게 전달한다.
        observer.onNext("1")
        
        // 2 complted 이벤트를 옵저버에게 전달한다.
        observer.onCompleted()
        
        // 3 completed 이벤트를 방출했으므로 더이상 이벤트를 전달하지 않는다.
        observer.onNext("?")

        return Disposables.create()
    }
}
```

(코드에 대한 설명은 주석으로 대체합니다.)

이제 구독을 해볼까요?

```swift
.subscribe(
  onNext: { print($0) },
  onError: { print($0) },
  onCompleted: { print("Completed") },
  onDisposed: { print("Disposed") }
)
.disposed(by: disposeBag)
```

(위 코드를 옵저버블 코드블럭 끝나는 곳에 추가해주면 됩니다.)

결과는 위에 주석에서 적은 것처럼 출력되겠죠.

`1 -> Completed 이벤트 -> disposed`
“?” 는 출력되지 않을 겁니다.

```
--- Example of: create ---
1
Completed
Disposed
```

그동안 Error의 출력에 대해서는 다루지 않았죠.

에러를 처리할 때는 enum을 통해서 정의하고 이를 전달하는 식으로 많이 구현합니다.

```swift
enum MyError: Error {
	// 발생할 수 있는 에러의 케이스별로 구현하면 됩니다.
  case anError
}
```

create 코드블럭에 가서 Error event를 호출해보겠습니다.

아래 코드를 onNext와 onCompleted 사이에 추가해주세요.
```swift
observer.onError(MyError.anError)
```

출력결과는 다음과 같습니다.
```
--- Example of: create ---
1
anError
Disposed
```

여기까지 create 오퍼레이터부터 시작해서 각 이벤트 처리하는 법과 사용방법에 대해 살펴봤습니다.


## 옵저버블 팩토리 생성
구독자를 기다리는 옵저버블을 만드는 것 보다 옵저버블 팩토리를 만들어서 이를 구독자에게 전달해주는 방법이 더 나은 방법일 수 있습니다.

코드로 보겠습니다.
```swift
        example(of: "deferred") {
            let disposeBag = DisposeBag()
            
            // 1 Boolean 타입 변수를 생성한다.
            var filp = false
            
            // 2 옵저버블을 생성한다.
            let factory: Observable<Int> = Observable.deferred {
                
                // 3 Boolean값을 토글한다.
                filp.toggle()
                
                // 4 분기문을 통해 옵저버블을 리턴한다.
                if filp {
                    return Observable.of(1, 2, 3)
                } else {
                    return Observable.of(4, 5, 6)
                }
            }
        }
```

이전과 다른 점은 `.deferred` 를 사용했다는 점 입니다.

deferred 오퍼레이터는 
```swift
public static func deferred(_ observableFactory: @escaping () throws -> Observable<Element>)
```
다음과 같이 선언하고 있습니다.

deferred 오퍼레이터는 옵저버블을 만들어내는 팩토리 클로저를 파라미터로 받고 있습니다.
그리고 구독이 일어나는 시점이 되면 옵저버블을 생성합니다. 

이름 뜻 그대로
> 옵저버블이 만들어지는 시점을 미룬다. == deferred
입니다.

보통 특정 데이터의 상태 (아마 Boolean을 통한 if분기문 이나 switch 분기문을 활용해서 나타내는 부분)에 따라서 다른 옵저버블을 리턴해야하는 경우 사용합니다.

위 코드에서는 한 번 옵저버블을 리턴하면 Boolean 값이 변경되고 있죠. 그래서 상태가 변할 때마다 
 123 -> 456 -> 123 -> 456 이런식으로 리턴되는 이벤트 element가 변경될 것입니다.

아래 코드를 추가해서 확인해보죠.

```swift
for _ in 0...3 {
  factory.subscribe(onNext: {
    print($0, terminator: "")
  })
  .disposed(by: disposeBag)

  print()
}
```

```
=== deferred ===
123
456
123
456
```


## Traits 사용하기
`Traits`은 일반 옵저버블보다 좁은 행동 집합을 갖는 옵저버블입니다.

`Traits` 는 선택적으로 사용하면 됩니다. (필수 X) 

> 그러면 왜 사용하는가.

코드가독성을 위해서 사용합니다. Rx의 장점 중에서 코드 가독성이 한 몫하니까요 ^^

Rx에서는 3 종류의 Traits 를 제공합니다.
- Single
- Maybe
- completable

`Single` 은 `success(value)` 나 `error(error)` 이벤트를 방출합니다.

`success(value)` 는 next와 completed 이벤트의 조합입니다. 

주로 one-time process에 주로 사용합니다. 
ex) 데이터를 다운로드 받는 경우

`Completable` 은 `completed` 과 `error(error)` 이벤트를 방출합니다. 이것들은 특정 값을 방출하진 않습니다. 성공적으로 오퍼레이션이 종료되거나 문제가 발생한 경우에 사용하는거죠.
ex) 파일 쓰기

`Maybe` 는 `Single` 과 `Completable` 의 조합입니다. 그러므로 `success(value)` , `completed` 혹은 `error(error)` 를 방출하죠.

상세한 건 추후에 다른 글에서 설명하겠습니다.


코드를 보겠습니다.
```swift
// disposebag을 생성한다.
let disposeBag = DisposeBag() 

// 에러를 열거형으로 정의한다.
enum FileReadError: Error {
 case fileNotFound, unreadable, encodingFailed
} 

// Single을 생성할 함수를 생성한다.
func loadText(from name: String) -> Single<String> {
	// Single을 리턴하고 create를 통해서 구독자에게 전달될 형태를 정의한다.
	return Single.create { single in 
  ... (생략) ...
}
```

1. disposebag을 생성한다.
2. 에러 열거형을 정의한다.
3. single을 생성할 수 있도록 메소드화시킨다.

간단하죠?
코드를 더 보겠습니다.
(아래 코드를 Singfle.cretae 코드블럭에 위치시키면 됩니다.)
```swift
// Disposables를 생성한다.
// 이전에 생성했는데 또 생성한 이유는, cretae 클로저에서 요구하는 리턴 타입에 맞추기 위함입니다.
let disposable = Disposables.create()

// 번들파일에 해당 파일이 있다면 해당 경로 인스턴스를 생성한다. 없으면 에러 방출
guard let path = Bundle.main.path(forResource: name, ofType: "txt") else { single(.failure(FileReadError.fileNotFound)) 
return disposable
}

// 파일매니저 싱글톤객체로 해서 경로로 이동한 이후 데이터를 인스턴스로 생성한다. 없으면 에러 방출
guard let data = FileManager.default.contents(atPath: path) else {
	single(.failure(FileReadError.unreadable))
	return disposable
}

// 해당 데이터의 type을 String으로 하고 인코딩은 "UTF-8" 로 설정한다. 타입캐스팅에 실패하면 에러를 방출
guard let contents = String(data: data, encoding: .utf8) else {
	single(.failure(FileReadError.encodingFailed)
	return disposable
}

// guard 문을 모두 통과했다면, "success" 방출과 함께 value를 방출한다.
single(.success(contents))
return disposable
```


호출은 아래와 같이 하겠죠.
```swift
loadText(from: "CopyRight")
	.subscribe {
		swith $0 {
			case .success(let string):
			 print(string)
			case .error(let error):
			 print(error)
		}
	}
	.disposed(by: disposeBag)
```

메소드만 호출하면, 이제 single이 생성되겠죠.

그리고 해당 메소드를 구독하면, 이벤트들을 전달받을 겁니다. 

구독자 입장에서는 이벤트를 어떻게 처리할 지 subscribe 클로저에서 처리하면 되겠구요.

여기까지 옵저버블에 대한 설명을 했습니다.


## 정리

Q. 옵저버블은 무엇인가?
Q. 옵저버블의 생명주기는 어떻게 되는가?
Q. 옵저버블은 어떻게 생성하는가?
Q. 옵저버블을 어떻게 구독하는가?
Q. 옵저버블을 메모리에서 어떻게 해제시키는가?

에 대해서 답할 수 있게 글을 작성했습니다.

읽어주셔서 감사합니다.



## 참고자료
- [RxSwift연산자-deferred](https://jcsoohwancho.github.io/2019-09-01-RxSwift%EC%97%B0%EC%82%B0%EC%9E%90-deferred/)
- https://www.raywenderlich.com/books/rxswift-reactive-programming-with-swift/v4.0/chapters/2-observables
