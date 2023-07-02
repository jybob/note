# 61. 在项目中使用Redis

由于Redis的存取效率非常高，在开发实践中，通常会将一些数据从关系型数据库（例如MySQL）中读取出来，并写入到Redis中，后续，当需要访问相关数据时，将优先从Redis中读取所需的数据，以此，可以提高数据的读取效率，并且，对一定程度的保护关系型数据库。

一旦使用Redis后，相关的数据就会同时存在于关系型数据和Redis中，即同一个数据有2份或更多（如果你使用了更多的Redis服务或其它数据处理技术），则可能出现数据不同步的问题！例如，当修改了或删除了关系型数据库中的数据，那Redis中的数据应该如何处理？同时更新？还是无视数据的变化？如果最终出现了关系型数据库和Redis中的数据不同的问题，则称之为“数据一致性问题”。

关于数据可能存在不一致的问题，首先，你必须知道，并不是所有的数据都必须同步，也就是说，当关系型数据库中的数据变化后，如果Redis中的数据没有同步发生变化，则Redis中的数据可以视为是“不准确的”，这个问题在许多应用场景中是可以接受的！例如热门话题的排行榜，或车票的余票数量、商品的库存余量等。

通常，应该Redis的前提应该是：

- 高频率访问的数据
  - 例如热门榜单
- 修改频率非常低的数据
  - 例如电商平台中商品的类别
- 对数据的“准确性”（一致性）要求不高的
  - 例如商品的库存余量

# 62. 应用Redis

在项目中应用Redis主要需要实现：

- 将数据从MySQL中读出
  - 已经由Mapper实现
- 【XX时】向Redis中写入
- 当需要读取数据时，将原本的从MySQL中读取数据改为从Redis中读取

推荐创建专门用于读写Redis的组件，则在项目的根包下创建`repo.IBrandRedisRepository`接口：

```java
public interface IBrandRedisRepository {}
```

并在项目的根包下创建`repo.impl.BrandRedisRepositoryImpl`类，实现以上接口，并在类上添加`@Repository`注解：

```java
@Repository
public class BrandRedisRepositoryImpl implements IBrandRedisRepository {}
```

然后，在`IBrandRedisRepository`接口中添加抽象方法：

```java
package cn.tedu.csmall.product.repo;

import cn.tedu.csmall.product.pojo.vo.BrandListItemVO;
import cn.tedu.csmall.product.pojo.vo.BrandStandardVO;

import java.util.List;

/**
 * 处理品牌缓存的数据访问接口
 *
 * @author java@tedu.cn
 * @version 0.0.1
 */
public interface IBrandRedisRepository {

    /**
     * 品牌数据项在Redis中的Key前缀
     */
    String BRAND_ITEM_KEY_PREFIX = "brand:item:";
    /**
     * 品牌列表在Redis中的Key
     */
    String BRAND_LIST_KEY = "brand:list";

    /**
     * 向Redis中写入品牌数据
     *
     * @param brandStandardVO 品牌数据
     */
    void save(BrandStandardVO brandStandardVO);

    /**
     * 向Redis中写入品牌列表
     *
     * @param brands 品牌列表
     */
    void save(List<BrandListItemVO> brands);

    /**
     * 从Redis中读取品牌数据
     *
     * @param id 品牌id
     * @return 匹配的品牌数据，如果没有匹配的数据，则返回null
     */
    BrandStandardVO get(Long id);

    /**
     * 从Redis中读取品牌列表
     *
     * @return 品牌列表
     */
    List<BrandListItemVO> list();

    /**
     * 从Redis中读取品牌列表
     *
     * @param start 读取数据的起始下标
     * @param end   读取数据的截止下标
     * @return 品牌列表
     */
    List<BrandListItemVO> list(long start, long end);

}
```

并在`BrandRedisRepositoryImpl`中实现以上方法：

```java
package cn.tedu.csmall.product.repo.impl;

import cn.tedu.csmall.product.pojo.vo.BrandListItemVO;
import cn.tedu.csmall.product.pojo.vo.BrandStandardVO;
import cn.tedu.csmall.product.repo.IBrandRedisRepository;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.ListOperations;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.stereotype.Repository;

import java.io.Serializable;
import java.util.ArrayList;
import java.util.List;

/**
 * 处理品牌缓存的数据访问实现类
 *
 * @author java@tedu.cn
 * @version 0.0.1
 */
@Slf4j
@Repository
public class BrandRedisRepositoryImpl implements IBrandRedisRepository {

    @Autowired
    RedisTemplate<String, Serializable> redisTemplate;

    public BrandRedisRepositoryImpl() {
        log.debug("创建处理缓存的数据访问对象：BrandRedisRepositoryImpl");
    }

    @Override
    public void save(BrandStandardVO brandStandardVO) {
        log.debug("准备向Redis中写入数据：{}", brandStandardVO);
        String key = getItemKey(brandStandardVO.getId());
        redisTemplate.opsForValue().set(key, brandStandardVO);
    }

    @Override
    public void save(List<BrandListItemVO> brands) {
        String key = getListKey();
        ListOperations<String, Serializable> ops = redisTemplate.opsForList();
        for (BrandListItemVO brand : brands) {
            ops.rightPush(key, brand);
        }
    }

    @Override
    public BrandStandardVO get(Long id) {
        String key = getItemKey(id);
        Serializable serializable = redisTemplate.opsForValue().get(key);
        if (serializable != null) {
            if (serializable instanceof BrandStandardVO) {
                return (BrandStandardVO) serializable;
            }
        }
        return null;
    }

    @Override
    public List<BrandListItemVO> list() {
        long start = 0;
        long end = -1;
        return list(start, end);
    }

    @Override
    public List<BrandListItemVO> list(long start, long end) {
        String key = getListKey();
        ListOperations<String, Serializable> ops = redisTemplate.opsForList();
        List<Serializable> list = ops.range(key, start, end);
        List<BrandListItemVO> brands = new ArrayList<>();
        for (Serializable item : list) {
            brands.add((BrandListItemVO) item);
        }
        return brands;
    }

    private String getItemKey(Long id) {
        return BRAND_ITEM_KEY_PREFIX + id;
    }

    private String getListKey() {
        return BRAND_LIST_KEY;
    }

}
```

