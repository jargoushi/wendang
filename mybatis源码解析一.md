# mybatis的初始化

> mybatis初始化时会解析全局配置文件及映射文件，将xml信息或注解中的配置存储到内存中，以方便mybatis运行时使用

# 用于调试源码数据

## 全局配置文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">

<configuration>

  <!--引用数据库配置文件-->
  <properties resource="resources/jdbc.properties"/>

  <settings>
    <setting name="cacheEnabled" value="true"/>
    <setting name="lazyLoadingEnabled" value="false"/>
    <setting name="multipleResultSetsEnabled" value="true"/>
    <setting name="useColumnLabel" value="true"/>
    <setting name="useGeneratedKeys" value="false"/>
    <setting name="defaultExecutorType" value="SIMPLE"/>
    <setting name="defaultStatementTimeout" value="25"/>
  </settings>

  <typeAliases>
    <typeAlias alias="Author" type="org.apache.ibatis.domain.blog.Author"/>
    <typeAlias alias="Blog" type="org.apache.ibatis.domain.blog.Blog"/>
    <typeAlias alias="Comment" type="org.apache.ibatis.domain.blog.Comment"/>
    <typeAlias alias="Post" type="org.apache.ibatis.domain.blog.Post"/>
    <typeAlias alias="Section" type="org.apache.ibatis.domain.blog.Section"/>
    <typeAlias alias="Tag" type="org.apache.ibatis.domain.blog.Tag"/>
  </typeAliases>

  <typeHandlers>
    <typeHandler javaType="String" jdbcType="VARCHAR" handler="org.apache.ibatis.builder.CustomStringTypeHandler"/>
  </typeHandlers>

  <objectFactory type="org.apache.ibatis.builder.ExampleObjectFactory">
    <property name="objectFactoryProperty" value="100"/>
  </objectFactory>

  <plugins>
    <plugin interceptor="org.apache.ibatis.builder.ExamplePlugin">
      <property name="pluginProperty" value="100"/>
    </plugin>
  </plugins>

  <!--使用development环境且配置数据库连接四大参数-->
  <environments default="development">
    <environment id="development">
      <transactionManager type="JDBC">
        <property name="" value=""/>
      </transactionManager>
      <dataSource type="UNPOOLED">
        <property name="driver" value="${driver}"/>
        <property name="url" value="${url}"/>
        <property name="username" value="${username}"/>
        <property name="password" value="${password}"/>
      </dataSource>
    </environment>
  </environments>

  <!--引用映射文件-->
  <mappers>
    <mapper resource="resources/GoodMapper.xml"/>
  </mappers>

</configuration>

```

## 数据库连接配置

```properties
driver=com.mysql.jdbc.Driver
url=jdbc:mysql://localhost:3306/miaosha?useUnicode=true&characterEncoding=utf-8&serverTimezone=UTC
username=root
password=123456
```

## 映射文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<!-- namespace为接口方法名 -->
<mapper namespace="org.apache.ibatis.mapper.GoodMapper">

    <!-- id为接口方法名 -->
  <select id="queryGoods" resultType="org.apache.ibatis.domain.Goods">
    select
      id, goods_name goodsName, goods_img goodsImg,
      goods_price goodsPrice, goods_stock goodsStock
    from
      goods
  </select>

  <insert id="insertGoods">
    insert into goods(id, goods_name, goods_img, goods_price, goods_stock)
    values (#{id}, #{goodsName}, #{goodsImg}, #{goodsPrice}, #{goodsStock})
  </insert>


</mapper>

```

## 接口

```java
package org.apache.ibatis.mapper;

import org.apache.ibatis.domain.Goods;

import java.util.List;

public interface GoodMapper {

  List<Goods> queryGoods();

  int insertGoods(Goods goods);
}

```

## 测试代码

```java
import org.apache.ibatis.domain.Goods;
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.mapper.GoodMapper;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import org.junit.Test;

import java.io.IOException;
import java.io.InputStream;

public class MybatisTest {

  @Test
  public void test() throws IOException {

    // 加载mybatis的全局配置文件
    InputStream inputStream = Resources.getResourceAsStream("resources/MapperConfig.xml");
    // 执行mybatis的初始化流程
    SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(inputStream);
    // 获取到sqlSession
    SqlSession sqlSession = factory.openSession(true);
    // 获取Mapper代理对象
    GoodMapper mapper = sqlSession.getMapper(GoodMapper.class);

    Goods good = new Goods();
    good.setId(5);
    good.setGoodsName("西瓜");
    good.setGoodsImg("http://www.baidu.com");
    good.setGoodsPrice(3.8D);
    good.setGoodsStock(1000);

    // 执行代理对象的方法
    int val = mapper.insertGoods(good);
    // 获取响应结果
    System.out.println("插入数据库的条数:" + val);
  }
}

```

# 配置文件解析

## 解析全局配置文件

根据全局配置文件路径使用dom4j解析为document对象

## 解析configuration节点

document的configuration节点为需要的配置

## 解析properties节点

![image-20201015150208036](https://boke-test.oss-cn-shanghai.aliyuncs.com/boke-test.oss-cn-shanghai.aliyuncs.compzyjbnXMuHmoR7P.png)

解析数据库连接配置文件中的数据，存储到configuration对象的variables变量中

## 解析其他标签

- settings
- typeAliases
- plugins
- objectFactory
- reflectorFactory
- environments
- databaseIdProvider
- typeHandlers

## 解析mapper节点

1. 根据映射文件地址解析映射文件

2. 解析映射文件中的mapper节点![image-20201015151313818](https://boke-test.oss-cn-shanghai.aliyuncs.com/boke-test.oss-cn-shanghai.aliyuncs.comNTWf9MwJ2sHdEyP.png)

3. 主要看一下select， insert, update, delete节点

   > 我们在xml中会在这四个节点下写CRUD sql语句以及一些动态标签

## 解析select|update|delete|insert节点

![image-20201015152642426](https://boke-test.oss-cn-shanghai.aliyuncs.com/boke-test.oss-cn-shanghai.aliyuncs.comjIxmH9YNtkJeT2E.png)

![image-20201015153406746](https://boke-test.oss-cn-shanghai.aliyuncs.com/boke-test.oss-cn-shanghai.aliyuncs.comk163YwbjKcNrXIJ.png)

1. 解析id, sql类型， sql语句，parameterType，resultType等信息

2. 创建MappedStatement对象，存储节点信息

   ![image-20201015155139080](https://boke-test.oss-cn-shanghai.aliyuncs.com/boke-test.oss-cn-shanghai.aliyuncs.comrn5TzjMQG34siNy.png)

3. 将MappedStatement对象存储到configuration对象的mappedStatements容器中

   ![image-20201015155333666](https://boke-test.oss-cn-shanghai.aliyuncs.com/boke-test.oss-cn-shanghai.aliyuncs.comgb1tscIuPEJkAmV.png)

# configuration对象

> 配置文件中的所有信息解析成功后都会存储到configuration对象中

![image-20201015155708779](https://boke-test.oss-cn-shanghai.aliyuncs.com/boke-test.oss-cn-shanghai.aliyuncs.comNf2lJCbcLP8jqOx.png)

对应了映射文件中的select节点和insert节点

# 总结

mybatis的初始化过程

> 1. 解析mybatis全局配置文件
> 2. 解析mybatis映射文件
> 3. 解析映射文件select, update, insert, delete节点为MappedStatement对象
> 4. 将信息组装到configuration对象中