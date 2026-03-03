字节飞书安卓岗一面

自我介绍，

java中的hashmap 结构是怎么样的怎么get和put等等

java中synchronized和ReentrantLock

都是怎么样的有什么区别（谁是公平锁）是不是可重入的怎么实现可重入的项目中用过吗怎么使用的 没怎么使用过对于安卓开发

java中jvm的回收机制是怎么样的什么时候会回收，怎么判断一个对象或者什么东西应该被回收，回收算法是怎么样的，kotlin中和这个一样吗？他做了什么优化

kotlin和java的区别

kotlin中的协程 suspend关键字干了些什么

协程又和线程有什么关系他俩有什么不同点

那进程又和他们有什么不同点

kotlin的内联函数是怎么优化的 他既然是将函数放到原位置那为啥不把所有的fun都设置成inline

会不会flutter--了解过一点dart不会

http1.0-http3.0经过了怎么样的优化

retrofit库他干了什么他常用的注解，为什么加一个这种注解就会进行网络请求（他是什么时期的注解为什么是这个时期的注解）、

网络请求的缓存标志有哪些 --etag 、lastmodify等等

我们经常去请求一些图片 我们怎么设置缓存机制（或者说我们怎么对glide进行一个进一步的优化）

答：lru 里面加一些判断这些优先级的比较关系将他融入进去巴拉巴拉纯胡说

那将这些设置成影响因子加入优先级我要是有一些图片只要请求过来了就不会变了以后用这些图片只能去本地协商缓存又要怎么做，你刚才提到了lru他是一种什么样的数据结构，为什么能保证效率又能实现这种效果呢

巴拉巴不会过

handler是安卓中常见的线程间通信的组件 他的原理是什么，我们使用的时候要注意什么，

刚刚提到内存泄漏 ，常见的内存泄漏都有什么，为了避免这种内存泄漏我们作为开发者怎么去注意这些问题

每个线程怎么去取队列里面的消息怎么判断队列里面是不是有消息，没有消息了又怎么办looper他的底层是怎么干，没消息了还会一直取吗

实习中你写到保证0bug交接这个是怎么实现的你中间碰到了一些什么难点怎么解决的

实习项目是mvvm吗？他和其他有什么区别他有什么好处和坏处

activity 的生命周期 A跳转B发生了什么有什么情况A生命周期一定是这样的吗不是的话是那种情况

问平时使用过ai吗都用过哪些你对ai是怎么看的

写算法 leetcode76最小匹配串 讲一讲你的代码



反问 

本人是第一次面这么大厂 作为面试官更看重我们实习生什么

答：大厂是给大家学习的地方、主要是筛选一些计算机基础比较好的学习能力强的、我们在这个浓厚的学习氛围的条件下一起帮助大家提升

问作为ai时代、ai迭代很快抛去此次面试来看我我应该做哪些提升来适应当下的环境

答：学好基础、该会的得会、我们不能让ai写的代码成为黑盒对于开发者来说、拥抱ai，用好奇去战胜恐惧

问业务 

答：飞书体系其实是很庞大的需求很多服务企业做一些企业化的流程能在飞书上执行



# 字节跳动飞书 Android 岗一面 — 知识点总结与优秀答案

------

## 一、Java HashMap 的结构与原理

**数据结构：** JDK 1.8 之后，HashMap 底层是 **数组 + 链表 + 红黑树** 的组合结构。

- 底层维护一个 `Node[] table` 数组（默认容量 16）。
- 每个数组槽位是一个链表头节点；当链表长度 ≥ 8 且数组长度 ≥ 64 时，链表转为**红黑树**（查找从 O(n) 优化到 O(log n)）。
- 负载因子默认 0.75，超过阈值（容量 × 负载因子）时触发**2倍扩容**并 rehash。

**put 流程：**

1. 计算 key 的 `hash(key)`（扰动函数，高16位异或低16位，减少碰撞）。
2. `index = (n-1) & hash` 定位数组槽位。
3. 若为空，直接插入；不为空则遍历链表/红黑树，key 相同则覆盖，否则尾插。
4. 插入后检查是否需要树化或扩容。

**get 流程：**

