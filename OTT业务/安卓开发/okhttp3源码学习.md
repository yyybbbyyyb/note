## 整体结构
![[okhttp3源码学习-1.png|697]]

+ OkHttpClient：这个是整个OkHttp的核心管理类，所有的内部逻辑和对象归OkHttpClient统一来管理，它通过Builder构造器生成，构造参数和类成员很多。作为用户使用的API，Client的构造器模式提供了许多的可配置选项，如最基础的connectTimeout、readTimeout，以及更深层次的connectionPool、dns、eventListener、cache等等（没列举完全）
+ Request和Response：Request是我们发送请求封装类，内部有url，header，method，body等常见的参数，Response是请求的结果，包含code，message，header，body，这两个类的定义是完全符合Http协议所定义的请求内容和响应内容。
+ Call：是一个接口，okhttp中实现为RealCall，负责请求的调度（同步的话走当前线程发送请求，异步的话则使用OkHttp内部的线程池进行）；同时负责构造内部逻辑责任链，并执行责任链相关的逻辑，直到获取结果。Callback为异步请求使用的回调API。
+ Interceptor：拦截器，采用责任链设计模式，okhttp提供五个默认的拦截器，分别处理重试重定向、处理请求头、缓存、建立连接、发送请求等功能，同时okhttp有较好的可拓展性与用户友好性，支持用户自定义应用拦截器和网络拦截器
+ Dispatcher：调度器，okhttp请求支持同步异步调用，异步则是在Dispatcher中使用线程池来进行处理，同时调度器控制最大并发请求数和每个主机最大并发数

## 拦截器

### 五个默认拦截器

+ RetryAndFollowUpInterceptor: Retry和FollowUp
	+ Retry是指：当网络有问题、出错、超时等情况出现而导致错误时，通过再发起一次请求来进行重试
	+ FollowUp是指：当需要授权、需要重定向、条件GET请求返回等情况发生时，需要接着进行一次请求。(StreamAllocation会在这里构造)。
+ BridgeInterceptor： 桥接拦截器。主要是在请求前后进行一些Header的处理（主要是Cookie的一些处理）。
+ CacheInterceptor： 负责缓存（磁盘缓存和Http缓存策略），核心在CacheStrategy当中。
+ ConnectInterceptor： 连接操作。
	+ 负责线路查找（Route查找，RouteSelector实现）、创建socket、握手、构建“输入/输出流”Source和Sink、并创建“解析器”Codec。
	+ 另一方面，借助于ConnectionPool，实现连接复用，即：连接前，从ConnectionPool中查询是否有可复用的Connection；如果没有，新建的Connection要放入Pool中以被使用。
+ CallServerInterceptor： 最后一层，进行Http流的读写和解析（也可以认为是序列化和反序列化）。借助于HttpCodec.writeRequestHeaders..等方法和Sink/Source实现。

### 拦截器的添加

拦截器的添加在okhttp3/RealCall.java的getResponseWithInterceptorChain方法中

```java
// 源码路径: okhttp3/RealCall.java
Response getResponseWithInterceptorChain() throws IOException {
	// 构建完整的拦截器列表
	List<Interceptor> interceptors = new ArrayList<>();
	
	// 1. 用户自定义的应用拦截器（最先执行）
	interceptors.addAll(client.interceptors());
	
	// 2. 内置核心拦截器
	interceptors.add(new RetryAndFollowUpInterceptor(client));
	interceptors.add(new BridgeInterceptor(client.cookieJar()));
	interceptors.add(new CacheInterceptor(client.internalCache()));
	interceptors.add(new ConnectInterceptor(client));
	
	// 3. 用户自定义的网络拦截器
	if (!forWebSocket) {
		interceptors.addAll(client.networkInterceptors());
	}
	
	// 4. 最终的网络请求拦截器
	interceptors.add(new CallServerInterceptor(forWebSocket));
	
	// 创建拦截器链
	Interceptor.Chain chain = new RealInterceptorChain(
		interceptors,
		transmitter,
		null,
		0,
		originalRequest);
	
	// 启动责任链
	return chain.proceed(originalRequest);
}
```

+ OkHttp把用户的拦截器放在默认拦截器之前
+ 一旦一个拦截器调用了chain.proceed方法，它之后的拦截器会依次被调用执行，其后它才能拿到被处理过的返回。
+ 用户的拦截器一般是在网络执行前对request进行操作、在网络执行后对response进行操作

### 自定义拦截器
![[okhttp3源码学习-2.png]]


## 请求管理

### 请求入口

+ 同步请求入口：RealCall.execute()
+ 异步请求入口：RealCall.enqueue()

Dispatcher在同步请求和异步请求中的不同作用，同步请求中Dispatcher不控制执行过程，只做状态跟踪，异步请求中Dispatcher实际执行由线程池调度

### Dispatcher

+ Dispatcher管理三个队列：
	+ runningAsyncCalls：正在执行的异步请求
	+ readyAsyncCalls：等待执行的异步请求
	+ runningSyncCalls：正在执行的同步请求

