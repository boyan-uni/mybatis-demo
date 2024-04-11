
# Mybatis-从入门到速成（实战篇）

操作参考文档：[Mybatis教程-实战看这一篇就够了 ](https://www.cnblogs.com/diffx/p/10611082.html) ，具体执行和代码执行有改动。

##  一. 新建项目JDBC测试

### 1）在IDEA创建新项目

1. 使用IDEA创建maven工程项目，选择对应的Java JDK版本，这里我选择OpenJDK17；
2. 命名项目为：mybatis-demo

### 2）在maven-pom.xml配置文件中引入mysql依赖

```xml
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>5.1.32</version>
</dependency>
```

### 3）连接mysql数据库，准备数据

- 连接数据库时注意填写正确的用户名和密码；
- 以下sql是调整后的，与原教程的有差异：

```sql
DROP TABLE IF EXISTS tb_user;
CREATE TABLE tb_user (
                         id char(32) NOT NULL,
                         user_name varchar(32) DEFAULT NULL,
                         password varchar(32) DEFAULT NULL,
                         name varchar(32) DEFAULT NULL,
                         age int(10) DEFAULT NULL,
                         sex int(2) DEFAULT NULL,
                         birthday date DEFAULT NULL,
                         created datetime DEFAULT NULL,
                         updated datetime DEFAULT NULL,
                         PRIMARY KEY (id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

INSERT INTO ssmdemo.tb_user ( id, user_name, password, name, age, sex, birthday, created, updated) VALUES ( '1', 'zpc', '123456', '鹏程', '22', '1', '1990-09-02', sysdate(), sysdate());
INSERT INTO ssmdemo.tb_user ( id, user_name, password, name, age, sex, birthday, created, updated) VALUES ( '2', 'hj', '123456', '静静', '22', '1', '1993-09-05', sysdate(), sysdate());
```

### 4) JDBC测试

- JDBCTest.java

```java
package com.boyan;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.ResultSet;

/**
 * @author boyan
 */
public class JDBCTest {
    public static void main(String[] args) throws Exception {
        Connection connection = null;
        PreparedStatement prepareStatement = null;
        ResultSet rs = null;

        try {
            // 加载驱动
            Class.forName("com.mysql.jdbc.Driver");
            // 获取连接
            String url = "jdbc:mysql://localhost:3306/ssmdemo";
            String user = "root";
            String password = "123456";
            connection = DriverManager.getConnection(url, user, password);
            // 获取statement，preparedStatement
            String sql = "select * from tb_user where id=?";
            prepareStatement = connection.prepareStatement(sql);
            // 设置参数
            prepareStatement.setLong(1, 1l);
            // 执行查询
            rs = prepareStatement.executeQuery();
            // 处理结果集
            while (rs.next()) {
                System.out.println(rs.getString("user_name"));
                System.out.println(rs.getString("name"));
                System.out.println(rs.getInt("age"));
                System.out.println(rs.getDate("birthday"));
            }
        } finally {
            // 关闭连接，释放资源
            if (rs != null) {
                rs.close();
            }
            if (prepareStatement != null) {
                prepareStatement.close();
            }
            if (connection != null) {
                connection.close();
            }
        }
    }
}

```

- 打印测试结果（成功，符合预期）

<img src="https://cdn.jsdelivr.net/gh/boyan-uni/pic-bed/img/mybatis-JDBCTest-Result.png" style="zoom:50%;" />

------

## 二. Quick Start

### 0）目录结构

<img src="https://cdn.jsdelivr.net/gh/boyan-uni/pic-bed/img/mybatis-MybatisTest-catalog-update.png" alt="image-20240405233333797" style="zoom:33%;" />

- 新建：
  - src（源文件）：MybatisTest.java，User.java（POJO类）
  - resources（资源配置文件）：mappers/MyMapper.xml，log4j.properties，mybatis-config.xml

### 1）maven：pom.xml 引入mybatis依赖

```xml
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>3.2.8</version>
</dependency>
```

### 2）mybatis全局配置文件：mybatis-config.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<!-- 根标签 -->
<configuration>
    <properties>
        <property name="driver" value="com.mysql.jdbc.Driver"/>
        <property name="url" value="jdbc:mysql://localhost:3306/ssmdemo?useUnicode=true&amp;characterEncoding=utf-8&amp;allowMultiQueries=true"/>
        <property name="username" value="root"/>
        <property name="password" value="123456"/>
    </properties>

    <!-- 环境，可以配置多个，default：指定采用哪个环境 -->
    <environments default="test">
        <!-- id：唯一标识 -->
        <environment id="test">
            <!-- 事务管理器，JDBC类型的事务管理器 -->
            <transactionManager type="JDBC" />
            <!-- 数据源，池类型的数据源 -->
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver" />
                <property name="url" value="jdbc:mysql://localhost:3306/ssmdemo" />
                <property name="username" value="root" />
                <property name="password" value="123456" />
            </dataSource>
        </environment>
        <environment id="development">
            <!-- 事务管理器，JDBC类型的事务管理器 -->
            <transactionManager type="JDBC" />
            <!-- 数据源，池类型的数据源 -->
            <dataSource type="POOLED">
                <property name="driver" value="${driver}" /> <!-- 配置了properties，所以可以直接引用 -->
                <property name="url" value="${url}" />
                <property name="username" value="${username}" />
                <property name="password" value="${password}" />
            </dataSource>
        </environment>
    </environments>

    <!-- 在全局配置文件中，添加 MyMapper.xml 文件配置-->
    <mappers>
        <mapper resource="mappers/MyMapper.xml"/>
    </mappers>
</configuration>
```

- 包含：mapper配置⬆️

### 3）mybatis配置Mapper：MyMapper.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<!-- mapper:根标签，namespace：命名空间，随便写，一般保证命名空间唯一 -->
<mapper namespace="MyMapper">
    <!-- statement，内容：sql语句。id：唯一标识，随便写，在同一个命名空间下保持唯一
       resultType：sql语句查询结果集的封装类型,tb_user即为数据库中的表
     -->
    <select id="selectUser" resultType="com.boyan.mybatis.User">
        select * from tb_user where id = #{id}
    </select>
</mapper>
```

- <font color=red>发现问题：“解决数据库字段名和实体类属性名不一致的问题”</font>
  - 这里先通过“在sql语句中使用别名”的方法解决这个问题：

```xml
		<select id="selectUser" resultType="com.boyan.mybatis.User">
        select
            tuser.id as id,
            tuser.user_name as userName,
            tuser.password as password,
            tuser.name as name,
            tuser.age as age,
            tuser.birthday as birthday,
            tuser.sex as sex,
            tuser.created as created,
            tuser.updated as updated
        from
            tb_user tuser
        where tuser.id = #{id};
    </select>
```

- 修改过后，运行Mybatis.java，得到正确的测试结果（图中是已添加log4j后的输出）：

![image-20240406155000881](https://cdn.jsdelivr.net/gh/boyan-uni/pic-bed/img/mybatis-MybatisTest-result.png)

### 4）MybatisTest.java

```java
package com.boyan.mybatis;

import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;

import java.io.InputStream;

public class MybatisTest {
    public static void main(String[] args) throws Exception {
        // 指定全局配置文件
        String resource = "mybatis-config.xml";
        // 读取配置文件
        InputStream inputStream = Resources.getResourceAsStream(resource);
        // 构建sqlSessionFactory
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        // 获取sqlSession
        SqlSession sqlSession = sqlSessionFactory.openSession();
        try {
            // 操作CRUD，第一个参数：指定statement，规则：命名空间+“.”+statementId
            // 第二个参数：指定传入sql的参数：这里是用户id(类型：String)
            User user = sqlSession.selectOne("MyMapper.selectUser", 1);
            System.out.println(user);
        } finally {
            sqlSession.close();
        }
    }
}
```

- 本文件的测试目的：确认Mybatis和数据库的连通性（✅测试通过）；

### 5）POJO类：User.java

- 通过数据库表自动生成的基础上，对需要改动的部分进行手动改动
  - age，sex的类型：long->Integer；
  - birthday的类型：java.util.Date;
  - 处理birthday(Date) -"java.text.SimpleDateFormat"-> String，在toString()函数里；

```java
package com.boyan.mybatis.pojo;

import java.text.SimpleDateFormat;
import java.util.Date;

public class User {

  private String id;
  private String userName;
  private String password;
  private String name;
  private Integer age;
  private Integer sex;
  private Date birthday;
  private String created;
  private String updated;

  public String getId() {
    return id;
  }

  public void setId(String id) {
    this.id = id;
  }

  public String getUserName() {
    return userName;
  }

  public void setUserName(String userName) {
    this.userName = userName;
  }

  public String getPassword() {
    return password;
  }

  public void setPassword(String password) {
    this.password = password;
  }

  public String getName() {
    return name;
  }

  public void setName(String name) {
    this.name = name;
  }

  public Integer getAge() {
    return age;
  }

  public void setAge(Integer age) {
    this.age = age;
  }

  public Integer getSex() {
    return sex;
  }

  public void setSex(Integer sex) {
    this.sex = sex;
  }

  public Date getBirthday() {
    return birthday;
  }

  public void setBirthday(Date birthday) {
    this.birthday = birthday;
  }

  public String getCreated() {
    return created;
  }

  public void setCreated(String created) {
    this.created = created;
  }

  public String getUpdated() {
    return updated;
  }

  public void setUpdated(String updated) {
    this.updated = updated;
  }

  @Override
  public String toString() {
    return "User{" +
            "id='" + id + '\'' +
            ", userName='" + userName + '\'' +
            ", password='" + password + '\'' +
            ", name='" + name + '\'' +
            ", age=" + age +
            ", sex=" + sex +
            ", birthday='" + new SimpleDateFormat("yyyy-MM-dd").format(birthday) + '\'' +
            ", created='" + created + '\'' +
            ", updated='" + updated + '\'' +
            '}';
  }
}
```

### 6）maven：pom.xml 引入日志依赖（log4j以及slf4j-api）

```xml
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-log4j12</artifactId>
    <version>1.7.5</version>
</dependency>
```

- 新建：resource/log4j.properties 文件

```properties
log4j.rootLogger=DEBUG,A1
log4j.logger.org.apache=DEBUG
log4j.appender.A1=org.apache.log4j.ConsoleAppender
log4j.appender.A1.layout=org.apache.log4j.PatternLayout
log4j.appender.A1.layout.ConversionPattern=%-d{yyyy-MM-dd HH:mm:ss,SSS} [%t] [%c]-[%p] %m%n
```

- 在运行程序时就会打印日志；

### 7）已调整目录结构（0中图是更新后的catalog截图）

------

## 三. 添加完整的CRUD操作（User.java）

### 0）目录结构

<img src="https://cdn.jsdelivr.net/gh/boyan-uni/pic-bed/img/mybatis-UserCRUD-catalog.png" alt="image-20240406173222256" style="zoom:33%;" />

### 1）创建interface：UserDao.java

- 先定义接口，再根据接口方法，编写实现类；

```java
package com.boyan.mybatis.dao;

import com.boyan.mybatis.pojo.User;

import java.util.List;

/**
 * 基于POJO类：User.java 的 CRUD 操作
 */
public interface UserDao {
    /**
     * 根据id查询用户信息
     *
     * @param id
     * @return
     */
    public User queryUserById(String id);

    /**
     * 查询所有用户信息
     *
     * @return
     */
    public List<User> queryUserAll();

    /**
     * 新增用户
     *
     * @param user
     */
    public void insertUser(User user);

    /**
     * 更新用户信息
     *
     * @param user
     */
    public void updateUser(User user);

    /**
     * 根据id删除用户信息
     *
     * @param id
     */
    public void deleteUser(String id);
}
```

### 2）创建实现类：UserDaoImpl.java

- SqlSession 封装 jdbc 操作，eg. selectOne, selectList, insert, update, delete;

```java
package com.boyan.mybatis.dao.impl;

import com.boyan.mybatis.dao.UserDao;
import com.boyan.mybatis.pojo.User;
import org.apache.ibatis.session.SqlSession;

import java.util.List;

/**
 * UserDao interface 的实现类
 */
public class UserDaoImpl implements UserDao {

    public SqlSession sqlSession;

    public UserDaoImpl(SqlSession sqlSession) {
        this.sqlSession = sqlSession;
    }

    /**
     * 根据id查询用户信息
     *
     * @param id
     * @return
     */
    @Override
    public User queryUserById(String id) {
        return this.sqlSession.selectOne("UserDao.queryUserById", id);
    }

    /**
     * 查询所有用户信息
     *
     * @return
     */
    @Override
    public List<User> queryUserAll() {
        return this.sqlSession.selectList("UserDao.queryUserAll");
    }

    /**
     * 新增用户
     *
     * @param user
     */
    @Override
    public void insertUser(User user) {
        this.sqlSession.insert("UserDao.insertUser", user);
    }

    /**
     * 更新用户信息
     *
     * @param user
     */
    @Override
    public void updateUser(User user) {
        this.sqlSession.update("UserDao.updateUser", user);
    }

    /**
     * 根据id删除用户信息
     *
     * @param id
     */
    @Override
    public void deleteUser(String id) {
        this.sqlSession.delete("UserDao.deleteUser", id);
    }
}
```

### 3）mybatis配置mapper：UserDaoMapper.xml

- 发现mapper.xml中的数据库执行语句，与impl类中的很相似（<font color=blue>引出五. 接口动态代理</font>）；

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!-- mapper:根标签，namespace：命名空间，随便写，一般保证命名空间唯一 -->
<mapper namespace="UserDao">
    <!-- statement，内容：sql语句。id：唯一标识，随便写，在同一个命名空间下保持唯一
       resultType：sql语句查询结果集的封装类型,tb_user即为数据库中的表
     -->
    <!--<select id="queryUserById" resultType="com.zpc.mybatis.pojo.User">-->
    <!--select * from tb_user where id = #{id}-->
    <!--</select>-->

    <!--使用别名-->
    <select id="queryUserById" resultType="com.boyan.mybatis.pojo.User">
        select
            tuser.id as id,
            tuser.user_name as userName,
            tuser.password as password,
            tuser.name as name,
            tuser.age as age,
            tuser.birthday as birthday,
            tuser.sex as sex,
            tuser.created as created,
            tuser.updated as updated
        from
            tb_user tuser
        where tuser.id = #{id};
    </select>

    <select id="queryUserAll" resultType="com.boyan.mybatis.pojo.User">
        select * from tb_user;
    </select>

    <!--插入数据-->
    <insert id="insertUser" parameterType="com.boyan.mybatis.pojo.User">
        INSERT INTO tb_user (
            id,
            user_name,
            password,
            name,
            age,
            sex,
            birthday,
            created,
            updated
        )
        VALUES
            (
                #{id},
                #{userName},
                #{password},
                #{name},
                #{age},
                #{sex},
                #{birthday},
                now(),
                now()
            );
    </insert>

    <update id="updateUser" parameterType="com.boyan.mybatis.pojo.User">
        UPDATE tb_user
        <trim prefix="set" suffixOverrides=",">
            <if test="userName!=null">user_name = #{userName},</if>
            <if test="password!=null">password = #{password},</if>
            <if test="name!=null">name = #{name},</if>
            <if test="age!=null">age = #{age},</if>
            <if test="sex!=null">sex = #{sex},</if>
            <if test="birthday!=null">birthday = #{birthday},</if>
            updated = now(),
        </trim>
        WHERE
        (id = #{id});
    </update>

    <delete id="deleteUser">
        delete from tb_user where id=#{id}
    </delete>
</mapper>
```

### 4）在mybatis-config.xml中添加UserDaoMapper.xml的配置

```xml
<mapper resource="mappers/UserDaoMapper.xml"/>
```

### 5）测试UserDao：UserDaoTest.java

- 在UserDao接口文件中，右键Generate测试类，测试接口中的各方法；

```java
package com.boyan.mybatis.dao;

import com.boyan.mybatis.dao.impl.UserDaoImpl;
import com.boyan.mybatis.pojo.User;
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import org.junit.After;
import org.junit.Before;
import org.junit.Test;

import java.io.InputStream;
import java.util.Date;
import java.util.List;

import static org.junit.Assert.*;

/**
 * UserDao interface 接口的测试类
 */
public class UserDaoTest {

    public UserDao userDao;
    public SqlSession sqlSession;

    @Before
    public void setUp() throws Exception {
        // mybatis-config.xml
        String resource = "mybatis-config.xml";
        // 读取配置文件
        InputStream is = Resources.getResourceAsStream(resource);
        // 构建SqlSessionFactory
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(is);
        // 获取sqlSession
        sqlSession = sqlSessionFactory.openSession();
        this.userDao = new UserDaoImpl(sqlSession);
    }

    @After
    public void tearDown() throws Exception {
        sqlSession.close();
    }

    @Test
    public void queryUserById() throws Exception {
        System.out.println(this.userDao.queryUserById("1"));
    }

    @Test
    public void queryUserAll() throws Exception {
        List<User> userList = this.userDao.queryUserAll();
        for (User user : userList) {
            System.out.println(user);   // 自动调用toString()方法
        }
    }

    @Test
    public void insertUser() throws Exception {
        User user = new User();
        user.setId("3");
        user.setAge(16);
        user.setBirthday(new Date("1990/09/02"));
        user.setName("大鹏");
        user.setPassword("123456");
        user.setSex(1);
        user.setUserName("evan");
        this.userDao.insertUser(user);
        this.sqlSession.commit();
    }

    @Test
    public void updateUser() throws Exception {
        User user = new User();
        user.setBirthday(new Date());
        user.setName("静鹏");
        user.setPassword("654321");
        user.setSex(1);
        user.setUserName("evanjin");
        user.setId("1");
        this.userDao.updateUser(user);
        this.sqlSession.commit();
    }

    @Test
    public void deleteUser() throws Exception {
        this.userDao.deleteUser("4");
        this.sqlSession.commit();
    }
}
```

- JUnit测试结果

![](https://cdn.jsdelivr.net/gh/boyan-uni/pic-bed/img/mybatis-UserDaoTest-Junit-result.png)

- 测试后tb_user数据库表中数据项

![image-20240406172616668](https://cdn.jsdelivr.net/gh/boyan-uni/pic-bed/img/mybatis-UserDaoTest-data-tb_user.png)

------

## 四. mybatis的“接口动态代理”改造CRUD

- 在上一阶段发现，发现因为在dao（mapper）的实现类中对sqlsession的使用方式很类似，因此mybatis提供了“**接口的动态代理**”：只写接口、Mapper.xml，不写实现类。

### 0）目录结构

<img src="https://cdn.jsdelivr.net/gh/boyan-uni/pic-bed/img/mybatis-UserMapper-catalog.png" alt="image-20240411161803276" style="zoom:33%;" />

### 1）创建interface: UserMapper.java（原UserDao接口）

```java
package com.boyan.mybatis.dao;

import com.boyan.mybatis.pojo.User;
import org.apache.ibatis.annotations.Param;
import java.util.List;

public interface UserMapper {
    /**
     * 登录（直接使用注解指定传入参数名称）
     * @param userName
     * @param password
     * @return
     */
    public User login(@Param("userName") String userName, @Param("password") String password);

    /**
     * 根据表名查询用户信息（直接使用注解指定传入参数名称）
     * @param tableName
     * @return
     */
    public List<User> queryUserByTableName(@Param("tableName") String tableName);

    /**
     * 根据Id查询用户信息
     * @param id
     * @return
     */
    public User queryUserById(String id);

    /**
     * 查询所有用户信息
     * @return
     */
    public List<User> queryUserAll();

    /**
     * 新增用户信息
     * @param user
     */
    public void insertUser(User user);

    /**
     * 根据id更新用户信息
     * @param user
     */
    public void updateUser(User user);

    /**
     * 根据id删除用户信息
     * @param id
     */
    public void deleteUserById(String id);
}

```



### 2）创建mapper：UserMapper.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!-- mapper:根标签，namespace：命名空间，随便写，一般保证命名空间唯一 ，为了使用接口动态代理，这里必须是接口的全路径名-->
<mapper namespace="com.boyan.mybatis.dao.UserMapper">
    <!--
       1.#{},预编译的方式preparedstatement，使用占位符替换，防止sql注入，一个参数的时候，任意参数名可以接收
       2.${},普通的Statement，字符串直接拼接，不可以防止sql注入，一个参数的时候，必须使用${value}接收参数
     -->
    <select id="queryUserByTableName" resultType="com.boyan.mybatis.pojo.User">
        select * from ${tableName}
    </select>

    <select id="login" resultType="com.boyan.mybatis.pojo.User">
        select * from tb_user where user_name = #{userName} and password = #{password}
    </select>

    <!-- statement，内容：sql语句。
       id：唯一标识，随便写，在同一个命名空间下保持唯一，使用动态代理之后要求和方法名保持一致
       resultType：sql语句查询结果集的封装类型，使用动态代理之后和方法的返回类型一致；resultMap：二选一
       parameterType：参数的类型，使用动态代理之后和方法的参数类型一致
     -->
    <select id="queryUserById" resultType="com.boyan.mybatis.pojo.User">
        select * from tb_user where id = #{id}
    </select>
    <select id="queryUserAll" resultType="com.boyan.mybatis.pojo.User">
        select * from tb_user
    </select>
    <!-- 新增的Statement
       id：唯一标识，随便写，在同一个命名空间下保持唯一，使用动态代理之后要求和方法名保持一致
       parameterType：参数的类型，使用动态代理之后和方法的参数类型一致
       useGeneratedKeys:开启主键回写
       keyColumn：指定数据库的主键
       keyProperty：主键对应的pojo属性名
     -->
    <insert id="insertUser" useGeneratedKeys="true" keyColumn="id" keyProperty="id"
            parameterType="com.boyan.mybatis.pojo.User">
        INSERT INTO tb_user (
            id,
            user_name,
            password,
            name,
            age,
            sex,
            birthday,
            created,
            updated
        )
        VALUES
            (
                #{id},
                #{userName},
                #{password},
                #{name},
                #{age},
                #{sex},
                #{birthday},
                NOW(),
                NOW()
            );
    </insert>
    <!--
       更新的statement
       id：唯一标识，随便写，在同一个命名空间下保持唯一，使用动态代理之后要求和方法名保持一致
       parameterType：参数的类型，使用动态代理之后和方法的参数类型一致
     -->
    <update id="updateUser" parameterType="com.boyan.mybatis.pojo.User">
        UPDATE tb_user
        <trim prefix="set" suffixOverrides=",">
            <if test="userName!=null">user_name = #{userName},</if>
            <if test="password!=null">password = #{password},</if>
            <if test="name!=null">name = #{name},</if>
            <if test="age!=null">age = #{age},</if>
            <if test="sex!=null">sex = #{sex},</if>
            <if test="birthday!=null">birthday = #{birthday},</if>
            updated = now(),
        </trim>
        WHERE
        (id = #{id});
    </update>
    <!--
       删除的statement
       id：唯一标识，随便写，在同一个命名空间下保持唯一，使用动态代理之后要求和方法名保持一致
       parameterType：参数的类型，使用动态代理之后和方法的参数类型一致
     -->
    <delete id="deleteUserById" parameterType="java.lang.String">
        delete from tb_user where id=#{id}
    </delete>
</mapper>
```



### 3）在mybatis-config.xml中添加UserMapper.xml的配置

```xml
<mapper resource="mappers/UserMapper.xml"/>
```



### 4）测试UserMapper：UserMapperTest.java

```java
package com.boyan.mybatis.dao;

import org.junit.Before;
import com.boyan.mybatis.pojo.User;
import com.boyan.mybatis.dao.UserMapper;
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import org.junit.Test;

import java.io.InputStream;
import java.util.Date;
import java.util.List;

public class UserMapperTest {

    public UserMapper userMapper;

    @Before
    public void setUp() throws Exception {
        // 指定配置文件
        String resource = "mybatis-config.xml";
        // 读取配置文件
        InputStream inputStream = Resources.getResourceAsStream(resource);
        // 构建sqlSessionFactory
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        // 获取sqlSession
        SqlSession sqlSession = sqlSessionFactory.openSession(true);

        // 1. 映射文件的命名空间（namespace）必须是mapper接口的全路径
        // 2. 映射文件的statement的id必须和mapper接口的方法名保持一致
        // 3. Statement的resultType必须和mapper接口方法的返回类型一致
        // 4. statement的parameterType必须和mapper接口方法的参数类型一致（不一定）
        this.userMapper = sqlSession.getMapper(UserMapper.class);
    }

    @Test
    public void testQueryUserByTableName() {
        List<User> userList = this.userMapper.queryUserByTableName("tb_user");
        for (User user : userList) {
            System.out.println(user);
        }
    }

    @Test
    public void testLogin() {
        System.out.println(this.userMapper.login("hj", "123456"));
    }

    @Test
    public void testQueryUserById() {
        System.out.println(this.userMapper.queryUserById("1"));
    }

    @Test
    public void testQueryUserAll() {
        List<User> userList = this.userMapper.queryUserAll();
        for (User user : userList) {
            System.out.println(user);
        }
    }

    @Test
    public void testInsertUser() {
        User user = new User();
        user.setId("7");
        user.setAge(20);
        user.setBirthday(new Date());
        user.setName("大神");
        user.setPassword("123456");
        user.setSex(2);
        user.setUserName("bigGod222");
        this.userMapper.insertUser(user);
        System.out.println(user.getId());
    }

    @Test
    public void testUpdateUser() {
        User user = new User();
        user.setBirthday(new Date());
        user.setName("静静");
        user.setPassword("123456");
        user.setSex(0);
        user.setUserName("Jingjing");
        user.setId("1");
        this.userMapper.updateUser(user);
    }

    @Test
    public void testDeleteUserById() {
        this.userMapper.deleteUserById("1");
    }
}
```

- JUnit测试结果

![image-20240409115445479](https://cdn.jsdelivr.net/gh/boyan-uni/pic-bed/img/mybatis-UserMapperTest-result.png)

- 测试后tb_user数据库表中数据项

![image-20240409121936720](https://cdn.jsdelivr.net/gh/boyan-uni/pic-bed/img/mybatis-UserMapperTest-tbuser-data.png)



------

## 五. 常见问题

### QA：数据库字段名和实体类属性名不一致的问题

- 问题：查询数据的时候，发现查不到userName的信息：

> User{id=‘2’, <font color=red>userName=‘null’</font>, password=‘123456’, name=‘静静’, age=22, sex=0, birthday=‘1993-09-05’, created=‘2018-06-30 18:22:28.0’, updated=‘2018-06-30 18:22:28.0’}

- 原因：数据库的字段名是user_name，POJO中的属性名字是userName，造成mybatis无法填充对应的字段信息。

- 解决方案：
  1. 在sql语句中使用别名：
  2. 参考后面的resultMap –mapper具体的配置的时候
  3. 参考驼峰匹配 — mybatis-config.xml 的时候
