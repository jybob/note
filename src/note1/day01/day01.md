张皓岚

微信号:Jtzhanghl

第五阶段

19天课程

第一部分:day1~day7 Vant 微服务概念搭建微服务架构程序,学习微服务相关组件

第二部分:day7~day16 酷鲨商城前台项目开发

第三部分:day17~day19 linux系统,虚拟机,Docker容器化等需要掌握的知识

学习方法

最重要的还是练习,而且是反复练习,要知道每行代码的作用,和为什么需要这行代码

课程中后阶段,需要抽时间背面试题

针对面试官的询问,我们需要讲我们理解的业务清晰的表达出来

微信答疑:

* 课程中的知识点

* 直播的问题

* 节奏,语速的问题

* 代码挑错

* 其它......

# Vant

## 什么是Vant

![image-20221020092254758](note/day01/imagesay01/images/day01/image-20221020092254758.png)

Vant是一个轻量，可靠的移动端组件库，2017开源

目前 Vant 官方提供了 [Vue 2 版本](https://vant-contrib.gitee.io/vant/v2)、[Vue 3 版本](https://vant-contrib.gitee.io/vant)和[微信小程序版本](http://vant-contrib.gitee.io/vant-weapp)，并由社区团队维护 [React 版本](https://gitee.com/link?target=https%3A%2F%2Fgithub.com%2F3lang3%2Freact-vant)和[支付宝小程序版本](https://gitee.com/link?target=https%3A%2F%2Fgithub.com%2Fant-move%2FVant-Aliapp)。

课程中我们使用Vue2版本对应的Vant学习

[https://youzan.github.io/vant/v2/#/zh-CN/]

##  Vant的优势

ElementUI是开发计算机浏览器(非移动端)页面的组件库

而Vant是开发移动端页面的组件库

我们设计的酷鲨商城前台项目使用手机\移动设备访问的,所以使用Vant更合适

##  Vant特性

- 🚀性能极佳，组件平均体积小于 1KB（min+gzip）
- 🚀 65+ 个高质量组件，覆盖移动端主流场景
- 💪 使用 TypeScript 编写，提供完整的类型定义
- 💪 单元测试覆盖率超过 90%，提供稳定性保障
- 📖 提供完善的中英文文档和组件示例
- .......

# 第一个Vant程序

## 创建Vue项目

参考第四阶段创建Vue项目的步骤,创建Vue项目

首先确定一个要创建项目的文件夹

例如在E盘下创建vue-home文件夹

进入文件夹,在地址栏输入cmd 打开dos命令行界面

```
E:\vue-home>vue create demo-vant
```

创建时具体选项,参照之前四阶段的笔记即可

下面我们就可以用idea打开我们创建的项目了

## 安装Vant支持

我们创建的Vue项目并不会默认就支持Vant

所以,我们需要安装Vant的支持到Vue项目中

在打开的idea界面最下方,找到Terminal点击

在出现的命令行中输入安装Vant的指令

运行安装Vant的npm命令:(-S的S是大写的!!!!!!!)

```
E:\vue-home\demo-vant>npm i vant@latest-v2 -S
```

安装结束后可能有警告,可以忽略

最终可能看到下面的结果

```
added 4 packages in 3s
```

到此为止,我们就将Vant运行需要的文件安装在vue项目中了

git地址:

https://gitee.com/jtzhanghl/vant2206.git

## 添加Vant引用

如果想要在项目中使用Vant提供的组件

需要在Vue项目代码中添加Vant的引用

Vue项目的src/main.js文件中,添加引用代码如下

```js
import Vant from 'vant'
import 'vant/lib/index.css'

Vue.use(Vant)
```

添加了上面的引用,当前Vue项目就可以使用Vant组件了

为了实时运行项目,我们先启动Vue项目,测试表示它正常运行

可以在idea提供的Terminal界面中编写如下代码

```
E:\vue-home\demo-vant>npm run serve
```

打开浏览器

输入localhost:8080

进入移动端页面调试模式

以Google浏览器为例按F12进入调试模式后点击移动端调试即可

![image-20220428095155966](note/day01/image-20220428095155966.png5155966.png)

## 按钮组件

所有组件的演示代码,都可以从官网获取

https://youzan.github.io/vant/v2/#/zh-CN/

在HomeView.vue文件中修改代码如下

```html
<template>
  <div class="home">
    <van-button type="primary">主要按钮</van-button>
    <van-button type="info">信息按钮</van-button>
    <van-button type="default">默认按钮</van-button>
    <van-button type="warning">警告按钮</van-button>
    <van-button type="danger">危险按钮</van-button>
  </div>
</template>
```

打开页面就能看到按钮的样式了

看到这个内容,表示当前Vant组件工作正常

如果没有看到效果

检查Vant组件的安装和引用

## 表单页面

登录作为移动端非常常见的界面

Vant表单是直接提供模板的,我们可以在官网找到表单链接直接使用

```html
<template>
  <div class="about">
    <!--
    @submit是vant组件提供的Event(事件),在表单提交验证成功时触发,绑定的方法在下面js中
    @failed是vant组件提供的Event(事件),在表单提交验证失败时触发,绑定的方法在下面js中
    如要了解更详细的解释或其他api,查阅官方文档即可
    -->
    <van-form @submit="onSubmit" @failed="onFailed">
      <van-field
          v-model="username"
          name="用户名"
          label="用户名"
          placeholder="用户名"
          :rules="[{ required: true, message: '不填写用户名不行' }]"
      />
      <van-field
          v-model="password"
          type="password"
          name="密码"
          label="密码"
          placeholder="密码"
          :rules="[{ required: true, message: '请填写密码' }]"
      />
      <div style="margin: 16px;">
        <van-button round block type="info" native-type="submit">提交</van-button>
      </div>
    </van-form>
  </div>
</template>


<script>
export default {
  data() {
    return {
      username: '',
      password: '',
    };
  },
  methods: {
    onSubmit(values) {
      console.log('submit', values);
    },
    onFailed(errorInfo){
      // 本方法会在表单提交验证失败时运行,也就是用户名和密码有null时运行
      // 参数errorInfo是错误信息,要想知道其中内容,直接输出即可
      console.log(errorInfo);
    }
  },
};
</script>
```

注意@submit和@failed这两个事件的绑定关系

https://gitee.com/jtzhanghl/vant2206.git

## area省市区选择

先在views文件夹下创建AreaView.vue文件

编写代码如下

```html
<template>
  <div>
    <van-area title="标题" :area-list="areaList" />
  </div>
</template>

<script>
const areaList = {
  province_list: {
    110000: '北京市',
    120000: '天津市',
  },
  city_list: {
    110100: '北京市',
    120100: '天津市',
  },
  county_list: {
    110101: '东城区',
    110102: '西城区',
    120103: '武清区',
    // ....
  },
};
export default {
  data(){
    return {areaList};
  }
}

</script>
```

定义路由设置

```js
const routes = [
  {
    path: '/',
    name: 'home',
    component: HomeView
  },
  {
    path: '/about',
    name: 'about',
    // route level code-splitting
    // this generates a separate chunk (about.[hash].js) for this route
    // which is lazy-loaded when the route is visited.
    component: () => import(/* webpackChunkName: "about" */ '../views/AboutView.vue')
  },
  {
    path: '/area',
    name: 'area',
    component: () => import('../views/AreaView.vue')
  }
]
```

通过输入路径http://localhost:8080/area

访问省市区选择页面

只有少量数据

如果想要真实的数据,那么就需要在上面areaList数据中填入大量数据

但是这个工作量很大,个人实现非常困难,所以可以使用业界通用的一个省市区信息json对象

先安装全国省市区数据包

idea的Terminal界面输入如下命令

```
E:\vue-home\demo-vant>npm i @vant/area-data
```

如果有警告,直接无视

安装结果可能为

```
added 1 package in 2s
```

如果该安装成功,就可以添加全国省市区数据到项目中了

AreaView.vue代码中添加如下

```html
<template>
  <div>
    <van-area title="标题" :area-list="areaList" @confirm="showArea" />
  </div>
</template>

<script>

// 从全国省市区数据包中获得数据对象 命名为areaList
import {areaList} from '@vant/area-data'
export default {
  data(){
    return {areaList};
  },
  methods:{
    showArea(area){
      console.log(area);
    }
  }
}

</script>
```

上面代码不但实现了全国的省市区选择也支持了用户在确认时,将信息输出的功能

**课堂作业**

新建一个AddressView.vue文件

完成Vant地址列表的效果

参考vant2官网的资料,创建页面,设置路由,加载正确代码

测试运行出现效果即可

## 商品列表

我们的电商网站一定会需要商品列表

移动端Vant直接支持了商品列表的格式

我们也不需要大范围的修改

创建ListView.vue

代码如下

```html
<template>
  <div>
    <van-row>
      <van-col span="8">综合</van-col>
      <van-col span="8">价格</van-col>
      <van-col span="8">销量</van-col>
    </van-row>
    <van-card
        num="10"
        price="15.00"
        desc="喝完能够恢复体力"
        title="特比好喝的饮料"
        thumb="https://img01.yzcdn.cn/vant/ipad.jpeg"
    >
      <template #tags>
        <van-tag type="primary">自营</van-tag>
        &nbsp;
        <van-tag plain type="primary">酷鲨物流</van-tag>
      </template>
      <template #footer>
        <van-button size="mini" type="primary">添加到购物车</van-button>
        <van-button size="mini" type="primary">立即购买</van-button>
      </template>
    </van-card>
    <van-card
        num="10"
        price="15.00"
        desc="喝完能够恢复体力"
        title="特比好喝的饮料"
        thumb="https://img01.yzcdn.cn/vant/ipad.jpeg"
    >
      <template #tags>
        <van-tag type="primary">自营</van-tag>
        &nbsp;
        <van-tag plain type="primary">酷鲨物流</van-tag>
      </template>
      <template #footer>
        <van-button size="mini" type="primary">添加到购物车</van-button>
        <van-button size="mini" type="primary">立即购买</van-button>
      </template>
    </van-card>

  </div>
</template>

<script>
</script>
```

其它的组件同学们可以自行进行使用或研究,课上就不带大家依次演示了



# 服务器端程序的演进过程

## 阶段一:静态服务器

早期的服务器状态,安装好一些固定内容,让用户访问

功能单一,如果不修改代码,内容是不会变的,只能做信息的呈现或输出

## 阶段二:普通动态服务器

网页中的数据可能来自数据库,数据库中的数据可以在后台中进行修改

实现不修改页面代码,但是变化页面内容的效果

因为有了数据库的支持,动态网站开始支持登录注册,增删改查功能

## 阶段三: 以用户共享内容为主的互联网生态

随着互联网的普及,个人的社交需求提升

出现了很多由用户贡献内容的网站

微博,抖音,淘宝,大众点评或类似的网站

## 阶段四: 微服务时代

随着用户的增加,各种并发的增高,要求我们的服务器在繁忙的情况下,也需要快速的做出响应

用户体验必须保证,这样就要求我们的项目有下面三个目标

**高并发,高可用,高性能**

高并发:很多用户同时访问这个服务器,这个服务器不能失能

高可用:全年365天每天24小时都可以访问,不能因为个别服务器的异常,导致整个项目的瘫痪

高性能:当用户访问服务器时,响应速度要尽量的快,即使并发高,也要有正常的响应速度

微服务的"三高"



## java服务器项目分类

现在市面上常见的java开发的项目可以分为两大类

1.企业级项目

一般指一个企业或机构内部使用的网站或服务器应用程序

使用的人群比较固定,并不向全国乃至全世界开放

例如,商业,企事业单位,医疗,金融,军事,政府等

所以这个项目没有代替品,对"三高"没有强烈要求

企业级项目一般会在权限和业务流程方面设计的比较复杂

2.互联网项目

能够向全国乃至全世界开放的网站或服务器应用程序

我们手机中安装的app大部分都是互联网应用

微信,支付宝,京东,淘宝,饿了么,美团,抖音,qq音乐,爱奇艺,高德地图等

它们因为商业竞争等原因,对服务器的性能有非常高的要求,就是我们之前提到的"三高"

但是互联网应用一般没有权限和业务非常复杂的需求

综上所述,企业级应用和互联网应用的偏重点不同



在当今java开发业界中,基本规律如下

* 如果开发的是企业级应用,使用单体架构的情况比较多
* 如果开发的是互联网应用,使用微服务架构的情况比较多

# 微服务概述

## 什么是微服务

微服务的概念是由Martin Fowler（**马丁·福勒**）在2014年提出的

![image-20220428153622848](note/day01/imagesay01/images/day01/image-20220428153622848.png)

微服务是由以单一应用程序构成的小服务，自己拥有自己的行程与轻量化处理，服务依业务功能设计，以全自动的方式部署，与其他服务使用 HTTP API 通信。同时服务会使用最小的规模的集中管理能力，服务可以用不同的编程语言与数据库等组件实现。

简单来说,微服务就是将一个大型项目的各个业务模块拆分成多个互不相关的小项目,而这些小项目专心完成自己的功能,而且可以调用其他小项目的方法,从而完成整体功能

京东\淘宝这样的大型互联网应用程序,基本每个操作都是一个单独的微服务在支持:

- 登录服务器
- 搜索服务器
- 商品信息服务器
- 购物车服务器
- 订单服务器
- 支付服务器
- 物流服务器
- .....

## 为什么需要微服务

![image-20220428154923065](note/day01/imagesay01/images/day01/image-20220428154923065.png)

左侧小餐馆就像单体项目

一旦服务器忙,所有业务都无法快速响应

即使添加了服务器,也不能很好的解决这个问题

不能很好的实现"高并发,高可用,高性能"

但是因为服务器数量少,所以成本低,适合并发访问少的项目

右侧大餐厅就是微服务项目

每个业务专门一批人来负责,业务之间互不影响

在某个模块性能不足时,针对这个模块添加服务器改善性能

万一一个服务器发生异常,并不会影响整体功能

但是完成部署的服务器数量多,成本高,需要较多投资,能够满足"高并发,高可用,高性能"的项目

## 怎么搭建微服务项目

在微服务概念提出之前(2014年),每个厂商都有自己的解决方案

但是Martin Fowler（**马丁·福勒**）提出了微服务的标准之后,为了技术统一和兼容性,很多企业开始支持这个标准

现在我们开发的微服务项目,大多数都是在马丁·福勒标准下的

如果我们自己编写支持这个标准的代码是不现实的,必须通过现成的框架或组件完成满足这个微服务标准的项目结构和格式

当今程序员要想快速开发满足上面微服务标准的项目结构,首选SpringCloud

# Spring Cloud

## 什么是SpringCloud

SpringCloud是由Spring提供的一套能够快速搭建微服务架构程序的**框架集**

框架集表示SpringCloud不是一个框架,而是很多框架的统称

SpringCloud就是为了搭建微服务架构项目出现的

有人将SpringCloud称之为"Spring全家桶",广义上指代Spring的所有产品

## Spring Cloud的内容

**内容的提供者**

* Spring自己提供的开发出来的框架或软件
* Netflix(奈非):早期的很长一段时间,提供了大量的微服务解决方案
* alibaba(阿里巴巴):新版本的SpringCloudAlibaba正在迅速占领市场(推荐使用)

课程中使用全套的阿里巴巴组件

**功能上分类**

* 微服务的注册中心
* 微服务间的调用
* 微服务的分布式事务
* 微服务的限流
* 微服务的网关
* ....

# 注册中心Nacos

## 什么是Nacos

Nacos是Spring Cloud Alibaba提供的一个软件

这个软件主要具有注册中心和配置中心(课程最后讲解)的功能

我们先学习它注册中心的功能

微服务中所有项目都必须注册到注册中心才能成为微服务的一部分

注册中心和企业中的人力资源管理部门有相似

![image-20220505094542229](note/day01/imagesay01/images/day01/image-20220505094542229.png)

当前微服务项目中所有的模块,在启动前,必须添加注册到Nacos的配置

所谓注册,就是将自己的信息,提交到Nacos来保存

## Nacos的下载

https://github.com/alibaba/nacos/releases/download/1.4.3/nacos-server-1.4.3.zip

国外网站,下载困难可以多试几次

或直接向项目经理老师索取

## Nacos的启动

因为Nacos是java开发的

我们要启动Nacos必须保证当前系统配置了java环境变量

简单来说就是要环境变量中,有JAVA_HOME的配置,指向安装jdk的路径

确定了支持java后,就可以启动Nacos了

mac系统同学一定要到http://doc.canglaoshi.org/查看homebrew相关知识

mac系统安装Nacos推荐

https://blog.csdn.net/gongzi_9/article/details/123359171

windows的同学保证java环境变量正常后

将nacos-server-1.4.2.zip压缩包解压

双击打开解压得到的文件夹后,再双击打开其中的bin目录

![image-20220505104318585](note/day01/imagesay01/images/day01/image-20220505104318585.png)

cmd结尾的文件是windows版本的

sh结尾的文件是linux和mac版本的

startup是启动文件,shutdown是停止文件

**Windows下启动Nacos不能直接双击cmd文件**

需要在dos窗口运行

在当前资源管理器地址栏输入cmd

```
E:\tools\nacos\bin>startup.cmd -m standalone
```

startup.cmd:windows启动nacos的命令文件

-m 表示要设置启动参数

standalone:翻译为标准的孤独的,意思是正常的使用单机模式启动

运行成功默认占用8848端口,并且在代码中提示

如果不输入standalone运行会失败

如果报了

"please set JAVA_HOME......."

表示当前项目没有配置java环境变量(主要是没有设置JAVA_HOME)

如果运行没有报错

打开浏览器输入地址

http://localhost:8848/nacos

![image-20220505105822385](note/day01/imagesay01/images/day01/image-20220505105822385.png)

如果是首次访问,会出现这个界面

登录系统

用户名:nacos

密码:nacos

登录之后可以进入后台列表

不能关闭启动nacos的dos窗口

我们要让我们编写的项目注册到Nacos,才能真正是微服务项目