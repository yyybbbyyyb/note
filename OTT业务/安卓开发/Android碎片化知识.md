

#### activity vs dialog vs fragment

+ 启动流程：
	+ activity：App进程 → ActivityManagerService(AMS) → 创建ActivityRecord → 调度Activity启动 → 创建Window → 创建DecorView → 执行生命周期回调 → 渲染UI
	+ dialog：Activity → 创建Dialog对象 → 创建Window → 添加到WindowManager → 渲染UI
	+ fragment：Activity → FragmentManager → 创建Fragment → 执行生命周期 → 添加View到Activity的ViewGroup → 渲染UI

+ activity
	+ 完整的应用组件，是Android四大组件之一
	+ 拥有独立的Window和完整的生命周期
	+ 启动需要通过startActivity()，涉及系统级别的进程间通信
+ dialog
	+ 依附于Activity的窗口，不是独立组件
	+ 拥有独立的Window，但生命周期依赖宿主Activity
	+ 通过dialog.show()直接显示，不涉及AMS
+ fragment
	+ Activity的UI片段，不是独立组件
	+ 没有独立Window，共享宿主Activity的Window
	+ 通过FragmentTransaction添加，完全在应用进程内

+ 性能上
	+ activity：跨进程通信（AMS）、WMS、资源加载、生命周期复杂
	+ dialog：无AMS参与、复用Context、Window创建简化、生命周期简单
	+ fragment：无系统服务参与、无Window创建、 无Binder调用、 View级别操作

#### DialogFragment

+ DialogFragment = Fragment + Dialog
+ DialogFragment内部会创建一个Dialog实例
+ DialogFragment有独立的Window（通过内部的Dialog）

```
Activity
	└─ Window (PhoneWindow)
		├─ DecorView
		│   └─ ContentView
		│       └─ Fragment (普通Fragment，无独立Window)
		│
		└─ Dialog Window (DialogFragment创建的)
			└─ Dialog DecorView
				└─ DialogFragment的View
```

+ 普通Fragment：View直接添加到Activity的ViewGroup中
+ DialogFragment：先创建Dialog Window，再把View添加到Dialog的DecorView中

#### ART AOT vs JIT

Android ART (Android Runtime) 的编译优化经历了一个从“激进”到“智能”的演变过程。简单来说，它是从 Android 4.4 开始引入，Android 5.0 正式取代 Dalvik，并在 Android 7.0 达到了一个成熟的混合编译形态。

1. 引入与替代 (Android 4.4 - 5.0)
	+ Android 4.4 (KitKat): ART 首次作为开发者选项引入，目的是让开发者提前适配。当时默认还是 Dalvik。
	+ Android 5.0 (Lollipop): ART 正式取代 Dalvik 成为默认运行时。

2. 纯 AOT 时代 (Android 5.0 - 6.0)
	在 Android 5.0 和 6.0 时期，ART 采用的是 全量 AOT (Ahead-of-Time，预先编译) 策略。
	+ 机制: 应用在安装时，系统会调用 dex2oat 工具，把 APK 里的所有字节码（Bytecode）一次性编译成本地机器码（Native Code）。
	+ 优点: 运行时直接执行机器码，性能非常好，不再需要像 Dalvik 那样边解释边执行。
	+ 缺点:
		+ 安装极慢: 大型应用安装可能需要几分钟。
		+ 占用空间大: 编译后的机器码文件（.oat）比原始 APK 大很多。
		+ 系统更新痛苦: 每次系统 OTA 升级后，所有应用都要重新编译（那个漫长的“正在优化应用 x/xxx”界面）。
	
3. 混合编译变革 (Android 7.0 Nougat) —— 里程碑
	这是 ART 优化历史上最重要的一次转折。Android 7.0 引入了 JIT (Just-In-Time) + AOT + PGO (Profile-Guided Optimization) 的混合编译策略。
	+ 机制:
		+ 安装时不编译: 应用安装速度极快，几乎秒装。
		+ 运行时 JIT: 用户使用应用时，ART 使用解释器 + JIT 运行。同时，它会记录哪些代码是“热点代码”（经常被执行的代码），并生成一个配置文件（Profile）。
		+ 空闲时 AOT: 当设备充电且处于空闲状态时，后台服务会读取 Profile 文件，只把那些“热点代码”编译成机器码。
	+ 效果：
		+ 安装快
		+ 不占太多空间（只编译热点代码）。
		+ “越用越快”: 随着你使用时间的增加，Profile 越来越精准，热点代码都被编译成了机器码，性能逐渐达到纯 AOT 的水平。

