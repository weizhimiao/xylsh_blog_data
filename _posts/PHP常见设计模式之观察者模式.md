---
title: PHP常见设计模式之观察者模式
date: 2016-10-18 22:40:00
tags:
- 设计模式
- 观察者模式
categories:
- PHP
---


观察者模式类似于javascript的事件。其核心在于允许我们的应用程序注册一个回调，当某个特定的事件发生时便会触发它。在javascript中，这些事件由单机（onclick）、页面加载（onload）、或者鼠标移动（onmousevoer）等动作组成。

观察者设计模式能够是我们更便利地创建查看目标对象状态的对象，并且提供与核心对象非耦合的指定功能性。

观察者设计模式使用场景（插件系统、缓存系统）
- 对一个对象状态的更新，需要其他对象同步更新，而且其他对象的数量动态可变。
-
- 对象仅需要将自己的更新通知给其他对象而不需要知道其他对象的细节。


<!-- more -->

### UML设计
![观察者模式](http://n.sinaimg.cn/games/3ece443e/20161018/GuanChaZheMoShi.png)

说明：我们通过一个名为Event的类实现，这个类共有两个方法：
- registerCallBack（）,这个方法允许我们使用规定的名称附加许多回调到一个事件中。
- trigger（）,这个方法将会触发刚才命名的事件，并调用该事件已注册的任何回调。

### 代码实现
event.php
```php
<?php

class Event{

  static protected $callbacks = array();

  static public function registerCallBack($eventName， $callback){
    if(!is_callable($callback)){
      throw new Exception("Invalid callback");
    }

    $eventName = strtolower($eventName);
    self::$callbacks[$eventName][] = $callback；
  }

  static public function trigger($eventName, $data){
    $eventName = strtolower($eventName);
    if(isset(self::$callbacks[$eventName])){
      foreach($self::$callbacks[$eventName] as $callback ){
          //回调可以是一个函数（包括匿名函数（闭包））、也可以是一个定义过__invoke()的对象
          $callback($data);
      }
    }  
  }
}

```

回调事件注册后将会被保存到Event类的静态受保护的Event::$callbacks属性中，成为一个以事件名作为Key的多为数组。如下所示，
```php
<?php
array(
  "eventName" => array(
    "callback 1",
    "callback 2"
    ...
    ),
    ...
  );
```
当触发一个事件时，我们仅遍历事件的Event::$callbacks子数组，然后依次调用每个回调。

### 使用示例

1、先定义一个MyDataRecord类表示数据层的一部分。这个类有个save（）方法，我们在save（）方法中添加一个save事件，每当我们调用它时，就会触发一个save事件，
```php
<?php

class MyDataRecord{
  public function save(){
    //保存操作...

    //触发 save 事件
    Event::trigger("save",array("Hello", "world"));
  }
}
```
2、接着我们创建回调，用事件名save通过Event::registerCallBack()来注册它。
```php
<?php
Event::registerCallBack("save", function($data){
  echo "Clear Cache".PHP_EOL;
  var_dump($data);
  });
```
3、现在每当调用MyDataRecord->save();方法时，都将使回调生效。
```php
<?php
//实例化一个daterecord类
$data = new MyDataRecord();
$data->save();
```
运行结果：
```
Clear Cache
array(2){
  [0] =>
  string(5) "Hello"
  [1] =>
  string(5) "world"
}
```

### 更多
1、同一个事件我们可以注册多个回调，这些回调将会通过FIFO（先进先出）来调用。

2、回调可以是一个函数（包括匿名函数，又称闭包），也可以是定义过 `__invoke()` 魔术方法的对象。`__invoke()`方法的作用是当我们试图将当前的这个对象作为函数使用时，这个方法就会自动调用。

```php
<?php
//Logger callback
class LogCallback{
  public function __invoke($data){
    echo "Log Data".PHP_EOL;
    var_dump($data);
  }
}

//注册 log callback
Event::registerCallBack("save", new LogCallback());

//注册 clear cache callback
Event::registerCallBack("save", function($data){
  echo "Clear cache".PHP_EOL;
  var_dump($data);
  });

//实例化一个 data record
$data = new MyDataRecord();
$data->save();
```
运行结果如下：
```
Log Data
array(2){
  [0] =>
  string(5) "Hello"
  [1] =>
  string(5) "world"
}
Clear Cache
array(2){
  [0] =>
  string(5) "Hello"
  [1] =>
  string(5) "world"
}
```

over~
