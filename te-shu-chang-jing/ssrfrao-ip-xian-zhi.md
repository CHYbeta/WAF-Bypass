# 0x01 前言
通常SSRF里的绕过，是指对IP的绕过。

# 0x02 法一： http基础认证
http基础认证 user@domain.com

若限制好了只能调用一个固定的域名,如www.baidu.com下的内容，则可用以下payload绕过：
```
www.baidu.com@attack.com
```

# 0x03 法二：xip.io
传送门：http://xip.io/ 。它处理类似[ip].xip.io的dns解析请求时,返回的ip总是指向[ip]。

请求 www.baidu.com 时，可用
payload1:
```
111.13.100.91.xip.io
```


# Refference
+ [math1as:CUIT-CTF WriteUp总结](http://www.math1as.com/index.php/archives/70/)