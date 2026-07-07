## 架构设计概览

### 核心架构组件
![[基线MVVM架构与数据绑定学习笔记-1.png|350]]

## 1. 数据层 (Model)

### 1.1 ObservableJceStruct

```java
public abstract class ObservableJceStruct extends JceStruct implements Observable {
	private transient PropertyChangeRegistry mCallbacks;
	
	// 属性变化通知
	public void notifyPropertyChanged(int fieldId) {
		synchronized (this) {
			if (mCallbacks == null) {
				return;
			}
		}
		mCallbacks.notifyCallbacks(this, fieldId, null);
	}
}
```

特点：
+ 实现Observable接口，支持数据变化监听
+ 提供属性变化通知机制
> 不过据观察继承该类的类不多，有ButtonInfo、VideoInfo等共五个

### 1.2 具体数据模型示例
```java
// 首页菜单数据模型
public class HomeMenuInfo extends BaseObservable {
	private ChannelInfo channelInfo;
	private boolean focused;
	private String title;
	private boolean imageMenu;
	
	@Bindable
	public ChannelInfo getChannelInfo() {
		return channelInfo;
	}
	
	@Bindable
	public int getFontSize() {
		return AutoDesignUtils.designsp2px(fontSize);
	}
	
	public void setChannelInfo(ChannelInfo channelInfo) {
		this.channelInfo = channelInfo;
		notifyPropertyChanged(BR.channelInfo);
	}
}
```

## 2. 视图模型层 (ViewModel)

### 2.1 ViewModel层次结构

```
BaseViewModel
├── TVBaseViewModel
│ ├── TVViewModel<T>
│ │ ├── TVCssViewModel<T>
│ │ │ ├──ItemInfoViewModel<T>
│ │ │ │ ├── HiveViewModel<Data, Component>
│ │ │ │ │ ├── BaseMenuViewModel
│ │ │ │ │ ├── AbsHeadPosterPlayerViewModel
│ │ │ │ │ ├── ...其他具体ViewModel
```

### 2.2 核心ViewModel实现
#### TVBaseViewModel - 基础ViewModel

```java
public abstract class TVViewModel<T> extends TVBaseViewModel<T> {
	private boolean mPreData;
	private T mData;
	
	// 异步刷新数据
	public final boolean updateUI(T data) {
		preData();
		onRecordUpdateStart();
		boolean ret = onUpdateUI(data);
		if (isUseAsyncModel()) {
			mData = data;
			mNeedUpdateUi.set(true);
			if (sAllowedPreload && mFistPageId > 0) {
				updateUiAsyncInner(false);
			} else {
				if (isAttached()) {
					updateUiAsyncInner(true);
				}
			}
		}
		if (mViewDuration != null) {
			mViewDuration.updateEndTime = getNanoTimeMills();
		}
		return ret;
	}
}
```

#### HiveViewModel - 组件化ViewModel

```java
public abstract class HiveViewModel<Data, Component extends BaseComponent> extends ItemInfoViewModel<Data> implements HiveViewHolder {
	private final Component mComponent;
	private Data mData;
	
	// 支持异步数据加载
	@Override
	protected void onUpdateUiAsync(T data) {
		if (shouldSetEasyModelOnUpdateUiAsync()) {
			getHiveView().setEasyMode(true);
		}
		super.onUpdateUiAsync(data);
		// 更新组件
		if (getComponent() != null) {
			getComponent().createCanvas();
			getComponent().setDesignRectAsync();
		}
	}
	
	@Override
	protected boolean onUpdateUI(Data data) {
		mData = data;
		return super.onUpdateUI(data);
	}
	
	@NonNull
	public abstract Component onComponentCreate();
}
```

### 2.3 具体ViewModel示例

```java
public class HeadPosterPlayerViewModel extends AbsHeadPosterPlayerViewModel<HeadPosterPlayerComponent> {

@Override
protected void updateData(@NonNull HeadPosterPlayerViewInfo data) {
	super.updateData(data);
	getComponent().setSecondTitle(data.secondTitle);
	updateTypedSpan(data);
	mAppendedVoiceKey = data.mainTitle + data.secondTitle;
}
}
```

## 3. 视图层 (View)

### 3.1 MVVM Activity基类

```java
public abstract class BaseMvvmActivity<VM extends BaseAndroidViewModel> extends TVActivity {

	protected VM mViewModel;
	
	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		
		// 初始化流程
		initParam(); // 参数初始化
		obtainViewModel(); // 获取ViewModel
		initView(); // 初始化视图
		initData(); // 初始化数据
	}
	
	private void obtainViewModel() {
		mViewModel = initViewModel();
		// 生命周期绑定
		mViewModel.setTVLifecycleOwner(this);
		getTVLifecycle().addObserver(mViewModel);
	}
}
```

### 3.2 组件化视图 - HiveView

