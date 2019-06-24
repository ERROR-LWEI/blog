---
title: vueconf
date: 2019-06-09 18:19:06
tags: vueconf
category:
    -
        笔记
---

### 前言
> 基于对之前的一版不太满意的原因的下，今天做了两张图细化一下。同时更想说清楚为什么会出现vdom，以及进行react和vue的跨平台方案的切入点到底在哪里。看多端方案和vdom之前其实还是有必要去看原始web开发的实际生产中怎么做到从html，js到浏览器应用的这个过程，这个对于多端方案的理解是有很大帮助，现有多端方案细化的点不是很清楚，但是在大的架构方面都是相差不大的。

##### web端开发
web应用其实可以解释成浏览器的子产品和小程序如出一辙，只是宿主平台不一样。企业或用户的产品都是运行在同一个应用上的，chrome本身就是一个产品你开多少个浏览器访问多少个网站都是在这个宿主上去做事情。看宿主和web应用或者hybrid或者是react-native，大的方式都是一致的，宿主对于用户操作通用行为的封装暴露出接口，同时抽象一套视图系统，同时将视图系统与系统操作能力注入到JScore中。宿主所能提供的就是前进后退，操作响应，网络与文件io，设备调用这些通用的业务功能，至于是付款还是表单输入信息，输入的是什么这些上层操作就不在宿主考虑的范围了，如果掺入了细化的逻辑，在通用性上宿主的功能就越界了变的不纯粹了。web业务或者html与js有热度的原因很大一部分在于，它的移植性十分的强。包括nodejs，应用逻辑和宿主的关系并不是很紧密，业务代码更大一部分放在了js这端。

#### 浏览器的功能结构
从web前端转换到其他端js前端开发时，其实回头继续看浏览器就可以了，js在其中的位置大致相同。如果只是做js这端的基础开发只需要了解js引擎与宿主的对接关系就能处理了。
<div align=center><img src="/gallery/webkit.jpeg" width="800" align=center />
</div>
<br/>
<br/>
这个是qtwebkit的整体结构，web浏览器外部对接一个js引擎，一个操作系统，一个文档系统。脚本系统不与文档系统直接对接，文档系统的作用就只在初始化视图时用到，剩下的我们看到的dom数据是宿主根据文档数据转化成的视图对象然后注入到js引擎中的。
```
### html文档用来申明一个node类的类型，以及类型具备的功能参数的，img表示是图片类型，src表示下载的功能参数，title表示浮窗功能。
### 如果用不支持图片格式的软件打开图片，图片就是下面这种信息。
0101 1100 0010 ...
1111 0000 1001 ...
### 具体它的解析器规则假设0101代表的是rgba中的每一个，而这每四个构成一个像素的表达,它就是一个屏幕图像信息的表示了。
### 表达方式可以是任何形式，只要有对应的解释器。html的<img src="" />也是一样，有相应的解释器去解释这一段字符串。
```
###### HTMLTokenizer
HTMLTokenizer是内核中的一个c++类，用来做词法分析检验文档词法的合法性。是从字节流到特定格式的字符串。网页头部设置的utf-8就是帮助词法分析器去做字节处理的，当然不输的话也能自己找到匹配的解析方式，内置的nextToken函数会循环去读每读取一段就标记一段的处理状态并输出一个解释的词语。
```
### 在界面上看到html文档可能是utf-8,不过在下载完成后是从一种文件系统到另一个系统的对接，文件会被直接转成字节码。
### HTMLTokenizer 的具体功能就是用来做字节流的解码的。转化成能认识的编码。如果是不支持的编码方式。也就出现了有时候看到的乱码，乱码出现的点就是在HTMLTokenizer这里了。
```
命名还是很好理解的，所以写啥注释，命名好了大概就是提示了。
<br/>
<div align=center>
    <img src="/gallery/ma.jpeg" width="800" align=center />
    <img src="/gallery/string.jpeg" width="800" align=center />
