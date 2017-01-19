---
title: 译Yii2-你需要知道的-2(MVC)
date: 2017-01-04 16:16:36
tags:
---

>**Yii What you need to known**系列一共五篇，是几年前的文章。来源博客Hash Solutions，[原文地址](http://blog.hashsolutions.in/technology/yii2-what-you-need-to-know-part-ii/)。
 关于Yii介绍文章其实已经很多了，但是为了锻炼一下自己的英文水平、增强自己对Yii的整体认知影响，所以准备画蛇添足，翻译完这系列文章。
 对于初学者或者想了解Yii的，我觉得**Yii What you need to known**系列应该是很不错的。
 
在这个系列的第二部分，我将介绍Yii2框架的核心概念MVC。Yii2的MVC结构基本与Yii1.1相同。
从图片我们可以看得出（:joy: 原文图片加载不出来了～）。但是当我们深入研究结构的构造时，我们发现了很多的变化，这将使Yii2
平台上的开发称为有价值的体验。

## Models（模型）

Yii中的`Models`从类`yii\base\Model`继承。`Models`通常用于保存数据和定义该数据（也称为业务逻辑）的验证规则。
业务逻辑通过提供验证和错误报告，简化了复杂的web表单生成models过程。

Yii的模型有以下基本特性：

+ 属性声明：model定义属性
+ 属性标签：每个属性可以与用于显示目的的标签相关联。
+ 大规模属性分配：能够在一个步骤中填充多个模型属性
+ 基于场景的数据验证

类Model也是高级model（具有额外功能）的基类，比如`Active Record`。`Active Record`（活动纪录类）的改变
保证了自己的地位，我们会在稍后的篇章进行介绍。

属性和属性标签与框架的上一次迭代或多或少相同。变化更突出的是，模型的场景和验证。替代了之前将场景和验证合二为一的处理函数，
Yii2将其分成了两个函数`rules()`和`scenarios()`。`rules()`指定任何属性的实际验证，而`scenarios()`函数指定哪些属性可以安全地分配给模型。
参考如下例子：

```PHP
class User extends ActiveRecord
{
    public function rules()
    {
        return [
            //当相应的字段是"safe"时，应用规则
            ['username', 'string', 'length' => [4, 32]],
            ['first_name', 'string', 'max' => 128],
            ['password', 'required'],
        ];
    }

    public function scenarios()
    {
        return [
            // on signup allow mass assignment of username
            'signup' => ['username', 'password'],
            'update' => ['username', 'first_name'],
        ];
    }
}
```

这里，`password`只在`signup`场景分配给model，它的验证也只在该场景下应用。
我们感觉到了model验证字段的根本性变化，分配是一个重大的进步，使开发和设计的模式更加灵活和容易。

大规模的分配字段，也发生了一些改变，如下：

```PHP
if (isset($_POST['userModel']))
{
   $model->attributes = $_POST['userModel'];
}
```

也可以`$model->load($_POST);`

这里的关键是`load()`会自动检查`$_POST`中适当的索引，由可覆盖的`$model->formName()`定义。


## Views （视图）

许多细微的变化贯穿于Yii2的View组件。最明显和最显着的变化是`render()`函数现在返回一个值而不是输出它。
一个例子：

```PHP
public function actionIndex()
{
    $models = Posts::find()->all();
    echo $this->render('index', array('models' => $models));
}
```

正如你所看见的渲染的内容可以被(echo)打印（你将会在`Controller`部分看到，返回输出是非常常见的）。此外，
对于那些渴望模板引擎支持的意见，现在有新的官方扩展，支持smarty和twig。

视图的一个主要变化是`$this`的用法。在Yii1.1中`$this`是控制器调用视图的上下文。但是在Yii2中，`$this`是指
`yii\web\View`这个组件。这允许了一些有用且有趣的事情，比如在页面中设置页面的`title`和`meta`、注册脚本等。
这使得代码更加直观。调用视图的对象仍然可以使用`$this->context`。


## Controllers （控制器）

首先展示一段代码：

```PHP
namespace app\controllers;

use yii\web\Controller;

class SiteController extends Controller
{
    public function actionIndex()
    {
        // will render view from "views/site/index.php"
        return $this->render('index');
    }

    public function actionTest()
    {
        // will just print "test" to the browser
        return 'test';
    }
}
```

正如我们所看到的，Yii2很好的使用了PHP的特性，命名空间。一个`action`的输出通常是`return`。
除此之外，控制器的基本原则仍然和原来一样。


## 总结

可以看出，Yii2的MVC组件的几乎所有变化都是更好的，我们相信它们将为开发人员提供增强的体验。
在下一部分中，我们旨在深入了解框架的`ActiveRecord`和`DB`处理部分，因为它经历了大量的更改，因此需要一个完整的篇幅来涵盖所有的内容。

快速导航，其他文章链接：

+ [part1](https://easy-yii.github.io/2017/01/04/know-PartOne/)
+ [part3](https://easy-yii.github.io/2017/01/05/know-PartThree/)
+ [part4](https://easy-yii.github.io/2017/01/05/know-PartFour/)
+ [part5](https://easy-yii.github.io/2017/01/09/know-PartFive/)