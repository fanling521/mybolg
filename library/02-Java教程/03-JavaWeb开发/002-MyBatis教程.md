# MyBatis 教程

## 什么是MyBatis

MyBatis 是支持普通 SQL 查询，存储过程和高级映射的优秀持久层框架，消除 了几乎所有的 JDBC 代码和参数，使用简单的 XML 或注解用于配置和原始映射,将接口和普通的 Java 对象映射成数据库中的记录。

## 操作说明

### 单独使用，不整合框架

####  第一步：新增配置文件，名称可自定义

**mybatis-conf.xml**

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

####  第三步 新增接口

```java
public interface UserMapper {
    SysUser selectSysUserById(long id);
}
```

####  第四步 新增映射文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.fanling.demo.dao.UserMapper">
    <select id="selectSysUserById" parameterType="long" resultType="com.fanling.demo.entity.SysUser">
        select * from sysuser where 1=1 and id=#{id}
    </select>
</mapper>
```

####  第五步 使用测试

实例化`SqlSessionFactory`对象，需要通过`SqlSessionFactoryBuilder`来创建，再实例化`SqlSession`对象，调用对象的`getMapper`泛型方法，然后直接使用dao类即可。

```java
public class App {
    public static void main(String[] args) {
        try {
            //引入配置文件
            InputStream inputStream= 
                Resources.getResourceAsStream("mybatis-conf.xml");
            // 创建SqlSessionFactory
            SqlSessionFactory sqlSessionFactory= 
                new SqlSessionFactoryBuilder().build(inputStream);
            // 创建SqlSession
            SqlSession sqlSession=sqlSessionFactory.openSession();
            //创建Dao
            UserMapper mapper = sqlSession.getMapper(UserMapper.class);
            System.out.println(mapper.selectSysUserById(3));
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

### 配置信息额外说明

#### 使用外部文件

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

#### 配置使用别名

MyBatis 首先为一些内置的数据类型做了别名映射。

一个别名映射，在**mybatis-conf.xml**中

```xml
<typeAliases>
	<typeAlias type="com.fanling.demo.entity.SysUser" alias="SysUser"/>
</typeAliases>
```

多个别名映射[**推荐**]

```xml
<typeAliases>
	<package name="com.fanling.demo.entity"/>
</typeAliases>
```

## 动态SQL

除了使用的中的**select**、**insert**、**update**、**delete**外，还有一些常见的标签格式。

### 特殊标签

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
	close              结束的字符
-->
<select id="selectMulStudentByIds" parameterType="list">
	select * from student where 
	<foreach collection="list" item="item" open="id in(" close=")" separator=",">
		 #{item}
	</foreach>
</select>
```

> resultMap，映射实体类的别名

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
```

### 映射关系

实际使用中，表的关系很复杂。

#### 一对一

在`resultMap`中新增映射关系，一般是实体类中有其他实体属性的。

```xml
<association property="" javaType="">
    <!--
	property：T 的属性名
	javaType：T 的泛型类名
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

## 缓存



## 注意事项

1. MyBatis配置文件要按一定的顺序。

