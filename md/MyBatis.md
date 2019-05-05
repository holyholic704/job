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



JDBC编程有哪些不足之处，MyBatis是如何解决这些问题的？

① 数据库链接创建、释放频繁造成系统资源浪费从而影响系统性能，如果使用数据库链接池可解决此问题。

解决：在SqlMapConfig.xml中配置数据链接池，使用连接池管理数据库链接。

② Sql语句写在代码中造成代码不易维护，实际应用sql变化的可能较大，sql变动需要改变java代码。

解决：将Sql语句配置在XXXXmapper.xml文件中与java代码分离。

③ 向sql语句传参数麻烦，因为sql语句的where条件不一定，可能多也可能少，占位符需要和参数一一对应。

解决： Mybatis自动将java对象映射至sql语句。

④ 对结果集解析麻烦，sql变化导致解析代码变化，且解析前需要遍历，如果能将数据库记录封装成pojo对象解析比较方便。

解决：Mybatis自动将sql执行结果映射至java对象。



### 三个基本要素

- 核心接口和类
- MyBatis 核心配置文件
- SQL 映射文件



## 基础的 MyBatis 项目

### 搭建步骤

* 导入 JAR 包：mybatis、mysql-connector-java

* 编写核心配置文件：mybatis-config.xml

* 创建持久化类和 SQL 映射文件



### Mybatis 编程步骤

1. 创建 SqlSessionFactory
2. 通过 SqlSessionFactory 创建 SqlSession
3. 通过 SqlSession 执行数据库操作
4. 调用 session.commit() 提交事务
5. 调用 session.close() 关闭会话



### 基本目录结构

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



#### 映射文件

```xml
<!-- 映射文件一般会跟DAO接口放在同一个包下 -->
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.test.dao.TestDao">
    <!-- resultType，返回的类型 -->
    <select id="selectAll" resultType="test">
		select * from test;
    </select>
    
    <!-- 字段名与实体类中的属性名不一致，可以使用别名或resultMap -->
    select id,name username from test;
    
    <!-- paramerType可省略，MyBatis会自动检测 -->
	<insert id="insertOne" paramerType="test">
		insert into test values (id,name);
        <!-- 获取主键 -->
        <!-- order为SQL语句执行之前或之后 -->
        <selectKey resultType="int" keyProperty="id" order="AFTER">
			SELECT @@identity
		</selectKey>
	</insert>
    
    <!-- 模糊查询 -->
    <select id="selectByName" resultType="test">
		select * from test where name like '%' #{name} '%'
        <!-- 另一种写法 -->
        select * from test where name like '%${name}%'
	</select>
</mapper>
```



##### ${} 和 #{} 的区别

* #{} 其实是占位符，以 ? 进行占位，类似 JDBC 的 PreparedStatement，可以防止 SQL 注入的问题。#{} 中的内容是实体类的属性名，底层是通过 **反射机制**，调用实体类相关属性的 get 方法来获取值的。如果需要获取用户的输入进行动态拼接的话，就用 #{}
* ${} 是字符串拼接，参数会被直接拼接到 SQL 语句中，**会有 SQL 注入问题**，如果 SQL 语句由程序员直接写好，不需要用户输入的话，可以使用 ${}，当然更建议使用 #{}



##### resultType 与 resultMap

* resultType：设置返回的类型，MyBatis 后台会自动创建一个 resultMap，基于属性名来映射到实体类属性上
* resultMap：将数据库表中的字段与实体类中的属性 **建立映射关系**，这样即使两者名字不一致，MyBatis 也会根据 resultMap 中的映射关系正常执行。**涉及到两张表的操作，即使字段名和实体类属性名一致，也要编写 resultMap 来进行关联**
* **两者的区别**
  * resultType 对应的是 Java 对象中的属性，大小写不敏感
  * resultMap 对应的是对已经定义好了 id 的 resultType 的引用，**大小写敏感**

