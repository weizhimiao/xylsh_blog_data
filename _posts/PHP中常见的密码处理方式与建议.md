---
title: PHP中常见的密码处理方式与建议
date: 2017-02-22 20:30:00
tags:
- 密码
- 哈希
categories:
- PHP
---

密码安全的重要性我们就不用再去强调，随着在线攻击的增多，如果我们对密码没有进行合适的处理或做防御措施，我们的应用就会肯定会收到来自各方的威胁和攻击。

![password](http://n.sinaimg.cn/games/3ece443e/20170223/QQTuPian20170223093340.png)


所以作为开发者，我们需要对用户的密码做好预防措施。

<!-- more -->

## 关于密码我们应该遵守的一些原则

### 绝对不能知道用户的密码
- 我们绝对不能知道用户的密码，也不能有获取用户密码的方式。
- **知道的越少（包括我们开发者自己）越安全。**

### 绝对不去约束用户的密码
- 最好不要去约束密码的长度、格式等。

- 如果要求密码符合一个特定的模式，其实对于那些不怀好意的人也提供了攻击的途径。

- 如果必须要约束的话，建议只限制最小长度。并把常用的密码或基于字典创建的密码加入黑名单，也是一个好主意。

### 绝对不通过电子邮件发送用户的密码

对于一个web应用来说，重置或修改密码时，我们应该在邮件里发送用于设定或修改密码的 `URL` 。而且这个`URL`中应该会包含一个唯一的令牌，这个令牌只能在设定或修改密码时使用一次。在设定或修改密码之后，我们就应该把这个令牌置为失效。

## 使用 `bcrypt` 计算用户密码的哈希值
> 目前，通过大量的审查，最安全的哈希算法是 `bcrypt` 。

首先，我们明确两个概念，哈希、加密。哈希和加密有什么区别？
- 加密
> 加密是双向算法，加密的数据之后通过解密还可以得到。

- 哈希
> 哈希是单向算法，哈希后的数据不能再还原成原始值。
> 哈希算法的用途，
> - 验证数据的完整性（要求算法速度快）
> - 用户提高密码等需要单向验证的数据的安全性（要求安全性高，甚至故意要求时间慢）

一般我们在数据库中保存的应该是计算出来的密码的哈希值。这样即使我们的数据库泄露了，他们也只能看到这些无意义的密码的哈希值。

哈希的算法有很多种，
- `MD5`
> `MD5`即`Message-Digest Algorithm 5`（信息-摘要算法5），用于确保信息传输完整一致。是计算机广泛使用的杂凑算法之一（又译摘要算法、哈希算法），主流编程语言普遍已有`MD5`实现。将数据（如汉字）运算为另一固定长度值，是杂凑算法的基础原理，`MD5`的前身有`MD2`、`MD3`和`MD4`。
>

- `SHA1`
> 安全哈希算法（`Secure Hash Algorithm`）主要适用于数字签名标准 （`Digital Signature Standard DSS`）里面定义的数字签名算法（`Digital Signature Algorithm DSA`）。对于长度小于`2^64`位的消息，`SHA1`会产生一个`160`位的消息摘要。当接收到消息的时候，这个消息摘要可以用来验证数据的完整性。在传输的过程中，数据很可能会发生变化，那么这时候就会产生不同的消息摘要。 `SHA1`有如下特性：不可以从消息摘要中复原信息；两个不同的消息不会产生同样的消息摘要,(但会有`1x10 ^ 48`分之一的机率出现相同的消息摘要,一般使用时忽略)。

- `bcrypt`
> `bcrypt`是专门为密码存储而设计的算法，基于`Blowfish`加密算法变形而来，由`Niels Provos`和`David Mazières`发表于`1999`年的`USENIX`。　`bcrypt`最大的好处是有一个参数（`work factor`)，可用于调整计算强度，而且`work factor`是包括在输出的摘要中的。随着攻击者计算能力的提高，使用者可以逐步增大`work factor`，而且不会影响已有用户的登陆。　`bcrypt`经过了很多安全专家的仔细分析，使用在以安全著称的`OpenBSD`中，一般认为它比`PBKDF2`更能承受随着计算能力加强而带来的风险。**bcrypt也有广泛的函数库支持，因此我们建议使用这种方式存储密码**。

- `scrypt`
> scrypt不仅计算所需时间长，而且占用的内存也多，使得并行计算多个摘要异常困难，因此利用`rainbow table`进行暴力攻击更加困难。**scrypt没有在生产环境中大规模应用，并且缺乏仔细的审察和广泛的函数库支持** 。但是，`scrypt`在算法层面只要没有破绽，它的安全性应该高于`PBKDF2`和`bcrypt`。

