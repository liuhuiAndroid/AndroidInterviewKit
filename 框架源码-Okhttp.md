### OkHttp的简单使用

```kotlin
// 创建一个 OkHttp 的实例
val client: OkHttpClient = OkHttpClient.Builder()
  .addInterceptor(HttpLoggingInterceptor())
  .cache(Cache(cacheDir, cacheSize))
	.dispatcher(dispatcher)
  .build()
// 创建 Request
val request: Request = Request.Builder().url("http://hencoder.com").build()
// 创建 Call 并发起网络请求
client.newCall(request).enqueue(object : Callback {
  override fun onFailure(call: Call, e: IOException) {}

  override fun onResponse(call: Call, response: Response) {}
})
```

###  OkHttp源码分析

**源码分析流程**

- Dispatcher：分发器就是来调配请求任务的，内部会包含一个线程池。可以在创建 OkHttpClient 时，传递我们自己定义的线程池来创建分发器。

  ```kotlin
  // 异步请求同时存在的最大请求数
  var maxRequests = 64
  // 异步请求同一域名同时存在的最大请求数
  var maxRequestsPerHost = 5
  // 闲置任务（没有请求时可执行一些任务，由使用者设置）
  var idleCallback: Runnable? = null
  // 异步请求使用的线程池
  private var executorServiceOrNull: ExecutorService? = null
  // 异步请求等待执行队列
  private val readyAsyncCalls = ArrayDeque<AsyncCall>()
  // 异步请求正在执行队列
  private val runningAsyncCalls = ArrayDeque<AsyncCall>()
  // 同步请求正在执行队列
  private val runningSyncCalls = ArrayDeque<RealCall>()
  ```

- 同步请求

  ```kotlin
    @Synchronized internal fun executed(call: RealCall) {
      runningSyncCalls.add(call)
    }
  ```

  因为同步请求不需要线程池，也不存在任何限制，所以分发器仅做一下记录。

- 异步请求

  ```kotlin
    internal fun enqueue(call: AsyncCall) {
      synchronized(this) {
        readyAsyncCalls.add(call)
        if (!call.call.forWebSocket) {
          val existingCall = findExistingCallWithHost(call.host)
          if (existingCall != null) call.reuseCallsPerHostFrom(existingCall)
        }
      }
      promoteAndExecute()
    }
  
    private fun promoteAndExecute(): Boolean {
      val executableCalls = mutableListOf<AsyncCall>()
      val isRunning: Boolean
      synchronized(this) {
        val i = readyAsyncCalls.iterator()
        while (i.hasNext()) {
          val asyncCall = i.next()
          if (runningAsyncCalls.size >= this.maxRequests) break
          if (asyncCall.callsPerHost.get() >= this.maxRequestsPerHost) continue
          i.remove()
          asyncCall.callsPerHost.incrementAndGet()
          executableCalls.add(asyncCall)
          runningAsyncCalls.add(asyncCall)
        }
        isRunning = runningCallsCount() > 0
      }
  ```

  先加入等待队列，当正在执行的任务未超过最大限制64，同时同一Host的请求不超过5个，则会添加到正在执行队列，同时提交给线程池。

  加入线程池直接执行没啥好说的，但是如果加入等待队列后，就需要等待有空闲名额才开始执行。因此每次执行完 一个请求后，都会调用分发器的 finished 方法。

  ```kotlin
    // 异步请求调用
    internal fun finished(call: AsyncCall) {
      call.callsPerHost.decrementAndGet()
      finished(runningAsyncCalls, call)
    }
  	// 同步请求调用
    internal fun finished(call: RealCall) {
      finished(runningSyncCalls, call)
    }
    private fun <T> finished(calls: Deque<T>, call: T) {
      val idleCallback: Runnable?
      synchronized(this) {
        // 不管异步还是同步，执行完后都要从队列移除(runningSyncCalls/runningAsyncCalls)
        if (!calls.remove(call)) throw AssertionError("Call wasn't in-flight!")
        idleCallback = this.idleCallback
      }
      // 重新调配请求
      val isRunning = promoteAndExecute()
      // 没有任务执行执行闲置任务
      if (!isRunning && idleCallback != null) {
        idleCallback.run()
      }
    }
  ```

