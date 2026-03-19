#  哔哩哔哩面经

1. 自我介绍
2. 实习平时用ai怎么用的
3. 讲一讲你实习的适配器模式
4. 什么情况下是会用适配器模式的  (需要一个统一的接口) 面试官 额……
5. 讲一讲你实习自定义view的经历
6. 讲一讲谷歌新东西mvi 在讲一讲mvvm 
7. 我们怎么合理的用viewmodel
8. 讲一讲livedata是怎么观察他的生命周期的 忘了不会 面试官直接笑哈哈哈
9. 讲一讲livedata/stateflow的粘性 说的一塌糊涂面试官又笑了（最后他告诉我了下一次面试有人问就说用旧值通知新值）
10. viewmode他的原理是怎么保存状态的，他的生命周期为什么比activity长 忘了面试官又笑哈哈哈
11. 讲一讲多线程安全是什么，有什么特性，详细讲讲每个特性
12. 讲一讲cas 
13. 你刚刚提到自旋，自旋又是什么概念 瞎说一顿面试官又笑了
14. synchronized 怎么用 修饰静态方法锁的是什么，修饰普通的方法锁的是什么
15.  synchronized 升级过程 升级到轻量级锁也就是有cas了 synchronized他主要是干了什么 不大会他又笑了
16. 讲一讲垃圾回收机制
17. 你知道的垃圾回收器有什么
18. 讲一讲对象的生命周期 忘了 瞎说一顿面试官又笑了
19. 谈hashmap 他为什么变成8 会变成红黑树 6回退到链表 (我说是因为泊松分布他说这个不谈，最后告诉我主要是因为7 8是一个临界值可能会发生奇怪的事情) 他是怎么扩容的，他线程不安全吗，线程安全用什么，comap,还有别的吗hashtable，hashtable是怎么保证线程安全的，加锁，他锁的是什么hashmap线程不安全多线程下会发生什么奇怪的东西
20. activity的生命周期，a跳b会发生啥，一定是这个流程吗
21. 反问 面试官 他讲了我好多回答错的东西(谢谢)

------

## 3. 适配器模式

**定义：** 将一个类的接口转换为客户端期望的另一个接口，使原本不兼容的类可以一起工作。

**三个角色：**

- Target（目标接口）：客户端期望的接口
- Adaptee（被适配者）：已有的、不兼容的类
- Adapter（适配器）：包装 Adaptee，实现 Target 接口

```kotlin
// 已有旧接口
class OldPlayer {
    fun playMp3(file: String) { ... }
}

// 目标接口
interface MediaPlayer {
    fun play(file: String)
}

// 适配器
class MediaAdapter(private val old: OldPlayer) : MediaPlayer {
    override fun play(file: String) = old.playMp3(file)
}
```

实习场景举例：对接多个第三方SDK时，各自接口不统一，用适配器统一成一套内部接口。

------

## 4. 什么情况下用适配器模式

面试官"额……"说明答"需要统一接口"太泛，应该更具体：

- 老系统接口不想改，但新模块需要用它
- 接入多个第三方库，接口各不相同，需要统一抽象
- Android中 `RecyclerView.Adapter` 就是典型——把数据源适配成 View

关键词：**兼容已有代码、解耦、第三方集成**



------

## 6. MVI vs MVVM

**MVVM：**

- Model - View - ViewModel
- View 通过 DataBinding 或 observe LiveData/StateFlow 响应数据
- ViewModel 持有状态，View 和 Model 双向可交互
- 缺点：State 分散，多个 LiveData 可能状态不一致

**MVI（Google 近年推崇）：**

- Model - View - Intent
- 单向数据流：**View 发出 Intent → ViewModel 处理 → 产生新 State → View 渲染**
- State 是**不可变的单一对象**，每次更新都是一个新 State
- 优点：状态可预测、易于调试、便于测试

```
用户点击 → Intent.LoadData
    → ViewModel 处理
    → emit UiState(loading=false, data=list)
    → View 根据 UiState 整体渲染
```

------

## 7. 怎么合理使用 ViewModel

