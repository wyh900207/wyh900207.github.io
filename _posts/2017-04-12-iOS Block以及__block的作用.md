---
layout: post
title:  "Block以及__block的作用"
date:   2018-04-12 11:50:52 +0800
description: 'iOS Block'
tag: iOS
---
# iOS Block

#### iOS 中的block共分为3类:

- NSGlobalBlock 全局
- NSMallocBlock 堆
- NSStackBock 栈

-----

- 定义一个block如下:

```objective-c
void (^block)(void) = ^{
    NSLog(@"Hello, this is a block.");
};

NSLog(@"%@", block);

block();
```

通过输出可以看出, 之前定义的block是NSGlobalBlock

![NSGlobalBlock输出](http://of66hypkg.bkt.clouddn.com/iOS-block%20NSGlobalBlock%E8%BE%93%E5%87%BA.png)

- 定义一个外部变量a, 在block中引用它

```objective-c
NSInteger a = 1;
    
void (^block)(void) = ^{
    NSLog(@"a == %ld", a);
    NSLog(@"Hello, this is a block.");
};

NSLog(@"%@", block);
```

通过输出可以看出, 这时候, 之前 定义的block已经是NSMallocBlock

![](http://of66hypkg.bkt.clouddn.com/iOS-block%20NSMallocBlock%E8%BE%93%E5%87%BA.png)

- 写一个如下的输出语句

```objective-c
NSInteger a = 1;

NSLog(@"This is %@", ^{
    NSLog(@"a == %ld", a);
});
```

通过输出可以看出, 这个block是NSStackBlock

![](http://of66hypkg.bkt.clouddn.com/iOS-block%20NSStackBlock%E8%BE%93%E5%87%BA.png)

**总结上边的内容, 我们发现**

> 1. 只定义一个block, 不进行赋值操作, 也不引用外部变量的时候, 这时候, 他就是一个NSStackBlock
> 2. 当进行了赋值操作的时候, 只进行赋值, 而不引用外部变量的时候, 为NSGlobalBlock
> 3. 一旦引用外部变量, 就会成为NSMallocBlock



#### 重点看block引用外部变量

写下边一段代码

```objective-c
__block int a = 1;

NSLog(@"前%p",&a);

void (^block)(void) = ^{
    NSLog(@"中%p",&a);
    a += 10;
    NSLog(@"%d",a);
};

NSLog(@"后%p",&a);

block();
```

输出结果为:

![](http://of66hypkg.bkt.clouddn.com/iOS-block%20%E5%BC%95%E7%94%A8%E5%A4%96%E9%83%A8%E5%8F%98%E9%87%8F.png)

我们可以看出, 当block内部引用了变量`a`以后, 变量`a`的地址发生了变化, 我们猜想, `__block`操作将变量从栈空间转移到了堆空间. 下边我们来验证猜想.



#### 验证猜想

- 创建一个叫`block.c`的文件, 写一段`c语言`diamante如下

```c
#include "stdio.h"

int main() {
    void (^block)(void) = ^{
        printf("hello lynn\n");
    };
    block();
    return 0;
};
```

- 终端内容使用`gcc block.`命令编译, 生成一个叫做`a.out`的可执行文件
- 接下来执行`a.out`文件
- 输出 如下

![](http://of66hypkg.bkt.clouddn.com/iOS-Block%20a.out%E8%BE%93%E5%87%BA.png)

这个不是中点, 接下来, 我们执行`clang -rewrite-objc block.c`, 生成一个`block.cpp`文件, 这个才是我们想要看的东西

-  在其中找到`main`函数的定义:

```cpp
int main() {
    void (*block)(void) = ((void (*)())&__main_block_impl_0((void *)__main_block_func_0, &__main_block_desc_0_DATA));
    ((void (*)(__block_impl *))((__block_impl *)block)->FuncPtr)((__block_impl *)block);
    return 0;
};
```

`__main_block_impl_0`的实现

```cpp
struct __main_block_impl_0 {
  struct __block_impl impl;
  struct __main_block_desc_0* Desc;
  __main_block_impl_0(void *fp, struct __main_block_desc_0 *desc, int flags=0) {
    impl.isa = &_NSConcreteStackBlock;
    impl.Flags = flags;
    impl.FuncPtr = fp;
    Desc = desc;
  }
};
```

通过这段代码`impl.isa = &_NSConcreteStackBlock;`我们可以清楚的知道, block其实是一个对象

- 接下来我们修改之前的`block.c`文件, 引入变量`a`

```c
#include "stdio.h"

int main() {
    int a = 1;
    void (^block)(void) = ^{
        printf("%d", a);
        printf("hello lynn\n");
    };
    block();
    return 0;
};
```

查看`__main_block_impl_0`实现

```cpp
struct __main_block_impl_0 {
  struct __block_impl impl;
  struct __main_block_desc_0* Desc;
  int a;
  __main_block_impl_0(void *fp, struct __main_block_desc_0 *desc, int _a, int flags=0) : a(_a) {
    impl.isa = &_NSConcreteStackBlock;
    impl.Flags = flags;
    impl.FuncPtr = fp;
    Desc = desc;
  }
};
```

发现引入了变量`a`, 再查看`__main_block_func_0`实现

```cpp
static void __main_block_func_0(struct __main_block_impl_0 *__cself) {
  int a = __cself->a; // bound by copy

        printf("%d", a);
        printf("hello lynn\n");
    }
```

这里取出了变量`a`的值, 赋值给了一个新定义的变量`a`

- 再修改之前的`block.c`文件, 重新生成`block.cpp`文件

```c
#include "stdio.h"

int main() {
    __block int a = 1;
    void (^block)(void) = ^{
        printf("%d", a);
        printf("hello lynn\n");
    };
    block();
    return 0;
};
```

查看`block.cpp`文件

```cpp
int main() {
    __attribute__((__blocks__(byref))) __Block_byref_a_0 a = {(void*)0,(__Block_byref_a_0 *)&a, 0, sizeof(__Block_byref_a_0), 1};
    void (*block)(void) = ((void (*)())&__main_block_impl_0((void *)__main_block_func_0, &__main_block_desc_0_DATA, (__Block_byref_a_0 *)&a, 570425344));
    ((void (*)(__block_impl *))((__block_impl *)block)->FuncPtr)((__block_impl *)block);
    return 0;
};
```

我们发现对变量`a`从栈拷贝到了堆, 跟我们的猜想一样



#### 由以上验证, 得到的结论是`__block`操作, 将变量从栈拷贝到了堆

