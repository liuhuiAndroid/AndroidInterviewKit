1. ⾸先是开发者使⽤ addInterceptor(Interceptor) 所设置的，它们会按照开发者的要求，在所有其他 Interceptor 处理之前，进⾏最早的预处理⼯作，以及在收到 Response 之后，做最后的善后⼯作。如果你有统⼀的 header 要添加，可以在这⾥设置；
2. 然后是 RetryAndFollowUpInterceptor ：它会对连接做⼀些初始化⼯作，并且负责在请求失败时的重试，以及重定向的⾃动后续请求。它的存在，可以让重试和重定向对于开发者是⽆感知的；
3. BridgeInterceptor ：它负责⼀些不影响开发者开发，但影响 HTTP 交互的⼀些额外预处理。例如，Content-Length 的计算和添加、gzip 的⽀持（Accept-Encoding: gzip）、gzip 压缩数据的解包，都是发⽣在这⾥；
4. CacheInterceptor ：它负责 Cache 的处理。把它放在后⾯的⽹络交互相关 Interceptor 的前⾯的好处是，如果本地有了可⽤的 Cache，⼀个请求可以在没有发⽣实质⽹络交互的情况下就返回缓存结果，⽽完全不需要开发者做出任何的额外⼯作，让 Cache 更加⽆感知；
5. ConnectInterceptor ：它负责建⽴连接。在这⾥，OkHttp 会创建出⽹络请求所需要的 TCP 连接（如果是 HTTP），或者是建⽴在 TCP 连接之上的 TLS 连接（如果是 HTTPS），并且会创建出对应的 HttpCodec 对象（⽤于编码解码 HTTP 请求）；
6. 然后是开发者使⽤ addNetworkInterceptor(Interceptor) 所设置的，它们的⾏为逻辑和使⽤ addInterceptor(Interceptor) 创建的⼀样，但由于位置不同，所以这⾥创建的 Interceptor 会看到每个请求和响应的数据（包括重定向以及重试的⼀些中间请求和响应），并且看到的是完整原始数据，⽽不是没有加 Content-Length 的请求数据，或者 Body 还没有被 gzip 解压的响应数据。多数情况，这个⽅法不需要被使⽤，不过如果你要做⽹络调试，可以⽤它；
7. CallServerInterceptor ：它负责实质的请求与响应的 I/O 操作，即往 Socket ⾥写⼊请求数据，和从 Socket ⾥读取响应数据。
