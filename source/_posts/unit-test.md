---
title: 测试 yii2-codeception(1)－－单元测试
date: 2017-01-12 14:25:26
tags:
---

## 测试
1、测试主要分为以下三种类型：

+ 单元测试：验证一个独立的代码单元是否按照期望的方式运行。test
+ 功能测试：在**浏览器模拟器**中以用户视角来验证期望的场景是否发生。cest
+ 验收测试：与功能测试相同，但实际上通过**真实的Web浏览器**运行测试。cept


2、BDD 与 TDD
 
 
 **TDD(Test Driven Development)**，测试驱动开发。是测试优先的开发模式。先写测试代码，再编写具体业务逻辑代码，然后运行单元测试，如果失败，就继续修正代码～。
 如果有新的功能，就如下面的流程图一样：
 ![Test Driven Development](http://joshldavis.com/img/tdd-vs-bdd/tdd-flowchart.png)
 
 但是，TDD开发模式，可能会造成一些问题～它要求先写测试，这就要求项目需求清晰，而且程序员对整个项目架构很熟悉，能提前抽离一些借口代码。否则，项目很容易失控。
 但是当一个业务模型及其复杂、内部模块之间的相互依赖性非常强，就比较麻烦了，这会导致程序员在拆分接口和写测试代码的时候工作量非常大。
 
 **BDD(Behavior Driven Development)**,行为驱动开发。这相当于，TDD的扩展。它旨在消除TDD可能导致的问题。
 
 这里只简单的说明下，如果大家很感兴趣，建议大家再多查查资料，深入了解一下～。
 

## codeception  与 PHPUnit 之间是什么关系？

`codeception` 采用`PHPUnit`作为它运行测试的后端，所以任何`PHPUnit`测试都可以在`codeception`测试中运行。
另外，**如果安装了`codeception`就不需要再另外安装`PHPUnit`啦~**

## 全局安装codeception

1、全局安装

```bash
composer global require "codeception/codeception"
composer global require "codeception/specify=*"    //用于在单元测试中写入规范
composer global require "codeception/verify=*"   
```

安装`codeception/specify`和`codeception/verify`，可以更好的形成BDD(Behavior Driven Development)规范测试.



2、查看状态：`composer global status`

3、建立符号链接：`sudo ln -s ~/.composer/vendor/bin/codecept /usr/local/bin/codecept`



4、codecept 常用命令：

+ `codecept bootstrap`
+ `codecept build`
+ `codecept run`

也可以这样使用

+ `composer exec codecept build`
+ `composer exec codecept run`


暂时还不清楚两种方式的区别～～

##  yii-basic

>我们在yii2中尝试使用codeception，采用的版本是basic版本。

使用composer创建yii－basic项目的时候，yii2已经添加`codeception`框架了。**如果已经全局安装了`codeception`，建议移除项目中的codeception，至于为啥，你看最下面（踩过的那些坑儿－2）**

但是没有安装yii2的扩展`yiisoft/yii2-codeception`，所以使用
`composer require --prefer-dist yiisoft/yii2-codeception`命令安装此扩展。

[yii2-codeception 扩展安装](http://www.yiiframework.com/doc-2.0/ext-codeception-index.html#installation)

###  yii2下的测试目录

```
－test
    - _data
    - _output # 输出文件夹 
    - _support
    - acceptance # 验收测试代码放置的文件夹
    - bin # 放置的是Yii2测试环境下，可以执行命令文件
    - functional # 功能测试代码放置的文件夹
    - unit  # 单元测试代码放置的文件夹
    - _bootstrap.php # 初始化文件
    - acceptance.suite.yml.example  # 验收测试配置文件
    - functional.suite.yml # 功能测试配置文件
    - unit.suite.yml  # 单元测试配置文件
```

1. `acceptance.suite.yml.example`文件配置
2. `functional.suite.yml`文件配置
3. `unit.suite.yml`文件配置

###  建立测试数据库：

+ 测试数据库的配置在,`config/test_db.php`文件中：`$db['dsn'] = 'mysql:host=localhost;dbname=yii2_basic_tests';`；根据自己的情况，修改相应`host`,`dbname`即可；
+ 进入`test`目录，
+ 如果有`codeception`目录的话，需进入`codeception/bin/`目录；
+ 没有的话，直接进入`/bin`目录；
+ 这里有两个文件：`yii`和`yii.bat`。
+ 在这个目录下，执行数据库迁移命令。`php yii migrate/up`

有关数据库迁移，请看[migrations](https://easy-yii.github.io/2016/12/06/migrations/)



### 试一下命令

 在创建的yii2-basic项目中，已经有一些预先框架自带的测试用例。我们尝试运行以下命令，看一下～
 
 在项目中执行，`php -S localhost:8080`，即可通过`http://localhost:8080/web/index.php`访问了～
 
 + `codecept build`,构建测试
 + `codecept run`，运行所有可以运行的测试代码
 + `codecept run acceptance`，运行验收测试的代码
 + `codecept run functional`，运行功能测试的代码
 + `codecept run unit`，运行单元测试的代码
 
 运行验收测试的时候，会报错～`Suite 'acceptance' could not be found`，因为没有找到验收测试。
 
 
### 写个单元测试，小试身手～

1. 创建测试文件，以下命令皆可，声称`ExampleTest.php`文件：
 
 + `codecept generate:phpunit unit Example`
  或
 + `codecept generate:test unit Example`
 
 
使用`codecept generate:test unit Test`命令，创建的`Test.php`文件内容如下：
 
```php
 <?php
 class Test extends \Codeception\Test\Unit
 {
     /**
      * @var \UnitTester
      */
     protected $tester;
 
     //预定义方法，可以在每个测试方法运行前创建一个测试对象
     protected function _before()
     {
     }
     
     //预定义方法，可以在每个测试运行结束后销毁一个测试对象
     protected function _after()
     {
     }
 
     // tests
     public function testMe()
     {
 
     }
 }
```

使用`codecept generate:phpunit unit Example`命令，创建的`ExampleTest.php`文件内容如下：

```php
<?php

class ExampleTest extends \PHPUnit_Framework_TestCase
{
    protected function setUp()
    {
    }

    protected function tearDown()
    {
    }

    // tests
    public function testMe()
    {
    }
}

```

大家可以发现以下区别：
`codecept generate:test unit Test`命令创建的文件：

+ 父类是`\Codeception\Test\Unit`
+ 方法是`_before()` 和 `_after()`

而codecept generate:phpunit unit Example`命令创建的文件：

+ 父类是`\PHPUnit_Framework_TestCase`，这个是`PHPUnit`创建形式。
+ 方法是`setUp()`和 `tearDown()`

**在`codeception`中`_before()` 和 `_after()`取代了在`PHPUnit`中的`setUp()`和 `tearDown()`，
事实上，`setUp()`和 `tearDown()`是在其继承的父类`\Codeception\Test\Unit`中已经完成了。**

2. 运行单元测试，以下命令皆可

 + `codecept run unit ExampleTest`运行某一个测试文件
  或
 + `codecept run unit`运行所有的单元测试
 
 
3. 传统的单元测试：

在yii2中写一个传统的单元测试，相关的`model`部分，自己补充，这里不多说了，具体代码如下：

```PHP
<?php
namespace tests\unit;

use app\models\User;
use Codeception\Specify;

class Test extends \Codeception\Test\Unit
{

	use Specify;
    /**
     * @var \UnitTester
     */
    protected $tester;

    protected function _before()
    {
    }

    protected function _after()
    {
    }

	public function testValidation() {
		$user = new User();
		$user->username = null;
		$this->assertFalse($user->validate(['username']));

		$user->username = 'toolooooongnaaaaaaameeeeeeeeeeeeeee';
		$this->assertFalse($user->validate(['username']));
		
		$user->username = 'davert';
		$this->assertTrue($user->validate(['username']));

	}
}
```

4. 测试数据库：

在上面代码中，添加一个方法：

```PHP
public function testSavingUser() {
		$user = new User();
		$user->setAttributes(['username'=>'username','password'=>'password','email'=>'yuan@qq.com']);
		$user->save(false);
		$this->assertEquals('username',$user->getAttribute('username'));
		$this->tester->canSeeRecord('app\models\User',array('username'=>'username','password'=>'password','email'=>'yuan@qq.com'));
	}
```

如果执行失败，会在terminal中显示failure，这时说明创建用户数据有问题，要开发者自己检查相应的错误原因并改正。
数据库数据，会在每次测试结束后填充和清除～

一般情况下，我们是不能直接访问数据库的`ORM`的，这里为什么能访问呢？

是因为，框架在单元测试配置文件中已经设置好了，单元测试的配置都在`unit.suite.yml`中，默认配置如下：

```PHP
# Codeception Test Suite Configuration

# suite for unit (internal) tests.
# RUN `build` COMMAND AFTER ADDING/REMOVING MODULES.

class_name: UnitTester
modules:
    enabled:
      - Asserts
      - Yii2:
            part: [orm, email]  ＃此处配置了orm
```

 除了`canSeeRecord()`方法，在Yii2中还有其他类似的方法，如下：
 
 + `seeRecord()`
 + `dontSeeRecord()`
 + `haveRecord()`


每个框架的配置略微不一样，具体想看[官方文档单元测试－Interacting with the Framework部分](http://codeception.com/docs/05-UnitTests)

5. BBD规范测试

`BBD` 全称 `Behavior Driven Development`,中文翻译过来是行为驱动测试。前文我们也介绍过～

在写测试的时候，当应功能需求发生变化，测试也要随时相应变化，所以测试随时准备好变化。
所以测试代码要非常的易读和容易维护。如果在团队中没有记录测试的约定，那当功能发生变化或者引入新功能时，找到相应的测试代码就不容易了～
所以我们不仅要测试代码覆盖整个应用程序，也要让测试代码可读性很强。
因此，我们要引入一个独立的项目 `specify`用于单元测试的规范～

我们将之前的代码修改下，引入`Codeception\Specify`,这个微小的库可以增加更多的可读性断言；
然后使用`specify`方法，代码如下：

```PHP
public function testValidation() {
		$user = new User();


		$this->specify('username is required',function() use($user) {
			$user->username = null;
			$this->assertFalse($user->validate(['username']));
		});

		$this->specify('username is too long',function() use($user) {
			$user->username = 'toolooooongnaaaaaaameeeeeeeeeeeeeee';
			$this->assertFalse($user->validate(['username']));
		});


		$this->specify('test true assert user',function() use($user) {
			$user->username = 'davert';
			$this->assertTrue($user->validate(['username']));
		});

	}

```

+ [specify(String $description,function closure)](https://github.com/Codeception/Specify)
  - 第一个参数`description`,是用于描述你要测试的场景。这样便于维护和阅读；
  - 第二个参数是一个匿名回调函数，是测试某场景的具体逻辑代码；
  - 每个`specify`里的代码都是相互独立的、不互相影响的；`specify`使用了**深浅克隆策略**来保存对象在每个作用域中保存对象。
  - 缺点是，在使用深克隆的时候会增加内存消耗，使用浅克隆的时候会不完全隔离；
  
6. 总结

PHPUnit测试是测试套件中的一流的。每当需要编写和执行单元测试时，不需要单独安装PHPUnit，而是直接使用Codeception来执行它们。
通过集成Codeception模块，可以将一些不错的功能添加到常规单元测试中。对于大多数单元和集成测试，PHPUnit测试就足够了。它们速度快，易于维护。

这篇文章只能算`codeception`测试框架的初探，还有很多关于单元的东西没有讲太清楚～这个需要我花更多的时间去研究和探索～～
当我深入研究有新发现的时候，会跟大家在这里分享的～




## 踩过的坑儿～～

1、如果已经执行过`codecept bootstrap`，再执行此命令，换出现如下情况：

```bash
bash:basic yuan$ codecept bootstrap

Project is already initialized in '.'

```


2、我执行`codecept build` 或 `codecept run`就会出现下面的报错情况。
这是因为**已经全局安装codeception 2.0，然后项目里安装了codeception 2.1＋。**
**解决办法：** 我用命令`composer remove codeception/codeception`把项目里的codeception移除之后，就正常了。
[我是在这里发现的，大家有兴趣也可以看看～～](https://github.com/yiisoft/yii2-codeception/issues/5)
```bash
bash:basic yuan$ codecept build
Building Actor classes for suites: functional, unit
PHP Catchable fatal error:  Argument 1 passed to Codeception\Module::__construct() must be an instance of Codeception\Lib\ModuleContainer, array given, called in /Users/yuan/.composer/vendor/codeception/codeception/src/Codeception/Configuration.php on line 327 and defined in /Users/yuan/PhpstormProjects/basic/vendor/codeception/base/src/Codeception/Module.php on line 70
PHP Stack trace:
PHP   1. {main}() /Users/yuan/.composer/vendor/codeception/codeception/codecept:0
PHP   2. Symfony\Component\Console\Application->run() /Users/yuan/.composer/vendor/codeception/codeception/codecept:27
PHP   3. Symfony\Component\Console\Application->doRun() /Users/yuan/.composer/vendor/symfony/console/Application.php:126
PHP   4. Symfony\Component\Console\Application->doRunCommand() /Users/yuan/.composer/vendor/symfony/console/Application.php:195
PHP   5. Symfony\Component\Console\Command\Command->run() /Users/yuan/.composer/vendor/symfony/console/Application.php:878
PHP   6. Codeception\Command\Build->execute() /Users/yuan/.composer/vendor/symfony/console/Command/Command.php:259
PHP   7. Codeception\Command\Build->buildActorsForConfig() /Users/yuan/.composer/vendor/codeception/codeception/src/Codeception/Command/Build.php:46
PHP   8. Codeception\Lib\Generator\Actor->__construct() /Users/yuan/.composer/vendor/codeception/codeception/src/Codeception/Command/Build.php:67
PHP   9. Codeception\Configuration::modules() /Users/yuan/PhpstormProjects/basic/vendor/codeception/base/src/Codeception/Lib/Generator/Actor.php:44
PHP  10. Codeception\Configuration::createModule() /Users/yuan/.composer/vendor/codeception/codeception/src/Codeception/Configuration.php:287
PHP  11. Codeception\Module->__construct() /Users/yuan/.composer/vendor/codeception/codeception/src/Codeception/Configuration.php:327

Catchable fatal error: Argument 1 passed to Codeception\Module::__construct() must be an instance of Codeception\Lib\ModuleContainer, array given, called in /Users/yuan/.composer/vendor/codeception/codeception/src/Codeception/Configuration.php on line 327 and defined in /Users/yuan/PhpstormProjects/basic/vendor/codeception/base/src/Codeception/Module.php on line 70

Call Stack:
    0.0002     238464   1. {main}() /Users/yuan/.composer/vendor/codeception/codeception/codecept:0
    0.0171    2700328   2. Symfony\Component\Console\Application->run() /Users/yuan/.composer/vendor/codeception/codeception/codecept:27
    0.0198    3058600   3. Symfony\Component\Console\Application->doRun() /Users/yuan/.composer/vendor/symfony/console/Application.php:126
    0.0200    3059520   4. Symfony\Component\Console\Application->doRunCommand() /Users/yuan/.composer/vendor/symfony/console/Application.php:195
    0.0200    3060024   5. Symfony\Component\Console\Command\Command->run() /Users/yuan/.composer/vendor/symfony/console/Application.php:878
    0.0202    3063904   6. Codeception\Command\Build->execute() /Users/yuan/.composer/vendor/symfony/console/Command/Command.php:259
    0.0202    3064536   7. Codeception\Command\Build->buildActorsForConfig() /Users/yuan/.composer/vendor/codeception/codeception/src/Codeception/Command/Build.php:46
    0.0363    4901592   8. Codeception\Lib\Generator\Actor->__construct() /Users/yuan/.composer/vendor/codeception/codeception/src/Codeception/Command/Build.php:67
    0.0372    5024000   9. Codeception\Configuration::modules() /Users/yuan/PhpstormProjects/basic/vendor/codeception/base/src/Codeception/Lib/Generator/Actor.php:44
    0.0372    5024720  10. Codeception\Configuration::createModule() /Users/yuan/.composer/vendor/codeception/codeception/src/Codeception/Configuration.php:287
    0.0382    5226888  11. Codeception\Module->__construct() /Users/yuan/.composer/vendor/codeception/codeception/src/Codeception/Configuration.php:327

```


如果大家有什么问题，或者本文有错误，或者有什么好资源共享，欢迎在留言或者[在这里提issue](https://github.com/easy-yii/easy-yii.github.io/issues/1)

参考文章：

+ [一个系列，yii2-codeception，主要是作者的心得，算不上教程](http://pjokumsen.co.za/category/yii2/)
+ [Programming With Yii2: Automated Testing With Codeception](https://code.tutsplus.com/tutorials/programming-with-yii2-automated-testing-with-codeception--cms-26790)
+ [codeception for yii2](http://codeception.com/for/yii)
+ [codeception文档－unit](http://codeception.com/docs/05-UnitTests)
+ [Yii2文档－测试部分](http://www.yiichina.com/doc/guide/2.0/test-overview)


