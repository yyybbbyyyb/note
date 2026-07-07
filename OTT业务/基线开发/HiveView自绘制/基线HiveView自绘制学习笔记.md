


## 自绘制

一些概念/模块：
+ component（BaseComponent）
+ canvas（Text、Drawable、Color、Animatable）
+ view（HiveView）
+ viewModel（HiveViewModel）
+ viewInfo

#### HiveView

+ HiveView（自绘制机制的视图，extend View）
+ create、setComponent

#### Component

+ BaseComponent（component的基类）
	+ onCreate、onDestroy
	+ set/isUpdateUiAsyncEnd（但貌似不需要管）
	+ onAttach、onDetach、onDraw
	+ onTriggerDraw（在每次draw绘制界面元素之前调用，实现此方法可以动态修改某些元素可见性）
	+ onMeasure（测量Component中的绘制元素大小和摆放位置）
	+ setDefaultCanvasRect、setDesignRectAsync
	+ onStateChanged
	+ create、addDefault、addDefaultCanvas、destroy、attach、detach、measure
	+ isAsyncMode、isBound
	+ stateChanged、visibilityChanged、focusChanged
	+ addElementBefore、setFocusedElement、setEasyDrawElement、removeElement、hideElement（但是不太理解是干嘛用的）
	+ setView
	+ getWidth、getHeight

#### Canvas

+ BaseCanvas（canvas的基类）
	+ draw、drawEasy
	+ isVisible、isActualVisible、setVisible、onVisibleChanged
	+ getContext、getResources
	+ setRect、getRect、setDesignRect、offsetDesignRectLeftAndRight、onRectChanged
	+ setState、isStateful、onStateChanged\
	+ onDraw
+ TextCanvas（extend Base）
	+ startTextShine、endTextShine（闪光动效）
	+ setGravity、setText、setBackgroundDrawable、setDesignTextSize、setAlpha、setTextColor、setShadowLayer
	+ setMaxDesignWidth、setMaxLines、setLineSpacing、setEllipsize、setMarqueeSpeed、setTextBold、...
 + RoundableCanvas（extend Base）
	 + setRound、setRoundType、setRadius、setRoundDynamic、
+  DrawableCanvas（extend Roundable）
	+ setDrawable、setScaleType、setAdjustViewBounds、setAlwaysStateful、setTag
+ ColorCanvas（extend Roundable）
	+ setColor
+ AnimatableDrawableCanvas（extend Drawable）
	+ setAnimation、setAnimatable、setAnimator、setDrawable、setScheduler、scheduleDrawable
+ NetworkImageCanvas（extend Drawable）
	+ setDefaultDrawable、setErrorDrawable、setCutImageIntoCircle、setImageUrl
+ 很多其他Canvas：TextGroupCanvas、ProgressBarCanvas

#### HiveViewModel

+ HiveViewModel
	+ initView、initRootView
	+ getHiveView、getComponent
	+ onBindAsync、onPreData
	+ onUpdateUI、onUpdateUiAsync、onUpdateUiAsyncEnd、onRequestBgSync



## 其他

#### setDesignRectAsync

+ 坐标系统: 使用的是绝对坐标，相对于父容器的左上角
+ 性能优化: 方法内部会检查坐标是否变化，只有变化时才触发重绘
+ 刷新机制: 坐标改变会触发 onRectChanged() 回调
+ 配合属性: 可以结合 setGravity() 设置对齐方式
+ 单位换算：AutoDesignUtils.designpx2px

#### 跑马灯

+ setIsStateful(boolean) - 感知焦点状态变化
+ setEllipsize(TextUtils.TruncateAt.MARQUEE) - 设置跑马灯模式
+ setMaxDesignWidth(int) - 设置最大宽度（必须设置，否则文本不会超长）
+ setMarqueeRepeatLimit(int) - 设置循环次数（-1 表示无限循环）
+ setMarqueeSpeed(int) - 设置速度（dp/秒）
+ start() / stop() - 手动控制跑马灯

```java
mMainTextCanvas.setIsStateful(true); // 让 TextCanvas 感知焦点状态变化
mMainTextCanvas.setMaxDesignWidth(380); // 限制最大宽度（略小于 HiveView 的 400dp）
mMainTextCanvas.setMarqueeRepeatLimit(TextCanvas.MARQUEE_REPEAT_FOREVER); // 无限循环
mMainTextCanvas.setMarqueeSpeed(30); // 30dp/秒
mMainTextCanvas.setEllipsize(TruncateAt.END); // 默认打点显示

public void onFocusChanged(boolean hasFocus) {
	if (hasFocus) {
		mMainTextCanvas.setEllipsize(TruncateAt.MARQUEE);
		mMainTextCanvas.start();
	} else {
		mMainTextCanvas.stop();
		mMainTextCanvas.setEllipsize(TruncateAt.END);
	}
}
```

#### Poster / Factory

+ PosterType：定义海报类型
+ PosterViewTypeConvert：根据type生成对应的ViewModel（往上有：ViewTypeConvert、GridModeConvert、CommonViewConvert、ViewConvertManager、InnerViewTypeConvert）
+ InnerViewTypeConvert：convertToViewModel方法收束了众多Manager的convert逻辑
+ TVViewModelFactory：用于创建一个ViewModel

#### UI反查View/Component

希望找到首页免费频道短剧部分垂直列表中的View

+ Layout Inspector抓布局找到了ClippingVerticalScrollGridView，进而在其代码getAdapter处断点调试，定位到ShortVideoFeedsMultiTabsPlayAdapter，其中定位到viewModel为PlayEntryW606H180ShortDramaViewModel，进而找到Component为PlayEntryW606H180ShortDramaComponent

