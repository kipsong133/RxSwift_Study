## Rx를 쓰는 이유

---

 RxSwift는 값이 변경되면, 그 값에 맞게 새롭게 연산해서 결과를 처리하고 UI까지 자동으로 변경되었으면 하는 니즈를 충족해줍니다.

기존에 아래와 같은 코드가 있다고 해봅시다.

```swift
var x = 1
var y = 2
print(x + y) // 3
a = 98 
```

- 위 코드를 보면 `a = 98`이 되었지만 위에 있는 코드는 자동으로 다시 연산하지 않습니다. 즉, 값은 변했으나, 연산결과는 아직 변경되지 않은 상태죠.
- 이것을 iOS UI 작업에서 생각해보면, 내가 네트워킹 혹은 다른 곳에서 값이 변경되었는데, 제가 구현한 UI는 변경되지 않은 상태입니다.



이러한 코드에 Rx를 적용해보겠습니다.

```swift
let bag = DisposeBag()
let x = BehaviorSubject(value: 1) // Default value: 1
let y = BehaviorSubject(value: 2) // Default value: 1

Observable.combineLatest(a, b) { $0 + $1 } // 2 개의 파라미터를 더한다.
	.subscribe(onNext: { print($0) })				 // next 이벤트에 대해 프린트한다.
	.disposed(by: bag)											 // disposeBag에 등록한다.

x.onNext(12)	// onNext 이벤트로 12를 전달한다.
```

- 위 코드를 설명하자면, x와 y를 더하도록 `Observable.combineLatest` 라는 객체를 사용했습니다. 이를 통해서 덧셈연산을 구현했죠. 마치 수식을 세우는 것 같죠?
- 그리고 `x.onNext`코드에서는 12를 전달하고 있습니다. 그리고 `print` 문을 전혀 쓰고 있지 않습니다.
- 하지만 Rx이므로 자동으로 연산되어 print문이 호출됩니다. (자세한 이야기는 이어지는 글에서 설명할게요.)



지금은 아주 단순한 코드입니다. 이것을 어떤 케이스에서 활용할지 고민해보고 싶었습니다.

- 주문화면을 구현하는데, `수량 x 가격` 이라는 연산이후에, 이것을 UI로 업데이트 하고 싶다.
- 사용자가 닉네임을 수정했는데, 닉네임 수정된 UI에 UI를 업데이트 하고 싶다.
- 회원가입 화면에서 동의하기를 모두 눌렀다면, 모두 동의 버튼도 이미지를 변경하고 싶다. 역으로 하나라도 동의가 안된다면, 모두동의 버튼의 이미지를 default로 되돌리고 싶다.

이런 경우가 있지 않을까요?



이런 질문이 있을 수 있겠죠.

> Q. 그런데 그런 것들은 Rx를 안써도 구현가능하지 않나요?



네, 맞습니다. 안써도 가능합니다. 프로퍼티옵저버를 활용하거나 이미 프로토콜에 구현된 메소드 그리고 생명주기 함수를 활용하면 충분히 구현가능합니다. even though, RxSwift 이냐면,

1. 안쓸경우에, 문제인 하나의 로직 구현을 위해서 코드가 분산된 문제가 해결됩니다. 다른 방식으로 구현하면, 하나의 로직을 구현하는데, 해당 로직들이 모두 분산되어있죠. 마치 책을 읽듯 내가 특정 로직을 실행하면 그 로직이 바로 다음줄에 있으면 좋겠는데, 프로퍼티는 맨위에 있고 델리게이트 메소드는 맨 아래에 있고 생명주기는 중간에 있고. 이런식으로 집약되어있지 않은 문제를 해결해줍니다.
2. Rx에서 제공하는 좋은 기능들을 아주 편리하게 사용할 수 있습니다. Rx를 안쓰면, 매번 유사한 기능을 매번 구현해야할 겁니다. (물론 어떻게 해서든 단순화 시킬 수 있겠지만, 그러면 Rx를 쓰면 편리하겠죠.) 위 코드뿐만 아니라, 제일 마지막 값만 리턴한다든가 버튼이 여러번 눌리는 것을 방지한다든가 다양한 기능들을 아주 편리하게 사용할 수 있습니다.
3. iOS 개발자들이 대중적으로 사용하고 있습니다. 어찌보면, 이것이 큰 이유가 아닐까 생각됩니다. 혼자 개발하고 미래에도 혼자만 개발할 생각이라면, 굳이 Rx가 필요 없을 수도 있겠네요. 다만, 컴퓨터과학의 많은 분야들이 약속을 하고 소통하는 것처럼 Rx를 사용하는 iOS 개발자끼리 커뮤니케이션 비용이 줄어들겠죠.



