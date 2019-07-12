---
title: 学习
date: 2017-10-10 11:26:39
update: 2017-07-10
tags: 学习记录
categories: 
# description: 记录一些平时收集的，感悟的东西

---

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

4-3