**请求流程**



**分发器线程池**

```kotlin
val executorService: ExecutorService
    get() {
      if (executorServiceOrNull == null) {
        executorServiceOrNull = ThreadPoolExecutor(
          0,  									// 核心线程
          Int.MAX_VALUE,				// 最大线程
          60, 									// 空闲线程闲置时间
          TimeUnit.SECONDS,			// 闲置时间单位
          SynchronousQueue(),		// 线程等待队列
          threadFactory("$okHttpName Dispatcher", false)	// 线程创建工厂
        )
      }
      return executorServiceOrNull!!
    }
```

在OkHttp的分发器中的线程池定义如上，其实就和 Executors.newCachedThreadPool() 创建的线程一样。首先核心线程为0，表示线程池不会一直为我们缓存线程，线程池中所有线程都是在60s内没有工作就会被回收。而最大线程 Integer.MAX_VALUE 与等待队列 SynchronousQueue 的组合能够得到最大的吞吐量。即当需要线程池执行任务时，如果不存在空闲线程不需要等待，马上新建线程执行任务。等待队列的不同指定了线程池的不同排队机制。一 般来说，等待队列 BlockingQueue 有: ArrayBlockingQueue 、 LinkedBlockingQueue 与 SynchronousQueue 。

假设向线程池提交任务时，核心线程都被占用的情况下：

ArrayBlockingQueue：基于数组的阻塞队列，初始化需要指定固定大小。

当使用此队列时，向线程池提交任务，会首先加入到等待队列中，当等待队列满了之后，再次提交任务，尝试加入
队列就会失败，这时就会检查如果当前线程池中的线程数未达到最大线程，则会新建线程执行新提交的任务。所以
最终可能出现后提交的任务先执行，而先提交的任务一直在等待。

LinkedBlockingQueue：基于链表实现的阻塞队列，初始化可以指定大小，也可以不指定。

当指定大小后，行为就和 ArrayBlockingQueu 一致。而如果未指定大小，则会使用默认的 Integer.MAX_VALUE 作 为队列大小。这时候就会出现线程池的最大线程数参数无用，因为无论如何，向线程池提交任务加入等待队列都会成功。最终意味着所有任务都是在核心线程执行。如果核心线程一直被占，那就一直等待。

SynchronousQueue：无容量的队列。

使用此队列意味着希望获得最大并发量。因为无论如何，向线程池提交任务，往队列提交任务都会失败。而失败后 如果没有空闲的非核心线程，就会检查如果当前线程池中的线程数未达到最大线程，则会新建线程执行新提交的任 务。完全没有任何等待，唯一制约它的就是最大线程数的个数。因此一般配合 Integer.MAX_VALUE 就实现了真正的无等待。

但是需要注意的是，我们都知道，进程的内存是存在限制的，而每一个线程都需要分配一定的内存。所以线程并不 能无限个数。那么当设置最大线程数为 Integer.MAX_VALUE 时，OkHttp同时还有最大请求任务执行个数: 64的限 制。这样即解决了这个问题同时也能获得最大吞吐。

**RetryAndFollowUpInterceptor**

重试：

```kotlin
catch (e: RouteException) {
  // The attempt to connect via a route failed. The request will not have been sent.
  // 路由异常，连接未成功，请求还没发出去
  if (!recover(e.lastConnectException, call, request, requestSendStarted = false)) {
    throw e.firstConnectException.withSuppressed(recoveredFailures)
  } else {
    recoveredFailures += e.firstConnectException
  }
  newExchangeFinder = false
  continue
} catch (e: IOException) {
  // An attempt to communicate with a server failed. The request may have been sent.
  // 请求发出去了，但是和服务器通信失败了。
  if (!recover(e, call, request, requestSendStarted = e !is ConnectionShutdownException)) {
    throw e.withSuppressed(recoveredFailures)
  } else {
    recoveredFailures += e
  }
  newExchangeFinder = false
  continue
}
```

