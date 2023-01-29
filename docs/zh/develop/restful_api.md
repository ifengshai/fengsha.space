---
title: restful_api
date: 2023-01-28 05:25:48
permalink: /pages/cabc7f/
categories:
  - zh copy
  - develop
tags:
  - 
author: 
  name: xugaoyi
  link: https://github.com/xugaoyi
---
# Restful API 设计指南
Restful架构就是目前比较流行的一种互联网架构,它结构清晰,符合标准,易于理解,方便扩展
## 一：起源
REST这个词，是Roy Thomas Fielding在他2000年的博士论文中提出的。
Fielding是一个非常重要的人，他是HTTP协议（1.0版和1.1版）的主要设计者、Apache服务器软件的作者之一、Apache基金会的第一任主席。所以，他的这篇论文一经发表，就引起了关注，并且立即对互联网开发产生了深远的影响。

他这样介绍论文的写作目的
“本文研究计算机科学两大前沿----软件和网络----的交叉点。长期以来，软件研究主要关注
软件设计的分类、设计方法的演化，很少客观地评估不同的设计选择对系统行为的影响。而相反地，网络研究主要关注系统之间通信行为的细节、如何改进特定通信机制的表现，常常忽视了一个事实，那就是改变应用程序的互动风格比改变互动协议，对整体表现有更大的影响。我这篇文章的写作目的，就是想在符合架构原理的前提下，理解和评估以网络为基础的应用软件的架构设计，得到一个功能强、性能好、适宜通信的架构。”

## 二：名称
Fielding将他对这种互联网软件的架构原则命名为 Rest,即 Representtational state Transfer,翻译为:“表现层状态转化”.
如果一个架构符合Rest原则,就称它为Restful架构.
要理解RESTful架构，最好的方法就是去理解Representational State Transfer这个词组到底是什么意思，它的每一个词代表了什么涵义。如果你把这个名称搞懂了，也就不难体会REST是一种什么样的设计。

### Resources(资源)
Rest(表现层状态转化)名称中省略了主语,表现层指的就是"资源"的变现层.
所谓资源,就是网络上一个实体,他可以是一段文本,一个图片,一首歌曲,一个服务…,总之就是一个具体的存在,你可以使用一个URI(统一资源定位符)来指向它.每种资源对应一种特定的URI,要访问一种资源,只需要直到指向他的URI就可以了.因此,URI就成了每一种资源的地址和独一无二的标识符.
所谓上网,就是与互联网上的一切资源进行互动,调用他的URI.

### Representation(表现层)
“资源” 是一种信息实体,它有多种表现形式,我们把"资源"具体呈现出来的形式，叫做它的"表现层"（Representation）。
比如，文本可以用txt格式表现，也可以用HTML格式、XML格式、JSON格式表现，甚至可以采用二进制格式；图片可以用JPG格式表现，也可以用PNG格式表现。
URI只代表资源实体不代表它的表现形式.严格来说,有些网址后面的 .html是不必要的.因为这个后缀表示格式,属于 表现层范畴,而URI应该只代表资源的位置.它的具体表现形式应该在HTTP请求头中,使用Accept和Content-Type来指定,这两个字段才是对表现层的描述.

### State Transfer(状态转化)
访问一个网站,就代表了客户端和服务端的一次互动,在这个互动中势必涉及到数据状态的转化.
互联网通信协议HTTP是一个无状态的协议,这意味着所有的状态都保存在服务端,因此，如果客户端想要操作服务器，必须通过某种手段，让服务器端发生"状态转化"（State Transfer）。而这种转化是建立在表现层之上的，所以就是"表现层状态转化"。

具体来说,就是HTTP协议里面四个表示操作方式的动词: GET,POST,PUT,DELETE.他们分别对应四种基本操作:GET用来获取资源，POST用来新建资源（也可以用于更新资源），PUT用来更新资源，DELETE用来删除资源。

综述
1. 每一个URI代表一种资源.
2. 客户端和服务器之间，传递这种资源的某种表现层
3. 客户端通过四个HTTP动词，对服务器端资源进行操作，实现"表现层状态转化"。

## 三：误区
RESTful架构有一些典型的设计误区。

最常见的一种设计错误，就是URI包含动词。因为"资源"表示一种实体，所以应该是名词，URI不应该有动词，动词应该放在HTTP协议中。

举例来说，某个URI是/posts/show/1，其中show是动词，这个URI就设计错了，正确的写法应该是/posts/1，然后用GET方法表示show。

如果某些动作是HTTP动词表示不了的，你就应该把动作做成一种资源。比如网上汇款，从账户1向账户2汇款500元，错误的URI是：

　　POST /accounts/1/transfer/500/to/2

正确的写法是把动词transfer改成名词transaction，资源不能是动词，但是可以是一种服务：

　　POST /transaction HTTP/1.1
　　Host: 127.0.0.1
　　
　　from=1&to=2&amount=500.00

另一个设计误区，就是在URI中加入版本号：

　　http://www.example.com/app/1.0/foo

　　http://www.example.com/app/1.1/foo

　　http://www.example.com/app/2.0/foo

因为不同的版本，可以理解成同一种资源的不同表现形式，所以应该采用同一个URI。版本号可以在HTTP请求头信息的Accept字段中进行区分（参见Versioning REST Services）：

　　Accept: vnd.example-com.foo+json; version=1.0

　　Accept: vnd.example-com.foo+json; version=1.1

Accept: vnd.example-com.foo+json; version=2.0
## 四：协议
API与用户的通信协议，总是使用HTTPS协议。
## 五：域名
应该尽量将API部署在专用域名之下。
https://api.example.com
如果确定API很简单，不会有进一步扩展，可以考虑放在主域名下。
https://example.org/api/
## 六：版本（Versioning）
应该将API的版本号放入URL。
https://api.example.com/v1/
另一种做法是，将版本号放在HTTP头信息中，但不如放入URL方便和直观。Github采用这种做法。

