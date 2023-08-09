# mybatis：

## 1.快速入门

![image-20230725144020105](C:\Users\黄\AppData\Roaming\Typora\typora-user-images\image-20230725144020105.png)

### 1.创建user表，添加数据：

```sql
create database mybatis;
use mybatis;
create table tb_user(
id int primary key auto_increment,
username varchar(20),
password varchar(20),
gender varchar(1),
addr varchar(30)
);
insert into tb_user value(1,'zhangsan','123','男','北京');
inster into tb_user value(2,'李四','234','女','天津');
insert into tb_user value(3,'王五','11','男','西安');
```

#### 数据库结果

![image-20230725204237435](C:\Users\黄\AppData\Roaming\Typora\typora-user-images\image-20230725204237435.png)

### 2.创建模块，导入坐标：

####  1.导入依赖

```pom
<dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>3.8.1</version>
      <scope>test</scope>
    </dependency>
    <!--连接数据库-->
    <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>8.0.31</version>
    </dependency>
    <!--mybatis依赖-->
    <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis</artifactId>
      <version>3.5.5</version>
    </dependency>
    <!--日志slf4j-->
    <dependency>
      <groupId>org.slf4j</groupId>
      <artifactId>slf4j-api</artifactId>
      <version>1.7.20</version>
    </dependency>
    <dependency>
      <groupId>ch.qos.logback</groupId>
      <artifactId>logback-classic</artifactId>
      <version>1.2.3</version>
    </dependency>
    <dependency>
      <groupId>ch.qos.logback</groupId>
      <artifactId>logback-core</artifactId>
      <version>1.2.3</version>
    </dependency>
  </dependencies>
```

#### 2.logback的配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration>
<configuration>

  <appender name="stdout" class="ch.qos.logback.core.ConsoleAppender">
    <encoder>
      <pattern>%5level [%thread] - %msg%n</pattern>
    </encoder>
  </appender>

  <logger name="org.mybatis.example.BlogMapper">
    <level value="trace"/>
  </logger>
  <root level="root">
    <appender-ref ref="stdout"/>
  </root>

</configuration>
```



### 3.编写 MyBatis 核心配置文件一>替换连接信息解决硬编码问题

#### mybatis-config.xml文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "https://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
  <environments default="development">
    <environment id="development">
      <transactionManager type="JDBC"/>
      <dataSource type="POOLED">
        <!--数据库连接-->
        <property name="driver" value="com.mysql.jdbc.Driver"/>
        <property name="url" value="jdbc:mysql://localhost:3306/mybatis?useSSl=false"/>
        <property name="username" value="root"/>
        <property name="password" value="Aviciiforever"/>
      </dataSource>
    </environment>
  </environments>
  <mappers>
    <!--加载sql的映射文件-->
    <mapper resource="UserMapper.xml"/>
  </mappers>
</configuration>
```



### 4.编写 SQL 映射文件一>统一管理sql语句，解决硬编码问题：

#### UserMapper.xml:

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "https://mybatis.org/dtd/mybatis-3-mapper.dtd">

<!--
namespce：命名空间，用于对sql进行分类管理，便于后期维护
-->
<mapper namespace="test">
  <select id="selectAll" resultType="com.hmh.mybatis.pojo.User">
    select * from tb_user;
  </select>
</mapper>
```



### 5.编码：

#### 项目大概结构：

![image-20230725204659351](C:\Users\黄\AppData\Roaming\Typora\typora-user-images\image-20230725204659351.png)

#### 1.定义pojo：

##### user类

```java
package com.hmh.mybatis.pojo;

public class User {
    private Integer id;
    private String username;
    private String password;
    private String gender;
    private String addr;

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    public String getGender() {
        return gender;
    }

    public void setGender(String gender) {
        this.gender = gender;
    }

    public String getAddr() {
        return addr;
    }

    public void setAddr(String addr) {
        this.addr = addr;
    }

    @Override
    public String toString() {
        return "User{" +
                "id=" + id +
                ", username='" + username + '\'' +
                ", password='" + password + '\'' +
                ", gender='" + gender + '\'' +
                ", addr='" + addr + '\'' +
                '}';
    }
}
//这里也可以使用注解
```



#### 2.加载核心配置文件，获取 SqlSessionFactory 对象:

##### 这里核心配置文件也就是mybatis-config.xml

```java
//1.加载mybatis的核心配置文件，获取SqlSessionFactory对象
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);

 //2.获取SqlSession对象
        SqlSession sqlSession = sqlSessionFactory.openSession();
```

#### 3.获取 SqlSession 对象执行 SQL 语句:

```java
//3.执行sql语句
        //第一个参数：sql语句的唯一标识
        //第二个参数：执行sql语句需要的参数
        List<User> users = sqlSession.selectList("test.selectAll");

```

#### 4.释放资源:

```java
 //4.关闭资源
        sqlSession.close();
```



### 最后完整测试代码及测试结果

```java
public class MybatisDemo {
    public static void main(String[] args) throws IOException {
        //1.加载mybatis的核心配置文件，获取SqlSessionFactory对象
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);

        //2.获取SqlSession对象
        SqlSession sqlSession = sqlSessionFactory.openSession();

        //3.执行sql语句
        //第一个参数：sql语句的唯一标识
        //第二个参数：执行sql语句需要的参数
        List<User> users = sqlSession.selectList("test.selectAll");

        System.out.println(users);

        //4.关闭资源
        sqlSession.close();

    }
}

```





## 解决sql映射文件的警告

![image-20230726200450080](C:\Users\黄\AppData\Roaming\Typora\typora-user-images\image-20230726200450080.png)

## 2.Mapper代理开发：

![image-20230726200616159](C:\Users\黄\AppData\Roaming\Typora\typora-user-images\image-20230726200616159.png)

### 此处存在硬编码的问题，及是将sqlID和namespace当作参数传到selectList方法里面，去执行List结果

### test.selectAll当作字符串写死代码，这里主要是selectAll这个sqlID存在硬编码问题，此时报错为

### Mapped Statements collection does not contain value for test.selectAll

### 此时就需要用mapper做代理开发：

![image-20230726201315353](C:\Users\黄\AppData\Roaming\Typora\typora-user-images\image-20230726201315353.png)

### 通过SqlSession来获取Mapper这个类，将UserMapper.class传进来，返回一个UserMapper类型

### 这个UserMapper是个接口，接口中将来会存在很多方法，这些方法和你UserMapper.xml文件中的

### sqlID一一对应

![image-20230726202250863](C:\Users\黄\AppData\Roaming\Typora\typora-user-images\image-20230726202250863.png)

### 1.定义与 SQL 映射文件同名的 Mapper 接口，并且将 Mapper 接囗和 SQL 映射文件放置在同一目录下

![image-20230727001031864](C:\Users\黄\AppData\Roaming\Typora\typora-user-images\image-20230727001031864.png)

#### 如上图所示：由于我们定义了一个UserMapper接口在com.hmh.mybatis.mapper包下

#### 所以我们在将UserMapper.xml这个sql映射文件定义在同一个包，这里有两种处理模式：

#### 一：把UserMapper.xml文件就放在UseMapper接口同一个目录下。

#### 二：就是在resources目录下建一个相同名称的包。注意：在resources目录下建包如图：

![image-20230727001802371](C:\Users\黄\AppData\Roaming\Typora\typora-user-images\image-20230727001802371.png)

#### 形如上图所示



### 2 ．设置 SQL 映射文件的 namespace 属性为 Mapper 接口全限定名：

#### 将原来的namespace改成对应包下的接口

```xml
<mapper namespace="test">
  <select id="selectAll" resultType="com.hmh.mybatis.pojo.User">
    select * from tb_user;
  </select>
