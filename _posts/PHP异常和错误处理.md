---
title: PHP异常和错误处理
date: 2016-09-30 23:10:00
tags:
- 异常处理
- 错误处理
categories:
- PHP
---
首先，需要明确以下这两个概念
- 异常(exception)
> PHP中的异常，是程序运行中不符合预期的情况及与正常流程不同的状况。
> 是属于逻辑和业务流程的一种中断，而不是语法错误。

- 错误(error)
> PHP中的错误则属于自身的问题，是一种非法的语法或者环境问题导致的、让编译器无法通过检查甚至无法运行的情况。

<!-- more -->

## 异常处理

PHP中玉带任何自身问题都会触发一个错误，而不是抛出异常。所以，如果想使用异常处理来处理不可预知的问题，这是PHP办不到的。但在处理一些可以预知的错误，PHP的异常处理机制还是能够帮我们解决一些问题的。例如：
- 代码冗余复杂，到处充斥着if...else
- 代码可读性差

而异常处理，其主要作用是将『正常执行过程的代码』和『处理问题怎么处理的代码』进行分离。

### PHP常见的异常类
#### Exception
> 所有异常的基类。

类摘要
```php
<?php
  Exception {
    /* 属性 */
    protected string $message ;     //异常消息内容
    protected int $code ;           //异常代码
    protected string $file ;        //抛出异常的文件名
    protected int $line ;           //抛出异常在该文件中的行号
    /* 方法 */
    public __construct ([ string $message = "" [, int $code = 0 [, Exception $previous = NULL ]]] )           //异常构造函数
    final public string getMessage ( void )    //获取异常消息内容
    final public Exception getPrevious ( void )//返回异常链中的前一个异常
    final public int getCode ( void )          //获取异常代码
    final public string getFile ( void )       //获取发生异常的程序文件名称
    final public int getLine ( void )          //获取发生异常的代码在文件中的行号
    final public array getTrace ( void )       //获取异常追踪信息
    final public string getTraceAsString ( void )//获取字符串类型的异常追踪信息
    public string __toString ( void )            //将异常对象转换为字符串
    final private void __clone ( void )          //异常克隆
  }
```

#### 错误异常类

类摘要
```php
<?php
  ErrorException extends Exception {
      /* 属性 */
      protected int $severity ;     //异常级别
      /* 方法 */
      public __construct ([ string $message = "" [, int $code = 0 [, int $severity = 1 [, string $filename = __FILE__ [, int $lineno = __LINE__ [, Exception $previous = NULL ]]]]]] )   //异常构造函数
      final public int getSeverity ( void )             // 获取异常的严重程度
      /* 继承的方法 */
      final public string Exception::getMessage ( void )
      final public Exception Exception::getPrevious ( void )
      final public int Exception::getCode ( void )
      final public string Exception::getFile ( void )
      final public int Exception::getLine ( void )
      final public array Exception::getTrace ( void )
      final public string Exception::getTraceAsString ( void )
      public string Exception::__toString ( void )
      final private void Exception::__clone ( void )
  }
```
常见使用方式（使用set_error_handle()函数讲错误信息托管至ErrorException）
```php
<?php
function exception_error_handler($errno, $errstr, $errfile, $errline ) {
    throw new ErrorException($errstr, 0, $errno, $errfile, $errline);
}
set_error_handler("exception_error_handler");
/* Trigger exception 抛出异常 */
strpos();
?>
```

#### 自定义异常类

```php
<?php
/**
 * 自定义一个异常处理类
 */
class MyException extends Exception
{
    // 重定义构造器使 message 变为必须被指定的属性
    public function __construct($message, $code = 0, Exception $previous = null) {
        // 自定义的代码

        // 确保所有变量都被正确赋值
        parent::__construct($message, $code, $previous);
    }
    // 自定义字符串输出的样式
    public function __toString() {
        return __CLASS__ . ": [{$this->code}]: {$this->message}\n";
    }
    public function customFunction() {
        echo "A custom function for this type of exception\n";
    }
}
```