- ViewModel **只持有UI状态**，不持有 View / Context 引用（防内存泄漏）
- 需要 Context 用 `AndroidViewModel`，但也要谨慎
- 异步操作放在 `viewModelScope` 里，自动跟随生命周期取消
- 多个 Fragment 共享数据时，用 `activityViewModels()` 共享同一个 ViewModel
- **不要在 ViewModel 里写 UI 逻辑**，只负责数据处理和状态维护

------

## 8. LiveData 如何观察生命周期

这是 **Lifecycle-aware** 的核心，面试官笑是因为这是 Jetpack 基础。

原理：

- LiveData 内部持有一个 `LifecycleBoundObserver`
- 它实现了 `LifecycleEventObserver`，注册到宿主（Activity/Fragment）的 `Lifecycle`
- 当生命周期变为 `DESTROYED` 时，**自动移除观察者**，避免内存泄漏
- 只在 `STARTED` / `RESUMED` 状态下回调，避免在后台更新UI崩溃

```kotlin
liveData.observe(viewLifecycleOwner) { value ->
    // 只在 STARTED 以上才会触发
}
```

------

## 9. StateFlow 的粘性问题

面试官给的答案：**用旧值通知新值**，展开解释：

**什么是粘性：** StateFlow 是有状态的，新订阅者订阅时会**立即收到当前最新值**，这就是"粘性"。

**为什么：** StateFlow 底层维护一个 `value`，collect 时会先 emit 当前 value，再监听后续变化。

**对比 SharedFlow：** SharedFlow 默认 `replay=0`，新订阅者不会收到旧值，可以用来处理一次性事件（如 Toast、导航）。

**实际应用建议：**

- UI状态（页面数据）→ 用 StateFlow，粘性是合理的
- 一次性事件（弹窗、跳转）→ 用 SharedFlow(replay=0)

------

## 10. ViewModel 的原理：为什么生命周期比 Activity 长

面试官笑说明这是高频考点。

**原理：**

- ViewModel 存储在 `ViewModelStore` 中，而 `ViewModelStore` 由 `ViewModelStoreOwner` 持有
- Activity 实现了 `ViewModelStoreOwner`
- 关键：Activity 因**配置变更**（旋转屏幕）重建时，系统会在销毁前调用 `onRetainNonConfigurationInstance()` 保存 `ViewModelStore`
- 新的 Activity 实例通过 `getLastNonConfigurationInstance()` 恢复同一个 `ViewModelStore`
- 所以 ViewModel 实例被复用了

**真正销毁时机：** `onDestroy` 且不是配置变更导致的（即用户真正退出），此时调用 `viewModelStore.clear()`

------

## 11. 多线程安全的特性

三大特性（Java内存模型 JMM）：

| 特性       | 含义                                                         |
| ---------- | ------------------------------------------------------------ |
| **原子性** | 操作不可分割，要么全做要么不做。`synchronized`、`Atomic` 类保证 |
| **可见性** | 一个线程修改变量，其他线程能立即看到。`volatile`、`synchronized` 保证 |
| **有序性** | 代码执行顺序与编写顺序一致，禁止指令重排。`volatile`、`happens-before` 规则保证 |

------

## 12. CAS（Compare And Swap）

- **含义：** 比较并交换，是一种**无锁**的原子操作
- **三个操作数：** 内存地址V、预期值A、新值B
- **逻辑：** 如果 V 当前值 == A，则把 V 更新为 B，否则不操作
- **Java 实现：** `Unsafe` 类提供，`AtomicInteger` 等底层都用 CAS
- **ABA 问题：** 值从 A 改成 B 再改回 A，CAS 感知不到，解决方案是加版本号（`AtomicStampedReference`）

------

## 13. 自旋

- **概念：** 线程获取锁失败时，**不挂起**，而是在循环中不断重试（"空转"），等待锁释放
- **优点：** 避免线程切换的开销（线程切换需要保存/恢复上下文，成本高）
- **缺点：** 如果锁被持有时间长，自旋会一直占用CPU，浪费资源
- **适用场景：** 锁持有时间**极短**的场景
- **JVM 实现：** 默认自旋次数约10次，超过后升级为重量级锁挂起线程

------

## 14. synchronized 的用法和锁对象

