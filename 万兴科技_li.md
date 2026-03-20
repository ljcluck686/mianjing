# 万兴科技

## 问好多性能优化的东西

实习

1. compose和传统xml view绘制有什么不一样，他们的性能有什么差异，他们的渲染次数有什么差异
2. Androidstudio上面有能查看重组次数的工具用过吗
3. okhttp原理是什么，里面有一个连接池
4. 数据库要一下插10000条数据怎么优化
5. 自定义view 有一个text从底部慢慢升起来怎么实现
6. 有一个大的技术文件要传到服务器上面 怎么去做 要保证可靠性和安全性和效率
7. anr遇到过吗原理是什么 如果内存很高会影响anr吗
8. 图片库用过吧 我本地要存好多个大文件有什么方法 弱引用加lru时间过期 除了时间过期还有别的方式吗
9. app启动性能优化做过吗  没有 那你觉得可以从哪下手
10. ui渲染性能优化做过吗 怎么做
11. activity 启动流程 四大组件 跨进程通信
12. 协程 是什么
13. 什么情况下回进行内存泄漏
14. 了解过hook吗
15. 埋点是自己手埋吗要是要让ai帮你怎么去写提示词
16. 反问

 

# 万兴科技实习面试题详解

## 1. Compose vs 传统XML View 绘制差异

### 渲染机制对比

**传统XML View：**

- 基于**命令式UI**，手动管理状态与UI同步
- 渲染流程：`Measure → Layout → Draw` 三阶段，每次更新可能触发整棵View树的重新测量/布局
- `invalidate()` 触发重绘，`requestLayout()` 触发重新测量+布局
- ViewGroup嵌套越深，遍历开销越大

**Jetpack Compose：**

- 基于**声明式UI**，状态驱动UI自动更新
- 渲染流程：`Composition → Layout → Drawing` 三阶段
- 核心优化：**智能重组（Recomposition）**，只重新执行状态发生变化的Composable函数，跳过未变化部分

### 性能差异

| 维度     | XML View                               | Jetpack Compose             |
| -------- | -------------------------------------- | --------------------------- |
| 布局嵌套 | 嵌套深影响性能，需ConstraintLayout优化 | 扁平化布局，天然减少层级    |
| 局部更新 | 难以精确控制，易触发父级重绘           | 精准重组，只更新变化节点    |
| 列表性能 | RecyclerView需手动优化ViewHolder       | LazyColumn内置高效复用机制  |
| 动画     | Animator体系，相对繁琐                 | 内置动画API，与状态深度集成 |

### 渲染次数差异

- XML View：状态变化 → `invalidate()` → 可能触发大范围重绘
- Compose：通过`@Stable`、`remember`、`derivedStateOf`等减少重组次数；未被读取的State变化**不会触发**其所在Composable重组

------

## 2. Android Studio 查看重组次数工具

**Layout Inspector（重组计数器）：**

- `Android Studio → View → Tool Windows → Layout Inspector`
- 连接设备后，在Compose树中可看到每个Composable的 **Recomposition Count** 和 **Skipped Count**
- Skipped越高说明优化越好（跳过了不必要的重组）

**其他工具：**

- **Composition Tracing**：结合Perfetto追踪重组的详细调用栈
- `RECOMPOSITION_HIGHLIGHT`：开启后，高频重组的节点会在屏幕上闪烁高亮显示

**优化方向：** 若某Composable重组次数异常高，排查是否有不稳定的参数类型（如普通`data class`未标注`@Stable`/`@Immutable`）。

------

## 3. OkHttp 原理 + 连接池

### 核心架构：责任链拦截器

```
Application Code
      ↓
 RetryAndFollowUpInterceptor   // 重试、重定向
      ↓
 BridgeInterceptor              // 处理请求头/响应头
      ↓
 CacheInterceptor               // 缓存策略
      ↓
 ConnectInterceptor             // 建立连接（从连接池取）
      ↓
 NetworkInterceptors            // 自定义网络拦截器
      ↓
 CallServerInterceptor          // 真正发送请求/读取响应
```

