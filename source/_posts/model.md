---
title: model
date: 2016-12-06 10:23:04
tags:
-----

## 前言

MVC设计模式是一个经典的设计模式。大多数框架都已mvc为基本架构。Model主要负责处理数据，可以理解为与数据库打交道。
Controller，主要处理view传来的请求，向model请求数据，并将请求的数据传给view。View,主要将用户所需的请求传给控制器、接收controller传来的数据，并在页面内渲染。
按照本人的理解，M负责数据，C负责业务逻辑，V负责页面交互。

这一节内容，通过一个简单的场景(博客)来主要梳理yii2框架的模型，重点说CURD操作(即create、update、read、delete)和关联模型，并会简单的说下控制器。

本文依旧先梳理思路，掌握整体结构，再一个个梳理知识点，建议结合文档看，list部分重点列举了用法，部分解释不清楚的参考文档。

+ [查询构建器](http://www.yiichina.com/doc/guide/2.0/db-query-builder)
+ [活动纪录](http://www.yiichina.com/doc/guide/2.0/db-active-record)

## One:Steps

### 场景

一个博客需要作者和文章内容。作者可以在后台管理自己的文章内容(增删改查)，用户可以在前台看到作者的文章和并对文章评论。

###  step one tables

简单的分析以下需求，我们建立的数据库如下。大家可利用上一节学习的内容，使用迁移来建立自己的数据库。记得修改相关数据库配置。

**用户表user**

| Columns|  Type|  Length |  Allow Null   |  Default | Comment |
|-------|-------|---------|--------------|----------|---------|
| id    |    int   |    11      |     no        |            |    用户id     |
| username   |  varchar     |    255      |     no        |            |  用户名       |
| password   |   varchar    |    255      |         no    |            |       密码  |
| group   |   int    |    3      |       no      |      1      |  分组       |
| email   |    varchar   |     255     |   no          |            | 邮箱        |
| created_at  |  bigint     |    20      |   no          |            |  创建时间       |
| updated_at |   bigint    |      20    |      yes       |            |  更新时间       |


**文章表post** 

| Columns|  Type|  Length |  Allow Nul   |  Default | Comment |
|-------|-------|---------|--------------|----------|---------|
| id    |      int |     11     |     no        |            |     文章id    |
| author_id   |   int    |    11      |   no          |            |   作者id      |
| title   |  varchar    |     255     |     no        |            |   文章标题      |
| body   |    text   |          |        no     |            |    文章内容     |
| created_at  |   bigint    |   20       |   no          |            |  创建时间       |
| updated_at |   bigint    |      20    |      yes       |            |    更新时间     |
| deleted_at |   bigint    |      20    |       yes      |            |    删除时间     |


**评论表comment**

| Columns|  Type|  Length |  Allow Nul   |  Default | Comment |
|-------|-------|---------|--------------|----------|---------|
| id    |     int  |    11      |     no        |            |    评论id     |
| title_id   |   int    |    11      |    no         |            |  评论的文章id       |
| user_id  |    int   |    11      |     no        |            |    用户id     |
| content   |  text    |          |       no      |            |    内容     |
| created_at  |  bigint     | 20         |   no          |            |   创建时间      |
| updated_at |   bigint    |    20      |      yes       |            |    更新时间     |
| deleted_at |    bigint   |     20     |       yes      |            |    删除时间     |


###  steps two 建立model

yii有很多类型的model，我们简单的了解下。

`yii\base\Model`,是基本模型,一般情况下会用来处理用户模型操作。它有很多子类。
比如`yii\db\ActiveRecord`,这也是我们经常用的高级模型类，进行相关数据库操作，当然它也具备`yii\base\Model`所有功能。大多数情况我们主要使用此高级模型类。

想了解更多，可以点击连接。[yii\base\Model-api](http://www.yiiframework.com/doc-2.0/yii-base-model.html)


`bankend`和`frontend`都可以继承`common\models\Post.php`，并把公共部分代码放置于这个文件。具体代码如下:

```
<?php

namespace common\models;

use yii\db\ActiveRecord;
use yii\behaviors\TimestampBehavior;

class Post extends ActiveRecord
{

//关联表名
	public static function tableName()
	{
		return 'post';
	}

//事件，创建更新，可自动对时间赋值，默认字段是，created_at updated_at
	public function behaviors() {
		return [
			TimestampBehavior::className(),
		];
	}


}
```

`backend\models\Post`继承`common\models\Post`

```
<?php
namespace backend\models;

use Yii;
use common\models\Post as BasePost;

class Post extends BasePost
{

	public function rules()
	{
		return [
			[['author_id', 'title', 'body'], 'required'],
		];
	}

	public function __construct()
	{
		$post = Yii::$app->request->post();//获取post请求传来的参数
		$this->load(['Post' => $post]);//通过load为字段赋值
	}

	/**
	 * @return bool
	 * 创建一篇文章
	 */
	public function postCreate()
	{
		/*$post = new self();
		$post->author_id = $post['author_id'];
		$post->title = $post['title'];
		$post->body = $post['body'];*/
		return $this->save();
	}

	/**
	 * @param $id
	 * @return bool
	 * 更新某篇文章
	 */
	public function postUpdate($id)
	{

		$post = self::findPostById($id);
		foreach ($this->attributes as $key => $value) {
			$post->$key = $value ? :$post->$key;
		}

		return $post->save();
	}

	/**
	 * @param $id
	 * @return bool
	 * 删除某篇文章
	 */
	public function postDelete($id)
	{
		$post = self::findPostById($id);
		//$post->delete();
		$post->deleted_at = time();
		return $post->save();
 	}

	/**
	 * @param $id
	 * @return static
	 * 根据id返回某篇文章
	 */
	public static function findPostById($id)
	{
		return Post::findOne(['id' => $id]);
	}

	/**
	 * @return array|\yii\db\ActiveRecord[]
	 * 返回未删除的文章并倒序排列
	 */
	public static function postList()
	{
		return self::find()->where(['deleted_at'=>null])->orderBy('id desc')->asArray()->all();
	}
}
```
### steps three 建立controller

```
<?php
/**
 * Created by PhpStorm.
 * User: yuan
 * Date: 16/10/31
 * Time: 14:54
 */

namespace backend\controllers;

use Yii;
use yii\base\Controller;
use backend\models\Post;
use yii\db\Query;

class PostController extends Controller
{


	public function actionIndex()
	{
		var_dump(Post::postList());
	}

	public function actionView() {
		$id = Yii::$app->request->get('id');
		var_dump(Post::findPostById($id)->attributes);die();
	}
	public function actionCreate()
	{
		$post = new Post();
		var_dump($post->postCreate());
	}

	public function actionUpdate()
	{
		$id = Yii::$app->request->get('id');
		$post = new Post();
		var_dump($post->postUpdate($id));
	}


	public function actionDelete()
	{
        $id =  Yii::$app->request->get('id');
        var_dump(Post::postDelete($id));
	}
}
```

### steps four 调试

推荐postman调试接口

## Two: List

###  关联表

### create

### update

### delete

### find

+ 返回所有数据`$post = Post::find()->all();`
+ 返回一条author_id的数据：`$post = Post::find()->where(['author_id'=>1])->one();`
+ 计算author_id为1的纪录条数总和：`$post = Post::find()->where(['author_id'=>1])->count();`
+ 以author_id为索引结果集，出现多条author_id相同的只会保留其中一条纪录返回：`$post = Post::find()->indexBy('author_id')->all();`
+ 用原生sql检索：`$sql = 'select * from post';$post = Post::findBySql($sql)->asArray()->all();`
+ 用原生sql检索：`$sql = 'select * from post';$post = Post::findBySql($sql)->asArray()->one();`
+ 返回一条id为1的纪录：`$post = Post::findOne(1);`
+ 返回一条id为1且author_id为1的纪录：`$post = Post::findOne(['id'=>1,'author_id'=>1]);`
+ 返回id为1、2、3的所有纪录，不能与`asArray()`组合：`$post = Post::findAll([1,2,3]);`
+ 返回author_id为1的所有纪录，不能与`asArray()`组合：`$post = Post::findAll(['author_id'=>1]);`
    
#### select

+ 返回所有纪录，但只返回id、author_id、title、bod字段：`$post = Post::find()->select(['id','author_id', 'title', 'body'])->all();`
+ 指定别名:
+ addSelect:

#### asArray
以数组形式返回：
`$post = Post::find()->select(['id','author_id', 'title', 'body'])->asArray()->all();`

#### distinct

以数组形式返回所有的author_id，并去除重复的author_id：`$post = Post::find()->select('author_id')->distinct()->asArray()->all()`

#### where

操作where的三种格式：

+ 字符串格式，例如：'status=1': 
    - `$post = Post::find()->select(['id','author_id', 'title', 'body'])->where('author_id = 1')->asArray()->all();`
    - `$post = Post::find()->select(['id','author_id', 'title', 'body'])->where('author_id = :user_id',[':user_id'=>1])->asArray()->all();`
+ 哈希格式，例如： ['status' => 1, 'type' => 2]: `$post = Post::find()->select(['id','author_id', 'title', 'body'])->where(['author_id'=>1,'title'=>'title1'])->asArray()->all();`
+ 操作符格式，例如：['like', 'name', 'test']，如下：

**and**
```
$post = Post::find()->select(['id','author_id', 'title', 'body'])
			->where(['and', 'author_id=1', ['or', "title='title1'", "title='title3'"]])
			->asArray()->all();
```
**or**
```
$post = Post::find()->select(['id','author_id', 'title', 'body'])
			->where(['or', "title='title1'", "title='title3'"])
			->asArray()->all();
```

**between**
```
$post = Post::find()->select(['id','author_id', 'title', 'body'])
			->where(['between','author_id',1,8])
			->asArray()->all();
```

**not between**
```
$post = Post::find()->select(['id','author_id', 'title', 'body'])
			->where(['not between','author_id',1,8])
			->asArray()->all();
```

**in**
```
$post = Post::find()->select(['id','author_id', 'title', 'body'])
			->where(['in','id',[1,2,3]])
			->asArray()->all();
或
$post = Post::find()->select(['id','author_id', 'title', 'body'])
			->where(['id' => [1,2,3,4,5,6]])
			->asArray()->all();
```
**not in**
````
$post = Post::find()->select(['id','author_id', 'title', 'body'])
			->where(['not in','id',[1,2,3,4,5,6,7,8]])
			->asArray()->all();
````

**like**:模糊查询
```
//返回字段body中有`body`和`update`的所有纪录
$post = Post::find()->select(['id','author_id', 'title', 'body'])
			->where(['like','body',['body','update']])
			->asArray()->all();
			
或

//返回字段body中有`body`的所有纪录
$post = Post::find()->select(['id','author_id', 'title', 'body'])
			->where(['like','body','body',])
			->asArray()->all();
```
**or like**
```
//返回字段title有`1`或`2`的所有纪录
$post = Post::find()->select(['id','author_id', 'title', 'body'])
			->where(['or like','title',['1','3']])
			->asArray()->all();
```
**not like**
```
//不要返回字段title有`1`或`2`的所有纪录
$post = Post::find()->select(['id','author_id', 'title', 'body'])
			->where(['not like','title',['1','2']])
			->asArray()->all();
```
**or not like**
```
//不要返回字段title中有9和title的所有纪录
$post = Post::find()->select(['id','author_id', 'title', 'body'])
			->where(['or not like','title',['9','title']])
			->asArray()->all();
```
**>,>=,<,<=**
```
//author_id小于3的所有纪录
$post = Post::find()->select(['id','author_id', 'title', 'body'])
			->where(['<','author_id',3])
			->asArray()->all();

//author_id大于等于3的所有纪录
$post = Post::find()->select(['id','author_id', 'title', 'body'])
			->where(['>=','author_id',3])
			->asArray()->all();
```

**exists**

```
//如果存在author_id 为2的数据，返回所有的纪录
$query = Post::find()->where(['author_id'=>2]);
$post = Post::find()->select(['id','author_id', 'title', 'body'])
			->where( ['exists',$query])
			->asArray()->all();
```

**not exists**

```
//如果不存在author_id 为2的数据，返回所有的纪录
$query = Post::find()->where(['author_id'=>2]);
$post = Post::find()->select(['id','author_id', 'title', 'body'])
			->where( ['exists',$query])
			->asArray()->all();

```

#### orderBy:排序

`$post = Post::find()->orderBy(['id' => SORT_ASC, 'author_id' => SORT_DESC,])->all();`
`$post = Post::find()->orderBy('id ASC, author_id DESC')->all();`
`$post = Post::find()->orderBy('id ASC')->addOrderBy('author_id DESC')->all();`

#### having

```
//查找body字段中有update的纪录，并计算所有该作者的文章总和
$post = Post::find()
		    ->select(['COUNT(*) AS count','author_id','body'])
			->having(['like','body','update'])
			->groupBy('author_id')
			->asArray()
			->all();
```
#### groupBy

```
//按照author_id分组并计算每组总和,只打印计数总和、author_id
$post = Post::find()
		    ->select(['COUNT(*) AS count','author_id'])
			->groupBy('author_id')
			->asArray()
			->all();					

```

Question:
`$post = Post::find()->groupBy('author_id')->asArray()->all();`，打印结果没有分组效果？？？？？？


判断id为2的纪录是否存在，存在返回true，不存在返回false：
`$post = Post::find()->where( [ 'id' => 2 ] )->exists();`

#### addXXX()

addXXX()，这种写法，一般作为条件补充。用法不多说，同上。

+ addHaving()
+ addSelect()
+ addParams()
+ addOrderBy()
+ addGroupBy()

#### filterWhere

过滤用户输入的空值,比如用户登陆时输入的用户名或者email，如果值为空，则默认不作为查询条件。

```
User::find()->filterWhere([
    'username' => $username,
    'email' => $email,		
]);
```

#### andFilterCompare('column_name','value','$defaultOperator = '=' ')

为特定列添加过滤条件，并允许用户选择过滤器运算符.如`<`,`>`,`>=`,`<=`,`<>`,`=`

```
$post = Post::find()
	->andFilterCompare('author_id','4','<')
	->asArray()
	->all();
```

#### limit  offset

limit 限制返回的条数，可以用空值或者负值禁用。

offset 比如返回10条数据，偏移量为3，则会返回后面7条数据.

```
$post = Post::find()
    ->select(['id','author_id','body'])
	->andFilterCompare('author_id','4','<')
	->limit(3)
	->offset(2)
	->asArray()
	->all();
```

### join

**join()**

```
//第一个参数，可为'left join' ,'right join','inner join'
$post = (new Query())->from('post')
    ->join('right join','user','user.id = post.author_id ')
	->all();
```

**innerJoin()**

`post` 为左表，`user`为右表。只有当左表和右表有对应的数据时，才会显示这条数据。比如，`post`必须有`author_id=1`且`user`有`id＝1`的数据，才会返回。

**leftJoin()**

```
$post = (new Query())->from('post')
	->innerJoin('user','user.id = post.author_id ')
	->all();
```

`post` 为左表，`user`为右表。会以左表为中心，即`post`，会全部加载左表的内容，如果右表没有匹配项，则右表相应字段值显示为null。

```
$post = (new Query())->from('post')
	->leftJoin('user','user.id = post.author_id ')
	->all();
```

**rightJoin()**

`post` 为左表，`user`为右表。会以右表为中心，即`user`，会全部加载右表的内容，如果左表没有匹配项，则左表相应字段值显示为null。

```
$post = (new Query())->from('post')
	->rightJoin('user','user.id = post.author_id ')
	->all();
```

**union()**

UNION 常用于**数据类似**的两张或多张表查询，如不同的数据分类表，或者是数据历史表等。

```
$query_one = (new Query())->from('post')
    ->andFilterCompare('author_id','4','<=');

$query_two = (new Query())->from('user')
    ->where(['id'=>1]);
var_dump($query_two->union($query_one)->all());
		
//打印的结构是user表的结构，大家可以尝试一下，如果两张表结构差距很大，结果会很有趣		
```

### 批处理

` yii\db\Query::all() `会把所有数据全部读出来，放到内存中。所以需要处理大数据时，不太适合。为了保持较低的内存占有，使用`batch`和`each`。

```
//each
$post = Post::find();
foreach($post->each() as $key=>$value) {
	var_dump($value->author_id);
}

//batch
$post = Post::find();
foreach($post->batch() as $posts) {
    //posts是一个包含100条或小于100条数据的数组
	foreach($posts as $key => $value) {
	var_dump($value->author_id);
	}
}
```


## Source Code Analysis 源码分析

### load()


> 后记，
下一章内容：
>
+ rules
+ scenario
+ validate
