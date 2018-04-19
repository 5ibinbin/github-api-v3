### 概述

##### 这边文章描述了官方Github REST API V3的组成部分。如果有任何疑问或问题请联系[GitHub](https://github.com/contact)

-    i. Current version
-   ii. Schema
-  iii. Authentication 
-   iv. Parameters
-    v. Root endpoint
-   vi. GraphQL global relay IDs
-  vii. Client errors
- viii. Http redirects
-   ix. Http verbs
-    x. Hypermedia
-   xi. Pagination
-  xii. Rate limiting
- xiii. User agent required
-  xiv. Conditional requests
-   xv. Cross origin resource sharing
-  xvi. JSON-P callbacks
- xvii. Timezones

#### 当前版本

默认情况下，所有对`https://api.github.com`的请求都会受到`REST API`的v3版本。我们鼓励你通过添加 `header`里面的参数`Accept`来明确的请求这个版本。类似于下面这样

	Accept: application/vnd.github.v3+json

更多有关`GitHub's GraphQL API v4`的相关信息，请移步到[这里](https://developer.github.com/v4/)

#### 结构

所有的`API`的访问都是通过`HTTPS`访问，并通过`https://api.github.com`进行请求，所有请求或者返回的数据都是`JSON`格式的数据。

![schema](https://github.com/5ibinbin/github-api-v3/blob/master/img/current1.png)

空白区域用`null`代替而不是省略

所有的时间戳会以`ISO 8601`格式返回:

	YYYY-MM-DDTHH:MM:SSZ

更多有关时间戳的时区问题，请移步`timezones`模块

###### 汇总显示

当我们想获取资源列表时，返回数据会包含某个资源的一部分属性。这就是资源的摘要显示。(某些属性对应当前列表API来说是很费时的。出于性能方面的考虑，列表接口要排除这些属性的显示。如果要获得这些属性，可以通过详情`API`来获取)。

**示例**：当你想获取一个仓库(`Github repository`)列表时，你可以显示每个仓库的摘要信息，下面我们以获取[octokit](https://github.com/octokit)的仓库列表为例：

	GET /orgs/octokit/repos
	
###### 详情显示

一般情况下，当你想获取单个的资源时，我们会返回给你当前资源的所有属性。这就是资源的详情展示。（注意：在资源没有获得授权时，我们可能只会显示一部分数据）。

**示例**：下面我们以获取[octokit/octokit.rb](https://github.com/octokit/octokit.rb)的具体信息为例，来告诉大家如何获取单个仓库的全部信息显示：

	GET /repos/octokit/octokit.rb
	
我们为每个`API`方法都提供了一个返回数据，返回数据展示了该方法返回的所有属性。

#### 认证

有三种方式可以通过`GitHub API v3`进行身份验证。某些情况下，请求需要认证的接口会返回`404 Not Found`，而不是`403 Forbidden`。这是问了避免无意泄露私有仓库的信息给未被认证的用户。

###### 基础认证

	curl -u "username" https://api.github.com
	
###### OAuth2 token(请求头部)

	curl -H "Authorization: token OAUTH-TOKEN" https://api.github.com
	
###### OAuth2 token(请求参数中)

	curl https://api.github.com/?access_token=OAUTH-TOKEN
	
更多`OAuth2 `请移步[这里](https://developer.github.com/apps/building-oauth-apps/)，注意：我们可以以编程的方式获取`OAuth2 Token`，而不是网站的应用程序。

###### OAuth2 key/secret

	curl 'https://api.github.com/users/whatever?client_id=xxxx&client_secret=yyyy'
	
这种应用场景应该是服务器到服务器的通讯，不要泄露你的`OAuth2`给你的客户端使用者。

有关更多没有认证的限制，请移步[这里](https://developer.github.com/v3/#increasing-the-unauthenticated-rate-limit-for-oauth-applications)

###### 登录失败情形
	
无效的身份认证将会返回`401 Unauthorized`

![Unauthorized](https://github.com/5ibinbin/github-api-v3/blob/master/img/unAuthorized.png)

在短时间内检测到多次无效的身份认证之后，`API` 将会暂时的拒绝该用户的所有认证（包括有效的认证）并且会返回`403 Forbidden`错误

![Forbidden](https://github.com/5ibinbin/github-api-v3/blob/master/img/Forbidden.png)

#### 参数

很多`API`方法才去可选参数。以`GET`请求为例，任何没有在路径中分割的参数都可以作为`HTTP` 查询字符串参数

	curl -i "https://api.github.com/repos/vmg/redcarpet/issues?state=closed"

在这个例子中，`vmg`和`redcarpet`是用来提供路径中对应的`:owner`和`:repo`参数而`:state`是在查询字符串中传递

对于 `POST`，`patch`，`PUT`，和`DELETE`请求来说，参数不能出现在`URL`中而且还要以JSON的形式并且`Content-Type`为`application/json`

	curl -i -u username -d '{"scopes":["public_repo"]}' https://api.github.com/authorizations

#### Root endpoint(实在不知道怎么翻译)

你可以发送一个`GET`请求来查看`REST API V3`支持的根节点的返回类型：

	curl https://api.github.com
	
#### GraphQL 全局ID

当准备迁移到[GraphQL API v4](https://developer.github.com/v4/)，你可以使用`jean-grey`来暴露使用`REST API V3`来获取的很多资源的`GraphQL ID `

	application/vnd.github.jean-grey-preview+json
	
响应的数据将会包含一个`node_id`字段

例如，如果你请求一个[用户认证接口](https://developer.github.com/v3/users/#get-the-authenticated-user)

	curl -i -u username:token
	https://api.github.com/user  

你将会获得一个包含`node_id`的认证用户的响应数据

```
Status: 200 OK
{
  "login": "octocat",
  "id": 1,
  "avatar_url": "https://github.com/images/error/octocat_happy.gif",
  "gravatar_id": "",
  "url": "https://api.github.com/users/octocat",
  "html_url": "https://github.com/octocat",
  "followers_url": "https://api.github.com/users/octocat/followers",
  "following_url": "https://api.github.com/users/octocat/following{/other_user}",
  "gists_url": "https://api.github.com/users/octocat/gists{/gist_id}",
  "starred_url": "https://api.github.com/users/octocat/starred{/owner}{/repo}",
  "subscriptions_url": "https://api.github.com/users/octocat/subscriptions",
  "organizations_url": "https://api.github.com/users/octocat/orgs",
  "repos_url": "https://api.github.com/users/octocat/repos",
  "events_url": "https://api.github.com/users/octocat/events{/privacy}",
  "received_events_url": "https://api.github.com/users/octocat/received_events",
  "type": "User",
  "site_admin": false,
  "name": "monalisa octocat",
  "company": "GitHub",
  "blog": "https://github.com/blog",
  "location": "San Francisco",
  "email": "octocat@github.com",
  "hireable": false,
  "bio": "There once was...",
  "public_repos": 2,
  "public_gists": 1,
  "followers": 20,
  "following": 0,
  "created_at": "2008-01-14T04:33:35Z",
  "updated_at": "2008-01-14T04:33:35Z",
  "total_private_repos": 100,
  "owned_private_repos": 100,
  "private_gists": 81,
  "disk_usage": 10000,
  "collaborators": 8,
  "two_factor_authentication": true,
  "plan": {
    "name": "Medium",
    "space": 400,
    "private_repos": 20,
    "collaborators": 0
  },
  "node_id": "MDQ6VXNlcjU4MzIzMQ=="
}
```

你可以使用`node_id `的值(本例中的`MDQ6VXNlcjU4MzIzMQ==`)来为`GraphQL API v4`提供一个具体的[node](https://developer.github.com/v4/guides/intro-to-graphql/#node)。只需要提供`node ID`，你就可以访问底层类型的数据。

你也许会发现使用[inline fragments](http://graphql.org/learn/queries/#inline-fragments)这种方式更加简单，例如：

[点此查看运行效果](https://developer.github.com/v4/explorer/?variables=%7B%7D&query=query%20%7B%0A%20%20node%28id%3A%22MDQ6VXNlcjU4MzIzMQ%3D%3D%22%29%20%7B%0A%20%20%20...%20on%20User%20%7B%0A%20%20%20%20%20%20id%0A%20%20%20%20%20%20name%0A%20%20%20%20%20%20login%0A%20%20%20%20%7D%0A%20%20%7D%0A%7D)

```
query {
  node(id:"MDQ6VXNlcjU4MzIzMQ==") {
   ... on User {
      id
      name
      login
    }
  }
}
```

想查看在`REST API v3 `资源列表下`API`返回的数据中包含`node ID`，请移步到[这里](https://developer.github.com/changes/2017-12-19-graphql-node-id/)

当我们构建使用`REST API v3` 或`GraphQL API v4` 的工具和集合是，最好去持久化这个 `ID`的值，以便我们可以更方便的从其他的API版本获取参考值。要查看更多的信息，请查看[Migrating from REST to GraphQL](https://developer.github.com/v4/guides/migrating-from-rest/)。

#### 客户端错误

在接收请求体的`API`的调用中，可能会有三种类型的客户端错误：

######1. 发送无效的`JSON`会返回`400 Bad Request`错误

```
HTTP/1.1 400 Bad Request
Content-Length: 35
	
{"message":"Problems parsing JSON"}
```
	
######2. 发送错误的`JSON`类型会返回`400 Bad Request`错误

```
HTTP/1.1 400 Bad Request
Content-Length: 40

{"message":"Body should be a JSON object"}
```

######3. 发送无效的字段会返回`422 Unprocessable Entity`

```
HTTP/1.1 422 Unprocessable Entity
Content-Length: 149

{
  "message": "Validation Failed",
  "errors": [
    {
      "resource": "Issue",
      "field": "title",
      "code": "missing_field"
    }
  ]
}
```

所有的错误对象都有资源和字段属性以便以你能够发现问题所在。而且还会有错误码来帮助你知晓是什么错误。下面是可能的验证错误码：

- `missing：This means a resource does not exist.`
- `missing_field : This means a required field on a resource has not been set.`
- `invalid ：This means the formatting of a field is invalid. The documentation for that resource should be able to give you more specific information.`
- `already_exists ：This means another resource has the same value as this field. This can happen in resources that must have some unique key (such as Label names).`

响应数据有时也会返回自定义的错误码（当`code`是自定义的时候）。错误信息通常会有一个描述错误的消息字段，而且大多数错误都会包含一个帮你解决错误的指定的`documentation_url`。