## 七：路径（Endpoint）
路径又称"终点"（endpoint），表示API的具体网址。

在RESTful架构中，每个网址代表一种资源（resource），所以网址中不能有动词，只能有名词，而且所用的名词往往与数据库的表格名对应。一般来说，数据库中的表都是同种记录的"集合"（collection），所以API中的名词也应该使用复数。

举例来说，有一个API提供动物园（zoo）的信息，还包括各种动物和雇员的信息，则它的路径应该设计成下面这样。

https://api.example.com/v1/zoos
https://api.example.com/v1/animals
https://api.example.com/v1/employees

## 八：HTTP动词
对于资源的具体操作类型，由HTTP动词表示。
常用的HTTP动词有下面五个（括号里是对应的SQL命令）。
GET（SELECT）：从服务器取出资源（一项或多项）。
POST（CREATE）：在服务器新建一个资源。
PUT（UPDATE）：在服务器更新资源（客户端提供改变后的完整资源）。
PATCH（UPDATE）：在服务器更新资源（客户端提供改变的属性）。
DELETE（DELETE）：从服务器删除资源。

还有两个不常用的HTTP动词。
HEAD：获取资源的元数据。
OPTIONS：获取信息，关于资源的哪些属性是客户端可以改变的。
下面是一些例子。

GET /zoos：列出所有动物园
POST /zoos：新建一个动物园
GET /zoos/ID：获取某个指定动物园的信息
PUT /zoos/ID：更新某个指定动物园的信息（提供该动物园的全部信息）
PATCH /zoos/ID：更新某个指定动物园的信息（提供该动物园的部分信息）
DELETE /zoos/ID：删除某个动物园
GET /zoos/ID/animals：列出某个指定动物园的所有动物
DELETE /zoos/ID/animals/ID：删除某个指定动物园的指定动物

## 九：过滤信息（Filtering）
如果记录数量很多，服务器不可能都将它们返回给用户。API应该提供参数，过滤返回结果。

下面是一些常见的参数。

?limit=10：指定返回记录的数量
?offset=10：指定返回记录的开始位置。
?page=2&per_page=100：指定第几页，以及每页的记录数。
?sortby=name&order=asc：指定返回结果按照哪个属性排序，以及排序顺序。
?animal_type_id=1：指定筛选条件
参数的设计允许存在冗余，即允许API路径和URL参数偶尔有重复。比如，GET /zoo/ID/animals 与 GET /animals?zoo_id=ID 的含义是相同的。

## 十：状态码
服务器向用户返回的状态码和提示信息，常见的有以下一些（方括号中是该状态码对应的HTTP动词）。

200 OK - [GET]：服务器成功返回用户请求的数据，该操作是幂等的（Idempotent）。
201 CREATED - [POST/PUT/PATCH]：用户新建或修改数据成功。
202 Accepted - [*]：表示一个请求已经进入后台排队（异步任务）
204 NO CONTENT - [DELETE]：用户删除数据成功。
400 INVALID REQUEST - [POST/PUT/PATCH]：用户发出的请求有错误，服务器没有进行新建或修改数据的操作，该操作是幂等的。
401 Unauthorized - [*]：表示用户没有权限（令牌、用户名、密码错误）。
403 Forbidden - [*] 表示用户得到授权（与401错误相对），但是访问是被禁止的。
404 NOT FOUND - [*]：用户发出的请求针对的是不存在的记录，服务器没有进行操作，该操作是幂等的。
406 Not Acceptable - [GET]：用户请求的格式不可得（比如用户请求JSON格式，但是只有XML格式）。
410 Gone -[GET]：用户请求的资源被永久删除，且不会再得到的。
422 Unprocesable entity - [POST/PUT/PATCH] 当创建一个对象时，发生一个验证错误。
500 INTERNAL SERVER ERROR - [*]：服务器发生错误，用户将无法判断发出的请求是否成功。
w3c状态码标准：https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html

## 十一：错误处理（Error handling）
如果状态码是4xx，就应该向用户返回出错信息。一般来说，返回的信息中将error作为键名，出错信息作为键值即可。
{
    error: "Invalid API key"
}

## 十二：Hypermedia API
RESTful API最好做到Hypermedia，即返回结果中提供链接，连向其他API方法，使得用户不查文档，也知道下一步应该做什么。

比如，当用户向api.example.com的根目录发出请求，会得到这样一个文档。
{"link": {
  "rel":   "collection https://www.example.com/zoos",
  "href":  "https://api.example.com/zoos",
  "title": "List of zoos",
  "type":  "application/vnd.yourformat+json"
}}
上面代码表示，文档中有一个link属性，用户读取这个属性就知道下一步该调用什么API了。rel表示这个API与当前网址的关系（collection关系，并给出该collection的网址），href表示API的路径，title表示API的标题，type表示返回类型。
Hypermedia API的设计被称为HATEOAS。Github的API就是这种设计，访问api.github.com会得到一个所有可用API的网址列表。
{
  "current_user_url": "https://api.github.com/user",
  "authorizations_url": "https://api.github.com/authorizations",
  // ...
}
从上面可以看到，如果想获取当前用户的信息，应该去访问api.github.com/user，然后就得到了下面结果。
{
  "message": "Requires authentication",
  "documentation_url": "https://developer.github.com/v3"
}
上面代码表示，服务器给出了提示信息，以及文档的网址。

## 十三：其他
（1）API的身份认证应该使用OAuth 2.0框架。

（2）服务器返回的数据格式，应该尽量使用JSON，避免使用XML。

（完）
