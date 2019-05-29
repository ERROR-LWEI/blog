---
title: typecript中的reflectApi元编程
date: 2019-05-30 21:33:00
tags: reflect
category:
    -
        typescript
---
### 元编程
>元编程（Metaprogramming）是指某类计算机程序的编写，这类计算机程序编写或者操纵其他程序（或者自身）作为它们的数据，或者在运行时完成部分本应在编译时完成的工作。很多情况下与手工编写全部代码相比工作效率更高。编写元程序的语言称之为元语言，被操作的语言称之为目标语言。一门语言同时也是自身的元语言的能力称之为反射。

这个是一个很抽象的概念，作为非计算机类专业的好像也不太好班门弄斧去解释这个概念。只能从字面去说一下自己的认识。“元编程”字面就是对原子进行编程，在具体使用中这个原子可以细化到一个变量也可以扩展到一个函数功能和一个对象。编程的概念其实描述的是一个过程，从输入到产出的一个整体的过程，开发者通过代码参与这个过程控制过程的流向。元编程的话，假设在一个过程中有一个a功能对象是专门用于处理一个订单分发策略的，那么这其中一定会有根据不同的标注信息做不同的动作，虽然同一个功能对象但是能根据不同的信息产生不同的效果（可能这就是控制反转，在不修改代码内部的情况下通过外部标注信息的修改达到修改结果的目的）。

```
import 'reflect-metadata';

@Module({
    import: [],
    controller: [],
    exports: [],
})
class Test {
    constructor(props) {}
    @Reflect.metadata('hello', 'word')
    getService() {}
}
```
利用reflect对类进行元数据的添加，在class外和class内使用装饰器去处理target目标有所不同。module部分修饰的目标是类本身，metadata部分修饰的是类的原型链对象prototype上的属性。前者如果在程序启动后获取的一般是未实例化下的类型注解，后者能获取到的注解信息是实例化后的对象的。
#### nestjs使用注解对元子进行操作的特点
##### 类型注解
@Module
这个是注解一个模块类型的，在nest去编写node服务时使用module表明这是一个模块类型的类，同时在内部做到了从path资源路径到module的关联。内部做到将module名称转换到path的过程，同时module的概念包含了service与import信息。
```
例：
/user/info/add
/user/msg/detail
这样一个资源路径转换成module
        user
    info    msg
add             detail
应用的全局其实就维护着这样一个信息，信息的获取与组织就通过注解去完成了。
```
每一个类型注解对应一套处理规则，module类型用于组织资源的关联信息Inject类型负责组织在module类传递的服务依赖信息。说回module的另一个注解信息exports，exports的功能用于导出service服务。无导出情况下service操作只在module内部可见，有导出操作后module的service功能在父子兄弟之间传递共享，所要应对的情况是：在detail内可能需要调用add的数据操作功能，因为service操作最终还是db操作的实体化。所以需要具备导出操作。
```
import起到连接module的作用建立上下级关系同时具备service的传导操作
    import
user  =>  info
user  =>  msg
    service
user  <=  info
user  <=  msg
这类似与在父进程中兄弟线程的共享内存
```
##### 依赖注入
直观点按nest的操作来看就是
```
@module({
    import: [],
    controller: [UserController],
    provider: [UserService]
})
class User {}

// 内部其实大概是这样的
new UserController(new UserService)

class UserService {
    constructor(userservice: UserService) {}
}

// 注入的过程在工厂函数构建应用时完成的
```
##### 操作
应用组织完成后，从前端调用接口到最后的数据操作大概要走下面的过程
```
db <= service <= controller
```

#### 模型转到前端
把这套方式转到前端处理的化，视图层先不管那只是很外在的东西不管界面长得千奇百怪最后还是要进行操作执行action去触发请求动作。所以重要的是收集存在的action动作信息更新创建等如果用saga的task概念去管理异步的化就可以将一个动作集合整合到一个task任务中去处理减少在视图层的组合请求动作（如前后一个动作依赖前一个动作的结果）这样能减少视图层逻辑的复杂度，增加应用的移植性，另外在saga中有管道的概念相同的动作能做到顺序执行并能暂停和重启回退这对于频繁的操作能起到很好的抗压作用。所以重要点其实不在视图层而是在redux应用的整个action动作的流程。
##### redux应用的中心
redux应用的整体使用状态去控制，一个action触发的是一个操作状态ADD_DETAIL之类的。需要组织的是应用的状态管理。那么基础的元数据就是应用的状态一个函数是用来处理何种状态的。
```
对应关系
db <= service <= controller
store <= action <= view
```

##### store
```
@Store('user')
class User {
    constructor() {}

    @type('ADD_USER')
    function add(state, action) {
        ......
    }

    @type('UPDATA_USER')
    function updata(state, action) {
        ......
    }
}
```
##### action
```
@Action()
class UserAction {
    constructor(user: User) {}
    
    @post('/api/user', 'json')
    function add(params) {
        return {
            type: user.ADD_USER
            params: {
                ...params
            }
        }
    }
}
```
##### view
```
@View
class User extends React.Component {
    constructor() {}
}

function View(state, action) {
    return function(Target) {
        return connect(state, action)(Target)
    }
}
```
#### import
nest中import完成了module中资源路径前前后关系，转换到前端可以用import去关系整个路由的前后关系。这样也省去前端去管理路由的额外的开发时间。

最主要的还是借鉴nest的方式想将应用的路由，api，状态管理的额外开发量合并到开发具体功能中省去这部分的时长。这部分的信息大多散落在各个文件中，不利于管理。