4. 持续进化 (Android 8.0 - 14+)
	+ Android 9.0 (Pie) - Cloud Profiles:
		+ 解决了 Android 7.0 “刚安装好第一次运行慢”的问题。
		+ Google Play 会收集所有用户的 Profile 数据，聚合出一份“云端 Profile”。当你下载应用时，这份 Profile 会一起下载下来。这样在安装时，ART 就能直接把热点代码编译好，让你第一次打开应用就很快。
	+ Baseline Profiles (基准配置文件):
		+ 现代 Android 开发（Jetpack）中，开发者可以手动随 APK 打包一个 baseline-prof.txt。
		+ 这允许开发者显式告诉系统：“这些是启动时的关键代码，请务必优先编译”。这极大地优化了冷启动性能。

#### RecyclerView中的ItemDecoration语法

ItemDecoration 是 RecyclerView 提供的一个强大的装饰器机制，用于在不修改 Adapter 和 ViewHolder 的情况下，为列表项添加视觉装饰和间距控制。

+ 绘制装饰：在 item 的上方、下方或覆盖层绘制内容（如分割线、背景、角标等）
+ 控制间距：为 item 添加 padding/margin，控制 item 之间的间距
+ 不侵入数据层：完全独立于 Adapter，不影响数据结构

```java
public abstract class ItemDecoration {
	// 1. 设置偏移量（最常用）
	public void getItemOffsets(Rect outRect, View view, RecyclerView parent, State state) {
		// outRect: 设置 item 的四周偏移
		// view: 当前 item 的 View
		// parent: RecyclerView 本身
		// state: RecyclerView 的状态
	}
	
	// 2. 在 item 下方绘制（背景层）
	public void onDraw(Canvas c, RecyclerView parent, State state) {
		// 在 item 绘制之前调用，绘制内容会被 item 覆盖
	}
	
	// 3. 在 item 上方绘制（前景层）
	public void onDrawOver(Canvas c, RecyclerView parent, State state) {
		// 在 item 绘制之后调用，绘制内容会覆盖 item
	}
}
```

#### EventBus总线

EventBus 是三方库 greenrobot EventBus(一个非常流行的 Android/Java 事件发布-订阅库)，是一个发布-订阅模式的事件总线库，用于简化 Android 组件间通信

三要素：
+ Event：普通 Java 对象（POJO），只当数据信封，不需继承任何类
+ Subscriber：用 @Subscribe 注解的方法，靠参数类型匹配事件（方法名随便起）
+ Poster：eventBus.post(new XxxEvent()) 发布事件

常用注解：
+ @Subscribe(threadMode = ThreadMode.MAIN) → 回调保证在主线程（可安全操作 UI），还可以设置优先级：priority = 1
+ 其他模式：POSTING(发布线程) / BACKGROUND / ASYNC
+ @SuppressWarnings("unused") → 因为方法是反射调用的，加它免"未被使用"警告

注册/反注册（必须配对！）：
```java
onResume → EventBus.register(this)     // 注册：反射扫描 @Subscribe 方法建映射表
onDestroy → EventBus.unregister(this)  // 反注册：否则强引用导致内存泄漏 + 崩溃
```








#### Handler
```java
mHandler.removeMessages(MSG_REQUEST);
mHandler.obtainMessage(MSG_REQUEST).sendToTarget();
```
先把队列里还没执行的同类任务删掉，再重新投递一个最新任务
这样做的好处是防抖 / 合并重复请求。比如短时间内多次调用，最终只保留最后一次请求任务，避免状态栏重复请求


#### SimpleArrayMap

|      | HashMap             | SimpleArrayMap                               |
| ---- | ------------------- | -------------------------------------------- |
| 底层结构 | 数组 + 链表/红黑树（真正哈希表）  | 两个平行数组（一个存 key，一个存 value）                    |
| 查找方式 | 计算 hash → 定位桶       | 计算 hash → <br>在 key 数组里线性遍历找相同 hash 和 equals |
| 内存开销 | 大（有 Node 对象、桶数组有冗余） | 小（没有额外包装对象）                                  |
| 适合场景 | 元素多、频繁增删查           | 元素少（几十个以内）                                   |
SimpleArrayMap 就是 Android 打造的"省内存版的 HashMap"，专门为元素少的场景优化，避免 HashMap 在 TV 这种内存敏感设备上产生过多对象。