### 连接池（ConnectionPool）原理

```kotlin
// 默认配置
ConnectionPool(
    maxIdleConnections = 5,   // 最多保留5个空闲连接
    keepAliveDuration = 5,    // 空闲连接存活5分钟
    timeUnit = TimeUnit.MINUTES
)
```

**核心机制：**

- 基于`HTTP Keep-Alive`，复用TCP连接，避免频繁三次握手
- 支持**HTTP/2多路复用**：同一连接并发多个请求
- 连接匹配规则：host + port + 协议完全一致才能复用
- 清理策略：后台线程定期清理超时空闲连接（`RealConnectionPool.cleanup()`）

------

## 4. 一次性插入10000条数据的优化

### 核心方案：事务 + 批量插入

```kotlin
// ❌ 错误做法：每条数据单独提交事务，10000次IO
data.forEach { db.insert(it) }

// ✅ 正确做法：一个事务批量插入
db.beginTransaction()
try {
    val stmt = db.compileStatement("INSERT INTO table VALUES (?,?,?)")
    data.forEach { item ->
        stmt.bindString(1, item.name)
        stmt.bindLong(2, item.id)
        stmt.clearBindings()
        stmt.executeInsert()
    }
    db.setTransactionSuccessful()
} finally {
    db.endTransaction()
}
```

### Room 方案

```kotlin
@Dao
interface DataDao {
    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insertAll(list: List<DataEntity>)  // Room自动批量处理
}

// 配合协程在IO线程执行
viewModelScope.launch(Dispatchers.IO) {
    dao.insertAll(tenThousandItems)
}
```

### 其他优化点

- **分批插入**：每批500~1000条，避免单事务过大锁表时间过长
- **WAL模式**：`PRAGMA journal_mode=WAL` 提升并发写入性能
- **关闭索引**：插入前临时关闭索引，插入完再重建（适合初始化场景）

------

## 5. 自定义View：文字从底部缓缓升起

```kotlin
class FloatingTextView @JvmOverloads constructor(
    context: Context, attrs: AttributeSet? = null
) : View(context, attrs) {

    private val paint = Paint(Paint.ANTI_ALIAS_FLAG).apply {
        textSize = 60f
        color = Color.BLACK
    }
    private val text = "Hello"
    
    // 动画进度 0f~1f
    private var animProgress = 0f

    override fun onDraw(canvas: Canvas) {
        super.onDraw(canvas)
        val textWidth = paint.measureText(text)
        val x = (width - textWidth) / 2f
        
        // 起始Y = 底部出屏幕；终止Y = 垂直居中
        val startY = height + 60f
        val endY = height / 2f
        val currentY = startY + (endY - startY) * animProgress
        
        // 同步淡入效果
        paint.alpha = (255 * animProgress).toInt()
        canvas.drawText(text, x, currentY, paint)
    }

    fun startAnimation() {
        ValueAnimator.ofFloat(0f, 1f).apply {
            duration = 800
            interpolator = DecelerateInterpolator()
            addUpdateListener {
                animProgress = it.animatedValue as Float
                invalidate()  // 触发重绘
            }
        }.start()
    }
}
```

**关键点：** 用`ValueAnimator`驱动进度值，在`onDraw`中根据进度计算Y坐标，配合`DecelerateInterpolator`实现先快后慢的自然感。

------

## 6. 大文件上传到服务器：可靠性 + 安全性 + 效率

### 三维方案设计

**① 效率：分片上传（Multipart Upload）**

```
文件切片（每片 2~5MB）
    ↓
并发上传多个分片（线程池控制并发数，如3~5个）
    ↓
服务端合并分片
```

**② 可靠性：断点续传**

