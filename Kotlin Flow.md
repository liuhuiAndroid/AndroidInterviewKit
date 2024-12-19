Flow：异步数据流框架

冷流

热流：StateFlow/SharedFlow，StateFlow 和 LiveData 定位类似；SharedFlow 可以保留历史数据，可以理解为 LiveData 粘性事件。StateFlow 只保留最新值，SharedFlow 可以保留历史数据。
