---
title: Objective-C 语言
date: 2015-07-10 12:24:27
tags: Objetive-C 
categories: 
- iOS
- Objetive-C
---

## 属性关键字

1.读写权限：readonly,readwrite(默认)
2.原子性: atomic(默认)，nonatomic。atomic读写线程安全，但效率低，而且不是绝对的安全，比如如果修饰的是数组，那么对数组的读写是安全的，但如果是操作数组进行添加移除其中对象的还，就不保证安全了。
3.引用计数：

- retain/strong
- assign：修饰基本数据类型，修饰对象类型时，不改变其引用计数，会产生悬垂指针，修饰的对象在被释放后，assign指针仍然指向原对象内存地址，如果使用assign指针继续访问原对象的话，就可能会导致内存泄漏或程序异常
- weak：不改变被修饰对象的引用计数，所指对象在被释放后，weak指针会自动置为nil
- copy：分为深拷贝和浅拷贝
浅拷贝：对内存地址的复制，让目标对象指针和原对象指向同一片内存空间会增加引用计数
深拷贝：对对象内容的复制，开辟新的内存空间

可变对象的copy和mutableCopy都是深拷贝
不可变对象的copy是浅拷贝，mutableCopy是深拷贝
copy方法返回的都是不可变对象

- @property (nonatomic, copy) NSMutableArray * array;这样写有什么影响？
因为copy方法返回的都是不可变对象，所以array对象实际上是不可变的，如果对其进行可变操作如添加移除对象，则会造成程序crash

## KVO (Key-value observing)

KVO是观察者模式的另一实现。
使用了isa混写(isa-swizzling)来实现KVO

使用setter方法改变值KVO会生效，使用setValue:forKey即KVC改变值KVO也会生效，因为KVC会去调用setter方法

``` 
- (void)setValue:(id)value
{
    [self willChangeValueForKey:@"key"];
    
    [super setValue:value];
    
    [self didChangeValueForKey:@"key"];
}
```

- 那么通过直接赋值成员变量会触发KVO吗？
不会，因为不会调用setter方法，需要加上
willChangeValueForKey和didChangeValueForKey方法来手动触发才行