이 외에도 Rx의 장점들이 있겠지만, 위 3 개는 제가 경험에서 느낀 장점입니다. 그러므로 

> RxSwift는 투자 대비 좋은 혜택을 준다.

로 정리하겠습니다.



## Observable

---

**결론**

```markdown
1. Observable  -> Observer : 이벤트를 전달한다.
2. Observer -> Observable : 옵저버블을 구독하다.
```



## Observable이란?

```swift
/// A type-erased `ObservableType`. 
///
/// It represents a push style sequence.
public class Observable<Element> : ObservableType {
    init() {
#if TRACE_RESOURCES
        _ = Resources.incrementTotal()
#endif
    }
    
    public func subscribe<Observer: ObserverType>(_ observer: Observer) -> Disposable where Observer.Element == Element {
        rxAbstractMethod()
    }
    
    public func asObservable() -> Observable<Element> { self }
    
    deinit {
#if TRACE_RESOURCES
        _ = Resources.decrementTotal()
#endif
    }
}
```

- 위 코드는 라이브러리의 코드를 그대로 가져온 것입니다.
- Observable 선언 코드에 보면 `ObservableType` 을 채택하고 있습니다. 그리고 `<Element>` 타입을 전달해줘야하네요.
- 그러먼 우리는 Observable을 만들 때, 타입을 꼭 써주면 될 것 같습니다. 그와 동시에 `subscribe` 메소드를 통해서 파라미터로 업저버를 전달해주면 되겠네요.(물론 `Observer`타입을 준수하는 객체어야겠죠.)



옵저버블은 신문사와 신문사 구독과 같다고 생각이 듭니다. 제가 특정 신문을 구독하면, 신문은 사건사고(이벤트)가 있을 때마다 이벤트를 구독자들에게 보내겠죠.(온라인 신문 기준) 그러면 구독자들은 해당 내용을 받을겁니다. 그런식으로 이벤트를 주고 그리고 그 이벤트를 받는 관계가 옵저버블과 옵저버라고 생각됩니다.



그러면 이걸 (When)언제쓰고, (How)어떻게 쓰는지 생각해볼게요.

- 이벤트를 주는 것을 iOS 프로그래밍 기준으로 생각해보면, ViewModel에서 네트워크 통신을 해서 값을 받은 이후에, ViewController에 전달하는 순간이 있을 것 같아요, 그리고 ViewController 내부에 있는 데이터를 업데이트하겠죠. 정리하면, 네트워크 통신을 통해서 홈화면에 보여줄 다량의 데이터를 받아오면(옵저버블이) 옵저버에게 해당 데이터를 전달해줍니다. 그리고 구독자는 그 이벤트를 어떻게 처리할 지 결정해두어야 겠죠. 자연스럽게 옵저버블은 이벤트 처리관련된 로직이 있어야만 합니다. (그건 이어지는 글에서 설명할게요.)

그러면 이제 어떻게 코드를 작성하는지 알아보겠습니다.

1. 옵저버블을 생성한다.
2. 옵저버블을 구독한다.
3. 이벤트를 전달한다.

이 순서로 구현해야겠죠.



Observable이 전달하는 이벤트

- next : 이벤트를 전달한다.
- error : 에러 이벤트를 전달하고, 옵저버블을 종료한다.
- completed : 컴플리트 이벤트를 전달하고, 옵저버블을 종료한다.



### 1. 옵저버블을 생성한다.

옵저버블을 생성하는 메소드는 `create` 라는 연산자를 활용합니다. Rxswift의 코드를 보면 아래와 같이 정의되어 있습니다.

```swift
extension ObservableType {
    // MARK: create

    /**
     Creates an observable sequence from a specified subscribe method implementation.

     - seealso: [create operator on reactivex.io](http://reactivex.io/documentation/operators/create.html)

     - parameter subscribe: Implementation of the resulting observable sequence's `subscribe` method.
     - returns: The observable sequence with the specified implementation for the `subscribe` method.
     */
    public static func create(_ subscribe: @escaping (AnyObserver<Element>) -> Disposable) -> Observable<Element> {
        AnonymousObservable(subscribe)
    }
}
```

- create 메소드의 리턴타입을 보면 `Observable` 을 리턴하고 있죠?

```swift
Observable<String>.create { (observer) -> Disposable in
    observer.on(.next(0)) // 구독자에게 next 이벤트를 전달한다.
    observer.onNext(1)    // 구독자에게 next 이벤트를 전달한다.
    observer.onCompleted() // 컴플리티드 메소드를 전달하고 옵저버블을 종료한다.
    return Disposables.create() // return 타입에 맞게 리턴한다.
}
```

