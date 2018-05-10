---
title: iOS异常捕获
date: 2018-05-03 10:41:35
categories:
- iOS
tags:
---
## 崩溃捕获
在iOS程序的崩溃中，主要有两种异常引起的。一个是Mach异常，一个是Object-C异常（NSException，OC层的异常）。

### Mach Exception
* Mach是一个XNU的微内核核心，Mach异常是指最底层的内核级异常，被定义在<mach/exception_types.h>下。每个thread，task，host都有一个异常端口数组，Mach的部分API暴露给了用户态，用户态的开发者可以直接通过Mach API设置thread，task，host的异常端口来捕获Mach异常，抓取Crash事件。

总结几个常见的Mach异常（Exception Type ， Exception Code）：
1. Exception Type
* EXC_BAD_ACCESS    (Bad Memory Access)
此类型的exception是我们最长碰到的Crash，通常用于访问了不该访问的内存导致。一般EXC_BAD_ACCESS后面的“（）”还会带有补充信息。
> SIGSEGV:通常由于重复释放对象导致，这种类型在ARC以后比较少见了
> SIGABRT:收到Abort信号推出，通常Foundation库中的容器为了保护状态正常会做一些检测，例如插入nil到数组中会出现此错误。
> SEGV:代码无效内存地址，比如空指针，未初始化指针，栈溢出等
> SIGBUS:总线错误，访问对象为初始化。与SIGSEGV不同的是，SIGSEGV访问的是无效地址，而此访问的是有效地址，但是总线访问异常（如地址对齐问题）
> SIGILL:尝试执行非法的指令，可能不被识别或者没有权限

* EXC_BAD_INSTRUCTION
此类异常通常由于线程执行非法指令导致
* EXC_ARITHMETIC
除零错误会抛出此类异常

2. Exception Code
> 0x8badf00d 读作“ate bad food”，程序启动或者恢复时间过长被watch dog终止。
> 0xdead10cc 读作“dead lock” 表示应用因为在后台运行时占用系统资源，如通讯录数据库不释放而被终止。
> 0xbad22222 该编码表示 VoIP 应用因为过于频繁重启而被终止。
> 0xdeadfa11 读做 “dead fall”! 该代码表示应用是被用户强制退出的。根据苹果文档, 强制退出发生在用户长按开关按钮直到出现 “滑动来关机”, 然后长按 Home按钮。强制退出将产生 包含0xdeadfa11 异常编码的崩溃日志, 因为大多数是强制退出是因为应用阻塞了界面。
> 0xbaaaaaad  ⽤用户按住Home键和⾳音量键,获取当前内存状态,不代表崩溃
> 0xc00010ff 读作“cool off” 因为太烫了被干掉

所有的Mach异常是FreeBSD上特有定义的高层异常，会在host层被ux_exception转换为相应的Unix信号，并通过threadsignal将信号投递到出错的线程。我们可以通过注册signalHandler来捕获信号：
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
```
void SignalExceptionHandler(int signal) {
    NSMutableString *mstr = [[NSMutableString alloc] init];
    [mstr appendString:@"Stack:\n"];
    void* callstack[128];
    int i, frames = backtrace(callstack, 128);
    char** strs = backtrace_symbols(callstack, frames);
    for (i = 0; i <frames; ++i) {
        [mstr appendFormat:@"%s\n", strs[i]];
    }
}

void InstallSignalHandler(void) {
    signal(SIGHUP, SignalExceptionHandler);
    signal(SIGINT, SignalExceptionHandler);
    signal(SIGQUIT, SignalExceptionHandler);
    signal(SIGABRT, SignalExceptionHandler);
    signal(SIGILL, SignalExceptionHandler);
    signal(SIGSEGV, SignalExceptionHandler);
    signal(SIGFPE, SignalExceptionHandler);
    signal(SIGBUS, SignalExceptionHandler);
    signal(SIGPIPE, SignalExceptionHandler);
}
    
```
### 应用级异常NSException
某个NSException导致程序Crash，只要我们拿到这个NSException，获取它的reason，name，callStackSymbols信息才能确定出问题的程序位置。方法很简单，通过注册NSUncaughtExceptionHandler捕获异常信息：

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

## Crash日志收集服务
一般情况下，第三方功能性SDK都会集成一个Crash收集服务，用来及时发现自己SDK的问题，但是当集成了多家服务时往往会出现冲突：NSSetUncaughtExceptionHandler（）这个函数是将函数地址当做参数传递，所以只要重复调用就会被覆盖，这样就不能保证崩溃收集的稳定性。
正确的做法是：无论是Signal捕获还是NSException捕获都会存在handler覆盖的问题，我们应该先判断是否前者已经注册了handler，如果有则应该把这个handler保存下来，在自己处理完自己的handler之后，再把这个handler抛出去，供前面的注册者处理。

> 使用友盟、bugly等第三方崩溃统计工具，原理都是根据系统产生的crash日志进行了一次提取或者封装，然后将封装后的crash文件上传到对应的服务端进行解析处理。优点是快速集成crash收集功能，有完善的后台管理界面和解析处理。

## 符号化iOS Crash文件
一般收集到的崩溃日志如下：
首先要了解符号表，符号表是内存地址与函数名、文件名以及行号的映射表。表示如下：
**<起始地址> <结束地址> <函数> [<文件名：行号>]**
为了能够快速并准备定位用户发生crash的代码位置，必须对app发生的crash的程序堆栈进行解析和还原。
等我们符号化之后显示如下：
![](https://ws1.sinaimg.cn/large/006tNc79ly1fr550q09plj311207gdhd.jpg)
这样我们就很容易定位到出错的位置。
### 异常信息
异常信息有三种类型
1. 已标记错误位置的，这种信息很明确了不用解析，如下：
```
    0x0000000109708aeb -[ViewController buttonClick:] + 43
```
2. 有模块地址的情况，如下：
```
test 0x00000001018157dc 0x100064000 + 24844252
```
以上面为例子，从左到右依次是： 二进制库名（test），调用方法的地址（0x00000001018157dc），模块地址（0x100064000）+偏移地址（24844252）

3. 无模块地址的情况
```
    test 0x00000001018157dc test + 24844252
```

### 解析
解析有多种，说一种吧（因为没有打包文件，无法获取的SYM文件，以下均为参考）
1. 获取dSYM符号表
xcode->window->organizer->右键你的应用 show finder->右键.xcarchive 显示包内容->dSYMs->test.app.dYSM
2.atos命令来符号化某个特定模块加载地址
```
atos [-arch 架构名] [-o 符号表] [-l 模块地址] [方法地址]
```
3. 使用终端，进到test.app.dSYM所在目录
* 有模块地址的情况运行
```
atos -arch arm64 -o test.app.dSYM/Contents/Resources/DWARF/test -l 0x100064000 0x00000001018157dc
```

* 无模块地址的情况
现将偏移地址转为16进制
> 24844252 = 0x17B17DC
> 然后方法的地址-偏移地址，得到的就是模块地址
> 0x00000001018157dc - 0x17B17DC = 0x100064000
> 最后运行 
```
atos -arch arm64 -o test.app.dSYM/Contents/Resources/DWARF/test -l 0x100064000 0x00000001018157dc
```
## 测试小Demo
自己写了一个测试小demo， [戳我查看](https://github.com/RiversMaJianCheng/JCCrashTest.git)；

本文参考：
[向晨宇的技术博客](http://www.iosxxx.com/blog/2015-08-29-iosyi-chang-bu-huo.html)
[iOS崩溃crash大解析](https://www.jianshu.com/p/1b804426d212)
