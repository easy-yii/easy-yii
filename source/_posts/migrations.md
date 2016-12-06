---
title: migrations
date: 2016-12-06 10:21:33
tags:
---
> 前言：这是一个yii2专题，从如何使用yii2到分析yii2源码，到最后思考如何设计一个框架。本人会致力于写的很详细。(PS:希望我能坚持下去，不会打脸:see_no_evil:)
本文主要讲述了，如何使用迁移，建立数据库，并涉及到一点儿关于设计数据库的思考。本文的结构，`steps`部分，带领大家一步步尝试；总结部分，在走完之后，做个小结，如常用命令List，如一些思考。
所以本文既可以当成教程，也可以当成一个查看手册，比如忘记某个命令，可以直接跳到总结那部分查找。


## steps

### step one 数据库配置

本文采用的是yii2-advanced版本，在common/config/main-local.php文件配置。

```
<?php
return [
    'components' => [
        'db' => [
            'class' => 'yii\db\Connection',
            'dsn' => 'mysql:host=localhost;dbname=数据库名字',
            'username' => 'mysql访问用户',
            'password' => '密码',
            'charset' => 'utf8',
        ],
    ],
];
```

### step two 创建迁移文件

**注意，本文使用的命令行，带有php，因为本人并未将`yii`设置为command**

执行 `php yii migrate/create post` 后，终端会输出一则消息询问你，是否创建该文件，输入 `yes` 即可。如下：

```
Yii Migration Tool (based on Yii v2.0.9)
Create new migration '/Users/yuan/PhpstormProjects/chg/console/migrations/m161023_120328_post.php'? (yes|no) [no]:yes
New migration created successfully.
```
创建的文件在`console/migrations`目录下，文件名`m161023_120328_post.php`

### step three 建立表结构

