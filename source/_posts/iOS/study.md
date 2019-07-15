---
title: 学习
date: 2017-10-10 11:26:39
update: 2017-07-10
tags: iOS课程学习笔记
categories: 
- iOS
- iOS课程学习笔记
# description: 记录一些平时收集的，感悟的东西

---

[学习网路课程地址](https://coding.imooc.com/class/202.html) ,不过是收费的366rmb。以下是笔记

# 高级
- 解决研发过程中的`关键问题`和`技术难题`
- `调优`设备流量、性能、电量
- `较强`的软件设计能力
- 对iOS内部原理有`深刻理解`
# 资深
- `精通`高性能编程以及性能调优
- `灵活运用`数据结构、算法解决`复杂程序设计`问题
- 提供性能优化、日志搜索、统计分析等`方案`
- 架构、模块`设计`

# 简历
- 排版清晰
- 重要和突出的表达
- 线上bug数变化、量化指标 质量上的指标
- 开发成本节约了多少 量化指标
- 基本信息、工作经历、项目经验、擅长技能
- 项目经验(主导、参与)，背景、方案、效果

# UI视图
- UITableView
- 事件传递&视图响应
- 图像显示原理
- 卡顿&掉帧
- 绘制原理&异步绘制
- 离屏渲染

# UITableView
- 重用机制
- 数据源同步
    - 并发方案数据copy 串行方案
- UI事件传递&响应
    - UIView提供内容，事件响应，响应链
    - CALayar的contents负责绘制
    - 单一指责原则
# 视图显示原理
## 卡顿、掉帧的原因
- CPU
    - 对象创建、调整、销毁
    - 预排版（布局计算、文本计算）
    - 预渲染（文本等异步绘制，图片解码等）
- GPU
    - 纹理渲染
    - 视图混合
## UIView的绘制原理
- [UIView setNeedsDisplay] -> [view.layer setNeedsDisplay] -> [CALayer display]
- [CALayer display]是否响应 [layer.delegate displayLayer]方法
- 是，进入异步绘制。否，进入系统绘制流程

    - CALayer creats backing store(CGContextRef)
    - 是否有layer has a delegate
    - 是，[layer.delegate drawLayer:inContext:]
    - 否，[CALayer drawInContext:]
    - 最后 CALayer uploads backing store to GPU

## 异步绘制 [layer.delegate displayLayer]
    - 代理负责生成对应的bitmap
    - 设置该bitmap作为layer.contents属性的值

    - 流程如下
```
        - [AsyncDrawingView setNeedsDisplay]
        - [CALayer display]
        - [AsyncDrawingView displayLayer:]
            - CGBitmapContextCreat()
            - CoreGraphic API
            - CGBitmapContextCreatImage()
        - [CALayer setContents]
```

## 离屏渲染(OFF-SCREEN RENDERING)
    在当前缓冲区创建新的缓冲区渲染
    缓冲区渲染，圆角(和maskToBounds一起使用)，孟层遮罩，阴影，光栅化

* 为何要避免离屏渲染?
- 离屏渲染会增加GPU的工作量, CPU+GPU提交一帧的时间超过了16.7ms，造成了不满60帧的卡顿
- 创建新的渲染缓冲区
- 上下文切换

# OC语言面试问题
## 分类
### 分类都做了什么事情？
   - 声明私有方法
   - 分解体积庞大的类文件
   - framework的私有方法公开
### 特点
   - 运行时决议（runtime时添加）
    - 可以为系统类添加分类
### 分类中可以添加哪些内容
   - 实例方法
    - 类方法
    - 协议
    - 属性（只是声明了set和get方法，而不是私有属性）
### 结构体(objc-runtime-680版本源代码)
   - const char * name
    - cls
    - instanceMethods
    - classMethods
    - protocols
    - instanceProperties
### 加载调用栈
   - _objc_init -> map_2_imags -> map_images_nolock -> _read_images -> remethodizeClass
### 源码分析
   - 效果上，分类添加的方法可以「覆盖」原类方法
   - 同名分类方法谁能生效取决于编译顺序
   - 名字相同的分类会引起编译报错

## 4-5 扩展相关面试问题
   - 声明私有属性
   - 声明私有方法
   - 声明私有成员变量
### 扩展特性
   - 编译时决议
   - 不能为系统类添加扩展
   - 只有声明

## 代理相关
- 软件设计模式
- @protocol形式提现
- 代理传递`一对一`
- weak规避循环引用
## 通知相关(NSNotification)
- 是使用`观察者模式`来实现的用于跨层传递消息的机制（网络层，ui层传递等等）
- 一对多

## KVO
- KVO是Objective-C对`观察者设计模式`的有一种实现
- Apple使用了isa混写（isa-swizzling）来实现kvo
- NSKVONotifying_A （子类）重写setter方法
```
- (void)setValue:(id)obj {
    [self willChangeValueForKey:@"keyPath"];
    //调用父类实现，也即原类的实现
    [supe setValue:obj];
    [self didChangeValueForKey:@"keyPath"];
}
```
- 1、通过kvc设置value能否生效？
- 2、通过成员变量直接赋值value能否生效？

## KVC (Key-value coding)键值编码
- valueForKey流程图

{% fi /images/iOS/KVC.png, KVC流程图， 100% %}

<!-- ![流程图](/images/iOS/KVC.png) -->
- Accessor Method
    - `<getKey>`
    - `<Key>`
    - `<isKey>`

- setValueForKey流程图
{% fi /images/iOS/kvc-set.png, KVC流程图， 100% %}

# RunTime
- 数据结构
- 类对象与元类对象
- 消息传递
- 方法缓存
- 消息转发
- Methond-Swizzling
- 动态添加方法
- 动态方法解析

## 数据结构
objc_object
- isa_t 
- 关于isa操作相关
- 弱引用相关
- 关联对象相关
- 内存管理
objc_class
- Class = objc_class继承自objc_object
    - Class puserClass
    - cache_t cache
    - class_data_bits_t bits;
isa指针
- 指针型isa，的内存地址代表Class的地址
- 非指针型的呢值，代表Class的地址
isa指向
- 关于对象，指向类对象，-class
- 类对象，元类对象， Class——isa-MetaClass
cache_t
- 用于`快速`查找方法执行函数,消息传递的速度
- 是`可增量扩展`的`哈希表`结构
- 是`局部性原理`的最佳应用(调用频率越高，越容易命中)
- 数据结构
    - bucket_t 这样的数组
        - key   
        - IMP
class_data_bits_t
- 主要是对class_rw_t的封装
    - class_ro_t
    - protocols(二维数组)
    - properties(二维数组)
    - methods(二维数组)
- clcass_rw_t 代表了类相关的读写信息、对class_ro_t的封装
- class_ro_t
    - name(类名等...)
    - ivars(二维数组)
    - properties(二维数组)
    - protocols(二维数组)
    - methodlist(二维数组)
- method_t
    - struct method_t
    = name。。。SEL name; 名称
    - chonst char* types; 函数体
        - * 返回值 参数1.。。参数n
        - v@: void， id  SEL
    - IMP imp;  实现

{% fi /images/iOS/runtime-data-structure.png, runtime结构体， 100% %}

# 类对象，对象(实例，isa指向类对象)，
实例对象可以通过isa指针找到类对象，类对象的isa指针可以找到元类对象,元类对象isa指向根原类对象，根元类对象isa指向根类对象

## 消息传递 
- void objc_mesSend(void /* id self, SEL op, ...*/)
- void objc_mesSendSuper(void /* struct objc_super * super, SEL op, ...*/)

### 缓存查找 哈希查找
通过SEL->hash->key找多对应的bucket_t的IMP
- 对于`已排序好`的列表,采用`二分法`查找对应执行函数
- 对于没有`排序好`的列表，采用`一般遍历`
- 缓存查找是否命中，  方法列表是否命中， 逐级父类方法是否命中, 消息转发
### x消息转发
#### 实例方法的转发
- 类方法 resolveInstanceMethode: 返回值bool。yes已处理，消息转发结束
- forwarding TargetForSelector:  返回值是id，因该有那个对象来处理，返回nil进入第三次转发
- methodSignatureForSelector: 返回方法签名,返回nil，消息无法处理
- forwardInvocation：

### Method-Swizzling
### 动态添加方法
- 通过消息转发机制,动态添加方法。在resolveInstanceMethod

### 动态方法解析
- @dynamic  动态运行时语言将函数决议推迟到运行时
- 编译时语言在编译器进行函数决议

### [obj foo]和objc_msgSend()函数有什么关系？
- [obj foo]在编译之后，就变成了和objc_msgSend()

### runtime如何通过Selector找到对应的IMP地址？
- 先从缓存方法列表----待补充
### 能否向编译后的类中增加实例变量？
- 不能！编译后结构体class_ro_t是只读属性

# 内存管理
- TaggedPointer
- NONPOINTER_ISA
- 散列表(引用技术表和弱引用表)

## NONPOINTER_ISA
## 散列表 Side Tables()结构
- 自旋锁
- 引用计数表
- 弱引用表
> 分离锁，提升并发问题
- Spinlock_t 自旋锁
    - Spinlock_t是『忙等』的锁
    - 轻量访问
- RefcountMap Hash表
- weak_table_t 弱引用表 Hash表

## MRC & ARC
- ARC是LLVM和Runtime协作的结果
- ARC禁止调用 retain/等
- ARC中新增weak，strong等属性关键字

## 引用计数管理
### retain实现
```
    SideTable& table = SideTables()[This];（哈西查找）
    size_t& refcntStorage = table.refcnts[This];   size_t就是无符号long型
    refcnStorage += SIDE_TABLE_RC_ONE;
```
### dealloc实现
1. _objc_rootDeallco()
2. rootDealloc()
3. 是否可以释放
    - nonpointer_isa
    - weakly_referenced
    - has_assoc
    - has_cxx_dtor
    - has_sidetable_rc
4. 3全部为no，调用object_dospose().否则可直接c函数free()释放

### 弱引用管理

## 自动释放池
* 是以`栈`为节点通过`双向链表`的形式结合而成
* 是和`线程`一一对应的
### 双向链表
{
    id * next;
    AutoReleasePoolPage * const parent;
    AutoReleasePoolPage * child;
    pthread_t const thread
}
多层嵌套就是多次插入哨兵对象

## 循环引用
- 自循环引用
- 相互循环引用
- 多循环引用

### 如何破除循环引用
- 避免产生循环引用
- 在合适的时机手动断环
* __block
    - MRC下，__block修饰不会增加引用计数，避免了循环引用
    - ARC下，__block修饰会被强引用，无法避免循环引用，需手动解环
- NSTimer的循环引用问题
    - nstimer 会对target强引用，Runloop会强制引用NSTimer；
    - 添加中间对象，NSTimer和对象之间
    - invalidInvocation


# Block
> Block是将`函数`以及`执行上下文`封装起来的`对象`.
使用【clang -rewite-objc file.m】查看编译之后的文件内容
## 截获变量
- 对于`基本数据`类型的`局部变量`截获其值
- 对于`对象`类型的局部变量`连同所有权修饰符`一起截获
- 以`指针形式`截获局部静态变量
- 不截获全局变量、静态全局变量
使用【clang -rewite-objc -fobjc-arc file.m】查看编译之后的文件内容
## __block修饰符
- `一般情况下`，对被截获的变量进行`赋值`操纵时，需要添加`__block修饰符`
## Block的内存管理
无论任何位置，都可以顺利的用__block修饰变量访问到
## 循环引用问题
- __block在MRC下不会引用计数+1，在arc下会引用计数+1

# 多线程
- GCD
- NSOperation
- NSTread （常驻线程）
- 多线程与锁
## GCD
- 同步、异步和串行/并发
- dispath_barrier_async 针对多读单写问题,栅栏函数
- dispath_group
### 同步串行
- 死锁原因，是队列引起的 循环等待
### 同步并发
### 异步串行
### 异步并发 performSelector: delay

## dispath_barrier_async()
## dispath_group_async()
使用GCD实现：ABC执行完，再执行D
> dispatch_group_notify(group,....){}
## NSOperation
* 需要和NSOperationQueue
- 添加任务依赖
- 任务执行状态控制
- 最大并发量

### 任务执行状态控制
- isReady
- isExecuting
- isFinished
- isCancelled
只重写了`main`方法，底层控制变更任务执行完成状态，以及任务退出
如果重写了`star`方法，自行控制任务状态
* gnustep=base-1.24.9
> 系统通过KVO的形势移出一个isFinished为YES的NSOperation

## NSThread
启动流程 star() -> 创建pthread -> main() -> [target performSelector:SEL] -> exit()

## 多线程和锁
- @synchronized
    - 创建单利对象的使用
- atomic
    - 属性关键字，赋值操作线程安全，其它不安全
- OSSpinLock
    - 自旋锁，`循环等待`询问，不释放当前资源
    - 轻量级的数据访问，简单的+1、-1
- NSRecursivelock 递归锁
    - 可以重录
- NSLock
    - 互斥
    - A方法中[lock lock],再次调用[lock lock]会造成死锁，使用NSRecursivelock
- dispath_semaphore_t
    - 

# RunLoop
> RunLoop是通过内部维护的`事件循环`来对`事件/消息进行管理`的一个对象
- 没有消息需要处理时，休眠以避免资源占用
    - 用户态 -> 内核态
- 有消息需要处理时，立刻被唤醒
    - 内核态 -> 用户态

## 数据结构
NSRunLoop 是CFRunLoop的封装，提供了面向对象的API, [开源地址](https://opensource.apple.com/tarballs/CF/CF-855.17.tar.gz)

- CFRunLoop
- CFRunLoopMode
- Source/Timer/Observer

### CFRunLoop 数据结构
1. pthread          一一对应 （Runloop和线程的关系）
2. currentMode      CFRunLoopMode
3. modes            NSMutableSet<CFRunLoopMode*>
4. commonModes      NSMutableSet<NSString*>
    - `不是一种实际存在`的Mode，
    - 同步Source/Timer/Observer到多个Mode中的`一种技术方案`
5. commonModeItems  observer Timer source,

### CFRunLoopMode
1. name                 NSDefaultRunloopMode
2. source0              
3. source1
4. observers
5. timers

### CFRunLoopSource
- source0 需要手动唤醒线程
- source1 具备唤醒线程的能力
### CFRunLoopObserver
- kCFRunLoopEntry
- kCFRunloopBeforeTimers
- kCFRunLoopBeforeSources
- kCFRunLoopBeforeWaiting
- ...

> 1个RunLoop有多个Model，一个model有多个 source、Timer、Observer

## RunLoop的Mode

## s事件循环的实现机制
void CFRunLoopRun(),用户态到核心态相互切换
{% fi /images/iOS/RunLoop-machining.png, RunLoop循环机制， 100% %}
