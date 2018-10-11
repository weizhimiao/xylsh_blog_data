---
title: Github中Webhooks的应用
date: 2017-02-03 20:30:00
tags:
- git
- 版本控制
categories:
- 软件工程
---


`Webhooks` 是 `Github` 为我们提供的一种订阅某些时间的功能。当这些事件被触发时，`Github` 会将信息以`POST`方式发送到我们指定的`URL`。通过`Webhooks` 我们可以记录更新，触发`CI构建`、更新备份镜像，甚至服务器的自动部署等等我们能够想到的各个地方。

`Github` 上对于每个账号或者项目上的每个事件最多可以创建20个 `Webhooks` 。
<!-- more -->

## 事件

配置 `Webhook` 时，我们可以自定义具体的事件。只有定义定义的这些事件才会触发对指定的`URL`的请求。默认，`webhooks` 只订阅 `push` 事件，不过我们也可通过 `API` 或者 `UI` 来更改订阅事件的列表。

每个事件对应于可能发生在我们组织或 `Repository` 中的一组特定操作。 例如，如果订阅 `issues` 事件，则每次打开，关闭，标记 `issues` 时，我们都会收到详细的有效内容。

可用事件有：

事件名称|描述
------|-----------
`*`	  |任何时间触发任何事件（通配符事件）。
`commit_comment`|每次提交被注释时。
`create`|每次分支或标记被创建时。
`delete`|每次分支或标记被删除时。
`deployment`|每次 `Repository` 被部署。
`deployment_status`|任何时候，Repository的部署有来自API的状态更新时。
`fork`  |每次`Repository`被`fork`时。
`gollum`|每次`Wiki Page` 被更新时。
`issue_comment`|每次 `Issue` 中的评论被添加、编辑、或删除时。
`issues`|每次 `issue` 被分配、未分配，标记，未标记，打开，编辑，重大事件，拆除，关闭或重新打开时。
`label`	|每次标签被创建、编辑、或删除时。
`member`|	每次添加或删除一个协作者用户，或者更改他们的权限时。
`membership`|任何时候添加或删除团队中用户。（仅组织钩子）
`milestone`|每次创建、关闭、打开、编辑或删除 `milestone` （里程碑）时。
`organization`|任何时候添加，删除或邀请用户加入组织。 仅组织钩子。
`page_build`|每次页面网站建立时。
`project_card`|项目卡片创建，更新或删除。
`project_column`|项目 `column` 创建，更新，移动或删除时。
`project`|项目创建、编辑、或删除时。
`public`|`Repository` 从private更改为public时。
`pull_request_review_comment`|每次pull请求被创建，编辑或删除差异注释时。
`pull_request_review`|每次提交Pull请求审核时。
`pull_request`|	每次 pull 请求被打开，关闭，重新打开，编辑，分配，未分配，请求审核，审核请求已移除，已标记，未标记或已同步时。
`push`  |每次 push 时（默认事件）

`repository`|每次 `Repository` 被创建、删除、公开或 私有化时。
`release`|每次 `release` 被发布到 `Repository` 中时
`status` |每次从API中更新版本库状态时。
`team`   |任何时候创建，删除，修改 `Repository` 的团队。 仅组织钩子
`team_add`|每次添加或修改 `Repository` 中的 team 成员时。
`watch`  |每次用户对`Repository`加星标时。

## 有效载荷
每个事件类型具有特定的有效载荷格式以及相关的事件信息。 即，不同的事件被触发发送到我们配置的 `URL` 的 `POST` 数据格式和内容也会不同。

例如，一个 `commit_comment` 事件如果被触发，将会向 URL 发送类似下面的内容。

