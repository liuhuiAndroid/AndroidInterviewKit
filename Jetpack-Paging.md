#### 学习 Paging

为了帮助开发者更高效、更容易地构建优秀的应用，Google 推出了 Android Jetpack。它包含了开发库、工具、以及最佳实践指南。

我们经常需要处理大量数据，但大多数情况下，只需要加载和显示其中的一小部分。如果请求并不需要的数据，会浪费用户的电池和带宽。而且，加入数据过于庞大，那么同步界面的时候，可能会消耗不少的流量，并且代码会更加复杂。

分页库 Paging Library 是 Jetpack 的一部分，它可以妥善的逐步加载数据，从而解决上述问题。分页库支持加载有限以及无限的 List，比如一个持续更新的信息源。我们知道，RecyclerView 一般用于显示大量数据，分页库可以与 RecyclerView 无缝集合，它还可以与 LiveData 或 RxJava 集成，观察界面中的数据变化。

分页库的原理是将数据分成多个 List，使用 RecyclerView 中的 Adapter，来观察 LiveData 中数据的变化。在这里 LiveData 的类是 List，在此基础上，我们加入了分页的功能，从而可以逐步加载内容。

我们讲一下分页库的主要组件，可以如何应用到架构中。分页库的核心组件是 PagedList 和 DataSource。PagedList 是一个集合类，它以分块的形式异步加载数据，每一块我们称之为页。DataSource 是将数据加载到 PagedList 中的基类，任何数据都可以作为 DataSource 的来源，比如网络、数据库、文件等等。DataSource.Factory 类可以用来创建 DataSource。分页库还提供了 PagedListAdapter，可以将 PageList 中的数据加载入 RecyclerView 中，PagedListAdapter 会在页加载时收到通知，收到新数据时，它会使用 DiffUtil 精细计算更新。分页库提供了 LivePagedListBuilder 类，用于获取 PagedList 类型的 LiveData 对象，创建 LivePagedListBuilder 的参数，包括 DataSource.Factory 对象和分页配置对象，如果倾向于使用 RxJava，而不是 LiveData，那么请使用 RxPagedListBuilder，它的构建方式与 LivePagedListBuilder 类似，不同之处在于 RxPagedListBuilder 返回一个 Observerable 对象或 Flowable 对象，而不是 LiveData 对象。

我们接下来看一下从数据库，或基于 Retrofit 网络源加载数据的常见场景，进一步了解分页库的作用。

第一个情景：假设数据源是数据库，Room 存储库可以作为分页库的数据源，对于给定查询关键字，Room 可以从 DAO 返回 DataSource.Factory，从而无缝处理 DataSource 的实现。

第二个情景：假设数据库是从网络加载的数据的缓存，在这个情景中，仍然从 DAO 返回 DataSource.Factory，不过还需要另外一个分页组件：BoundaryCallback。当界面显示缓存中靠近结尾的数据的时候，BoundaryCallback 将加载更多的数据。在获得更多的数据后，分页库将自动更新界面。在这里，不要忘了将 BoundaryCallback 与之前创建的 LivePagedListBuilder 进行关联，关联之后，PagedList 就可以使用它了。

第三个情景：仅将网络作为数据源，在这个情景中，需要创建 DataSource 和 DataSource.Factory，选择 DataSource 类型的时候，需要综合考虑后端 API 的架构。如果通过键值请求后端数据，那我们用 ItemKeyedDataSource。举个例子，我们需要获取在某个特定日期起 Github 的前100项代码提交，该日期将成为 DataSource 的键，ItemKeyedDataSource 允许自定义如何加载初始页，以及如何加载某个键值前后的数据，如果后端API返回数据是分页之后的，那么我们可以用 PageKeyedDataSource，比如，Github API 中 SearchRepositories 就可以返回分页数据。我们在 Github API 的请求中指定查询关键字和想要的哪一页，同时也可以指明每个页面的项数。不管网络数据源的创建方式是什么，都需要创建 DataSource.Factory，有了 DataSource.Factory 就可以创建 DataSource。

我们将完整实例上传到了 Github，包括如何处理错误情况以及如何进行重试。

最后，我们来简单总结一下如何使用分页库。首先，要定义 DataSource，需要的时候，创建 BoundaryCallback，使用 LivePagedListBuilder 创建 PagedList 类的 LiveData，将 Adapter 转化为 PagedListAdapter，最后，UI 声明观察 PagedList 类的 LiveData，并将 PagedList 设为 Adapter。

希望能给大家带来灵感，做出高效、流畅并且节省带宽的分页列表。

[视频 Android Jetpack: 分页库 (Paging Library)](<https://www.bilibili.com/video/av35089294>)

[《即学即用Android Jetpack - Paging》](https://www.jianshu.com/p/0b7c82a5c27f)

[第一步：Paging官方文档](<https://developer.android.com/topic/libraries/architecture/paging>)