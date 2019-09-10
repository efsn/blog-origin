---
title: how to reactor
date: 2019-07-22 22:38:44
tags:
- functional
- reactive
categories:
---
# How to Reactor
> What's Reactive
>
> Why's Reactive
>
>How to reactive

## What
> 基于事件驱动/订阅模型
Servlet3.0之前，线程会Block到业务处理完并返回后结束Servlet（单线程）
Servlet3.0规范中一个新特性：异步处理支持；在接收到请求之后，Servlet线程可以将耗时的task委派给另一个线程来完成，在不生成响应的情况下返回至容器（Master/Worker模型）

在传统的编程范式中，一般通过`Iterator`模式来遍历一个序列，这种遍历方式是由调用者来控制节奏的，采用`pull`方式。每次通过`next()`方法来获取序列中的下一个值。

**Reactive Stream**采用`push`方式，当publisher有新的数据产生时，这些数据会被`push`到*subcriber*来进行处理；在Stream中可以添加各种不同的操作来处理数据，形成数据链。这种以声明式添加的处理链只有在订阅操作时才会真正执行。

到一个重要的概念是`backpressure`，在基本的消息推送模式中，当消息发布者产生数据的速度过快时，会使得消息订阅的处理速度无法跟上产生的速度，从而给订阅者造成很大的压力。当压力过大时，有可能造成订阅者本身的奔溃，所产生的及联效应甚至
可能造成整个系统的瘫痪。`backpressure`的作用在于提供一种从订阅到生产者的反馈渠道。订阅者可以通过`request()`方法来声明其一次所能处理的消息`request()`方法调用

//TODO 反应式流规范？？？
Reactor完全基于反应式流规范设计和实现的库，Reactor也是Spring5中反应式编程的基础。

<!-- more -->

### Mono & Flux
> Reactor中的两个基本概念
>
> 核心组件

Flux表示0～N个元素的异步序列，该序列可以包含三种不同类型的消息通知
- 正常包含元素的消息
- 序列结束的消息
- 序列出错的消息

当消息通知产生时，订阅者对应的方法会被调用。
- onNext
- onComplete
- onError

Mono表示0～1个元素的异步序列。该序列中同样包含与Flux相同的三种类型的消息通知。Flux和Mono之间可以进行转换？？？
对一个Flux序列进行count操作，得到的结果是一个Mono<Long>对象，把两个Mono序列合并在一起，得到一个Flux对象？？？

有多种方式可以创建Flux序列
- just()，可以指定序列中包含的全部元素，创建出来的Flux序列在发布这些元素之后会自动结束。
- fromArray fromIterable fromStream
- empty()，创建一个不包含任何元素，只发布结束消息的序列
- error，创建一个只包含错误消息的序列
- never，创建一个不包含任何消息通知的序列
- range
- interval(Duration)，创建一个包含了从0开始递增的Long对象的序列，其中包含的元素按照指定的间隔来发布。除了间隔时间之外，还可以指定起始元素发布之前的延迟时间
- intervalMillis，与interval的作用相同，只不过指定来时间单位毫秒

```kotlin
Flux.just(1,2,3).subcribe{println(it)}
Flux.fromArray(arrayOf(1,2,3)).subcribe{println(it)}
Flux.empty().subcribe{println(it)}
Flux.error(RuntimeException("Error")).subcribe{println(it)}
Flux.range(10,10).subcribe{println(it)}
Flux.interval(Duration.of(10,ChronoUnit.SECONDS)).subcribe{println(it)}
Flux.intervalMillis(3000).subcribe{println(it)}
```
以上只适合简单的序列生成，当逻辑复杂时，则应该使用generate或者create

generate通过同步和逐一的方式来产生Flux，序列的产生是通过调用所提供的SynchronousSink对象的next、complete、error方法来完成的。
逐一生成的含义是在具体的生成逻辑中，next方法只能最多被调用一次。在某些情况下，序列的生成可能是有状态的，需要用到某些状态对象。此时可以使用generate方法的另一种形式
generate(Callable<S> stateSupplier, BiFunction<S,SynchronousSink<T, S>> generator)，其中stateSupplier用来提供初始的状态对象。在进行序列生成时，状态对象会作为generate使用的第一个参数传入，
可以在对应的逻辑中对该状态进修改以供下一次生成时使用。

在如下示例中，第一个序列的生成逻辑中通过next()方法产生一个简单的值，然后通过complete()方法来结束该序列。
如果不调用complete()方法，所产生的是一个无限序列。第二个序列的生成逻辑中的状态对象是一个ArrayList对象。实际产生的值是一个随机数。产生的随机数被添加到ArrayList中。
当产生了10个数时，通过complete()方法来结束序列。

```kotlin
Flux.generate{sink ->  
    sink.next("hello")
    sink.complete() //如果不complete则生成一个无限序列
}

Flux.generate({ mutableListOf() }){list, sink ->
    val x = 1
    list.add(x)
    sink.next(x)
    if(list.size == 5){
        sink.complete()
    }
    return list
}.subcribe{println(it)}

```

create()






（目前推荐Kotlin的std.lib??）
```java
T.toFlux() | T.toMono()
```









Reactor是JVM的NIO编程基础，具有高效的需求管理。直接与Java8的CompletableFuture、Stream、Duration。

### 8个比较维度
|维度|参考|
|---|---|
|Composable|CompletableFuture|
|Lazy|Stream|
|Reusable|Optional|
|Asynchronous|Obervable(RxJava v1)|
|Cacheable|Observable(RxJava v2)|
|Push or Pull|Flowable(RxJava v2)|
|Backpressure|Flux(Reactor Core|
|Operator fusion|--|

### WebFlux & WebMvc

### Usage
- 异步响应
- 流式响应
- 背压
- websocket支持


### How to test with springboot
> mock & integretion ??
