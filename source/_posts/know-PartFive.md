---
title: 译Yii2-你需要知道的-5(Yii2应用结构，安全)
date: 2017-01-09 16:19:17
tags:
---


> **Yii What you need to known**系列一共五篇，是几年前的文章。来源博客Hash Solutions，[原文地址](http://blog.hashsolutions.in/technology/yii2-what-you-need-to-know-part-iv-events-behaviors-errors/)。
关于Yii介绍文章其实已经很多了，但是为了锻炼一下自己的英文水平、增强自己对Yii的整体认知影响，所以准备画蛇添足，翻译完这系列文章。
对于初学者或者想了解Yii的，我觉得**Yii What you need to known**系列应该是很不错的。

在Yii2和它的变化系列的最后一期，我们会快速的浏览一下这个流行框架这次更新的其他细节，是我们在前面章节没有提到的。


## Yii2应用结构&安装

在开始开发Yii2，你会注意到的第一件事情是Yii2是一个新的、改进过的应用架构以及安装过程。
Yii2用composer来管理依赖。此外，Yii2应用结构分为两大类，基础版本和高级版本。
基本安装，创建一个基本的应用程序结构，开箱即用。对于小型和通用的Web应用程序，基本安装将是一个很好的起点。

许多开发人员都抱怨过Yii是一个相当基本的应用结构。Yii2纠正了这一点，它提供了一个先进的应用结构，它将整个应用程序划分为：

+ backend－后端开发应用程序
+ common－整个应用共用的文件
+ console－控制台应用程序
+ environments－环境配置
+ frontend－前端开发应用程序

这是非常厉害的，它提供了一个架构可以让你立马开始在一个更高级的应用上工作。在之前的迭代，使用Yiiboilerplate或类似
的代码可以相同的效果。这些目录，或多或少类似于Yii1.1。如下图所示：

![应用结构](http://i2.wp.com/blog.hashsolutions.in/wp-content/uploads/2014/10/application-structure.png)


## 安全

安全性是Yii框架的开发人员深入研究的一个方面。几位安全专家，包括[Tom Worster](https://github.com/tom--) and [Anthony Ferrara](https://github.com/ircmaxell),
已经回顾了Yii的代码，确保该框架尽可能的安全。

yii2框架提供了Yii 1.0的许多特性，使得实现安全应用程序变得轻而易举，如类`UserIdentity`,访问控制过滤器和基于角色的访问控制。
所有这些特征的实现或多或少地类似于先前的迭代。

在Yii2中，新增加了一个组件`security`，是安全性变的更强大和容易。因此，安全相关功能可以使用以下语句轻松访问：
```PHP
Yii::$app->security->encrypt()
```


## Transactions (事务)

事务的最大变化是使用回调来定义事务的能力。

```PHP
$connection->transaction(function() {
    $order = new Order($customer);
    $order->save();
    $order->addItems($items);
});

```

此外，有一些为`transactions`定义的事件，允许你在 `transaction`生命周期内调用相应的操作。
`beginTransaction`和`commitTransaction`就是两个事件，它们分别在刚开始和提交的时候被触发。


## AssetManagement(资源管理)

Yii2终于带来了最新和最好的软件包管理工具，以管理资源。Yii2通过composer接口使用bower和npm为您的项目带来全面和现代的依赖关系管理。
有关资源管理的更多信息，请参阅这里的[包管理工具](http://www.yiiframework.com/doc-2.0/guide-structure-assets.html)。

## 开发工具

Yii的调试工具(Debugger)在改进和扩大之后，非常的像 Symfony 的调试工具。
此外，Gii工具现在可以从控制台运行，从而提高生产力，同时执行代码生成的重复任务。

## 模版引擎

已经为流行的模板引擎（如Smarty和Twig）添加了支持，以及与Yii2一起使用的特殊语法。

## 总结

虽然本系列文章快速浏览了Yii2框架中的所有功能和改进的部分，但它并不能全面列出所有可以找到的变化。
当我写这篇文章，测试版本已经发布，最终版本预计将在未来几个星期发布。现在是个好机会，看看这些这好东西了。
以下是一些链接，可以帮助你开始：

+ [官方指南](http://www.yiiframework.com/doc-2.0/index.html)
+ [Api类](http://www.yiiframework.com/doc-2.0/index.html)
+ [github仓库](https://github.com/yiisoft/yii2)

希望你喜欢这个系列。请让我们知道你对Yii2的看法，以及它是否真的是PHP框架的未来。

快速导航，其他文章链接：

+ [part1](https://easy-yii.github.io/2017/01/04/know-PartOne/)
+ [part2](https://easy-yii.github.io/2017/01/04/know-PartTwo/)
+ [part3](https://easy-yii.github.io/2017/01/05/know-PartThree/)
+ [part4](https://easy-yii.github.io/2017/01/05/know-PartFour/)


> 恩，翻译完了，真的是很老的文章了。文章虽老，但是写的很好。让我对Yii更加深刻，之前一直觉得行为和事件有点神秘，一直没有去尝试，
在翻译的过程中试着写了写，恩，其实也没那么神秘。代码，真的是，写了，才算是你的～～个人觉得可以在学习过程中，看看[深入理解Yii2.0](http://www.digpage.com/index.html)
理解理解其背后的原理～吐槽一下，不知道为什么感觉Yii不是特别活跃呢～～感觉并不像Laravel那么活跃，感觉一个像是中年人，一个像是青少年～～
