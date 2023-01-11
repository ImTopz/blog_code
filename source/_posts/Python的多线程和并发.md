---
title: Python的多线程和并发框架
date: 2023-1-11 17:00:00
tags: 学习
---

# Python多线程下的GIL

1、**GIL是什么？**GIL的全称是Global Interpreter Lock(全局解释器锁)，来源是python设计之初的考虑，为了数据安全所做的决定。

2、每个CPU在同一时间只能执行一个线程（在单核CPU下的多线程其实都只是并发，不是并行，并发和并行从宏观上来讲都是同时处理多路请求的概念。但并发和并行又有区别，并行是指两个或者多个事件在同一时刻发生；而并发是指两个或多个事件在同一时间间隔内发生。）

**在Python多线程下，每个线程的执行方式：**

1.获取GIL

2.执行代码直到sleep或者是python虚拟机将其挂起。

3.释放GIL

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/7d046bea-00df-44b9-9341-8e8da2255dd6/Untitled.png)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6c6b589e-5b7d-4296-bfe1-598d2f08a76b/Untitled.png)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ebfbe395-563b-4ff7-a77a-f0b9e2929a15/Untitled.png)

参考代码:

```python
# coding:utf-8
import threading, time

def my_counter():
    i = 0
    for _ in range(100000000):
        i = i+1
    return True

def main1():
    thread_ary = {}
    start_time = time.time()
    for tid in range(2):
        t = threading.Thread(target=my_counter)
        t.start()
        t.join()  # 第一次循环的时候join方法引起主线程阻塞，但第二个线程并没有启动，所以两个线程是顺序执行的

    print("单线程顺序执行total_time: {}".format(time.time() - start_time))

def main2():
    thread_ary = {}
    start_time = time.time()
    for tid in range(2):
        t = threading.Thread(target=my_counter)
        t.start()
        thread_ary[tid] = t

    for i in range(2):
        thread_ary[i].join()  # 两个线程均已启动，所以两个线程是并发的

    print("并发执行total_time: {}".format(time.time() - start_time))

if __name__ == "__main__":
    main1()
    main2()
```

### GIL并不意味着线程安全

   有GIL并不意味着python一定是线程安全的，那什么时候安全，什么时候不安全，我们必须搞清楚。之前我们已经说过，一个线程有两种情况下会释放全局解释器锁，一种情况是在该线程进入IO操作之前，会主动释放GIL，另一种情况是解释器不间断运行了1000字节码（Py2）或运行15毫秒（Py3）后，该线程也会放弃GIL。既然一个线程可能随时会失去GIL，那么这就一定会涉及到线程安全的问题。GIL虽然从设计的出发点就是考虑到线程安全，但这种线程安全是粗粒度的线程安全，即不需要程序员自己对线程进行加锁处理（同理，所谓细粒度就是指程序员需要自行加、解锁来保证线程安全，典型代表是 Java , 而 CPthon 中是粗粒度的锁，即语言层面本身维护着一个全局的锁机制,用来保证线程安全）。那么什么时候需要加锁，什么时候不需要加锁，这个需要具体情况具体分析。下面我们就来针对每种可能的情况进行分析和总结。

  首先来看第一种线程释放GIL的情况。假设现在线程A因为进入IO操作而主动释放了GIL，那么在这种情况下，由于线程A的IO操作等待时间不确定，那么等待的线程B一定会得到GIL锁，这种比较“礼貌的”情况我们一般称为“协同式多任务处理”，相当于大家按照协商好的规则来，线程是安全的，不需要额外加锁。

  接下来，我们来看另外一种情况，即线程A是因为解释器不间断执行了1000字节码的指令或不间断运行了15毫秒而放弃了GIL，那么此时实际上线程A和线程B将同时竞争GIL锁。在同时竞争的情况下，实际上谁会竞争成功是不确定的一个结果，所以一般被称为“抢占式多任务处理”，这种情况下当然就看谁抢得厉害了。当然，在python3上由于对GIL做了优化，并且会动态调整线程的优先级，所以线程B的优先级会比较高，但仍然无法肯定线程B就一定会拿到GIL。那么在这种情况下，线程可能就会出现不安全的状态。

所以，总结一下，如果多线程的操作中不是IO密集型，并且计算操作不是原子级的操作时，那么我们需要考虑线程安全问题，否则都不需要考虑线程安全。当然，为了避免担心哪个操作是原子的，我们可以遵循一个简单的原则：始终围绕共享可变状态的读取和写入加锁。毕竟，在 Python 中获取一个 **threading.Lock** 也就是一行代码的事**。**

### 同时，需要了解一下操作系统的并发跟并行两个概念

并行： 在同一时刻有多条指令在多个处理器上同时执行，通俗点就是CPU数>=任务数的情况；并发：在同一时刻只能有一条指令执行，但是多个任务被快速轮换执行，使得宏观上让人感觉到有多个任务同时执行的效果。通俗点来说就是CPU数<任务数的情况。

## python多线程的适用场景：

1、CPU密集型代码(各种循环处理、计数等等)，在这种情况下，ticks计数很快就会达到阈值，然后触发GIL的释放与再竞争（多个线程来回切换当然是需要消耗资源的），所以**python下的多线程对CPU密集型代码并不友好。**