1. 同样计算 hash 定位槽位。
2. 先比较头节点，若匹配返回；否则遍历链表或红黑树查找。

**线程安全问题：** HashMap 非线程安全，多线程场景用 `ConcurrentHashMap`（分段锁/CAS + synchronized）。

------

## 二、synchronized 与 ReentrantLock 的区别

| 对比项     | synchronized                                   | ReentrantLock                                |
| ---------- | ---------------------------------------------- | -------------------------------------------- |
| 实现层面   | JVM 内置关键字                                 | JUC 包，Java 层实现                          |
| 是否公平锁 | **非公平锁**（不可改变）                       | 默认**非公平**，可传参 `true` 变为**公平锁** |
| 是否可重入 | ✅ 可重入                                       | ✅ 可重入                                     |
| 锁释放     | 自动释放（退出同步块）                         | 必须手动 `unlock()`，建议放在 `finally`      |
| 功能       | 基本互斥                                       | 支持超时获取、可中断、多条件变量 `Condition` |
| 性能       | JDK 优化后（偏向锁→轻量级锁→重量级锁）差距不大 | 高并发竞争激烈时更灵活                       |

**如何实现可重入：**

- `synchronized`：JVM 对象头 Mark Word 记录持有线程ID和重入计数，同一线程再次进入计数+1，退出-1，归零才释放。
- `ReentrantLock`：内部 AQS（AbstractQueuedSynchronizer）维护 `state` 计数和 `exclusiveOwnerThread`，同一线程每次 lock() state+1，unlock() state-1，归零释放。

**Android 开发场景：**

- 简单的方法/代码块同步 → 优先用 `synchronized`，写法简单安全。
- 需要超时等待、可中断、公平性控制时 → 用 `ReentrantLock`。

------

## 三、JVM 垃圾回收机制

**判断对象是否可回收：**

- ~~引用计数法~~（有循环引用缺陷，JVM 不采用）
- **可达性分析（GC Roots）**：从 GC Roots（栈帧局部变量、静态变量、JNI 引用等）出发，不可达的对象即为垃圾。

**回收算法：**

| 算法      | 原理                               | 优点           | 缺点           |
| --------- | ---------------------------------- | -------------- | -------------- |
| 标记-清除 | 标记垃圾后直接清除                 | 简单           | 内存碎片       |
| 标记-复制 | 存活对象复制到另半区               | 无碎片，效率高 | 内存利用率 50% |
| 标记-整理 | 存活对象向一端移动，清理边界外     | 无碎片         | 移动对象开销   |
| 分代收集  | Young（复制算法）+ Old（标记整理） | 综合效率最优   | 实现复杂       |

**现代 GC（G1/ZGC）：** 将堆划分为 Region，可预测停顿时间，ZGC 实现几乎全程并发，STW 时间 < 1ms。

**Kotlin 与 Java JVM 的 GC：** Kotlin 编译为同样的字节码运行在 JVM 上，GC 机制完全一致。

**Kotlin 的内存优化：**

- 默认不可空类型（`String` vs `String?`）在编译期避免大量 null check，减少无效对象分配。
- 内联函数（`inline`）消除 lambda 对象分配（每个 lambda 默认会创建匿名类实例）。
- 数据类、`object` 单例等语法糖在编译器层面做了优化。

------

## 四、Kotlin 与 Java 的主要区别

| 特性              | Kotlin                                              | Java                        |
| ----------------- | --------------------------------------------------- | --------------------------- |
| 空安全            | 编译期空检查（`?` / `!!`）                          | 运行时 NullPointerException |
| 数据类            | `data class` 自动生成 equals/hashCode/toString/copy | 需手动或 Lombok             |
| 扩展函数          | 支持，无需继承                                      | 不支持                      |
| 协程              | 官方支持                                            | 需第三方（Project Loom）    |
| 函数式            | 一等公民，lambda 更简洁                             | Java 8+ 支持但语法繁琐      |
| 默认不可变        | `val` 不可变，推荐函数式风格                        | 需显式 `final`              |
| 智能转换          | `is` 检查后自动转换类型                             | 需显式强转                  |
| 默认参数/命名参数 | 支持                                                | 不支持，需重载              |

------