```kotlin
// 本地记录已上传的分片索引
val uploadedChunks = sharedPrefs.getStringSet("uploaded_chunks", mutableSetOf())

chunks.forEach { chunk ->
    if (chunk.index.toString() in uploadedChunks!!) return@forEach  // 跳过已上传
    uploadChunk(chunk)
    uploadedChunks.add(chunk.index.toString())
    sharedPrefs.edit().putStringSet("uploaded_chunks", uploadedChunks).apply()
}
```

**③ 安全性**

- 传输层：强制 **HTTPS/TLS 1.3**
- 完整性校验：每个分片上传时携带 **MD5/SHA-256** 校验值，服务端验证
- 身份认证：请求头携带 **JWT Token**，防止未授权上传
- 服务端合并后再次校验整体文件Hash

**整体流程：**

```
初始化上传（获取uploadId） → 并发分片上传（携带MD5） → 轮询/回调确认 → 服务端合并验证
```

------

## 7. ANR 原理 + 内存对ANR的影响

### ANR 触发条件

| 场景                        | 超时时间             |
| --------------------------- | -------------------- |
| 主线程输入事件（触摸/按键） | **5秒**              |
| BroadcastReceiver onReceive | 前台10秒 / 后台60秒  |
| Service 生命周期方法        | 前台20秒 / 后台200秒 |

### 原理

- ActivityManagerService（AMS）通过**埋定时炸弹**机制：发送消息前设定Timeout Handler，若规定时间内未收到完成信号，触发ANR流程
- 收集主线程堆栈 + `/data/anr/traces.txt`

### 内存高 → 会影响ANR！

```
内存不足
  → 频繁GC（Stop-The-World，暂停所有线程）
  → 主线程被GC暂停时间累计过长
  → 同时触发内存Trim，系统回收资源占用CPU
  → 主线程响应超时 → ANR
```

**结论：** 内存压力大时，GC停顿 + 系统资源争抢 **会间接导致ANR**，要特别关注内存泄漏和大对象分配。

------

## 8. 图片库本地存储大文件 + 过期策略

### 常见策略组合

**LRU（最近最少使用）+ 时间过期：** 这是Glide/Picasso的基础策略

**除时间过期外，其他淘汰策略：**

| 策略                        | 说明                                    | 适用场景           |
| --------------------------- | --------------------------------------- | ------------------ |
| **LRU（大小）**             | 缓存总大小超限时淘汰最久未使用          | 通用图片缓存       |
| **LFU（访问频率）**         | 淘汰访问次数最少的文件                  | 热点数据明显的场景 |
| **文件大小权重**            | 优先淘汰体积大的文件以快速释放空间      | 存储敏感场景       |
| **优先级标记**              | 手动标记文件重要程度，低优先级先淘汰    | 业务区分缓存       |
| **磁盘水位线**              | 磁盘占用超过阈值（如总空间80%）触发清理 | 防止撑爆磁盘       |
| **弱引用（WeakReference）** | 内存紧张时由GC自动回收，无需手动管理    | 内存缓存层         |

**DiskLruCache（推荐）**：OkHttp和Glide磁盘缓存的底层实现，基于Journal日志记录LRU顺序，稳定可靠。

------

## 9. App 启动性能优化

### 启动类型

- **冷启动**：进程不存在，从零开始（最慢，优化重点）
- **温启动**：进程存在但Activity被销毁
- **热启动**：进程和Activity均存在

### 优化切入点

```
Application.onCreate()
├── 懒加载：非必要SDK延迟初始化（如推送、统计）
├── 异步初始化：用线程池并行初始化无依赖的模块
└── 使用 App Startup 库统一管理初始化顺序

MainActivity.onCreate()
├── 布局优化：减少层级，用ViewStub延迟加载不可见区域
├── 避免主线程IO（SP、数据库查询移至子线程）
└── 首屏数据预加载（在SplashActivity期间并行请求）

视觉优化：
└── 设置 windowBackground 为品牌色/启动图，消除白屏黑屏
```

