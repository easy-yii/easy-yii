---
title: 译Yii2-你需要知道的-4(Events, Behaviors & Errors)
date: 2017-01-05 11:30:04
tags:
---

> **Yii What you need to known**系列一共五篇，是几年前的文章。来源博客Hash Solutions，[原文地址](http://blog.hashsolutions.in/technology/yii2-what-you-need-to-know-part-iv-events-behaviors-errors/)。
关于Yii介绍文章其实已经很多了，但是为了锻炼一下自己的英文水平、增强自己对Yii的整体认知影响，所以准备画蛇添足，翻译完这系列文章。
对于初学者或者想了解Yii的，我觉得**Yii What you need to known**系列应该是很不错的。

在我们探索Yii2框架系列的本篇，我们将会介绍Yii2框架的事件(`Event`)管理、行为(`Behavior`)管理和错误处理(`Error Handling`)。
Yii2处理这些关键特征与Yii相比较而言，有明显的区别。

## 事件(Events)

对于那些从未在Yii项目中使用过`Events`，我要告诉你们，你们错过了一个强大的功能。
`Events`非常有用，有两个原因。首先，它们可以让组件更加灵活。其次，您可以将自己的代码挂接到正在使用的框架和扩展的工作流中。

可以使用`component`对象的`on()`方法为你的代码分配一个`event`处理程序。
第一个参数是，我们要观察的事件名称；第二个参数是，当事件发生时，要调用的函数。

```PHP
$component->on($eventName, $handler);
```

`$handler`可以是以下任意一个：

+ 全局函数的名称
+ 由模型名称和方法名称组成的数组
+ 由对象和方法名组成的数组
+ 匿名函数

示例：

```PHP
// 全局函数:
$component->on($eventName, 'functionName');

// 模型名称和方法名称:
$component->on($eventName, ['Modelname', 'functionName']);

//对象和方法名:
$component->on($eventName, [$obj, 'functionName']);

// 匿名函数:
$component->on($eventName, function ($event) {
// Use $event.
});
```

另外，`event handlers`(事件处理程序)也可以附加在配置文件里。

大多数`events`在正常的工作流程中都会被触发，比如，`beforeSave()`事件会发生在一个`ActiveRecord`保存之前。

但是你依然可以用`trigger`方法手动触发一个事件，在组件中调用附加的处理程序；

```PHP
$this->trigger('myEvent');

// or

$event = new CreateUserEvent(); // extended from yii\base\Event
$event->userName = 'Alexander';
$this->trigger('createUserEvent', $event);

```

相对较新的事件管理特性是，一次性向所有的实例添加事件处理程序；

```PHP
Event::on(ActiveRecord::className(), ActiveRecord::EVENT_AFTER_INSERT, function ($event) {
    Yii::trace(get_class($event->sender) . ' is inserted.');
});
```


## 行为（Behaviors）

在Yii框架中，行为实现了[Mixin](https://en.wikipedia.org/wiki/Mixin)模式。它们可以增强组件的功能，但是不用修改组件的代码；
行为还可以响应在组件中触发的事件，从而拦截正常代码的执行。与PHP`Traits`不同，`behaviors`可以在代码运行时，附加到类。

我们用组件的`behaviors()`方法为其分配`behavior`。`timestamp behavior`（时间戳行为），一个得益于行为的很好的例子。

```PHP
use yii\behaviors\TimestampBehavior;

class User extends ActiveRecord
{
    // ...

    public function behaviors()
    {
        return [
            'timestamp' => [
                'class' => TimestampBehavior::className(),
                'attributes' => [
                    ActiveRecord::EVENT_BEFORE_INSERT => ['created_at', 'updated_at'],
                    ActiveRecord::EVENT_BEFORE_UPDATE => 'updated_at',
                ],
            ],
        ];
    }
}
```

在上面代码中，键名`timestamp`可以在组建中直接引用。比如，`$user->timestamp`给出了附加的`timestamp behavior`（时间戳行为）实例。
相对应的数组，是生成一个`TimestampBehavior`对象的配置。

除了响应`ActiveRecord`的插入和修改事件，`TimestampBehavior`提供了一个方法`touch()`可以将当前时间戳分配给指定的属性。综上所述，
你可以在组件中直接使用此方法，如:`$user->touch('login_time');`

你也可以创建你自己的`behavior`（行为），例如：

```PHP
namespace app\components;

use yii\base\Behavior;

class MyBehavior extends Behavior
{
    public $attr;
}
```

可以在你类中，直接被访问:

```PHP
public function behaviors()
{
    return [
        'mybehavior' => [
            'class' => 'app\components\MyBehavior',
            'attr' => 'member_type'
        ],
    ];
}
```

Yii错误处理总是不同于使用纯PHP处理错误。 Yii将所有非致命错误转换为异常。
```PHP
use yii\base\ErrorException;
use Yii;

try {
    10/0;
} catch (ErrorException $e) {
    Yii::warning("Tried dividing by zero.");
}

// execution may continue
```

如上所示，您可以使用try-catch处理错误。
`exception`对象有如下属性：

+ 状态码： HTTP状态码（如，403，500）；仅适用于HTTP异常。
+ 代码：异常的代码；
+ 信息：错误信息；
+ 文件：错误发生时，PHP脚本的文件名；
+ 行数：错误发生在哪一行；
+ 跟踪：调用堆栈的错误。

## 总结：

可以看出Yii2的事件处理和行为是非常值得关注的。
希望你喜欢这个系列中的这一期。请随时提供您的反馈和想法。


快速导航，其他文章链接：

+ [part1](https://easy-yii.github.io/2017/01/04/know-PartOne/)
+ [part2](https://easy-yii.github.io/2017/01/04/know-PartTwo/)
+ [part3](https://easy-yii.github.io/2017/01/05/know-PartThree/)
+ [part5](https://easy-yii.github.io/2017/01/09/know-PartFive/)