</mapper>
```



### 3.在Mapper接口中定义方法方法名就是 SQL 映射文件中sql语句的 id ，并保持参数类型和返回值类型一致

```xml
<select id="selectAll" resultType="com.hmh.mybatis.pojo.User">
    select * from tb_user;
  </select>
```

#### 解释该代码中id和resultType的作用

#### id：这里是对应你接口中的方法名，例如：这里我id值写的是selectAll

#### 接下来在我UserMapper接口中定义有selectAll这个方法

#### resultType：这里对应我们返回值的结果类型，例如：我这里resultType写的是com.hmh.mybatis.pojo.User

#### 前面com.hmh.mybatis.pojo这段是包名，User是返回的结果类型



#### 因此我们在UserMapper下会定义一个返回User类型结果的方法，代码如下：

```java
public interface UserMapper {
    //查询所有用户
    //返回值不是一个user对象，而是一个list集合
    List<User> selectAll();

}
```

### 4 ，编码：

##### 1. 通过 SqlSession 的 getMapper 方法获取 Mapper 接口的代理对象

##### 2 ．调用对应方法完成sql的执行

```java
public class MybatisDemo2 {
    public static void main(String[] args) throws IOException {
        //1.加载mybatis的核心配置文件，获取SqlSessionFactory对象
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);

        //2.获取SqlSession对象
        SqlSession sqlSession = sqlSessionFactory.openSession();

        //3.执行sql语句
        //第一个参数：sql语句的唯一标识
        //第二个参数：执行sql语句需要的参数
        //List<User> users = sqlSession.selectList("test.selectAll");
        //3.1获取mapper接口的代理对象
        UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
        List<User> users = userMapper.selectAll();

        System.out.println(users);

        //4.关闭资源
        sqlSession.close();

    }
}
```

#### 细节：如果 Mapper 接口名称和 SQL 映射文件名称相同，并在同一目录下，则可以使用包扫描的方式简化SQL映射文件的加载这里用package来扫描Mapper接口



```xml
 <mappers>
    <!--加载sql的映射文件-->
   <!-- <mapper resource="com/hmh/mybatis/mapper/UserMapper.xml"/>-->
    <!--Mapper代理方式-->
     <package name="com.hmh.mybatis.mapper"/>
  </mappers>
```

#### 注意：这里可能会出现的报错The error may exist in com/hmh/mybatis/mapper/UserMapper.java (best guess)

####  Cause: org.apache.ibatis.binding.BindingException: Type interface com.hmh.mybatis.mapper.UserMapper is already known to the MapperRegistry.

#### 这里报错起因是类型接口com.hmh.mybatis.mapper.UserMapper已经为MapperRegistry所知。

#### 所以这里我们解决方法是检查在mybatis-config.xml里面mapper资源加载二选一：

##### 1.加载sql的映射文件

```xml
<mapper resource="com/hmh/mybatis/mapper/UserMapper.xml"/>
```

##### 2.Mapper代理方式：

```xml
 <package name="com.hmh.mybatis.mapper"/>
```





## 4.mybatis核心配置文件：[mybatis – MyBatis 3 | 配置](https://mybatis.org/mybatis-3/zh/configuration.html)



![image-20230727153726097](C:\Users\黄\AppData\Roaming\Typora\typora-user-images\image-20230727153726097.png)

![image-20230727144552876](C:\Users\黄\AppData\Roaming\Typora\typora-user-images\image-20230727144552876.png

### 以现有的mybatis-config.xml文件讲解：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "https://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
  <environments default="development">
    <environment id="development">
      <transactionManager type="JDBC"/>
      <dataSource type="POOLED">
        <!--数据库连接-->
        <property name="driver" value="com.mysql.jdbc.Driver"/>
        <property name="url" value="jdbc:mysql://localhost:3306/mybatis?useSSl=false"/>
        <property name="username" value="root"/>
        <property name="password" value="Aviciiforever"/>
      </dataSource>
    </environment>
  </environments>
  <mappers>
    <!--加载sql的映射文件-->
    <mapper resource="com/hmh/mybatis/mapper/UserMapper.xml"/>
    <!--Mapper代理方式-->
    <package name="com.hmh.mybatis.mapper"/>
  </mappers>
</configuration>
```

### environments:MyBatis 可以配置成适应多种环境，这种机制有助于将 SQL 映射应用于多种数据库之中， 现实情况下有多种理由需要这么做。例如，开发、测试和生产环境需要有不同的配置；或者想在具有相同 Schema 的多个生产数据库中使用相同的 SQL 映射。还有许多类似的使用场景。

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "https://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <!--environments:配置数据库连接环境信息。可以配置多个environment，通过default属性切换不同的environment·-->
  <environments default="development">
    <environment id="development">
      <transactionManager type="JDBC"/>
      <dataSource type="POOLED">
        <!--数据库连接-->
        <property name="driver" value="com.mysql.jdbc.Driver"/>
        <property name="url" value="jdbc:mysql://localhost:3306/mybatis?useSSl=false"/>
        <property name="username" value="root"/>
        <property name="password" value="Aviciiforever"/>
      </dataSource>
    </environment>
      
       <environment id="test">
      <transactionManager type="JDBC"/>
      <dataSource type="POOLED">
        <!--数据库连接-->
        <property name="driver" value="com.mysql.jdbc.Driver"/>
        <property name="url" value="jdbc:mysql://localhost:3306/mybatis?useSSl=false"/>
        <property name="username" value="root"/>
        <property name="password" value="Aviciiforever"/>
      </dataSource>
    </environment>
  </environments>
  <mappers>
    <!--加载sql的映射文件-->
    <mapper resource="com/hmh/mybatis/mapper/UserMapper.xml"/>
    <!--Mapper代理方式-->
    <package name="com.hmh.mybatis.mapper"/>
  </mappers>