**度量工具：** `adb shell am start -W packageName/.MainActivity` 查看 `TotalTime`；Android Studio Profiler的App Startup模块。

------

## 10. UI 渲染性能优化

### 核心目标：保持16ms/帧（60fps）

**诊断工具：**

- `GPU呈现模式分析`（开发者选项）：柱状图超过绿线（16ms）即为卡顿
- `Layout Inspector`：检查布局层级
- `Systrace/Perfetto`：帧耗时的精细分析

### 优化手段

```
布局层级优化
├── 用ConstraintLayout替代多层嵌套LinearLayout
├── merge标签消除冗余根布局
└── ViewStub延迟加载不常显示的区域

过度绘制优化
├── 开发者选项开启"显示过度绘制"，消除红色区域
├── 移除不必要的background
└── clipRect()限制Canvas绘制范围（自定义View）

列表优化
├── RecyclerView：DiffUtil精准刷新，避免notifyDataSetChanged()
├── 图片按需加载，压缩到ImageView实际尺寸
└── 复杂item使用Prefetch预取

主线程卸载
└── 耗时操作（解码、IO）移至协程IO调度器
```

------

## 11. Activity 启动流程 + 四大组件 + 跨进程通信

### Activity 启动流程（简化版）

```
App调用 startActivity()
    ↓
ActivityManagerService（AMS）通过Binder接收请求
    ↓
检查目标Activity是否存在、权限校验
    ↓
若目标进程不存在 → 通知Zygote fork新进程
    ↓
新进程启动 ActivityThread.main()
    ↓
ApplicationThread（Binder）通知主线程创建Activity
    ↓
执行 onCreate → onStart → onResume
    ↓
ViewRootImpl与WMS建立连接，进行measure/layout/draw
```

### 四大组件

- **Activity**：用户界面，任务栈管理
- **Service**：后台任务（前台Service保活）
- **BroadcastReceiver**：事件广播（静态/动态注册）
- **ContentProvider**：数据共享，跨进程数据访问的标准接口

### 跨进程通信（IPC）方式

| 方式                  | 特点                                            |
| --------------------- | ----------------------------------------------- |
| **Binder**            | Android核心IPC，AMS/WMS底层，一次拷贝，高效安全 |
| **AIDL**              | 基于Binder的接口定义，支持复杂对象传递          |
| **Messenger**         | 基于AIDL的简化版，串行消息队列                  |
| **ContentProvider**   | 专为数据共享设计，支持CRUD                      |
| **BroadcastReceiver** | 一对多，低耦合但不可靠                          |
| **Socket/管道**       | 底层通信，较少直接使用                          |

------

## 12. 协程是什么

### 本质

协程是**可挂起的轻量级任务**，不是线程，运行在线程之上。挂起时不阻塞线程，线程可去处理其他协程。

```kotlin
// 对比：传统回调地狱
api.getUser { user ->
    api.getOrders(user.id) { orders ->
        api.getDetail(orders[0].id) { detail ->
            // 嵌套地狱
        }
    }
}

// 协程：同步写法完成异步操作
viewModelScope.launch {
    val user = api.getUser()          // 挂起，不阻塞线程
    val orders = api.getOrders(user.id)
    val detail = api.getDetail(orders[0].id)
    // 线性清晰
}
```

### 关键概念

- **CoroutineScope**：协程的生命周期范围，`viewModelScope`随ViewModel销毁自动取消
- **Dispatcher**：调度器，`Main`（主线程）/ `IO`（网络/磁盘）/ `Default`（CPU密集）
- **suspend**：标记可挂起函数，只能在协程或其他suspend函数中调用
- **挂起原理**：编译器将suspend函数转换为状态机，本质是`Continuation`的回调

------

## 13. 内存泄漏的常见场景

