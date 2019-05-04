## MyBatis 概述

### MyBatis 是什么

MyBatis 是一个基于 Java 的 **持久层框架**，它支持定制化 SQL、存储过程以及高级映射。主要作用就是在 Java 中操作数据库，其实就是在 JDBC 的基础上进行了封装，开发者只需要关注 SQL 语句本身，避免了几乎所有的 JDBC 代码和手动设置参数以及获取结果集



### Hibernate 是什么

Hibernate 是一个开源的全自动的 **对象关系映射框架**，它对 JDBC 进行了封装，它将 POJO 与数据库表建立映射关系，Hibernate 可以自动生成SQL语句，自动执行

 

#### 什么是对象关系映射

对象关系映射（Object - Relationship - Mapping，ORM），是一种思想，他的实质就是 **将关系数据库中的业务数据用对象的形式表示出来**，并通过面向对象的方式将这些对象组织出来，实现系统的业务逻辑。**说到底就是 Java 实体对象跟数据库数据的映射关系**



### MyBatis 和 Hibernate 的区别

* MyBatis 是半自动，Hibernate 是全自动
  * MyBatis 只有基本的字段映射，需要通过手写sql来实现和管理，Hibernate 可以自动生成 SQL

* Hibernate 数据库移植性大于 MyBatis
  * Hibernate 通过它强大的映射结构和 HQL 语言，降低了对象与数据库的耦合性，而 MyBatis 由于需要手写 SQL，移植性也会随之降低很多

* Hibernate 拥有完整的日志系统，MyBatis 则欠缺一些
  * Hibernate 日志系统非常健全，涉及广泛，而 MyBatis 则除了基本记录功能外，功能薄弱很多

* Hibernate 配置要比 MyBatis 复杂的多，学习成本也比 MyBatis 高，但是如果使用 Hibernate 很熟练的话，实际上开发效率甚至超越 MyBatis

* MyBatis 使用简单，需要比 Hibernate 关心很多技术细节

* SQL 直接优化上，MyBatis 比 Hibernate 更方便
  * 由于 MyBatis 的sql都是写在xml里，因此优化sql比Hibernate方便很多。而Hibernate的sql很多都是自动生成的，无法直接维护sql；总之写sql的灵活度上Hibernate不及MyBatis。



### 三个基本要素

- 核心接口和类
- MyBatis 核心配置文件
- SQL 映射文件



## MyBatis 使用

### 搭建步骤

* 导入 JAR 包：mybatis、mysql-connector-java

* 编写核心配置文件：mybatis-config.xml

* 创建持久化类和 SQL 映射文件



Mybatis 编程步骤

1. 创建 SqlSessionFactory
2. 通过 SqlSessionFactory 创建 SqlSession
3. 通过 SqlSession 执行数据库操作
4. 调用 session.commit() 提交事务
5. 调用 session.close() 关闭会话



### 基础的 MyBatis 项目

#### 基本目录结构

* src/main/java
  * 实体类
  * DAO 接口
  * DAO 实现类
  * 工具类
* src/main/resources
  * 数据库配置文件
  * 核心配置文件
* src/test/java
  * 测试类



#### 映射文件（TestMapper.xml）

```xml
<!-- 映射文件一般会跟DAO接口放在同一个包下 -->
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.test.dao.TestDao">
    <!-- resultType，返回的类型 -->
    <select id="selectAll" resultType="test">
		select * from test
    </select>
    
    <!-- paramerType可省略，MyBatis会自动检测 -->
	<select id="insertOne" paramerType="test">
		insert into test values (id,name)
	</select>
</mapper>
```



#### Dao 实现类

```java
public class StudentDaoImpl implements StudentDao {
    @Override
    public List<Test> selectAll() {
        List<Test> test = null;
        try(SqlSession sqlSession = MyBatisUtil.getSqlSession()) {
            stu = sqlSession.selectList("selectAll");
        }
        return test;
    }
}
```



#### 主配置文件（mybatis-config.xml）

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <!-- 获取数据库配置文件 -->
    <properties resource="db.properties"/>
    
    <!-- 设置实体类的别名，无需再写包名，减少冗余，有两种写法 -->
    <typeAliases>
        <!-- 1.默认别名为实体类的首字母小写，可以使用alias手动设置别名 -->
        <typeAlias type="com.test.bean.Test" alias="test"/>
        <!-- 2.指定一个包，MyBatis会自动搜索该包下需要的实体类 -->
        <!-- 默认别名为实体类的首字母小写，可以在实体类上使用@Alias注解手动设置别名 -->
        <package name="com.test.bean"/>
    </typeAliases>
    
    <environments default="development">
        <environment id="development">
            <!-- 指定MyBatis使用的事务管理器，MyBatis支持两种事务管理器类型 -->
            <!-- JDBC：通过commit()方法提交，rollback()方法回滚，默认需要手动提交 -->
            <!-- MANAGED：由容器来管理事务，默认情况下会关闭连接，使用Spring框架之后，无需再配置				     事务管理器，Spring会使用自带的管理器 -->
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="${driver}"/>
                <property name="url" value="${url}"/>
                <property name="username" value="${user}"/>
                <property name="password" value="${password}"/>
            </dataSource>
        </environment>
    </environments>
    <!-- mapper映射器，告诉配置文件mapper所在的路径，有四种写法 -->
    <mappers>
        <!-- 1.使用相对于类路径的资源引用 -->
        <mapper resource="com/test/dao/TestMapper.xml"/>
        <!-- 2.使用完全限定资源定位符 -->
        <mapper url="D:\MyBatis\src\main\java\com\test\dao\TestMapper.xml"/>
        <!-- 3.使用完全限定资源定位符 -->
    </mappers>
</configuration>
```



#### 数据库配置文件（db.propertie）

```properties
driver = com.mysql.jdbc.Driver
url = jdbc:mysql://127.0.0.1:3306/test
user = root
password = root
```



#### 工具类

```java
public class MyBatisUtil {
    private static SqlSessionFactory ssf;

    // 只创建一个SqlSessionFactory对象
    static {
        // 读取配置文件
        try (InputStream is = Resources.getResourceAsStream("mybatis-config.xml")) {
            ssf = new SqlSessionFactoryBuilder().build(is);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    public static SqlSession getSqlSession() {
        return ssf.openSession();
    }
	
    // 该方法可省略，SqlSession的父类实现了AutoCloseable接口，执行结束会自动执行close方法
    public static void close(SqlSession ss) {
        if (ss != null) {
            ss.close();
        }
    }
}
```



#### 测试类

```java
public class Test {
    
    @Test
    public void selectAll(){
        TestDao td = new TestDaoImpl();
        td.selectAll().forEach((t -> {
            System.out.println(t);
        }));
    }
}
```



#### Maven 项目

Maven 项目需要在 pom.xml 文件中的 build 标签中添加以下内容，因为要在 dao 包下编写 xml 文件，**如果不添加下面内容的话，Maven 是不会将 xml 文件发布到编译后的 classes 目录下**，就会导致 MyBatis 到不到该文件

```xml
<resources>
	<resource>
		<directory>src/main/java</directory>
		<includes>
			<include>**/*.xml</include>
		</includes>
	</resource>
</resources>
```





































## 更多

* [MyBatis官方中文文档](http://www.MyBatis.org/MyBatis-3/zh/index.html)
* 