- 위 코드는 정상적으로 실행이 안됩니다. 왜냐하면 `Observable` 의 Element Type을 String으로 했는데, next 이벤트는 Int로 전달하고 있습니다.
- Create 의 코드를 보면 `AnyObserver<Elemnt>` 라는 부분의 Element와 Observable의 타입이 다르기 때문에 에러가 발생합니다. 이처럼 옵저버블과 옵저버의 타입을 일치시켜줘야 컴파일에러를 피할 수 있습니다.  그래서 Int로 모두 통일하면 에러가 없겠죠.

```swift
Observable<Int>.create { (observer) -> Disposable in
    observer.on(.next(0)) // 구독자에게 next 이벤트를 전달한다.
    observer.onNext(1)    // 구독자에게 next 이벤트를 전달한다.
    observer.onCompleted() // 컴플리티드 메소드를 전달하고 옵저버블을 종료한다.
    return Disposables.create() // return 타입에 맞게 리턴한다.
}
```



오퍼레이터중에서 `from` 이라는 연산자를 활용하면 좀더 편리하게 구현가능합니다.

```swift
Observable.from([0, 1])
```

- from 메소드는 배열에 있는 값을 순차대로 이벤트로 방출한 이후에 컴플리티드 이벤트를 전달하고 종료합니다.

즉, 위에 있는 코드 중에서

```swift
observer.on(.next(0))
observer.onNext(1) 
obsserver.onCompleted()
```

이 부분을 코드 한 줄로 구현 가능하다는 뜻입니다.



무언가 다량의 이벤트를 순차대로 전달해야한다고 판단이 든다면 from 을 사용하면 좀 더 편리하게 구현할 수 있겠네요.



> 여기까지 하면 아무일도 발생하지 않는데요?

네, 여기까지의 코드는 옵저버블을 "생성"만 한 상태입니다. 마치 신문사가 신문회사를 차리기만 한 상태입니다. 아직 신문구독자가 아무도 없으니 신문을(혹은 인터넷기사를) 발행하지 않겠죠. 그러므로, 구독자가 생기면 해당 이벤트를 전달합니다.



위 내용을 보면, 제가 RxSwift의 장점 중 하나인, `코드가 분산된 문제가 해결됩니다`  가 보이시나요?

앞으로 옵저버블을 생성한 바로 아래 코드에서 제가 원하는 로직을 모두 작성하면 됩니다. 그리고 그 순서로 일을 처리하고 싶을 때, 구독만 하면 되는 것이죠. ~~구독과 좋아요 알림설정까지...~~

개인적으로 이 개념이 처음 볼 때는 너무 생소해서 무슨 말인가 싶었습니다. 그러다가 코드를 작성하면서, 델리게이트 메소드와 프로퍼티 등등 코드들이 분산되는 것이 심해지는 순간을 맛보고 난 이후에, Rx가 코드 가독성을 상당히 올려준다는 것을 느꼈습니다.



이어지는 글에서 이제 구독을 해보겠습니다.



## Observer

---

옵저버를 구독하는 방법은 간단합니다. `subscribe` 메소드를 이용하면 됩니다.

```swift
    /**
     Subscribes an event handler to an observable sequence.
     
     - parameter on: Action to invoke for each event in the observable sequence.
     - returns: Subscription object used to unsubscribe from the observable sequence.
     */
    public func subscribe(_ on: @escaping (Event<Element>) -> Void) -> Disposable {
        let observer = AnonymousObserver { e in
            on(e)
        }
        return self.asObservable().subscribe(observer)
    }
```

- 위 코드는 "ObservableType" 에 있는 소스코드입니다.
- 코드내용을 보면 'on' 이라는 파라미터에 이벤트를 전달합니다. 그리고 옵저버에서 다시, e 라는 이벤트를 전달받아서 표현식이 호출되고 있습니다.
- 정리하면, 옵저버블에서 전달받은 이벤트를 옵저버에게 전달합니다. on의 표현식은 개발자가 작성한 코드대로 진행되겠죠.
- 위 메소드는 next 이벤트만 처리할 때, 사용하면 되겠죠. 아마 대부분의 경우 이 코드를 사용해서 구독자에게 이벤트를 전달할 것 같네요.



subscribe의 다른 메소드르 보겠습니다.

