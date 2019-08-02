---
title: c++ 回调机制
categories: cpp
date: 2019-08-01 15:15:19
tags: [cpp, callback]
---

我们会经常碰到需要使用回调函数的场合，比如：异步socket、定时器、windows消息处理等等。
这里将列出几种回调函数的实现机制，分析各自的优劣以供选择。
<!--more-->

将 *静态函数* 或 *静态成员函数* 作为回调函数的实现比较简单，而且除了像 std::sort 这种地方，一般很少会用到，这里就不多做说明了。下面列出的都是将 *成员函数* 作为回调函数的实现。

----------
## 接口类

```cpp
    class CallbackInterface
    {
    public:
        virtual void onCallback() = 0;
        ...
    };

    class Callee : public CallbackInterface
    {
    public:
        virtual void onCallback() { ... }
        ...
    };

    class Caller
    {
    public:
        void register(CallbackInterface* callback) { ... }
        void update()
        {
            ...
            callback->onCallback();
            ...
        }
        ...
    };
    
    // main
    caller.register(&callee);
```

这种方式适用于 Callee 和 CallbackInterface 自然符合继承语义的情况：Callee 是一种 CallbackInterface，而 Caller 作为管理者，面对的是一堆的 CallbackInterface，至于具体是哪个 Callee 在做事，又是怎么做的， Caller 并不需要知道。
如果 Callee 和 CallbackInterface 并不自然符合继承语义，最好不要使用这种方式，不然可能会碰到下列限制：
* 需要定义 N 个 CallbackInterface，每个 CallbackInterface 需要定义具体接口函数。
* 一个 Callee 可能需要继承多个 CallbackInterface，且各个 CallbackInterface 的接口函数不能同名。
* 不支持一个 Callee 对应多个 Caller 的情况。

----------
## 公共基类

```cpp
    class Object {...}
    typedef void(Object::*CALLBACK)();    

    class Callee : public Object
    {
    public:
        void onCallback() { ... }
        ...
    }

    class Caller
    {
    public:
        void register(Object* obj, CALLBACK callback) { ... }
        void update()
        {
            ...
            obj->*callback();
            ...
        }
        ...
    }
    
    // main
    caller.register(&callee, (CALLBACK)(&Callee::onCallback));
```

cocos-2dx 3.0 之前的版本用的就是这种方式。
尽管它已经可以满足大多数需要回调函数的场合，但也还是有一些显而易见的缺点：
* 所有的 Callee 需要继承自一个公共基类 Object。
* register 时需要做强制类型转换，这使得编译期无法对回调函数本身的参数类型和数量进行检查，如果不匹配将导致运行时错误。
* 无法在 register 时传递不定参数给回调函数。
 
----------
## std::function

```cpp
    class Callee
    {
    public:
        void onCallback() { ... }
        ...
    }

    class Caller
    {
    public:
        void register(const std::function<void()>& callback) { ... }
        void update()
        {
            ...
            callback();
            ...
        }
        ...
    }
    
    // main
    caller.register(std::bind(&Callee::onCallback, callee));
```
    
对比之前的实现，这种方式几乎解决了所有的缺点。
* Callee 不需要继承公共基类或者回调接口类。
* 多个 Callee 可以绑定于多个 Caller。
* register 时不需要进行强制类型转换，也能在编译期检查回调函数类型。
* register 时还能传递不定参数给回调函数。

----------
## 模板

```cpp
    class Callee
    {
    public:
        void onCallback() { ... }
        ...
    }
    
    class Caller
    {
    public:
        void register(const CBFunctor0 & callback) { ... }
        void update()
        {
            ...
            callback();
            ...
        }
        ...
    }
    
    // main
    caller.register(makeFunctor((CBFunctor0*)0,callee,&Callee::onCallback));
```

需要包含一个回调函数库：<http://www.tedfelix.com/software/callback.h>    
具体的实现原理和过程可以查看： <http://www.tutok.sk/fastgl/callback.html>
从使用者角度看，除了不能传递不定参数给回调函数，它跟 *std::function 方式* 几乎一样。
当所用编译器不支持 c++11 特性时，可以考虑用这种方式。

----------
肯定还有其它实现回调机制的方式，碰到的时候再加进来分析。
目前看来 *std::function 方式* 是一种比较完美的方案，但在实际应用中使用它也还是会碰到一些问题。
而且使用成员函数作为回调函数，还需要考虑当回调函数将被调用时，如何判断绑定的对象是否已经被销毁。
等等这些问题，后续将会有专门的篇幅进行探讨。