```xml
<!-- type属性用来指定要映射的实体类 -->
<resultMap id="testMapper" type="test">
    <!-- column表示数据库中的字段名，property表示实体类中的属性名 -->
    <!-- 在resultMap中添加id的属性指定主键，可以提高MyBatis的查询性能 -->
    <id column="id" property="id"/>
    <result column="name" property="username"/>
</resultMap>

<select id="selectTest" resultMap="testMapper">
    select id,name from test
</select>
```



##### 别名

![20190504184410](../md.assets/20190504184410.png)



#### Dao 实现类

```java
public class StudentDaoImpl implements StudentDao {
    @Override
    public List<Test> selectAll() {
        List<Test> test = null;
        try(SqlSession sqlSession = MyBatisUtil.getSqlSession()) {
            // 返回对象集合
            // 返回单个对象为selectOne
            stu = sqlSession.selectList("selectAll");
        }
        return test;
    }
    
    @Override
    public void insertStudent(Student student) {
        try(SqlSession sqlSession = MyBatisUtil.getSqlSession()) {
            sqlSession.insert("insertOne", test);
            // 提交事务
            sqlSession.commit();
        }
    }
}
```



#### 核心配置文件

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
            	<!-- JDBC：直接使用了JDBC的提交和回滚设置，默认需要手动提交 -->
            	<!-- MANAGED：由容器来管理事务，默认情况下会关闭连接 -->
            <transactionManager type="JDBC"/>
            <!-- 配置数据源和数据库连接基本属性，有三种内建的数据源类型 -->
            	<!-- UNPOOLED：不使用连接池，每次请求都会创建一个数据库连接，使用完毕后关闭 -->
            	<!-- POOLED：使用MyBatis自带的数据库连接池 -->
            	<!-- JNDI：配置外部数据源 -->
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
        <!-- 3.使用映射器接口的完全限定类名，需满足三个要求 -->
        	<!-- 映射文件名要与DAO接口名称相同 -->
        	<!-- 映射文件要与接口在同一包中 -->
        	<!-- 映射文件中的namespace属性值为DAO接口的全类名 -->
        <mapper class="com.test.dao.TestDao"/>
        <!-- 4.将包内的映射器接口实现全部注册为映射器，需满足四个要求 -->
        	<!-- DAO使用mapper动态代理实现 -->
        	<!-- 映射文件名要与DAO接口名称相同 -->
        	<!-- 映射文件要与接口在同一包中 -->
        	<!-- 映射文件中的namespace属性值为DAO接口的全类名 -->
        <package name="com.test.dao"/>
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
        // 自动提交事务：ssf.openSession(true);
        // 括号内为false或不写为不自动提交事务
        return ssf.openSession();
    }
	
    public static void close(SqlSession ss) {
        if (ss != null) {
            ss.close();
        }
    }
}
```



#### 测试类

```java
@Test
public void selectAll(){
	TestDao td = new TestDaoImpl();
	td.selectAll().forEach((t -> {
		System.out.println(t);
	}));
}
```



## MyBatis 核心对象与生命周期

* SqlSessionFactoyBuilder：**用过即丢**
  * 用来创建 SqlSessionFactory，创建完毕后，就不再需要它了。所以 SqlSessionFactoryBuilder 最佳作用域是 **方法作用域**，即局部方法变量，**用完即销毁**，生命周期就是调用方法的开始到结束。可以重用 SqlSessionFactoryBuilder 来创建多个 SqlSessionFactory 实例，但是最好还是不要让其一直存在，以保证所有的 XML 解析资源可以被释放给更重要的事情

* SqlSessionFactory：**Application**
  * SqlSessionFactory 一旦被创建就应该在应用的运行期间一直存在，没有任何理由丢弃它或重新创建另一个实例。使用 SqlSessionFactory 的最佳实践是 **在应用运行期间不要重复创建多次**，因此 SqlSessionFactory 的最佳作用域是 **应用作用域**。有很多方法可以做到，最简单的就是使用单例模式或者静态单例模式

* SqlSession：**Session**
  * 每个线程都应该有它自己的 SqlSession 实例。SqlSession 的实例 **不是线程安全** 的，因此是 **不能被共享** 的，所最佳的作用域是 **请求或方法作用域**。绝对不能将 SqlSession 实例的引用放在一个类的静态域，甚至一个类的实例变量也不行。也绝不能将 SqlSession 实例的引用放在任何类型的托管作用域中，如 Servlet 框架中的 HttpSession。如果现在正在使用一种 Web 框架，要考虑 SqlSession 放在一个和 HTTP 请求对象相似的作用域中。换句话说，每次收到的 HTTP 请求，就可以打开一个 SqlSession，返回一个响应，就关闭它，**应该把关闭操作放到 finally 块中** 以确保每次都能执行关闭
* 映射器实例：**Session**
  * 映射器是一些由你创建的、绑定你映射的语句的接口。映射器接口的实例是从 SqlSession 中获得的，因此，映射器实例的最大作用域是和请求它们的 SqlSession 相同的，最佳作用域是 **方法作用域**。映射器实例应该在调用它们的方法中被请求，用过即丢。不需要显式地关闭映射器实例，尽管在整个请求作用域保持映射器实例也不会有什么问题，但就像 SqlSession 一样，在这个作用域上管理太多的资源的话会难于控制，最好把映射器放在方法作用域内



## MyBatis 主要部件

- Configuration：MyBatis 所有的配置信息都保存在 Configuration 对象之中，配置文件中的大部分配置都会存储到该类中
- SqlSession：MyBatis 工作的主要顶层 API，表示和数据库交互时的会话，完成必要数据库增删改查功能
- Executor：MyBatis 执行器，是 MyBatis **调度的核心**，负责 SQL 语句的生成和查询缓存的维护
- StatementHandler：封装了 JDBC Statement操作，负责对 JDBC statement 的操作
- ParameterHandler：负责对用户传递的参数转换成 JDBC Statement 所对应的数据类型
- ResultSetHandler：负责将 JDBC 返回的 ResultSet 结果集对象转换成 List 类型的集合
- TypeHandler：负责 Java 数据类型和 JDBC 数据类型之间的映射和转换
- MappedStatement：维护一条 <select|update|delete|insert> 节点的封装
- SqlSource：根据用户传递的 parameterObject，动态地生成 SQL 语句，将信息封装到 BoundSql 对象中，并返回
- BoundSql：表示动态生成的 SQL 语句以及相应的参数信息



## mapper 动态代理

MyBatis 无需编写 DAO 实现类，直接通过 DAO 接口来定位到 mapper 中的 SQL 语句



### 如何使用 mapper 动态代理

映射文件 mapper 标签 **添加 namespace** 属性，将当前映射文件与 DAO 接口关联起来。映射文件中的 **id 名要与 DAO 接口中的方法名一致**，将方法和 SQL 语句关联起来

```xml
<mapper namespace="com.test.dao.TestDao">
```

**测试类**

```java
private SqlSession ss;
private TestDao td;