```swift
    /**
     Subscribes an element handler, an error handler, a completion handler and disposed handler to an observable sequence.
     
     - parameter onNext: Action to invoke for each element in the observable sequence.
     - parameter onError: Action to invoke upon errored termination of the observable sequence.
     - parameter onCompleted: Action to invoke upon graceful termination of the observable sequence.
     - parameter onDisposed: Action to invoke upon any type of termination of sequence (if the sequence has
     gracefully completed, errored, or if the generation is canceled by disposing subscription).
     - returns: Subscription object used to unsubscribe from the observable sequence.
     */
    public func subscribe(
        onNext: ((Element) -> Void)? = nil,
        onError: ((Swift.Error) -> Void)? = nil,
        onCompleted: (() -> Void)? = nil,
        onDisposed: (() -> Void)? = nil
    ) -> Disposable {
            let disposable: Disposable
            
            if let disposed = onDisposed {
                disposable = Disposables.create(with: disposed)
            }
            else {
                disposable = Disposables.create()
            }
            
            #if DEBUG
                let synchronizationTracker = SynchronizationTracker()
            #endif
            
            let callStack = Hooks.recordCallStackOnError ? Hooks.customCaptureSubscriptionCallstack() : []
            
            let observer = AnonymousObserver<Element> { event in
                
                #if DEBUG
                    synchronizationTracker.register(synchronizationErrorMessage: .default)
                    defer { synchronizationTracker.unregister() }
                #endif
                
                switch event {
                case .next(let value):
                    onNext?(value)
                case .error(let error):
                    if let onError = onError {
                        onError(error)
                    }
                    else {
                        Hooks.defaultErrorHandler(callStack, error)
                    }
                    disposable.dispose()
                case .completed:
                    onCompleted?()
                    disposable.dispose()
                }
            }
            return Disposables.create(
                self.asObservable().subscribe(observer),
                disposable
            )
    }
}
```

- 파라미터로 총 4 개를 받고 있습니다 `onNext`, `onError`, `onCompleted ` 그리고 `onDisposed` 를 입력받고있습니다.
- 최종적으로 disposable을 생성하면서 메모리를 정리합니다.
- 이것은 구독자가 이벤트를 처리해야하는데, 4 종류에 따라서 어떤 로직을 실행할지 작성해주는 subscribe 메소드 입니다.
- 이 메소드는 네트워크 통신처럼 에러가 발생할 우려가 있는 경우에 사용하면 적절하다고 생각됩니다.



이제 코드로 작성해볼게요.



먼저 옵저버블을 생성합니다.

```swift
let observable = Observable.from([1, 2, 3])
```



그리고 해당 옵저버블을 넥스트이벤트만 처리하는 subscribe 메소드로 구독하겠습니다.

```swift
observable.subscribe { event in
    print("옵저버블: 이벤트를 옵저버에게 전달합니다.")
    print(event)
}
```

결과는 다음과 같습니다.

```
옵저버블: 이벤트를 옵저버에게 전달합니다.
next(1)
옵저버블: 이벤트를 옵저버에게 전달합니다.
next(2)
옵저버블: 이벤트를 옵저버에게 전달합니다.
next(3)
옵저버블: 이벤트를 옵저버에게 전달합니다.
completed
```

- 프린트된 콘솔창을보면, 이벤트를 1을 전달하고, 다시 옵저버에게 이벤트를 전달하고 2를 전달하고 이벤트를 전달하고 3을 전달하고  마지막으로 컴플리티드 이벤트를 전달합니다.
- 마치 for문처럼 클로제 내부를 순회하고 있죠. 즉, 1 -> 2 -> 3 이 아니라 1 -> 코드블럭전부호출 -> 2 -> 코드블럭 전부호출 -> 3 코드블럭 전부호출 로 호출합니다. 1을 전달하면서 2 에 접근해서 연산처리를 하거나 그럴 수 없다는 뜻이죠. 개별적인 동작입니다.
- 그리고 마지막엔 컴플리티드를 호출해서 구독자에게 끝났음을 전달하네요.



컴플리티드 부분에서 모든 로직을 처리한 이후에 `tableview.reloadData()` 같은 로직을 호출하고 싶을 수도 있지 않을까요?

그럴 때, 두 번째 subscribe 메소드를 이용합니다.

```swift
observable
    .subscribe(
        onNext: {
            print("옵저버블: 이벤트를 옵저버에게 전달합니다.")
            print($0)
        },
        onError: { _ in print("에러발생")},
        onCompleted: { print("종료되었습니다.")},
        onDisposed: { print("메모리 해제")})
```

- 이런식으로, 각각의 이벤트에 따라서 다르게 표현식을 정의합니다. 결과는 아래와 같습니다.

```
옵저버블: 이벤트를 옵저버에게 전달합니다.
1
옵저버블: 이벤트를 옵저버에게 전달합니다.
2
옵저버블: 이벤트를 옵저버에게 전달합니다.
3
종료되었습니다.
메모리 해제
```

- 원하는 방식으로 Error, completed disposed 에 대해서 처리할 수 있죠?



정리해보겠습니다.

1. 옵저버블은 어떤 데이터를 어떤 순서로 전달하고 종료할지를 구현한다.
2. 옵저버는 데이터를 받으면 어떻게 처리할 지를 구현한다.
3. 옵저버블과 옵저버는 subscribe 메소드로 연결한다.