```
{
  "action": "created",
  "comment": {
    "url": "https://api.github.com/repos/baxterthehacker/public-repo/comments/11056394",
    "html_url": "https://github.com/baxterthehacker/public-repo/commit/9049f1265b7d61be4a8904a9a27120d2064dab3b#commitcomment-11056394",
    "id": 11056394,
    "user": {
      "login": "baxterthehacker",
      "id": 6752317,
      "avatar_url": "https://avatars.githubusercontent.com/u/6752317?v=3",
      "gravatar_id": "",
      "url": "https://api.github.com/users/baxterthehacker",
      "html_url": "https://github.com/baxterthehacker",
      "followers_url": "https://api.github.com/users/baxterthehacker/followers",
      "following_url": "https://api.github.com/users/baxterthehacker/following{/other_user}",
      "gists_url": "https://api.github.com/users/baxterthehacker/gists{/gist_id}",
      "starred_url": "https://api.github.com/users/baxterthehacker/starred{/owner}{/repo}",
      "subscriptions_url": "https://api.github.com/users/baxterthehacker/subscriptions",
      "organizations_url": "https://api.github.com/users/baxterthehacker/orgs",
      "repos_url": "https://api.github.com/users/baxterthehacker/repos",
      "events_url": "https://api.github.com/users/baxterthehacker/events{/privacy}",
      "received_events_url": "https://api.github.com/users/baxterthehacker/received_events",
      "type": "User",
      "site_admin": false
    },
    "position": null,
    "line": null,
    "path": null,
    "commit_id": "9049f1265b7d61be4a8904a9a27120d2064dab3b",
    "created_at": "2015-05-05T23:40:29Z",
    "updated_at": "2015-05-05T23:40:29Z",
    "body": "This is a really good change! :+1:"
  },
  "repository": {
    "id": 35129377,
    "name": "public-repo",
    "full_name": "baxterthehacker/public-repo",
    "owner": {
      "login": "baxterthehacker",
      "id": 6752317,
      "avatar_url": "https://avatars.githubusercontent.com/u/6752317?v=3",
      "gravatar_id": "",
      "url": "https://api.github.com/users/baxterthehacker",
      "html_url": "https://github.com/baxterthehacker",
      "followers_url": "https://api.github.com/users/baxterthehacker/followers",
      "following_url": "https://api.github.com/users/baxterthehacker/following{/other_user}",
      "gists_url": "https://api.github.com/users/baxterthehacker/gists{/gist_id}",
      "starred_url": "https://api.github.com/users/baxterthehacker/starred{/owner}{/repo}",
      "subscriptions_url": "https://api.github.com/users/baxterthehacker/subscriptions",
      "organizations_url": "https://api.github.com/users/baxterthehacker/orgs",
      "repos_url": "https://api.github.com/users/baxterthehacker/repos",
      "events_url": "https://api.github.com/users/baxterthehacker/events{/privacy}",
      "received_events_url": "https://api.github.com/users/baxterthehacker/received_events",
      "type": "User",
      "site_admin": false
    },
    "private": false,
    "html_url": "https://github.com/baxterthehacker/public-repo",
    "description": "",
    "fork": false,
    "url": "https://api.github.com/repos/baxterthehacker/public-repo",
    "forks_url": "https://api.github.com/repos/baxterthehacker/public-repo/forks",
    "keys_url": "https://api.github.com/repos/baxterthehacker/public-repo/keys{/key_id}",
    "collaborators_url": "https://api.github.com/repos/baxterthehacker/public-repo/collaborators{/collaborator}",
    "teams_url": "https://api.github.com/repos/baxterthehacker/public-repo/teams",
    "hooks_url": "https://api.github.com/repos/baxterthehacker/public-repo/hooks",
    "issue_events_url": "https://api.github.com/repos/baxterthehacker/public-repo/issues/events{/number}",
    "events_url": "https://api.github.com/repos/baxterthehacker/public-repo/events",
    "assignees_url": "https://api.github.com/repos/baxterthehacker/public-repo/assignees{/user}",
    "branches_url": "https://api.github.com/repos/baxterthehacker/public-repo/branches{/branch}",
    "tags_url": "https://api.github.com/repos/baxterthehacker/public-repo/tags",
    "blobs_url": "https://api.github.com/repos/baxterthehacker/public-repo/git/blobs{/sha}",
    "git_tags_url": "https://api.github.com/repos/baxterthehacker/public-repo/git/tags{/sha}",
    "git_refs_url": "https://api.github.com/repos/baxterthehacker/public-repo/git/refs{/sha}",
    "trees_url": "https://api.github.com/repos/baxterthehacker/public-repo/git/trees{/sha}",
    "statuses_url": "https://api.github.com/repos/baxterthehacker/public-repo/statuses/{sha}",
    "languages_url": "https://api.github.com/repos/baxterthehacker/public-repo/languages",
    "stargazers_url": "https://api.github.com/repos/baxterthehacker/public-repo/stargazers",
    "contributors_url": "https://api.github.com/repos/baxterthehacker/public-repo/contributors",
    "subscribers_url": "https://api.github.com/repos/baxterthehacker/public-repo/subscribers",
    "subscription_url": "https://api.github.com/repos/baxterthehacker/public-repo/subscription",
    "commits_url": "https://api.github.com/repos/baxterthehacker/public-repo/commits{/sha}",
    "git_commits_url": "https://api.github.com/repos/baxterthehacker/public-repo/git/commits{/sha}",
    "comments_url": "https://api.github.com/repos/baxterthehacker/public-repo/comments{/number}",
    "issue_comment_url": "https://api.github.com/repos/baxterthehacker/public-repo/issues/comments{/number}",
    "contents_url": "https://api.github.com/repos/baxterthehacker/public-repo/contents/{+path}",
    "compare_url": "https://api.github.com/repos/baxterthehacker/public-repo/compare/{base}...{head}",
    "merges_url": "https://api.github.com/repos/baxterthehacker/public-repo/merges",
    "archive_url": "https://api.github.com/repos/baxterthehacker/public-repo/{archive_format}{/ref}",
    "downloads_url": "https://api.github.com/repos/baxterthehacker/public-repo/downloads",
    "issues_url": "https://api.github.com/repos/baxterthehacker/public-repo/issues{/number}",
    "pulls_url": "https://api.github.com/repos/baxterthehacker/public-repo/pulls{/number}",
    "milestones_url": "https://api.github.com/repos/baxterthehacker/public-repo/milestones{/number}",
    "notifications_url": "https://api.github.com/repos/baxterthehacker/public-repo/notifications{?since,all,participating}",
    "labels_url": "https://api.github.com/repos/baxterthehacker/public-repo/labels{/name}",
    "releases_url": "https://api.github.com/repos/baxterthehacker/public-repo/releases{/id}",
    "created_at": "2015-05-05T23:40:12Z",
    "updated_at": "2015-05-05T23:40:12Z",
    "pushed_at": "2015-05-05T23:40:27Z",
    "git_url": "git://github.com/baxterthehacker/public-repo.git",
    "ssh_url": "git@github.com:baxterthehacker/public-repo.git",
    "clone_url": "https://github.com/baxterthehacker/public-repo.git",
    "svn_url": "https://github.com/baxterthehacker/public-repo",
    "homepage": null,
    "size": 0,
    "stargazers_count": 0,
    "watchers_count": 0,
    "language": null,
    "has_issues": true,
    "has_downloads": true,
    "has_wiki": true,
    "has_pages": true,
    "forks_count": 0,
    "mirror_url": null,
    "open_issues_count": 2,
    "forks": 0,
    "open_issues": 2,
    "watchers": 0,
    "default_branch": "master"
  },
  "sender": {
    "login": "baxterthehacker",
    "id": 6752317,
    "avatar_url": "https://avatars.githubusercontent.com/u/6752317?v=3",
    "gravatar_id": "",
    "url": "https://api.github.com/users/baxterthehacker",
    "html_url": "https://github.com/baxterthehacker",
    "followers_url": "https://api.github.com/users/baxterthehacker/followers",
    "following_url": "https://api.github.com/users/baxterthehacker/following{/other_user}",
    "gists_url": "https://api.github.com/users/baxterthehacker/gists{/gist_id}",
    "starred_url": "https://api.github.com/users/baxterthehacker/starred{/owner}{/repo}",
    "subscriptions_url": "https://api.github.com/users/baxterthehacker/subscriptions",
    "organizations_url": "https://api.github.com/users/baxterthehacker/orgs",
    "repos_url": "https://api.github.com/users/baxterthehacker/repos",
    "events_url": "https://api.github.com/users/baxterthehacker/events{/privacy}",
    "received_events_url": "https://api.github.com/users/baxterthehacker/received_events",
    "type": "User",
    "site_admin": false
  }
}
```

