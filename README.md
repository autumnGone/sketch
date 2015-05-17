# ![Logo](https://github.com/xiaopansky/Sketch/raw/master/sample/src/main/res/drawable-mdpi/ic_launcher.png) Sketch

Sketch是Android上的一个图片加载器，目的是为了帮助开发者从本地或网络读取图片，然后处理并显示在ImageView上

Sketch is an image loader for Android, the purpose is to help the developers to read a picture from a local or network, and then processed and displayed on the ImageView.

![sample](https://github.com/xiaopansky/Sketch/raw/master/docs/sample.jpg)

###特点（Features）
>* ``支持GIF图片``. 默认支持GIF图片，直接导入android-gif-drawable即可使用
>* ``多种URI支持``. 支持``http://``、``https://``、``asset://``、``content://``、``/sdcard/sample.jpg``、``drawable://``等6种URI。
>* ``异步加载``. 采用线程池来处理每一个请求，并且网络加载和本地加载会放在不同的线程池中执行，保证不会因为网络加载而堵塞本地加载。
>* ``支持缓存``. 采用Lru算法在本地和内存中缓存图片，本地缓存可设置最大容量、保留容量以及有效期。
>* ``支持ViewHolder``. 即使你在ListView中使用了ViewHolder也依然可以使用ImageLoader来加载图片，并且图片显示绝对不会混乱。
>* ``SketchImageView``. 提供更强大的SketchImageView，只需调用displayImage***系列方法即可显示各种图片。
>* ``重复下载过滤``. 如果两个请求的图片地址一样的话，第二个就会等待，一直到第一个下载成功后才会继续处理。
>* ``即时取消无用请求``. SketchImageView在onDetachedFromWindow或被重复利用的时候会及时取消之前的请求。
>* ``支持进度回调``. 通过progressListener()方法即可设置并开启进度回调。
>* ``防止加载过大Bitmap`` 提供maxSize参数来控制加载到内存的图片的尺寸（默认为当前屏幕宽高的1.5倍），另外还会自动根据ImageView的layout size来调整maxSize。
>* ``裁剪图片``. 通过resize可对图片进行裁剪，另外还会自动根据ImageView的layout size来调整resize。
>* ``多种级别供你自由玩转图片``. 除了display()方法可用来显示图片之外，你还可以通过load()方法加载图片或通过download()方法下载图片。
>* ``强大的自定义功能``. 可自定义请求分发与执行、缓存、解码、处理、显示、默认图片、失败图片等。
>* ``提供RequestOptions``. 通过RequestOptions你可以提前定义好一系列的属性，然后在显示图片的时候一次性设置，另外你还可以通过Sketch.putOptions(Enum<?>, RequestOptions)存储RequestOptions。然后在使用的时候指定名称即可。
>* ``使用方便``. 直接通过Sketch.with(Context).display()方法即可显示图片，无需事先在Application中做任何设置
>* ``兼容RecyclerView``. RecyclerView增加了一些新的特性，导致在onDetachedFromWindow()中直接回收图片或设置drawable为null会导致一些显示异常和崩溃，现已完美兼容

###示例APP（Sample app）
>* [Get it on Google Play](https://play.google.com/store/apps/details?memoryCacheId=me.xiaoapn.android.imageloader)
>* [Download APK](https://github.com/xiaopansky/Sketch/raw/master/releases/SketchSample-2.5.1.apk)

扫描二维码下载示例APP

![SampleApp](https://github.com/xiaopansky/Sketch/raw/master/releases/sample_apk_download_qr.png)

###简介（Introduction）

####支持的URI以及使用的方法（Support URI and the use of the method）：

|Type|Scheme|Fetch method used in SketchImageView|
|:--|:--|:--|
|File in network|http://, https:// |displayImage(String)|
|File in SDCard|/|displayImage(String)|
|Content Provider|content://|displayContentImage(Uri)|
|Asset in app|asset://|displayAssetImage(String)|
|Resource in app|resource://|displayRecourceImage(int)|

####支持的图片类型（Support picture type）

|Type|Scheme|jpeg|png|webp|gif|apk icon|
|:--|:--|:--|:--|:--|:--|:--|:--|
|File in network|http://, http:// |yes|yes|yes|yes|no|
|File in SDCard|/|yes|yes|yes|yes|yes|
|Content Provider|content://|yes|yes|yes|yes|no|
|Asset in app|asset://|yes|yes|yes|yes|no|
|Resource in app|resource://|yes|yes|yes|yes|no|

示例（Sample）：
```java
SketchImageView sketchImageView = ...;

// display from image network
sketchImageView.displayImage("http://b.zol-img.com.cn/desk/bizhi/image/4/1366x768/1387347695254.jpg");

// display image from SDCard
sketchImageView.displayImage("/sdcard0/sample.png");

// display APK icon
sketchImageView.displayImage("/sdcard0/google_play.apk");

// display resource drawable
sketchImageView.displayResourceImage(R.drawable.sample);

// display image from asset 
sketchImageView.displayAssetImage("sample.jpg");

// display image from ContentProvider
Uri uri = ...;
sketchImageView.displayContentImage(uri);
```
[点击查看SketchImageView详细使用说明（Click to view the SketchImageView detailed instructions）](https://github.com/xiaopansky/Sketch/wiki/SketchImageView)

####download()、load()、display()
Sketch除了有display()方法用来显示图片之外，还有load()用来加载图片和download()方法用来下载图片

这里会弄一个流程图来表示整个过程

>* download()：下载网络图片，此方法仅仅实现将图片下载到本地；
>* load()：加载图片，此方法在将图片下载到本地之后会将图片加载到内存并实现本地缓存功能；
>* display()：显示图片，此方法在将图片下载并加载到内存之后，会将图片放到内存缓存中，然后显示在ImageView上，并实现内存缓存。

实际上整个显示图片的过程可分为下载、加载和显示三部分，这三个方法正好对应这三部分，因此你可以根据你的需求选择不同的方法来处理图片

``这三个方法的用法都一样``

display()与load()、download()的区别
>* display()的fire()方法必须在主线程执行，否则将会有异常发生
>* 在使用display()方法显示图片的时候，Sketch会自动根据ImageView的layout size计算maxSize
>* 可使用的属性display()最多，download()最少具体如下表所示：

对应关系

|属性|download|load|display|
|:--|:--|:--|:--|
|使用方法|Sketch.download()|Sketch.load()|Sketch.display()|
|返回的helper|DownloadHelper|LoadHelper|DisplayHelper|
|使用的options|DownloadOptions|LoadOptions|DisplayOptions|

下面是属性表（'-'代表不支持）

|属性|download()|load()|display()|
|:--|:--|:--|:--|
|enableDiskCache|true|true|true|
|maxSize|-|屏幕的1.5倍|ImageView的layout size 或屏幕的1.5倍|
|resize|-|null|null|
|imageProcessor|-|null|null|
|scaleType|-|FIT_CENTER|FIT_CENTER|
|enableMemoryCache|-|-|true|
|imageDisplayer|-|-|DefaultImageDisplayer|
|loadingDrawable|-|-|null|
|loadFailDrawable|-|-|null|
|listener|null|null|null|
|progressListener|null|null|null|


####你可能还感兴趣的功能：
>* [使用``SketchImageView``代替ImageView快速显示图片](https://github.com/xiaopansky/Sketch/wiki/SketchImageView)
>* [处理图片成``圆形``的、``椭圆形``的或者加上``倒影效果``（ImageProcessor）](https://github.com/xiaopansky/Sketch/wiki/ImageProcessor)
>* [以``渐变``或``缩放``的方式显示图片（ImageDisplayer）](https://github.com/xiaopansky/Sketch/wiki/ImageDisplayer)
>* [使用``maxSize``防止加载过大的图片以``节省内存``](https://github.com/xiaopansky/Sketch/wiki/maxSize)
>* [使用``resize``裁剪图片](https://github.com/xiaopansky/Sketch/wiki/resize)
>* [使用``RequestOptions``定义属性模板来简化属性设置](https://github.com/xiaopansky/Sketch/wiki/RequestOptions)
>* [监听加载``开始``、``成功``、``失败``以及``进度``](https://github.com/xiaopansky/Sketch/wiki/listener)
>* [自定义``inSampleSize``计算规则或``图片解码器``（ImageDecoder）](https://github.com/xiaopansky/Sketch/wiki/ImageDecoder)
>* [设置``内存缓存最大容量``（MemoryCache）](https://github.com/xiaopansky/Sketch/wiki/MemoryCache)
>* [设置``磁盘缓存最大容量``、``磁盘缓存目录``或``保留空间大小``（DiskCache）](https://github.com/xiaopansky/Sketch/wiki/DiskCache)
>* [设置下载``失败重试次数``、``超时时间``（ImageDownloader）](https://github.com/xiaopansky/Sketch/wiki/ImageDownloader)
>* [取消请求](https://github.com/xiaopansky/Sketch/wiki/CancelRequest)
>* [自定义``请求执行顺序``、``任务队列长度``或``线程池大小``（RequestExecutor）](https://github.com/xiaopansky/Sketch/wiki/RequestExecutor)
>* [使用ImageView](https://github.com/xiaopansky/Sketch/wiki/UseImageView)
>* [暂停加载新图片，进一步提升列表流畅度](https://github.com/xiaopansky/Sketch/wiki/pause-or-resume-sketch)

###使用指南（Usage guide）
####导入Sketch（Getting started with Sketch）

>* [sketch-1.3.0.jar](https://github.com/xiaopansky/Sketch/raw/master/releases/sketch-1.3.0.jar)
>* [sketch-1.3.0-sources.zip](https://github.com/xiaopansky/Sketch/raw/master/releases/sketch-1.3.0-sources.zip)

####配置权限（Configure the required permissions）
在AndroidManifest.xml文件中添加以下权限
```xml
<uses-permission android:name="android.permission.INTERNET"/>
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
```

####在XML中使用SketchImageView
res/layout/item_user.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<me.xiaopan.SketchImageView
    android:id="@+id/image_main_head"
    android:layout_width="130dp"
    android:layout_height="130dp"
  />
```

####在Adapter中设置URI显示图片
```java
@Override
public View getView(int position, View convertView, ViewGroup parent) {
	if(convertView == null){
		convertView = LayoutInflater.from(context).inflate(R.layout.item_user, parent, false);
	}

	SketchImageView headImageView = (SketchImageView) convertView;
	headImageView.displayImage("http://b.zol-img.com.cn/desk/bizhi/image/4/1366x768/1387347695254.jpg");
	return convertView;
}
```

###新计划（New plan）
####下一版开发计划（The next version of the development plan）
>* 支持读取已安装APP或本地APK文件的图标（可能要新增一个类型，比如app://com.android.core或/sdcard/test.apk）
>* 示例APP增加一个页面，展示读取已安装APP或本地APK文件的图标的功能。页面分两部分，分别显示已安装APP列表和扫描到的本地APK包列表
>* 修复首页竖的默认图貌似裁剪跑偏的问题
>* 增加在移动网络下不加载网络图片的功能，另外结合SketchImageView支持点击加载（是否要新增加一个可以提示“点击加载”的默认图片，同loadingDrawable时同级的）
>* 支持外部添加一个Bitmap到内存缓存中，这样将会大大增加灵活性（那么外部将有权利设置缓存ID以及决定是否用RecycleDrawable就要放在MemoryCache中了）
>* 考虑将默认图也放到内存缓存中，试图通过这样的方式解决之前担心的默认图太多导致始终占用内存的问题
>* 考虑如何支持用已缓存的小缩略图作为默认图片（比如支持从内存缓存中加载默认图）
>* 改善框架实现，便于使用者自定义整个流程中的每个环节

####需求池（Demand pool）
>* 结合android-gif-drawable支持GIF图
>* loading图片支持直接设置bitmap以及各种类型的Drawable
>* 优化imagedecoder解码图片部分的实现，现在的局限性比较大，不方便自定义，比如去掉各种decodelistener
>* DrawableHolder还要改，貌似有的时候不需要解码生成新的图片
>* 考虑支持图片下载断点续传
>* 解码缓存文件失败的时候支持删除缓存文件并重新次下载
>* 考虑如何处理因异常（比如程序被强行终止）产生的下载垃圾文件，比如在申请空间的时候清理那些下载垃圾文件
>* 考虑一下怎么能让使用者更加方便的自定义新的加载方式，就比如我要显示apk的图标，却因为现有框架的限制导致必须开发者来支持才可以

###更新日志（Change log）
####1.3.0
**SketchImageView**
>* ``修复``. 兼容RecyclerView，因为在RecyclerView中View的生命周期略有变化，导致图片显示异常，现已修复
>* ``修复``. 取消了在setImageFromUri()方法中的过滤请求功能，因为这里只能根据URI过滤。例如：同一个URI在同一个SketchImageView上调用setImageFromUri()方法显示了两次，但是这两次显示的时候SketchImageView的宽高是不一样的，结果就是第一次的显示请求继续执行，第二次的显示请求被拒绝了。现在去掉过滤功能后统一都交给了Sketch处理，结果会是第一次的显示请求被取消，第二次的显示请求继续执行。
>* ``新增``. 新增在图片表面显示进度的功能，你只需调用setEnableShowProgress(boolean)方法开启即可
>* ``优化``. debug开关不再由Sketch.isDebug()控制，而是在SketchImageView中新增了一个debugMode参数来控制
>* ``新增``. 新增类似MaterialDesign的点击涟漪效果。你只需注册点击事件或调用setClickable(true)，然后调用setEnableClickRipple(true)即可
>* ``修复``. 修复了使用SketchImageView时设置了DisplayOptions、DisplayListener等参数，但最终没有通过setImageFrom***()方法显示图片而是通过Sketch.with(context).display(imageUrl, sketchImageView)显示图片最终导致DisplayOptions、DisplayListener等参数不起作用的BUG
>* ``修改``. setImageBy***()系列方法，改名为setImageFrom***()

**Download**
>* ``优化``. 优化HttpClientImageDownloader，读取数据的时候出现异常或取消的时候主动关闭输入流，避免堵塞连接池，造成ConnectionPoolTimeoutException异常
>* ``修改``. 默认下载器改为HttpUrlConnectionImageDownloader.java，而HttpClientImageDownloader则作为备选
>* ``修改``. ImageDownloader.setTimeout()改名为setConnectTimeout()
>* ``优化``. 优化下载的实现，使其更稳定

**Cache**
>* ``删除``. 删除SoftReferenceMemoryCache.java
>* ``移动``. 移动DiskCache.java、LruDiskCache.java、LruMemoryCache.java、MemoryCache.java到cache目录下
>* ``优化``. 调整LruDiskCache的默认保留空间为100M
>* ``新增``. LruDiskCache增加maxSize功能
>* ``修复``. 修复在2.3及以下缓存RecyclingBitmapDrawable的时候忘记添加计数导致Bitmap被提前回收而引发崩溃的BUG
>* ``删除``. 去掉了diskCacheTimeout功能，事实证明这个功能没多大用处，并且还影响了当容量不足时清理文件的功能

**Decode**
>* ``优化``. 优化了默认的inSampleSize的计算方法，增加了限制图片像素数超过目标尺寸像素的两倍，这样可以有效防止那些一边特小一边特大的图片，以特大的姿态被加载到内存中
>* ``优化``. 将计算默认maxSize的代码封装成一个方法并放到了ImageSizeCalculator.java中
>* ``修复``. 计算maxSize的时候不再考虑ImageView的getWidth()和getHeight()，这是因为当ImageView的宽高是固定的，在循环重复利用的时候从第二次循环利用开始，最终计算出来的size都将是上一次的size，显然这是个很严重的BUG。当所有的ImageView的宽高都是一样的时候看不出来这个问题，都不一样的时候问题就出来了。
>* ``优化``. 默认解码器在遇到1x1的图片时按照失败处理

**Display**
>* ``优化``. 优化了默认ImageDisplayer的实现方式
>* ``修改``. 修改ColorFadeInImageDisplayer的名字为ColorTransitionImageDisplayer；OriginalFadeInImageDisplayer的名字为TransitionImageDisplayer
>* ``修改``. 当你使用TransitionImageDisplayer作为displayer的时候会默认开启resizeByImageViewLayoutSize功能，因为不开启resizeByImageViewLayoutSize的话图片最终就会显示变形

**Execute**
>* ``优化``. 默认任务执行器的任务队列的长度由20调整为200，这是由于如果你一次性要显示大量的图片，队列长度比较小的话，后面的将会出现异常
>* ``优化``. 默认任务执行器的线程池的keepAliveTime时间由1秒改为60秒

**Process**
>* ``修复``. 计算resize的时候不再考虑ImageView的getWidth()和getHeight()，这是因为当ImageView的宽高是固定的，在循环重复利用的时候从第二次循环利用开始，最终计算出来的size都将是上一次的size，显然这是个很严重的BUG。当所有的ImageView的宽高都是一样的时候看不出来这个问题，都不一样的时候问题就出来了。
>* ``优化``. 优化了默认ImageProcessor的实现方式
>* ``优化``. 优化自带的几种图片处理器，对ScaleType支持更完善，更准确

**Request**
>* ``修改``. DisplayListener.From.LOCAL改名为DisplayListener.From.DISK

**Sketch**
>* ``优化``. 将一些配置移到了Configuration.java中，debugMode的设置直接改成了静态的
>* ``新增``. 增加pause功能，你可以在列表滚动时调用pause()方法暂停加载新图片，在列表停止滚动后调用resume()方法恢复并刷新列表，通过这样的手段来提高列表滑动流畅度
>* ``修改``. image uri不再支持“file:///mnt/sdcard/image.png”，直接支持“/mnt/sdcard/image.png”
>* ``修复``. 修复了由于DisplayHelper、LoadHelper、DownloadHelper的options()方法参数为null时返回了一个null对象的BUG，这会导致使用SketchImageView时由于没有设置DisplayOptions而引起崩溃
>* ``修改``. 修改DisplayHelper中loadFailedDrawable()方法的名称为loadFailDrawable()
>* ``修复``. 修复DisplayHelper、LoadHelper、DownloadHelper中调用options()方法设置参数的时候会直接覆盖Helper中的参数的BUG，修改后的规则是如果helper中为null，且Options中的参数被设置过才会覆盖
>* ``优化``. 默认图片和失败图片使用ImageProcessor处理时支持使用DisplayHelper中的resize和scaleType
>* ``优化``. 调用display()方法显示图片时，当uri为null或空时显示loadingDrawable
>* ``优化``. display的fire方法去掉了异步线程过滤，由于display基本都是在主线程执行的过滤异步线程没有意义
>* ``修改``. 不再默认根据ImageView的Layout Size设置resize，新增resizeByImageViewLayoutSize()方法开启此功能。另外当你使用TransitionImageDisplayer作为displayer的时候会默认开启resizeByImageViewLayoutSize功能，因为不开启resizeByImageViewLayoutSize的话图片最终就会显示变形

####1.2.0
>* ``优化``. 改善了需要通过Handler在主线程执行回调以及显示的方式，以前是使用Runnable，现在时通过Message，这样就避免了创建Runnable，由于display是非常频繁的操作，因此这将会是有意义的改善
>* ``优化``. 优化了DisplayHelper的使用，以前是为每一次display都创建一个DisplayHelper，现在是只要你是按照display().fire()这样连续的使用，那么所有的display将共用一个DisplayHelper，这将会避免创建大量的DisplayHelper
>* ``优化``. ProgressListener.onUpdateProgress(long, long)改为ProgressListener.onUpdateProgress(int, int)，因为int足够用了

[查看更多...](https://github.com/xiaopansky/Sketch/wiki/Change-log)

###License
```java
/*
 * Copyright (C) 2013 Peng fei Pan <sky@xiaopan.me>
 * 
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 * 
 *   http://www.apache.org/licenses/LICENSE-2.0
 * 
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */
```