两个异常都是根据 recover 方法判断是否能够进行重试，如果返回 true ，则表示允许重试。

```kotlin
  private fun recover(
    e: IOException,
    call: RealCall,
    userRequest: Request,
    requestSendStarted: Boolean
  ): Boolean {
    // The application layer has forbidden retries.
    // 配置OkhttpClient设置了不允许重试(默认允许)，则一旦发生请求失败就不再重试
    if (!client.retryOnConnectionFailure) return false
    // We can't send the request body again.
    if (requestSendStarted && requestIsOneShot(e, userRequest)) return false
    // This exception is fatal.
    // 判断是不是属于重试的异常
    if (!isRecoverable(e, requestSendStarted)) return false
    // No more routes to attempt.
    if (!call.retryAfterFailure()) return false
    // For failure recovery, use the same route selector with a new connection.
    return true
  }
```

所以首先使用者在不禁止重试的前提下，如果出现了某些异常，并且存在更多的路由线路，则会尝试换条线路进行 请求的重试。其中某些异常是在 isRecoverable 中进行判断：

经过了异常的判定之后，如果仍然允许进行重试，就会再检查当前有没有可用路由路线来进行连接。简单来说，比 如 DNS 对域名解析后可能会返回多个 IP，在一个IP失败后，尝试另一个IP进行重试。

重定向：

如果请求结束后没有发生异常并不代表当前获得的响应就是最终需要交给用户的，还需要进一步来判断是否需要重 定向的判断。重定向的判断位于 followUpRequest 方法。

```kotlin
@Throws(IOException::class)
  private fun followUpRequest(userResponse: Response, exchange: Exchange?): Request? {
    val route = exchange?.connection?.route()
    val responseCode = userResponse.code
    val method = userResponse.request.method
    when (responseCode) {
      // 407 客户端使用了HTTP代理服务器，在请求头中添加 “Proxy-Authorization“，让代理服务器授权
      HTTP_PROXY_AUTH -> {
        val selectedProxy = route!!.proxy
        if (selectedProxy.type() != Proxy.Type.HTTP) {
          throw ProtocolException("Received HTTP_PROXY_AUTH (407) code while not using proxy")
        }
        return client.proxyAuthenticator.authenticate(route, userResponse)
      }
			// 401 需要身份验证 有些服务器接口需要验证使用者身份 在请求头中添加 “Authorization”
      HTTP_UNAUTHORIZED -> return client.authenticator.authenticate(route, userResponse)

      // 308 永久重定向
      HTTP_PERM_REDIRECT, 
      // 307 临时重定向
      HTTP_TEMP_REDIRECT, HTTP_MULT_CHOICE, HTTP_MOVED_PERM, HTTP_MOVED_TEMP, HTTP_SEE_OTHER -> {
        return buildRedirectRequest(userResponse, method)
      }
			// 408 客户端请求超时
      HTTP_CLIENT_TIMEOUT -> {
        // 408's are rare in practice, but some servers like HAProxy use this response code. The
        // spec says that we may repeat the request without modifications. Modern browsers also
        // repeat the request (even non-idempotent ones.)
        // 408 算是连接失败了，所以判断用户是不是允许重试
        if (!client.retryOnConnectionFailure) {
          // The application layer has directed us not to retry the request.
          return null
        }

        val requestBody = userResponse.request.body
        if (requestBody != null && requestBody.isOneShot()) {
          return null
        }
        val priorResponse = userResponse.priorResponse
        if (priorResponse != null && priorResponse.code == HTTP_CLIENT_TIMEOUT) {
          // We attempted to retry and got another timeout. Give up.
          return null
        }

        if (retryAfter(userResponse, 0) > 0) {
          return null
        }

        return userResponse.request
      }
			// 503 服务不可用 和408差不多，但是只在服务器告诉你 Retry-After:0(意思就是立即重试) 才重请求
      HTTP_UNAVAILABLE -> {
        val priorResponse = userResponse.priorResponse
        if (priorResponse != null && priorResponse.code == HTTP_UNAVAILABLE) {
          // We attempted to retry and got another timeout. Give up.
          return null
        }

        if (retryAfter(userResponse, Integer.MAX_VALUE) == 0) {
          // specifically received an instruction to retry without delay
          return userResponse.request
        }

        return null
      }

      HTTP_MISDIRECTED_REQUEST -> {
        // OkHttp can coalesce HTTP/2 connections even if the domain names are different. See
        // RealConnection.isEligible(). If we attempted this and the server returned HTTP 421, then
        // we can retry on a different connection.
        val requestBody = userResponse.request.body
        if (requestBody != null && requestBody.isOneShot()) {
          return null
        }

        if (exchange == null || !exchange.isCoalescedConnection) {
          return null
        }

        exchange.connection.noCoalescedConnections()
        return userResponse.request
      }

      else -> return null
    }
  }
```