2.06以后的版本，提供了更简便创建列的方法。我们将各种类型的列试一下。详细可参照[yii\db\SchemaBuilderTrait](http://www.yiiframework.com/doc-2.0/yii-db-schemabuildertrait.html)

```
public function up()
    {
	    $this->createTable('posts',[
		    'id' => $this->primaryKey(),
		    'big_integer' => $this->bigInteger(),
		    'binary' => $this->binary(),
		    'boolean' => $this->boolean(),
		    'char' => $this->char(),
		    'date' => $this->date(),
		    'date_time' => $this->dateTime(),
		    'decimal' => $this->decimal(),
		    'double' => $this->double(),
		    'float' => $this->float(),
		    'integer' => $this->integer()->notNull()->defaultValue(1),
		    'money' => $this->money(),
		    'small_integer' => $this->smallInteger(),
		    'string' => $this->string(64),
		    'text' => $this->text(),
		    'time' => $this->time(),
		    'timestamp' => $this->timestamp()
	    ]);
    }

    public function down()
    {
        $this->dropTable('posts');
    }
```

`up`方法，创建表结构；`down`方法，用于回滚`up`方法中的操作。最后，再详细说下每种列的类型。

### step four 迁移

执行`php yii migrate`即可，此时可以看一下数据库对应列的类型

### step five 添加列、外键、索引，删除列、外键、索引

在刚刚建的`posts`表中添加新的列，并添加外键、索引。

创建执行命令`php yii migrate/create update_posts`,创建了一个新的文件`m161023_132017_update_posts.php`。

在文件中添加代码：

```
public function up()
    {
	    $this->addColumn('posts', 'author_id', $this->integer());
	    $this->addColumn('posts', 'category_id', $this->integer());
	    $this->addColumn('posts', 'title', $this->string());
	    $this->addColumn('posts', 'body', $this->text());

	    $this->createIndex(
		    'idx-posts-author_id',
		    'posts',
		    'author_id'
	    );

	    $this->addForeignKey(
		    'fk-posts-author_id',
		    'posts',
		    'author_id',
		    'user',
		    'id',
		    'CASCADE'
	    );
    }

    public function down()
    {
	    $this->dropForeignKey(
		    'fk-posts-author_id',
		    'posts'
	    );

	    $this->dropIndex(
		    'idx-posts-author_id',
		    'posts'
	    );
	    $this->dropColumn('posts', 'author_id');
	    $this->dropColumn('posts', 'category_id');
	    $this->dropColumn('posts', 'title');
	    $this->dropColumn('posts', 'body');
    }
```

注意:在`up`方法中，先创建索引，再创建外键；在`down`方法中，删除外键，再删除索引。

+ 创建索引的方法:$this->createIndex('idx-表名-字段','表名','字段');[方法详情](http://www.yiiframework.com/doc-2.0/yii-db-command.html#createIndex()-detail)
+ 创建外键的方法:$this->addForeignKey('fk-表名-字段','表名','字段','关联的表名','关联的表明中的字段', 'CASCADE');[方法详情](http://www.yiiframework.com/doc-2.0/yii-db-command.html#addForeignKey()-detail)

**想进一步学习，延伸外键、索引方面的知识，可以看底部相关链接。**


### step six 事务

当需要保证每一步对数据库的操作都要成功时，如果中间有任何一步对数据库的操作失败，我们都希望撤销之前对数据库的操作。
我们这个时候即需要用到事务，它可以保证数据库的完整性和一致性。

在yii2中，如何实现呢，非常简单。只需将`save()`,`down()`方法分别替换成`safeUp()`,`safeDown()`即可。

### step seven 恢复迁移
    
恢复迁移的意思是，撤销对数据库的操作。可以执行以下命令，尝试看看。
    
+ 还原最近一次提交的迁移: `php yii migrate/down`
+ 还原最近三次提交的迁移: `php yii migrate/down 3`

还有相关重做迁移、查看迁移历史、修改迁移历史，这里不多说，看一下总结部分－迁移命令，并尝试一下。

### step eight 使用命令行选项

#### `interactive`

默认为true。每次执行migrate之前，会提示用户，输入yes or no，如果设置为false，则可省去这些提示。
使用如：

+ `php yii migrate --interactive=0` 
+ `php yii migrate/down 1 --interactive=0`

#### migrationPath

默认值是，@app/migrations。是存放迁移文件的目录。如果我们使用模块(modules)开发,一般每个模块也会有存放迁移文件的目录。
当迁移模块里的迁移文件时，我们就需要指定迁移路径。[关于模块更多的了解，请点击此](http://www.yiichina.com/doc/guide/2.0/structure-modules)
使用如：`yii migrate --migrationPath=@app/modules/forum/migrations`

#### db

string (默认值为 db)，指定数据库 application component 的 ID。 它指的是将会被该命令迁移的数据库。

在配置文件中，配置一个db2，如何配置看step one。使用`yii migrate --db=db2`，即可迁移到id为db2的数据库。

也可以这样,在代码中指定数据库的id，虽然以下的迁移会迁移到id为db2的数据库中，但是迁移历史信息会放在id为db的数据库中。
如果很多迁移文件，到要迁移到id为db2的数据库中，可以将`init()`部分代码封装成一个基类，让要迁移到id为db2的数据库的迁移
文件继承这个基类就可以了。

```
<?php

use yii\db\Migration;

class m150101_185401_create_news_table extends Migration
{
    public function init()
    {
        $this->db = 'db2';
        parent::init();
    }
}
```

#### templateFile

默认的模版文件是@yii/views/migration.php，文件位置在`vendor/yiisoft/yii2/views/migration.php`。

该选项的值，可以是文件路径，也可以是路径别名。

#### generatorTemplateFiles

模版生成器，默认值是下面数组。数组中的健即是，上一小结的别名。以下文件位置都在`vendor/yiisoft/yii2/views/`目录下。
如何创建一个新的模版，可能要修改框架源码。有兴趣的可详细研究`yii\console\controllers\MigrateController`源码。

```
array(
 'create_table' => '@yii/views/createTableMigration.php',
  'drop_table' => '@yii/views/dropTableMigration.php',
  'add_column' => '@yii/views/addColumnMigration.php',
  'drop_column' => '@yii/views/dropColumnMigration.php',
  'create_junction' => '@yii/views/createJunctionMigration.php'
);
```

#### fields

在创建迁移文件时，就声明字段和字段类型。创建完成后，迁移文件会包括这些已声明的字段。

`yii migrate/create create_post --fields="title:string,body:text"`

## 总结

###  Yii字段类型  VS 数据库类型

在 step three 中，我们将Yii2提供的创建不同数据库字段类型的方法都试了一下，现在我们简单总结并分析下。

| Yii2               |  数据库Type |  Length |      说明     |
|--------------------|------------|---------|--------------|
| primaryKey()|   INT      |    11   | Extra:auto_increment；主键；一般表中都会有一个自增长id，默认为主键。|
| bigInteger()|   BIGINT   |    20   |                |
| binary()|   BLOB   |       |   二进制             |
| boolean()|   TINYINT   |  1     |   布尔类型       |
| char()|   CHAR   |  1     |       存定长，不可变，速度快，存在浪费空间的可能 ，会处理尾部空格，上限255        |
| date()|   DATE   |      | 日期，YYYY-MM-DD               |
| dateTime()|  DATETIME   |      | 日期和时间，YYYY-MM-DD  HH:MM:SS |
| decimal()|  DECIMAL   |    10,0  | 默认(10,0),decimal(p,s)，p整数位，s小数位；在金融行业，设计数据库的时候，涉及到金额、利率经常使用此类型|
| float()| FLOAT  |    10,0  |                |
| integer()| INT  |    11  |                |
| money()| DECIMAL  |    19,4  | money，其实本质还是decimal类型，不同在于精读，整数位19，小数位4。|
| smallInteger()| SMALLINT  |    6  |                |
| string()| VARCHAR  |    255  |   存变长，速度慢，不存在空间浪费，不处理尾部空格，上限65535，但是有存储长度实际65532最大可用。             |
| text()| TEXT  |      | 存变长大数据，速度慢，不存在空间浪费，不处理尾部空格，上限65535，会用额外空间存放数据长度，顾可以全部使用65535。               |
| timestamp()| TIMESTAMP  |      | 默认值：CURRENT_TIMESTAMP;Extra:on update CURRENT_TIMESTAMP；时间戳，Unix纪元，1970-01-01 00:00:00 UTC时间 最大到：2038-01-09 03:14:07 UTC          |


#### Attention 关于数据库设计的一点思考

+ [MySQL之char、varchar和text的设计](http://www.cnblogs.com/billyxp/p/3548540.html)
+ [时间存时间戳还是格式化的时间？datetime、timestamp、还是bigint？](https://segmentfault.com/q/1010000000655428)


### 数据库访问方法

详情看[API](http://www.yiiframework.com/doc-2.0/yii-db-migration.html)

+ `yii\db\Migration::execute($sql,$params = [])` 执行一条 SQL语句 
+ `yii\db\Migration::insert($table, $columns )` 插入单行数据 
+ `yii\db\Migration::batchInsert($table, $columns, $rows)` 插入多行数据 
+ `yii\db\Migration::update($table, $columns, $condition = '', $params = [] )` 更新数据
+ `yii\db\Migration::delete($table, $condition = '', $params = [])` 删除数据  
+ `yii\db\Migration::createTable()` 创建一张表
+ `yii\db\Migration::renameTable()` 重命名表明
+ `yii\db\Migration::dropTable()` 删除一张表
+ `yii\db\Migration::truncateTable()` 清除一张表的数据
+ `yii\db\Migration::addColumn()` 添加一列
+ `yii\db\Migration::renameColumn()` 重命名字段名称
+ `yii\db\Migration::dropColumn()` 移出列
+ `yii\db\Migration::alterColumn()` 修改字段
+ `yii\db\Migration::addPrimaryKey()` 添加一个主键
+ `yii\db\Migration::dropPrimaryKey()` 移除一个主键
+ `yii\db\Migration::addForeignKey()` 添加一个外键
+ `yii\db\Migration::dropForeignKey()` 移除外键
+ `yii\db\Migration::createIndex()` 创建一个索引
+ `yii\db\Migration::dropIndex()` 移除一个索引
+ `yii\db\Migration::addCommentOnColumn()` 添加某个字段的注释
+ `yii\db\Migration::dropCommentFromColumn()` 移除某个字段的注释
+ `yii\db\Migration::addCommentOnTable()` 添加某张表的注释
+ `yii\db\Migration::dropCommentFromTable()` 移除某张表的注释


部分示例：

```
$this->execute("insert into news (title,content) VALUES ('insert','insert');"); //暂时不清楚，$params到底要传什么？

$this->insert('news', ['title' => 'test 1','content' => 'content 1']);

$this->batchInsert('news',['title','content'],[0=>['insert1','content1'],1=>['insert2','content2']]);

$this->delete('news', ['id' => 1]);

$this->createTable('news', ['id' => $this->primaryKey(),'title' => $this->string()->notNull(),'content' => $this->text(),]);
	    
$this->truncateTable('news');

$this->dropTable('news');

$this->dropForeignKey('fk-posts-author_id','posts');
  
 $this->addForeignKey('fk-posts-author_id','posts','author_id','user','id','CASCADE');
        	    
$this->dropIndex('idx-posts-author_id','posts');

$this->createIndex('idx-posts-author_id','posts','author_id');
        	    
$this->dropColumn('posts', 'body');
```


### 迁移命令

当你忘记迁移命令时，你只需在控制台执行，`php yii help migrate`，即可看到相关migrate命令，可以省去查看文档的时间。

+ 创建迁移(`name`参数应当只包含字母、下划线、数字): `php yii migrate/create name`
+ 执行迁移: `php yii migrate` 即 `php yii migrate/up`
+ 还原最近一次提交的迁移: `php yii migrate/down`
+ 还原最近三次提交的迁移: `php yii migrate/down 3`
+ 重做最近一次提交的迁移: `yii migrate/redo` 
+ 重做最近三次提交的迁移:  `yii migrate/redo 3` 
+ 显示最近10次提交的迁移: `yii migrate/history`
+ 显示最近5次提交的迁移: `yii migrate/history 5`
+ 显示所有已经提交过的迁移: `yii migrate/history all`
+ 显示前10个还未提交的迁移: `yii migrate/new`        
+ 显示前5个还未提交的迁移: `yii migrate/new 5`   
+ 显示所有还未提交的迁移:  `yii migrate/new all` 


### 思考

Question One：在个人看来，使用Migration将数据库也纳入版本管理，可以更好方便的管理数据库并纪录数据库的变化，纪录数据库在哪个
时间点创建了数据库，修改了数据库。甚至可以通过版本工具，如`Git`，知道哪位小组成员改了数据库。但，本人碰到这么一个观点。这个
观点是，使用迁移工具不安全。会造成员工可能因为个人情绪胡乱修改数据库或者删除数据库。个人觉得，在一般开发中，起码有三个环境：开发环境、测试环境、线上环境。
在上线之前，是肯定要在测试环境测试过的。一般配置文件，如数据库配置文件，可以设置默认不上传。而上线，又是一般有专门的人负责部署。团队可能会因为大小，
部分成员也要负责部署。并不会随便让什么人修改数据库。不知道大家对于使用Migration对安全的威胁有什么看法？


>后记，migration只是帮助我们更好的管理数据库。它的核心还是如何设计数据库，建议刚开始学习PHP的童鞋们，补充一下关于数据库的知识。如，
基本的sql语法、表之间的关联关系、事务、悲观锁、数据库引擎等。


## 延伸:

+ [mysql 外键](http://blog.51yip.com/mysql/1136.html)
+ [mysql 索引](https://segmentfault.com/a/1190000003072424)
+ [mysql 存储引擎的选择](https://github.com/YuanLianDu/YLD-with-Php/blob/master/articles/mysql/engine.md)
+ [高级版本和基础版本的区别](https://github.com/yiisoft/yii2-app-advanced/blob/master/docs/guide/start-comparison.md)
