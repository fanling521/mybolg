# MyBatis 教程

## 什么是MyBatis

MyBatis是支持普通`SQL`查询，存储过程和高级映射的优秀持久层框架，消除了几乎所有的`JDBC`代码和参数，使用简单的`XML`或注解用于配置和原始映射，可将接口和普通的`Java`对象映射成数据库中的记录。

## MyBatis操作说明

### 基本使用步骤

####  第一步：新增全局配置文件

**mybatis-conf.xml**中配置环境，包含事务管理和数据源。

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <!--Environment环境类，是一个单例类，可以配置多个环境，但是启动后只有一个-->
    <environments default="development">
        <!--可新增多个环境，父类属性default配置环境-->
        <environment id="development">
            <!--JDBC/MANAFED-->
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/user"/>
                <property name="username" value="fanl"/>
                <property name="password" value="fanl@0920"/>
            </dataSource>
        </environment>
    </environments>
    <mappers>
          <!--每次新增都需要加上映射文件的路径-->
        <mapper resource="mapper/UserMapper.xml"/>
    </mappers>
</configuration>
```

####  第二步  新增实体类

```java
public class SysUser {
    private long id;
    private String userName;
    private String pwd;
    private String nickName;
    private int status;
    //getter setter...略
}
```

####  第三步 增删查改的接口

```java
public interface UserMapper {
    /**
    * 根据id查询用户信息
    */
    SysUser selectSysUserById(long id);
}
```

####  第四步 新增映射文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!--namespace是Mapper接口的路径-->
<mapper namespace="com.fanling.demo.dao.UserMapper">
    <!--
	id            实现方法的方法名
	parameterType 实现方法参数的类型
	resultType    实现方法返回类型
	-->
    <select id="selectSysUserById" parameterType="long" resultType="com.fanling.demo.entity.SysUser">
        select * from sysuser where 1=1 and id=#{id}
    </select>
</mapper>
```

####  第五步 使用测试

实例化`SqlSessionFactory`对象，需要通过`SqlSessionFactoryBuilder`来创建，再实例化`SqlSession`对象，调用对象的`getMapper`泛型方法，然后直接使用**dao**类即可。

```java
public class App {
    public static void main(String[] args) {
        try {
            // 引入全局配置文件
            InputStream inputStream= 
                Resources.getResourceAsStream("mybatis-conf.xml");
            // 创建SqlSessionFactory
            SqlSessionFactory sqlSessionFactory= 
                new SqlSessionFactoryBuilder().build(inputStream);
            // 创建SqlSession
            SqlSession sqlSession=sqlSessionFactory.openSession();
            // 创建Dao
            UserMapper mapper = sqlSession.getMapper(UserMapper.class);
            System.out.println(mapper.selectSysUserById(3));
            // 关闭SqlSession
            sqlSession.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

### 其他特殊配置说明

#### 外部数据源配置文件

```xml
<properties resource="jdbc.propertise"/>
<environments default="development">
	<environment id="development">
		<transactionManager type="JDBC"></transactionManager>
		<dataSource type="POOLED">
			<property name="driver" value="${db.driver}"/>
			<property name="url" value="${db.url}"/>
			<property name="username" value="${db.username}"/>
			<property name="password" value="${db.password}"/>
		</dataSource>
	</environment>
</environments>

```

#### 配置使用实体类的别名

MyBatis首先为一些内置的数据类型做了别名映射，如写`int`或者`Integer`都可以。

一个别名映射，在全局配置文件**mybatis-conf.xml**中

```xml
<typeAliases>
	<typeAlias type="com.fanling.demo.entity.SysUser" alias="SysUser"/>
</typeAliases>
<!--多个别名映射[推荐]-->
<typeAliases>
	<package name="com.fanling.demo.entity"/>
</typeAliases>
```

## MyBatis的动态SQL

除了使用的中的**select**、**insert**、**update**、**delete**外，还有一些常见的标签格式。

### 常见的特殊标签

> where 标签

```xml
<!--不用写where-->
<where>
	<if test="name!=null and name!=''">
    and name=#{name}
	</if>
