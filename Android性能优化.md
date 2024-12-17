#### 启动速度优化

启动资源精简

异步加载非必要第三方SDK

延迟初始化方案

traceview 和 systrace 

jetpack startup：实现一种在应用启动时初始化组件的简单而高效的方法。



#### 内存优化

内存问题

1. 内存抖动：锯齿状、GC导致卡顿
2. 内存泄漏：可用内存减少、频繁GC
3. 内存溢出：OOM、程序异常

内存分析工具 MAT

Glide：正确应对系统内存不足，OnLowMemory 和 OnTrimMemory 回调

```
    override fun onTrimMemory(level: Int) {
        super.onTrimMemory(level)
        if (level == ComponentCallbacks2.TRIM_MEMORY_UI_HIDDEN) {
            Glide.get(this).clearMemory()
        }
        Glide.get(this).onTrimMemory(level)
    }

    override fun onLowMemory() {
        super.onLowMemory()
        Glide.get(this).onLowMemory()
    }
```





#### 布局优化

Systrace

Layout Inspector：查看视图层次结构



#### 卡顿优化

Systrace：监控和跟踪Api调用、线程运行情况，生成html报告

自动化卡顿检测方案

ANR-WatchDog



[Android App包瘦身优化实践 - 美团技术团队](https://tech.meituan.com/2017/04/07/android-shrink-overall-solution.html)

[安装包立减1M–微信Android资源混淆打包工具](http://mp.weixin.qq.com/s?__biz=MzAwNDY1ODY2OQ==&mid=208135658&idx=1&sn=ac9bd6b4927e9e82f9fa14e396183a8f#rd)



#### App瘦身，体积精简

业务优化

1. 业务拆分
   - 业务更细粒度拆分，组件化，按需接入
2. 功能裁剪：
   - 简化现有业务流程，低频功能删除

技术优化

1. 代码瘦身
   1. 代码混淆；开启 minifyEnabled，app 混淆时会删除未使用的代码
   2. 开启混淆工具 R8 的 Full Mode，进一步压缩代码
   3. 三方库处理
   4. 移除无用代码，无用代码清理； 
   5. 公共代码抽取复用；
   6. 枚举替换成 @IntDef
2. 资源瘦身
   1. 冗余资源：使用工具检查冗余资源并删除；开启 shrinkResources，移除未使用资源
   2. 图片压缩，只保留特定屏幕密度的图片资源；图片使用 webp 格式
   3. 资源混淆
   4. 资源在线化
3. so 瘦身
   1. 自研及第三方SDK体积压缩优化

4. 编译优化
   - 符号表裁剪
   - 多个动态库合并为一个
   - 使用redex工具优化压缩dex文件
5. 编码算法升级：
   - 图片编码算法升级；
   - 语言编码算法升级；



#### App稳定性优化

1. 正确认识
   1. 稳定性是大问题，Crash是P0优先级
   2. 稳定性可优化的面很广
2. 稳定性维度
   1. Crash纬度
   2. 性能纬度
   3. 业务高可用纬度
3. 稳定性优化概览
   1. 重在预防，监控必不可少
   2. 思考更深一层、重视隐含信息
   3. 长效保持需要科学流程