+ 默认并发策略：
	+ 最大并发请求数：64
	+ 每个主机最大并发数：5

+ Dispatcher中的核心方法
	+ executorService方法负责创建连接所需的线程池
	+ promoteAndExecute负责将等待队列中的请求判断是否可以转移到执行队列
	+ finished方法负责请求后检查是否还有请求，若无回调idleCallback

+ 线程池特点：
	+ 无核心线程，最大线程数无限，懒加载无边界限制的线程池
	+ 空闲线程60秒后回收
	+ 使用SynchronousQueue（每个任务必须有可用线程立即处理）
	+ 适合大量短生命周期的异步任务

## 连接复用和连接池

### 核心类

+ StreamAllocation
	+ newStream方法负责查找或创建一个可用的连接Connection，再从Connection中获得对应的HttpCodec负责request/response的序列化和反序列化
	+ findHealthyConnection负责查找/创建一个可用连接，这个方法会先查找连接池中是否具有可复用的连接（ConnectionPool.get()），如果有可以直接返回；如果没有，则需新建，包括线路查找（Route查找，RouteSelector实现）、创建socket、握手、构建“输入/输出流”Source和Sink等过程

+ ConnectionPool
	+ 连接池维护TCP连接，减少重复握手开销
		+ 最大空闲连接数：5
		+ 空闲连接存活时间：5分钟
		+ 自动清理过期连接
	+ 值得一提的是这个类中的get和put等私有方法采用门面模式被封装到的Internal类中
	+ Connection的自动回收由cleanupRunnable负责，大概流程如下：
		+ 每次put一个Connection时，都检查是否在运行cleanUpRunnable，如果没有运行就放入线程池中执行。这意味着只要ConnectionPool非空，这个runnable就在执行。
		+ 遍历connections，找出其中“没有被使用”的、且空闲的时间最长的Connection，检查它是否满足(空闲socket连接超过5个且keepalive时间大于5分钟)的条件，如果满足就清除、同时立刻重复该操作；如果不满足条件，就等待这个连接的到期时间，也就是过一会儿再进行清除操作；如果都是活跃连接，则5分钟后再次进行清理操作。
	+ 基于引用计数法的连接活跃判定：
		+ RealConnection中有个allocations成员变量，负责追踪引用着该Connection的StreamAllocation
		+ 当构建一个RealConnection时，这个allocations列表里添加当前的StreamAllocation的弱引用
		+ 当从ConnectionPool中取出一个Connection时，也在列表中添加当前的StreamAllocation的弱引用
		+ 当一个网络请求结束/出错/需要关闭时，则需要将弱引用清除
		+ 当这个列表为空时，也就是前面所说的“没有在被使用”，那么很快它就需要从连接池中被清除了
		+ 弱引用添加和删除的地方在StreamAllocaion的aquire和release方法中。

## 缓存策略

### 磁盘缓存

OkHttp的磁盘缓存主要是通过Cache实现的。主要使用的是DiskLruCache。不过稍微改造了一下：

+ 使用FileSystem类封装了一些File（文件/文件夹）的操作
+ 文件流的读写接入了Okio。

### Http缓存

Http的缓存策略主要是通过几个重要的头部实现的，大概包括这样几个步骤：

+ 客户端根据Response的Header判断是否过期
	+ Expire: 文件过期时间
	+ cache-control: 文件存续时间
+ 若已经过期，判断是否有缓存条件信息，如果有，进行条件GET查询，由服务端进一步决策是否过期
	+ Tag和If-None-Match
	+ Last-Modified和If-Modified-Since
+ 如果服务端判断未过期，还可以进一步使用，则返回304NotModified
+ 如果过期，正常请求

其他的缓存控制Header包括：
+ no-cache
+ only-if-cached

OkHttp中，关于缓存的实现类是CacheStrategy.Factory。主要是通过实现上述逻辑。在CacheInterceptor中，首先从Cache中获取已经缓存的response，然后将原始的请求request和cacheResponse传递给Factory，它会根据前述的策略（主要是头部解析和处理），判定缓存是否可用：如果可用，则将这里的cacheResponse封装在CacheStrategy中返回；如果不可用，需要请求，则对原始的request进行可能的修改（主要是条件get参数），作为networkRequest封装在CacheStrategy中返回。

## References

 + [https://github.com/MirrorCitrus/divetrace/blob/master/dive-open-source/网络库：OkHttp3源码阅读.md](https://github.com/MirrorCitrus/divetrace/blob/master/dive-open-source/网络库：OkHttp3源码阅读.md)
 + [第三章：Okhttp 拦截器链深度解析3.1 拦截器链整体架构 3.1.1 责任链模式在OkHttp中的应用 OkHtt - 掘金](https://juejin.cn/post/7527884547535896586)
 + [OkHttp源码深度解析OkHttp应该是目前Android平台上使用最为广泛的开源网络库了，Android 在6.0之 - 掘金](https://juejin.cn/post/6844904102669844493?from=search-suggest)