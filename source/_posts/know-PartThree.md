---
title: 译Yii2-你需要知道的-3(Active Record)
date: 2017-01-05 10:06:39
tags:
---

>**Yii What you need to known**系列一共五篇，是几年前的文章。来源博客Hash Solutions，[原文地址](http://blog.hashsolutions.in/technology/yii2-need-know-part-iii-activerecord/)。
  关于Yii介绍文章其实已经很多了，但是为了锻炼一下自己的英文水平、增强自己对Yii的整体认知影响，所以准备画蛇添足，翻译完这系列文章。
  对于初学者或者想了解Yii的，我觉得**Yii What you need to known**系列应该是很不错的。


在我们探索Yii2框架系列的这一期，我们将详细介绍下这个强大框架的`ActiveRecord`和数据库方面。
我们觉得这是Yii2在增强和添加新功能方面最出色的地方之一。

## 新的`ActiveRecord`类的数据库支持

在Yii2中`ActiveRecord`支持了很多新的数据库，包括基于NoSQL的。这些数据库包括`elasticsearch`,`Redis`,`Sphinx search`,`MongoDB`.
这是一个非常好的消息，因为我们在切换数据库时，不需要触碰我们的代码，因为我们的代码都是用`ActiveRecord`和`DB`去交互的。


## 在Yii2用`ActiveRecord`来选择数据

你遇到的第一个主要变化将是`model()`调用的消亡。现在所有的`find`方法都来自于`find()`或者`findBySql()`。
用blog举个例子：`Post::model()->findAll()`被`Post::find()->all()`取代了。在从数据库选择纪录时，展现了
前所未有的灵活性，而且一旦我们习惯它，也非常直观。通过权威指南告诉我们如何使用`Active Record `从数据库选择数据的代码，我们将可以更加
深刻的感受这种变化。

```PHP
// 取回所有customers数据，并按照id排序:
$customers = Customer::find()
    ->where(['status' => Customer::STATUS_ACTIVE])
    ->orderBy('id')
    ->all();

// 返回id为1的customer数据:
$customer = Customer::find()
    ->where(['id' => 1])
    ->one();

// 返回状态为`active`的所有用户的总数:
$count = Customer::find()
    ->where(['status' => Customer::STATUS_ACTIVE])
    ->count();

// 返回以customer ID为索引的结果集:
$customers = Customer::find()->indexBy('id')->all();
// `$customers`数组是以customer的id为索引的

// 使用原生SQL语句取回所有客户:
$sql = 'SELECT * FROM customer';
$customers = Customer::findBySql($sql)->all();
```

还有一些小捷径，这里有些函数可以替换掉Yii1.1的`findByPk()`和`find()`方法

```PHP
// 返回id为1的customer数据:
$customer = Customer::findOne(1);

// 返回id为1，且`status`是`STATUS_ACTIVE`的customer数据:
$customer = Customer::findOne([
    'id' => 1,
    'status' => Customer::STATUS_ACTIVE,
]);

// 返回id为1、2或3的customer数据:
$customers = Customer::findAll([1, 2, 3]);

// 返回状态为删除的所有customer数据:
$customer = Customer::findAll([
    'status' => Customer::STATUS_DELETED,
]);
```

## 关联关系（Relations）

过去使用`relations()`，现在被返回`ActiveRecord`对象的getter取代

```PHP
class Post extends yii/db/ActiveRecord
{
   public function getComments()
   {
      return $this->hasMany('Comment', array('post_id' => 'id'));
   }
}
```

这种关系依然可以使用`$game->players`这种方式，但是现在你还可以自定义查询条件。例如：

```PHP
$comments = $post->getComments()->andWhere('approve=1')->all();
```

## 其他

这还有很多微妙的变化，会使你再次热爱上这个框架。你可能也想用数组来保存从数据库获取的数据，因此可以节省内存。
现在可以使用`asArray()`来实现：

```PHP
$posts = Post::find()->asArray()->all();
```

当获取大量数据的时候，你会期望批量返回数据，以此使你的内存使用情况控制在内存限制范围以内。你在`ActiveRecord`中依然可以使用这个技术

```PHP
// 一次返回10篇文章
foreach (Post::find()->batch(10) as $posts) {
    // `$posts`是一个数组或者是一些post对象
}
// 以此获取10篇文章，并且依次迭代它们
foreach (Post::find()->each(10) as $post) {
    // $post 是一个 `Post` 对象
}
// 批量查询和即时加载
foreach (Post::find()->with('comments')->each() as $post) {
}
```

另外，Yii2现在支持嵌套事务，因此你不必担心事务是否已在当前调用堆栈中声明。

## 总结

这绝不是Yii2在`ActiveRecord`所做改进和进步的详细列表。我们发现有些改变，相当的灵活和直接。
你可以让我们直到你的想法。对于良好的[Angular2 Yii2项目结构](http://digitalreveries.in/2016/12/angular2-yii2-project-structure/)，您可以访问链接的页面。