</where>
```

> if 标签

```xml
<!--时间格式不可比较-->
<if test="name!=null and name!=''">
    and name=#{name}
</if>
```

> set 标签，用于update 标签

```xml
<!--update 语句中不用写set，存在不合适的逗号也会自动删除-->
<set>
	<if test="name!=null and name!=''">
    name=#{name},
	</if>
</set>
```

> foreach 标签，可用于in查询

```xml
<!--
	collection属性     array/map/list
	open属性           开始的字符
	close             结束的字符
	separator         分隔符
-->
<select id="selectMulStudentByIds" parameterType="list">
	select * from student where 
	<foreach collection="list" item="item" open="id in(" close=")" separator=",">
		 #{item}
	</foreach>
</select>
```

> resultMap，映射实体类和数据库字段

```xml
<resultMap id="" type="">
 	<!--
		id: 一般是实体名+ResultMap
		type: 实体名的全路径或者别名配置
	-->
    <!--主键-->
    <id property="{实体类属性名}" column="{数据库列名}"/>
    <!--其他普通字段-->
    <result property="{实体类属性名}" column="{数据库列名}"/>
</resultMap>
<!--
还可以实现映射关系
-->
```

### 映射关系

场景：实际使用中，表的关系很复杂，多表查询等。

#### 一对一

在`resultMap`中新增映射关系，一般是实体类中有其他实体属性的。

```xml
<association property="" javaType="">
    <!--
	property：映射的属性名
	javaType：映射属性所属的泛型类名
	-->
</association>
```

#### 一对多

在`resultMap`中新增映射关系，一般实体类存在集合属性的。

```xml
<collection property="" ofType="">
    <!--
	property：List<?> 的属性名
	ofType：List<?> 的泛型类名
	-->
</collection>
```

#### 多对多

在`resultMap`中新增映射关系

```xml
<collection property="" ofType="">
    <!--
	property：List<?> 的属性名
	ofType：List<?> 的泛型类名
	-->
    <association property="" javaType="">
    	<!--
		property：T 的属性名
		javaType：T 的泛型类名
		-->
	</association>
</collection>
```

## MyBatis缓存

### 一级缓存

一级缓存是`SqlSession`级别的缓存。在操作数据库时需要构造`sqlSession`对象，在对象中有一个数据结构（HashMap）用于存储缓存数据。不同的`sqlSession`之间的缓存数据区域（HashMap）是互相不影响的。

执行commit操作会更新缓存。

具体的执行过程：

- 第一次发起查询，先去找缓存中查询，如果没有，从数据库查询信息，将信息存储到一级缓存中。
- 如果期间`sqlSession`去执行了**commit**操作，则会清空`SqlSession`中的一级缓存，这样做的目的为了让缓存中存储的是最新的信息，避免**脏读**。
- 第二次发起相同的查询先去找缓存中，缓存中有，直接从缓存中获取信息。

### 二级缓存

二级缓存是**mapper**级别的缓存，多个`SqlSession`去操作同一个**Mapper**的sql语句，多个`SqlSession`可以共用二级缓存，二级缓存是跨`SqlSession`的。

二级缓存是基于mapper文件的`namespace`的，也就是说多个`sqlSession`可以共享一个mapper中的二级缓存区域。

**开启二级缓存**

首先在全局配置文件 **mybatis-conf.xml** 文件中加入如下代码：

```xml
<!--开启二级缓存  -->
<settings>
    <setting name="cacheEnabled" value="true"/>
</settings>
```
其次在 ***Mapper.xml** 文件中开启缓存，同时实体类需要实现序列化接口。
```xml
<!-- 开启二级缓存 -->
<cache />
```

对于访问多的查询请求且用户对查询结果实时性要求不高，可以使用二级缓存，因为是单服务器的缓存，通常会整合第三方框架，如`ehcache`。

## MyBatis的注意事项

1. MyBatis配置文件要按一定的顺序。如以下顺序：properties，settings， typeAliases，environments，databaseIdProvider，mappers，具体约束看`mybatis-3-config.dtd`
2. `${}`和`#{}`的区别，`${}`是sql中的连接符，存在sql注入隐患，`#{}=?`是表示一个占位符
