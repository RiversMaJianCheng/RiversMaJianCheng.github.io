---
title: Java 基础语法学习及程序设计
date: 2018-04-17 20:43:08
categories:
- Java
tags:
---
首先Java是一门面向对象的编程语言，吸收了C++语言的各种优点，并且摒弃了C++里难以理解的多继承、指针等概念。1996年1月，Sun公司发布了Java的第一个开发工具包（JDK 1.0），这是Java发展历程中的重要里程碑，标志着Java成为一种独立的开发语言。

> 下面说一下JDK、JRE、JVM的含义：
> JDK(Java Development Kit)：Java 开发工具包。
> JRE (Java Runtime Environment)：Java运行环境。
> JVM(Java Virtual Machine)：Java虚拟机。
>其中JDK中包含JRE，JRE中包含JVM。如果仅仅执行Java程序，那么仅安装JRE即可，如果要开发Java程序，则需要安装JDK，在运行Java程序是需要启动JVM，由JVM加载、解释、运行.class字节码文件。
**本文的讲解都是在集成开发工具是Eclipse**

本文主要介绍Java基础语法及程序设计进行讲解，将分为以下几个章节：
* 语法基础
* 流程控制语句
* 数组和方法
* 类和对象
* 面向对象的三大特征
* 抽象类与接口
* 对象的管理

## 语法基础
1. Java语法格式：
```
package 包路径;
public class 类名 {
    public statid void main(String[] args) {
        程序代码
    }
}
```
2. main方法是执行程序的入口，当开始运行当前程序时，Java虚拟机会按顺序由上至下执行main方法内部的程序代码。
定义标识符不能与关键字冲突，业界定义标识符时遵守一定规范大致如下：
* 包名一律使用小写字母
* 类名中每个单词首字母都要大写
* 变量名与方法名使用“驼峰命名法”
* 常用名所有字母大写，如有多个单词则使用“_”连接，如USER_ID
注释分为三种：

单行注释
```
 //我是一个单行注释
```
多行注释
```
/*我是一个多行注释*/
```
文档注释
```
/**
 *这是啥意思
 *这又是啥意思
 *我是文档注释
 */
```
变量的声明：需要注意的是局部变量声明后必须进行初始化。不然编译器会报错。
Java的基本数据类型有八种：byte、short、int、long、float、double、char、boolean。
**注：**需要注意的是不同类型变量赋值时的数值溢出问题，如下：
```
public class Example {

    public static void mian(String[] args) {
    
        long index = 300; //300(int类型)隐式转换成long
        int anther = index; //把index变量赋值给anther，编译器报错
        int anther = (int)index; //强制转换为int不会报错，可能会出现值的问题
    }

}
```
## 流程控制语句
流程控制语句主要分为3种：顺序结构、分支结构和循环结构。
1. 顺序结构就是我们常编写的代码，从上到下执行的结构。
2. 分支结构分为两大类：if语句和switch语句。
3. 循环结构语句有三类：
    * while循环
    * do...while循环
    * for 循环

鉴于流程控制语句比较简单就不多介绍了。

## 数组和方法
1. 数组时Java中极为常用的数据类型，声明数组变量时需要明确定义数组中存储的数据类型，如果数组中存储int类型数据就要声明为：int[]。Java要求声明数组时必须指定元素的数据类型，并且数组中的元素必须与该类型一致，否则会导致语法错误。如下初始化：
```
int[] ary;
ary = new int[5];
double[] ary2 = new double[5];
```
还可以如下初始化：
```
int[] ary1 = {100, 200, 300, 400};
int[] ary2 = new int[] {100, 200, 300, 400};
```
**注：**
    * 整型数组元素的默认值为0
    * 浮点型数组元素的默认值为0.0
    * boolean类型数组元素的默认值为false
    * char类型元素的默认值看不到，也是有的是Unicode编码为0的字符
    * String类型数组默认值为null
Java数据类型分为两大类：基本数据类型和引用数据类型。
2. 方法是程序的核心，Java中定义一个方法的具体格式如下：

