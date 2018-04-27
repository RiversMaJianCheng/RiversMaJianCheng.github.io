---
title: Java图形界面开发
date: 2018-04-23 21:06:15
categories:
- Java
tags:
---
## GUI简介
GUI全称是Graphical User Inteface，就是图形用户界面。之前学的Java基础时，都是通过控制台跟程序进行交互，这种方式非常麻烦，用户体验也非常差。有了图形用户界面之后，用户就可以通过程序提供的界面操作软件，生活中，大部分的软件都是使用GUI技术开发的，如QQ，Office，浏览器等。
在Java中，针对GUI开发提供了丰富的类库，这些类库分别位于java.awt 和 javax.swing包下，简称AWT和Swing。AWT在Java1.0就出现了，但是AWT有很多弊端，所以后来SUN公司在AWT的基础上又开发了Swing。
> Swing并没有完全把AWT替换，而是在AWT的基础上进行优化和扩展，Swing中提供了更加丰富和强大的组件。


Swing开发步骤大致可以分为以下几步：
* 创建一个窗体
* 创建一个面板添加到窗体中
* 使用画笔在面板中绘制
* 给面板添加某种事件监听器，并对事件进行处理
下面对以上进行简单的一一说明：
## JFrame窗体
JFrame窗体是Swing中的一个组件，它是一个独立存在的顶级窗口，不能放置在其他容器内同时它也是其它图形界面组件的容器，需要将其他的组件放入到窗体中来完成应用程序界面的设计。JFrame作为窗体拥有图标、标题、最小化、最大化和关闭按钮。如下图所示：
![](https://ws3.sinaimg.cn/large/006tNc79ly1fqritj33ycj30m80goq36.jpg)
相关demo如下：
```
public static void main(String[] args) {
    //创建一个窗体
    JFrame frame = new JFrame();
    //设置窗体大小
    frame.setSize(500, 600);
    //设置窗体的标题
    frame.setTitle("JFrameDemo");
    //设置居中显示
    frame.setLocationRelativeTo(null);
    //设置点击关闭按钮的默许操作
    frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
    //是窗体可见
    frame.setVisible(true);
}
```
## JPanel面板
JPanel面板也是Swing中的一个容器组件，可以将其他的Swing基础组件放入到面板中进行布局，组成丰富多彩的用户界面。面板不能独立存在，只能作为中间容器组件被添加到其他的容器组件中，一般情况下会将面板添加到JFrame窗体中，并且一个窗体中可以添加多个面板。下面是窗体中添加面板以及面板中添加按钮的具体代码以及实现；
```
public static void main(String[] args) {

    //创建一个窗体
    JFrame frame = new JFrame();
    //设置窗体大小
    frame.setSize(500, 600);
    //设置窗体的标题
    frame.setTitle("JFrameDemo");
    //设置居中显示
    frame.setLocationRelativeTo(null);
    //设置点击关闭按钮的默许操作
    frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

    JPanel  panel = new JPanel();
    panel.setBackground(Color.green);
    //创建按钮
    JButton button = new JButton("按钮");
    panel.add(button);
    frame.add(panel);
    //是窗体可见
    frame.setVisible(true);

}
```
图片如下：
![](https://ws3.sinaimg.cn/large/006tNc79ly1fqrjkas8suj30rs0xcgma.jpg)

## 使用画笔在面板中绘制
Swing组件中有很多方法用于绘制，下面重点介绍paint(Graphics g)和repaint() 方法。
paint(Graphics g)方法：是由Swing调用来绘制Swing组件的，应用程序不应该直接调用，应用程序应该使用repaint()方法来对组件进行重新绘制。
repaint()方法：用于重新绘制改组件，调用该方法会引起这个组件的paint(Graphics g)方法尽可能快地调用。

```
public static void main(String[] args) {

    JFrame frame = new JFrame("绘制椭圆");
    frame.setSize(300, 400);
    frame.setLocationRelativeTo(null);
    frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
    DrawOvaPanel panel = new DrawOvaPanel();
    frame.add(panel);
    frame.setVisible(true);
}
```
DrawOvaPanel 的实现如下：
```
public class DrawOvaPanel extends JPanel {
    public void paint(Graphics g) {
        //设置画笔颜色
        g.setColor(Color.RED);
        //通过画笔绘制椭圆
        g.drawOval(100, 150, 100, 200);
    }
}
```
实现的效果如下：
![](https://ws3.sinaimg.cn/large/006tNc79ly1fqrkamozdpj30go0m8jrv.jpg)

## 给面板添加某种事件监听器，并对事件进行处理
首先要清楚几个比较重要的概念：
* 事件源（Event Source）：就是事件发生的地方，通常就是产生事件的组件。例如某个按钮发生了点击事件，那么事件源就是这个按钮。
* 事件（Event）：事件封装了GUI组件上发生的特定事件。例如，单击一个按钮，按键盘上的某个按键等都是事件。GUI中产生事件对象都是继承自Event，该类中有一个方法getSource（）可以获取该事件的事件源对象。
* 事件监听器（Event Listener）：负责监听事件源上发生的事件，并对各种事件作出响应处理。
* 事件处理器：监听器对象监听到事件以后，会调用自身的某个方法对事件进行处理。需要注意的是监听器属于接口类型，实现某一种监听器必须实现该监听器的所有方法，这些方法就是事件处理器。
**事件处理的过程和步骤**
1. 定义实现事件监听接口的类
2. 创建事件监听器
3. 将事件监听器注册到事件源

