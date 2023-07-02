### 酷鲨商城项目步骤:

1. 创建工程coolshark  打3个勾  
2. 在application.properties配置文件中添加连接数据库信息
3. 把老师工程中四个页面和imgs文件夹赋值到新工程中, 然后鼠标选中static 按下ctrl+shift+f9重新编译  然后再运行 访问首页,能够正常显示即可

4. 建库建表

create database cs charset=utf8; 

use cs; 

create table user(id int primary key auto_increment,username varchar(50),password  varchar(50),nick varchar(50));

 insert into user values(null,'admin','123456','管理员');

### 登录功能步骤: 

1. 在 login.html 中给登录按钮添加点击事件 点击时调用 login 方法,并且在 data 里面声明一个 user 对象让页面中的文本框和密码框和 user 对象里面的用户名和密码进行双向绑定,实现 login 方法在方法中向/login 发出请求,根据响应的 123 控制显示的内容,如果是 1 代表成功 跳转到首页 

2. 创建 User 实体类,创建 UserMapper 在 mapper 中提供一个 selectByUsername 方法 

3. 创建 UserController 添加 login 方法处理/login 请求, 调用 mapper 里面的方法 给客户端响 应 1,2,3

### 首页分类展示步骤:

1. 建表并准备数据

   create table category(id int primary key auto_increment,name varchar(20));

   insert into category values(null,'精彩活动'),(null,'当季女装'),(null,'品牌男装'),(null,'环球美食'),(null,'医药健康'),(null,'啦啦啦');

2. 创建Category实体类, 创建CategoryMapper 添加select方法 查询所有

3. 创建CategoryController 添加select方法处理/category/select请求

4. 在index.html页面让分类展示部分的代码和categoryArr数组进行绑定,  在created方法中向/category/select地址发出请求获取所有分类数据 把得到的数据赋值给categoryArr即可.

### 首页轮播图展示步骤

1. 建表和准备数据

   create table banner(id int primary key auto_increment,url varchar(200));

   insert into banner values(null,'/imgs/b1.jpg'),(null,'/imgs/b2.jpg'),(null,'/imgs/b3.jpg'),(null,'/imgs/b4.jpg');

2. 创建Banner实体类有id和url两个属性

3. 创建BannerMapper 里面创建一个select方法

4. 在index.html首页中的created方法里面向/banner/select地址发出请求获取数据把得到的数据赋值给bannerArr数组, 让页面中的轮播图和bannerArr数组进行绑定

5. 创建BannerController 添加select方法处理/banner/select请求,调用mapper里面的select方法获取到所有轮播图的数据响应给客户端