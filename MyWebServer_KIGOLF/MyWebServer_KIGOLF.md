## 服务器框架
实现C/S（BS）的模型，同步I/O（次要），I/O复用（主要）

- [ ] 客户端预计TCP直连。
- [ ] 分布式或许可以兼容p2p
- [ ] 尝试信号驱动模型

- [ ] 多个部分缺少书的图片

### 服务器模型

分为C/S和p2p模型。
C/S为客户端/服务器模型，适合资源集中的场景，能保证数据提供的一致性和完整性。
	特例：BS：浏览器/服务器。更方便，更安全，复杂任务处理能力稍差。
p2p对等模型，淡化提供者和使用者的区别，可以看做是C/S的扩展。

### 编程框架
服务器有4个模块：
1. I/O模块：处理客户连接，读取网络数据
2. 逻辑单元：业务进程或线程
3. 网络存储单元：数据库，文件，缓存
4. 请求队列：前三个模块间的通信方式


## 有限状态机

### HTTP解析状态机


## I/O模块
- [ ] 本部分在tcp卷1上有详细描述，复习一下

同步I/O，I/O复用，信号，异步I/O。

## 逻辑单元
服务器处理三类事件：I/O事件，信号和定时事件

- [ ] 三种事件？

### 两种事件处理模式：Reactor，Proactor

#### Reactor
主线程只监听事件，并依靠I/O复用通知工作线程。I/O操作和处理逻辑在工作线程

#### Proactor
主线程监听事件，内核完成I/O，工作线程处理逻辑

#### 模拟Proactor
主线程监听事件，分析类型，插入队列，完成I/O。工作线程处理逻辑

### 两种并发模式：半同步/半异步，领导者/追随者

#### 半同步/半异步
同步则按代码顺序执行，异步则允许“中断”去执行其他代码
同步效率低，实时性差，但简单。
异步效率高，实时性高，但复杂，难以开发，也不适用于大量的并发。、

因而，同步线程用于处理客户逻辑，这无需中断。
异步线程监听端口，处理I/O，进行队列插入等。

#### 变体：半同步/半反应堆
异步线程只有监听，I/O被移至工作线程中，属于Reactor的事件处理模式。