@Test
public void selectAll(){
    ss = MyBatisUtil.getSqlSession();
    // 获得TestDao对象
    td = ss.getMapper(TestDao.class);
    
    td.selectAll().forEach((t -> {
		System.out.println(t);
	}));
    
    MyBatisUtil.close();
}
```

将 DAO 的实现类删除之后，MyBatis 底层 **只会调用 selectOne() 或 selectList() 方法**。框架选择方法的标准是测试类中用于接收返回值的对象类型。**若接收类型为 List，选择 selectList() 方法；否则选择 selectOne() 方法**



## 动态 SQL

执行查询操作的时候可能会有多个条件，但用户在输入的时候，填写的条件数不确定，可以使用动态 SQL 来解决这个问题，**动态 SQL 会根据传入的条件动态拼接 SQL 语句**



### if 标签

```xml
<select id="selectIf" resultType="test">
	select * from test
    <!-- 添加1=1为true的条件，当两个条件均未设定只剩下一个where，这时SQL语句就不正确了 -->
    where 1=1
    <if test="name!=null and name!=''">
		and name '%' #{name} '%'
    </if>
    <if test="age>=0">
		and age > #{age}
    </if>
</select>
```



### where 标签

```xml
<select id="selectWhere" resultType="test">
    select * from test
    <!-- 使用where标签就无需再写1=1了，第一个if标签可以不加and -->
    <where>
        <if test="name!=null and name!=''">
            name '%' #{name} '%'
        </if>
        <if test="age>=0">
            and age > #{age}
        </if>
    </where>
