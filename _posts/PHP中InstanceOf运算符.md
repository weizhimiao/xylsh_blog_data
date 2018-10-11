---
title: instanceof运算符的运用
date: 2016-09-22 21:30:00
categories:
- PHP
---


在PHP5中，通过方法传递变量的类型有不确定性。我们很难判断，一些操作是否可以运行。
使用instanceof运算符，可以判断当前实例是否可以有这样的一个形态。当前实例使用 instanceof与当前类，父类（向上无限追溯），已经实现的接口比较时，返回真。

代码格式：实例名 instanceof 类名

<!-- more -->

### instanceof 运算符的运用
如下例子可以运行。

没用instanceof运算符判断就会报错，示例代码如下：

```
<?php
   class User{
     private $name="zhenlw";
     public function  getName(){
       return "UserName is ".$this->name;
     }
  }
  class NormalUser extends User {
    private $age = 99;
    public function getAge(){
      return "age is ".$this->age;
    }
  }
  class UserAdmin{    //操作.
    public static function  getUserInfo(User $user){
      echo $user->getAge();
    }
  }
  $User = new User();
  UserAdmin::getUserInfo($User);
?>
```
运行后报如下错误：
```
Fatal error: Call to undefined method User::getAge() in D:\xampp\htdocs\test\8\test.php on line 20
```
因为你传入的对象参数根本没有getAge方法，如果是NormalUser的对象的话就可以正常运行了，这个时候我们运用instanceof运算符来进行判断，修改后的示例代码如下：

```
<?php
  class User{
    private $name="zhenlw";
    public function  getName(){
      return "UserName is ".$this->name;
    }
  }
  class NormalUser extends User {
    private $age = 99;
    public function getAge(){
      return "age is ".$this->age;
    }
  }
  class UserAdmin{  //操作.
    public static function  getUserInfo(User $user){
      if($user instanceof NormalUser){
        echo $user->getAge();
      } elseif($user instanceof User){
        echo $user->getName();
      }
    }
  }
  $User = new User(); // 这里是User的对象.
  UserAdmin::getUserInfo($User);
  echo "<br>";
  $normaluser = new NormalUser(); // 这里是NormalUser的对象.
  UserAdmin::getUserInfo($normaluser);
?>
```
运行结果：
```
UserName is zhenlw
age is 99
```
看到运行结果就知道了吧，运用instanceof判断数据类型后就可以保证代码的健壮性，不论你传入的是哪个对象，都可以正确的对此对象进行处理。


转自：http://blog.sina.com.cn/s/blog_4ce69a220100ksds.html
