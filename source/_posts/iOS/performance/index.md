---
title: iOS性能优化
date: 2015-07-10 12:24:27
tags: 
- iOS性能优化 
categories: 
- iOS
- iOS性能优化
---

## 如何提升 `tableview` 的流畅度？


本质上是降低 CPU、GPU 的工作，从这两个大的方面去提升性能。

- CPU：对象的创建和销毁、对象属性的调整、布局计算、文本的计算和排版、图片的格式转换和解码、图像的绘制
- GPU：纹理的渲染


### 卡顿优化在 CPU 层面
* 尽量用轻量级的对象，比如用不到事件处理的地方，可以考虑使用 CALayer 取代 UIView
* 不要频繁地调用 UIView 的相关属性，比如 frame、bounds、transform 等属性，尽量减少不必要的修改
* 尽量提前计算好布局，在有需要时一次性调整对应的属性，不要多次修改属性
* Autolayout 会比直接设置 frame 消耗更多的 CPU 资源
* 图片的 size 最好刚好跟 UIImageView 的 size 保持一致
* 控制一下线程的最大并发数量
* 尽量把耗时的操作放到子线程
    * 文本处理（尺寸计算、绘制）
    * 图片处理（解码、绘制）
    
    
### 卡顿优化在 GPU层面

* 尽量避免短时间内大量图片的显示，尽可能将多张图片合成一张进行显示
* GPU能处理的最大纹理尺寸是 4096x4096，一旦超过这个尺寸，就会占用 CPU 资源进行处理，所以纹理尽量不要超过这个尺寸
* 尽量减少视图数量和层次
* 减少透明的视图（alpha<1），不透明的就设置 opaque 为 YES
* 尽量避免出现离屏渲染

