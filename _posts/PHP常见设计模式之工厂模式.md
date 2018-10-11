---
title: PHP常见设计模式之工厂模式
date: 2016-10-14 21:00:00
tags:
- 设计模式
- 工厂模式
categories:
- PHP
---

## 什么是工厂模式
工厂模式是我们最常用的实例化对象模式了，顾名思义工厂设计模式，就是用于制造对象的一种设计模式，是一种用来代替new操作的一种模式。

其的最大价值在于它可以将多个对象设置封装成单一、简单的方法调用。

对外提供获取某个对象的新实例的接口，同时使调用代码避免确定实际实例化基类的步骤。

<!-- more -->

## 工厂模式的应用和使用场景
通常情况下，虽然我们很少使用工厂模式，但它仍然最适合初始化基于驱动安装的许多变种中的一种。例如，不同的配置、会话或缓存存储引擎。例如，当我们设置一个日志对象时，我们需要设置日志类型（如，基于文本、MySQL或者其他）、日志的位置、以及类似于凭证条目。

## 工厂模式的实现
### UML设计

![UML设计图](http://n.sinaimg.cn/games/3ece443e/20161014/GongChangMoShiUMLTu.png)

说明：
- 三个基类Log_File、Log_Mysql、Log_Sqlite,都具有名为dosomething()的公用方法，该方法采用他们各自独特的方式执行具体的逻辑。且其返回类型也应该完全相同。
- Log_factory类用于创建下面三个任意一个基类的实例，并将其返回至代码流。

### 代码实现
工厂类Log_factory实现
```php
<?php

class Log_factory{

  public function getLog($type = 'file', array $options){
    $type = strtolower($type);

    $class = "Log_".unfirst($type);
    require_once str_replace('_', DIRECTORY_SEPARATOR, $class).'.php';

    $log = new $class($options);
    switch($type){
      case 'file':
        $log->setPath($options['location']);
        break;
      case 'mysql':
        $log->setUser($options['username']);
        $log->setPassword($options['password']);
        $log->setDBname($options['location']);
        break;
      case 'sqlite':
        $log->setDBPath($options['location']);
        break;
    }

    return $log;
  }
}
```


**Tips:**
在实际应用中，我们可以把getLog()方法生成的对象添加到Registry，这样就不用一遍又一遍的实例化这些对象。