## 五、Kotlin 协程

**`suspend` 关键字做了什么：**

- `suspend` 是编译器魔法，并不创建新线程。
- 编译器将 suspend 函数转换为状态机（CPS，Continuation Passing Style），函数签名末尾隐式添加 `Continuation` 参数。
- 每个挂起点对应状态机的一个状态，恢复时从上次挂起的地方继续执行。
- 挂起时释放当前线程（不阻塞），任务完成后在指定调度器线程恢复。

**协程 vs 线程：**

| 对比     | 协程                       | 线程                    |
| -------- | -------------------------- | ----------------------- |
| 调度     | 用户态调度，轻量           | 内核态调度，开销大      |
| 内存     | 初始栈极小（KB级）         | 默认 1MB 栈空间         |
| 切换开销 | 极低（无系统调用）         | 高（上下文切换）        |
| 并发模型 | 结构化并发，挂起不阻塞线程 | 阻塞式并发              |
| 数量     | 可轻松开启数万个           | 数百~数千（受系统限制） |

**协程 vs 线程 vs 进程：**

- **进程**：资源分配最小单位，拥有独立内存空间，进程间通信（IPC）开销大。
- **线程**：CPU 调度最小单位，同进程内共享内存，切换开销中等。
- **协程**：运行在线程之上，由程序自身调度，切换开销极低，是对线程的复用和抽象。

------

## 六、Kotlin 内联函数（inline）

**优化原理：** `inline` 修饰的函数，编译器在调用处将**函数体直接展开（内联）**，同时将传入的 lambda 也内联，避免创建 Function 对象和虚方法调用。

**为什么不把所有函数都设为 inline：**

1. **代码膨胀**：每个调用处都展开函数体，若函数体大、调用次数多，编译产物体积会急剧增大（APK 变大、指令缓存命中率下降）。
2. **递归函数无法内联**：inline 函数不支持递归。
3. **函数引用限制**：inline 函数内的 lambda 参数不能被存储或间接调用（除非用 `noinline`）。
4. **适用场景有限**：只有接收函数类型参数（lambda）时收益最大；普通函数内联收益甚微。

**最佳实践：** 高阶函数 + 小函数体时用 `inline`，如 `filter`、`map`、`let` 等标准库函数。

------

## 七、HTTP 1.0 → HTTP 3.0 演进

| 版本     | 核心优化                                                     | 痛点                                       |
| -------- | ------------------------------------------------------------ | ------------------------------------------ |
| HTTP/1.0 | 每次请求新建 TCP 连接                                        | 连接开销极大                               |
| HTTP/1.1 | 持久连接（Keep-Alive）、管道化（Pipelining）                 | 队头阻塞（HOL Blocking）、Header 冗余      |
| HTTP/2.0 | **多路复用**（单连接并发流）、**头部压缩**（HPACK）、**服务端推送**、二进制帧 | TCP 层队头阻塞仍存在（丢包重传阻塞所有流） |
| HTTP/3.0 | 底层改用 **QUIC**（基于 UDP）、连接迁移、0-RTT 握手、彻底解决队头阻塞 | UDP 穿透防火墙受限，部署成本高             |

------

## 八、Retrofit 的原理与注解

**Retrofit 做了什么：** Retrofit 是一个基于 OkHttp 的 REST 客户端封装库，核心是通过**动态代理**将接口方法调用转换为 HTTP 请求。

**核心流程：**

1. 调用 `retrofit.create(ApiService::class.java)` 时，使用 `java.lang.reflect.Proxy.newProxyInstance` 创建接口的动态代理对象。
2. 调用任意接口方法时，触发 `InvocationHandler.invoke()`。
3. Retrofit 解析该方法的注解（`@GET`、`@POST`、`@Path`、`@Query` 等），构建 `Request` 对象。
4. 通过 OkHttp 发起请求，结合 `CallAdapter` 和 `Converter` 处理响应（如转换为 Kotlin 协程、RxJava、或 Gson 解析）。

**注解为什么是"编译期"？** 严格说，Retrofit 的注解是**运行时注解**（`@Retention(RUNTIME)`），在运行时通过反射读取。之所以"加个注解就能请求"，是因为动态代理在运行时读取注解并构建请求逻辑，而非编译期代码生成（Retrofit 与 KSP/APT 的注解处理器不同）。

