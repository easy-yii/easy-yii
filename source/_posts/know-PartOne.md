---
title: 译Yii2-你需要知道的-1
date: 2017-01-04 11:38:30
tags:
---

> **Yii What you need to known**系列一共五篇，是几年前的文章。来源博客Hash Solutions，[原文地址](http://blog.hashsolutions.in/technology/yii2-need-know/)。
关于Yii介绍文章其实已经很多了，但是为了锻炼一下自己的英文水平、增强自己对Yii的整体认知影响，所以准备画蛇添足，翻译完这系列文章。
对于初学者或者想了解Yii的，我觉得**Yii What you need to known**系列应该是很不错的。

## 简介(Introduction)
  Yii项目始于2008年1月1日，主要由当时开发PRADO框架中的一些人负责的。他们的愿景是改进PRADO框架并解决其所有缺点。
在2008年10月3日，Yii1.0正式发布，因其性能、易用且遵守当时行业的一些最佳做法，所以迅速成为PHP框架大军中的主要力量。

随着时间的推移，PHP语言也在发展、升华、而且加入了越来越多新的特性。Yii的开发者也开始思考用PHP的高级特性(比如命
名空间、匿名函数等)重写一个新的Yii框架。Yii2.0开始发展，它至少要支持PHP5.4+(Yii2.0可以支持到PHP5.1+)，并于2011
年4月第一次提交了代码。仅仅是一年的发展，现在团队已经发布了Yii2.0的第一个alpha版本，当我写这篇文章时，他们正在准备测试
版本，这实在是值得祝贺的壮举。

所以，我认为是时候看看这个前沿框架了。在这篇文章及随后几篇文章，我会仔细看看Yii2就新的特性和实践标准为开发者提供了些什么。
这些文章主要面向已经有Yii开发经验和想知道Yii的预计变化的开发者。
    
## 命名空间(Namespaces)
最明显的改变是，你在使用Yii2的时候，会用到namespaces(命名空间)。Namespaces(命名空间)是在php5.3引入的，你会发现每一个Yii2的核心类现在都被命名空间了。
（例如，yii\web\Request）。它摆脱了在Yii1.1类名中使用的“C”前缀，而且命名空间摆脱了命名空间冲突的可能。同时允许更短的类名，而且在此过程中提高了代码的可读性。

## 对象和组件类(Object and Component Class)
Yii1.1中的类`CComponent`已经被拆分成两个类：`\yii\base\Object`和`\yii\base\Component`。`Object`是基类，你可以在创建或者管理自定义数据
结构的时候用到它。`Object`具有统一配置方式，可以通过配置数组来初始化。类似下面的代码可以用来初始化各种对象：

```PHP
$object = Yii::createObject([

'class' => 'MyClass',

'property1' => 'abc',

'property2' => 'cde',

], [$param1, $param2]);
```

`Components`(组件) 都是 `Objects`(对象)，具有一些额外的特性如`Event Handling`(事件处理)和`Behaviors`(行为).`Event Handling`(事件处理)
在Yii2中更加强大。现在没有必要去定义一个`on`开头的方法了。现在为`event`绑定一个`handler`应该用`on`方法，如`$component->on($eventName, $handler, $params);`
`Events`特别有用，不管是在让组件变的更加灵活还是让框架和扩展耦合的更好。现在可以用`trigger`触发方法来触发事件，如：`$this->trigger('myEvent');`
    
`Behaviors`(行为，在其他语言中称为mixins)可以增强现存组件的功能，而不用修改他的代码。`Behaviors`可以通过扩展组件的`behaviors()`方法或使用`attachBehavior()`方法来添加。

## 路径别名(Path Aliases)
Yii2将路径别名的使用扩展到了文件目录路径和URL。一个别名必须以`@`开头作为标示符，这样才可以和文件目录路径及URL区分开来。例如，别名`@yii`指的是Yii安装目录，而`@web`包含当前运行的Web应用程序的基本URL。
以下别名由核心框架预定义：

+ `@yii`，框架目录
+ `@app`，当前程序运行的基本路径
+ `@runtime`，运行目录
+ `@vendor`，composer扩展目录
+ `@webroot`，当前运行的Web应用程序的web根目录
+ `@web`，当前运行的Web应用程序的基本URL

## helper类(Helper Classes)
    
`Helper`类只包含静态方法，使用方法如下：

```PHP
    use \yii\helpers\Html;
    echo Html::encode('Test > test');
```

框架提供了下面几个类：

+ ArrayHelper
+ Console
+ FileHelper
+ Html
+ HtmlPurifier
+ Image
+ Inflector
+ Json
+ Markdown
+ Security
+ StringHelper
+ Url
+ VarDumper

## 总结(Conclusion)
这些都是Yii2.0框架最基本的知识，为将框架设计中的最新和最前沿概念带入Yii平台奠定了基础。在下一篇文章中，
我将详细讨论MVC概念，以及它们在这个流行框架的新版本中如何改变。

快速导航，其他文章链接：

+ [part2](https://easy-yii.github.io/2017/01/04/know-PartTwo/)
+ [part3](https://easy-yii.github.io/2017/01/05/know-PartThree/)
+ [part4](https://easy-yii.github.io/2017/01/05/know-PartFour/)
+ [part5](https://easy-yii.github.io/2017/01/09/know-PartFive/)