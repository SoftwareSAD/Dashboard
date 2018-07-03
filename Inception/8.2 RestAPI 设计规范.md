# REST API设计规范

### 什么是REST？

REST作为Representation State Transfer的缩写，意为“表现层状态转化”。
其最具标志性的特征为面向资源，资源是针对服务器的抽象概念，依据REST要求，必须通过统一的接口来对资源执行各种操作，对于每一个资源只能执行一组有限操作。

### 什么是REST API？

REST API即是符合REST架构的接口设计。

### REST API接口规范

 **1. 动作**（括号中对应数据库命令）
 
```
 GET(SELECT)：从服务器检索特定资源或资源列表
 POST(CREATE)：在服务器上创建新资源
 PUT(UPDATE)：更新服务器上的资源，提供整个资源
 PATCH(UPDATE)：更新资源，仅提供更改的属性
 DELETE(DELETE)：删除资源
 HEAD：获取资源元数据，例如上一次的更新时间
 OPTIONS：检索关于客户端被允许对资源做什么的信息
```

 **2. 路径**
 
 路径表示API的具体网址，每个网址代表一种资源，只能用名词命名，且该名词常与数据库表格名称相对应，而表作为同种记录的集合，因此名词常用复数表示。
 
 以某API提供公司（Company）的信息为例：
 
```
 GET　　　/companies：列出所有公司
 POST　 　/companies：新建一个公司
 GET　　　/companies/ID：获取某指定公司的信息
 PUT　　　/companies/ID：更新某指定公司的信息（全部）
 PATCH　　/companies/ID：更新某指定公司的信息（部分）
 DELETE　　/companies/ID：删除某公司
 GET　　　/companies/ID/staffs：列出某指定公司的所有职员
```
 
 反例：
 
```
 GET　　　/getCompanies （出现动词） 
 POST　 　/createCompany  （出现动词和单数）
 DELETE　　/deleteCompanies/ID  （出现动词）
``` 

 **3. 版本**
 
 将API版本号放入URL：
 
```
 http://api.system.com/v3/
```
 
 **4. 过滤信息**
 
 若记录数较多，服务器需经过过滤返回结果，这是需要API提供相应参数，常用参数如下：
 
``` 
 ?limit=10：指定返回记录数
 ?offset=30：指定返回记录的开始位置（偏移量）
 ?page_number=1&page_number=50：按指定页数返回对应记录
 ?sortby=id&order=asc：指定返回列表的排序索引及顺序
 ?company_id=6：设定筛选条件
```

参数的设计允许存在冗余，如：

```
 GET　　　/companies/ID/staffs
 等价于
 GET　　　/staffs?company_id=ID
```

 **5. 状态码**
 
 状态码范围：
 
```
 1xx 信息，请求收到，继续处理
 2xx 成功，行为被成功地接受、理解和采纳
 3xx 重定向，为了完成请求，必须进一步执行的动作
 4xx 客户端错误，请求包含语法错误或者请求无法实现
 5xx 范围的状态码是保留给服务器端错误用的。这些错误常常由底层的函数抛出，甚至开发人员也没法处理，发送这类状态码的目的是确保客户端获得某种响应。由于客户端无从知晓服务器此时状态，因此这类状态码应尽量避免
```

常见状态码：

```
 200 OK - [GET]：服务器成功返回用户请求的数据
 201 CREATED - [POST/PUT/PATCH]：用户新建或修改数据成功
 202 ACCEPTED - [*]：表示一个请求已经进入后台排队
 204 NO CONTENT - [DELETE]：用户删除数据成功
 400 INVALID REQUEST - [POST/PUT/PATCH]：用户发出错误请求，服务器拒绝新建或修改数据
 404 NOT FOUND - [*]：用户针对不存在的记录发出请求，服务器未发出操作
 502 网关错误
 ...
```

**6. 错误处理**

如果状态码是4xx，就应该向用户返回出错信息。一般情况下返回信息的键名为error，键值为错误信息，如：

```
{
    "error": "NOT FOUND"
    ...
}
```

**7. Hypermedia API**（超文本）

REST API 能够做到向返回结果提供链接，连向其他API。

如，当用户向api.system.com的根目录发出请求，会得到这样一个文档。

```
{
    ...
    "link": {
        "rel":"collection https://www.example/companies",
        "href":"https://api.system.com/companies",
        "title":"List of Companies",
        "type":"application/vnd.yourformat+json"
    }
    ...
}
```
其中link属性为用户明确下一步的API调用，rel表示此API与当前网址的关系，href为API路径，title为API标题，type为返回类型。