目前，通过大量的审查，最安全的哈希算法是 `bcrypt` 。与 `MD5` 和 `SHA1` 不同， `bcrypt` 算法会自动加盐，来防止潜在的彩虹表攻击。 `bcrypt` 算法会花费大量的时间反复处理数据，来生成安全的哈希值。在这个过程中，处理数据的次数叫工作因子（`work factor`）。工作因子的值越高，破解密码哈希值的时间会成指数倍增长。

`bcrypt` 算法永不过时，如果计算机的运算速度变快了，我们只需要提高工作因子即可。

顺带说一下，任何情况下尽可能的不要使用 `md5` 算法，至少也要使用 SHA 系列的哈希算法。因为`md5`算法以目前计算机的计算能力来说显得比较简单，而 `md5` 的性能优势现在也已经完全可以忽略不计了。

## 密码哈希API

上面我们说到 `bcrypt` 算法最安全，最适合对我们的密码进行哈希。 `PHP` 在 `PHP5.5.0+` 的版本中提供了原生的`密码哈希API`供我们使用，这个`密码哈希API`默认使用的就是 `bcrypt` 哈希算法，从而大大简化了我们计算密码哈希值和验证密码的操作。

### PHP原生密码哈希API


密码哈希函数：
- `password_get_info`
> 返回指定的哈希值的相关信息

- `password_hash`
> 创建密码的哈希（hash）

- `password_needs_rehash`
> 检查给定的哈希是否与给定的选项匹配

- `password_verify`
> 验证密码是否和哈希匹配

#### password_get_info

说明
```php
array password_get_info ( string $hash )
```
参数
- `hash`, 一个由 password_hash() 创建的散列值。

示例，
```php
<?php
var_dump(password_get_info($hash));
// Example
array(3) {
  ["algo"]=>
  int(1)
  ["algoName"]=>
  string(6) "bcrypt"
  ["options"]=>
  array(1) {
    ["cost"]=>
    int(10)
  }
}
?>
```

#### password_hash
**说明**
```php
string password_hash ( string $password , integer $algo [, array $options ] )
```

password_hash() 使用足够强度的单向散列算法创建密码的哈希（hash）。 password_hash() 兼容 crypt()。 所以， crypt() 创建的密码哈希也可用于 password_hash()。

当前支持的算法：

- `PASSWORD_DEFAULT`
> 使用 `bcrypt` 算法 (PHP 5.5.0 默认)。 注意，该常量会随着 `PHP` 加入更新更高强度的算法而改变。 所以，使用此常量生成结果的长度将在未来有变化。 因此，数据库里储存结果的列可超过60个字符（最好是255个字符）。

- `PASSWORD_BCRYPT`
> 使用 `CRYPT_BLOWFISH` 算法创建哈希。 这会产生兼容使用 "`$2y$`" 的 crypt()。 结果将会是 60 个字符的字符串， 或者在失败时返回 `FALSE。`

支持的选项：

- `salt` - 手动提供哈希密码的盐值（`salt`）。这将避免自动生成盐值（`salt`）。
> 省略此值后，password_hash() 会为每个密码哈希自动生成随机的盐值。这种操作是有意的模式。
>
> **Warning 盐值（salt）选项从 PHP 7.0.0 开始被废弃（deprecated）了。 现在最好选择简单的使用默认产生的盐值。**

- `cost` - 代表算法使用的 `cost`。crypt() 页面上有 `cost` 值的例子。
> 省略时，默认值是 10。 这个 `cost` 是个不错的底线，但也许可以根据自己硬件的情况，加大这个值。


**参数**
- `password`, 用户的密码。
>  使用 `PASSWORD_BCRYPT` 做算法，将使 `password` 参数最长为72个字符，超过会被截断。

- `algo`, 一个用来在散列密码时指示算法的密码算法常量。

- `options`, 一个包含有选项的关联数组。目前支持两个选项：
  - `salt`，在散列密码时加的盐（干扰字符串），
  - `cost`，用来指明算法递归的层数。这两个值的例子可在 crypt() 页面找到。
  > 省略后，将使用随机盐值与默认 `cost`。

