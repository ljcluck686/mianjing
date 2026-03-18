# 友邦咨询

1. mvc mvvm
2. activity生命周期，如何在activity不重建的情况下进行屏幕旋转，viewmodel 请求数据是应该在activity生命周期的哪个阶段

我来回答这两个Android面试题。

------

## 1. MVC vs MVVM

### MVC (Model-View-Controller)

```
View ←→ Controller ←→ Model
  ↑_________________________↑
```

| 层级           | 职责                            |
| -------------- | ------------------------------- |
| **Model**      | 数据层，业务逻辑                |
| **View**       | UI展示（XML布局）               |
| **Controller** | Activity/Fragment，处理用户交互 |

**问题：** Activity既是View又是Controller，导致Activity臃肿，难以测试，View和Model耦合严重。

------

### MVVM (Model-View-ViewModel)

```
View ←(观察)→ ViewModel ←→ Model
   (数据绑定/LiveData/StateFlow)
```

| 层级          | 职责                                     |
| ------------- | ---------------------------------------- |
| **Model**     | 数据层，Repository，网络/数据库          |
| **View**      | Activity/Fragment，只负责UI渲染          |
| **ViewModel** | 持有UI状态，处理业务逻辑，不持有View引用 |

**优势：**

- View与Model完全解耦
- ViewModel不持有View引用，**可配置变更后存活**（屏幕旋转不销毁）
- 易于单元测试
- 配合LiveData/StateFlow实现响应式UI

------

## 2. Activity 生命周期

### 完整生命周期

```
        启动
         ↓
      onCreate()        ← 初始化，setContentView
         ↓
      onStart()         ← 可见，不可交互
         ↓
      onResume()        ← 可见，可交互（前台运行）
         ↓
    [用户操作/系统事件]
         ↓
      onPause()         ← 部分遮挡，不可交互
         ↓
      onStop()          ← 完全不可见
         ↓
      onDestroy()       ← 销毁
```

------

### 屏幕旋转不重建 Activity

在 `AndroidManifest.xml` 中配置：

```xml
<activity
    android:name=".MainActivity"
    android:configChanges="orientation|screenSize|keyboardHidden" />
```

然后在 Activity 中重写：

```kotlin
override fun onConfigurationChanged(newConfig: Configuration) {
    super.onConfigurationChanged(newConfig)
    // 手动处理旋转后的UI调整
    if (newConfig.orientation == Configuration.ORIENTATION_LANDSCAPE) {
        // 横屏处理
    } else {
        // 竖屏处理
    }
}
```

**原理：** 加了 `configChanges` 后，系统不再销毁重建 Activity，而是回调 `onConfigurationChanged()`，开发者自行处理UI变化。

------

### ViewModel 请求数据应在哪个阶段？

**推荐：在 `onCreate()` 中触发，由 ViewModel 发起请求。**

```kotlin
class MainActivity : AppCompatActivity() {

    private val viewModel: MainViewModel by viewModels()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        // ✅ 在 onCreate 观察数据
        viewModel.data.observe(this) { data ->
            // 更新UI
        }

        // ✅ 数据请求在 ViewModel 的 init 块中自动触发
        // 无需手动调用，旋转屏幕也不会重复请求
    }
}

class MainViewModel : ViewModel() {
    val data: LiveData<Result> = liveData {
        // ✅ 只在 ViewModel 首次创建时执行
        emit(repository.fetchData())
    }
}
```

**为什么是 `onCreate` 而不是其他？**

| 阶段       | 是否合适   | 原因                                          |
| ---------- | ---------- | --------------------------------------------- |
| `onCreate` | ✅ **推荐** | 只调用一次（旋转不重建ViewModel），早于UI展示 |
| `onStart`  | ⚠️ 不推荐   | 每次回到前台都会调用，导致重复请求            |
| `onResume` | ❌ 不推荐   | 调用更频繁，如弹窗消失也会触发                |

**核心原因：** ViewModel 的生命周期比 Activity 长，屏幕旋转时 Activity 重建，但 **ViewModel 实例不变**，`init` 块或 `liveData {}` 构建器只执行一次，天然防止重复请求。



