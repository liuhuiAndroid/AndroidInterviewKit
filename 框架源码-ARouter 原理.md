https://github.com/Xiasm/EasyRouter

ARouter 的基本原理

1. **注解处理器：** ARouter 使用注解来标记需要进行路由的页面，例如使用 `@Route` 注解标记的 Activity 或 Fragment。注解处理器在编译时扫描这些注解，并生成相应的路由表。

2. **路由表：** 路由表是 ARouter 中的核心数据结构，它记录了每个页面的路径、类名等信息。路由表的生成是通过注解处理器在编译时自动生成的。这样，ARouter 在运行时可以快速查找到页面对应的信息。

3. **路由导航：** 在需要进行页面跳转的地方，调用 ARouter 的 API 进行导航。ARouter 根据传入的路径在路由表中查找对应的页面信息，然后使用反射机制实例化目标页面，并进行页面跳转。

4. **参数传递：** ARouter 支持通过路由表传递参数。在进行页面跳转时，可以携带参数，ARouter 将参数封装到 Intent 或 Bundle 中，然后传递给目标页面。

5. **拦截器：** ARouter 提供了拦截器机制，允许开发者在路由导航过程中进行拦截和处理。拦截器可以用于权限验证、登录检查等操作。

6. **降级策略：** ARouter 支持降级策略，即当无法找到对应的页面时，可以指定一个降级页面进行跳转，或者执行自定义的降级处理逻辑。

如果没有 ARouter 路由框架，如何实现来进行组件间的通信和页面跳转？

1. Intent
2. Event Bus => 进行组件间通信
3. 接口回调
4. 中心化的服务定位器模式
5. 依赖注入
