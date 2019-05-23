---
title: a跳转操作
date: 2019-05-23 19:54:28
tags: HTML
category:
    - 
        HTML
---
## 超链接标签
html中的a标签超链接可以实现，开启新tab，下载，跳转，锚点，调起外部功能。

#### 跳转连接
利用a标签进行路由跳转
```
<a href="www.zhihu.com" title="知乎">知乎</a>
```
#### 开启tab
利用a标签跳转页面打开新页面
```
<a href="www.zhihu.com" title="知乎" target="_target">知乎</a>
```

#### 锚点
利用a标签跳转到页面的其他地方
```
<a href="#">返回页面顶部</a>
```

#### 下载连接
进行资源下载
```
<a href="*****" download>文档</a>
```

#### 调起外部服务
```
### 移动端电话拨打
<a href="tel:+134*******">联系电话</a>
### 邮件发送
<a href=“mailto:error_lwei@163.com”>发送给谁</a>
### 抄送
<a href="mailto:error_lwei@163.com" cc="lemonpaimc@163.com">抄送</a>
```
