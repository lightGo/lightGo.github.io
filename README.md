# RXJS

## 简介
RxJS是一套Js库，专注于解决非同步问题的完整解决方案，对于异步，我们会有很多种解答的方式，例如：ajax,XhrRequest,promise...

## 基础知识

先看两个简单的DEMO，用这两段代码来描述一件事

> 遍历器模式(Iterator Pattern)

     let arr=['A','B'];
     let iterator = arr[Symbol.iterator]();

> 观察者模式(Observer Pattern)

     document.body.addEventListener('click',(e)=>{console.log(e)});
     Observable.fromEvent(document.body,'click').subscribe((e)=>{console.log(e)});

这两个都是异步获取元素的，差异在于前者主动请求，后者等待推送，Observable就是这两者概念的结合，陆续的我们会介绍一些概念

Observable: Observer,Subscription,Operator

Subject

Schedulers


## Observable 之 Observer

Observer是一个简单的概念，就是一个观察者，一个由回调函数组成的对象，一个属性{closed}，三个方法{next?,error?,complete?}，
Observer是可选的。在next、error 和 complete处理逻辑部分缺失的情况下，Observable仍然能正常运行，为包含的特定通知类型的处理逻辑会被自动忽略，当我们订阅的时候，最常用的就是next和error，例:

     var Subscription1=Observable.subscribe((res)=>{console.log(res)},(error)=>{console.log(error)})

## Observable 之 Subscription

Subscription是可以关闭当前订阅的，拿上面的Subscription1来说，我如果不想订阅的时候，我可以调用Subscription1.unsubscribe()来解除订阅，Subscription有两个比较实用的方法，一个是unsubscribe()，unsubscribe()上边已经解释了，另外一个是remove()，remove()可以移除嵌套在内的子subscriber

## Observable 之 Operator

操作符将会占用很大的篇幅来描述，Observable的操作符实在是太多了，仅介绍比较常用的，对于其他操作符有兴趣，可以直接看API

### 创建 Operator：create,of,from,fromPromise,fromEvent

     //create用于创建一个Observable对象，of用于同步请求，用来下面的observable1和observable2是等价的，就当前的情况，from也可以写出等价式子，
     //from接收的是一个iterator对象，甚至你可以传递一个字符串给他
     var observable1 = Observable.create((observer)=>{observer.next('A');observer.next('B')};)

     //创建同步Observable的最佳方式
     var observable2 = Observable.of('A','B');

     //神奇的from
     var observable3 = Observable.from(['A','B']);
     var observable4 = Observable.from('AB');
     var observable5 = Observable.from(new Promise((resolve, reject) => {resolve('AB')}));
     var observable5 = Observable.fromPromise(new Promise((resolve, reject) => {resolve('AB')}));
     var observable6 = Observable.fromEvent(document.body,'click').subscribe((e)=>{console.log(e)});
     observable1.subscribe((res)=>{console.log(res)});


### 创建 Operator：empty, never, throw,interval, timer

     //empty 明确的告诉你，什么事情都没有做
     var observable1 = Observable.empty();

     //never 订阅周期无穷，永远订阅不到
     var observable2 = Observable.never()

     //throw 手动抛出异常
     var observable3 = Observable.throw('Fail')

     //interval setInterval
     var observable4 = Observable.interval(1000)

     //timer setTimeOut
     var observable5 = Observable.timer(1000,?5000)


### 过滤 Operator：take,takeUntil,first,Last,takeLast

     //take 取前几个,比如最初的例子，我仅仅取一次点击事件
     var observable1 = Observable.fromEvent(document.body,'click').take(1).subscribe();

     //takeUntil 直到发生某些事情，才会终止当前的订阅行为
     var observable2 = Observable.interval(1000);
     var click = Observable.fromEvent(document.body,'click');
     observable2.takeUntil(click).subscribe();

     //first 与take表现相似，不过功能简单仅仅取第一个，take(1) == first()
     var observable3 = Observable.interval(1000);
     observable3.first().subscribe()

     //Last,takeLast获取最后一个和获取最后几个
     var observable4 = Rx.Observable.interval(1000).take(6);
     var observable5 = observable4.Last();
     var observable6 = observable4.takeLast(2);