</configuration>
```

### 这里在environments下，environment是配置多个数据库环境 ，以此切换不同的数据源：

```
 <environments default="development">//这里可将default的值设为environment的id值development或者test
```

### transactionManager：事务的管理方式，这里我们设置事务类型为JDBC事务，将来设置事务也不需要用mybatis来管理事务，将来都是用spring的方式来管理，这一步以后会被接管



### dataSource：数据库连接池，默认的数据库连接池是pooled

```xml
   <dataSource type="POOLED">
```

### 将来数据库的数据源将来也会被spring接管



### 起别名：

```xml
  <typeAliases>
    <package name="com.hmh.mybatis.pojo"/>
  </typeAliases>
```

### 这里相当于给该包下的所有实体类起了一个别名，所以我们这里返回resultType类型的的时候可以简化

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "https://mybatis.org/dtd/mybatis-3-mapper.dtd">

<!--
namespce：命名空间，用于对sql进行分类管理，便于后期维护
-->
<mapper namespace="com.hmh.mybatis.mapper.UserMapper">
  <select id="selectAll" resultType="User">
    select * from tb_user;
  </select>
</mapper>
```

### 实现的效果跟上面一样



## 5.配置文件完成增删改查：

![屏幕截图 2023-08-02 165526](D:\图片\Screenshots\屏幕截图 2023-08-02 165526.png)

![屏幕截图 2023-08-02 165235](D:\图片\Screenshots\屏幕截图 2023-08-02 165235.png)

### 1.数据库设计tb_brand:

```sql
-- 删除tb_brand表
drop table if exists tb_brand;
-- 创建tb_brand表
create table tb_brand(
--     id主键
   id int primary key auto_increment,
--    品牌名称
   brand_name varchar(20),
--    企业名称
   company_name varchar(20),
--    排序字段
   ordered int,
--    描述信息
   description varchar(100),
--    状态:0:禁用 1:启用
    status int
);
-- 添加数据
insert into tb_brand(brand_name,company_name,ordered,description,status)
values('三只松鼠','三只松鼠有限股份公司',5,'好吃不上火',0),
      ('华为','华为技术有限公司',100,'华为致力于把数字世界带入每个人，每个家庭，每个组织，构建万物互联的智能世界',1),
      ('小米','小米科技有限公司',50,'are you ok',1);

select * from tb_brand;

```



### 2.实体类Brand：

```java
package com.hmh.mybatis.pojo;

public class Brand {
    //id主键
    private Integer id;
    //品牌名称
    private String brandName;
    //企业名称
    private String companyName;
    //排序字段
    private Integer ordered;
    //品牌描述
    private String description;
    //状态 0：禁用 1：启用
    private Integer status;

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getBrandName() {
        return brandName;
    }

    public void setBrandName(String brandName) {
        this.brandName = brandName;
    }

    public String getCompanyName() {
        return companyName;
    }

    public void setCompanyName(String companyName) {
        this.companyName = companyName;
    }

    public Integer getOrdered() {
        return ordered;
    }

    public void setOrdered(Integer ordered) {
        this.ordered = ordered;
    }

    public String getDescription() {
        return description;
    }

    public void setDescription(String description) {
        this.description = description;
    }

    public Integer getStatus() {
        return status;
    }

    public void setStatus(Integer status) {
        this.status = status;
    }

    @Override
    public String toString() {
        return "Brand{" +
                "id=" + id +
                ", brandName='" + brandName + '\'' +
                ", companyName='" + companyName + '\'' +
                ", ordered=" + ordered +
                ", description='" + description + '\'' +
                ", status=" + status +
                '}';
    }
}

```



### 3.测试实体类：

![image-20230802213438728](C:\Users\黄\AppData\Roaming\Typora\typora-user-images\image-20230802213438728.png)

#### test包下新建一个包com.hmh.mybatis.test,然后新建Java测试类MybatisTest



### 4.安装MybatisX插件：

![image-20230802213822383](C:\Users\黄\AppData\Roaming\Typora\typora-user-images\image-20230802213822383.png)

### 前面准备工作我想大家已经完成了，接下来就是对增删改查的实现喵



### 查询：

#### 1.查询所有数据：

![屏幕截图 2023-08-02 222050](D:\图片\Screenshots\屏幕截图 2023-08-02 222050.png)

##### 注意：不管完成增删改查哪一个功能时，都要遵循上面123的步骤，规范开发习惯。

##### 1.创建接口BrandMapper：

```java
public interface BrandMapper {
    /*
    * 查询所有
    * */
   public List<Brand> selectAll();
}
```

##### 2.创建BrandMapper.xml文件：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "https://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.hmh.mybatis.mapper.BrandMapper">

    <select id="selectAll" resultType="Brand">
        select * from tb_brand;
    </select>
</mapper>
```

##### 3.测试及结果：

```java
 @Test
    public void testSelectAll() throws IOException {
         //1.加载mybatis的核心配置文件，获取SqlSessionFactory对象
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);

        //2.获取SqlSession对象
        SqlSession sqlSession = sqlSessionFactory.openSession();

        BrandMapper brandMapper = sqlSession.getMapper(BrandMapper.class);
        List<Brand> brands = brandMapper.selectAll();
        System.out.println(brands);

        sqlSession.close();
    }
```

![image-20230802224853913](C:\Users\黄\AppData\Roaming\Typora\typora-user-images\image-20230802224853913.png)

##### 大家能注意到我这里查出来的brandName和companyName都为null，这就表明brandName和companyName是没有封装上的。

##### 为什么会出现这种情况捏？

##### 在数据库里面我们这两个字段为brand_name和company_name,而在Brand类里面这两个字段为brandName和companyName驼峰式命名，数据库字段与Brand类的属性值未对应上，所以这里我们就需要对这两个字段做映射。

```xml
<!--
数据库字段名称与实体类属性名称不一致，则不能自动封装数据
解决方案:
1.在sql语句中使用别名
  <select id="selectAll" resultType="Brand">
             select id,brand_name as brandName,company_name as companyName,ordered,description,status from tb_brand;
    </select>
 对不一样的字段使用别名，别名就是实体类中的属性名称，这样就可以自动封装数据
 但是这样的话，每次都要写别名，比较麻烦,可以使用mybatis的sql片段来抽取公共的sql语句
 sql片段
    <sql id="brandColumns">
        id,brand_name as brandName,company_name as companyName ,ordered,description,status
    </sql>
    <select id="selectAll" resultType="Brand">
        select <include refid="brandColumns"/> from tb_brand;
   但是sql片段使用不灵活，如果有多个表需要使用这个sql片段，但是表中的字段不一样，这样就不能使用sql片段了  
