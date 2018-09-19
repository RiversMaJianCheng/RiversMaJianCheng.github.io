---
title: iWatch开发基础
date: 2018-07-05 15:48:37
tags:
---
最近研究了一下iWatch开发。下面就介绍开发的基本步骤吧。

1. 首先WatchKit应用需要一个配套的iOS应用，WatchKit应用和WatchKit扩展捆绑在一起打包进iOS应用中的。并且WatchKit开发需要iOS8.2 SDK及以后版本。

2. 创建Watch工程有两种方式，一种是直接创建新的Watch项目；还有一种是在当前iOS应用Xcode项目中添加一个新的WatchKit target，Xcode会自动配置并初始化WatchKit应用以及WatchKit扩展需要的资源。

## 建工程（用第二种方式创建）
如下图：
1. 添加Watch Target 
![](https://ws4.sinaimg.cn/large/006tNbRwgy1fvesfny9wlj31iw0vedlw.jpg)

2. 创建好成功之后，就会出现类似下图所示：

![](https://ws2.sinaimg.cn/large/006tNbRwgy1fveunu7extj31kw1bih3c.jpg)

3. 接着开始选择iWatch运行如下图：
![](https://ws1.sinaimg.cn/large/006tNbRwgy1fveur3ry5tj31kw0ujh4e.jpg)

4. 运行结果如下图：
![](https://ws2.sinaimg.cn/large/006tNbRwgy1fveuyu9cf4j31421fw1a1.jpg)

## iWatch 界面布局
简单来说就是iwatch写界面一定要用storyboard，能够极大地节约开发时间和成本。但用storyboard开发时，一定要善于使用group控件，group控件能够极大方便调整布局。

如下图l层级结构所示：
![](https://ws1.sinaimg.cn/large/006tNbRwgy1fvevh4licvj31h00tc43e.jpg)

## iWatch 传值
一般来说，iWatch只是用来展示数据，一般不需要让iWatch对手机端App进行传值，一般都是手机端的数据展示在iWatch上的。（下面演示的是双向传值的过程）


### iWatchApp 中
 首先配置和激活会话（ViewController类中）
 ```objc
 - (void)viewDidLoad {
 [super viewDidLoad];
     if([WCSession isSupported]){
         WCSession *session = [WCSession defaultSession];
         session.delegate = self;
         [session activateSession];
     }
 }
 
 - (IBAction)clickButton:(UIButton *)sender {
 
     dispatch_async(dispatch_get_main_queue(), ^{
     
         int i = arc4random() % 100;
         NSString *string = [NSString stringWithFormat:@"%.2f", 8 + (i / 100.f)];
         //发送消息给iWatch
         [[WCSession defaultSession]sendMessage:@{@"iOS":string} replyHandler:^(NSDictionary<NSString *,id> * _Nonnull replyMessage) {
         } errorHandler:^(NSError * _Nonnull error) {
         }];
    });
     
     
 }
 -(void)session:(WCSession *)session didReceiveMessage:(NSDictionary<NSString *,id> *)message replyHandler:(void (^)(NSDictionary<NSString *,id> * _Nonnull))replyHandler
 {
     NSString *msg=[NSString stringWithFormat:@"%@",[message valueForKey:@"watch"]];
     self.labelMsg.text = msg;
     NSLog(@"%@",message[@"watch"]);
     if (replyHandler) {
         replyHandler(@{@"IOS":@"IOS"});
     }
 }

 ```

### iWatchKit Extension 中
 首先配置和激活会话（InterfaceController类中）

```Objc
- (void)willActivate {
    [super willActivate];
    if([WCSession isSupported]){
        WCSession *session = [WCSession defaultSession];
        session.delegate = self;
        [session activateSession];
    }
}

#pragma mark =====delegate =======
//收到消息代理方法
-(void)session:(WCSession *)session didReceiveMessage:(NSDictionary<NSString     *,id> *)message replyHandler:(void (^)(NSDictionary<NSString *,id> * _Nonnull))replyHandler
    {
    dispatch_async(dispatch_get_main_queue(), ^{
        NSString *msg=[NSString stringWithFormat:@"%@",[message valueForKey:@"iOS"]];
        [self.labelMoney setText: msg];
        //向iPhone发送回复消息，代码块参数不能为nil
        NSString *string = [NSString stringWithFormat:@"接收从iWatch发送数据成功-%@-",msg];
        [session sendMessage:@{@"watch":string}    replyHandler:^(NSDictionary<NSString *,id> * _Nonnull replyMessage) {
        } errorHandler:^(NSError * _Nonnull error) {
        }];
    });

    if (replyHandler) {
        replyHandler(@{@"WATCH":@"WATCH"});
    }
}

```
[demo点我](https://github.com/RiversMaJianCheng/blogDemo.git)

