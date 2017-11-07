---
title: Rxjava 学习笔记
categories: 技术
toc: true 
theme: next
date: 2017-07-27 22:10:00
---
#[Rxjava](http://reactivex.io/)
[GitHub](https://github.com/ReactiveX/RxJava)
[JavaDoc](http://reactivex.io/RxJava/2.x/javadoc/)

## 与Rxjava1.x 的差距
1. Nulls

2.x 不允许发射事件的时候传入 null值。这意味着 Observable<Void> 不再发射任何值，而是正常结束或者抛出空指针。

2. Flowable 支持背压

背压： 被观察者发送事件的速度远快于观察者的处理速度的情况下

3. Single/Completable/Maybe

- Completable 侧重于观察结果，发送 0 事件
- Single 顾名思义，只能发送一个事件
- Maybe 是上面两种的结合体，发送0-1事件
4. 线程调度相关

没有 `Schedulers.immediate()` 和 `Schedulers.test()`

5. Function相关
1.x  ->   2.x

- Func1 -> Function
- Func2 -> BiFunction 
- onCompleted -> onComplete
- CompositeSubscription -> CompositeDisposable
- limit 被移除 -> take

## 配置
` compile 'io.reactivex.rxjava2:rxjava:2.1.2'`

# 教程

## Observable.Create
 - onComplete 后，可以发射数据，但不接受数据
 - 发射事件方法相比 1.x 多了一个throws Excetion
 - Disposable 可以直接调用切断
     + isDisposed = false，接收器正常   
     + `disposable.dispose()` 后，isDisposed = true，接收器停止接收

```java
 Observable.create(new ObservableOnSubscribe<Integer>() {
            @Override
            public void subscribe(@NonNull ObservableEmitter<Integer> e) throws Exception {
                System.out.print("\nemit 1");
                e.onNext(1);
                System.out.print("\nemit 2");
                e.onNext(2);
                System.out.print("\ncall onComplete");
                e.onComplete();
                System.out.print("\nemit 3");
                e.onNext(3);
            }
        }).subscribe(new Observer<Integer>() {
            private Disposable disposable;

            @Override
            public void onSubscribe(@NonNull Disposable d) {
                this.disposable = d;
                System.out.print("\nonSubscribe,isDisposed:" + d.isDisposed());
            }

            @Override
            public void onNext(@NonNull Integer integer) {
                System.out.print("\nonNext(), value:" + integer);
                if (integer == 1) {
                    System.out.print("\ndisposable.dispose()");
                    disposable.dispose();
                    System.out.print("\ndisposable.isDisposed().value:"+disposable.isDisposed());
                }
            }

            @Override
            public void onError(@NonNull Throwable e) {
                System.out.print("\nonError(), Message:" + e.getMessage());
            }

            @Override
            public void onComplete() {
                System.out.print("\nonComplete()");
            }
});
```

结果

```java
onSubscribe,isDisposed:false
emit 1
onNext(), value:1
disposable.dispose()
disposable.isDisposed().value:true
emit 2
call onComplete
emit 3
```