-->
```

##### 这里用resultMap可以解决上面的不足：

```xml
<resultMap id="brandResultMap" type="Brand">               
    <!--id:唯一标识，type:映射类型,也支持别名的方式-->                               
    <!--                                                   
    id:主键字段的映射  result:实体类中的属性名称，一般字段的映射                                
    property:实体类中的属性名称                                     
    column:数据库中的字段名称                                       
    -->                                                    
    <result column="brand_name" property="brandName"/>     
    <result column="company_name" property="companyName"/> 
</resultMap>                                               
<select id="selectAll" resultMap="brandResultMap">         
    select * from tb_brand;
</select>
```

#####   resultMap:结果集映射，将数据库中的字段映射到实体类中的属性-->

      resultMap的使用步骤:-->
          1.编写resultMap-->
          2.在select标签中使用resultMap-->

#### 2.查询详情：

![image-20230803112546342](C:\Users\黄\AppData\Roaming\Typora\typora-user-images\image-20230803112546342.png)

##### 1.定义mapper接口：需要参数id，返回结果为brand

```java
 User selectById(Integer id);
```

##### 2.SQL映射文件：

```xml
<select id="selectById" parameterType="int" resultMap="brandResultMap"> 
    select * from tb_brand where id = #{id};        
</select>                                           
```

##### 为什么要写#{}

##### 在mybatis中有参数占位符分别为#{}和${}

##### #{}:会将参数转换为？，相当于JDBC里面的PrepareStatement，防止sql注入

##### ${}:拼接sql，会存在sql注入的问题

##### 两者的使用时机：

##### 在传参数是，一般使用#{}，而${}在表名或者列明不固定的情况下使用

##### 设置参数类型parameterType

##### 特数字符：在xml文件中因为<是标签的开始符号，所以在xml中使用<会报错。

##### 如何解决xml文件中<报错的问题：

##### 1.转义字符 ：&lt;就是小于符号

```xml
<select id="selectById" resultMap="brandResultMap">
select tb_brand where id &lt; #{id};
</select>
```



##### 2.CDATA区：

```xml
<select id="selectById" resultMap="brandResultMap">
select tb_brand where id 
    <![CDATA[
<
]]>
    #{id};
</select>
```

##### 注意在特殊字符较少时就用转义字符，反之，则用CDATA区

##### 3.测试类

```java
      @Test
    public void testSelectById() throws IOException {
        //获取id为1的用户信息，现在设置为静态的，后面再改成动态的
        int id = 1;
         //1.加载mybatis的核心配置文件，获取SqlSessionFactory对象
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);

        //2.获取SqlSession对象
        SqlSession sqlSession = sqlSessionFactory.openSession();

        BrandMapper brandMapper = sqlSession.getMapper(BrandMapper.class);
          Brand brand = brandMapper.selectById(id);
          System.out.println(brand);

          sqlSession.close();
    }
```



#### 3.条件查询：

![image-20230804093333435](C:\Users\黄\AppData\Roaming\Typora\typora-user-images\image-20230804093333435.png)

![image-20230804093358115](C:\Users\黄\AppData\Roaming\Typora\typora-user-images\image-20230804093358115.png)

![image-20230804093424850](C:\Users\黄\AppData\Roaming\Typora\typora-user-images\image-20230804093424850.png)

##### 多条件查询：

##### 1.接口定义：

```java
 /*
    * 条件查询
    * 这里参数接收有三种方式
    * 1.多个参数，使用@Param注解
    * 2.使用map
    * 3.使用对象
    * */
    List<Brand> selectByCondition(@Param("status") int status,@Param("brandName") String brandName,@Param("companyName") String companyName);

    List<Brand> selectByCondition2(Brand brand);

    List<Brand> selectByCondition3(Map map);
```

##### 2.SQL定义：

```xml
 <select id="selectByCondition" resultMap="brandResultMap">
        select * from tb_brand where status = #{status} and company_name like #{companyName} and brand_name like #{brandName};
    </select>
    <select id="selectByCondition2" resultMap="brandResultMap">
        select * from tb_brand where status = #{status} and company_name like #{companyName} and brand_name like #{brandName};
    </select>
    <select id="selectByCondition3" resultMap="brandResultMap">
        select * from tb_brand where status = #{status} and company_name like #{companyName} and brand_name like #{brandName};
    </select>
```

##### 3.测试：

```java
 @Test
    public void testSelectByCondition() throws IOException {
        //获取status为1的,brandName叫华为，companyName叫华为的用户信息，现在设置为静态的，后面再改成动态的
        int status = 1;
        String brandName = "华为";
        String companyName = "华为";


        //处理参数,对模糊查询的参数进行处理，即是将参数写成模糊表达式的参数形式，前后加上%
        //%是占位符，代表一个或多个，前后都加代表包含华为的值；下划线_代表一个占位符，只占一个
        brandName = "%" + brandName + "%";
        companyName = "%" + companyName + "%";

        //1.加载mybatis的核心配置文件，获取SqlSessionFactory对象
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);

        //2.获取SqlSession对象
        SqlSession sqlSession = sqlSessionFactory.openSession();

        BrandMapper brandMapper = sqlSession.getMapper(BrandMapper.class);
        List<Brand> brands = brandMapper.selectByCondition(status,brandName,companyName);
        System.out.println(brands);

        sqlSession.close();
    }
      @Test
    public void testSelectByCondition2() throws IOException {
        //获取status为1的,brandName叫华为，companyName叫华为的用户信息，现在设置为静态的，后面再改成动态的
        int status = 1;
        String brandName = "华为";
        String companyName = "华为";


        //处理参数,对模糊查询的参数进行处理，即是将参数写成模糊表达式的参数形式，前后加上%
        //%是占位符，代表一个或多个，前后都加代表包含华为的值；下划线_代表一个占位符，只占一个
        brandName = "%" + brandName + "%";
        companyName = "%" + companyName + "%";

        //封装对象
          Brand brand = new Brand();
          brand.setStatus(status);
          brand.setBrandName(brandName);
          brand.setCompanyName(companyName);

        //1.加载mybatis的核心配置文件，获取SqlSessionFactory对象
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);

        //2.获取SqlSession对象
        SqlSession sqlSession = sqlSessionFactory.openSession();

        BrandMapper brandMapper = sqlSession.getMapper(BrandMapper.class);
        List<Brand> brands = brandMapper.selectByCondition2(brand);
        System.out.println(brands);

        sqlSession.close();
    }
       @Test
    public void testSelectByCondition3() throws IOException {
        //获取status为1的,brandName叫华为，companyName叫华为的用户信息，现在设置为静态的，后面再改成动态的
        int status = 1;
        String brandName = "华为";
        String companyName = "华为";


        //处理参数,对模糊查询的参数进行处理，即是将参数写成模糊表达式的参数形式，前后加上%
        //%是占位符，代表一个或多个，前后都加代表包含华为的值；下划线_代表一个占位符，只占一个
        brandName = "%" + brandName + "%";
        companyName = "%" + companyName + "%";

        //封装对象
           Map map = new HashMap();
           map.put("status",status);
           map.put("brandName",brandName);
           map.put("companyName",companyName);

        //1.加载mybatis的核心配置文件，获取SqlSessionFactory对象
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);

        //2.获取SqlSession对象
        SqlSession sqlSession = sqlSessionFactory.openSession();

        BrandMapper brandMapper = sqlSession.getMapper(BrandMapper.class);
        List<Brand> brands = brandMapper.selectByCondition3(map);
        System.out.println(brands);

        sqlSession.close();
    }
