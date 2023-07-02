# 2. 通过Mybatis实现数据库编程

## 2.11. 根据id查询相册详情（续）

则在项目的根包下创建`pojo.vo.AlbumStandardVO`类：

```java
package cn.tedu.csmall.product.pojo.vo;

import lombok.Data;

import java.io.Serializable;

/**
 * 相册的标准VO类
 *
 * @author java@tedu.cn
 * @version 0.0.1
 */
@Data
public class AlbumStandardVO implements Serializable {

    /**
     * 记录id
     */
    private Long id;

    /**
     * 相册名称
     */
    private String name;

    /**
     * 相册简介
     */
    private String description;

    /**
     * 自定义排序序号
     */
    private Integer sort;

}
```

在`AlbumMapper`接口中添加抽象方法：

```java
/**
 * 根据id查询相册标准信息
 *
 * @param id 相册id
 * @return 匹配的相册的标准信息，如果没有匹配的数据，则返回null
 */
AlbumStandardVO getStandardById(Long id);
```

在`AlbumMapper.xml`中配置以上抽象方法映射的SQL语句：

```xml
<!-- AlbumStandardVO getStandardById(Long id); -->
<select id="getStandardById" resultType="cn.tedu.csmall.product.pojo.vo.AlbumStandardVO">
    SELECT id, name, description, sort
    FROM pms_album
    WHERE id=#{id}
</select>
```

完成后，`AlbumMapperTests`测试类中编写并执行测试方法：

```java
@Test
void testGetStandardById() {
    Long id = 5L;
    Object result = mapper.getStandardById(id);
    System.out.println("根据id=" + id + "查询标准信息完成，结果=" + result);
}
```

## 2.12. 练习：根据id查询属性模板详情

## 2.12. 练习：根据id查询品牌详情

提示：品牌表是`pms_brand`。

## 2.13. 关于`<resultMap>`

当Mybatis处理查询的结果集时，会自动将**列名（Column）**与**属性名（Property）**相同的数据进行封装，例如，将查询结果集中名为`name`的数据封装到对象的`name`属性中，并且，默认情况下，对于列名与属性名不同的数据，不予处理！

> 提示：查询结果集中的列名（Column）默认是字段名（Field），而字段名是设计数据表时指定的。

在XML文件中，可以配置`<resultMap>`标签，此标签的作用就是：指导Mybatis将查询结果集中的数据封装到对象中。

建议通过`<resultMap>`标签配置列名与属性名的对应关系，例如：

```xml
<resultMap id="StandardResultMap" type="cn.tedu.csmall.product.pojo.vo.BrandStandardVO">
    <result column="product_count" property="productCount"/>
    <result column="comment_count" property="commentCount"/>
    <result column="positive_comment_count" property="positiveCommentCount"/>
</resultMap>
```

> 提示：在单表查询时，列名与属性名本来就相同的部分，可以不必在`<resultMap>`进行配置。

然后，在`<select>`标签上，不再配置`resultType`，而改为配置`resultMap`，且此属性的值就是`<resultMap>`标签的id值，例如：

```xml
<!-- BrandStandardVO getStandardById(Long id); -->
<select id="getStandardById" resultMap="StandardResultMap">
    暂不关心SQL语句部分
</select>
```

配置的`<resultMap>`是可以复用的，即：如果另一个`<select>`查询的结果也使用相同的VO类进行封装，则另一个`<select>`也配置相同的`resultMap`即可。

由于`<resultMap>`对应特定的VO类，而VO类是与字段列表对应的，所以，如果多个`<select>`复用了同一个`<resultMap>`，那这些`<select>`查询的字段列表必然是相同的，则可以通过`<sql>`和`<include>`来复用字段列表，例如：

```xml
<!-- BrandStandardVO getStandardById(Long id); -->
<select id="getStandardById" resultMap="StandardResultMap">
    SELECT
        <include refid="StandardQueryFields"/>
    FROM
        pms_brand
    WHERE
        id=#{id}
</select>

<sql id="StandardQueryFields">
    id, name, pinyin, logo, description,
    keywords, sort, sales, product_count, comment_count,
    positive_comment_count, enable
</sql>
```

## 2.14. 查询相册列表

需要执行的SQL语句大致是：

```mysql
SELECT id, name, description, sort FROM pms_album ORDER BY id DESC
```

在许多数据的查询功能中，查询详情（标准信息）和查询列表时，需要查询的字段列表很可能是不同的，所以，应该使用不同的VO类（为了避免后续维护添加字段导致的调整，即使当前查询详情和查询列表的字段完全相同，也应该使用不同的VO类）！

