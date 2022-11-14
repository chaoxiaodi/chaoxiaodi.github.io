---
layout: post
title: "python asyncio 学习/躺坑记录"
subtitle: '记录学习asyncio模块时遇到的问题'
author: "chaoxiaodi"
header-style: text
tags:
  - python
  - asyncio
  - async/await
  - 并行
  - 异步
---

### 前言

    asyncio is a library to write concurrent code using the async/await syntax.

    asyncio is used as a foundation for multiple Python asynchronous frameworks that provide high-performance network and web-servers, database connection libraries, distributed task queues, etc.

    asyncio is often a perfect fit for IO-bound and high-level structured network code.

    --摘自官网文档

最近在使用python时有很多需要并发的需求

之前一直在使用threadpool进行并发

这篇记录下学习使用asyncio的一个过程

由于接触协程比较晚

大部分文档已经是很久之前的了

#### 最简单的例子

并发执行三次相同的函数

输出结果为先输出三次 One 再输出三次 Two

    #!/usr/bin/env python3
    # countasync.py

    import asyncio

    async def count():
        print("One")
        await asyncio.sleep(1)
        print("Two")

    async def main():
        await asyncio.gather(count(), count(), count())

    if __name__ == "__main__":
        asyncio.run(main())

函数必须使用async 关键字进行定义

使用await 关键字告诉cpu可以执行其他任务了

有不少开源的大型项目在使用asyncio的时候

会使用 await asyncio.sleep(0)

来切换cpu运行任务

按照python 官网介绍

await 后面必须是可等待的

原文：

    We say that an object is an awaitable object if it can be used in an await expression. Many asyncio APIs are designed to accept awaitables.

    There are three main types of awaitable objects: coroutines, Tasks, and Futures.

coroutines 通过 async 关键字定义的函数

Tasks 通过asyncio.create_task(count()) 方法可以把coroutines封装成一个可等待的task

Futures 可以通过 asyncio的低级api进行定义，下面会提到

#### 经典的输出案例

通过嵌套两层循环

在一个协程中调用另一个

父协程会等待子协程执行完成

    #!/usr/bin/env python3
    # rand.py

    import asyncio
    import random

    # ANSI colors
    c = (
        "\033[0m",   # End of color
        "\033[36m",  # Cyan
        "\033[91m",  # Red
        "\033[35m",  # Magenta
    )

    async def makerandom(idx: int, threshold: int = 6) -> int:
        print(c[idx + 1] + f"Initiated makerandom({idx}).")
        i = random.randint(0, 10)
        while i <= threshold:
            print(c[idx + 1] + f"makerandom({idx}) == {i} too low; retrying.")
            await asyncio.sleep(idx + 1)
            i = random.randint(0, 10)
        print(c[idx + 1] + f"---> Finished: makerandom({idx}) == {i}" + c[0])
        return i
        
    async def main():
        res = await asyncio.gather(*(makerandom(i, 10 - i - 1) for i in range(3)))
        return res
        
    if __name__ == "__main__":
        random.seed(444)
        r1, r2, r3 = asyncio.run(main())
        print()
        print(f"r1: {r1}, r2: {r2}, r3: {r3}")

### 协程封装同步方法

如果开发中使用到的模块不支持协程

那么就可以用下面的方法进行封装

    import asyncio
    import requests
    async def download_image(url):
        # 发送网络请求，下载图片（遇到网络下载图片的IO请求，自动化切换到其他任务）
        print("开始下载:", url)
        loop = asyncio.get_event_loop()
        # requests模块默认不支持异步操作，所以就使用线程池来配合实现了。
        future = loop.run_in_executor(None, requests.get, url)
        response = await future
        print('下载完成')
        # 图片保存到本地文件
        file_name = url.rsplit('_')[-1]
        with open(file_name, mode='wb') as file_object:
            file_object.write(response.content)
    if __name__ == '__main__':
        url_list = [
            'https://www3.autoimg.cn/newsdfs/g26/M02/35/A9/120x90_0_autohomecar__ChsEe12AXQ6AOOH_AAFocMs8nzU621.jpg',
            'https://www2.autoimg.cn/newsdfs/g30/M01/3C/E2/120x90_0_autohomecar__ChcCSV2BBICAUntfAADjJFd6800429.jpg',
            'https://www3.autoimg.cn/newsdfs/g26/M0B/3C/65/120x90_0_autohomecar__ChcCP12BFCmAIO83AAGq7vK0sGY193.jpg'
        ]
        tasks = [download_image(url) for url in url_list]
        loop = asyncio.get_event_loop()
        loop.run_until_complete( asyncio.wait(tasks) )

上面代码直接使用pythonav中的代码

通过使用future的方法来实现异步使用requests库

这段代码是好使的

但有一个问题是 如果loop.run_in_executor() 里想给 requests.get 传递多个参数就会报错

经过查找资料