**常用注解：**

- 请求方法：`@GET`、`@POST`、`@PUT`、`@DELETE`、`@PATCH`
- 参数：`@Path`、`@Query`、`@QueryMap`、`@Body`、`@Field`、`@Header`
- 标记：`@FormUrlEncoded`、`@Multipart`、`@Streaming`

------

## 九、HTTP 缓存机制

**强缓存（不发请求）：**

- `Cache-Control: max-age=3600`（优先级更高）
- `Expires: GMT时间`（HTTP/1.0）

**协商缓存（发请求，服务端判断）：**

- `Last-Modified` / `If-Modified-Since`：基于文件修改时间，精度秒级。

- ```
  ETag
  ```

   / 

  ```
  If-None-Match
  ```

  ：基于内容哈希，精度更高，优先于 Last-Modified。

  - 服务端返回 `304 Not Modified` 时，客户端直接使用本地缓存。

------

## 十、Glide 缓存优化

**Glide 默认的三级缓存：**

1. **活跃缓存（Active Resources）**：正在使用的图片，弱引用持有。
2. **内存缓存（LruCache）**：最近使用的图片，强引用，LRU 淘汰。
3. **磁盘缓存（DiskLruCache）**：本地文件，支持原图或变换后的图。

**LRU（Least Recently Used）的数据结构：**

- LinkedHashMap

  ：内部维护一个双向链表 + HashMap。

  - HashMap 保证 O(1) 查找。
  - 双向链表维护访问顺序，最近访问的移到链表头，链表尾即为最久未使用的，超出容量时删除尾节点。
  - `LinkedHashMap(capacity, 0.75f, true)` 第三个参数 `true` 表示按访问顺序排序。

**进一步优化：**

- 自定义 `MemoryCategory` 或 `BitmapPool` 根据设备内存动态调整缓存大小。

- 对于

  不会改变的图片

  （如 CDN 静态资源）：

  - Glide 加载时指定 `DiskCacheStrategy.DATA` 或 `DiskCacheStrategy.ALL`。
  - 服务端响应头设置 `Cache-Control: immutable` 或极长 `max-age`。
  - 结合 ETag 协商缓存确保资源更新时能获取新版本。

------

## 十一、Handler 机制

**原理：** Handler、Looper、MessageQueue、Message 四者协作：

1. `Looper.prepare()` 为当前线程创建 `Looper` 和 `MessageQueue`（存储在 ThreadLocal）。
2. `Handler` 构造时绑定当前线程的 `Looper`，`sendMessage()`/`post()` 将消息放入 MessageQueue。
3. `Looper.loop()` 开启无限循环，不断从 MessageQueue 取消息分发给对应 Handler 的 `handleMessage()`。

**消息队列取消息的机制（底层）：**

- MessageQueue 内部使用 Linux 的 **epoll 机制**（`nativePollOnce`）。
- 有消息时唤醒，立即处理；无消息或消息未到时间时，线程**阻塞**（释放 CPU），不会空转浪费资源。
- 新消息入队时，通过 `nativeWake` 写入管道触发 epoll 唤醒。

**使用注意：**

- 主线程 Handler 持有 Activity 引用 → **内存泄漏**。

------

## 十二、常见内存泄漏及避免方式

| 泄漏场景          | 原因                                                       | 解决方案                                                     |
| ----------------- | ---------------------------------------------------------- | ------------------------------------------------------------ |
| Handler 内部类    | 非静态内部类持有外部 Activity 引用，消息队列中有待处理消息 | 静态内部类 + WeakReference，页面销毁时 `removeCallbacksAndMessages(null)` |
| 单例持有 Context  | 单例生命周期 = 应用生命周期，持有 Activity Context         | 改用 `applicationContext`                                    |
| 静态变量持有 View | View 持有 Activity 引用                                    | 页面销毁时置 null                                            |
| 注册未注销        | BroadcastReceiver、EventBus、监听器未注销                  | 在 `onDestroy` 中注销                                        |
| 未关闭资源        | Cursor、Stream、数据库连接未关闭                           | try-finally 或 `use {}`                                      |
| 匿名内部类/lambda | 持有外部类引用（如 Retrofit callback、RxJava 订阅）        | 使用 `viewModelScope` / `lifecycleScope`，生命周期自动取消   |
| WebView           | WebView 持有 Activity 引用                                 | 单独进程或动态 addView，销毁时手动 destroy                   |