完成后，在`src/test/java`的根包下创建`repo.BrandRedisRepositoryTests`测试类，编写并执行测试：

```java
package cn.tedu.csmall.product.repo;

import cn.tedu.csmall.product.pojo.entity.Brand;
import cn.tedu.csmall.product.pojo.vo.BrandListItemVO;
import cn.tedu.csmall.product.pojo.vo.BrandStandardVO;
import lombok.extern.slf4j.Slf4j;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

import java.util.ArrayList;
import java.util.List;

@Slf4j
@SpringBootTest
public class BrandRedisRepositoryTests {

    @Autowired
    IBrandRedisRepository repository;

    @Test
    void testSave() {
        BrandStandardVO brand = new BrandStandardVO();
        brand.setId(1L);
        brand.setName("华为");

        repository.save(brand);
        log.debug("向Redis中写入数据完成！");
    }

    @Test
    void testSaveList() {
        List<BrandListItemVO> brands = new ArrayList<>();
        for (int i = 1; i <= 8; i++) {
            BrandListItemVO brand = new BrandListItemVO();
            brand.setId(i + 0L);
            brand.setName("测试品牌" + i);
            brands.add(brand);
        }

        repository.save(brands);
        log.debug("向Redis中写入列表数据完成！");
    }

    @Test
    void testGet() {
        Long id = 10000L;
        Object result = repository.get(id);
        log.debug("从Redis中读取【id={}】的数据，结果：{}", id, result);
    }

    @Test
    void testList() {
        List<?> list = repository.list();
        log.debug("从Redis中读取列表，列表中的数据的数量：{}", list.size());
        for (Object item : list) {
            log.debug("{}", item);
        }
    }

    @Test
    void testListRange() {
        long start = 2;
        long end = 5;
        List<?> list = repository.list(start, end);
        log.debug("从Redis中读取列表，列表中的数据的数量：{}", list.size());
        for (Object item : list) {
            log.debug("{}", item);
        }
    }

}
```

关于在项目中应用Redis，首先考虑何时将MySQL中的数据读取出来并写入到Redis中！常见的策略有：

- 直接尝试从Redis中读取数据，如果Redis中无此数据，则从MySQL中读取并写入到Redis
  - 从运行机制上，类似单例模式中的懒汉式
- 当项目启动时，就直接从MySQL中读取数据并写入到Redis
  - 从运行机制上，类似单例模式中的饿汉式
  - 这种做法通常称之为“缓存预热”

当使用缓存预热的处理机制时，需要使得某段代码是项目启动时就自动执行的，可以自定义组件类实现`AppliacationRunner`接口，重写其中的`run()`方法，此方法将在项目启动完成之后自动调用

























# 【技能描述】

【了解/掌握/熟练掌握】开发工具的使用，包括：Eclipse、IntelliJ IDEA、Git、Maven；
【了解/掌握/熟练掌握】Java语法，【理解/深刻理解】面向对象编程思想，【了解/掌握/熟练掌握】Java SE API，包括：String、日期、IO、反射、线程、网络编程、集合、异常等；
【了解/掌握/熟练掌握】HTML、CSS、JavaScript前端技术，并【了解/掌握/熟练掌握】前端相关框架技术及常用工具组件，包括：jQuery、Bootstrap、Vue脚手架、Element UI、axios、qs、富文本编辑器等；
【了解/掌握/熟练掌握】MySQL的应用，【了解/掌握/熟练掌握】DDL、DML的规范使用；
【了解/掌握/熟练掌握】数据库编程技术，包括：JDBC、数据库连接池（commons-dbcp、commons-dbcp2、Hikari、druid），及相关框架技术，例如：Mybatis Plus等；
【了解/掌握/熟练掌握】主流框架技术的规范使用，例如：SSM（Spring，Spring MVC， Mybatis）、Spring Boot、Spring Validation、Spring Security等；
【理解/深刻理解】Java开发规范（参考阿里巴巴的Java开发手册）；
【了解/掌握/熟练掌握】基于RESTful的Web应用程序开发；
【了解/掌握/熟练掌握】基于Spring Security与JWT实现单点登录；





