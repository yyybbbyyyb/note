## 基础知识
[App fundamentals](https://developer.android.com/guide/components/fundamentals)

+ apk、aab
+ 安卓应用沙盒机制（principle of least privilege）
+ 共享Linux ID跨应用共享数据
+ 安卓系统权限访问
+ 安卓四大应用组件

## Android的四大组件及其功能

>四大组件：Activity、Service、BroadcastReceiver、ContentProvider
### Activity
[Introduction to activities](https://developer.android.com/guide/components/activities/intro-activities)
[The activity lifecycle](https://developer.android.com/guide/components/activities/activity-lifecycle)
[Activity state changes](https://developer.android.com/guide/components/activities/state-changes)
[Tasks and the back stack](https://developer.android.com/guide/components/activities/tasks-and-back-stack)

+ Configuring the manifest
	+ Declare activities([\<activity\>](https://developer.android.com/guide/topics/manifest/activity-element))
	+ Declare intent filters([\<intent-filter\>](https://developer.android.com/guide/topics/manifest/intent-filter-element))
	+ Declare permissions([\<uses-permission\>](https://developer.android.com/guide/topics/manifest/uses-permission-element))

+ The activity lifecycle
	+ onCreate、onStart、onResume、onPause、onStop、onRestart、onDestory ![[Android官方文档阅读笔记-1.png|194]]

+ Saving and restoring transient UI state
	+ onSaveInstanceState()、onRestoreInstanceState(),结合ViewModel

+ Starting one activity from another
	+ startActivity、startActivityForResult，参数Intent
	+ ActivityA在拉起ActivityB时，两个Activity的生命周期回调顺序：
		+ Activity A’s onPause() method executes.
		+ Activity B’s onCreate(), onStart(), and onResume() methods execute in sequence. Activity B now has user focus.
		+ If Activity A is no longer visible on screen, its onStop() method executes.

+ Activity state changes
	+ Configuration change occurs（屏幕旋转、Handle multi-window cases）
	+ Activity or dialog appears in foreground
	+ User taps or gestures Back
	+ System kills app process

+ Lifecycle of a task and its back stack
	+ Back tap behavior for root launcher activities
	+ Background and foreground tasks
	+ Multiple activity instances
	+ Multi-window environments
	+ Lifecycle recap

+ Manage tasks
	+ Define launch modes
		+ Using the manifest file：![[Android官方文档阅读笔记-2.png|227]]
		+ Using intent flags：FLAG_ACTIVITY_NEW_TASK、FLAG_ACTIVITY_CLEAR_TOP、FLAG_ACTIVITY_SINGLE_TOP
	+ Handle affinities
		+ taskAffinity、allowTaskReparenting
	+ Clear the back stack
		+ clearTaskOnLaunch、alwaysRetainTaskState、finishOnTaskLaunch
	+ Start a task

### Service
[Services overview](https://developer.android.com/develop/background-work/services)
[Foreground services overview](https://developer.android.com/develop/background-work/services/fgs)
[Bound services overview](https://developer.android.com/develop/background-work/services/bound-services)

+ types of services：Foreground、Background、Bound（两种使用方式：启动（onStartCommand）、绑定（onBind））

+ Choosing between a service and a thread（服务默认情况下仍然在主线程中运行，如果服务执行密集或阻塞操作，应该在服务内创建一个新线程）

+ The basics
	+ onStartCommand()、onBind()、onCreate()、onDestroy()
	+ Declaring a service in the manifest
	+ Creating a started service
		+ Extending the Service class（START_NOT_STICKY、START_STICKY、START_REDELIVER_INTENT）
		+ Starting a service
		+ Stopping a service
	+ Creating a bound service
	+ lifecycle of a service![[Android官方文档阅读笔记-3.png|245]]

+ Bound services
	+ Bind to a started service
	+ Create a bound service(IBinder)：Extend the Binder class、Use a Messenger、Use AIDL
	+ Bind to a service（ServiceConnection）

+ Foreground services
	+ Declare foreground services and request permissions
	+ Launch/Stop a foreground service
	+ Foreground service timeouts
	+ Foreground service types：Camera、Connected device、Data sync、Health、Location、Media、Media processing、Media projection、Microphone、Phone call、Remote messaging、Short service、Special use、System exempted
	+ Troubleshoot foreground services

### BroadcastReceiver
[Broadcasts overview](https://developer.android.com/develop/background-work/background-tasks/broadcasts)

+ About system broadcasts

+ Receive broadcasts
	+ Context-registered receivers
		+ Unregister your broadcast receiver
		+ Register receivers in the smallest scope
		+ Create stateful and stateless composable
	+ Manifest-declared receivers
	+ Effects on process state（不要做耗时操作）

+ Send broadcasts
	+ sendOrderedBroadcast(Intent, String)
	+ sendBroadcast(Intent)

+ Restrict broadcasts with permissions
	+ Send broadcasts with permissions
	+ Receive broadcasts with permissions

+ Security considerations

### Intent
[Intents and intent filters](https://developer.android.com/training/basics/intents)

+ Intent types：Explicit intents、Implicit intents
+ Building an intent：Component name、Action、Data、Category、Extras、Flags
+ Receiving an implicit intent：\<intent-filter\>（\<action\>、\<data\>、\<category\>）

### ContentProvider
[Content providers](https://developer.android.com/guide/topics/providers/content-providers)

+ Content providers可以帮助应用程序管理对自身存储或其他应用程序存储的数据的访问，并提供与其他应用程序共享数据的方式

## Fragment

### Fragment概念和使用
[Fragment](https://developer.android.com/guide/fragments)

+ Create a fragment
	+ FragmentContainerView、savedInstanceState == null

+ Fragment manager
	+ getSupportFragmentManager()、getChildFragmentManager()、getParentFragmentManager()
	+ FragmentTransaction、addToBackStack()、findFragmentById()、findFragmentByTag()
	+ setPrimaryNavigationFragment()、saveBackStack()、restoreBackStack()
	+ FragmentFactory(无参数构造函数)、FragmentFactory.instantiate、FragmentScenario

+ Fragment transactions
	+ commit()、.setReorderingAllowed(true)
	+ add()、remove()、replace()、commitNow()（同步但与addToBackStack不兼容）
	+ 操作排序非常重要，尤其是在使用 setCustomAnimations()时
	+ show()、hide() （影响可见性不影响生命周期）
	+ detach()、attach()（Fragment 所处状态 (STOPPED) 保持不变）

+ Animate transitions between fragments
	+ 设置动画、设置转换、使用共享元素转换（FragmentTransaction.addSharedElement()）
	+ 推迟转换（startPostponedEnterTransition()）、通过 RecyclerView 使用共享元素转换

+ Fragment lifecycle
	+ Fragment 的视图有一个单独的 Lifecycle
	+ INITIALIZED、CREATED、STARTED、RESUMED、DESTROYED
	+ ![[Android官方文档阅读笔记-4.png|173]]

+ Saving state with fragments
	+ 视图状态、SavedState、NonConfig
	+ onSaveInstanceState()、onRestoreInstanceState()

+ Communicate with fragments
	+ 使用 ViewModel 共享数据（ViewModelProvider）
	+ 使用 Fragment Result API 获取结果(setFragmentResultListener()、setFragmentResult())

+ Working with the app bar
+ Displaying dialogs with DialogFragment
+ Debug your fragments
+ Test your fragments

## Handler与异步任务

### 异步后台处理
[Asynchronous background processing](https://developer.android.com/develop/background-work/background-tasks/asynchronous)

+ 使用 Java 线程异步工作
	+ 线程池：ExecutorService、Executor、ThreadPoolExecutor
	+ 配置线程池

### 进程与线程
[processes-and-threads](https://developer.android.com/guide/components/processes-and-threads)

+ 进程
	+ android:process
+ 线程：应用启动后拉起的线程为主线程，也叫UI线程
+ 工作线程
	+ Activity.runOnUiThread(Runnable)
	+ View.post(Runnable)
	+ View.postDelayed(Runnable, long)

### Handler

+ 功能：
	+ Handler 是 Android 提供的消息和 Runnable 调度机制，用于与某个线程的消息队列（MessageQueue）交互
	+ 每个 Handler 都与某个线程的 Looper 和 MessageQueue 关联。你创建 Handler 时，它会被绑定到当前线程的 Looper
	+ Handler 用来发送 Message 或 Runnable 给这个队列，然后在对应线程中执行

+ 场景：
	+ 子线程向主线程发送消息更新 UI
	+ 在某个线程内部顺序执行延迟任务（postDelayed）
	+ 实现定时、循环执行任务
	+ 用于跨线程通信

+ Looper：
	+ Looper 负责为线程建立消息循环。默认线程是没有 Looper 的，只有调用 Looper.prepare() 和 Looper.loop() 才能让线程具备消息循环
	+ Handler 内部通过 Looper 获取这个线程的 MessageQueue，然后通过 sendMessage() / post() 加入队列
	+ Looper.loop() 会持续取出队列中的消息，交给 Handler 执行

+ 原理：
	+ Handler 持有一个 Looper 和对应的 MessageQueue
	+ 调用 handler.sendMessage(msg) 或 handler.post(runnable) 会把 Message/Runnable 放入 MessageQueue
	+ Looper 在循环中从 MessageQueue 中取出消息并分发给 Handler
	+ Handler 的 handleMessage() 或 Runnable 的 run() 在 Looper 所在线程执行

### AsyncTask
[AsyncTask](https://developer.android.com/reference/android/os/AsyncTask)

+ AsyncTask 是一个帮助类，用来简化后台任务 + UI 更新操作。
+ 通过 execute() 启动任务，它会在后台线程执行 doInBackground()，完成后在主线程回调 onPostExecute()
+ AsyncTask 在较新版本 Android 中被 deprecated（不推荐用于长期或复杂后台任务）

## View相关

### View、ViewGroup
[Layouts in views](https://developer.android.com/develop/ui/views/layout/declaring-layout)

+ View - widgets - Button、TextView…
+ ViewGroup - layouts - LinearLayout、ConstraintLayout…

### 常用的Layout、View属性

+ 常用Layout、View：
	+ RelativeLayout、LinearLayout、FrameLayout、ConstraintLayout
	+ Button、TextView、EditText、ListView、CheckBox、RadioButton、Gallery、Spinner、AutoCompleteTextView、ImageSwitcher、TextSwitcher

+ 常用属性
	+ layout_width、layout_height、layout_margin、padding、gravity、layout_gravity、layout_weight
	+ (RelativeLayout)
		+ layout_alignParentTop / layout_alignParentLeft（贴到父布局边）
		+ layout_centerInParent（居中）
		+ layout_toLeftOf / layout_toRightOf（相对其他控件）
		+ layout_below（位于指定控件下方）
	+ (ConstraintLayout)
		+ layout_constraintLeft_toLeftOf / Right_toRightOf / Top_toTopOf / Bottom_toBottomOf 与父或其他控件锚定
		+ layout_constraintHorizontal_bias / Vertical_bias 偏移（0～1）
		+ layout_constraintDimensionRatio 宽高比，例如 “1:1”
	+ text、textSize、textColor、textStyle、lines、maxLines、ellipsize、background、foreground、visibility、enabled、clickable、src、scaleType…

### 自定义view
[Create custom view components](https://developer.android.com/develop/ui/views/layout/custom-views/custom-components)
[How Android draws views Android](https://developer.android.com/guide/topics/ui/how-android-draws)
[Create a view class](https://developer.android.com/develop/ui/views/layout/custom-views/create-view)
[Create a custom drawing](https://developer.android.com/develop/ui/views/layout/custom-views/custom-drawing)
[Make a custom view interactive](https://developer.android.com/develop/ui/views/layout/custom-views/making-interactive)
[Optimize a custom view](https://developer.android.com/develop/ui/views/layout/custom-views/optimizing-view)

+ 上面官方文档中各节内容：自定义组件的概述、测量/布局、创建attr/class、onDraw、交互、优化

+ summary
	+ 自定义View的三种方式
		+ 扩展现有View：继承Button、TextView等，添加自定义功能
		+ 组合现有View：将多个View组合成复合控件
		+ 完全自定义View：继承View类，完全自定义绘制和交互
	+ Android绘制机制
		+ 测量（Measure）：确定View的大小
		+ 布局（Layout）：确定View的位置
		+ 绘制（Draw）：在Canvas上绘制内容
	+ 自定义View的关键步骤
		+ 重写onMeasure()处理尺寸测量
		+ 重写onDraw()进行自定义绘制
		+ 重写onTouchEvent()处理触摸交互
		+ 使用invalidate()触发重绘
	+ 性能优化要点 避免在onDraw()中创建对象
		+ 减少不必要的invalidate()调用
		+ 使用硬件加速
		+ 合理使用clipRect()减少绘制区域

+ 阅读源码
```
ViewRootImpl.performTraversals()
├── 前置检查阶段
│     ├── dispatchAttachedToWindow() ← View 首次附加时
│     └── dispatchWindowVisibilityChanged() ← 窗口可见性变化
│         └── onWindowVisibilityChanged()
│
├── Insets 计算阶段
│     ├── dispatchApplyWindowInsets() ← 窗口插入区域变化
│     └── fitSystemWindows() ← 旧版 API
│
├── Measure 阶段
│     └── performMeasure()
│     └── measure()
│         ├── onMeasure() ← ⭐ 核心回调 1
│         └── onMeasureComplete() ← Android 14+ 新增
│
├── Layout 阶段
│     └── performLayout()
│     └── layout()
│         ├── setFrame()
│         │ └── onSizeChanged() ← ⭐ 核心回调 2
│         ├── onLayout() ← ⭐ 核心回调 3 (ViewGroup)
│         └── requestLayout() 的重入检测
│
├── Draw 阶段
│     └── performDraw()
│         └── draw()
│         ├── onDraw() ← ⭐ 核心回调 4
│         ├── dispatchDraw() ← ViewGroup 绘制子 View
│         ├── drawAutofilledHighlight() ← 自动填充高亮
│         └── onDrawForeground() ← 前景绘制
│
└── 后处理阶段
    ├── onWindowFocusChanged() ← 窗口焦点变化
    ├── onVisibilityChanged() ← View 可见性变化
    └── ViewTreeObserver 回调
        ├── onPreDraw()
        ├── onGlobalLayout()
        ├── onWindowAttachChanged()
        └── onDrawListener()
```

### 属性动画/Animation
[Animations and Transitions](https://developer.android.com/develop/ui/views/animations)

+ Introduction to animations
	+ Animate bitmaps
	+ Animate UI visibility and motion
		+ android:animateLayoutChanges=“true”
		+ Physics-based motion（ObjectAnimator、DynamicAnimation）
	+ Animate layout changes
	+ Animate between activities

+ Property Animation Overview
	+ overview：Duration、Time interpolation、Repeat count and behavior、Animator sets、Frame refresh delay
	+ API
		+ Animators：ValueAnimator、ObjectAnimator、 AnimatorSet
		+ Evaluators：Int/Float/Argb/TypeEvaluator
		+ Interpolators：AccelerateDecelerateInterpolator、AccelerateInterpolator、AnticipateInterpolator…
+ 三种Animator的用法
+ Animation listeners：onAnimationStart/End/Repeat/Cancel；onAnimationUpdate（ValueAnimator）、AnimatorListenerAdapter
+ Animate layout changes to ViewGroup objects：android:animateLayoutChanges=“true”
+ Animate view state changes using StateListAnimator
+ Use a TypeEvaluator
+ Use Interpolators
+ Specify keyframes
+ Animate views：translationX/Y、rotation、rotationX/Y、scaleX/Y、pivotX/Y、x/y、alpha、ViewPropertyAnimator
+ Declare animations in XML

### Touch、Key事件分发
[Input events overview](https://developer.android.com/develop/ui/views/touch-and-input/input-events)
[Handle keyboard input](https://developer.android.com/develop/ui/views/touch-and-input/keyboard-input)

+ Event listeners
	+ onClick()、onLongClick()、onFocusChange()、onKey()、onTouch()、onCreateContextMenu()
	+ 带有布尔返回值的回调，表示事件是否被消耗，false则继续传递：onLongClick()、onKey()、onTouch()

+ Event handlers
	+ onKeyDown(int, KeyEvent)、onKeyUp(int, KeyEvent)、Activity.dispatchTouchEvent(MotionEvent)等等…
	+ View类自身“内置”的默认处理逻辑，优先级低于listener

+ Handle keyboard actions
	+ onKeyDown()、onKeyUp()、getModifiers()、getMetaState()、isShiftPressed()、isCtrlPressed()

### RecyclerView
[Create dynamic lists with RecyclerView](https://developer.android.com/develop/ui/views/layout/recyclerview)
[Customize a dynamic list](https://developer.android.com/develop/ui/views/layout/recyclerview-custom)

+ 关键类：RecyclerView、ViewHolder、Adapter、LayoutManager

+ 实现 RecyclerView 的步骤
	+ 确定列表或网格的外观（Layout）
	+ 设计列表中每个元素的外观和行为（ViewHolder）
	+ 定义数据与 ViewHolder 相关联的 Adapter（Adapter）

+ 规划布局
	+ LinearLayoutManager
	+ GridLayoutManager
	+ StaggerGridLayoutManager

+ 实现Adapter和 ViewHolder
	+ adapter
		+ onCreateViewHolder
		+ onBindViewHolder
		+ getItemCount

+ 启用无边框显示
	+ 调用enableEdgeToEdge()
	+ 如果列表项最初与系统栏重叠，可以将 android:fitsSystemWindows 设置为 true 或使用 ViewCompat.setOnApplyWindowInsetsListener
	+ 通过在 RecyclerView 上将 android:clipToPadding 设置为 false，允许列表项在滚动时绘制在系统栏下方。
	+ 以下视频展示了停用（左侧）和启用（右侧）全屏显示的 RecyclerView：

+ 自定义动态列表
	+ 修改布局
	+ 为列表项添加动画：RecyclerView.ItemAnimator
	+ 启用列表项选择： recyclerview-selection库

## Android SDK

### Profiler
[分析应用性能 | Android Studio | Android Developers](https://developer.android.google.cn/studio/profile)

+ Profileable v. debuggable apps
+ Record a system trace（UI Jank）
+ Capture a heap dump（Find memory leaks）
+ Sample the callstack
+ Record Java/Kotlin allocations
+ Record Java/Kotlin methods
+ Record native allocations
+ Inspect your app live
+ Chart glossary：Call chart、Events table、Flame chart、Process Memory (RSS) 、Top down and bottom up charts

### adb常用命令

+ 安装：adb install path/to/app.apk
+ 卸载：adb uninstall package.name
+ 查看设备上的package：adb shell pm list packages（-s：系统内置应用，-3第三方应用，-f带路径，
+ 拉起Activity：adb shell am start -n package.name/.MainActivity
+ 发送Broadcast：adb shell am broadcast -a ACTION_NAME --es key value
+ 模拟按键：adb shell input keyevent KEYCODE_<…>
+ 模拟触摸/点击/滑动：adb shell input tap x y / adb shell input swipe x1 y1 x2 y2
+ 查看设备/连接设备：adb devices / adb connect (或 USB)
+ 重启：adb reboot

## 构建系统

### Configure your build
[Configure your build](https://developer.android.com/build)
[Android build structure](https://developer.android.com/build/android-build-structure)

+ Android build structure
+ Android build glossary:
	+ Build types、Product flavors、Build variants、Manifest entries
	+ Dependencies、Signing、Code and resource shrinking、Multiple APK support
+ Java versions in Android builds
+ Build configuration files
	+ The Gradle Wrapper file
	+ The Gradle settings file
	+ The top-level build file
	+ The module-level build file
		+ Android SDK Settings
			+ compileSdk 编译程序时所用的 Android SDK, 能够访问新的 API
			+ targetSdk 设置应用程序的运行时行为(targetSdk 必须小于或等于 compileSdk)
			+ minSdkVersion 指定应用运行所支持的最低 Android API 级别
	+ Gradle properties files

### Configure build variants
[Configure build variants](https://developer.android.com/build/build-variants#groovy)

### library Module vs application Module

+ Application Module: 最终会生成APK(或AAB)，即实际应用。通常包含manifest、res、源码、Gradle配置等。
+ Library Module: 用来封装公共代码、资源、功能(例如公共UI组件库、工具库、功能模块等)，最终生成AAR/JAR，可以被 application或其他library依赖/引入。

## Performance

### 渲染
[performance of rendering](https://developer.android.com/topic/performance/rendering?hl=zh-cn)

+ 减少过度绘制
	+ GPU 过度绘制调试工具（开发者选项->调试 GPU 过度绘制->显示过度绘制区域）
	+ GPU 渲染模式分析工具（开发者选项->GPU渲染模式分析->在屏幕上显示为条形图）
	+ 解决过度绘制问题：
		+ 移除布局中不必要的背景
		+ 使视图层次结构扁平化
		+ 降低透明度

+ 性能和视图层次结构
	+ 管理复杂布局：建议使用ConstraintLayout or FrameLayout
	+ Double Taxation
	+ 诊断视图层次结构问题：Perfetto、GPU 渲染分析、Lint、布局检查器
	+ 解决视图层次结构问题
		+ 移除多余的嵌套布局
		+ 采用合并或包含(\<merge\>)
		+ 采用开销较低的布局

+ 改善布局性能
	+ 优化布局层次结构
	+ 通过 \<include\> 重复使用布局
	+ 按需加载视图(ViewStub)