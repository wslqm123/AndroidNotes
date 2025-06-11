# AndroidNotes
**Android 复习知识树**

```
Android 复习知识树
├── 1. 核心基础 (Java & Kotlin)
│   ├── 1.1 Java 基础 
│   │   ├── [ ] 1.1.1 面向对象 (封装、继承、多态) (深入)
│   │   ├── [ ] 1.1.2 抽象类与接口 (对比与应用) (*)
│   │   ├── [ ] 1.1.3 Java 集合框架详解 (List, Set, Map 实现类特性与原理) (*)
│   │   ├── [ ] 1.1.4 HashMap 源码/原理 (哈希冲突, 扩容, 线程安全) (*)
│   │   ├── [ ] 1.1.5 Java 泛型深入 (类型擦除, 通配符) (*)
│   │   ├── [ ] 1.1.6 Java 反射原理与应用 (*)
│   │   ├── [ ] 1.1.7 Java I/O (BIO, NIO, AIO 概念) (*)
│   │   ├── [ ] 1.1.8 Java 多线程与并发 (Thread, Runnable, Callable, 线程池, synchronized, Lock, volatile, AQS) (*)
│   │   └── [ ] 1.1.9 JVM 内存模型与 GC 机制 (概览) (*)
│   ├── 1.2 Kotlin 核心特性 (重点)
│   │   ├── [ ] 1.2.1 Kotlin 与 Java 互操作性 (深入)
│   │   ├── [ ] 1.2.2 Kotlin 空安全机制 (? !!, ?., ?:, let 等) (深入)
│   │   ├── [ ] 1.2.3 Kotlin 函数式编程 (Lambda, 高阶函数) (深入)
│   │   ├── [ ] 1.2.4 Kotlin 扩展函数与属性 (深入)
│   │   ├── [ ] 1.2.5 Kotlin 数据类 (data class) 与密封类 (sealed class) (深入)
│   │   ├── [ ] 1.2.6 Kotlin 对象声明与伴生对象 (深入)
│   │   └── [ ] 1.2.7 Kotlin 委托属性 (by lazy, by Delegates.observable 等) (深入)
│   ├── 1.3 Kotlin 协程 (Coroutines)
│   │   ├── [X] 1.3.1 协程基础 (suspend, CoroutineScope, Job, Deferred, Dispatchers) (原1.2.8，已概览)
│   │   ├── [X] 1.3.2 结构化并发与协程生命周期管理 (深入) (*)
│   │   ├── [X] 1.3.3 协程上下文 (CoroutineContext) 与调度器 (Dispatchers) 详解 (*)
│   │   ├── [X] 1.3.4 协程的取消与异常处理 (深入) (*)
│   │   ├── [X] 1.3.5 使用 async 和 await 进行并发 (深入) (*)
│   │   └── [X] 1.3.6 Channel 与协程间通信 (*)
│   └── 1.4 Kotlin Flow (P0级)
│       ├── [X] 1.4.1 Flow 基础 (冷流 vs 热流, 构建 Flow, 末端操作符)
│       ├── [X] 1.4.2 Flow 中间操作符 (map, filter, transform, onEach, debounce, etc.)
│       ├── [X] 1.4.3 Flow 组合与扁平化操作符 (zip, combine, flatMapConcat, flatMapMerge, flatMapLatest)
│       ├── [X] 1.4.4 Flow 的异常处理 (catch, onCompletion, retry)
│       ├── [X] 1.4.5 Flow 的背压 (Backpressure) 处理策略
│       ├── [X] 1.4.6 Flow 冷热流转换与共享 (shareIn, stateIn)
│       └── [X] 1.4.7 Flow 与 Android UI 交互最佳实践 (collect, repeatOnLifecycle, asLiveData, Compose)
│
├── 2. Android 核心组件
│   ├── 2.1 Activity
│   │   ├── [ ] 2.1.1 Activity 生命周期 (各回调方法) (深入)
│   │   ├── [ ] 2.1.2 Activity 启动模式与任务栈 (深入)
│   │   ├── [ ] 2.1.3 Activity 状态保存与恢复 (onSaveInstanceState, ViewModel) (深入)
│   │   └── [ ] 2.1.4 Activity 间通信 (Intent, Bundle, ActivityResultLauncher) (深入)
│   ├── 2.2 Fragment
│   │   ├── [ ] 2.2.1 Fragment 生命周期与 Activity 关系 (深入)
│   │   ├── [ ] 2.2.2 Fragment 与 Activity 通信方式 (深入)
│   │   └── [ ] 2.2.3 FragmentManager 与 FragmentTransaction (深入)
│   ├── 2.3 Service
│   │   ├── [ ] 2.3.1 Service 生命周期与启动方式 (startService, bindService) (深入)
│   │   ├── [ ] 2.3.2 前台服务 (Foreground Service) (深入)
│   │   ├── [ ] 2.3.3 IntentService (概念与替代方案) (深入)
│   │   └── [ ] 2.3.4 AIDL与Binder机制 (IPC核心) (*)
│   ├── 2.4 BroadcastReceiver
│   │   ├── [ ] 2.4.1 静态注册与动态注册 (深入)
│   │   ├── [ ] 2.4.2 有序广播与无序广播 (深入)
│   │   └── [ ] 2.4.3 本地广播 (LocalBroadcastManager - 废弃与替代) (*)
│   ├── 2.5 ContentProvider
│   │   ├── [ ] 2.5.1 ContentProvider 作用与 URI (深入)
│   │   └── [ ] 2.5.2 ContentResolver 使用 (深入)
│   └── 2.6 Handler 消息机制
│       ├── [ ] 2.6.1 Handler, Looper, MessageQueue, Message 核心原理与应用 (深入)
│       ├── [ ] 2.6.2 Handler 内存泄漏分析与最佳实践 (*)
│       └── [ ] 2.6.3 HandlerThread 与 AsyncTask (缺陷与现代替代方案回顾) (*)
│
├── 3. UI 与视图系统
│   ├── 3.1 View 绘制
│   │   ├── [ ] 3.1.1 View 绘制流程 (measure, layout, draw) (深入)
│   │   └── [ ] 3.1.2 requestLayout, invalidate, postInvalidate 区别 (深入)
│   ├── 3.2 常用布局
│   │   ├── [ ] 3.2.1 LinearLayout, RelativeLayout, FrameLayout 特性与使用 (深入)
│   │   ├── [ ] 3.2.2 ConstraintLayout (特性、优势、使用技巧) (深入)
│   │   └── [ ] 3.2.3 CoordinatorLayout 与 AppBarLayout, CollapsingToolbarLayout (深入)
│   ├── 3.3 RecyclerView
│   │   ├── [ ] 3.3.1 RecyclerView 核心组件 (Adapter, ViewHolder, LayoutManager) (深入)
│   │   ├── [ ] 3.3.2 RecyclerView 优化 (DiffUtil, 局部刷新, 分页加载等) (深入)
│   │   └── [ ] 3.3.3 RecyclerView 多种 Item 类型处理 (深入)
│   ├── 3.4 自定义 View
│   │   ├── [ ] 3.4.1 自定义 View 概览 (继承 View/ViewGroup, 核心方法) (深入)
│   │   └── [ ] 3.4.2 自定义属性 (attrs.xml, TypedArray) (深入)
│   ├── 3.5 事件机制
│   │   ├── [ ] 3.5.1 View 事件分发机制 (dispatchTouchEvent, onInterceptTouchEvent, onTouchEvent) (深入)
│   │   └── [ ] 3.5.2 滑动冲突解决 (深入)
│   ├── 3.6 动画
│   │   ├── [ ] 3.6.1 Android 动画概览 (属性动画 vs 视图动画) (深入)
│   │   ├── [ ] 3.6.2 属性动画 (ValueAnimator, ObjectAnimator, AnimatorSet) (深入)
│   │   ├── [ ] 3.6.3 帧动画与Drawable动画 (*)
│   │   ├── [ ] 3.6.4 Transition Framework (转场动画) (*)
│   │   └── [ ] 3.6.5 MotionLayout 简介 (*)
│   ├── 3.7 Material Design Components
│   │   └── [ ] 3.7.1 Material Design Components 简介与常用组件 (深入)
│   └── 3.8 Jetpack Compose
│       ├── [ ] 3.8.1 Compose 核心思想 (声明式UI, @Composable 函数, 状态驱动) (*)
│       ├── [ ] 3.8.2 Compose 布局 (Column, Row, Box, ConstraintLayout, Modifiers) (*)
│       ├── [ ] 3.8.3 Compose 状态管理 (remember, mutableStateOf, ViewModel 与 State hoisting) (*)
│       ├── [ ] 3.8.4 Compose 副作用处理 (LaunchedEffect, SideEffect, DisposableEffect) (*)
│       ├── [ ] 3.8.5 Compose 主题与样式 (MaterialTheme, Typography, Colors, Shapes) (*)
│       ├── [ ] 3.8.6 Compose 动画与手势 (*)
│       ├── [ ] 3.8.7 Compose 导航 (Navigation for Compose) (*)
│       ├── [ ] 3.8.8 Compose 与传统 View 互操作 (*)
│       └── [ ] 3.8.9 Compose 性能优化与调试技巧 (*)
│
├── 4. 数据存储与处理
│   ├── 4.1 SharedPreferences
│   │   └── [ ] 4.1.1 SharedPreferences 使用与注意事项 (深入)
│   ├── 4.2 文件存储
│   │   └── [ ] 4.2.1 文件存储 (内部存储, 外部私有/公共, 分区存储) (深入)
│   ├── 4.3 SQLite & Room
│   │   ├── [ ] 4.3.1 Room 组件 (Entity, DAO, Database) (深入)
│   │   ├── [ ] 4.3.2 Room 数据库迁移 (Migration) (*)
│   │   └── [ ] 4.3.3 Room 类型转换器 (TypeConverter) 与关系查询 (@Relation) (*)
│   └── 4.4 DataStore (Jetpack)
│       └── [ ] 4.4.1 DataStore 概览 (Preferences DataStore vs Proto DataStore) (深入)
│
├── 5. 网络编程
│   ├── 5.1 HTTP/HTTPS 基础
│   │   └── [ ] 5.1.1 HTTP/HTTPS 基础概念 (请求方法, 状态码, URL, 头部) (深入)
│   ├── 5.2 OkHttp
│   │   ├── [ ] 5.2.1 OkHttp 核心功能与优势 (深入)
│   │   └── [ ] 5.2.2 OkHttp 拦截器 (Interceptor) 机制与应用 (深入)
│   ├── 5.3 Retrofit
│   │   ├── [ ] 5.3.1 Retrofit 原理 (动态代理, 注解) 与 OkHttp 关系 (深入)
│   │   └── [ ] 5.3.2 Retrofit Converter 与 CallAdapter (深入)
│   ├── 5.4 JSON 解析
│   │   └── [ ] 5.4.1 JSON 解析库概览 (Gson/Moshi/org.json) (深入)
│   ├── 5.5 图片加载库
│   │   ├── [ ] 5.5.1 图片加载库概览 (Glide/Picasso/Coil) (深入)
│   │   └── [ ] 5.5.2 图片加载库缓存策略与生命周期管理 (深入)
│   └── [ ] 5.6 网络安全基础 (HTTPS原理, SSL/TLS, 证书校验, 中间人攻击防范) (*)
│
├── 6. 架构设计与模式
│   ├── 6.1 常用架构模式
│   │   ├── [ ] 6.1.1 MVC, MVP, MVVM 模式概念与对比 (深入)
│   │   └── [ ] 6.1.2 ViewModel 与 LiveData/StateFlow 在 MVVM 中的角色 (深入)
│   ├── 6.2 Clean Architecture
│   │   └── [ ] 6.2.1 Clean Architecture 分层思想概览 (深入)
│   ├── 6.3 SOLID 原则
│   │   └── [ ] 6.3.1 SOLID 原则概览及其在 Android 中的应用 (深入)
│   └── 6.4 依赖注入 (DI)
│       ├── [ ] 6.4.1 依赖注入 (DI) 原理与好处 (深入)
│       └── [ ] 6.4.2 Dagger2/Hilt 核心概念与使用 (Component, Module, Provides, Inject, Scope) (深入)
│
├── 7. 性能优化
│   ├── 7.1 UI 渲染优化
│   │   ├── [ ] 7.1.1 UI 渲染优化方向 (过度绘制, 布局层级) (深入)
│   │   └── [ ] 7.1.2 卡顿检测与分析工具 (Profiler, Systrace/Perfetto, JankStats) (深入)
│   ├── 7.2 内存优化
│   │   ├── [ ] 7.2.1 内存优化方向 (内存泄漏, Bitmap优化, 数据结构) (深入)
│   │   └── [ ] 7.2.2 LeakCanary 原理与使用 (*)
│   ├── 7.3 启动优化
│   │   ├── [ ] 7.3.1 启动优化方向 (冷/温/热启动分析, SplashScreen, Baseline Profiles, App Startup) (深入)
│   ├── 7.4 电量优化
│   │   └── [ ] 7.4.1 电量优化方向 (WakeLock, JobScheduler/WorkManager, Doze, App Standby) (深入)
│   ├── 7.5 APK 大小优化
│   │   └── [ ] 7.5.1 APK 大小优化方向 (ProGuard/R8, 资源优化, App Bundle) (深入)
│   └── 7.6 线程优化
│       ├── [ ] 7.6.1 ANR 分析与解决 (深入)
│       └── [ ] 7.6.2 线程池的合理配置与使用 (*)
│
├── 8. Jetpack 核心组件 (回顾与补充)
│   ├── [ ] 8.1.1 ViewModel 核心作用与生命周期感知 (已在6.1.2中深入，可视为回顾)
│   ├── [ ] 8.2.1 LiveData/Flow (StateFlow, SharedFlow) 可观察数据与生命周期 (已在6.1.2中深入，可视为回顾)
│   ├── [ ] 8.3.1 Navigation Component 核心作用与导航图 (深入)
│   ├── [ ] 8.4.1 WorkManager 核心作用与后台任务调度 (深入)
│   ├── [ ] 8.5.1 Paging 3 核心作用与分页加载 (深入)
│   └── [ ] 8.6.1 Lifecycle-aware Components 概览 (LifecycleObserver, LifecycleOwner) (*)
│   └── [ ] 8.7.1 Data Binding 与 View Binding 概览与对比 (*)
│
├── 9. 测试
│   ├── 9.1 单元测试
│   │   ├── [ ] 9.1.1 单元测试概念 (JUnit, Mockito) (深入)
│   │   └── [ ] 9.1.2 Robolectric 测试框架简介 (*)
│   ├── 9.2 UI 测试 (仪器化测试)
│   │   ├── [ ] 9.2.1 UI 测试概念 (Espresso, UI Automator) (深入)
│   └── [ ] 9.3 测试金字塔与TDD/BDD理念 (*)
│
├── 10. 构建与发布
│   ├── 10.1 Gradle 构建系统
│   │   ├── [ ] 10.1.1 Gradle 基础 (build.gradle, 依赖管理) (深入)
│   │   └── [ ] 10.1.2 Gradle Build Types 与 Product Flavors (深入)
│   │   └── [ ] 10.1.3 自定义 Gradle Task 与 Plugin 简介 (*)
│   ├── 10.2 代码混淆与优化
│   │   └── [ ] 10.2.1 ProGuard / R8 (代码混淆、优化、移除未使用代码) (深入)
│   ├── 10.3 签名与发布
│   │   └── [ ] 10.3.1 签名与发布 (keystore, APK/AAB 生成, Play App Signing) (深入)
│
├── 11. Android 系统与新特性
│   ├── 11.1 权限管理
│   │   └── [ ] 11.1.1 Android 权限管理 (静态, 运行时, 近期版本变更) (深入)
│   ├── 11.2 Android 版本迭代核心变更
│   │   ├── [ ] 11.2.1 分区存储 (Scoped Storage - Android 10+) (深入)
│   │   ├── [ ] 11.2.2 后台执行限制 (后台定位、后台服务 - Android 8+) (深入)
│   │   └── [ ] 11.2.3 通知渠道与通知权限 (Notification Channels - Android 8+, POST_NOTIFICATIONS - Android 13+) (深入)
│   │   └── [ ] 11.2.4 其他重要版本特性 (如深色主题Dark Theme, 手势导航等) (*)
│   ├── 11.3 多窗口与画中画
│   │   └── [ ] 11.3.1 多窗口模式 (Multi-Window) 与画中画 (Picture-in-Picture - PiP) (深入)
│   ├── 11.4 NDK 与 JNI
│   │   └── [ ] 11.4.1 NDK 与 JNI 基础概念与使用场景 (深入)
│
└── 12. 软技能与面试技巧
    ├── 12.1 项目经验
    │   └── [ ] 12.1.1 项目经验梳理与 STAR 法则应用 (深入)
    ├── 12.2 问题解决能力
    │   └── [ ] 12.2.1 解决问题的能力展现 (场景题应对) (深入)
    ├── 12.3 沟通表达
    │   └── [ ] 12.3.1 沟通与表达能力 (清晰、条理、自信) (深入)
    ├── 12.4 学习能力与技术热情
    │   └── [ ] 12.4.1 学习能力与技术热情 (如何展现) (*)
    ├── 12.5 提问环节
    │   └── [ ] 12.5.1 提问环节的准备与技巧 (向面试官提问) (深入)
    ├── [ ] 12.6 模拟面试与反馈 (*)
    ├── [ ] 12.7 针对特定公司的准备 (*)
    └── [ ] 12.8 算法与数据结构准备 (如果需要) (*)

└── 13. 总结与展望 (个人复习收尾)
    ├── [ ] 13.1.1 知识体系查漏补缺
    ├── [ ] 13.2.1 面试心态调整与准备
    └── [ ] 13.3.1 关注 Android 最新动态与趋势
```



