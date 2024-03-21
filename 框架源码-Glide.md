- Glide 缓存机制

  - 内存缓存
  - 磁盘缓存

- 如何实现LRU Cache

  LinkedHashMap：它有一个有序的哈希表，适合实现 LRU 缓存的行为

- 如何自己实现加载超大图，避免 OOM

  1. 使用 Bitmap Option 的 inSampleSize
  2. 分块加载