```

##### 讲解为啥这里brandName="%"+brandName+"%",companyName="%"+companyName+"%".

##### 在这里要讲解到模糊查询：

##### 1."%" 百分号通配符: 表示任何字符出现任意次数 (可以是0次)。

2."_" 下划线通配符:表示只能匹配单个字符,不能多也不能少,就是一个字符。当然，也可以like "陈"，数量不限。
3.like操作符:LIKE作用是指示mysql后面的搜索模式是利用通配符而不是直接相等匹配进行比较；但如果like后面没出现通配符，则在SQL执行优化时将 like 默认为 “=”执行

在mybatis里面的模糊查询也可以在sql语句里面,既可以不用在测试中处理参数：

```xml
<select>
 select * from tb_brand where status = #{status} and company_name like '%#{companyName}%' and brand_name like '%#{brandName}%';
</select>

<select>
 select * from tb_brand where status = #{status} and company_name like concat('%',#{companyName},'%') and brand_name like concat('%',#{brandName},'%');
</select>

<select>
 select * from tb_brand where status = #{status} and company_name like "%"#{companyName}"%" and brand_name like "%"#{brandName}"%";
</select>
```

##### 这里我们的程序是有bug的，必须要三个条件同时输入才可以查找对应数据，所以我们这里要用动态sql解决上面的bug

![image-20230805143847631](C:\Users\黄\AppData\Roaming\Typora\typora-user-images\image-20230805143847631.png)

##### 上面接口不用动，需要对sql进行改造

```xml
 <select id="selectByCondition3" resultMap="brandResultMap">
        select * from tb_brand where
     <!--test这里是做条件判断的内容-->
        <if test="status != null">
            status = #{status}
        </if>
        <if test="companyName != null and companyName !='' ">
            and company_name like #{companyName}
        </if>
        <if test="brandName != null and brandName != ''">
            and brand_name like #{brandName}
        </if>
    </select>
```

##### 但是这里可能会出现报错

![image-20230805145344267](C:\Users\黄\AppData\Roaming\Typora\typora-user-images\image-20230805145344267.png)

##### 在我们注释掉statusmap.put("status",status);时，就会出现上面的报错

```java
  public void testSelectByCondition3() throws IOException {
        //获取status为1的,brandName叫华为，companyName叫华为的用户信息，现在设置为静态的，后面再改成动态的
        int status = 1;
        String brandName = "华为";
        String companyName = "华为";


        //处理参数,对模糊查询的参数进行处理，即是将参数写成模糊表达式的参数形式，前后加上%
        //%是占位符，代表一个或多个，前后都加代表包含华为的值；下划线_代表一个占位符，只占一个
        brandName = "%" + brandName + "%";
        companyName = "%" + companyName + "%";

        //封装对象
           Map map = new HashMap();
           //map.put("status",status);
           map.put("brandName",brandName);
           //map.put("companyName",companyName);

        //1.加载mybatis的核心配置文件，获取SqlSessionFactory对象
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);

        //2.获取SqlSession对象
        SqlSession sqlSession = sqlSessionFactory.openSession();

        BrandMapper brandMapper = sqlSession.getMapper(BrandMapper.class);
        List<Brand> brands = brandMapper.selectByCondition3(map);
        System.out.println(brands);

        sqlSession.close();
    }
```

##### 根据上面报错可以看到是因为sql语法出错了：而这里拼接的sql语句为： select * from tb_brand where  and brand_name like ?

##### 如果这里我们去掉and，那么如果有status条件时也会出现sql语法出错的问题



##### 如何解决：

##### 1.利用恒等式

```xml
 <select id="selectByCondition3" resultMap="brandResultMap">
        select * from tb_brand where 1=1
        <if test="status != null">
            and status = #{status}
        </if>
        <if test="companyName != null and companyName !='' ">
            and company_name like #{companyName}
        </if>
        <if test="brandName != null and brandName != ''">
            and brand_name like #{brandName}
        </if>
    </select>
```

##### 这里在拼接sql时就不会绕开这个恒等式，以此做判断

##### 2.利用mybatis提供的where标签：

```xml
 <select id="selectByCondition3" resultMap="brandResultMap">
        select * from tb_brand 
     <where>
        <if test="status != null">
            and status = #{status}
        </if>
        <if test="companyName != null and companyName !='' ">
            and company_name like #{companyName}
        </if>
        <if test="brandName != null and brandName != ''">
            and brand_name like #{brandName}
        </if>
     </where>
    </select>
```

##### 接下来我们看看这里sql是怎样拼接的

![image-20230805151226533](C:\Users\黄\AppData\Roaming\Typora\typora-user-images\image-20230805151226533.png)

##### 这里可以看见where标签会自动生成一个where做条件判断，从有满足判断那里开始

![image-20230805151508372](C:\Users\黄\AppData\Roaming\Typora\typora-user-images\image-20230805151508372.png)



##### 单条件-动态条件查询：

![image-20230805151629293](C:\Users\黄\AppData\Roaming\Typora\typora-user-images\image-20230805151629293.png)

##### 1.定义接口：

```java
 /*
    * 单条件动态查询
    * */
    List<Brand> selectByConditionSingle(Brand brand);
```



##### 2.编写sql语句：

```xml
 <select id="selectByConditionSingle" resultMap="brandResultMap">
        select * from tb_brand
        where
        <choose><!--choose这里相当于switch-->
            <when test="status != null"><!--when这里相当于case-->
                status = #{status}
            </when>
            <when test="companyName != null and companyName !='' ">
                company_name like #{companyName}
            </when>
            <when test="brandName != null and brandName != ''">
                brand_name like #{brandName}
            </when>
        </choose>
    </select>