**示例**
示例1，使用默认算法哈希密码
```php
<?php
/**
 * 我们想要使用默认算法哈希密码
 * 当前是 BCRYPT，并会产生 60 个字符的结果。
 *
 * 请注意，随时间推移，默认算法可能会有变化，
 * 所以需要储存的空间能够超过 60 字（255字不错）
 */
echo password_hash("rasmuslerdorf", PASSWORD_DEFAULT)."\n";
?>
// 输出类似于：
// $2y$10$.vGA1O9wmRjrwAVXD98HNOgsNpDczlqm3Jq7KnEd1rVAGv3Fykk1a
```
示例2，手动设置 `cost`
```php
<?php
/**
 * 在这个案例里，我们为 BCRYPT 增加 cost 到 12。
 * 注意，我们已经切换到了，将始终产生 60 个字符。
 */
$options = [
    'cost' => 12,
];
echo password_hash("rasmuslerdorf", PASSWORD_BCRYPT, $options)."\n";
?>
// 输出类似于：
// $2y$12$QjSH496pcT5CEbzjD/vtVeH03tfHKFy36d4J0Ltp3lRtee9HDxY3K
```
示例3，如何选择一个适合当前服务器的 cost

```php
<?php
/**
 * 这个例子对服务器做了基准测试（benchmark），检测服务器能承受多高的 cost
 * 在不明显拖慢服务器的情况下可以设置最高的值
 * 8-10 是个不错的底线，在服务器够快的情况下，越高越好。
 * 以下代码目标为  ≤ 50 毫秒（milliseconds），
 * 适合系统处理交互登录。
 */
$timeTarget = 0.05; // 50 毫秒（milliseconds）

$cost = 8;
do {
    $cost++;
    $start = microtime(true);
    password_hash("test", PASSWORD_BCRYPT, ["cost" => $cost]);
    $end = microtime(true);
} while (($end - $start) < $timeTarget);

echo "Appropriate Cost Found: " . $cost . "\n";
?>
```
输出类似于：
```
Appropriate Cost Found: 10
```
#### password_needs_rehash

说明
```php
boolean password_needs_rehash ( string $hash , integer $algo [, array $options ] )
```

参数
- `hash`, 一个由 password_hash() 创建的散列值。
- `algo`, 一个用来在散列密码时指示算法的密码算法常量。
- `options`, 一个包含有选项的关联数组。目前支持两个选项：
  - `salt`，在散列密码时加的盐（干扰字符串），
  - `cost`，用来指明算法递归的层数。这两个值的例子可在 crypt() 页面找到。

示例，
```php
<?php
$password = 'rasmuslerdorf';
$hash = '$2y$10$YCFsG6elYca568hBi2pZ0.3LDL5wjgxct1N8w/oLR/jfHsiQwCqTS';

// cost 参数可随硬件的提升也不断提升
$options = array('cost' => 11);

// 使用纯文本密码 验证存储的散列
if (password_verify($password, $hash)) {
    // 检查是否有更新的散列算法可用或 cost 是否已经改变
    if (password_needs_rehash($hash, PASSWORD_DEFAULT, $options)) {
        // 如果是，请创建一个新的哈希值，并替换旧的哈希值
        $newHash = password_hash($password, PASSWORD_DEFAULT, $options);
    }

    // 用户登录验证完成
    // ...
}
?>
```

#### password_verify
说明
```php
boolean password_verify ( string $password , string $hash )
```
> 注意 password_hash() 返回的哈希包含了算法、 cost 和盐值。 因此，所有需要的信息都包含内。使得验证函数不需要储存额外盐值等信息即可验证哈希。

参数
- `password`, 用户的密码。
- `hash`, 一个由 password_hash() 创建的散列值。

示例，
```php
<?php
// 想知道以下字符从哪里来，可参见 password_hash() 的例子
$hash = '$2y$07$BCryptRequires22Chrcte/VlQH0piJtjXl.0t1XkA8pw9dMXTpOq';

if (password_verify('rasmuslerdorf', $hash)) {
    echo 'Password is valid!';
} else {
    echo 'Invalid password.';
}
?>
```
以上例程会输出：
```
Password is valid!
```

### PHP5.50 之前的密码哈希 API

安东尼·费拉拉（PHP原生密码哈希 API的开发者）为PHP5.5.0 以下的版本也提供了 `ircmaxell/password-compat`组件（https://packagist.org/packages/ircmaxell/password-compat）。

这个组件也实现了PHP密码哈希API中的所有函数，
- `password_get_info`

- `password_hash`

- `password_needs_rehash`

- `password_verify`

我们可以直接使用 `Composer` 把这个组件添加到我们的应用中就行了。例如，
```bash
composer require ircmaxell/password-compat
```