```java
public class HomeMenuItemComponent extends TVBaseComponent {
	@BindNode DrawableCanvas mFocusShadow;
	@BindNode TextCanvas mMenuText;
	@BindNode DrawableCanvas mNormalImage;
	@BindNode DrawableCanvas mSelectedImage;
	@BindNode DrawableCanvas mFocusedImage;
	
	@Override
	public void onCreate() {
		super.onCreate();
		addElement(mFocusShadow, mMenuText, mNormalImage,
		mSelectedImage, mFocusedImage);
		
		// 设置样式和状态
		mFocusShadow.setDrawable(drawable(R.drawable.home_menu_focused_bg_normal));
		mMenuText.setDesignTextSize(FontSize.MAIN_SIZE_36);
		mMenuText.setTextColor(mNormalColor);
	}
}
```

## 4. 数据绑定机制

### 4.1 DataBinding核心类

```java
public abstract class BaseHiveDataBinding<Component extends TVBaseComponent, Data> {
	private Component mComponent;

	public void setRoot(@NonNull Component component) {
		mComponent = component;
	}
}
```

### 4.2 具体数据绑定实现

```java
public class HiveDataBinding<Component extends TVBaseComponent, Data> extends BaseHiveDataBinding<Component, Data> {

	private final FocusShadowDataBinding<Component> mFocusShadow = FocusShadowDataBinding.create();
	private final PlayRoundIconDataBinding<Component> mPlayIcon = PlayRoundIconDataBinding.create();

	@Override
	public void setCss(@NonNull TVBaseViewModel<?> viewModel, @Nullable ModelCss modelCss) {
		mFocusShadow.setCss(viewModel, modelCss);
		mPlayIcon.setCss(viewModel, modelCss);
	}
}
```

### 4.3 时序图
![[基线MVVM架构与数据绑定学习笔记-2.png|320]]

## 5. 完整数据流示例 — HomeMenu

### 5.1 首页菜单数据流

以首页菜单为例，完整的数据流转过程：
![[基线MVVM架构与数据绑定学习笔记-3.png|239]]

### 5.2 代码实现流程

#### 步骤1：数据模型定义
```java
public class HomeMenuInfo extends BaseObservable {
	private ChannelInfo channelInfo;
	private boolean focused;
	private String title;
	
	@Bindable
	public ChannelInfo getChannelInfo() {
		return channelInfo;
	}
	
	public void setTitle(String title) {
		if (!TextUtils.equals(title, this.title)) {
			this.title = title;
			notifyPropertyChanged(BR.title);
		}
	}
}
```

#### 步骤2：ViewModel处理
```java
public class BaseMenuViewModel extends HiveViewModel<HomeMenuInfo, HomeMenuItemComponent> {

	// 这里是因为有三个子类实现了这个基类，具体的刷新方法在子类中
	@Override
	protected boolean onUpdateUI(HomeMenuInfo data) {
		super.onUpdateUI(data);
		return true;
	}
}
```

#### 步骤3：视图组件更新
```java
public class HomeMenuItemComponent extends TVBaseComponent {
	@BindNode TextCanvas mMenuText;
	@BindNode DrawableCanvas mFocusShadow;
	
	public void setSelectedColor(int selectedColor) {
		this.mSelectedColor = selectedColor;
		if (isSelected() && !isFocused()) {
			mMenuText.setTextColor(mSelectedColor);
		}
	}
	
	public void setFocusShadow(Drawable drawable) {
		mFocusShadow.setDrawable(drawable);
	}
}
```

## 6. 完整数据流示例 —— HeadPosterPlayer

### 1. 整体架构

头图海报播放组件采用分层架构设计：
![[基线MVVM架构与数据绑定学习笔记-4.png|180]]
### 2. 核心类关系

#### 2.1 继承关系
+ HeadPosterPlayerViewModel 继承 AbsHeadPosterPlayerViewModel 继承 HiveViewModel ...

#### 2.2 数据模型
+ HeadPosterPlayerViewInfo: 头图海报的数据结构
+ PosterPlayerViewInfo: 播放器使用的数据结构
+ PosterPlayModel: 播放器的数据模型

### 3. 数据流程详解

#### 3.1 数据解析流程
![[基线MVVM架构与数据绑定学习笔记-5.png|566]]
#### 3.2 异步更新流程
![[基线MVVM架构与数据绑定学习笔记-6.png|562]]
#### 3.3 播放器数据转换流程
![[基线MVVM架构与数据绑定学习笔记-7.png|568]]

### 4. 关键数据字段
#### 4.1 HeadPosterPlayerViewInfo主要字段
+ posterUrl: 海报图片URL
+ mainTitle: 主标题
+ secondTitle: 副标题
+ playableID: 播放数据
+ unfocusTypeTags: 未获焦标签
+ focusTypeTags: 获焦标签
+ ottTags: 角标信息
+ fallbackPoster: 兜底海报

#### 4.2 播放状态相关
+ mFocused: 是否获焦
+ mPlayable: 是否可播放
+ isSupportTiny: 是否支持小窗播放

### 5. 生命周期管理
#### 5.1 ViewModel生命周期
![[基线MVVM架构与数据绑定学习笔记-8.png|565]]
#### 5.2 播放器生命周期
![[基线MVVM架构与数据绑定学习笔记-9.png|572]]