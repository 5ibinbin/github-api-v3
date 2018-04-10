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