2、IO密集型代码(文件处理、网络爬虫等)，多线程能够有效提升效率(单线程下有IO操作会进行IO等待，造成不必要的时间浪费，而开启多线程能在线程A等待时，自动切换到线程B，可以不浪费CPU的资源，从而能提升程序执行效率)。**所以python的多线程对IO密集型代码比较友好。**

**change**:而在python3.x中，GIL不使用ticks计数，改为使用计时器（执行时间达到阈值后，当前线程释放GIL），这样对CPU密集型程序更加友好，**但依然没有解决GIL导致的同一时间只能执行一个线程的问题，所以效率依然不尽如人意。**

**多核多线程比单核多线程更差，原因是单核下多线程，每次释放GIL，唤醒的那个线程都能获取到GIL锁，所以能够无缝执行，但多核下，CPU0释放GIL后，其他CPU上的线程都会进行竞争，但GIL可能会马上又被CPU0拿到，导致其他几个CPU上被唤醒后的线程会醒着等待到切换时间后又进入待调度状态，这样会造成线程颠簸(thrashing)，导致效率更低**

多线程类似于同时执行多个不同程序，多线程运行有如下优点：

- 使用线程可以把占据长时间的程序中的任务放到后台去处理。
- 用户界面可以更加吸引人，这样比如用户点击了一个按钮去触发某些事件的处理，可以弹出一个进度条来显示处理的进度
- 程序的运行速度可能加快
- 在一些等待的任务实现上如用户输入、文件读写和网络收发数据等，线程就比较有用了。在这种情况下我们可以释放一些珍贵的资源如内存占用等等。

线程在执行过程中与进程还是有区别的。每个独立的进程有一个程序运行的入口、顺序执行序列和程序的出口。但是线程不能够独立执行，必须依存在应用程序中，由应用程序提供多个线程执行控制。

每个线程都有他自己的一组CPU寄存器，称为线程的上下文，该上下文反映了线程上次运行该线程的CPU寄存器的状态。

指令指针和堆栈指针寄存器是线程上下文中两个最重要的寄存器，线程总是在进程得到上下文中运行的，这些地址都用于标志拥有线程的进程地址空间中的内存。

- 线程可以被抢占（中断）。
- 在其他线程正在运行时，线程可以暂时搁置（也称为睡眠） -- 这就是线程的退让。

## Python多线程编程实战：

网络请求接口多线程代码例子

```sql
SPLIT_COUNT = 100
times = math.ceil(len(self.__fileName) / SPLIT_COUNT)  # 线程数量
count = 0
print("----------从文件中提取数据开始-------------")
for items in range(times):  # 划分线程
    _list = self.__fileName[count: count + SPLIT_COUNT]
    thread = Thread(target=self.__fileDeal, args=(_list,))
    thread_list.append(thread)
    thread.start()
    count += SPLIT_COUNT
for _item in thread_list:
    _item.join()  # 阻塞（等待所有线程结束后）
```

这样可以在__fileDeal函数中具体实现一个对网络接口的请求，可以极大的加快接口请求速度。

```python
def __fileDeal(self, tmpList):
        """
        将所有csv的数据提取到list里面
        :param tmpList: 是传入过来的仓库名称
        :return: self.__dataList
        """
        __tmpList = []  # 用来存储每个csv的临时数据
        __updateTime = datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")
        for item in tqdm(tmpList):
            try:
                df = pandas.read_csv(item, encoding='gb2312')
                df = df.fillna(
                    'null',  
                    inplace=False  
                )
                for row in df.values:
                    __tmpDict = {
                        "platform": "",
                        "warehouse": "",
                        "outer_id": "",
                        "number": 0,
                        "price": 0.0,
                        "market_price": 0.0,
                        "article": "",
                        "season": "",
                        "brand": "",
                        "category": "",
                        "gender": "",
                        "discount": "",
                        "update_time": __updateTime

                    }
                    if ((float(row[5]) < 1.2) or (float(row[5]) > 50)):
                        continue
                    __tmpDict['warehouse'] = item.split('.')[0]
                    __tmpDict['article'] = re.sub('[="]', '', row[0])
                    __tmpDict['size'] = re.sub('[="]', '', row[1])
                    __tmpDict['number'] = int(row[3])
                    __tmpDict['price'] = float(row[4]) * 0.1 * float(row[5])
                    __tmpDict['discount'] = float(row[5])
                    __tmpDict['market_price'] = float(row[4])  
                    __tmpDict['gender'] = row[6]
                    __tmpDict['season'] = row[7]
                    __tmpDict['brand'] = row[8]
                    __tmpDict['category'] = row[9]
                    __tmpList.append(__tmpDict)

            except:
                pass

        self.__dataList.extend(__tmpList)
        __tmpList = []
```

这里我编写了一个多线程解压提取数据的demo,相对于单线程也是极大的提高了文件提取的速度，多线程编程相对于单线程编程带来了一定的压力，我们在编程时候也要考虑规避内存溢出、异步编程等一系列的问题。

**在这里推荐大家了解一下python的Celery,但是本人用的是国产开源框架funboost，也是推荐大家学习一下，因为celery本身具有较多的缺点**