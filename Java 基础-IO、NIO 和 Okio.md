# Java I/O 、NIO 和 Okio

[为什么我觉得 Java 的 IO 很复杂？](https://www.zhihu.com/question/67535292)

#### Java IO

- I/O 是什么？

  - 程序内部和外部进行数据交互的过程，就叫输入输出
    - 程序内部是谁？内存
    - 程序外部是谁？
      - 一般来说有两类：本地文件和网络
      - 也有别的情况，比如你和别的程序做交互，和你交互的程序也属于外部，但一般来说，就是文件和网络这么两种
    - 从文件里或者从网络上读数据到内存里，就叫输入
    - 从内存里写到文件里或者发送到网络上，就叫输出
  - Java I/O的作用只有一个：和外界做数据交互

- I/O 用法

  - 使用流：例如 FileInputStream / FileOutputStream

  - 可以用 Reader 和 Writer 来对字符进行读写

  - 流的外面还可以套别的流，层层嵌套都可以

  - BufferedXXX 可以给流加上缓冲。对于输入流，是每次多读一些放在内存里面，下次再取数据就不用再和外部做交互（即不必做 I/O 操作）；对于输出流，是把数据先在内存里面攒一下，攒够一波了再往外部去写。

    通过缓存的方式减少和外部的交互，从而可以提高效率。

  - 文件的关闭：close()

  - 需要用到的写过的数据，flush() 一下可以保证数据真正写到外部去（读数据没有这样的担忧）

  - Java I/O 的原理就是内存和外界的交互，涉及的类非常多，具体等用的时候再关注

  - 使用 Socket 和 ServerSocket 进行网络交互

    ```java
    try {
        ServerSocket serverSocket = new ServerSocket(8080);
        Socket socket = serverSocket.accept();
        BufferedReader reader = new BufferedReader(new InputStreamReader(socket.getInputStream()));
        BufferedWriter writer = new BufferedWriter(new OutputStreamWriter(socket.getOutputStream()));
        String data;
        while (true) {
            data = reader.readLine();
            writer.write("data: " + data);
            writer.write("\n");
            writer.flush();
        }
    } catch (IOException e) {
        e.printStackTrace();
    }
    ```

#### NIO

- NIO 和 IO 的区别：

  - 传统 IO 用的是插管道的方式，用的是 Stream

    NIO 用的也是插管道的方式，用的是 Channel，NIO 的 Channel 是双向的

  - NIO 也用到 Buffer

    - 它的 Buffer 可以被操作
    - 它强制使用 Buffer
    - 它的 Buffer 不好用

  - NIO 有非阻塞式的支持

    - 只是支持非阻塞式，而不是全是非阻塞式。默认是阻塞式的
    - 而且就算是非阻塞式，也只是网络交互支持，文件交互式不支持的

- 使用

  - NIO 的 Buffer 模型

  - 用 NIO 来读文件的写法

    ```java
    try {
        RandomAccessFile file = new RandomAccessFile("./19_io/text.txt", "r");
        // 使用 file.getChannel() 获取到 Channel
        FileChannel channel = file.getChannel();
        // 然后创建一个 Buffer
        ByteBuffer byteBuffer = ByteBuffer.allocate(1024);
        // 把文件内容读入 channel
        channel.read(byteBuffer);
        // 读完以后，用 flip() 翻页
        byteBuffer.flip();
        // 开始使用 Buffer
        System.out.println(Charset.defaultCharset().decode(byteBuffer));
        // 使用结束之后记得 clear
        byteBuffer.clear();
    } catch (FileNotFoundException e) {
    	e.printStackTrace();
    } catch (IOException e) {
    	e.printStackTrace();
    }
    ```

#### Okio

- Okio：A modern I/O library for Android, Kotlin, and Java.

  适用于 Android，Kotlin 和 Java 的现代 I / O 库。

- 官网：https://github.com/square/okio

- Okio 基础使用部分：https://www.jianshu.com/p/3e0935bf2d45

- Okio 源码分析部分：https://www.jianshu.com/p/dccb6e1bd536

- Okio源码解析：https://juejin.im/post/5e0ed3835188253a5c7d1536

- 特点：

  - 它也是基于插管的，而且是单向的，输入源叫 Source，输出目标叫 Sink
  - 支持 Buffer（缓冲）
    - 像 NIO 一样，可以对 Buffer 进行操作
    - 但不强制使用 Buffer

- 核心类

  - ByteString：不可变的字节序列，像是 String 的兄弟
  - Buffer：可变的字节序列，类似 ArrayList
  - Source：类似于 Java 的 Inputstream
  - Sink：类似于 Java 的 Outputstream

- 用法：

  - 按行读取文本

    ```java
    public void readLines(File file) throws IOException {
        // 1. 构建 Source，类似于 Java 的 InputStream
        try (Source fileSource = Okio.source(file);
             // 2. 构建 BufferedSource，类似于 Java 的 BufferedInputStream
             BufferedSource bufferedSource = Okio.buffer(fileSource)) {
            while (true) {
                // 3. 按 UTF8 格式逐行读取字符
                String line = bufferedSource.readUtf8Line();
                if (line == null) break;
                if (line.contains("square")) {
                    System.out.println(line);
                }
            }
        }
    }
    ```

  - 写入文本

    ```java
    public void writeEnv(File file) throws IOException {
        // 1. 构建 Sink，类似于 Java 的 OutputStream
        try (Sink fileSink = Okio.sink(file);
             // 2. 构建 BufferedSink，类似于 Java 的 BufferedOutputStream
             BufferedSink bufferedSink = Okio.buffer(fileSink)) {
            // 3. 写入文本
            for (Map.Entry<String, String> entry : System.getenv().entrySet()) {
                bufferedSink.writeUtf8(entry.getKey());
                bufferedSink.writeUtf8("=");
                bufferedSink.writeUtf8(entry.getValue());
                bufferedSink.writeUtf8("\n");
            }
        }
    }
    ```

- 源码解析

  - Sink：类似于 Java 的 Outputstream

    ```kotlin
    interface Sink : Closeable, Flushable {
      fun write(source: Buffer, byteCount: Long)
    
      override fun flush()
    
      fun timeout(): Timeout
    
      override fun close()
    }
    ```

  - Okio.sink(file) 方法基于 OutputStream 获取一个 Sink

    ```java
      private static Sink sink(final OutputStream out, final Timeout timeout) {
        if (out == null) throw new IllegalArgumentException("out == null");
        if (timeout == null) throw new IllegalArgumentException("timeout == null");
    	// 构造 Sink 的匿名内部类并返回
        return new Sink() {
          // 主要实现了 write 方法
          @Override public void write(Buffer source, long byteCount) throws IOException {
            // 状态检验
            checkOffsetAndCount(source.size, 0, byteCount);
            while (byteCount > 0) {
              timeout.throwIfReached();
              // 从 Buffer 中获取了一个 Segment
              Segment head = source.head;
              int toCopy = (int) Math.min(byteCount, head.limit - head.pos);
              // 从 Segment 中取出数据，写入 Sink 所对应的 OutputStream
              out.write(head.data, head.pos, toCopy);
    
              head.pos += toCopy;
              byteCount -= toCopy;
              source.size -= toCopy;
    
              if (head.pos == head.limit) {
                source.head = head.pop();
                // 对当前 Segment 进行回收
                SegmentPool.recycle(head);
              }
            }
          }
    
          @Override public void flush() throws IOException {
            out.flush();
          }
    
          @Override public void close() throws IOException {
            out.close();
          }
    
          @Override public Timeout timeout() {
            return timeout;
          }
    
          @Override public String toString() {
            return "sink(" + out + ")";
          }
        };
      }
    ```

  - Source：类似于 Java 的 Inputstream

    ```kotlin
    interface Source : Closeable {
      fun read(sink: Buffer, byteCount: Long): Long
    
      fun timeout(): Timeout
    
      override fun close()
    }
    ```

  - 通过 source 方法根据  InputStream 创建一个 Source

    ```java
      private static Source source(final InputStream in, final Timeout timeout) {
        if (in == null) throw new IllegalArgumentException("in == null");
        if (timeout == null) throw new IllegalArgumentException("timeout == null");
    	// 构建并实现了 Source 的一个匿名内部类并返回
        return new Source() {
          @Override public long read(Buffer sink, long byteCount) throws IOException {
            // 状态检测
            if (byteCount < 0) throw new IllegalArgumentException("byteCount < 0: " + byteCount);
            if (byteCount == 0) return 0;
            try {
              timeout.throwIfReached();
              // 获取到一个可以写入的 Segment
              Segment tail = sink.writableSegment(1);
              int maxToCopy = (int) Math.min(byteCount, Segment.SIZE - tail.limit);
              // 从 InputStream 中读取数据向 Segment 中写入
              int bytesRead = in.read(tail.data, tail.limit, maxToCopy);
              if (bytesRead == -1) return -1;
              tail.limit += bytesRead;
              sink.size += bytesRead;
              return bytesRead;
            } catch (AssertionError e) {
              if (isAndroidGetsocknameError(e)) throw new IOException(e);
              throw e;
            }
          }
    
          @Override public void close() throws IOException {
            in.close();
          }
    
          @Override public Timeout timeout() {
            return timeout;
          }
    
          @Override public String toString() {
            return "source(" + in + ")";
          }
        };
      }
    ```

