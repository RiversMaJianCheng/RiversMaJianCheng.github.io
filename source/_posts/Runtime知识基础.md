---
title: Runtime基础知识
date: 2018-04-27 09:46:34
tags:
---
首先看一下我们Objective-C中的方法调用**[receiver message]**，这就叫“发消息”，这个消息机制是由运行时实现的，它会被编译器转化为：
```
objc_msgSend(receiver, selector)
```
如果消息有参数，会被转化为：
```
objc_msgSend(receiver, selector, arg1, arg2, ...)
```
如果消息的接受者能够找到对应的selector，那么就相当于直接执行了接收者这个对象的特定方法，否则要么被转发，或者临时向接收者动态添加这个selector对应的实现内容，要么就崩溃了。
Objective-C Runtime是一个将C语言转换成面向对象语言的扩展。OC无法通过编译器直接把函数地址硬编码进入可执行文件，而是在程序运行时候，利用RunTime根据条件作出决定。

## Runtime基础数据结构
objc_msgSend(receiver, selector)，他的真身是：
```
id objc_msgSend(id self, SEL op, ...);
苹果官方文档解释：
self：A pointer that points to the instance of the class that is to receive the message.（一个指向接受消息的类的实例指针）
op：The selector of the method that handles the message.（处理消息的方法选择器）
```
### SEL
SEL的数据结构是这样的：
```
typedef struct objc_selector *SEL;
方法选择器在运行时中被用来代表方法名；一个方法选择器就是一个在运行时时被注册的一个C字符串；
```
objc_msgSend函数第二个参数类型为SEL，selector是方法选择器，可以理解为区分方法的ID，这个ID的数据结构是SEL；你可以用 Objc 编译器命令 @selector() 或者 Runtime 系统的 sel_registerName 函数来获得一个 SEL 类型的方法选择器。
### id和Class

1. objc_msgSend 第一个参数类型为id，它是一个指向类实例的指针，也是我们常说的对象。
```
typedef struct objc_object *id;

```
id是一个指向objc_object的结构体指针，它的数据结构如下：
```
struct objc_object {
private:
    isa_t isa;

public:

    // ISA() assumes this is NOT a tagged pointer object
    Class ISA();

    // getIsa() allows this to be a tagged pointer object
    Class getIsa();
    ... 此处省略其他方法声明
}
```
objc_object 结构体包含一个isa指针，类型为isa_t联合体。根据isa就可以找到对象所属的类。
PS：isa指针不总是指向实例对象所属的类，不能依靠它来确定类型，而是应该用class方法来确定实例对象的类。因为KVO的实现机理就是将被观察对象的isa指针指向一个生成的中间类而不是我们真实创建的类。

isa其实会在三个对象中存在：实例对象会有isa指针，类对象也有isa指针，元类对象也有isa指针。实例对象的isa指针指向的是类，类对象的isa指针指向的是元类，元类的isa指针指向的是根元类。根元类的isa指针指向他自己（一般为NSObject的元类）。

2. Class是一个指向objc_class结构体的指针。就是我们常说的类。
```
typedef struct objc_class *Class;
```
objc_class包含很多方法，主要都是对它的几个成员做处理。
```
struct objc_class : objc_object {
    // Class ISA;
    Class superclass;
    cache_t cache;             // formerly cache pointer and vtable
    class_data_bits_t bits;    // class_rw_t * plus custom rr/alloc flags
    class_rw_t *data() { 
        return bits.data();
    }
... 省略其他方法
}
```
以上可以看出：类也可以当做一个objc_object来对待，也就是说类和对象都是对象，分别称作类对象（class object）和实例对象（instance object）。

