## 基本的目录

### 应用安装包内的数据目录

每个 Android 应用在系统上都会有一个独立的目录，用于存放该应用的数据，这个目录一般位于 /data/data/\<package_name\>/，这个目录通常包含了以下几个子目录：

+ files/：存放应用的私有文件，如配置文件、下载的文件等。
+ databases/：存放 SQLite 数据库文件。
+ shared_prefs/：存放 SharedPreferences 文件。
+ cache/：存放应用的缓存数据。

这些文件是私有的，其他应用无法直接访问。

### SD 卡（/storage/emulated/0 或 /mnt/sdcard）

SD 卡（或者更现代的设备中的内部存储）通常挂载在 /storage/emulated/0/ 或 /mnt/sdcard/ 目录下。这里可以存放用户的公共数据，如图片、视频、音频、下载的文件等。

+ /storage/emulated/0/：是 Android 内部存储的挂载点，也就是常见的“内置存储”，在不同的设备上，可能会以 sdcard0 或类似名称显示。
+ /sdcard/：指向内部存储的符号链接，在某些设备上也可能会指向外部存储设备。

### Root 目录（/）

Root 目录是 Android 系统的顶层目录。在 Android 上，root 目录并不像在传统 Linux 系统中那样可以直接访问，而是受限的。普通用户无法访问根文件系统，只有获得 root 权限的设备（即“Rooted”设备）才能访问。

一般来说，root 目录下包含如下内容：

+ /system/：存放操作系统的系统文件，包括应用程序和框架。
+ /data/：存放应用数据（如前述的应用私有数据目录）和用户的设置。
+ /cache/：存放缓存数据。
+ /proc/：虚拟文件系统，包含系统和进程信息。

## 开发机和 Android 设备的文件传输

+ adb push/pull
+ stetho文件传输插件
	+ stetho的 FilesDumperPlugin 通过获取应用内部存储目录（如通过 context.getFilesDir() 获取）的文件进行操作。
	+ stetho依靠命令行重定向，比特级别的传输文件

## 一些特殊的路径

+ 外部存储：
	+ 路径：/sdcard/ 或 /storage/emulated/0/
	+ 这是共享的存储区域，可以存放文件并进行访问。
	+ File externalStorage = Environment.getExternalStorageDirectory();

+ 应用的外部文件目录：
	+ 路径：/storage/emulated/0/Android/data/<package_name>/files/
	+ 每个应用有自己独立的目录，用于存放公共文件。
	+ **File externalFilesDir = mContext.getExternalFilesDir(null);**

+ 应用的缓存目录：
	+ 路径：/storage/emulated/0/Android/data/<package_name>/cache/
	+ 用于存放临时缓存文件，通常是应用本身可访问的。
	+ File cacheDir = mContext.getCacheDir();
	+ **File externalCacheDir = mContext.getExternalCacheDir();**

+ 临时存储目录：
	+ 路径：/data/local/tmp/
	+ 设备的临时目录，普通用户可以访问，适合存放临时文件。
	+ File tmpDir = new File("/data/local/tmp/");

+ 私有沙盒目录：
	+ 路径：/data/data/<package_name>/files/
	+ 每个应用有独立的沙盒目录，无法直接访问，但应用可以将文件导出到外部存储。
	+ **File filesDir = mContext.getFilesDir();（一般接getParentFile获得应用内部存储）**