</div>
<br/>
这其中也包含一部分xss攻击的处理，对于包含非法信息的标签进行过滤。在进行视图对接前就将数据处理干净，默默的感觉有点像接口参数校验。
<br/>
<br/>
<div align=center><img src="/gallery/morphology.jpeg" width="800" align=center/>
</div>
<br/>
##### HTMLDocumentParser
虽然HTMLDocumentParser与HTMLTokenizer是两个不同的小系统，但他们在整体里面所起到的作用是近似的都是在做前置数据的准备。一个负责检查转译筛选，一个负责将它转变成另一种形式，HTMLDocumentParser所做的工作就是将词语转成节点。这个处理过程是已栈的方式去做的，即遇到< 标记开始入栈 遇到</标记进行出栈。
```
## 解析下面一段文本的方式
<div><p></p></div>
[div,p]
[div]
[]
先压入div标签，再压入p标签，处理完p之后再处理完div最后div出栈
<img src="..." /> => { type: 'img', src: '....', .... }
```
HTMLDocumentParser内部调用的是HTMLTreeBuilder的constructTree方法进行词语到节点的转化，然后看c++的语法是真漂亮虽然多很多代码。其中很多这种std::是它的智能指针功能，内部有引用计数做垃圾回收。经过前两步处理完的数据最后的节点信息就是即将进行渲染的信息，html和js交互处理的实际上是这部分信息，比如在html加载的过程遇到js控制dom更改的过程。GUI和js访问的是同一个资源信息，GUI根据节点信息渲染dom，当js申请操作修改时节点信息就被锁住，GUI工作就需要被暂停，当js修改信息完成资源访问就被释放。
<br/>
<div align=center><img src="/gallery/treebuild.jpeg" width="800" align=center/>
</div>
<div align=center><img src="/gallery/treefun.jpeg" width="800" align=center/>
</div>
<div align=center><img src="/gallery/pross.jpeg" width="800" align=center/>
</div>
<br/>
##### node系统
node系统可以说是webkit最核心的功能，它赋予了js中的document下所有dom中的各种能力，比如调用底层注册事件等。不能完全看明白或者理解oop是什么，不过能学习到如何通过oop的方式去进行多系统的对接，可以把node与其他系统的连接看成下面这种形式。
<br/>
<div align=center><img src="/gallery/node-tree.jpeg" width="800" align=center/>
</div>
<br/>
```
class Event {
    constructor() {
    }

    static queue = [];

    add() {...}

    remove() {...}
}

class GuI {
    constructor() {}

    static render() {
        sys.Gui()
    }

    start() {....}
    stop() {....}
}

class Src {
    constructor() {}

    Io(src) {
        sys.Io(src);
    }
}

class Node extends Event, GuI {
    constructor() { super(); }

    ....
}
```
###### react中通过继承做界面的组合
真的比较懒所以这个图真的有点丑！这个方式是从其他人那里学到的，不过他的方式是不做整体页面的组合，都是做颗粒度比较小的界面部件的类型封装，这种是很灵活的。
<div align=center><img src="/gallery/react.jpeg" width="800" align=center/>
</div>
```
### js没有多继承搞这种真的是有点不舒服
### 一层层的拆成小的碎片

class Search extends React.Component {
    constructor(props) {
        super(props);
        this.state = {
            param: {},
        }
        this.initState();
    }

    initState(param) {
        return null;
    }

    /** 获取表单数据 **/
    getFormData() {
        ......
        暴露给外部获取数据
    }

    /** 过滤表单数据 **/
    filterFormData() {
        ......
        内部功能函数
    }

    render() {
        return (
            <div>{ 视图逻辑 }</div>
        )
    }
}

class SearchBtn extends Search {
    constructor(props) {
        super(props);
    }

    /** 与外部对接的过程 **/
    onSubmitReq(param) {
        return null;
    }

    /** 内部处理过程 **/
    onSubmit() {
        const data = this.getFormData();
        this.onSubmit(data);
    }

    render() {
        return (
            <div>
            { super.render() }
            <div>
                { 按钮视图 }
            </div>
            </div>
        )
    }
}

class List extends SearchBtn {
    constructor(props) {
        super(props);
    }

    initState() {
        this.state.list = [];
    }

    onSubmitReq(params) {
        api(params);
    }

    render() {
        return (
            <div>
                { super.render() }
                <table>
            </div>
        )
    }
}
```

node通过继承将各种能力添加到dom中，拆看来看单个父类的功能是很纯净的。在Event中自己维护和系统事件的对接，当监听到某一事件的触发，就调用注册的监听的控制器传入上层，在底层系统中的事件处理是没有目标的这个概念只有事件类型input类型事件click类型事件。这个处理方式也是对于本系统可控的方式的一点，系统的边界必须明确，比如从a-b的对接，其实我并不知道具体是a里面的谁要求做了这件事,事实是b也不需要知道。如果b连谁做了也要分析的话今天有abc申请了明天有mgn申请了，那这也太头痛了我只需要知道自己能处理的是input操作click操作处理了就丢出去你自己找是谁申请的。所以到前端这边也就存在了捕获和冒泡这种东西。为了保持底层的在移植的过程中的通用性。同时将对接的多个能力拆分到不同的类型中，分散治理将功能隔离开。
<div align=center><img src="/gallery/event.jpeg" width="800" align=center/>
</div>
<div align=center><img src="/gallery/event-sys.jpeg" width="800" align=center/>
</div>
<div align=center><img src="/gallery/addle1.jpeg" width="800" align=center/>
</div>
```
function add(a, b, c, d) {
    .......
    // **从一个应用到另一个应用**
    sys.add(a,b,c)
}
```
最后再回想conf上崔红保大佬说的不要频繁setData,以及为什么会出现vdom这个东西。由于js去直接操作dom不是在前端这边操作数据而是调用内核暴露出来的能力去修改内核中存储的dom节点信息。在信息不大的情况下其实都还好，大数据量的dom操作情况下相当于频繁的从a-b-c-d而且是从一个系统到另一系统的调用一个线程到另一个线程的通信。这个代价是高的，既然视图都是从数据去转成视图的，那前端先处理好视图数据最后集体调内核能力，vdom就是视图数据在前端的一种体现因为要做时间切片等优化在数据上又扩充了一些其他信息，react的fiber架构中单个节点中出现的time信息就是一个例子。
#### vdom
vdom有点像在js这端做的从html到节点转换的工作，当然附加了更多的运行时的数据，包含原有dom比原有dom的信息更丰富。react，vue之前的dom是由html文档提供初始化的视图数据浏览器解析然后返回到js，js属于直接操作dom，因为js本地其实是不存储视图关系（也可以用jq去实现vdom），所以有更改就要直接调用api操作浏览器内部的数据，vdom所解决的就是视图数据在前端的管理以及控制js这端调用浏览器操作的频率。