```

##### 3.测试

```java
  @Test
    public void testSelectByConditionSingle() throws IOException {
        //获取status为1的,brandName叫华为，companyName叫华为的用户信息，现在设置为静态的，后面再改成动态的
        int status = 1;
        String brandName = "华为";
        String companyName = "华为";


        //处理参数,对模糊查询的参数进行处理，即是将参数写成模糊表达式的参数形式，前后加上%
        //%是占位符，代表一个或多个，前后都加代表包含华为的值；下划线_代表一个占位符，只占一个
        brandName = "%" + brandName + "%";
        companyName = "%" + companyName + "%";

        //封装对象
          Brand brand = new Brand();
          brand.setStatus(status);
          //brand.setBrandName(brandName);
          //brand.setCompanyName(companyName);

        //1.加载mybatis的核心配置文件，获取SqlSessionFactory对象
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);

        //2.获取SqlSession对象
        SqlSession sqlSession = sqlSessionFactory.openSession();

        BrandMapper brandMapper = sqlSession.getMapper(BrandMapper.class);
        List<Brand> brands = brandMapper.selectByConditionSingle(brand);
         System.out.println(brands);


         sqlSession.close();
    }
```

##### 如果这里我们一个条件都不选，既是是没有封装任何查询参数的值，都为null，及会出现报错

![image-20230805153712337](C:\Users\黄\AppData\Roaming\Typora\typora-user-images\image-20230805153712337.png)

##### 这里我们可以看见sql语句的拼接where后面没有条件，所以出现了sql语法的报错

##### 如何解决：

##### 这里我们还有个标签没有使用otherwise标签，可以在添加一个默认选项来避免

```xml
 <select id="selectByConditionSingle" resultMap="brandResultMap">
        select * from tb_brand
        where
        <choose><!--choose这里相当于switch-->
            <when test="status != null"><!--when这里相当于case-->
                status = #{status}
            </when>
            <when test="companyName != null and companyName !='' ">
                company_name like #{companyName}
            </when>
            <when test="brandName != null and brandName != ''">
                brand_name like #{brandName}
            </when>
            <otherwise><!--otherwise这里相当于default-->
                1=1
            </otherwise>
        </choose>
    </select>
```

##### 也可以通过where动态变化，也可以避免

```xml
<select id="selectByConditionSingle" resultMap="brandResultMap">
        select * from tb_brand
        <where>
            <choose><!--choose这里相当于switch-->
            <when test="status != null"><!--when这里相当于case-->
                status = #{status}
            </when>
            <when test="companyName != null and companyName !='' ">
                company_name like #{companyName}
            </when>
            <when test="brandName != null and brandName != ''">
                brand_name like #{brandName}
            </when>
        </choose>
        </where>
    </select>
```



### 添加：

![image-20230805154700976](C:\Users\黄\AppData\Roaming\Typora\typora-user-images\image-20230805154700976.png)

#### 1.接口：

```java
/*
    * 添加操作
    * */
    void add(Brand brand);
```

#### 2.编写sql语句

```xml
 <insert id="add">
        insert into tb_brand(brand_name,company_name,ordered,description,status) values(#{brandName},#{companyName},#{ordered},#{description},#{status});
    </insert>
```

#### 3.测试及结果，与数据对比，检查是否添加成功

```java
 @Test
    public void testAdd() throws IOException {
        //获取status为1的,brandName叫华为，companyName叫华为的用户信息，现在设置为静态的，后面再改成动态的
        int status = 1;
        String brandName = "波导手机";
        String companyName = "波导手机";
        String description = "波导手机，手机中的战斗机";
        int ordered = 1;

        //封装对象
          Brand brand = new Brand();
          brand.setStatus(status);
          brand.setBrandName(brandName);
          brand.setCompanyName(companyName);
          brand.setDescription(description);
          brand.setOrdered(ordered);

        //1.加载mybatis的核心配置文件，获取SqlSessionFactory对象
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);

        //2.获取SqlSession对象
        SqlSession sqlSession = sqlSessionFactory.openSession();

        BrandMapper brandMapper = sqlSession.getMapper(BrandMapper.class);
        brandMapper.add(brand);


         sqlSession.close();
    }
```

##### 测试结果：

![image-20230805160012217](C:\Users\黄\AppData\Roaming\Typora\typora-user-images\image-20230805160012217.png)

##### 数据库结果：

![image-20230805160216913](C:\Users\黄\AppData\Roaming\Typora\typora-user-images\image-20230805160216913.png)

##### 结果发现这里并没有添加上数据，为甚么捏？

##### 从上面测试结果来看，在 JDBC 连接上将自动提交设置为 false，所以这里没有自动提交上事务，得手动提交事务，所以这里就把事务回滚了，没有添加到数据库里面去。

##### 解决方案：自己手动提交事务就行了

```java
     @Test
    public void testAdd() throws IOException {
        //获取status为1的,brandName叫华为，companyName叫华为的用户信息，现在设置为静态的，后面再改成动态的
        int status = 1;
        String brandName = "波导手机";
        String companyName = "波导手机";
        String description = "波导手机，手机中的战斗机";
        int ordered = 1;


        //封装对象
          Brand brand = new Brand();
          brand.setStatus(status);
          brand.setBrandName(brandName);
          brand.setCompanyName(companyName);
          brand.setDescription(description);
          brand.setOrdered(ordered);

        //1.加载mybatis的核心配置文件，获取SqlSessionFactory对象
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);

        //2.获取SqlSession对象
        SqlSession sqlSession = sqlSessionFactory.openSession();

        BrandMapper brandMapper = sqlSession.getMapper(BrandMapper.class);
        brandMapper.add(brand);
       
        //提交事务
        sqlSession.commit();

         sqlSession.close();
    }
```

##### 结果对比：

![image-20230805161111448](C:\Users\黄\AppData\Roaming\Typora\typora-user-images\image-20230805161111448.png)

![image-20230805161133916](C:\Users\黄\AppData\Roaming\Typora\typora-user-images\image-20230805161133916.png)



#### 关于mybatis的事务：

##### 我们在开启openSession是默认为false，就要手动提交事务，所以我们可以选择自动提交事务，也就是将openSession设置为true

```java
  @Test
    public void testAdd() throws IOException {
        //获取status为1的,brandName叫华为，companyName叫华为的用户信息，现在设置为静态的，后面再改成动态的
        int status = 1;
        String brandName = "波导手机";
        String companyName = "波导手机";
        String description = "波导手机，手机中的战斗机";
        int ordered = 1;


        //封装对象
          Brand brand = new Brand();
          brand.setStatus(status);
          brand.setBrandName(brandName);
          brand.setCompanyName(companyName);
          brand.setDescription(description);
          brand.setOrdered(ordered);

        //1.加载mybatis的核心配置文件，获取SqlSessionFactory对象
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);

        //2.获取SqlSession对象
        SqlSession sqlSession = sqlSessionFactory.openSession(true); //true设置自动提交事务

        BrandMapper brandMapper = sqlSession.getMapper(BrandMapper.class);
        brandMapper.add(brand);
       
       

         sqlSession.close();
    }
