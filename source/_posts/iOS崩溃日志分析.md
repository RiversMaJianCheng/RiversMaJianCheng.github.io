---
title: iOS异常捕获
date: 2018-05-03 10:41:35
categories:
- iOS
tags:
---
## 崩溃捕获
在iOS程序的崩溃中，主要有两种异常引起的。一个是Mach异常，一个是Object-C异常（NSException）。
* Mach是一个XNU的微内核核心，Mach异常是指最底层的内核级异常，被定义在<mach/exception_types.h>下。每个thread，task，host都有一个异常端口数组，Mach的部分API暴露给了用户态，用户态的开发者可以直接通过Mach API设置thread，task，host的异常端口来捕获Mach异常，抓取Crash事件。

所有的Mach异常都在host层被ux_exception转换为相应的Unix信号，并通过threadsignal将信号投递到出错的线程。我们可以通过注册signalHandler来捕获信号：
```
    signal(SIGSEGV, signalHandler);
```
signal信号的类型：
```
    SIGABRT–程序中止命令中止信号
    SIGALRM–程序超时信号
    SIGFPE–程序浮点异常信号
    SIGILL–程序非法指令信号
    SIGHUP–程序终端中止信号
    SIGINT–程序键盘中断信号
    SIGKILL–程序结束接收中止信号
    SIGTERM–程序kill中止信号
    SIGSTOP–程序键盘中止信号
    SIGSEGV–程序无效内存中止信号
    SIGBUS–程序内存字节未对齐中止信号
    SIGPIPE–程序Socket发送失败中止信号
```
* 应用级异常NSException，某个NSException导致程序Crash，只要我们拿到这个NSException，获取它的reason，name，callStackSymbols信息才能确定出问题的程序位置。方法很简单，通过注册NSUncaughtExceptionHandler捕获异常信息：

```
    // 将下面C函数的函数地址当做参数
    NSSetUncaughtExceptionHandler(&UncaughtExceptionHandler);
```
```
// 设置一个C函数，用来接收崩溃信息
void UncaughtExceptionHandler(NSException *exception)
{ 
    // 可以通过exception对象获取一些崩溃信息，我们就是通过这些崩溃信息来进行解析的，例如下面的symbols数组就是我们的崩溃堆栈。
    NSArray *returnAddresses = [exception callStackReturnAddresses];
    NSArray *symbols = [exception callStackSymbols];
    NSString *reason = [exception reason];
    NSString *name = [exception name];
}
```
简单看一个数组越界的崩溃信息：
![](https://ws3.sinaimg.cn/large/006tKfTcly1fqy3gvgu3ej31c6124kas.jpg)

## 常见的Exception Type 和 Exception Code
### Exception Type 
* EXC_BAD_ACCESS
此类型的exception是我们最长碰到的Crash，通常用于访问了不该访问的内存导致。一般EXC_BAD_ACCESS后面的“（）”还会带有补充信息。
> SIGSEGV:通常由于重复释放对象导致，这种类型在ARC以后比较少见了
> SIGABRT:收到Abort信号推出，通常Foundation库中的容器为了保护状态正常会做一些检测，例如插入nil到数组中会出现此错误。
> SEGV:代码无效内存地址，比如空指针，未初始化指针，栈溢出等
> SIGBUS:总线错误，与SIGSEGV不同的是，SIGSEGV访问的是无效地址，而此访问的是有效地址，但是总线访问异常（如地址对齐问题）
> SIGILL:尝试执行非法的指令，可能不被识别或者没有权限

* EXC_BAD_INSTRUCTION
此类异常通常由于线程执行非法指令导致
* EXC_ARITHMETIC
除零错误会抛出此类异常

### Exception Code
> 0x8badf00d 读作“ate bad food”，程序启动或者恢复时间过长被watch dog终止。
> 0xdead10cc 读作“dead lock” 表示应用因为在后台运行时占用系统资源，如通讯录数据库不释放而被终止。
> 0xbad22222 该编码表示 VoIP 应用因为过于频繁重启而被终止。
> 0xdeadfa11 读做 “dead fall”! 该代码表示应用是被用户强制退出的。根据苹果文档, 强制退出发生在用户长按开关按钮直到出现 “滑动来关机”, 然后长按 Home按钮。强制退出将产生 包含0xdeadfa11 异常编码的崩溃日志, 因为大多数是强制退出是因为应用阻塞了界面。
> 0xbaaaaaad  ⽤用户按住Home键和⾳音量键,获取当前内存状态,不代表崩溃
> 0xc00010ff 读作“cool off” 因为太烫了被干掉