```kotlin
// 1. 静态变量持有Context
companion object {
    var activity: Activity? = null  // ❌ Activity无法被GC
}

// 2. 单例持有View/Activity引用
object Manager {
    var view: View? = null  // ❌
}

// 3. 未注销的监听器/广播
override fun onResume() {
    EventBus.getDefault().register(this)  // ✅ 记得在onPause注销
}

// 4. Handler内存泄漏（非静态内部类持有外部类引用）
val handler = object : Handler(Looper.getMainLooper()) {
    override fun handleMessage(msg: Message) {
        // 隐式持有Activity引用，延迟消息导致Activity无法回收
    }
}
// ✅ 改用弱引用 + 静态内部类

// 5. 协程/线程未取消
// ✅ 使用viewModelScope，自动跟随生命周期取消

// 6. Bitmap未释放（低版本Android）
// 7. WebView内存泄漏（用独立进程或手动destroy）
```

**检测工具：** LeakCanary（自动检测并输出引用链）、Android Studio Memory Profiler

------

## 14. Hook 了解吗

### Hook 是什么

在不修改原始代码的前提下，**拦截并修改程序的执行流程**。

### Android常见Hook方案

**① 反射/动态代理（Java层）**

```kotlin
// Hook ActivityManagerService，拦截startActivity
val iActivityManagerClass = Class.forName("android.app.IActivityManager")
Proxy.newProxyInstance(loader, arrayOf(iActivityManagerClass)) { _, method, args ->
    if (method.name == "startActivity") {
        // 拦截并修改参数
    }
    method.invoke(original, *args!!)
}
```

**② AspectJ（AOP面向切面）**

- 编译期织入，无运行时开销
- 常用于埋点、日志、性能监控

**③ native Hook（PLT Hook / Inline Hook）**

- 框架：`bhook`、`xhook`、`shadowhook`
- 拦截native层函数，如内存分配、IO操作

**④ Xposed框架**（需root）

- 替换`app_process`，在方法执行前后注入代码

**实际应用场景：**

- 无侵入式埋点
- 线上崩溃监控（Hook Crash Handler）
- 性能监控（Hook帧绘制、IO耗时）
- 热修复（Hook类加载）

------

## 15. 埋点：手动 vs AI辅助提示词

### 手动埋点 vs 无侵入埋点

| 方式          | 优点                   | 缺点                       |
| ------------- | ---------------------- | -------------------------- |
| 手动埋点      | 精准，灵活，含业务语义 | 开发成本高，容易遗漏       |
| AOP无侵入埋点 | 代码解耦，统一管理     | 难以携带业务参数           |
| 可视化埋点    | 运营可自助配置         | 仅能捕捉点击，无法捕捉逻辑 |

### 让 AI 帮你写埋点的提示词模板

```
【角色】
你是一名熟悉Android开发和数据分析的高级工程师。

【任务】
根据我提供的业务页面描述和代码，帮我生成完整的埋点代码。

【要求】
1. 埋点点位：页面曝光（onResume）、关键按钮点击、核心业务事件（如支付成功/失败）
2. 埋点格式：事件名用下划线命名（如 btn_checkout_click），
   属性包含：page_name、element_id、timestamp、user_id、业务相关字段
3. 使用我们的埋点工具类：AnalyticsManager.track(eventName, properties)
4. 对每个埋点写一行注释说明其业务含义
5. 不要修改原有业务逻辑，只在合适的生命周期/点击回调处添加埋点

【输入代码】
[粘贴你的Activity/Fragment/ViewModel代码]

【业务背景】
[描述这个页面的核心功能，如：这是一个商品详情页，用户可以查看商品、加购、立即购买]

【期望输出】
完整的带埋点的代码文件，并在文末输出所有埋点的清单表格（事件名、触发时机、关键属性）。
```

**提示词关键要素：** 指定角色 → 明确格式规范 → 提供代码上下文 → 描述业务背景 → 要求输出清单，这样AI生成的埋点既符合规范，又有业务语义，review成本极低。