### PHP中异常用法
>

首先我们需要知道，***在PHP中只有手动抛出异常后才能捕获异常，对于抛出的异常只有进行捕获并作出相应的操作抛出异常才有意义，否则没有任何意义。***

抛出异常
```php
<?php
  function check($n){
    if(empty($n)){
      throw new Exception("参数错误");
    }
  }
```

#### try...catch
示例：关于上传操作的异常处理

方式一：异常发生时立即捕获
```php
<?php
  try{
    //可能出现错误的代码
    if(文件上传不成功)
      throw (上传异常)；
    if(更新数据库不成功)
      throw (数据库异常操作)；
  } catch(异常){
    //必须的补救措施，例如删除文件、删除数据库记录
    ...
  }
```
方式二：分散抛出，集中捕获
```php
<?php
  上传{
    if(文件上传不成功) throw (上传异常)；
    if(更新数据库不成功)  throw (数据库异常)；
  }
  其他{
    if(其他操作失败) throw （其他操作异常）；
  }
  //其他代码...
  try{
      上传；
      其他；
  } catch(上传异常){
      //上传异常处理、例如删除文件
  } catch(数据库异常){
      //数据库异常处理、比如删除数据库记录等
  } catch(其他异常){
      //其他异常处理，比如记录异常日志等
  }
```

**需要注意，exception作为超类应该放在最后捕获**


#### try...catch...finally
示例：
```php
<?php
  function inverse($x) {
      if (!$x) {
          throw new Exception('Division by zero.');
      }
      return 1/$x;
  }
  //
  try {
      echo inverse(5) . "\n";
  } catch (Exception $e) {
      echo 'Caught exception: ',  $e->getMessage(), "\n";
  } finally {
      echo "First finally.\n";
  }
  //
  try {
      echo inverse(0) . "\n";
  } catch (Exception $e) {
      echo 'Caught exception: ',  $e->getMessage(), "\n";
  } finally {
      echo "Second finally.\n";
  }
  //
  // Continue execution
  echo "Hello World\n";
?>

```

### PHP异常处理使用场景
#### 对程序的悲观预测
即当一个操作可能会由于一些不可控的因素（比如网络问题等），可能会出现执行不成功的情况。那么在种情况下就可以抛出异常，然后进行捕获，对异常情况进行细致的处理。
#### 程序的需要和对业务的需要
需要强调的是，异常是业务处理中必不可少的环节，我们不能对异常视而不见。
需要用到异常处理的情况
- 不希望业务代码中充斥着大量的打印、调试等处理；
- 业务中自定义的异常，对现实生活中各种业务进行补充；
- 对数据一致性有要求的业务操作中；

异常处理机制可以把每一件事当做事务来考虑，还可以把异常处理当做成一种内建的恢复系统。



#### 语言级别的健壮性要求
我们都知道PHP在健壮性这一点是不足的。对很多异常是没有强制性限制的。所以很多异常需要我们自己来做。

通过try...catch处理，我们可以把异常造成的逻辑中断破坏降低到最小的范围，并且经过补救处理后不影响业务逻辑的完整性；乱抛异常和只抛不捕获，或者捕获而不补救，都会导致数据混乱。




## 错误处理
### PHP中的错误级别
PHP的错误有很多类，包括warning、notice、deprecated、fatal error等。

常用到的错误级别
- deprecated，是最低级别的错误，表示不推荐、不建议。其虽不影响PHP正常的业务流程，但一般情况下建议修正；
- notice，通知级别错误。表示你语法中存在不恰当的地方。常见的是 使用未定义的变量就会报此错误。这种错误也影响PHP正常流程；
- warning，警告级别错误，是比较高级别的错误，表示在语法中出现很不恰当的地方，比如函数参数不匹配。这种级别的错误可能会导致出现不可预期的结果，所以建议修正；
- fetal error，致命错误。直接导致PHP流程终结，后面代码不在执行。
- prase error，语法解析错误。导致PHP代码无法通过语法检查。

