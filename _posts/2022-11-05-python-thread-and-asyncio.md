---
layout: post
title: "google cloud python sdk 学习使用记录"
subtitle: '使用python调用google api'
author: "chaoxiaodi"
header-style: text
tags:
  - python
  - thread
  - asyncio
  - 并发/并行
---

### 前言

在日常使用python开发的过程中

经常会用到一些多进程或多线程去并发处理问题

涉及到并发的问题，同时伴随的就是线程安全问题

除了多线程来提高一些io耗时类的性能

python同样支持异步(协程/并行)的方式

下面就用经典的多线程的问题来进行记录

### 多线程

#### 经典问题

一个线程对全局变量进行1千万次加1操作

然后一个线程对全局变量进行1千万次减1操作

按照预期，应该最后结果为0

但最后实际的执行结果却大大出乎意料

甚至执行多次会出现不同结果

    import threading

    num = 0 

    def add():
        global num
        for i in range(10000000):
            num += 1

    def sub():
        global num
        for i in range(10000000):
            num -= 1


    if __name__ == "__main__":
        thread01 = threading.Thread(target=add)
        thread02 = threading.Thread(target=sub)

        thread01.start()
        thread02.start()

        thread01.join()
        thread02.join()

        print("num : %s" % num)

#### 线程锁

为了解决上面的经典问题

python 提供了保证线程安全的线程锁

[python thread lock docs](https://docs.python.org/3/library/threading.html#lock-objects)

通过实例化一个线程锁

并通过加锁/解锁操作来保证线程安全

加锁后的代码示例如下

    import threading

    num = 0 

    def add():
        # 加锁
        # 在没有解锁之前只有此线程可以操作num变量
        lock.acquire()
        global num
        for i in range(10000000):
            num += 1
        # 解锁
        lock.release()

    def sub():
        lock.acquire()
        global num
        for i in range(10000000):
            num -= 1
        lock.release()


    if __name__ == "__main__":
        lock = threading.Lock()
        thread01 = threading.Thread(target=add)
        thread02 = threading.Thread(target=sub)

        thread01.start()
        thread02.start()

        thread01.join()
        thread02.join()

        print("num result : %s" % num)

这只是一个简单的锁使用示例

在python docs中提供了多种锁 可以根据需要进行使用

对于加锁解锁的操作必须是成对出现的

如果连续两次加锁，那就会造成死锁问题

为了方便处理加锁并及时解锁

python也提供了with关键字进行处理

with关键字 在好多地方都有应用

比如

with open() 打开文件 可以在打开文件操作完成后自动close

with app.app_context() 可以在flask中使用上下文

在线程锁中的应用可以看下代码示例

    import threading

    num = 0 

    def add():
        with lock:
            global num
            for i in range(10000000):
                num += 1

    def sub():
        with lock:
            global num
            for i in range(10000000):
                num -= 1


    if __name__ == "__main__":
        lock = threading.Lock()
        thread01 = threading.Thread(target=add)
        thread02 = threading.Thread(target=sub)

        thread01.start()
        thread02.start()

        thread01.join()
        thread02.join()

        print("num result : %s" % num)


### asyncio

当使用了上面简单的线程锁后

代码就又变成了串行执行的方式

由于涉及到线程切换可能比直接串行执行还要更加耗时

(当然了，这个代码中人感受不到耗时)

python 3.4之后 引入了 asyncio 模块

Async IO 与编程语言无关，算是一种模型(范式) 好多语言都有对应的实现

asyncio 在3.5中引入了async/await关键字来使用

下面使用asyncio 来解决下上面多线程并发带来的问题

    import asyncio

    num = 0

    async def add():
        global num
        for i in range(10000000):
            num += 1
        await asyncio.sleep(0)

    async def sub():
        global num
        for i in range(10000000):
            num -= 1
        await asyncio.sleep(0)

    async def main():
        task_add = add()
        task_sub = sub()
        await asyncio.gather(task_add, task_sub)

    if __name__ == '__main__':
        asyncio.run(main())
        print(num)

这样代码可以得到预期的结果 num=0

为了测试这个代码是不是真的并行  通过output以及增加耗时操作来测试

    import asyncio

    num = 0

    async def add():
        global num
        print('add start', num)
        for i in range(10000000):
            num += 1
            await asyncio.sleep(0)
        print('add end', num)
        await asyncio.sleep(4)
        print('add end after', num)

    async def sub():
        global num
        print('sub start' ,num)
        for i in range(10000000):
            num -= 1
        print('sub end' ,num)
        await asyncio.sleep(10)
        print('sub end after' ,num)

    async def main():
        task_add = add()
        task_sub = sub()
        await asyncio.gather(task_add, task_sub)

    if __name__ == '__main__':
        asyncio.run(main())
        print('num', num)

    time python test-asyncio.py
    # output

    start add 0
    start sub 1
    end sub  -99999
    end add 0
    end add after 0
    end sub after 0
    num 0
    python test-asyncio.py  1.32s user 0.35s system 16% cpu 10.292 total

通过结果可以清晰的看出

代码先是执行了add的操作

当完成第一次循环后 遇到await关键字

告诉其他协程，我要处理一会，你们可以开始工作了

然后就是开始执行了sub

直到遇到await关键字

又回到add中继续执行

注意这里是继续执行，而不是重新开始

由于add sleep的时间短 当执行完add后

又继续执行sub

看下最后的耗时

如果按照文字描述，好像是先执行了add

再执行sub

但实际情况是计算可以认为是同时在进行



### 后记

本文简单对多线程以及asyncio进行了学习记录

本人也是刚刚学习asyncio

如有理解不当之处

可以指出一起交流

### 参考

[用一万字从0到1讲解Python中的Async IO](https://www.toutiao.com/article/7156024774551306784)
[pythonav资源分享](https://pythonav.com/wiki/detail/6/91/)
[python asyncio](https://docs.python.org/3.9/library/asyncio.html)


Q：594934249

---我是超小弟·一名不务专业的秃头运维---

博客：[blog.chaoxiaodi.tech](https://blog.chaoxiaodi.tech)

github：[github:chaoxiaodi](https://github.com/chaoxiaodi)

微信公众号：老骥不伏枥只是近黄昏

![](/img/erweima.jpg)