</select>
```



### choose 标签

```xml
<select id="selectWhere" resultType="test">
    select * from test
    <!-- 不需要再写and -->
    <where>
        <choose>
        	<when test="name!=null and name!=''">
            	name '%' #{name} '%'
        	</when>
        	<when test="age>=0">
            	age > #{age}
        	</when>
            <otherwise>
            	1!=1
            </otherwise>
		</choose>
    </where>
</select>
```



### foreach 标签

```xml
<!-- 遍历数组或集合，相当于SQL中的in语句 -->
<select id="selectForEach" resultType="test">
    select * from test
    <!-- 遍历数组使用array，遍历集合使用list-->
    <if test="array!=null and array.length>0">
		where id in
        <!-- collection表示要遍历的类型 -->
        <!-- open、close、separator表示对遍历内容的SQL拼接 -->
        <!-- 可以遍历自定义数据类型的集合 -->
        <foreach collection="array" open="(" close=")" item="id" separator=",">
            #{id}
        </foreach>
    </if>
</select>
```



### sql 标签

```xml
<!-- 定义一个可被复用的sql片段，在使用时写上include标签将sql标签中的内容引入 -->
<sql id="select">
    select * from test
</sql>

<select id="selectSQL" resultType="test">
    <!--使用sql片段-->
    <include refid="select"/>

    <if test="array!=null and array.length>0">
		where id in
        <foreach collection="array" open="(" close=")" item="id" separator=",">
            #{id}
        </foreach>
    </if>
</select>
```

在 MyBatis 的动态 SQL 中，有时会对一些数据进行比较，可能会导致 xml 文件解析出现问题，可以使用实体符号代替，还可以将这些数据放到 **`<![CDATA[ ]]>`** 里面，这里面的内容 xml 是不会解析的

|   元符号   |      实体符号      |
| :--------: | :----------------: |
| `＜`、`<=` |  `&lt;`、`&lt;=`   |
| `＞`、`>=` |   `&gt`、`&gt;=`   |
|    `&`     |      `&amp;`       |
|  `"`、`'`  | `&quot;`、`&apos;` |



## 关联查询

### 一对多查询

在查询一方对象的时候同时把跟它所关联的多方对象也查询出来

```java
public class Player {
    private int id;
    private String name;
}
// 一个Team关联着多个Player
public class Team {
    private int id;
    private String name;
    private List<Player> playerList;
}
```

```xml
<mapper namespace="com.test.dao.TestDao">
    <!-- 当到两张表的操作，即使字段名和实体类属性名一致也要编写resultMap进行关联 -->
	<resultMap type="team" id="tt">
		<id column="tid" property="id" />
		<result column="tname" property="name" />
        <!-- collection中填写的内容关联的是Player中的属性 -->
        <!-- property指定关联属性，即实体类的集合的属性，ofType指定集合属性的泛型类型 -->
		<collection property="playerList" ofType="player">
			<id column="pid" property="id" />
			<result column="pname" property="name" />
		</collection>
	</resultMap>

	<select id="selectTeamById" resultMap="tt">
		select t.id tid,t.name tname,
        p.id pid,p.name pname
		from team t,player p
		where t.id = p.tid
		and p.tid = #{id}
	</select>
</mapper>
```



### 多对一查询

在查询多方对象的时候，同时将其所关联的一方对象也查询出来。由于在查询多方对象时也是一个一个查询，所以多对一关联查询，**其实就是一对一关联查询**，即一对一关联查询的实现方式与多对一的实现方式是相同的

```java
public class Player {
    private int id;
    private String name;
    private Team team;
}

public class Team {
    private int id;
    private String name;
}
```