[iOS 保持界面流畅的技巧](https://blog.ibireme.com/2015/11/12/smooth_user_interfaces_for_ios/)


#### 1.预排版，提前计算
在接收到服务端返回的数据后，尽量将 `CoreText` 排版的结果、单个控件的高度、`cell` 整体的高度提前计算好，将其存储在模型的属性中。需要使用时，直接从模型中往外取，避免了计算的过程。

尽量少用 `UILabel`，可以使用 `CALayer` 。避免使用 `AutoLayout` 的自动布局技术，采取纯代码的方式 

#### 2.预渲染，提前绘制

例如圆形的图标可以提前在，在接收到网络返回数据时，在后台线程进行处理，直接存储在模型数据里，回到主线程后直接调用就可以了

避免使用 CALayer 的 Border、corner、shadow、mask 等技术，这些都会触发离屏渲染。

#### 3.异步绘制


#### 4.全局并发线程

#### 5.高效的图片异步加载


## 如何有效降低 `APP` 包的大小？

降低包大小需要从两方面着手

### 可执行文件

* 编译器优化
    - Strip Linked Product、Make Strings Read-Only、Symbols Hidden by Default 设置为 YES
    - 去掉异常支持，Enable C++ Exceptions、Enable Objective-C Exceptions 设置为 NO， Other C Flags 添加 -fno-exceptions   
* 利用 AppCode 检测未使用的代码：菜单栏 -> Code -> Inspect Code

* 编写LLVM插件检测出重复代码、未被调用的代码




### 资源

> 资源包括 图片、音频、视频 等

* 优化的方式可以对资源进行无损的压缩
* 去除没有用到的资源： https://github.com/tinymind/LSUnusedResources


## 继续优化

### 1、重用和延迟加载(lazy load) Views

- 更多的view意味着更多的渲染，也就是更多的CPU和内存消耗，对于那种嵌套了很多view在UIScrollView里边的app更是如此。
- 这里我们用到的技巧就是模仿UITableView和UICollectionView的操作: 不要一次创建所有的subview，而是当需要时才创建，当它们完成了使命，把他们放进一个可重用的队列中。这样的话你就只需要在滚动发生时创建你的views，避免了不划算的内存分配。



### 2、Cache, Cache, 还是Cache!

- 一个极好的原则就是，缓存所需要的，也就是那些不大可能改变但是需要经常读取的东西。
- 我们能缓存些什么呢？一些选项是，远端服务器的响应，图片，甚至计算结果，比如UITableView的行高。
- NSCache和NSDictionary类似，不同的是系统回收内存的时候它会自动删掉它的内容。

### 3、权衡渲染方法.性能能还是要bundle保持合适的大小。

### 4、处理内存警告.移除对缓存，图片object和其他一些可以重创建的objects的strong references.

### 5、重用大开销对象

### 6、一些objects的初始化很慢，比如NSDateFormatter和NSCalendar。然而，你又不可避免地需要使用它们，比如从JSON或者XML中解析数据。想要避免使用这个对象的瓶颈你就需要重用他们，可以通过添加属性到你的class里或者创建静态变量来实现。

### 7、避免反复处理数据.在服务器端和客户端使用相同的数据结构很重要。

### 8、选择正确的数据格式.解析JSON会比XML更快一些，JSON也通常更小更便于传输。从iOS5起有了官方内建的JSON deserialization 就更加方便使用了。但是XML也有XML的好处，比如使用SAX 来解析XML就像解析本地文件一样，你不需像解析json一样等到整个文档下载完成才开始解析。当你处理很大的数据的时候就会极大地减低内存消耗和增加性能。

### 9、正确设定背景图片

- 全屏背景图，在view中添加一个UIImageView作为一个子View
- 只是某个小的view的背景图，你就需要用UIColor的colorWithPatternImage来做了，它会更快地渲染也不会花费很多内存：

### 10、减少使用Web特性。想要更高的性能你就要调整下你的HTML了。第一件要做的事就是尽可能移除不必要的javascript，避免使用过大的框架。能只用原生js就更好了。尽可能异步加载例如用户行为统计script这种不影响页面表达的javascript。注意你使用的图片，保证图片的符合你使用的大小。

### 11、Shadow Path 。CoreAnimation不得不先在后台得出你的图形并加好阴影然后才渲染，这开销是很大的。使用shadowPath的话就避免了这个问题。使用shadow path的话iOS就不必每次都计算如何渲染，它使用一个预先计算好的路径。但问题是自己计算path的话可能在某些View中比较困难，且每当view的frame变化的时候你都需要去update shadow path.

### 12、优化Table View

- 正确使用reuseIdentifier来重用cells
- 尽量使所有的view opaque，包括cell自身
- 避免渐变，图片缩放，后台选人
- 缓存行高
- 如果cell内现实的内容来自web，使用异步加载，缓存请求结果
- 使用shadowPath来画阴影
- 减少subviews的数量
- 尽量不适用cellForRowAtIndexPath:，如果你需要用到它，只用-一次然后缓存结果
- 使用正确的数据结构来存储数据
- 使用rowHeight, sectionFooterHeight 和 sectionHeaderHeight来设定固定的高，不要请求delegate

### 13、选择正确的数据存储选项

- NSUserDefaults的问题是什么？虽然它很nice也很便捷，但是它只适用于小数据，比如一些简单的布尔型的设置选项，再大点你就要考虑其它方式了
- XML这种结构化档案呢？总体来说，你需要读取整个文件到内存里去解析，这样是很不经济的。使用SAX又是一个很麻烦的事情。
- NSCoding？不幸的是，它也需要读写文件，所以也有以上问题。
- 在这种应用场景下，使用SQLite 或者 Core Data比较好。使用这些技术你用特定的查询语句就能只加载你需要的对象。
- 在性能层面来讲，SQLite和Core Data是很相似的。他们的不同在于具体使用方法。
- Core Data代表一个对象的graph model，但SQLite就是一个DBMS。
- Apple在一般情况下建议使用Core Data，但是如果你有理由不使用它，那么就去使用更加底层的SQLite吧。
- 如果你使用SQLite，你可以用FMDB这个库来简化SQLite的操作，这样你就不用花很多经历了解SQLite的C API了。

