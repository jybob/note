### 后台商品列表展示步骤:

1. 在admin.html页面中删除数组中的假数据, 在created方法中向/product/select发请求获取商品信息把得到的商品信息给到productArr数组
2. 在ProductController里面添加select方法处理/product/select请求 调用mapper里面的select方法把返回的集合直接响应给客户端
3. 实现mapper中的 select方法

### 表字段和属性字段不一致解决方案:

1. 通过别名(这种用法不能复用不推荐)

2. 通过@Result注解  

3. 通过在application.properties里面添加配置信息

   ![image-20220916093838544](day08.assets/image-20220916093838544.png)

### 删除商品

1. 给删除按钮添加点击事件 点击时向/product/delete发出请求把商品id传递过去
2. 在ProductController中添加delete方法处理/product/delete请求, 在方法中先通过id查询到商品图片的url,然后删除文件, 删除后调用mapper的deleteById方法 删除数据库里面的数据
3. 实现mapper里面deleteById方法 



### 首页排行榜显示步骤:

1. 删除topArr里面的假数据,  在created方法中向/product/selectTop 发出请求把请求到的数组赋值给topArr
2. 在ProductController中添加selectTop方法处理/product/selectTop,方法中调用mapper的selectTop方法 把接收到的list集合直接响应给客户端
3. 实现mapper中的selectTop方法 ,在SQL语句中通过销量降序排序只查询前6条数据

### 首页商品展示步骤:

1. 把首页中productArr数组中的假数据删除, 在created方法中向/product/selectForIndex发出请求把请求到的数组赋值给productArr 
2. 在ProductController中添加selectForIndex方法处理/product/selectForIndex,方法中调用mapper的selectForIndex方法 把接收到的list集合直接响应给客户端
3. 实现mapper中的selectForIndex方法

### 点击商品查看详情步骤:

1. 首页的图片和商品标题 改成超链接 请求地址为 /detail.html?id=xxx 
2. 在detail.html页面中的created方法中 向/product/selectById发出请求, 查询到的商品对象赋值给Vue里面的对象 然后页面中的标签和此对象进行绑定 
3. 在ProductController中添加selectById方法处理/product/selectById请求, 方法中调用mapper的selectById方法 通过id查询到商品信息 响应给客户端
4. 实现mapper里面的selectById方法   





### 晚课: 表设计之权限管理

- 权限管理需要设计三张主表和两张关系表
- 主表包括:用户表, 角色表,权限表
- 关系表:  用户-角色关系表,   角色权限关系表

![image-20220916200145257](image-20220916200145257.png)

- 创建三张主表: 用户表, 角色表,权限表

  create table user(id int primary key auto_increment, name varchar(20));

  create table role(id int primary key auto_increment, name varchar(20));

  create table module(id int primary key auto_increment, name varchar(20));

- 创建两张关系表: 用户角色关系表      角色权限关系表

  create table u_r(uid int, rid int);

  create table r_m(rid int,mid int);

- 准备数据:

  insert into user values(null,'刘德华'),(null,'杨幂'),(null,'张学友');

  insert into role values(null,'男游客'),(null,'男管理员'),(null,'女游客');

  insert into module values(null,'男浏览'),(null,'男发帖'),(null,'男评论'),(null,'男删帖'),(null,'女浏览');

  刘德华(男游客,女游客)    杨幂(女游客)     张学友(男管理员)

  insert into u_r values(1,1),(1,3),(2,3),(3,2);

  insert into r_m values(1,1),(2,1),(2,2),(2,3),(2,4),(3,5);

- 关联查询练习题:

  1. 查询刘德华拥有哪些角色

     select r.name

     from user u join u_r ur on u.id=ur.uid

     join role r  on r.id=ur.rid where u.name='刘德华';

  2. 查询女游客对应的用户都有谁

     select u.name

     from user u join u_r ur on u.id=ur.uid

     join role r  on r.id=ur.rid   where r.name='女游客';

  3. 查询男管理员对应哪些权限

     select  m.name

     from role r join r_m rm on r.id=rm.rid

     join module m on m.id=rm.mid  where  r.name='男管理员';

  4. 查询刘德华拥有哪些权限

     select m.name

     from user u join u_r ur on u.id=ur.uid 

     join r_m rm on rm.rid=ur.rid

     join module m on m.id=rm.mid

     where u.name='刘德华';

  5. 查询有女浏览权限的用户都有谁

     select u.name

     from user u join u_r ur on u.id=ur.uid 

     join r_m rm on rm.rid=ur.rid

     join module m on m.id=rm.mid 

     where m.name='女浏览';

  