类与对象的继承层次关系如下图：
![](https://ws4.sinaimg.cn/large/006tNc79ly1fqr5ogradrj30px0r541d.jpg)


由上图可以看出：
> 实例对象的isa指针指向（该对象所属的）类（class），这个类中存放着普通成员变量和动态方法（‘-’开头的方法）
> 类的isa指针指向元类（metaclass），元类中存放着static类型的成员变量和static类型的方法（‘+’开头的方法）
> 所有的metaclass的isa指针都指向根metaclass，而根metaclass的isa指针指向自己
> 根metaclass的超类是NSObject，NSObject的超类为nil（没有超类）

### IMP
IMP在objc.h中的定义是：
```
typedef void (*IMP)(void /* id, SEL, ... */ );
```
它就是一个函数指针，由编译器生成，当发送消息时，是由这个函数指针决定最终要执行的那段代码。而IMP这个函数就指向了这个方法的实现。
IMP指向的方法与objc_msgSend函数类型相同，参数都包含id和SEL类型。每个方法名都对应一个SEL类型的方法选择器，每个实例对象中的SEL对应的方法实现也是唯一的，通过一组id和SEL参数就能确定唯一的方法实现地址，反之亦然。

## 消息传递
在发送消息时，会转换成Runtime的objc_msgSend函数调用。
```
[receiver message];
```
Runtime 会将其转成类似这样的：
```
objc_msgSend(receiver, selector);
```
具体规则如下：
Runtime会根据类型自动转换成下列某一个函数：
> objc_msgSend:普通的消息都会通过该函数发送
> objc_msgSend_stret:消息中有数据结构作为返回值时，通过此函数发送和接受返回值
> objc_msgSendSuper: 和objc_msgSend类似，把消息发送给父类实例
> objc_msgSendSuper_stret:和objc_msgSend_stret类型，这里把消息发给父类的实例并接受返回值。

当消息被发送到实例对象时，是如下图所示处理的：
![](https://ws1.sinaimg.cn/large/006tKfTcly1fqr7cowenqj30qk0w840d.jpg)

objc_msgSend函数的调用过程：
第一步：检测这个selector是不是要忽略的。
第二步：检测这个target是不是nil对象。nil对象发送任何一个消息都会被忽略掉。
第三步：
1. 调用实例方法时，它会首先在自身isa指针指向的类（class）methodLists中查找该方法，如果找不到则会通过class的super_class指针找到父类的类对象结构体，然后从methodLists中查找该方法，如果仍然找不到，则继续通过super_class向上一级父类结构体中查找，直至根class；
2. 当我们调用某个某个类方法时，它会首先通过自己的isa指针找到metaclass，并从其中methodLists中查找该类方法，如果找不到则会通过metaclass的super_class指针找到父类的metaclass对象结构体，然后从methodLists中查找该方法，如果仍然找不到，则继续通过super_class向上一级父类结构体中查找，直至根metaclass；
第四部：前三部都找不到就会进入动态方法解析。

## Runtime消息转发
发送消息后，如果当前类找不到则会沿着继承树向上一直搜索直到跟类(一般是NSObject)，如果还是找不到并且消息转发都失败了就会抛出unrecognized selector。下面介绍消息转发的最后三次机会：
* 动态方法解析
* 备用接受者
* 完整消息转发
![](https://ws4.sinaimg.cn/large/006tKfTcly1fqr7jamac9j30sv0fm0tq.jpg)

1. 动态方法解析：object-c运行时会调用+resolveInstanceMothod: 或者 +resolveClassMethod: 让你有机会提供一个函数的实现，如果添加了函数（你需要用class_addMethod函数完成向特定类添加特定方法实现），那么消息就得到处理。
2. 备用接受者：如果上个方法我们没有实现消息处理，运行时就会进行下一步：forwardingTargetForSelector:，如果目标对象实现了-forwardingTargetForSelector:，runtime这时就会调用这个方法，让你有把这个消息转发给其他对象的机会。
3. 完整消息转发：当备用接受者不做消息处理返回为nil时，完整消息转发机制就会被触发，这时forwardInvocation:方法会被执行。流程是：首先发送-methodSignatureForSelector:消息获得函数的参数和返回值类型。如果-methodSignatureForSelector:返回nil，runtime则会发出-doesNotRecognizeSelector:消息，也就是程序崩溃了。如果返回了一个函数签名，runtime就会创建一个NSInvocation对象并发送-forwardInvocation: 消息给目标对象。

### 动态方法解析
```
void xiaomageMethod(id obj, SEL _cmd){
    NSLog(@"我消息转发了第一个过程");
}
+ (BOOL)resolveInstanceMethod:(SEL)sel{

    //模拟第一个流程
    //如果是执行foo函数，就动态解析，指定新的IMP
    if (sel == @selector(xiaomage)) {
        class_addMethod([self class], sel, (IMP)xiaomageMethod, "v@");
        return YES;
    }
    return [super resolveInstanceMethod:sel];

}
```
这里需要注意的是只要实现了xiaomageMethod方法，无论返回YES或者NO，都不会进行下一步的消息转发。（我猜测是我们调用了xiaomageMethod的IMP，就认为消息转发成功，也就没有必要进行下去）。

###  备用接受者
```
- (id)forwardingTargetForSelector:(SEL)aSelector{

    //模拟第二步消息转发流程
    SecondStepForward *secondStep = [[SecondStepForward alloc] init];
    if ([secondStep respondsToSelector:aSelector]) {
        return secondStep;
    }
    return [super forwardingTargetForSelector:aSelector];
}
```
这个方法，我们要把处理对象转移给其他类，如果这里返回self的话，因为self没有实现xiaomage的方法，所以跟返回nil没有区别的。（可能是编译器做了优化处理，我看很多文章都是这里返回位self会死循环，实际测试不会的）。这里消息只能被转发给一个对象。

### 完整消息转发
```
- (NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector{

    NSMethodSignature *signature =  [super methodSignatureForSelector:aSelector];
    if (!signature) {
        if ([ThirdStepForward instancesRespondToSelector:aSelector]) {
        signature = [ThirdStepForward instanceMethodSignatureForSelector:aSelector];
        }
    }
    return signature;
    //下面这种方式也可以
    /*
    if (aSelector == @selector(xiaomage)) {
        return [NSMethodSignature signatureWithObjCTypes:"v@:"];////签名，进入forwardInvocation
    }
    */

}

- (void)forwardInvocation:(NSInvocation *)anInvocation{
    NSLog(@"anInvocation = %@",anInvocation.target);
    SEL sel = anInvocation.selector;

    if ([ThirdStepForward instancesRespondToSelector:sel]) {
        [anInvocation invokeWithTarget:[[ThirdStepForward alloc] init]];
    }else{
        [self doesNotRecognizeSelector:sel];
    }

    //下面方法也可以
    /*
    ThirdStepForward *thirdStep = [[ThirdStepForward alloc] init];
    if ([thirdStep respondsToSelector:sel]) {
        [anInvocation invokeWithTarget:thirdStep];
    }else{
        [self doesNotRecognizeSelector:sel];
    }
    */
}

```
相关的描述格式：[戳我查看](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Articles/ocrtTypeEncodings.html#//apple_ref/doc/uid/TP40008048-CH100-SW1);
这里首先会发送methodSignatureForSelector消息获取一个函数的签名（包含返回值，参数，以及target），如果这个返回值为nil，那程序就调用doesNotRecognizeSelector挂掉了。返回一个方法签名后，就会创建一个NSInvocation对象并发送给forwardInvocation消息给目标对象。然后我们在forwardInvocation方法里让其他对象执行相应的方法。这里可以将消息发送给任意多对象。

## demo
[戳我查看](https://github.com/RiversMaJianCheng/runtimeDemo.git)

## 参考
[iOS runtime forwardInvocation一些总结](https://blog.csdn.net/zhaochen_009/article/details/54602930)
[iOS Runtime详解](https://juejin.im/post/5ac0a6116fb9a028de44d717)
[Objective-C Runtime](http://yulingtianxia.com/blog/2014/11/05/objective-c-runtime/)