[Passing args, kwargs, to run_in_executor](https://stackoverflow.com/questions/53368203/passing-args-kwargs-to-run-in-executor)

找到这样一种 解决办法

在解决这个问题的时候同时也在官方文档进行查找

恰好，我使用的python版本的3.9以上的

asyncio 在python3.9引入了一个新的高级api去实现上面的操作

asyncio.to_thread()

    import asyncio
    import requests
    async def download_image(url):
        # 使用to_thread 来给函数传递多个参数
        response  = asyncio.to_thread(requests.get, url=url, timeout=10, params=params)
        # 或者
        kwargs = {'url': url, 'timeout': 3, 'params': params}
        response  = asyncio.to_thread(requests.get, **kwargs)
        print('下载完成')
        # 图片保存到本地文件
        file_name = url.rsplit('_')[-1]
        with open(file_name, mode='wb') as file_object:
            file_object.write(response.content)
    if __name__ == '__main__':
        url_list = [
            'https://www3.autoimg.cn/newsdfs/g26/M02/35/A9/120x90_0_autohomecar__ChsEe12AXQ6AOOH_AAFocMs8nzU621.jpg',
            'https://www2.autoimg.cn/newsdfs/g30/M01/3C/E2/120x90_0_autohomecar__ChcCSV2BBICAUntfAADjJFd6800429.jpg',
            'https://www3.autoimg.cn/newsdfs/g26/M0B/3C/65/120x90_0_autohomecar__ChcCP12BFCmAIO83AAGq7vK0sGY193.jpg'
        ]
        tasks = [download_image(url) for url in url_list]
        loop = asyncio.get_event_loop()
        loop.run_until_complete( asyncio.wait(tasks) )

通过使用asyncio.to_thread可以给requests.get传递多个需要的参数

    Asynchronously run function func in a separate thread.

    Any *args and **kwargs supplied for this function are directly passed to func. Also, the current contextvars.Context is propagated, allowing context variables from the event loop thread to be accessed in the separate thread.

    Return a coroutine that can be awaited to get the eventual result of func.

    This coroutine function is primarily intended to be used for executing IO-bound

    官网关于to_thread方法的解释

    简单理解就是to_thread可以吧*args **kwargs 直接传给要执行的func 同时可以允许函数访问当前上下文变量
    然后返回一个可等待结果的协程
    对应上面提到的 await 必须是一个可等待对象


### 浅读几段相关源码

官方文档中

在低级api events里提示

要尽量使用提供的高级api

如 aysncio.run()

    def run(main, *, debug=None):
        if events._get_running_loop() is not None:
            raise RuntimeError(
                "asyncio.run() cannot be called from a running event loop")

        if not coroutines.iscoroutine(main):
            raise ValueError("a coroutine was expected, got {!r}".format(main))

        loop = events.new_event_loop()
        try:
            events.set_event_loop(loop)
            if debug is not None:
                loop.set_debug(debug)
            return loop.run_until_complete(main)
        finally:
            try:
                _cancel_all_tasks(loop)
                loop.run_until_complete(loop.shutdown_asyncgens())
                loop.run_until_complete(loop.shutdown_default_executor())
            finally:
                events.set_event_loop(None)
                loop.close()

其实在run的函数里

先是检查了是否已经有协程在运行

或者理解为当前是否在协程中

如果 asyncio.run(func()) func中再次使用 asyncio.run 就会触发报错

如果想自己控制事件循环可以使用低级api

最简单实现

    async def loop_main(main):
    loop = asyncio.new_event_loop()
    asyncio.set_event_loop(loop)
    return loop.run_until_complete(main)

再看一下python3.9之后新增的 to_thread()

    async def to_thread(func, /, *args, **kwargs):
        """Asynchronously run function *func* in a separate thread.

        Any *args and **kwargs supplied for this function are directly passed
        to *func*. Also, the current :class:`contextvars.Context` is propogated,
        allowing context variables from the main thread to be accessed in the
        separate thread.

        Return a coroutine that can be awaited to get the eventual result of *func*.
        """
        loop = events.get_running_loop()
        ctx = contextvars.copy_context()
        func_call = functools.partial(ctx.run, func, *args, **kwargs)
        return await loop.run_in_executor(None, func_call)

对应上面使用 run_in_executor 传递多个参数报错查找资料时提到的解决办法



### 后记

在使用日常使用python进行开发时

有时经常会遇到一些报错

除了通过google查问题外

也要搭配官网进行排查

尤其是在版本差别比较大的情况下

网上有些文章以及答案在当时环境下可能确实是最好的答案

但新的版本官方可能提供了更好的解决办法

### 参考

[python asyncio docs](https://docs.python.org/3.9/library/asyncio.html)

[用一万字从0到1讲解Python中的Async IO](https://www.toutiao.com/article/7156024774551306784)

[pythonav](https://pythonav.com/wiki/detail/6/91/)

Q：594934249

---我是超小弟·一名不务专业的秃头运维---

博客：[blog.chaoxiaodi.tech](https://blog.chaoxiaodi.tech)

github：[github:chaoxiaodi](https://github.com/chaoxiaodi)

微信公众号：老骥不伏枥只是近黄昏

![](/img/erweima.jpg)
