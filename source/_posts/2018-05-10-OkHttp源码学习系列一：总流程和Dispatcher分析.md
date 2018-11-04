---
title: OkHttp源码学习系列一：总流程和Dispatcher分析
date: 2018-05-10 21:20:29
categories:
 - Android
tags:
 - Android
 - OKHttp
 - 源码分析
---
> 本文为本人原创，转载请注明作者和出处。

> OkHttp可以说是目前Android开发中最流行的基础网络框架了。相信你也一定早已学会了它的基本用法，今天我们来进一步学习它的源码，了解其请求原理，学习它的代码设计思想。

系列文章索引：
[OkHttp源码学习系列一：总流程和Dispatcher分析](https://www.jianshu.com/p/425695e3ae03)
[OkHttp源码学习系列二：拦截链分析](https://www.jianshu.com/p/41a5d45085b4)
[OkHttp源码学习系列三：缓存总结](https://www.jianshu.com/p/8928f4b128f1)


###总体请求流程
```
OkHttpClient client = new OkHttpClient.Builder().build();
Request request = new Request.Builder().url("xxxx").build();
Call call = client.newCall(request);
call.enqueue(new Callback() {
    @Override
    public void onFailure(Call call, IOException e) {
        Log.d("fail");
    }

    @Override
    public void onResponse(Call call, Response response) throws IOException {
        Log.d("success");
    }
});
```
在分析之前，为了防止大家已经忘了Okhttp是怎么使用的（可能有很多人和我一样用了retrofit酒忘了），我们先来一段最简单的异步请求：依然是square公司执着使用的Builder设计模式创建出OkhttpClient和Request对象。之后把request对象放入client对象中new出一个call来。这个call是什么呢？
<!-- more -->

点进去看其源码注释：A call is a request that has been prepared for execution. A call can be canceled. As this object represents a single request/response pair (stream), it cannot be executed twice. Call是一个准备好执行的request，并且它可以被取消。这个对象就代表了一次网络请求中的请求/响应流。

这里我们使用的是enqueue方法将该call对象加入请求队列。该方法需要传入一个callback回调来执行response返回后的处理。那么，在enqueue或者同步请求execute方法之后Okhttp到底做了什么，让网络请求得以实现？不要着急，我们先来看下我画的大致流程图：

![Okhttp请求流程.png](https://upload-images.jianshu.io/upload_images/5586297-d1171c533068c051.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

大致讲解一下，在call执行了execute或enqueue方法后会将请求交给Dispatcher处理。Dispatcher将请求放入分配好的线程执行，执行的过程实际上就是图中五个拦截器组成的拦截链，拦截处理请求的过程。拦截器的内容我将放到下一篇去将，这篇将重点和大家一起捋清Dispatcher分发请求的过程。

回到之前的call，我们点进去看一下这个Call是一个接口，实现它的类就一个RealCall。来看一下RealCall执行enqueue/execute方法的源码：

```
@Override public Response execute() throws IOException {
    synchronized (this) {
      if (executed) throw new IllegalStateException("Already Executed");
      executed = true;
    }
    captureCallStackTrace();
    try {
      client.dispatcher().executed(this);
      Response result = getResponseWithInterceptorChain();
      if (result == null) throw new IOException("Canceled");
      return result;
    } finally {
      client.dispatcher().finished(this);
    }
}
      
@Override public void enqueue(Callback responseCallback) {
    synchronized (this) {
      if (executed) throw new IllegalStateException("Already Executed");
      executed = true;
    }
    captureCallStackTrace();
    client.dispatcher().enqueue(new AsyncCall(responseCallback));
}
```

###Dispatcher源码分析
可以看到无论是同步还是异步，最后调用到的都是dispatcher的executed和enqueue方法来执行。这里它每次执行前都进行了是否已执行判断，防止一次请求重复执行。另外一方面异步请求将传入的callback封装成了一个AsyncCall，AsyncCall实现了runnable接口，可以在线程池中执行。而同步方法正如我们之前所说，在分发器执行后交给拦截链来获得response。另外这个Dispatcher对象是在OkhttpClient对象创建好之后内部创建的一个对象，就不要问它是哪里来的了。继续看Dispatcher内的方法是怎么处理的：

```
synchronized void executed(RealCall call) {
    runningSyncCalls.add(call);
}
      
synchronized void enqueue(AsyncCall call) {
    if (runningAsyncCalls.size() < maxRequests && runningCallsForHost(call) < maxRequestsPerHost) {
      runningAsyncCalls.add(call);
      executorService().execute(call);
    } else {
      readyAsyncCalls.add(call);
    }
}
```

可以看到Dispatcher所做的就是根据不同情况将call放入三个不同的队列数组。这里就不画图了，源码解释的很清楚：

```
/** Ready async calls in the order they'll be run. */
    private final Deque<AsyncCall> readyAsyncCalls = new ArrayDeque<>();

/** Running asynchronous calls. Includes canceled calls that haven't finished yet. */
    private final Deque<AsyncCall> runningAsyncCalls = new ArrayDeque<>();

/** Running synchronous calls. Includes canceled calls that haven't finished yet. */
    private final Deque<RealCall> runningSyncCalls = new ArrayDeque<>();
```

对于同步请求的RealCall放入runningSyncCalls当中；对于异步请求的AsyncCall，如果当前满足可以执行的条件：runningAsyncCalls中的请求小于最大限制（默认64）、当前call的host已经在执行的call小于最大限制（默认5），则将AsyncCall放入runningAsyncCalls中，否则将其放入readyAsyncCalls中。至于为什么选择队列这种数据结构就不用我解释了吧，这里肯定是先添加的call先执行。

可以看到添加到runningSyncCalls和runningAsyncCalls的call都立即执行了线程池的execute方法，至于readyAsyncCalls中的call何时添加到runningAsyncCalls中并执行，直接看promoteCalls方法就可以了，该方法一目了然我就不再贴代码解释了。

往回看一点RealCall执行同步请求后，将call添加到runningSyncCalls中，再通过拦截链来执行请求，执行完之后调用dispatcher的finish方法。

再来看异步请求的AsyncCall，由于它本身实现了runnable，线程池执行了它我们直接看它的run方法就可以知道它执行的时候做了什么，而这里run方法执行了execute方法，我们来看一下：

```
@Override protected void execute() {
  boolean signalledCallback = false;
  try {
    Response response = getResponseWithInterceptorChain();
    if (retryAndFollowUpInterceptor.isCanceled()) {
      signalledCallback = true;
      responseCallback.onFailure(RealCall.this, new IOException("Canceled"));
    } else {
      signalledCallback = true;
      responseCallback.onResponse(RealCall.this, response);
    }
  } catch (IOException e) {
    if (signalledCallback) {
      // Do not signal the callback twice!
      Platform.get().log(INFO, "Callback failure for " + toLoggableString(), e);
    } else {
      responseCallback.onFailure(RealCall.this, e);
    }
  } finally {
    client.dispatcher().finished(this);
  }
}
```

总的流程和同步方法差不多，都是先调用拦截链获取response，最后调用dispatcher的finish方法。不同的是这里会根据请求的失败与否调用当初传入的callback的onFailure/onResponse方法。

讲到这里Dispatcher讲的差不多了，但是它里面调用频率挺高的finish方法我们还没讲。对于多线程执行，当然是要有始有终了，这很重要：

```
private <T> void finished(Deque<T> calls, T call, boolean promoteCalls) {
    int runningCallsCount;
    Runnable idleCallback;
    synchronized (this) {
      if (!calls.remove(call)) throw new AssertionError("Call wasn't in-flight!");
      if (promoteCalls) promoteCalls();
      runningCallsCount = runningCallsCount();
      idleCallback = this.idleCallback;
    }
    
    if (runningCallsCount == 0 && idleCallback != null) {
      idleCallback.run();
    }
}
```

由于finish方法如果是异步请求会在多线程中执行，为了线程安全加了同步锁。首先会将call从其所属的队列中移除。如果是异步请求，promoteCalls为true，则调用promoteCalls方法，该方法的作用上面已经讲过，就是在条件满足的情况下把readyAsyncCalls中的请求转移到runningAsyncCalls队列中。紧接着会调用runningCallsCount方法（square公司的程序员经常把变量名和方法名起的一样，咋一看真的懵逼-_-）重新计算一下当前正在执行的请求有多少个（就是runningAsyncCalls.size()+runningSyncCalls.size()）。最后如果正在执行的call数量为0，也就是我们没有正在执行的请求了，会调用idleCallback的run方法。这个idleCallback是什么东西呢？它是我们自己传入的一个回调，也就是每次okhttp把所有请求执行完了就会回调。目前为止我还并没有想到什么样的需求需要这样的一个callback，知道的小伙伴欢迎指教～～

最后总结一下，okhttp的分发器Dispatcher的作用就是分发同步/异步请求给线程池来执行，起一个总的调度作用。这样捋下来是不是很清晰很明了呢？下一章我讲继续和大家一起分析okhttp执行网络请求的核心代码——拦截链。
