---
title: PHP常见设计模式之策略模式
date: 2016-10-19 22:40:00
tags:
- 设计模式
- 策略模式
categories:
- PHP
---

策略模式允许类的使用者为这个类根据需要注入不同的依赖。通常情况下，这些依赖表现为对象、闭包或者回调方式，他们完成类所必要的要求以执行预期行为。

<!-- more -->
## UML设计

![策略模式](http://n.sinaimg.cn/games/3ece443e/20161020/CeLueMoShi-YiLaiZhuRu.png)

## 代码实现

对于每一个依赖，我们可以指定一个setter方法（添加一个getter方法更好），它将接收可以满足依赖要求的参数。

示例：使用策略设计模式实现一个日志类
```php
<?php

class Log{

  protected $engine = false;

  public function add($logArr){
    if(!$this->engine){
      throw new Exception( "unable to write log. no engine set");
    }

    $logArr['datetime'] = time();
    $this->engine->add($logArr);
  }

  public function setEngine(Log_Engine_Interface $engin){
    $this->engine = $engin;
  }

  public function getEngine(){
    return $this->engine;
  }

}
```

现在我们用Log类，传入我们希望使用的任何一种数据存储引擎。

我们先定义一个接口或者抽象类，通过接口或者类的类型提示，确保每个驱动程序都符合要求。这里我们用接口来进行约束，使用add()给日志添加一个事件。
```php
<?php
interface Log_Engine_Interface{
  public function add(array $LogData);
}
?>
```
之后我们来定义一个引擎。

基于文件的存储引擎
```php
<?php
class Log_Engine_File implements Log_Engine_Interface{

  public function add(array $data){
    $line = json_encode($data).PHP_EOL;

    $location = "/var/log/app_file_log.log";
    if(!file_put_contents($location, $line, FILE_APPEND)){
      throw new Exception( "an error occurred writing to file. ");
    }
  }
}
```

接下来我们就可在程序中调用Log类：

```php
<?php
  $engine = new Log_Engine_File();

  $log = new Log();
  $log->setEngine($engine);

  $logData = array(
    "user" => "zhangsan",
    "action" => "spend",
    "msg" => "....."
  );
  $log->add($logData);
```

当然我们可以还可以和注册表模式结合起来，使之更加方便我们使用。

```php
<?php
  $engine = new Log_Engine_File();

  $log = new Log();
  $log->setEngine($engine);

  //加入到注册表，方便我们随时使用
  Register::add($log);
```

策略模式的伟大之处在于它不像工厂模式，日志类无需知道每一个不同的存储引擎的相关具体内容。这就意味着任何使用日志类的开发者都可以添加他们自己的存储引擎，只需要相应的存储引擎符合接口就行。例如我们可以继续给日志类增加MySQL存储引擎、Memcache存储引擎等等。



over~
