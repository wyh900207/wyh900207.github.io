---
layout: post
title:  "创建Observalbe"
date:   2017-07-31 17:21:00 +0800
description: '创建Observalbe'
tag: RxSwift
---

我们去[Observable+Creation.swift](https://github.com/ReactiveX/RxSwift/blob/84358af11882cfd2227e03b3648d1400eed08623/RxSwift/Observables/Observable%2BCreation.swift)里看一下, create方法如下:

```swift
public static func create(_ subscribe: @escaping (AnyObserver<E>) -> Disposable) -> Observable<E> {
        return AnonymousObservable(subscribe)
    }
```

这个方法只有一个参数`subscribe`,**关键也就在于`subscribe`**

这里的`subscribe`并不是一个真正的订阅者, 而是定义当有人订阅`Observable`中的时间时, 应该如何向订阅者发送不同的事件

`subscribe`是一个可逃逸闭包, `AnyObserver<E>`此时实际上是一个订阅的替身, 其实就是通过这个替身来想订阅者发送各种事件行为.

这样我们不难写出这样的代码:

```swift
let testObservable = Observable<Int>.create { observer in
    // next event
    observer.onNext(10)
    observer.onNext(20)
    
    // complete event
    observer.onCompleted()
}
```

最后, 这个`subscribe`要返回一个`Disposable`对象, 以此来取消订阅, 并释放 `testObservable`占用的资源.

```swift
let testObservable = Observable<Int>.create { observer in
    // next event
    observer.onNext(10)
    observer.onNext(20)
    
    // complete event
    observer.onCompleted()
    
    // Disposable
    return Disposables.create()
}
```

当需要发送错误事件时, 只需要发送一个遵从`Error`协议的具体错误就可以了, 例如定义一个错误如下:

```swift
enum RequestError: Error {
    case NetworkNotFound = "404"
}
```

想发送`next event`一样, 发送`error event`就可以了:

```swift
let testObservable = Observable<Int>.create { observer in
    // error event
    observe.onError(RequestError.NetworkNotFound)
}
```