```xml
<mapper namespace="com.test.dao.TestDao">
    <resultMap type="player" id="pp">
		<id column="pid" property="id" />
		<result column="pname" property="name" />
		<!-- association标签体现出两个实体类对象之间的关系 -->
        <!-- property指定关联属性，即实体类的属性，javaType指定关联属性的类型 -->
		<association property="team" javaType="team">
			<id column="tid" property="id" />
			<result column="tname" property="name" />
		</association>
	</resultMap>
	<select id="selectPlayerById" resultMap="pp">
		select t.id tid,t.name tname,
        p.id pid,p.name pname
		from team t,player p
		where t.id = p.tid
		and p.id = #{id}
	</select>
</mapper>
```



#### 注意

若定义的类是双向关联，即双方的属性中均有对方对象作为域属性出现，在定义各自的 toString() 方法时，只让某一方可以输出另一方，**不要让双方的 toString() 方法均可输出对方**，这样会造成栈内存溢出的错误



*更多：[MyBatis 关联查询](https://blog.csdn.net/abc5232033/article/details/79054247)*



## 自关联查询

自己同时充当多方和一方，即多和一都在同一张表中，一般这样的表其实可以看做是一个树形结构，在数据库表中有一个外键，该外键表示当前数据的父节点



### 一对多关联查询

```java
public class Employee {
    private int id;
    private String name;
    private String job;
    // 表示多的一方，即当前员工的所有下属
    private List<Employee> children;
}
```

```xml
<mapper namespace="com.test.dao.TestDao">
	<resultMap type="employee" id="em">
		<id column="id" property="id" />
        <!-- 形成递归查询 -->
        <!-- select属性表示会继续执行selectChildrenByPid这个sql语句 -->
        <!-- column表示将id作为属性传入SQL中，此处是pid -->
        <!-- column中的id要跟SQL语句中的id一致，查询出的id会作为条件pid再次传入SQL中执行 -->
		<collection property="children" ofType="employee" select="selectChildrenById"
			column="id">
		</collection>
	</resultMap>

	<select id="selectChildrenById" resultMap="em">
		select id,name,job
		from employee
		where mgr = #{mgr}
	</select>
</mapper>
```



### 多对一关联查询

```java
public class Employee {
    private int id;
    private String name;
    private String job;
    // 表示一的一方，即当前员工的上级领导对象
    private Employee leader;
}
```

```xml
<mapper namespace="com.test.dao.TestDao">
	<resultMap type="employee" id="em">
		<id column="id" property="id" />
        
		<association property="leader" javaType="employee" select="selectLeaderById"
			column="mgr">
        </association>
	</resultMap>

	<select id="selectLeaderById" resultMap="em">
		select id,name,job,mgr
		from employee
		where id = #{id}
	</select>
</mapper>
```



### 多对多关联查询

由两个互反的一对多关系组成，多对多关系都会通过一个中间表来建立

```java
public class Student {
    private int id;
	private String name;
	private int age;
	private double score;
	private List<Course> courses;
}

public class Course {
    private int id;
    private String name;
    private List<Student> students;
}
```

```xml
<mapper namespace="com.test.dao.TestDao">
	<resultMap type="course" id="cc">
		<id column="cid" property="id" />
		<result column="cname" property="name" />
		<collection property="students" ofType="student">
			<id column="sid" property="id" />
			<result column="sname" property="name" />
		</collection>
	</resultMap>

	<select id="selectCourseStudent" resultMap="cc">
		select c.id cid,c.name cname,s.id sid,s.name sname
		from course c,student s,student_course sc
		where c.id = #{id} and s.id = sc.sid and c.id = sc.cid;
	</select>
</mapper>
```



## 延迟加载

也称为懒加载，**在进行表的关联查询时，按照设置延迟对关联对象的 select 查询。**如在进行一对多查询的时候，只查询出一方，当程序中需要多方的数据时，MyBatis 再发出 SQL 语句进行查询，这样 **延迟加载就可以的减少数据库压力**



### 基本原理

使用 CGLIB 创建目标对象的代理对象，当调用目标方法时，进入拦截器方法，如调用 a.getB().getName()，拦截器 invoke() 方法发现 a.getB() 是null值，那么就会单独发送事先保存好的查询关联 B 对象的 SQL，把 B 查询上来，然后调用 a.setB(b)，于是 A 的对象 b 属性就有值了，接着完成 a.getB().getName() 方法的调用



### 关联对象加载时机

MyBatis 根据对 **关联对象查询的 select 语句的执行时机**，分为三种类型。

* 直接加载
  * 执行完对主加载对象的 selec t语句，马上执行对关联对象的 select 查询

* 侵入式延迟
  * 执行对主加载对象的查询时，不会执行对关联对象的查询。但当要访问主加载对象的详情属性时，就会马上执行关联对象的 select 查询

* 深度延迟
  * 执行对主加载对象的查询时，不会执行对关联对象的查询。访问主加载对象的详情时也不会执行关联对象的 select 查询。只有当真正访问关联对象的详情时，才会执行对关联对象的select查询



### 延迟加载的应用要求

关联对象的查询与主加载对象的查询 **必须是分别进行的 select 语句，不能是使用多表连接所进行的 select 查询**。多表连接查询，其实就是对一张表的查询，对由多个表连接后形成的一张表的查询。会一次性将多张表的所有信息查询出来。延迟加载 **只是对关联对象的查询有迟延设置**，对于 **主加载对象都是直接执行查询语句** 的，只对于 resultMap 中的 collection 和 association 起作用

#### 单独查询

```xml
<mapper namespace="com.test.dao.TestDao">
	<resultMap type="team" id="tt">
		<id column="id" property="id" />
		<collection property="playerList" ofType="player" select="selectPlayerById"
			column="id" />
	</resultMap>

	<select id="selectTeamByIdAlone" resultMap="tt">
		select id,name
		from team
		where id = #{id}
	</select>

	<select id="selectPlayerById" resultType="player">
		select id,name
		from player
		where tid = #{tid}
	</select>
</mapper>
```



#### 开启延迟加载

```xml
<!-- 全局参数设置，该标签需要放在properties与typeAliases之间 -->
<settings>
	<!-- 延迟加载的总开关，true表示开启，false为关闭 -->
	<setting name="lazyLoadingEnabled" value="true"/>
	<!-- 侵入式延迟加载，true表示开启，默认为false -->
    <!-- value为false时表示开启深度延迟加载 -->
	<setting name="aggressiveLazyLoading" value="true"/>
</settings>
```

若只希望某些查询支持深度延迟加载的话，可以在 resultMap 中的 collection 或 association 添加 **fetchType 属性**，配置为 **lazy 为开启深度延迟**，配置 **eager 为不开启深度延迟**。fetchType 属性将取代全局配置参数 lazyLoadingEnabled 的设置



## 缓存

查询缓存主要是为了提高查询访问速度，当用户执行一次查询后，会将数据结果放到缓存中，下次再执行此查询时直接从缓存中获取数据。如果在缓存中找到了数据那叫做 **命中**



### 作用域

查询缓存的作用域根据映射文件 mapper 的 namespace 划分，相同 namespace 的 mapper 查询数据存放在同一个缓存区域。不同 namespace 下的数据互不干扰。一级缓存和二级缓存，都是按  namespace 进行分别存放的



### 一级缓存

也叫作本地缓存，是基于 org.apache.ibatis.cache.impl.PerpetualCache 类的 **HashMap 本地缓存**，**其作用域是 SqlSession**，即同一个 SqlSession 中两次执行相同的 SQL 查询语句，第一次执行完毕后，会将查询结果写入到缓存中，第二次会从缓存中直接获取数据，而不再到数据库中进行查询，减少了数据库的访问，提高查询效率。**一级查询缓存是默认开启状态，且不能关闭**

因为 **mapper 中的 id 具有唯一性**，所以 MyBatis 是通过这个 id 来判断缓存中是否存在的。如果有两个 SQL 语句一模一样，但是两者的 id 不一样，此时 MyBatis 是不会为这两个 SQL 语句建立相同缓存的。如果一条 select 语句中有查询条件的话，该查询条件也会被作为特征值，即再有相同条件查询的时候，会命中



#### 增删改对一级缓存的影响

在第一次查询数据库后，MyBatis会建立缓存，下一次访问该数据的时候会直接从缓存中获取，如建立缓存后，下次访问前，对数据进行了增删改的操作，此时 **无论是否 commit，都会清空一级缓存**



### **二级缓存**

MyBatis 内置的二级缓存为 org.apache.ibatis.cache.impl.PerpetualCache。与一级缓存不同的是 **二级缓存的生命周期与整个应用同步**，与 SqlSession 是否关闭没有关系

使用二级缓存要先序列化实体类，让实体类实现 Serializable 接口，如果该实体类有父类的话，父类也要实现 Serializable 接口。之后，在映射文件中的 mapper 标签下添加 **`<cache/>`** 标签



#### cache 标签

```xml
<!-- 逐出策略，当二级缓存中的对象达到最大值时，通过逐出策略将缓存中的对象移出缓存 -->
<!-- 默认为LRU，FIFO：先进先出；LRU：未被使用时间最长的 -->
<cache eviction="LRU"/>

<!-- 刷新缓存的时间间隔，即清空缓存，单位毫秒 -->
<!-- 一般不指定，即当执行增删改时刷新缓存，长时间未刷新缓存可能会出现过期数据 -->
<cache flushInterval="100000"/>

<!-- 设置缓存中数据是否只读，只读的缓存会给所有调用者返回缓存对象的相同实例 -->
<!-- 性能会好一些，但是不能被修改 -->
<!-- 默认是false，读写的缓存会通过序列化返回缓存对象的拷贝，因为要拷贝对象，会慢一些，但是安全 -->
<cache readOnly="false"/>

<!-- 二级缓存中可以存放的最多对象个数，默认为1024个 -->
<cache size="1024"/>
```



#### 增删改对二级缓存的影响

增删改操作，无论是否进行提交 commit()，均会清空一级、二级查询缓存，使查询再次从数据库中 select。二级缓存中的 key 是不会清空，只清空 key 对应的值。如果想要设置增删改操作的时候不清空二级缓存的话，可以在 insert、delete、update 标签中添加属性 **`flushCache="false"`**，默认为 true



#### 关闭二级缓存

```xml
<!-- 全局关闭，所有查询均不使用二级缓存 -->
<!-- 默认为true，即二级缓存默认是开启的，false为关闭 -->
<setting name="cacheEnabled" value="false"/>
```

局部关闭可以只关闭某个 select 查询的二级缓存，在 select 标签中 **将 useCache 属性设置为 false**









**二级缓存的使用注意事项**

**在一个命名空间下使用二级缓存**

二级缓存对于不同的命名空间namespace的数据是互不干扰的，倘若多个namespace中对一个表进行操作的话，就会导致这不同的namespace中的数据不一致的情况。

**在单表上使用二级缓存**

在做关联关系查询时，就会发生多表的操作，此时有可能这些表存在于多个namespace中，这就会出现上一条内容出现的问题了。

**查询多于修改时使用二级缓存**

在查询操作远远多于增删改操作的情况下可以使用二级缓存。因为任何增删改操作都将刷新二级缓存，对二级缓存的频繁刷新将降低系统性能。

 

**Mybatis****外置二级缓存**

Mybatis除了自带的二级缓存，还支持一些第三方的缓存，并且由于Mybatis只擅长sql，所以这些第三方缓存的性能要比Mybatis的好一些， **ehCache是一款知名的缓存框架，Hibernate框架的默认缓存策略使用的就是ehCache。使用ehCache二级缓存，实体类无需实现Serializable接口。**



























































## 更多

* [MyBatis官方中文文档](http://www.MyBatis.org/MyBatis-3/zh/index.html)
* 