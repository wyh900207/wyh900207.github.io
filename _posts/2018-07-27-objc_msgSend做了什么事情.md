---
layout: post
title:  "objc_msgSend做了什么事情"
date:   2018-07-10 16:01:00 +0800
description: 'OC底层'
tag: iOS, OC
---

# objc_msgSend做了什么事情

OC中的方法调用实质是发送消息(`objc_msgSend()`)

`objc_msgSend()`方法, 默认有2个必传参数:

* 接收者
* SEL选择器

#### `objc_msgSend`方法的底层, 苹果是用汇编语言实现的, 发送消息以后, 主要是查找方法, 查找方法的步骤是:

1. `消息发送`: 从实例对象对应的`Class`或者`Meta Class`中查找方法
2. `动态方法解析`: 如果`步骤1`没有找到方法, 会开始进行`resolve`, 动态解析. 在这个过程中, 如果开发者手动添加了方法实现, 那么会将方法添加到相应的`Class`或`Meta Class`中, 然后重新进行`步骤1`
3. `消息转发`: 如果`步骤2`执行结束之后, 也没有找到方法, 开始进行方法转发(`___forwarding___ 方法`), 若到此还没有查找到方法, 就会报错`不识别方法选择器`

## 一. 从实例对象对应的`Class`或者`Meta Class`中查找方法

*实例对象对应的`Class`和`Meta Class`对象实际结构是一样的, 只是存储内容的差别而已*

当`实例对象`或者`类对象`调用方法以后, 会将方法缓存到对应的`Class`或`Meta Class`中. 

**苹果特意在此添加了缓存, 提高查找速度. 并且, 不同于普通的`for`或`while`循环遍历, 苹果针对这片缓存区采用了`散列`算法, 通过空间换时间的方式, 来大大提高了查找效率.**

1. `缓存查找`: 当发送消息以后, 马上从`Class`中开始查找方法, 并且优先从`Cache`中查找
2. `方法列表查找`: 若当前`Class`对应的`Cache`中找不到方法, 开始从`Class`对应的方法列表中查找
3. `Super Class查找`: 当前实例对象的`Class`或`Meta Class`中未查找到方法以后, 会通过`Class`或`Meta Class`的`isa指针`, 找到对应的父类, 继续进行上述2个步骤, 直至找到方法, 并缓存到`Cache`中

**注意:** `步骤3`中, 在找到方法以后, 缓存到`Cache`中. 这里的`Cache`并不是指对应父类的`Cache`, 而是实例对象(方法接收者)对应的`Class`或`Meta Class`的`Cache`中.

## 二. 动态方法解析

若`第一次`无法正常从`Cache`和`方法列表`中查到方法, 则会进行动态方法解析. 一定注意, 这里说的是第一次.

动态解析:

1. 若调用的是实例方法, 系统会调用`- (void)resolveInstanceMethod:(SEL)sel`方法, 我们可以在这里, 对`sel`进行判断, 从而利用`runtime`的特性动态添加实例方法
2. 若调用的是类方法, 系统会调用`- (void)resolveClassMethod:(SEL)sel`方法, 同样可以在这里, 对`sel`进行判断, 利用`runtime`动态添加类方法
3. 在`- (void)resolveInstanceMethod:(SEL)sel`或`- (void)resolveClassMethod:(SEL)sel`方法中动态添加相应的`实例方法`或`类方法`之后, 这个方法实际上被会添加到对应的`Class`或`Meta Class`的方法列表中
4. 此时, 系统会执行`retry`操作, **第二次**从`Cache`和`Class`方法列表中查找方法, 并缓存到`Cache`中. (注意这里说的第二次)

## 三. 消息转发

若经过动态方法解析以后, 都没有找到对应的方法, 那么系统会进行最后一步操作, 叫做`消息转发`

`消息转发`会先后调用以下几个方法:

1. `forwardingTargetForSelector:` 返回值不为nil, 调用`objc_msgSend(返回值, SEL)`; 返回值为nil, 执行下一步
2. `methodSignatureForSelector:` 返回值不为nil, 调用`forwardInvocation:`方法; 返回值为nil, 执行下一步
3. `doesNotRecognizeSelector:`






