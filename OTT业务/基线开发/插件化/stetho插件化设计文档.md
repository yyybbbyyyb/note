> 仿照已有的Plugin_Rmonitor、Plugin_OpenTelementry等插件实现stetho插件化接入基线，本文档侧重于在基线中接入插件需要完成哪些内容

### 更改文件清单

```Groovy
Plugin_Base/src/com/tencent/qqlivetv/plugincenter/perform/IStethoPerformer.java
Plugin_Base/src/com/tencent/qqlivetv/plugincenter/utils/PluginUtils.java

Plugin_Stetho/.gitignore
Plugin_Stetho/build.gradle
Plugin_Stetho/proguard-rules.pro
Plugin_Stetho/proguard.cfg
Plugin_Stetho/src/main/AndroidManifest.xml
Plugin_Stetho/src/main/java/com/ktcp/stetho/StethoPerformer.java
Plugin_Stetho/src/main/java/com/ktcp/stetho/plugins/HelloWorldDumperPlugin.java
Plugin_Stetho/src/main/res/values/strings.xml

Resources/plugin/plugin_list.json
app/build.gradle
gradle.properties
proj.android/patch/stetho.xml
proj.android/plugin_debug.gradle
proj.android/src/com/tencent/qqlivetv/plugin/Plugin.java
proj.android/src/com/tencent/qqlivetv/plugin/perform/stetho/DefaultStethoPerformer.java
settings.gradle

// 以下为Stetho个性化的更改，并不通用
proj.android/res/layout/activity_easter_eggs.xml
proj.android/src/com/ktcp/video/activity/EasterEggsActivity.java
proj.android/src/com/tencent/qqlivetv/stetho/TvStetho.java
proj.android/tools/stetho/dumpapp
proj.android/tools/stetho/dumpappQuick.sh
proj.android/tools/stetho/hprof_dump.sh
proj.android/tools/stetho/stetho_open.py
build-ext.gradle
```

### Plugin_Base

+ 注册一个接口 IStethoPerformer ，继承 IPerformer ，其中的方法则为基线中需要调用插件的方法（包括初始化、数据交换等等）
+ PluginUtils 中添加一个常量字段 MODULE_STETHO ，并且在下方完善 isAvePlugin 方法

### Plugin_Stetho

在根目录中创建一个新的模块 Plugin_\*（可以选择copy一个已有的插件），但需要更改一些东西：
+ StethoPerformer 类实现 IStethoPerformer 接口
+ AndroidManifest.xml 中的包路径、label、插件的meta-data信息等需要更改
+ proguard.cfg 中最下方类完整名称需要更改
+ build.gradle 中需要更改内容较多，包括applicationId、stetho_plugin_version、stetho_plugin_version_code（这两个字段会注册在后续提到的gradle.properties 中）、implementation 依赖环境等等

### 其他

+ plugin_list.json 中添加组件信息，此文件中可配置后台下载路径
+ app/build.gradle 中添加一项 mergetTask.dependsOn "buildStethoPlugin${buildType}"，用于本地调试插件（启动本地调试插件时把方法isDebugPluginOpen 返回值改成true）
+ gradle.properties中写stetho_plugin_version、stetho_plugin_version_code
+ stetho.xml中也是列举插件信息，其中type根据情况选择（本插件选择：1 dex文件）
+ plugin_debug.gradle 这个文件中添加刚刚 build.gradle 中调用的Task，可以利用 addCommonPluginBuildTask 方法
+ Plugin.java 中实现同步和异步获得插件的方法（利用PluginLauncher中提供的方法）
+ 提供一个 DefaultStethoPerformer.java，实现 IStethoPerformer，所有的方法空函数体即可，提供一个no-op的默认模式
+ settings.gradle 中引入自己写的 Plugin_\* 模块

### 个性化

+ TvStetho.java 是基线中初始化和管理Stetho的辅助工具类
+ Stetho插件的初始化添加到了彩蛋页中
+ proj.android/tools/stetho/ 中放置的是stetho所用脚本
+ build-ext.gradle 中可以规范引入包名