| 用法             | 锁的是什么                         |
| ---------------- | ---------------------------------- |
| 修饰**普通方法** | 当前实例对象（`this`）             |
| 修饰**静态方法** | 当前类的 Class 对象（`XXX.class`） |
| 修饰**代码块**   | 括号内指定的对象                   |

```kotlin
synchronized(this) { }          // 锁实例
synchronized(MyClass::class.java) { }  // 锁Class对象
```

------

## 15. synchronized 锁升级过程

面试官笑是因为这是 JVM 底层重点。

**四个状态（单向升级，不可降级）：**

1. **无锁**：没有竞争
2. **偏向锁**：只有一个线程访问，在对象头记录线程ID，下次该线程进来无需 CAS，几乎零开销
3. **轻量级锁**：有第二个线程来竞争，升级为轻量级锁，**通过 CAS 将锁记录写入对象头（Mark Word）**，失败则自旋等待
4. **重量级锁**：自旋超过阈值或等待线程过多，升级为重量级锁，线程挂起，依赖操作系统 Mutex，性能最差

**synchronized 核心做了什么：** 修改对象头的 **Mark Word**，记录锁状态和持有线程信息。

------

## 16. 垃圾回收机制

**如何判断对象可以被回收：**

- ~~引用计数法~~（有循环引用问题）
- **可达性分析**：从 GC Roots 出发，不可达的对象可回收

**GC Roots 包括：** 栈帧中的局部变量、静态变量、JNI引用等

**回收算法：**

- **标记-清除**：标记后直接清除，产生内存碎片
- **标记-整理**：清除后整理内存，无碎片，但慢
- **复制算法**：存活对象复制到另一块内存，效率高，适合新生代（存活率低）
- **分代收集**：新生代用复制，老年代用标记-整理

------

## 17. 常见垃圾回收器

| 回收器 | 特点                               |
| ------ | ---------------------------------- |
| Serial | 单线程，STW，适合客户端            |
| ParNew | Serial 的多线程版                  |
| CMS    | 并发标记清除，低停顿，但有碎片     |
| G1     | 分Region，可预测停顿时间，JDK9默认 |
| ZGC    | 超低延迟，停顿 < 10ms，JDK15+      |

Android 使用的是 **ART 虚拟机**，GC 机制有所不同，采用并发复制回收器（CC）。

------

## 18. 对象的生命周期

Java 对象从创建到回收：

1. **类加载**：首次使用时 JVM 加载 .class 文件
2. **实例创建**：`new` 关键字，在堆上分配内存，初始化零值，调用构造方法
3. **使用中**：被引用，GC Roots 可达
4. **不可达**：没有引用指向它，等待 GC
5. **`finalize()`**：第一次GC前调用（不推荐依赖），对象可能"复活"
6. **真正回收**：第二次GC时被回收内存

------

## 19. HashMap 相关问题

**为什么链表长度8转红黑树，6退回链表：**

- 面试官说得对：7/8 是临界区间，避免频繁转换（如果阈值一样，增删操作会反复转换）
- 长度7时不转换，留缓冲区，减少抖动

**扩容机制：**

- 初始容量16，负载因子0.75，超过 `16 * 0.75 = 12` 时扩容
- 扩容为**2倍**，重新 hash 所有元素

**线程不安全：** 多线程下扩容时可能形成**环形链表**（JDK7），导致死循环；JDK8改为尾插法，但仍有数据丢失问题

**线程安全方案：**

- `ConcurrentHashMap`：分段锁（JDK7）/ CAS + synchronized 锁单个桶头节点（JDK8）
- `Hashtable`：方法全加 `synchronized`，锁的是**整个对象**，性能差，几乎废弃

------

## 20. Activity 生命周期：A 跳 B

**标准流程：**

```
A.onPause() → B.onCreate() → B.onStart() → B.onResume() → A.onStop()
```

**不一定是这个流程的情况：**

- B 是**透明 Activity** 或**Dialog样式**：A 不会走 `onStop()`，只走 `onPause()`
- B 的 `launchMode` 是 `singleTop` 且 B 已在栈顶：走 `onNewIntent()` 而非重新创建
- 开启了**硬件加速**或配置变更：流程有细微差异
- A 调用 `finish()`：A 的 `onStop` → `onDestroy` 也会执行





