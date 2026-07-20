

## 重要的基础容器类

### Activity 基类体系

| 类                       | 作用                                                     |
| ----------------------- | ------------------------------------------------------ |
| TvBaseActivity          | 继承FragmentActivity，最底层的通用 Activity 基类                  |
| TvBaseBackActivity      | 继承TvBaseActivity，提供了基本的回退出栈功能                          |
| BaseActivity            | 继承TvBaseBackActivity，有NetWorkErrorTips、mBackgroudView等 |
| TVActivity              | 继承BaseActivity，项目TV页面基础 Activity，很多普通页面继承它             |
| BaseMvvmActivity\<T\>   | 继承TVActivity，MVVM架构的基类Activity                         |
| BasePlayerActivity\<T\> | 继承TVActivity，播放Activity基类                              |
| DetailBaseActivity      | 继承BasePlayerActivity，详情页Activity基类                     |
| OnePageActivity         | 继承BasePlayerActivity，OnePage框架Activity基类               |

### Fragment 基类体系
| 类                         | 作用                                       |
| ------------------------- | ---------------------------------------- |
| TvFragment                | 继承TvBaseFragment，项目内大多数 Fragment 的基础类    |
| BaseOnePageFragment       | 继承TvFragment，通用首页/频道页/详情页外层 Fragment 基类  |
| BaseOneDetailPageFragment | 继承BaseOnePageFragment，新详情页外层基类           |
| DetailBaseFragment        | 继承TvFragment，老详情页 Fragment 基类            |


## 首页相关 Activity / Fragment

```
LeftNavHomeUIFragment 
  ├── LeftNavNormalFragment
  │     ├── LeftNavHomeDefaultFragment
  │     ├── LeftNavShortVideoFragment
  │     ├── LeftNavHistoryFragment
  │     ├── HomeImmerseFragment
  │     ├── LeftNavSearchFragment
  │     └── LeftNavParentSettingFragment
  ├── ChildFragment
  └── ElderFragment
```

| 类                            | 作用                                  |
| ---------------------------- | ----------------------------------- |
| LeftNavHomeUIFragment        | 首页主 UI Fragment，负责模式切换、状态栏、频道菜单、焦点等 |
| LeftNavNormalFragment        | 普通模式首页主体，管理频道页 ViewPager/频道数据       |
| LeftNavHomeDefaultFragment   | 普通频道页默认内容                           |
| LeftNavShortVideoFragment    | 首页短视频频道                             |
| LeftNavHistoryFragment       | 首页历史/追剧类频道                          |
| HomeImmerseFragment          | 首页沉浸式频道                             |
| LeftNavSearchFragment        | 首页左导航搜索入口                           |
| LeftNavParentSettingFragment | 家长设置入口                              |
| ChildFragment                | 少儿模式首页                              |
| ElderFragment                | 长辈模式首页                              |


## 详情页相关 Activity / Fragment

```
OneDetailCoverPageFragment
  ↓
DoubleRowDetailCoverPageFragment
  ├── containerPlayer       左侧播放器区域
  ├── containerInfo         左侧信息区域
  ├── rvSideDetailList      右侧上半区列表
  ├── rvSideDetailList2     右侧下半区列表
  └── DialogManagerFragment 半屏浮窗管理器
        ↓
        HalfScreenPageContentFragment / HalfScreenCoverContentFragment / ...
```

| 类                                | 作用                    |
| -------------------------------- | --------------------- |
| OneDetailCoverPageFragment       | 新详情页外层管理 Fragment     |
| DoubleRowDetailCoverPageFragment | 新详情页真正的双列 UI Fragment |
| OneDetailCoverFullPageFragment   | 全屏样式详情页 Fragment      |
| BaseOneDetailPageFragment        | 新详情页外层基类              |

## 半屏浮窗相关 Fragment

核心目录：`proj.android/src/com/tencent/qqlivetv/detail/halfcover/`

```
DialogManagerFragment                      # 详情页半屏浮窗总管理器
  ↓ 接收 ShowDialogEvent
具体半屏内容 Fragment
  ├── HalfScreenPageContentFragment        # 最通用的半屏内容页，很多半屏类型最终走它
  ├── HalfScreenCoverContentFragment       # cover 内容型半屏
  ├── HalfScreenCoverProfileFragment       # 资料/简介类半屏
  ├── HalfScreenChildContentAddFragment    # 少儿内容添加类半屏   
  ├── HalfScreenLoginFragment              # 半屏登录
  └── 其他业务半屏 Fragment
```




## 播放器相关 Activity / Fragment

TVPlayerActivity
ShortVideoPlayerActivity
DetailPlayerFragment
TvPlayerFragment


## 频道二级页 

NewChannelPageFragment