*基本功：如何从hello world构建一个web服务器*

# 什么是Web服务器

Web 服务器是一种软件（服务器程序）或者硬件设备（云服务器），用于存储、处理和传送网页和其他 web 内容给客户端。
通常使用 HTTP 协议（通常到Web服务器几乎和http服务器混淆，即使Web服务器可以使用其他的协议，如FTP文件服务器，SMTP邮件服务器等）来与客户端进行通信。
使用HTTP协议时，接收客户端的请求并发送相应的数据或错误码。

![](https://raw.githubusercontent.com/Huixxi/Algorithm-with-Cplusplus/master/AlgorithmImages/web-server.svg "https://developer.mozilla.org/en-US/docs/Learn/Common_questions/What_is_a_web_server")
## 通信过程

通信，有两个方向：客户端向服务端发请求，服务端向客户端发回应。

## 服务器对象

```C++
class WebServer{
public:
	WebServer();
	~WebServer();
	void init(int port , string user, string passWord, string databaseName, int log_write , int close_log, int opt_linger, int trigmode, int sql_num,int thread_num,  int actor_model);
}
```
这是服务器对象的设计，观察其初始化参数：
1. port 服务器监听端口
2. user 访问mysql的账户
3. passWord 访问mysql的密码
4. databaseName 这是网站注册系统的账号和密码的 数据库[[#^fa8a09]] 名称
5. log_write 日志写 [[#部件——日志]] 
6. close_log 日志关闭 [[#部件——日志]]
# 构建步骤

这是接下来要讨论的主题：服务器结构，过程和方法的讨论结果——按此步骤，即可完成

## 接收http请求

使用socket接收报文。
接收到的报文，就是一串字符串。以http形式解析它。

# 部件——日志

从Webserver对象的初始化开始追踪：
```c++
void WebServer::init(int log_write, int close_log...)
{
	m_log_write = log_write;
	m_close_log = close_log;
}
/*传入的实参在config中定义：
Config::Config(){
	//端口号,默认9006
	PORT = 9006;
	//日志写入方式，默认同步
	LOGWrite = 0;
	//关闭日志,默认不关闭
	close_log = 0;
}*/
```
日志模块是服务器的基础模块。
在服务器运行的过程中，会追踪有哪些客户对该台服务器发起了请求。
对于概率性error事件，可以在重复测试时通过日志来查询错误复现时候的情况。记录error或者crash时的信息（时间、关键变量的值、出错的文件及行号、线程等）
简言之，日志是跟踪和回忆某个时刻或者时间段内的程序行为进而定位问题的一种重要手段。

日志有两种写入方式：同步和异步
同步：应用程序自身在事件发生时输出到磁盘，阻塞代码，较低响应性能，代码简单。注重于日志完整性和顺序性，用于关键操作记录或错误记录等
异步：程序将事件写到一块内存缓存，开一个额外的线程进行磁盘写入。不阻塞，高响应，复杂。

```c++
void WebServer::log_write()
{
	if (0 == m_close_log)
	{
//初始化日志
		if (1 == m_log_write)
			Log::get_instance()->init("./ServerLog", m_close_log, 2000, 800000, 800);
		else
			Log::get_instance()->init("./ServerLog", m_close_log, 2000, 800000, 0);
	}
}
```

# 杂项

```
mysql> show tables;
+-----------------------+
| Tables_in_webServerDB |
+-----------------------+
| user                  |
+-----------------------+
1 row in set (0.00 sec)

mysql> select * from user
    -> ;
+------------+---------------+
| username   | passwd        |
+------------+---------------+
| name       | passwd        |
| ninemerlin | 8888888888888 |
+------------+---------------+
2 rows in set (0.00 sec)

```

^fa8a09