在根包下创建`pojo.vo.AlbumListItemVO`类：

```java

```

在`AlbumMapper`接口中添加抽象方法：

```java
List<AlbumListItemVO> list();
```

在`AlbumMapper.xml`中配置SQL语句：

```xml
<select id="list" resultMap="ListResultMap">
	SELECT <include refid="ListQueryFields"/> 
    FROM pms_album 
    ORDER BY id DESC
</select>

<sql id="ListQueryFields">
    id, name, description, sort
</sql>

<resultMap id="ListResultMap" type="cn.tedu.csmall.product.pojo.vo.AlbumListItemVO">
</resultMap>
```

在`AlbumMapperTests`中编写并执行测试：

```java
@Test
void testList() {
    List<?> list = mapper.list();
    System.out.println("查询列表完成，列表中的数据的数量=" + list.size());
    for (Object item : list) {
        System.out.println(item);
    }
}
```

## 2.14. 练习：查询属性模板列表

## 2.15. 动态SQL：`<foreach>`

Mybatis的动态SQL机制表现为：根据参数的不同，生成不同的SQL语句。

例如需要实现：根据若干个id批量删除相册数据。

需要执行的SQL语句大致是：

```mysql
DELETE FROM pms_album WHERE id=? OR id=? ... OR id=?
```

或：

```mysql
DELETE FROM pms_album WHERE id IN (?, ?, ... ?);
```

首先，在`AlbumMapper`接口中添加抽象方法，可以是：

```java
int deleteByIds(List<Long> ids);
```

也可以是：

```java
int deleteByIds(Long[] ids);
```

还可以是：

```java
int deleteByIds(Long... ids);
```

> 提示：可变参数的本质仍是一个数组！

然后，在`AlbumMapper.xml`中配置SQL语句：

```xml
<!-- int deleteByIds(Long[] ids); -->
<delete id="deleteByIds">
    DELETE FROM pms_album WHERE id IN (
    	<foreach collection="array" item="id" separator=",">
            #{id}
    	</foreach>
    )
</delete>
```

或：

```xml
<!-- int deleteByIds(Long[] ids); -->
<delete id="deleteByIds">
    DELETE FROM pms_album WHERE 
    <foreach collection="array" item="id" separator=" OR ">
        id=#{id}
    </foreach>
</delete>
```

关于`<foreach>`标签的属性配置：

- `collection`：表示被遍历的参数列表，如果抽象方法的参数只有1个，当参数类型是`List`集合类型时，当前属性取值为`list`，当参数类型是数组类型时，当前属性取值为`array`
- `item`：用于指定遍历到的各元素的变量名，并且，在`<foreach>`的子级，使用`#{}`时的名称也是当前属性指定的名字
- `separator`：用于指定遍历过程中各元素的分隔符（或字符串等）

完成后，在`AlbumMapperTests`中编写并执行测试：

```java
@Test
void testDeleteByIds() {
    Long[] ids = {1L, 3L, 5L, 7L, 9L};
    int rows = mapper.deleteByIds(ids);
    System.out.println("批量删除数据完成，受影响的行数=" + rows);
}
```

## 2.16. 练习：批量插入相册数据

需要执行的SQL语句大致是：

```mysql
INSERT INTO pms_album (name, description, sort) VALUES (?,?,?), (?,?,?), ... (?,?,?);
```

在`AlbumMapper`接口中添加抽象方法：

```java
int insertBatch(List<Album> albumList);
```

在`AlbumMapper.xml`中配置SQL语句：

```xml
<!-- int insertBatch(List<Album> albumList); -->
<insert id="insertBatch" useGeneratedKeys="true" keyProperty="id">
    INSERT INTO pms_album (name, description, sort) VALUES
    <foreach collection="list" item="album" separator=",">
        (#{album.name}, #{album.description}, #{album.sort})
    </foreach>
</insert>
```

在`AlbumMapperTests`中编写并执行测试：

```java
@Test
void testInsertBatch() {
    List<Album> albumList = new ArrayList();
    for (int i = 1; i <= 10; i++) {
        Album album = new Album();
        album.setName("批量插入的测试相册名称" + i);
        album.setDescription("批量插入的测试相册简介" + i);
        album.setSort(66);
        albumList.add(album);
    }
    
    int rows = mapper.insertBatch(albumList);
    System.out.println("批量插入数据完成，受影响的行数=" + rows);
}
```

## 2.17. 练习：批量删除属性模板数据

## 2.18. 练习：批量插入属性模板数据

## 2.19. 动态SQL：`<if>`

假设需要修改相册表中的数据，需要执行的SQL语句大致是：