<div align=center><img src="/gallery/vdom.jpeg" width="800" align=center/>
</div>
<br/>
<br/>
```
updatequeue = [vdom1, vdom2, vdom3, vdom4];
// 每个vdom包含扩展信息又包含原始信息，始终能根据原始信息找到对应的原始dom和调用原始dom的api
vdom = {
    dom: dom,
    time: 123,
    vdomobject: {}, 
}
当调用set的时候更新并推到队列里，如果一个时间片内的长度是5，那么当有5个更新节点时就一次推动更新。当然描述可能存在误差但是主要是想表明在这里面，先借助vdom做前置处理，最后反应结果到dom上。
```
到这里其实多端方案的思路其实也比较清楚了，深入的话就到实际开发中去了。
##### react时间切片更新方案的大概方式
由于vdom是对于dom的一种反应，所以在前端js层是一个多叉树的结构去存储视图数据的，每个节点就是一个完整的fiber。前端开发使用vue或者react这套方案去做本质还是对一颗树的节点进行操作。所以，就会涉及到对于树的查找对于树的节点替换更新。多叉树的查找性能随着树的深度而加深，diff方案优化的是在前端对于数据树中的单个更新节点的查找，最后的更新还是整棵树的更新。
<br/>
<br/>
<div align=center><img src="/gallery/treeu.jpeg" width="800" align=center/>
</div>
<br/>
<br/>
来看下react的最小更新方案，react的fiber方案中。在单个节点也会存在一个updatequeue，如果打印一下父子组件的生命周期会发现它的过程是先子组件触发然后父子组件触发，它的更新方案是节点内的自治自下而上的传导，也就是小范围内的触发更新。vueconf上3.0说依旧保持模版方案，并且会加入动静结合的方式对存在变动的节点加入监听自己的理解应该也是这么个意思。在将模版信息转成对象信息的时候在这个转化的过程中就会插入记录有花括号的或者有for循环，就是揭露有可能存在更改的更改的节点，因为vue是以劫持的方式去监听更新的，如果用代理的方式去处理每个节点都创建一个代理对象，这自然在性能上是很大的开销因为你会为此加入类型推导判断是否存在变化，也就是在运行时不管有没有变化都会做判断逻辑，也就必然存在时间的增加。这和要求我们尽量是用const不用let是一个意思，const声明的是常量不会存在类型变化，v8内核不会在运行时不会为此做类型变化的判断。另外在尤大提到的3.0将去除class和装饰器的支持之前自己是很喜欢用装饰器的也并没有考虑到装饰器带来的不稳定性，这涉及到函数的返回副作用，这个也是监测点。
```
#### react的高阶函数
function Component(Target) {
    return class extends React.Component {

    }
}
高阶函数的副作用在于尽管在代码层面看起来他们都是一样的结构，但是函数的返回始终是一个新的对象，他们其实并不相等。代码存在复用，在实际的运行时内存中的数据不存在复用。
js的函数参数的指针是隐式的，即不管传入的参数是否在函数内有操作，所返回的值都是要在内存中创建一个新的数据与指针，存在空间的损耗。
```
真正的更新操作是，js在一定的周期内内部自治完成对节点信息的替换更新，然后调用api将维护完的数据树同步到浏览器中。像上图中的两颗树，当某个范围内的节点触发set时，将js存储的数据树中关于视图部分的信息更新掉。
```
{
    type: 'div',
    data: {
        list: [1,2,3],
    },
    html: `<div>{ this.list }</div>`
}
{
    data: {
        list: [3,4,5],
    },
    html: `<div>{ this.list }</div>`
}
```
像这样的节点整颗更新掉之后，html的信息其实已经替换成最新的，然后innerhtml=*****。触发浏览器的视图刷新，这样就避免了一次次的调用api，也就变成了时间切片了，在一定时间内我这边自己做好处理再通知你。

##### 现在的多端方案使用vue和react，借用的点是什么？
其他端和浏览器是一致的，他们唯一借助的点是vue和react关于视图数据的管理和更新。唯一存在魔改的点在视图功能的对接口，即在浏览器端src和title可能对应的是node类中的一中功能，在安卓端这两个属性对应的实现点又是另一种方式。
```
<img src=**** title=***/>
```
所以慢慢的有一种感觉就是虽然都是react和vue但是其实已经不是之前在web端使用的了。
