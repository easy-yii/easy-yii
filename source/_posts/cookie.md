---
title: cookie
date: 2017-03-24 16:19:15
tags:
---

## 删除cookie

+  移除一个 Cookie 对象：'Yii::$app->response->getCookies()->remove('cookie_name')'
+  移除所有 Cookie：'Yii::$app->response->getCookies()->removeAll()'

## 获取cookie

```PHP
    $cookie = Yii::$app->request->cookies;
    
    $cookie->get('cookie_name'); //返回一个\yii\web\Cookie 对象
    
    $cookie->getValue('cookie_name'); //直接返回 Cookie 的值
    $cookie[‘cookie_name’];//直接返回 Cookie 的值
 
    $cookie->has(‘快乐的小飞侠’);   //判断一个 Cookie 是否存在
   
    $cookie->count();//读取 Cookie 的总数
    $cookie->getCount();//跟count() 一样
```


## 添加cookie

```php
    $cookie = new Cookie();
    $cookie->name = $unionid;
    $cookie->value = $result['data']['access_token'];
    $cookie->expire = time() + $result['data']['expires_in'] - 3600;
    $cookie->httpOnly = true; //无法通过 js 读取 cookie
    yii::$app->response->cookies->add($cookie);
```
