---
title: 使用refect管理redux应用
date: 2019-05-11 14:44:57
tags: redux
category:
    -
        react
    -
        redux
---
> 想到这么干就要是最近使用了nextjs，按这种形式去写真的很能偷懒。完全不用花太多的经历去管理功能模块在形式上的连接关系，只要做完注解，工厂函数在初始化的时候自动就会帮我们把信息组织好了。所以，从用过这个框架之后，对于注解也有一点认识了，起码在nestjs里面它所发挥的作用是帮助开发者去组织功能之间的关系。

## 前言

### nest的web服务端的结构

``` bash
### 控制器，用来提供具体的接口
@Controller('/user')
class UserController {}

### 服务，提供数据操作
@Injectable()
class UserService {}

### 模块，
$ hexo new "My New Post"
```

More info: [Writing](https://hexo.io/docs/writing.html)

### Run server

``` bash
$ hexo server
```

More info: [Server](https://hexo.io/docs/server.html)

### Generate static files

``` bash
$ hexo generate
```

More info: [Generating](https://hexo.io/docs/generating.html)

### Deploy to remote sites

``` bash
$ hexo deploy
```

More info: [Deployment](https://hexo.io/docs/deployment.html)
