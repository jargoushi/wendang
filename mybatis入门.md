# Mybatis是什么

> MyBatis 是一款优秀的**持久层框架**，它支持定制化SQL、存储过程以及高级映射。MyBatis 避免了几乎所有的 JDBC代码和手动设置参数以及获取结果集，它可以使用简单的**XML**或**注解**来配置和映射SQL信息，将接口和 Java 的 POJOs(Plain Old Java Objects,普通的 Java对象)映射成数据库中的记录。

# 开发流程

加粗的流程只需搭建框架时进行配置

1. **引用相关pom依赖**
2. **创建mybatis全局配置文件**
3. 编写实体类
4. 编写持久化接口
5. 编写映射文件
6. 调用相关api操作
   1. 加载配置文件
   2. 获取sqlSessionFactory
   3. 获取sqlSession
   4. 获取Mapper代理对象
   5. 执行代理对象的方法
   6. 获取响应结果

# 框架搭建

## 引用pom依赖

```xml
<dependencies>
	<!-- mybatis依赖 -->
    <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis</artifactId>
      <version>3.4.6</version>
    </dependency>

	<!-- 数据库依赖 -->
    <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>5.1.47</version>
    </dependency>

	<!-- 单元测试依赖 -->
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.12</version>
      <scope>test</scope>
    </dependency>
  </dependencies>
```

## 数据库配置文件

````properties
driver=com.mysql.jdbc.Driver
url=jdbc:mysql://localhost:3306/miaosha?useUnicode=true&characterEncoding=utf-8&serverTimezone=UTC
username=root
password=123456
````

## mybatis全局配置文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
    PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
    "http://mybatis.org/dtd/mybatis-3-config.dtd">

<configuration>

   <!--引用数据库配置文件-->
  <properties resource="resources/jdbc.properties"/>

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

## sql初始化脚本

```sql
CREATE TABLE `goods`  (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '商品ID',
  `goods_name` varchar(16) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL COMMENT '商品名称',
  `goods_title` varchar(64) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL COMMENT '商品标题',
  `goods_img` varchar(64) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL COMMENT '商品的图片',
  `goods_detail` longtext CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL COMMENT '商品的详情介绍',
  `goods_price` decimal(10, 2) NULL DEFAULT 0.00 COMMENT '商品单价',
  `goods_stock` int(11) NULL DEFAULT 0 COMMENT '商品库存，-1表示没有限制',
  PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 1 CHARACTER SET = utf8mb4 COLLATE = utf8mb4_general_ci ROW_FORMAT = Compact;

INSERT INTO `goods` VALUES (1, 'iphoneX', 'Apple iPhone X (A1865) 64GB 银色 移动联通电信4G手机', '/img/iphonex.png', 'Apple iPhone X (A1865) 64GB 银色 移动联通电信4G手机', 8765.00, 10000);
INSERT INTO `goods` VALUES (2, '华为Meta9', '华为 Mate 9 4GB+32GB版 月光银 移动联通电信4G手机 双卡双待', '/img/meta10.png', '华为 Mate 9 4GB+32GB版 月光银 移动联通电信4G手机 双卡双待', 3212.00, -1);
INSERT INTO `goods` VALUES (3, 'iphone8', 'Apple iPhone 8 (A1865) 64GB 银色 移动联通电信4G手机', '/img/iphone8.png', 'Apple iPhone 8 (A1865) 64GB 银色 移动联通电信4G手机', 5589.00, 10000);
INSERT INTO `goods` VALUES (4, '小米6', '小米6 4GB+32GB版 月光银 移动联通电信4G手机 双卡双待', '/img/mi6.png', '小米6 4GB+32GB版 月光银 移动联通电信4G手机 双卡双待', 3212.00, 10000);
```



# 需求1

> 查询所有商品信息

## 实体对象

````java
package org.apache.ibatis.domain;

public class Goods {

  private int id;

  private String goodsName;

  private String goodsImg;

  private String goodsDetail;

  private Double goodsPrice;

  private int goodsStock;

  // 省略set,get,toString
}

````

## 编写dao接口

```java
package org.apache.ibatis.mapper;

import org.apache.ibatis.domain.Goods;

import java.util.List;

public interface GoodMapper {

  List<Goods> queryGoods();
}

```

## 映射文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<!-- namespace为接口的全限定名 -->
<mapper namespace="org.apache.ibatis.mapper.GoodMapper">

    <!-- id为接口中的方法名, resultType为映射对象的全限定名 -->
  <select id="queryGoods" resultType="org.apache.ibatis.domain.Goods">
    select
      id, goods_name goodsName, goods_img goodsImg,
      goods_price goodsPrice, goods_stock goodsStock
    from
      goods
  </select>

</mapper>

```

## 单元测试

```java
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
    // 执行代理对象的方法
    List<Goods> goods = mapper.queryGoods();
    // 获取响应结果
    for (Goods good : goods) {
      System.out.println(good);
    }
  }
```

## 执行结果

```

```

![image-20201013175454665](https://i.loli.net/2020/10/13/9m7yrENGRegBT5L.png)

```

```

# 需求2

> 插入商品信息

## 实体对象

```java
package org.apache.ibatis.domain;

public class Goods {

  private int id;

  private String goodsName;

  private String goodsImg;

  private String goodsDetail;

  private Double goodsPrice;

  private int goodsStock;

  // 省略set,get,toString
}
```

## 编写dao接口

```java
public interface GoodMapper {

  List<Goods> queryGoods();

  int insertGoods(Goods goods);
}

```

## 映射文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="org.apache.ibatis.mapper.GoodMapper">

  <insert id="insertGoods">
    insert into goods(id, goods_name, goods_img, goods_price, goods_stock)
    values (#{id}, #{goodsName}, #{goodsImg}, #{goodsPrice}, #{goodsStock})
  </insert>


</mapper>
```

## 单元测试

```java
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
```

## 执行结果

![image-20201013180442350](https://i.loli.net/2020/10/13/qLno7Nd4Wg5Ebfe.png)

## 查看数据库记录

![image-20201013180511784](https://i.loli.net/2020/10/13/qBsOnLaGdeRMTP8.png)