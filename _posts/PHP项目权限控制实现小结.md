---
title: PHP项目权限控制实现小结
date: 2016-10-07 20:00:00
tags:
  - 权限控制
  - RBAC
categories:
  - PHP
---

RBAC，Role Based Access Control,基于角色的访问控制。实体关系如下：

![RBAC实体关系](http://n.sinaimg.cn/games/3ece443e/20161007/RBACShiTiGuanXi.png)

<!-- more -->

## 实现逻辑设计

用户表，user
- id,用户标识
- username，用户名
- passwd，密码
- status,状态

角色表，role
- id,角色标识
- name，角色名称
- desc，角色描述
- status，状态

权限节点表，node
- id，权限节点标识
- name，权限名称
- title，权限描述
- status，状态
- sort，排序
- pid，父标识
- level，层级（一般，1、项目，2、模块（类），3、操作（方法））

用户角色关系表，role_user
- id，关系标识
- role_id,角色标识
- user_id,用户标识

角色权限关系表, access
- id，关系标识
- role_id，角色标识
- node_id，权限标识

## 实现过程
角色 --> 权限节点 --> 角色权限关系 --> 用户 --> 角色用户关系

角色：
- 创建角色
- 角色管理

权限
- 添加权限节点
- 权限管理

权限节点列表的『权限结构』的递归显示问题，可用无限制分类来实现。

无限制分类简单实现
```php
class Tree{
  static public $treeList = array();  //存放无限制分类结果

  public function create($data, $pid=0){
    foreach ($data as $key => $value) {
      if($value['pid'] == $pid){
          self::$treeList[] = $value;
          unset($data[$key]);
          self::$create($data, $value['id']);
      }
    }
    return self::$treeList;
  }
}
```

角色权限关系
- 添加角色-权限节点关系（添加角色新权限时，必须先删除原来的权限设置）
- 管理角色-权限节点关系

用户：
- 添加用户
- 用户管理

用户角色关系
- 添加用户-角色关系
- 管理用户-角色关系

## 权限控制应用


实现权限控制方法步骤：
1. 通过用户-角色关系表，获得用户所属角色
2. 从权限表中，获得所有权限列表，并应用无限级分类实现分类排序
3. 根据用户角色，通过角色-权限关系表获得当前拥有的权限几点信息
4. 组合所有权限列表，和当前拥有的权限数据，在所有权限节点列表中标识当前用户是否具有对各个节点的权限。去除当前用户没有权限操作的权限节点。将当前的权限信息保存到当前用户的会话中。
5. 用户进行操作各个权限节点时，首先检测当前的操作是否在会话中的权限信息中存在。不存在则按照权限不足处理，存在则继续进行操作。

over~
