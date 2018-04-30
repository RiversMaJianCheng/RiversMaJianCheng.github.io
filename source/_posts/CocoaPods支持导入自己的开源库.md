---
title: CocoaPods支持导入自己的开源库
date: 2018-04-30 07:24:30
tags:
---
## GitHub建仓库
去GitHub建自己的仓库，如下图所示：
![](https://ws1.sinaimg.cn/large/006tKfTcly1fquc2hqs1pj31bg0z8wkm.jpg)

1. 仓库建好之后，把项目克隆到本地，cd到克隆下来的文件夹
2. 在此文件夹下建一个工程项目（或者建好之后拖进来）
3. 创建存放开源代码的文件夹和pod关联文件，然后进入工程把开源代码文件夹add进去
其中创建pod关联文件命令如下(注意保持pod不带后缀的文件名和存放开源代码文件夹的名称一致)：
```
    pod spec create 你开源到pod的文件名
```
以上都建立好之后就是下图的样子：
![](https://ws3.sinaimg.cn/large/006tKfTcly1fqucjptpszj30re0midhv.jpg)

## 配置.podspec内容
这个根据自己的情况配置podspec文件简单模板(我的如下)：
```
Pod::Spec.new do |s|

    s.name         = "JCDrawSpiderChart"
    s.version      = "0.0.2"
    s.summary      = "动态蜘蛛网状图"
    s.description  = "根据传入的参数，动态绘制蜘蛛网状图"

    s.homepage     = "https://github.com/RiversMaJianCheng/JCDrawSpiderChart"
    # s.screenshots  = "www.example.com/screenshots_1.gif", "www.example.com/screenshots_2.gif"

    s.license      = "MIT"
    s.author             = { "majiancheng" => "“ma_jcheng@126.com”" }

    # s.platform     = :ios
    s.platform     = :ios, "9.0"

    s.source       = { :git => "https://github.com/RiversMaJianCheng/JCDrawSpiderChart.git", :tag => "#{s.version}" }
    s.source_files  = "JCDrawSpiderChart", "JCDrawSpiderChart/**/*.{h,m}"
    #s.exclude_files = "Classes/Exclude"
    s.requires_arc = true
    s.framework      = "UIKit"
    # s.xcconfig = { "HEADER_SEARCH_PATHS" => "$(SDKROOT)/usr/include/libxml2" }
    # s.dependency "JSONKit", "~> 1.4"

end
```
> s.version：版本号，每次更新必定修改的内容
> s.author：作者以及邮箱
> s.platform：平台以及最低版本限制
> s.source：源代码的一个路径，这里指向的github上对应的某一个版本，而版本号就是和该文件版本相关联，所以这里要确保github上存在这个版本号，也就是验证之前的打标签
> s.source_files：资源文件路径
> s.dependency：依赖库，有时候代码会依赖于其他第三方，需要在这里配置

**以上都配置好之后，push到GitHub**，相关命令如下：
```
git add .
git commit -m "提交文件"
git push
```
## 验证 podspec 文件
### 打标签

在当前文件件下给项目打上标签即版本号，注意标签的版本号必须与.podspec后缀的文件中的版本号相同，执行命令行：
```
git tag 0.0.2（本地版本）
git push --tags（打上远程版本）
```
### 注册
对于一个新的开源库第一次添加cocoapods支持，是需要进行注册的，而且会有邮件进行确认（进入自己的邮箱点击链接确认）
```
// 邮箱 以及 名称 进行注册
pod trunk register ma_jcheng@126.com 'jianchengma'

//注册成功后可以查看个人信息
pod trunk me
```
### 验证
* 直接验证
```
pod spec lint JCDrawSpiderChart.podspec
```
* 忽略警告（如果有错误就要根据提示修改，警告可以直接忽略）
```
pod spec lint --allow-warnings
```
## 推送podsec文件到cocoapods
推送的过程中会再次验证一遍，所以有时候有会出现验证不通过的情况
```
pod trunk push JCDrawSpiderChart.podspec
```
如果在验证的时候添加了忽略警告命令，那么在推送的时候也必须加上，不然前面的警告会再次出现导致验证又通不过。
```
pod trunk push JCDrawSpiderChart.podspec --allow-warnings
```
整个验证效果图如下：
![](https://ws3.sinaimg.cn/large/006tKfTcly1fqudg52p1bj317a0tqn5i.jpg)

**注：**
这时候执行命令会出现以下结果：
```
pod search JCDrawSpiderChart
```
![](https://ws2.sinaimg.cn/large/006tKfTcly1fqudqs8jpmj30vk09640m.jpg)

## 更新.podspec文件
由于我们的开源库版本更新，所以我们的 podspec 文件也必须更新，至于更新步骤就是重复上面步骤：
1. 修改podspec文件中s.version对应的版本号
2. 给我们开源库打上标签（tag 和 s.version是一致的）
3. 验证
4. 推送

## 参考
[CocoaPods支持导入自己的开源库](https://arthurcao.com/2017/04/25/cocoapods-and-podspec/)
[把代码开源到cocoapods](http://www.rockyd.cn/2016/12/25/2016-12-25codeopensource/)
