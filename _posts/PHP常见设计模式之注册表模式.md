---
title: PHP常见设计模式之注册表模式
date: 2016-10-14 20:40:00
tags:
- 设计模式
- 注册表模式
categories:
- PHP
---

## 什么是注册表模式
注册表（registry）模式仅是一个单独的全局类，在我们需要时允许代码检索一个对象的相同实例，也可以在我们需要的时候创建另一个实例。

注册表就像是一个对象库，只要我们随时签入或者签出对象，而不必担心因为将这些对象保留太久而引起功能障碍。

我们认为注册表模式中最简单的方式就是键/值存储，键作为一个对象的实例，而值就是实例本身。当我们需要管理键/值对的数组时，这个模式便开始发挥功效，存储最早实例化的实例，并且返回一个引用到请求中的同一个实例。

## 注册表模式和单例模式的关系
相同点：
> 和单例模式一样，注册表模式也是用于访问全局可重用的对象；

区别：
> 注册表模式不负责创建对象，纯粹用于保持全局存储，可以容纳任何数量的相同类的实例。这使得它非常适合类似于数据库连接和配置对象等的采用单例模式满足不了其需求的情况。

<!-- more -->

## 注册表模式的实现
示例，
```php
<?php
class Registry{

  private static $_store = array();

  public static function add($object, $name = null){
    $name = (!is_null($name)) ? $name : get_class($object);
    $name = strtolower($name);

    $return = null;
    if(isset(self::$_store[$name])){
      $return = self::$_store[$name];
    } else{
      self::$_store[$name] = $object;
      $return = self::$_store[$name];
    }
    return $return;
  }

  public static function get($name){
    if(!self::contains($name)){
      throw new Exception("object does not exist in registry");
    }
    return self::$_store[$name];
  }

  public static function contains($name){
    if(!isset(self::$_store[$name])){
      return false;
    }
    return true;
  }

  public static function remove($name){
    if(self::contains($name)){
      unset(self::$_store[$name]);
    }
  }  
}
```

注册表模式的4个实现方法：
- Registry::set(),添加一个对象到注册表，你可以指定一个名称（为多个实例）或者使用默认的类名（为单例，与行为类似）
- Registry::get(),从注册表的名字中检索一个对象
- Registry::contains(),在注册表中检查一个对象是否存在
- Registry::unset(),通过对象名在注册表中删除一个对象


## 注册表模式的应用
创建了Registry类之后，我们可以通过两种方式来使用它。
- 外部
- 内部


示例，两种方式的数据库连接代码。

### 外部
```php
<?php
  $read = new DBReadConnection();
  Registry::add($read);

  $write = new DBWriteConnection();
  Registry::add($wite);

  //在之后的任何代码中，我们可以通过以下方式获得对应实例
  $read = Registry::get('DBReadConnection');
  $write = Registry::get('DBWriteConnection');
```

在这个示例中，我们没有传入实例名称，而是通过使用类名从注册表中提取对象。
在Registry类能访问到的任何地方该对象都可用。

### 内部
```php
<?php
abstract class DBConnection extend PDO{
  public statuc function getInstance($name = null){
    $class = get_called_class();

    $name = (!is_null($name)) ? $name : $class;
    $name = strtolower($name);
    if(!Registry::contains($name)){
      $instance = new $class();
      Registry::add($instance, $name);
    }
    return Registry::get($name);
  }
}

class DBWriteConnection extend DBConnection{
  public function __contruct(){
    parent::_contruct(APP_DB_DSN, APP_DB_USER, APP_DB_PWD);
  }
}

class DBReadConnection extend DBConnection{
  public function __contruct(){
    parent::_contruct(APP_DB_DSN, APP_DB_USER, APP_DB_PWD);
  }
}
```
要使用这些代码，我们只需在任一读或者写连接类中调用DBConnection::getInstance()即可，就像
```php
<?php
  //获得一个读实例
  $read_db = DBReadConnection::getInstance();
  //获得一个写实例
  $write_db  = DBWriteConnection::getInstance();

  //获得另一个读实例
  $another_read_db = DBReadConnection::getInstance("another_db");
```
在某些方面，这是一个单例模式和工厂模式的混合物。

## 注册的若干问题
对于外部注册表，你不能延迟加载；也就是说，在我们使用之前，必须初始化注册表中的每一个对象。如果操作顺序变得更为复杂，我们可能就会错过某个对象而产生预料之外的错误。
