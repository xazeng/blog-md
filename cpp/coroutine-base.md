---
title: c++ 协程基础
categories: cpp
date: 2020-09-28 22:15:19
tags: [cpp, coroutine]
---

从 c++20 开始，c++ 终于有了协程支持，鉴于眼下 msvc 和 gcc 都已经支持了这一特性，我也准备在实际开发中用起来。本文仅针对协程的基础进行阐述，并未提供上层封装的思路。
<!-- more -->

其实 c++20 之前，网上就有很多 c++ 的协程库。比如腾讯开源的 [libco](https://github.com/Tencent/libco) 虽然只支持 linux 平台，但是能支撑微信后台大规模使用，显然性能上肯定没问题。

虽然这些库也能用，但跟编译器层面上支持的协程相比，性能上还是略逊一筹，而且在使用上多少也都会有一些限制。比如 libco，需要运行一个永久阻塞的函数，这种独占性导致一个系统线程内不能再有其他调度等等。

---

## 协程执行过程

### 启动

* 通过 `operator new` 创建用来保存协程状态的对象
* 拷贝协程函数参数到协程状态中
* 创建 `promise` 对象
* 调用 `promise.get_return_object` 创建 resumable 对象，在协程第一次挂起时作为返回值回给上层调用
* 调用 `co_await promise.initial_suspend()`，这里决定了协程是否一创建就运行
* 然后就是执行协程函数中的代码

### 挂起

* `auto v = co_await expr`  

  * 先获取 awaitable
    * 如果 `expr` 来自 `initial_suspend` 或者 `final_suspend`，awaitable 就是 `expr` 本身
    * 否则就调用 `promise.await_transform`， 如果 promise 有这个方法的话
    * 否则 awaitable 就是 `expr` 本身  
  
  * 再获取 awaiter
    * 如果有匹配的 `co_await` 运算符重载函数，那么这个函数的返回就是 awaiter
    * 否则 awaitable 本身也是 awaiter 对象  

  * 调用 `awaiter.await_ready` ， 返回  `false` 则继续挂起

  * 协程挂起，保存状态，然后调用 `awaiter.await_suspend(handle)`  
    * 如果返回 `void` ， 控制权交出
    * 如果返回 `true` ， 控制权交出
    * 如果返回 `false` , 唤醒当前协程继续执行
    * 还可以返回其他协程的 handler， 然后唤醒它，这个机制有待考察

* `auto v = co_yield expr`  
  相当于 `co_await promise.yield_value(expr)`

### 唤醒  

`handler.resume()`

* 恢复协程状态，然后调用 `awaiter.await_resume`，其返回值会被作为 `co_await expr` 的返回值返回。

### 返回

`co_return exp`

* `exp` 是 `void` 或者没有返回值，则调用 `promise.return_void()`
* 有返回值，则调用 `promise.return_value(exp)`
* 用创建相反的顺序销毁相关对象
* 调用 `co_await promise.final_suspend()`，如果这里没有真的挂起，那么要注意 handler 对象在退出函数后会被自动销毁。

### 异常

* 捕获异常，然后调用 `promise.unhandled_exception()`
* 调用 `co_await promise.final_suspend()`，之后如果再被唤醒，会进入未定义的行为

---

## 代码示例

    struct suspend_always {
    	bool await_ready() noexcept {
            std::cout << __FUNCSIG__ << std::endl;
    		return false;
    	}

        // 挂起后，返回上层前
    	void await_suspend(std::experimental::coroutine_handle<>) noexcept {
            std::cout << __FUNCSIG__ << std::endl;
        }

        // resume 时，协程继续前
    	int await_resume() noexcept {
            std::cout << __FUNCSIG__ << std::endl;
            return 314; // 返回给 co_await
        }
    };

    struct suspend_never {
    	bool await_ready() noexcept {
            std::cout << __FUNCSIG__ << std::endl;
    		return true;
    	}

    	void await_suspend(std::experimental::coroutine_handle<>) noexcept {
            std::cout << __FUNCSIG__ << std::endl;
        }

    	void await_resume() noexcept {
            std::cout << __FUNCSIG__ << std::endl;
        }
    };

    struct resumable
    {
        struct promise_type
        {
            int _value;
            std::string _strValue;
            resumable get_return_object() { 
                std::cout << __FUNCSIG__ << std::endl;
                return { *this };
            }

            auto initial_suspend() {
                std::cout << __FUNCSIG__ << std::endl;
                return suspend_never{}; // 如果用 suspend_always，那么协程不会自动开始，需要被 resume
            }

            auto final_suspend() {
                std::cout << __FUNCSIG__ << std::endl;
                return suspend_always{}; // 如果用 never，那么协程函数结束后，resumable._co 其实已经无效了，promise_type 会在协程函数结束时直接析构掉
            }

            auto yield_value(const char* str) {
                std::cout << __FUNCSIG__ << std::endl;
                _strValue.assign(str);
                return suspend_always();
            }

            void return_value(int val) {
                std::cout << __FUNCSIG__ << std::endl;
                _value = val; 
            }
            // return_value 和 return_void  不能同时定义

            void unhandled_exception() {
                std::cout << __FUNCSIG__ << std::endl;
                std::terminate(); 
            }

            promise_type() {
                std::cout << __FUNCSIG__ << std::endl;
            }

            promise_type(int v) {
                std::cout << __FUNCSIG__ << std::endl;
                _value = v;
            }

            ~promise_type() {
                std::cout << __FUNCSIG__ << std::endl;
            }

        };

        void resume() {
    		std::cout << __FUNCSIG__ << std::endl;
            _co.resume(); 
    		std::cout << "after call resume" << std::endl;
        }
        int get() {
    		std::cout << __FUNCSIG__ << std::endl;
            return _co.promise()._value;
        }
        const std::string& getStr() {
    		std::cout << __FUNCSIG__ << std::endl;
            return _co.promise()._strValue;
        }

        using HDL = std::experimental::coroutine_handle<promise_type>;
        resumable() {
    		std::cout << __FUNCSIG__ << std::endl;
        }
        resumable(const resumable&) = delete;
        ~resumable() {
    		std::cout << __FUNCSIG__ << std::endl;
            if (_co) { _co.destroy(); }
        }

    private:
        resumable(promise_type& p) : _co(HDL::from_promise(p)) {
    		std::cout << __FUNCSIG__ << std::endl;
        }
        HDL _co;
    };

    resumable coroutine(int v)
    {
        std::cout << "before suspension" << std::endl;
        auto ret = co_await suspend_always{}; // ret 从 suspend_always.await_resume 返回
        std::cout << "resumed" << std::endl;
        co_yield "co_yield return"; // 值传给 promise_type.yield_value
        co_return v+1; // 值传给 promise_type.return_value
        //co_return;
    }

    void comain()
    {
        auto co = coroutine(42);

        co.resume();
        std::cout << co.getStr() << std::endl;

        co.resume();
        std::cout << co.get() << std::endl;

        return;
    }