---
title: dataProvider
date: 2017-02-15 13:56:07
tags:
---

`dataProvider`类，封装了排序(sort)和分页(pagination)功能；

1、正常使用：

```php
use yii\data\ActiveDataProvider;

$query = Post::find()->where(['status' => 1]);
$provider = new ActiveDataProvider([
    'query' => $query, // 或 $query->asArray()
    'pagination' => [
        'pageSize' => 10,
    ],
    'sort' => [
        'defaultOrder' => [
            'created_at' => SORT_DESC,
            'title' => SORT_ASC, 
        ]
    ],
]);

// returns an array of Post objects
$posts = $provider->getModels();
$totalCount = $provider->totalCount
```


2、联表查询，并联表排序

post model:

```php

//建立关联关系
public function getAuthor() {
    reutrn $this->hasOne(Article::className(),['id' => 'author_id])
}

$query = Post::find()
    ->joinWith('author',true,'RIGHT JOIN')
    ->where(['status' => 1]);

$provider = new ActiveDataProvider([
    'query' => $query, // 或 $query->asArray()
    'pagination' => [
        'pageSize' => 10,
    ],
    'sort' => [
        'defaultOrder' => [
            'created_at' => SORT_DESC,
            'title' => SORT_ASC, 
        ],
        'attributes' => [
            'article_type' => [
                	'asc' => ['article.type' => SORT_ASC],
                    'desc' => ['article.type' => SORT_DESC]
            ]  
        ],
    ],
]);

```

 + 当author表和post表都有type时，指明表名，如 `article.type`
 + 事实上，当使用`ActiveDataProvider`时，`post`表中的每个字段都是可以用来排序的


3、禁用分页:`$provider->setPagination(false);`

4、禁用排序：`$provider->setSort(false);`



## 排序[yii\data\Sort ](http://www.yiiframework.com/doc-2.0/yii-data-sort.html)

1、单一排序：

```php
$sort = new Sort([
			'defaultOrder' => ['id' => SORT_DESC,],
			'attributes' => [
				'id'   //这里必填相应的排序字段，否则，报错，如：Undefined index: id
			]
		]);

		$data = TradeRecord::find()->orderBy($sort->orders)->asArray()->all();

```

2、混合排序:

主要根据bill_id排序；如果bill_id相同，再根据id排序；

```php
$sort = new Sort([
			'defaultOrder' => ['id' => SORT_DESC,],
			'attributes' => [
				'id' => [
					'asc' => ['bill_id' => SORT_ASC,'id' =>SORT_ASC],
					'desc' => ['bill_id' => SORT_DESC,'id' => SORT_DESC],
					'default' => SORT_DESC,//默认排序方式
					'label' => 'id' //是在调用link()创建排序链接时，是用的标签
				],
			]
		]);

		$data = TradeRecord::find()->orderBy($sort->orders)->asArray()->all();
```