**检测工具：** LeakCanary（自动检测）、Android Studio Profiler（Memory 分析）。

------

## 十三、MVVM 架构

**MVVM vs MVP vs MVC：**

|                  | MVC      | MVP            | MVVM                                     |
| ---------------- | -------- | -------------- | ---------------------------------------- |
| View 与逻辑      | 耦合严重 | Presenter 解耦 | ViewModel 解耦 + 数据绑定                |
| 测试性           | 差       | 较好           | 好                                       |
| Android 官方支持 | 无       | 无             | Jetpack (ViewModel + LiveData/StateFlow) |

**MVVM 优点：**

- ViewModel 不持有 View 引用，不受生命周期影响（旋转屏幕不丢失数据）。
- LiveData/StateFlow 自动感知生命周期，避免内存泄漏。
- 单向数据流，状态可预测，便于测试。

**MVVM 缺点：**

- 简单页面过度设计，样板代码多。
- 数据绑定调试困难（双向绑定错误难定位）。
- 学习曲线相对较高。

------

## 十四、Activity 生命周期与 A 跳转 B

**标准跳转（A 启动 B）：**

```
A.onPause() → B.onCreate() → B.onStart() → B.onResume() → A.onStop()
```

**特殊情况：**

- 若 B 是**透明主题**或**对话框 Activity**：A 不会调用 `onStop()`，因为 A 仍然可见。
- 若 B 的 `launchMode = singleTask` 且已在栈中：会调用 B 的 `onNewIntent()` 而非 `onCreate()`。
- 若开启动画，时序可能略有不同。
- 按 Home 键：`onPause → onStop`；按 Back 键返回：`onRestart → onStart → onResume`。

------

## 十五、LeetCode 76 — 最小覆盖子串（滑动窗口）

**思路：双指针滑动窗口**

```kotlin
fun minWindow(s: String, t: String): String {
    val need = HashMap<Char, Int>()
    for (c in t) need[c] = (need[c] ?: 0) + 1

    var left = 0; var right = 0
    var valid = 0          // 已满足条件的字符种数
    var start = 0; var minLen = Int.MAX_VALUE

    val window = HashMap<Char, Int>()

    while (right < s.length) {
        val c = s[right++]
        if (need.containsKey(c)) {
            window[c] = (window[c] ?: 0) + 1
            if (window[c] == need[c]) valid++
        }
        // 当窗口覆盖 t 时，收缩左边界
        while (valid == need.size) {
            if (right - left < minLen) {
                start = left; minLen = right - left
            }
            val d = s[left++]
            if (need.containsKey(d)) {
                if (window[d] == need[d]) valid--
                window[d] = window[d]!! - 1
            }
        }
    }
    return if (minLen == Int.MAX_VALUE) "" else s.substring(start, start + minLen)
}
```

**时间复杂度：** O(|S| + |T|)，每个字符最多被访问两次。 **空间复杂度：** O(|S| + |T|)。

------

## 十六、面试官反问问题参考答案

**Q：大厂更看重实习生什么？** 计算机基础扎实（数据结构、算法、操作系统、网络）、学习能力强、代码规范、主动沟通、能将理论落地到实际工程问题。

**Q：AI 时代如何提升自己适应环境？**

- 夯实基础：AI 生成的代码仍需工程师审查和调试，理解底层原理才能辨别对错。
- 拥抱工具：熟练使用 Copilot、Claude 等提效，将重复工作自动化，聚焦更复杂的架构设计和业务判断。
- 保持好奇：关注新技术趋势（端侧 AI、LLM 应用开发），探索 AI 与 Android 结合的新场景。
- 软实力：沟通、产品思维、跨端协作能力是 AI 难以替代的核心竞争力。

------

*总结整理于 2026年3月 | 字节跳动飞书 Android 岗一面复盘*