整个是否需要重定向的判断内容很多，记不住，这很正常，关键在于理解他们的意思。如果此方法返回空，那就表 示不需要再重定向了，直接返回响应;但是如果返回非空，那就要重新请求返回的 Request ，但是需要注意的是， 我们的 followup 在拦截器中定义的最大次数为20次。

**BridgeInterceptor**

**CacheInterceptor**

**ConnectInterceptor**

**CallServerInterceptor**

**源码分析总结**

- OkHttpClient 相当于配置中⼼，所有的请求都会共享这些配置(例如出错是否重试、共享的连接池)。 OkHttpClient 中的配置主要有:

  1. Dispatcher dispatcher：调度器，⽤于调度后台发起的⽹络请求，有后台总请求数和单主机总请求数的控制。

  2. List<Protocol> protocols：⽀持的应⽤用层协议，即 HTTP/1.1、HTTP/2 等。

  3. List<ConnectionSpec> connectionSpecs：应⽤层⽀持的 Socket 设置，即使⽤明⽂传输(⽤于HTTP)还是某个版本的TLS(⽤于HTTPS)。

  4. List<Interceptor> interceptors：⼤多数时候使用的 Interceptor 都应该配置到这⾥。

  5. List<Interceptor> networkInterceptors：直接和⽹络请求交互的 Interceptor 配

     置到这⾥，例如如果你想查看返回的 301 报⽂或者未解压的 Response Body，需要在这⾥看。

  6. CookieJar cookieJar：管理 Cookie 的控制器。OkHttp 提供了 Cookie 存取的判断⽀持(即什么时候需要存 Cookie，什么时候需要读取 Cookie，但没有给出具体的存取实现。 如果需要存取 Cookie，你得⾃己写实现，例如用 Map 存在内存⾥，或者⽤别的方式存在本地存储或者数据库。

  7. Cache cache：Cache 存储的配置。默认是没有，如果需要用，得⾃己配置出 Cache 存储的⽂件位置以及存储空间上限。

  8. HostnameVerifier hostnameVerifier：⽤于验证 HTTPS 握手过程中下载到的证书所属者是否和⾃己要访问的主机名一致。

  9. CertificatePinner certificatePinner：⽤于设置 HTTPS 握⼿过程中针对某个 Host 的 Certificate Public Key Pinner，即把⽹站证书链中的每一个证书公钥直接拿来提前配置进 OkHttpClient 里去，以跳过本地根证书，直接从代码⾥进⾏认证。这种用法⽐较少见，⼀般⽤于防⽌⽹站证书被人仿制。

  10. Authenticator authenticator：⽤于⾃动重新认证。配置之后，在请求收到 401 状态码的响应是，会直接调用 authenticator ，⼿动加入 Authorization header 之后自动重新发起请求。

  11. boolean followRedirects：遇到重定向的要求是，是否自动 follow。

  12. boolean followSslRedirects：在重定向时，如果原先请求的是 http 而重定向的目标是 https，或者原先请求的是 https ⽽重定向的⽬标是 http，是否依然自动 follow。(记得， 不是「是否自动 follow HTTPS URL 重定向的意思，⽽是是否自动 follow 在 HTTP 和 HTTPS 之间切换的重定向)

  13. boolean retryOnConnectionFailure：在请求失败的时候是否自动重试。注意，⼤多数的请求失败并不属于 OkHttp 所定义的「需要重试」，这种重试只适用于「同一个域名的多个 IP 切换重试」「Socket 失效重试」等情况。

  14. int connectTimeout：建立连接(TCP 或 TLS)的超时时间。

  15. int readTimeout：发起请求到读到响应数据的超时时间。

  16. int writeTimeout：发起请求并被目标服务器接受的超时时间。(为什么?因为有时候对方服务器可能由于某种原因⽽不读取你的 Request)

- newCall(Request) ⽅法会返回一个 RealCall 对象，它是 Call 接⼝的实现。当调⽤ RealCall.execute() 的时候， RealCall.getResponseWithInterceptorChain() 会被调用，它会发起⽹络请求并拿到返回的响应，装进⼀一个 Response 对象并作为返回值返回; RealCall.enqueue() 被调用的时候大同小异，区别在于 enqueue() 会使⽤用 Dispatcher 的线程池来把请求放在后台线程进⾏，但实质上使用的同样也是 getResponseWithInterceptorChain() ⽅法。

- getResponseWithInterceptorChain() ⽅法做的事：把所有配置好的 Interceptor 放在 一个 List ⾥，然后作为参数，创建一个 RealInterceptorChain 对象，并调⽤ chain.proceed(request) 来发起请求和获取响应。

- 在 RealInterceptorChain 中，多个 Interceptor 会依次调用⾃己的 intercept() 方法。这个⽅法会做三件事：

  - 对请求进行预处理

  - 预处理之后，重新调⽤ RealInterceptorChain.proceed() 把请求交给下一个 Interceptor

  - 在下一个 Interceptor 处理完成并返回之后，拿到 Response 进行后续处理

    当然了，最后⼀个 Interceptor 的任务只有一个：做真正的网络请求并拿到响应

- 从上到下，每级 Interceptor 做的事:

  1. ⾸先是开发者使用 addInterceptor(Interceptor) 所设置的 Interceptor，它们会按照开发者的要求，在所有其他 Interceptor 处理之前，进⾏最早的预处理工作，以及在收到 Response 之后，做最后的善后工作。如果你有统⼀的 header 要添加，可以在这里设置;
  2. 然后是 RetryAndFollowUpInterceptor：它负责在请求失败时的重试，以及重定向的自动后续请求。它的存在，可以让重试和重定向对于开发者是⽆感知的;
  3. BridgeInterceptor：它负责一些不影响开发者开发，但影响 HTTP 交互的一些额外预处理。例如，Content-Length 的计算和添加、gzip 的⽀持(Accept-Encoding: gzip)、 gzip 压缩数据的解包，都是发⽣在这里;
  4. CacheInterceptor：它负责 Cache 的处理。把它放在后⾯的⽹络交互相关 Interceptor 的前⾯的好处是，如果本地有了可⽤的 Cache，⼀个请求可以在没有发⽣实质⽹络交互的情况下就返回缓存结果，⽽完全不需要开发者做出任何的额外⼯作，让 Cache 更加⽆感知;
  5. ConnectInterceptor：它负责建⽴连接。在这⾥，OkHttp 会创建出⽹络请求所需要的 TCP 连接(如果是 HTTP)，或者是建立在 TCP 连接之上的 TLS 连接(如果是 HTTPS)， 并且会创建出对应的 HttpCodec 对象(⽤于编码解码 HTTP 请求);
  6. 然后是开发者使⽤ addNetworkInterceptor(Interceptor) 所设置的，它们的行为逻辑和使⽤ addInterceptor(Interceptor) 创建的⼀样，但由于位置不同，所以这⾥创建的 Interceptor 会看到每个请求和响应的数据(包括重定向以及重试的⼀些中间请求和响应)，并且看到的是完整原始数据，⽽不是没有加 Content-Length 的请求数据，或者 Body 还没有被 gzip 解压的响应数据。多数情况，这个方法不需要被使用;
  7. CallServerInterceptor：它负责实质的请求与响应的 I/O 操作，即往 Socket 里写入请求数据，和从 Socket ⾥读取响应数据。

### OkHttp 五大拦截器

1. ⾸先是开发者使⽤ addInterceptor(Interceptor) 所设置的，它们会按照开发者的要求，在所有其他 Interceptor 处理之前，进⾏最早的预处理⼯作，以及在收到 Response 之后，做最后的善后⼯作。如果你有统⼀的 header 要添加，可以在这⾥设置；
2. 然后是 RetryAndFollowUpInterceptor ：它会对连接做⼀些初始化⼯作，并且负责在请求失败时的重试，以及重定向的⾃动后续请求。它的存在，可以让重试和重定向对于开发者是⽆感知的；
3. BridgeInterceptor ：它负责⼀些不影响开发者开发，但影响 HTTP 交互的⼀些额外预处理。例如，Content-Length 的计算和添加、gzip 的⽀持（Accept-Encoding: gzip）、gzip 压缩数据的解包，都是发⽣在这⾥；
4. CacheInterceptor ：它负责 Cache 的处理。把它放在后⾯的⽹络交互相关 Interceptor 的前⾯的好处是，如果本地有了可⽤的 Cache，⼀个请求可以在没有发⽣实质⽹络交互的情况下就返回缓存结果，⽽完全不需要开发者做出任何的额外⼯作，让 Cache 更加⽆感知；
5. ConnectInterceptor ：它负责建⽴连接。在这⾥，OkHttp 会创建出⽹络请求所需要的 TCP 连接（如果是 HTTP），或者是建⽴在 TCP 连接之上的 TLS 连接（如果是 HTTPS），并且会创建出对应的 HttpCodec 对象（⽤于编码解码 HTTP 请求）；
6. 然后是开发者使⽤ addNetworkInterceptor(Interceptor) 所设置的，它们的⾏为逻辑和使⽤ addInterceptor(Interceptor) 创建的⼀样，但由于位置不同，所以这⾥创建的 Interceptor 会看到每个请求和响应的数据（包括重定向以及重试的⼀些中间请求和响应），并且看到的是完整原始数据，⽽不是没有加 Content-Length 的请求数据，或者 Body 还没有被 gzip 解压的响应数据。多数情况，这个⽅法不需要被使⽤，不过如果你要做⽹络调试，可以⽤它；
7. CallServerInterceptor ：它负责实质的请求与响应的 I/O 操作，即往 Socket ⾥写⼊请求数据，和从 Socket ⾥读取响应数据。



#### Http请求过程

1. 请求到达运营商的DNS服务器并解析成对应的IP地址

   可能遇到的问题：DNS被劫持、DNS解析慢

   解决方案：使用HttpDNS，绕过运营商域名解析过程；使用阿里云提供的DNS解析服务

2. 创建连接，根据IP地址找到相应的服务器，发起一个请求

3. 服务器找到对应的资源原路返回访问给用户



#### 弱网请求优化

1. 设置连接超时和读取超时

   在进行网络请求时，设置合理的连接超时和读取超时时间。这可以防止应用在弱网络条件下等待时间过长而导致性能问题。

2. 使用适当的网络请求库

   选择一个适合弱网络环境的网络请求库。一些流行的库，如OkHttp和Retrofit，提供了对请求超时、重试策略等方面的灵活控制。

3. 启用请求重试机制

   在网络不稳定的情况下，启用请求重试机制有助于应对临时性的连接问题。但要小心设置最大重试次数，以避免对服务器造成不必要的负担。

4. 启用缓存

   使用HTTP缓存来存储先前的响应，以减少对网络的依赖。在OkHttp中，可以通过配置Cache来启用缓存。

5. 压缩请求和响应

   启用Gzip等压缩算法，减小请求和响应的大小，从而降低网络传输的时间。

6. 网络状态监听

   监听网络状态变化，根据当前网络状态调整应用行为，例如在网络恢复时重新发起之前失败的请求。
