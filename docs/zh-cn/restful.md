## REST (表达性状态转移)

## restful(接口设计规范)

- **GET**（SELECT）：从服务器取出资源（一项或多项）。
- **POST**（CREATE）：在服务器新建一个资源。
- **PUT**（UPDATE）：在服务器更新资源（客户端提供完整资源数据）。
- **PATCH**（UPDATE）：在服务器更新资源（客户端提供需要修改的资源数据）。
- **DELETE**（DELETE）：从服务器删除资源。

RESTful是面向资源的，每种资源可能由一个或多个URI对应，但一个URI只指向一种资源。

**「接口尽量使用名词，禁止使用动词。」** 下面是一些例子：

```text
GET    /classes：列出所有班级
POST   /classes：新建一个班级
GET    /classes/classId：获取某个指定班级的信息
PUT    /classes/classId：更新某个指定班级的信息（一般倾向整体更新）
PATCH  /classes/classId：更新某个指定班级的信息（一般倾向部分更新）
DELETE /classes/classId：删除某个班级
GET    /classes/classId/teachers：列出某个指定班级的所有老师的信息
GET    /classes/classId/students：列出某个指定班级的所有学生的信息
DELETE classes/classId/teachers/ID：删除某个指定班级下的指定的老师的信息
```

## **REST架构限制条件**

Fielding在论文中提出REST架构的6个**限制条件**，也可称为RESTful 6大原则， 标准的REST约束应满足以下6个原则：

**客户端-服务端（Client-Server）**: 这个更专注客户端和服务端的分离，服务端独立可更好服务于前端、安卓、IOS等客户端设备。

**无状态（Stateless）**：服务端不保存客户端状态，客户端保存状态信息每次请求携带状态信息。

**可缓存性（Cacheability）** ：服务端需回复是否可以缓存以让客户端甄别是否缓存提高效率。

**统一接口（Uniform Interface）**：通过一定原则设计接口降低耦合，简化系统架构，这是RESTful设计的基本出发点。当然这个内容除了上述特点提到部分具体内容比较多详细了解可以参考这篇[REST论文内容](https://link.zhihu.com/?target=https%3A//www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm)。

**分层系统（Layered System）**：客户端无法直接知道连接的到终端还是中间设备，分层允许你灵活的部署服务端项目。

**按需代码（Code-On-Demand，可选）**：按需代码允许我们灵活的发送一些看似特殊的代码给客户端例如JavaScript代码。

REST架构的一些风格和限制条件就先介绍到这里，后面就对RESTful风格API具体介绍。

## **URL设计规范**

URL为统一资源定位器 ,接口属于服务端资源，首先要通过URL这个定位到资源才能去访问，而通常一个完整的URL组成由以下几个部分构成：

```
URI = scheme "://" host  ":"  port "/" path [ "?" query ][ "#" fragment ]
```

scheme: 指底层用的协议，如http、https、ftp
host: 服务器的IP地址或者域名
port: 端口，http默认为80端口
path: 访问资源的路径，就是各种web 框架中定义的route路由
query: 查询字符串，为发送给服务器的参数，在这里更多发送数据分页、排序等参数。
fragment: 锚点，定位到页面的资源

我们在设计API时URL的path是需要认真考虑的，而RESTful对path的设计做了一些规范，通常一个RESTful API的path组成如下：

```text
/{version}/{resources}/{resource_id}
```

version：API版本号，有些版本号放置在头信息中也可以，通过控制版本号有利于应用迭代。
resources：资源，RESTful API推荐用小写英文单词的**复数**形式。
resource_id：资源的id，访问或操作该资源。

## **状态码和返回数据**

服务端处理完成后客户端也可能不知道具体成功了还是失败了，服务器响应时，包含**状态码**和**返回数据**两个部分。

**状态码**

我们首先要正确使用各类状态码来表示该请求的处理执行结果。状态码主要分为五大类：

> 1xx：相关信息
> 2xx：操作成功
> 3xx：重定向
> 4xx：客户端错误
> 5xx：服务器错误

每一大类有若干小类，状态码的种类比较多，而主要常用状态码罗列在下面：

200 `OK - [GET]`：服务器成功返回用户请求的数据，该操作是幂等的（Idempotent）。
201 `CREATED - [POST/PUT/PATCH]`：用户新建或修改数据成功。
202 `Accepted - [*]`：表示一个请求已经进入后台排队（异步任务）
204 `NO CONTENT - [DELETE]`：用户删除数据成功。
400 `INVALID REQUEST - [POST/PUT/PATCH]`：用户发出的请求有错误，服务器没有进行新建或修改数据的操作，该操作是幂等的。
401 `Unauthorized - [*]`：表示用户没有权限（令牌、用户名、密码错误）。
403 `Forbidden - [*]` 表示用户得到授权（与401错误相对），但是访问是被禁止的。
404 `NOT FOUND - [*]`：用户发出的请求针对的是不存在的记录，服务器没有进行操作，该操作是幂等的。
406 `Not Acceptable - [GET]`：用户请求的格式不可得（比如用户请求JSON格式，但是只有XML格式）。
410 `Gone -[GET]`：用户请求的资源被永久删除，且不会再得到的。
422 `Unprocesable entity - [POST/PUT/PATCH]` 当创建一个对象时，发生一个验证错误。
500 `INTERNAL SERVER ERROR - [*]`：服务器发生错误，用户将无法判断发出的请求是否成功。



### 项目完整Git地址：

https://github.com/Mini-Bar/Springboot-restful.git

#### 引用

https://zhuanlan.zhihu.com/p/121522365

https://zhuanlan.zhihu.com/p/334809573