```



#### 添加接口-主键返回

![image-20230805161710973](C:\Users\黄\AppData\Roaming\Typora\typora-user-images\image-20230805161710973.png)

##### 这里我们获取id值

```java
  @Test
    public void testAdd2() throws IOException {
        //获取status为1的,brandName叫华为，companyName叫华为的用户信息，现在设置为静态的，后面再改成动态的
        int status = 1;
        String brandName = "波导手机";
        String companyName = "波导手机";
        String description = "波导手机，手机中的战斗机";
        int ordered = 1;


        //封装对象
          Brand brand = new Brand();
          brand.setStatus(status);
          brand.setBrandName(brandName);
          brand.setCompanyName(companyName);
          brand.setDescription(description);
          brand.setOrdered(ordered);

        //1.加载mybatis的核心配置文件，获取SqlSessionFactory对象
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);

        //2.获取SqlSession对象
        SqlSession sqlSession = sqlSessionFactory.openSession(true); //true设置自动提交事务

        BrandMapper brandMapper = sqlSession.getMapper(BrandMapper.class);
        brandMapper.add(brand);
        Integer id = brand.getId();
        System.out.println(id);
       

         sqlSession.close();
    }
```

##### 这里的测试结果id的值为null，这是为什么捏？

##### 因为这里没有指定数据库表的id的值进行获取，导致该处获取的id值为null。

##### 以后在添加关联数据的时候会出现问题，比如订单和订单项数，一个订单对应多个订单项数，也就是订单项的外键指向订单的主键。在添加订单时，有多个订单项也要同时添加到数据库表中，在添加订单项时要指定orderId的值，而orderId的值在添加完成后是获取不到，所以订单项在添加时就无法指定orderId的值，也就无法添加成功

##### 如何解决：我们的insert语句在执行时id的就已经存在了，只不过没有绑定到对象上，所以这里我们可以设置useGeneratedKey=true，并且同时设置属性keyProperty="id"指向对象的主键的名称，绑定对象的id和数据库主键值，就可以将id的值拿出来设置到属性上，再从getId获取id值

```xml
  <insert id="add" useGeneratedKeys="true" keyProperty="id">
        insert into tb_brand(brand_name,company_name,ordered,description,status) values(#{brandName},#{companyName},#{ordered},#{description},#{status});
    </insert>
```

![image-20230805230412175](C:\Users\黄\AppData\Roaming\Typora\typora-user-images\image-20230805230412175.png)



### 修改功能：

#### 修改全部字段：

![image-20230805230846323](C:\Users\黄\AppData\Roaming\Typora\typora-user-images\image-20230805230846323.png)

##### 1.接口：

```java
 /*
    * 修改操作
    * */
    int update(Brand brand);
```

##### 2.编写sql语句：

```xml
  <update id="update">
        update tb_brand set brand_name = #{brandName},company_name = #{companyName},ordered = #{ordered},description = #{description},status = #{status} where id = #{id};
    </update>
```

##### 3.测试：

```java
      @Test
    public void testUpdate() throws IOException {
        //获取status为1的,brandName叫华为，companyName叫华为的用户信息，现在设置为静态的，后面再改成动态的
        int status = 1;
        String brandName = "miku";
        String companyName = "miku-company";
        String description = "喜欢miku都应该喜欢fufu";
        int ordered = 200;
        int id = 5;


        //封装对象
          Brand brand = new Brand();
          brand.setStatus(status);
          brand.setBrandName(brandName);
          brand.setCompanyName(companyName);
          brand.setDescription(description);
          brand.setOrdered(ordered);
          brand.setId(id);

        //1.加载mybatis的核心配置文件，获取SqlSessionFactory对象
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);

        //2.获取SqlSession对象
        SqlSession sqlSession = sqlSessionFactory.openSession();

        BrandMapper brandMapper = sqlSession.getMapper(BrandMapper.class);
        int update = brandMapper.update(brand);
        System.out.println(update);

           //提交事务
        sqlSession.commit();

         sqlSession.close();
    }
```

![image-20230805232445410](C:\Users\黄\AppData\Roaming\Typora\typora-user-images\image-20230805232445410.png)

##### 测试结果返回为1，则是修改成功了。



#### 修改动态sql：

##### 上面测试必须要对所有字段传入相对应的值，如果没有设置值，就会默认为null值传入数据库。所以我们对修改部分字段采用动态sql

![image-20230805232602348](C:\Users\黄\AppData\Roaming\Typora\typora-user-images\image-20230805232602348.png)

##### 1.接口：

```java
  int update2(Brand brand);
```



##### 2.编写sql：

```xml
 
    <update id="update2">
        update tb_brand
        <set><!--这里使用set标签为了解决sql拼接的问题，跟上面的where标签作用一样-->
            <if test="brandName != null and brandName != ''">
                brand_name = #{brandName},
            </if>
            <if test="companyName != null and companyName != ''">
                company_name = #{companyName},
            </if>
            <if test="ordered != null">
                ordered = #{ordered},
            </if>
            <if test="description != null and description != ''">
                description = #{description},
            </if>
            <if test="status != null">
                status = #{status}
            </if>
        </set>
        where id = #{id};
    </update>
```



##### 3.测试：

```java
 @Test
    public void testUpdate2() throws IOException {
        //获取status为1的,brandName叫华为，companyName叫华为的用户信息，现在设置为静态的，后面再改成动态的
        int status = 1;
        String brandName = "kagamine";
        String companyName = "miku-company";
        String description = "喜欢kagamine";
        int ordered = 200;
        int id = 4;


        //封装对象
          Brand brand = new Brand();
          brand.setStatus(status);
          brand.setBrandName(brandName);
          brand.setCompanyName(companyName);
          brand.setDescription(description);
          brand.setOrdered(ordered);
          brand.setId(id);

        //1.加载mybatis的核心配置文件，获取SqlSessionFactory对象
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);

        //2.获取SqlSession对象
        SqlSession sqlSession = sqlSessionFactory.openSession();

        BrandMapper brandMapper = sqlSession.getMapper(BrandMapper.class);
        int update = brandMapper.update(brand);
        System.out.println(update);

           //提交事务
        sqlSession.commit();

         sqlSession.close();
    }
```



### 删除功能：

#### 1.删除一个：

![image-20230808120818306](C:\Users\黄\AppData\Roaming\Typora\typora-user-images\image-20230808120818306.png)

##### 1.接口：

```java
 /*
    * 删除功能
    * */
    int deleteById(int id);