```mysql
UPDATE pms_album SET name=?, description=?, sort=? WHERE id=?
```

如果按照以上SQL来设计抽象方法，则抽象方法大致是：

```java
int updateById(Album album);
```

并且，配置以上抽象方法映射的SQL语句：

```xml
<!-- int updateById(Album album); -->
<update id="updateById">
    UPDATE
        pms_album
    SET
        name=#{name},
        description=#{description},
        sort=#{sort}
    WHERE
        id=#{id}
</update>
```

如果采取以上做法，就无法实现“只修改部分字段的值”！例如：只修改`sort`时，如果在`Album`对象中只封装了`id`和`sort`属性值，则`name`和`description`这2个属性的值就是`null`，在执行SQL语句时，将会把表中原有数据的`name`和`description`字段的值更新为`null`，这是不符合原本的需求的！

期望的执行效果应该是：传入了对应的值，则更新对应字段的值，对于没有传入参数的部分，也不更新表中对应字段的数据！

如果要实现以上效果，则需要使用动态SQL中的`<if>`，这个标签的作用就是对参数进行判断的！

```xml
<!-- int updateById(Album album); -->
<update id="updateById">
    UPDATE
        pms_album
    <set>
        <if test="name != null">
            name=#{name},
        </if>
        <if test="description != null">
            description=#{description},
        </if>
        <if test="sort != null">
            sort=#{sort},
        </if>
    </set>
    WHERE
        id=#{id}
</update>
```

## 2.20. 动态SQL：其它

在Mybatis中，`<if>`标签并没有对应的类似Java中的`else`标签！如果需要实现类似Java中`if ... else ...`的效果，可以使用2个条件完全相反的`<if>`标签，例如：

```xml
<if test="name != null">
	某代码
</if>
<if test="name == null">
	另外一段代码
</if>
```

以上做法的缺点在于：实际上执行了2次条件的判断，在性能略微有浪费。

或者，使用`<choose>`系列标签，以实现类似`if ... else ...`的效果：

```xml
<choose>
	<when test="判断条件">
    	满足条件时的SQL片段
    </when>
    <otherwise>
    	不满足条件时的SQL片段
    </otherwise>
</choose>
```

# 3. SLF4j日志

在Spring Boot项目中，`spring-boot-starter`依赖项中已经包含日志框架！

在Spring Boot项目中，当添加了Lombok依赖项后，可以在任何类上添加`@Slf4j`注解，则Lombok会在编译期声明一个名为`log`的日志对象变量，此变量可以调用相关方法来输出日志！

```java
package cn.tedu.csmall.product;

import lombok.extern.slf4j.Slf4j;
import org.junit.jupiter.api.Test;
import org.springframework.boot.test.context.SpringBootTest;

@Slf4j
@SpringBootTest
public class Slf4jTests {

    @Test
    void testLog() {
        log.info("输出了一条日志……");
    }

}
```

SLF4j的日志的可显示级别，根据信息的重要程度，从不重要到严重依次是：

- `trace`
- `debug`
- `info`：一般信息
- `warn`：警告信息
- `error`：错误信息

调用`log`变量来输出日志时，可以使用以上级别对应的方法，则可以输出对应级别的日志！

在Spring Boot项目中，日志的默认显示级别是`info`，则默认情况下只会显示`info`及更加严重的级别的日志！如果需要修改日志的显示级别，需要在`application.properties`中配置`logging.level`的属性，例如：

```properties
# 日志的显示级别
logging.level.cn.tedu.csmall=info
```

**注意：在配置以上属性时，必须在`logging.level`右侧加上要配置显示级别的包的名称，此包名就是配置日志显示级别的根包。**


























# 作业

补全`mall_pms`数据库中所有数据表的以下数据访问功能，要求均有对应的测试：

- 插入数据功能
    - 注意：`pms_spu`、`pms_sku`这2张表的`id`不是自动编号的，所以，在插入数据时，必须提供`id`字段的值，并且，在配置XML中的`<insert>`时，不需要配置`useGeneratedKeys`和`keyProperty`属性
- 批量插入数据功能
- 根据id删除数据
- 根据若干个id批量删除数据
- 根据id修改数据
- 统计表中数据的数量
- 根据id查询数据详情
- 查询数据列表

推荐开发优先级：相册(`pms_album`) > 属性模板(`pms_attribute_template`) > 品牌(`pms_brand`) > 属性(`pms_attribute`) > 类别(`pms_category`) > 其它

开发进度要求：
- 9月26日前完成：相册、属性模板、品牌、属性、类别
- 9月30日前完成：全部

提交截止时间：2022-09-30 23:00