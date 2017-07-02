---
title: Rxjava(待整理)
categories: 技术
toc: true 
theme: next
date: 2017-07-01 23:00:00
---

[ RxJava中的Single操作符与Subjects](http://blog.csdn.net/wbwjx/article/details/51265266)

[探索专为 Android 而设计的 RxJava 2](https://my.oschina.net/bv10000/blog/809176)

## Subscriber  ##
1. onNext
1. onCompleted
1. onError
1. onStart
2. unsubscribe
3. doOnSubscribe  指定的线程来做准备工作


## （二）基本实现 ##
#### 1. 创建 Observer ####
（1）Rx1.x

        Subscriber<String> subscriber = new Subscriber<String>() {
	    	@Override
	    	public void onNext(String s) {
	    		Log.d(tag, "Item: " + s);
	    	}
	    	
	    	@Override
	    	public void onCompleted() {
	    		Log.d(tag, "Completed!");
	    	}
	    	
	    	@Override
	    	public void onError(Throwable e) {
	    		Log.d(tag, "Error!");
	    	}
	    };
（2）Rx2.x

		  Observer<String> subscriber1 = new Observer<String>() {
            @Override
            public void onSubscribe(Disposable d) {
            }

            @Override
            public void onNext(String s) {
            }

            @Override
            public void onError(Throwable e) {
            }

            @Override
            public void onComplete() {
            }
        };

#### 2. 创建 Observable ####

- create ： 最基本的创造事件序列的方法
（1）Rx1.x

		Observable observable = Observable.create(
		    new Observable.OnSubscribe<String>() {
			    @Override
			    public void call(Subscriber<? super String> subscriber) {
			        subscriber.onNext("Hello");
			        subscriber.onNext("Hi");
			        subscriber.onNext("Aloha");
			        subscriber.onCompleted();
	    	}
		});
（2）Rx2.x

		Observable<String> observable = Observable.create(
               new ObservableOnSubscribe<String>() {
                   @Override
                   public void subscribe(ObservableEmitter<String> e) throws Exception {
                       e.onNext("Hello");
                       e.onNext("RxJava2");
                       e.onComplete();
                   }
               }
        );
- just(T...) ： 将传入的参数依次发送出来。

    	Observable observable = Observable.just("Hello", "Hi", "Aloha");
		// 将会依次调用：
		// onNext("Hello");
		// onNext("Hi");
		// onNext("Aloha");
		// onCompleted();
- from(T[]) / from(Iterable<? extends T>) ： 将传入的数组或 Iterable 拆分成具体对象后，依次发送出来。

		String[] words = {"Hello", "Hi", "Aloha"};
		Observable observable = Observable.from(words);
		// 将会依次调用：
		// onNext("Hello");
		// onNext("Hi");
		// onNext("Aloha");
		// onCompleted();
#### 3. Subscribe (订阅) ####
		observable.subscribe(observer);
		// 或者：
		observable.subscribe(subscriber);
## （二）线程控制 Scheduler ##
### 1.API ###
1. Schedulers.immediate()
2. Schedulers.newThread(): 启用新线程
3. Schedulers.io(): I/O 操作（读写文件、读写数据库、网络信息交互等）所使用的 Scheduler，无数量上限的线程池，可以重用空闲的线程
4. Schedulers.computation(): 计算所使用的 Scheduler。CPU 密集型计算，固定的线程池，大小为 CPU 核数
5. AndroidSchedulers.mainThread():指定的操作将在 Android 主线程运行

`subscribeOn()`:指定 `subscribe()` 所发生的线程，即 `Observable.OnSubscribe` 被激活时所处的线程。或者叫做事件产生的线程。

`observeOn()`: 指定 `Subscriber` 所运行在的线程。或者叫做事件消费的线程。

	Observable.just(1, 2, 3, 4).subscribeOn(Schedulers.io()) // 指定 subscribe() 发生在 IO 线程
        .observeOn(AndroidSchedulers.mainThread()) // 指定 Subscriber 的回调发生在主线程
        .subscribe(new Action1<Integer>() {
        	@Override
            public void call(Integer number) {
            	Log.d(tag, "number:" + number);
            }
   	});
## （三）变换  ##
1. `map()`
1. `flatMap()` :返回的是个 `Observable` 对象


#### Action1 ####

		Observable.just("Hello, world!")  
    		.subscribe(new Action1<String>() {  
        		@Override  
        		public void call(String s) {  
              		System.out.println(s);  
        		}  
    	});  

#### Func1 ####

		Observable.just("Hello, world!")  
  			.map(new Func1<String, String>() {  
      			@Override  
      			public String call(String s) {  
          			return s + " -Dan";  
      			}  
  			})  
  			.subscribe(s -> System.out.println(s));  


> 1. Creating Observables(Observable的创建操作符)，比如：Observable.create()、Observable.just()、Observable.from()等等；
> 1. Transforming Observables(Observable的转换操作符)，比如：observable.map()、observable.flatMap()、observable.buffer()等等；
> 1. Filtering Observables(Observable的过滤操作符)，比如：observable.filter()、observable.sample()、observable.take()等等；
> - filter 是用于过滤数据的，返回false表示拦截此数据。
> - take 用于指定订阅者最多收到多少数据。
> - 
> 1. Combining Observables(Observable的组合操作符)，比如：observable.join()、observable.merge()、observable.combineLatest()等等；
> 1. Error Handling Operators(Observable的错误处理操作符)，比如:observable.onErrorResumeNext()、observable.retry()等等；
> 1. Observable Utility Operators(Observable的功能性操作符)，比如：observable.subscribeOn()、observable.observeOn()、observable.delay()等等；
> 1. Conditional and Boolean Operators(Observable的条件操作符)，比如：observable.amb()、observable.contains()、observable.skipUntil()等等；
> 1. Mathematical and Aggregate Operators(Observable数学运算及聚合操作符)，比如：observable.count()、observable.reduce()、observable.concat()等等；
> 1. 其他如observable.toList()、observable.connect()、observable.publish()等等；


doOnNext 允许我们在每次输出一个元素之前做一些额外的事情。

	public class MyHandler {
	    public void myOnClick(View view){
	        Toast.makeText(view.getContext(),"hello",Toast.LENGTH_SHORT).show();
	    }
	    public boolean myOnLongClick(View view){
	        Toast.makeText(view.getContext(),"hello world",Toast.LENGTH_SHORT).show();
	        return true;
	    }
	}