```



##### 2.编写sql：

```xml
 <delete id="deleteById">
        delete from tb_brand where id = #{id};
    </delete>
```



##### 3.测试：

数据库最初的结果

![image-20230808123933551](C:\Users\黄\AppData\Roaming\Typora\typora-user-images\image-20230808123933551.png)

```java
  @Test
    public void testDeleteById() throws IOException {
        //获取status为1的,brandName叫华为，companyName叫华为的用户信息，现在设置为静态的，后面再改成动态的

        int id = 6;

        //1.加载mybatis的核心配置文件，获取SqlSessionFactory对象
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);

        //2.获取SqlSession对象
        SqlSession sqlSession = sqlSessionFactory.openSession();

     BrandMapper brandMapper = sqlSession.getMapper(BrandMapper.class);
        int delete = brandMapper.deleteById(id);
          System.out.println(delete);
           //提交事务
        sqlSession.commit();

         sqlSession.close();
    }
```

![image-20230808124102787](C:\Users\黄\AppData\Roaming\Typora\typora-user-images\image-20230808124102787.png)

执行结果：update为1

数据库最终结果：

![image-20230808124227430](C:\Users\黄\AppData\Roaming\Typora\typora-user-images\image-20230808124227430.png)

#### 2.批量删除：

![image-20230808122235883](C:\Users\黄\AppData\Roaming\Typora\typora-user-images\image-20230808122235883.png)

#### 1.接口：

##### 这里为实现批量删除，所以在传递参数时是用数组作为参数。

```java
 /*
    * 批量删除
    * */
   void deleteByIds(@Param("ids") int[] ids);
```



#### 2.编写sql：

```xml
  <!--mybatis会将数组参数封装为map集合，map集合有key值和value值 默认情况下key的名称叫array，value对应的数组
    这里就是根据key值来获取数组这里collection="array"
    也可以通过@Param注解来改变map集合默认key的名称-->
    <!--separator这里是分割符，因为foreach在遍历id时，id作为参数要进行sql语句的拼接-->
    <delete id="deleteByIds">
        delete from tb_brand where id in
        <foreach collection="ids" item="id" separator=",">
            #{id}
        </foreach>
    </delete>
```



#### 3.测试：

数据库结果：

![image-20230808124227430](C:\Users\黄\AppData\Roaming\Typora\typora-user-images\image-20230808124227430.png)

```java
        @Test
    public void testDeleteByIds() throws IOException {
        //获取status为1的,brandName叫华为，companyName叫华为的用户信息，现在设置为静态的，后面再改成动态的

        int[] ids = {7,8};

        //1.加载mybatis的核心配置文件，获取SqlSessionFactory对象
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);

        //2.获取SqlSession对象
        SqlSession sqlSession = sqlSessionFactory.openSession();

     BrandMapper brandMapper = sqlSession.getMapper(BrandMapper.class);
         brandMapper.deleteByIds(ids);
          System.out.println(delete);
           //提交事务
        sqlSession.commit();

         sqlSession.close();
    }

```

![image-20230808125500543](C:\Users\黄\AppData\Roaming\Typora\typora-user-images\image-20230808125500543.png)

数据库结果：
![image-20230808125539216](C:\Users\黄\AppData\Roaming\Typora\typora-user-images\image-20230808125539216.png)



### mybatis参数传递：

![image-20230808222247307](C:\Users\黄\AppData\Roaming\Typora\typora-user-images\image-20230808222247307.png)

### @param注解

#### 举例说明：

#### 1.接口：

```java
  /*
    * 参数传递：
    * 1.POJO类型类似User，Brand类
    * 2.Map集合
    * 3.Collection集合
    * 4.List集合
    * 5.Array数组
    * 6.多个参数，使用@Param注解
    *
    * */
    User select(@Param("username") String username,@Param("password") String password);
```



#### 2.编写sql：

```xml
 <select id="select" resultType="User">
    select * from tb_user where username = #{username} and password = #{password};
  </select>
```



#### 3.测试：

```java
  @Test
    public void selectTest() throws IOException {
        String username= "zhangsan";
        String password = "123";

         //1.加载mybatis的核心配置文件，获取SqlSessionFactory对象
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);

        //2.获取SqlSession对象
        SqlSession sqlSession = sqlSessionFactory.openSession();

        //3.执行sql语句;
        UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
        User user = userMapper.select(username,password);
        System.out.println(user);

        sqlSession.close();
    }
```

#### 参数前加@Param注解的原因：

##### mybatis提供了paramNameResolver类来封装参数



#### 多个参数是如何封装的：

##### 如果mybatis发现是多个参数的话，会将多个参数封装成map集合，同时会将多个参数作为value值，而map集合存在键值

```xml
 <!--传入多个参数是，mybatis会将多个参数封装为map集合
  多个参数会作为map集合的value值，而此时键值是默认的
  map.put("arg0",参数值1)
  map.put("param1",参数值1)
  map.put("arg1",参数值2)
  map.put("param2",参数值2)-->
  <select id="select" resultType="User">
    select * from tb_user where username = #{username} and password = #{password};
  </select>
```

##### 我们先不用@Param注解，观察报错：

![image-20230809091455825](C:\Users\黄\AppData\Roaming\Typora\typora-user-images\image-20230809091455825.png)

##### 这里就可以看到可以用的键值，将sql里面的参数改成这里的键值

```xml
 <select id="select" resultType="User">
    select * from tb_user where username = #{arg0} and password = #{arg1};
  </select>
```

![image-20230809091727840](C:\Users\黄\AppData\Roaming\Typora\typora-user-images\image-20230809091727840.png)

##### 结果是可行的。

##### 所以@Param直接是来替换创建的map集合的键值

```java
 /*
    * 参数传递：
    * 1.POJO类型类似User，Brand类:直接作为参数使用，保证属性名和参数占位符一致
    * 2.Map集合:保证key和参数占位符一致
    * 3.Collection集合:封装为一个map集合，使用@Param注解，替换arg0的键名
    * map.put("arg0",collection集合);
    * map.put("collection",collection集合);
    * 4.List集合: 封装为一个map集合，使用@Param注解，替换arg0的键名
    * map.put("arg0",list集合);
    * map.put("collection",list集合);
    * map.put("list",list集合);
    * 5.Array数组: 封装为一个map集合，使用@Param注解，替换arg0的键名
    * map.put("arg0",array数组);
    * map.put("array",array数组);
```

![image-20230809094233644](C:\Users\黄\AppData\Roaming\Typora\typora-user-images\image-20230809094233644.png)

### 注解开发：

![image-20230809101614054](C:\Users\黄\AppData\Roaming\Typora\typora-user-images\image-20230809101614054.png)
