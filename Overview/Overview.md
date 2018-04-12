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

![Unauthorized](unauthenticated)

在短时间内检测到多次无效的身份认证之后，`API` 将会暂时的拒绝该用户的所有认证（包括有效的认证）并且会返回`403 Forbidden`错误

#### 参数

很多`API`方法才去可选参数。以`GET`请求为例，任何没有在路径中分割的参数都可以作为`HTTP` 查询字符串参数

	curl -i "https://api.github.com/repos/vmg/redcarpet/issues?state=closed"

对于 `POST`，`patch`，`PUT`，和`DELETE`请求来说，参数不能出现在`URL`中而且还要以JSON的形式并且`Content-Type`为`application/json`