其它事件的内容和格式，参考(https://developer.github.com/v3/activity/events/types/)

**注意：** 有效负载上限为`5 MB`。 如果我们的事件产生一个更大的有效载荷，`webhook` 将不会被触发。 例如，如果同时推送了多个分支或标签，则可能会发生在创建事件上。 所以建议监控我们的有效负载大小以确保 `webhooks` 能够被触发。

### 请求Header
对 `Webhook` 配置的 `URL` 端点发出的 `HTTP` 请求将包含几个特殊`Header`头：

`Header`	|`Description`
--------|-----------
`X-GitHub-Event`	|触发本次请求的事件的名称。
`X-Hub-Signature`	|HMAC对有效负载的十六进制摘要，使用钩子的密钥作为密钥（如果已配置）。
`X-GitHub-Delivery`|本次请求的唯一ID。

此外，请求的`User-Agent`将具有前缀`GitHub-Hookshot/`。


请求示例
```
POST /payload HTTP/1.1

Host: localhost:4567
X-Github-Delivery: 72d3162e-cc78-11e3-81ab-4c9367dc0958
User-Agent: GitHub-Hookshot/044aadd
Content-Type: application/json
Content-Length: 6615
X-GitHub-Event: issues

{
  "action": "opened",
  "issue": {
    "url": "https://api.github.com/repos/octocat/Hello-World/issues/1347",
    "number": 1347,
    ...
  },
  "repository" : {
    "id": 1296269,
    "full_name": "octocat/Hello-World",
    "owner": {
      "login": "octocat",
      "id": 1,
      ...
    },
    ...
  },
  "sender": {
    "login": "octocat",
    "id": 1,
    ...
  }
}

```

## `Ping`事件
当我们创建新的 `webhooks` 时，`Github` 会向我们配置的 `URL` 发送一个简单的 `ping` 事件，以检测我们配置的 `URL` 是否正确可用。 该事件未存储，因此无法通过`Events API`检索。 可以通过调用`ping`端点再次触发`ping`。

`Ping`事件有效负载

键|值
--|--
`zen` |`GitHub`产生的随机字符串
`hook_id`|触发`ping`的`webhook`的`ID`
`hook` |`webhook`配置
