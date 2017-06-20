---
title: XSS
date: 2017-06-17 19:42:42
tags: [javaScript]
---

XSS是跨站脚本攻击，在客户端发出请求时，XSS代码出现在URL中，作为输入提交到服务器端，服务器端进行解析后并响应，XSS代码随着响应内容一起传回给浏览器，最后浏览器解析执行XSS代码。<!--more-->
## XSS的攻击方式
1. 反射性
在客户端发出请求时，XSS代码出现在URL中，作为输入提交到服务器端，服务器端进行解析后并响应，XSS代码随着响应内容一起传回给浏览器，最后浏览器解析执行XSS代码。

2.存储型
存储型XSS和反射型XSS的差别仅在于，提交的代码会存储在服务器端(数据库，内存，文件系统)，下次请求目标页面时不用再提交XSS代码

## Example
前端: url为http://www.test.com/?XSS='<img onerror="alert(1)">'
nodejs后端: 返回给浏览器:{XSS:req.query.XSS}
这样的话，浏览器就会对XSS代码进行解析，然后弹出1.

## XSS的防御方式
1. 编码 
2. 过滤
3. 校正

### 编码
在后端对XSS代码进行编码
"    => &quot;
&    =>&amp;
<    =>&lt;
>    =>&gt;
空格 =>&nbsp;"
这样做的目的是，服务器把XSS代码发送给浏览器的时候,浏览器先进行HTML解码，然后
进行Javascript解码，如果不进行编码的话，比如：浏览器解析到script的时候，会
直接加载并引入js代码，所以进行编码，这样HTML解析得到的只是script这几个字符
而不是标签。
### 过滤
在这一步是 对敏感标签进行过滤，比如script,iframe,frame,style.等，同时对标签内的属性进行过滤，比如：onclick，onerror.
这里用到了html parse这个插件