### 过滤 Operator：skip,filter,map,mapTo,startWith

     //skip 与take相反，可以跳过几个
     var observable1 =  Observable.interval(1000).take(4).skip(2);

     //filter 过滤器，根据字意就可以知道，是一个功能强大的过滤器
     var observable2 = Observable.interval(1000).filter((x) => (x>3));

     //map 传参一个回调函数
     var observable3 = Observable.interval(1000).map((x) => (x++));

     //mapTo 将Observable的返回值置为特定值
     var observable4 = Observable.interval(1000).mapTo(20);

     //startWith 将指定Observable的初始值
     var observable5 = Observable.interval(1000).startWith(80000);


### 合并 Operator：concat,concatAll,merge,combineLatest,zip

     //concat,concatAll类似，用于顺序合并流,1-2-3顺序发生的顺序表现
     var observable1 = Observable.from('ABC');
     var observable2 = Observable.interval(500).take(2);
     var observable3 = Observable.of('K','J');
     Observable.concat(observable1,observable2,observable3);
     var source = Observable.of(observable1, observable2, observable3).concatAll().subscribe();

     //merge，用于合并并行的流，当有多个推送值时，哪个先来用哪个
     var observable4 = Observable.interval(500).mapTo(0);
     var observable5 = Observable.interval(500).mapTo(1);

     //combineLatest 合并两个流的每一次最新的数据，之后返回，表现形式比较特殊
     var observable6 = Observable.interval(500).take(3);
     var observable7 = Observable.interval(300).take(6);
     var example = observable6.combineLatest(observable7, (x, y) => x + y);

     //zip 多个流，取相同顺位的数据进行合并
     var observable8 = Observable.interval(500).take(3);
     var observable9 = Observable.interval(300).take(6);
     var example = observable8.zip(observable9, (x, y) => x + y);


### 合并 Operator：scan,buffer{buffer,bufferCount,bufferTime,bufferToggle,bufferWhen}

      //scan类似于原生Js的reduce方法，及其的相似
      var observable1 = Observable.from('rxjs').zip(Rx.Observable.interval(600), (x, y) => x);
      var observable2 = observable1.scan((origin, next) => origin + next, '');

      //buffer当触发某些事件的时候，一次性发出相关数据流
      var observable3 = Observable.interval(300);
      var observable4 = Observable.interval(1000);
      var observable5 = observable3.buffer(observable4);//每次observable4有数据，observable3之前累积的为一组
      var observable6 = observable3.bufferTime(1000);//一秒一组
      var observable7 = observable3.bufferCount(3);//三个一组
      var observable8 = Observable.fromEvent(document, 'click');
      var observable9 = observable8.bufferWhen(() =>Observable.interval(1000 + Math.random() * 4000));
      var observable10 = observable8.bufferToggle(openings, i =>*   i % 2 ? Observable.interval(500) : Observable.empty(););


### 优化 Operator：delay,delayWhen,throttle,debounce,distinct


### 优化 Operator：catch,retry,repeat,retryWhen


### 优化 Operator：mergeAll,switch,switchMap,concatMap,mergeMap


## Observable 之 Subject

首先Subject可以订阅Observable，所以代表他是一個Observer，同时Subject又可以被Observer订阅，代表他是一个Observable


### Subject Operator：Subject, BehaviorSubject, ReplaySubject, AsyncSubject

    var subject = new Rx.Subject();
    subject.subscribe(observerA);
    subject.subscribe(observerB);
    subject.next();
    //BehaviorSubject用于获取当前流的实时状态，它总是获取最新的
    var subject = new Rx.BehaviorSubject(0);
    subject.subscribe(observerA);
    setTimeout(()=>{subject.subscribe(observerB);},1000)
    //ReplaySubject用于订阅重复值,AsyncSubject用于订阅最后一个
    ReplaySubject(1) = BehaviorSubject(0);


## Observable 之 Scheduler


### Scheduler Operator：queue, asap, async, animationFrame














