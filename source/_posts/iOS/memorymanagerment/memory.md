---
title: 内存管理
date: 2015-07-10 12:24:27
tags: 
- iOS
- Objective-C
- iOS内存管理
categories: 
- iOS
- iOS内存管理
---


## `autoReleasePool` 什么时候释放?

`App`启动后，苹果在主线程 `RunLoop` 里注册了两个 `Observer`，其回调都是 `_wrapRunLoopWithAutoreleasePoolHandler()`。

第一个 `Observer` 监视的事件是 `Entry(即将进入Loop)`，其回调内会调用 `_objc_autoreleasePoolPush()` 创建自动释放池。其 `order` 是 `-2147483647`，优先级最高，保证创建释放池发生在其他所有回调之前。

第二个 `Observer` 监视了两个事件： `BeforeWaiting`(准备进入休眠) 时调用`_objc_autoreleasePoolPop()`  和 `_objc_autoreleasePoolPush()` 释放旧的池并创建新池；`Exit`(即将退出Loop) 时调用 `_objc_autoreleasePoolPop()` 来释放自动释放池。这个 `Observer` 的 `order` 是 `2147483647`，优先级最低，保证其释放池子发生在其他所有回调之后。

在主线程执行的代码，通常是写在诸如事件回调、`Timer`回调内的。这些回调会被 `RunLoop` 创建好的 `AutoreleasePool` 环绕着，所以不会出现内存泄漏，开发者也不必显示创建 `Pool` 了。

## 在 `Obj-C` 中，如何检测内存泄漏？你知道哪些方式？
目前我知道的方式有以下几种

* Memory Leaks
* Alloctions
* Analyse
* Debug Memory Graph
* MLeaksFinder

泄露的内存主要有以下两种：

* `Laek Memory` 这种是忘记 `Release` 操作所泄露的内存。
* `Abandon Memory` 这种是循环引用，无法释放掉的内存。


上面所说的五种方式，其实前四种都比较麻烦，需要不断地调试运行，第五种是腾讯阅读团队出品，效果好一些，感兴趣的可以看一下这两篇文章：

- [MLeaksFinder：精准 iOS 内存泄露检测工具](http://wereadteam.github.io/2016/02/22/MLeaksFinder/)
- [MLeaksFinder 新特性](http://wereadteam.github.io/2016/07/20/MLeaksFinder2/)