错误信息显示控制
方式一：
php.ini
```
error_reporting = E_ALL | E_STRICT  #指定显示错误级别
display_errors = On                 #错误信息显示控制
```
方式二：PHP代码中
1. error_reporting(0),表示屏蔽所有错误信息。正式部署时采用这样的策略，来防止错误信息泄露敏感信息。
2. @mysql_connect(),抑制错误信息输出。



### PHP错误处理机制

#### trigger_error
该方法用于主动抛出一个错误。示例
```php
<?php
  if(mt_rand(0,1) == 1){
    triggererror("random no eq 0",E_USER_ERROR);
  }

```

#### set_error_handler
PHP中提供了set_error_handler（）函数可以用来接管PHP错误处理。

**set_error_handler(error_function, error_type);**
- error_function,规定发生错误时运行的函数。（必须）
- error_types,规定在哪个错误级别报告级别会显示用户定义的错误，（可选，默认『E_ALL』）

示例：
```php
<?php
  function customError($errNo, $errStr, $errFile, $errLine){
    echo "<b>错误代码：</b>[$errNo]{$errStr}\r\n";
    echo "错误所在的代码行：{$errLine},文件:{$errFile}\r\n";
    echo "PHP 版本 ，".PHP_VERSION."(".PHP_OS.")\r\n";
    die();  //如果有必要，用户自定义的错误处理程序必须终止（die（）；）当前脚本。
  }

  set_error_handler("custonError", E_ALL | E_STRICT);
  $a = array('o' => 2,3,4,8);
  echo $a[o]
```

自定义的错误处理函数一定要有这四个输入变量$errno, $errstr, $errfile, $errline.
- errno,代表错误等级的一组常量。比如 E_WARNING 其二进制掩码为4，表示告警信息。

**注意：** 自定义的错误处理函数并不能托管所有种类的错误，比如E_ERROR, E_PARSE, E_CORE_ERROR, E_CORE_WARNING, E_COMPILE_ERROR, E_COMPILE_WARNING,以及E_STRICT中的部分。

**注意：** 如果使用自定义的 set_error_handler 接管PHP的错误处理，先前代码里的错误抑制 @ 也将会失效，这是这种错误也会被显示。

#### restore_error_handler
该函数可以取消 set_error_handler 的错误接管.


## 结合PHP错误处理主动抛出异常

结合PHP的错误处理机制和异常处理机制，通过组合也可以实现同时捕获异常和非致命错误。
示例
```php
<?php
  function customError($errNo, $errStr, $errFile, $errLine){
    //自定义错误处理时，可以手动抛出异常
    throw new Exception($level."|".$errStr);
  }
  set_error_handler("custonError", E_ALL | E_STRICT);
  try{
    $a = 5/0;
  } catch(Exception $e){
      echo "错误信息".$e->getMessage();
  }
```
通过上面示例可以捕获到异常和非致命错误，但对于 fetal error 这样的错误却捕获不到。但是通过其他的一些方法，我们还是可以做一些处理。 register_shutdown_function 该函数会在程序终止或die时触发一个函数，我们可以利用该触发函数做一些最后的收尾操作。

调用时机
- 当页面被用户强制停止时
- 当程序代码运行超时时
- 当ＰＨＰ代码执行完成时，代码执行存在异常和错误、警告

void register_shutdown_function ( callable $callback [, mixed $parameter [, mixed $... ]] )

示例
```php
<?php
  function shutdown()
  {
      // This is our shutdown function, in
      // here we can do any last operations
      // before the script is complete.
      echo 'Script executed with success', PHP_EOL;
  }
  register_shutdown_function('shutdown');
?>
```

对于 prase error 级别的错误，PHP层面就没有办法做什么了。我们只能通过开启错误日志，来记录这些错误。

php.ini设置
```
log_errors = On
error_log = /usr/log/log.log
```
