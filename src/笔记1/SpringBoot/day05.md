### 文件上传步骤

1. 创建一个html页面,此页面需要用到elementUI组件, 所以从Web阶段day07的hello.html复制粘贴而来
2. 从ElementUI文档中找到Upload组件,  找到里面的照片墙组件, 把相关代码复制粘贴到自己的页面中
3. 修改组件代码中action的地址为/upload  给el-upload组件添加name属性 设置上传文件的参数名

4. 创建UploadController处理 /upload请求, 在参数列表处声明MultipartFile类型的变量接收传递过来的文件参数 soutp控制台输出传递过来的参数 进行测试, 测试前先在application.properties配置文件夹中添加 设置上传文件大小的相关代码

![image-20220913114800509](day05.assets/image-20220913114800509.png)

5. 得到图片的原始文件名 取出后缀 然后再得到唯一文件名, 把图片保存到指定的文件夹路径下,如果文件夹不存在需要创建文件夹 
6. 设置静态资源文件夹: 在application.properties配置文件中添加以下内容,因为客户端只能访问到服务器静态资源文件夹的图片, 而工程磨人的静态资源文件夹只有static, 通过以下配置代码可以将files文件夹也设置为静态资源文件夹,这样客户端就可以访问到了

![image-20220913114930036](day05.assets/image-20220913114930036.png)

7. 上传图片完成时会调用某个方法,需要通过以下代码进行配置,在调用的方法中可以获得上传成功的图片路径(服务器中保存在磁盘上面的唯一文件名)

   ![image-20220913115115102](day05.assets/image-20220913115115102.png)



### 表设计面试题    

2021年过年时小明在这些天都收到了许多亲戚\朋友还有同事的红包,也发出了一些红包,有的是微信,有的是支付宝也有现金,请参考下面的题目帮小明设计表格保存红包的信息

1. 设计表 (至少包含一张流水表) 

- 先列出需要保存的所有信息

  - 人物关系,红包类型,红包金额,红包时间,性别,名字

  - 流水表: id,红包金额,红包类型,时间,人物id

    create table trade(id int primary key auto_increment,money int,type varchar(10),time date,p_id int );

  - 人物表: id, 名字,性别,关系

    create table person(id int primary key auto_increment,name varchar(20),gender char(1),rel varchar(10));
    
  - 往人物表中插入以下数据:
刘德华,男,亲戚           杨幂,女,亲戚             马云,男,同事               特朗普,男,朋友 
    
    貂蝉,女,朋友
    
    insert into person values(null,'刘德华','男','亲戚'),(null,'杨幂','女','亲戚'),(null,'马云','男','同事'),(null,'特朗普','男','朋友'),(null,'貂蝉','女','朋友');
    
  - 往流水表中插入以下数据
  
  ```
      刘德华 微信 收1000 2021-03-20   
      杨幂 现金 收500 发50 2021-04-14    
      马云 支付宝 收20000 发5 2021-03-11    
      特朗普 微信 收2000  2021-05-18    
      貂蝉 微信 发20000 2021-07-22
  ```
  
  ​    insert into trade values(null,1000,'微信','2021-03-20',1),
  
  (null,500,'现金','2021-04-14',2),(null,-50,'现金','2021-04-14',2),
  
  (null,20000,'支付宝','2021-03-11',3),(null,-5,'支付宝','2021-03-11',3),
  
  (null,2000,'微信','2021-05-18',4),
  
  (null,-20000,'微信','2021-07-22',5);
  
  update trade set money=-20000 where id=7;

1. 统计2021年2月15号到现在的所有红包收益

   select sum(money) from trade where time>='2021-2-15';

2. 查询2021年2月15号到现在 金额大于100 所有女性亲戚的名字和金额

   select name,money

   from trade t join person p on t.p_id=p.id

   where time>'2021-2-15' and abs(money) >100 and gender='女' and rel='亲戚';

3. 查询三个平台(微信,支付宝,现金)分别收入的红包金额  

   select type,sum(money) from trade where money>0 group by type ;



