## Mybatis Quick Fast 

#### 1、IDEA创建Maven工程

1. New Project->Maven->Create from archetype->org.apache.maven.archetypes:maven-archetype-quickstart

2. 在pom.xml中写入下列依赖

   ```xml
       <dependency>
         <groupId>mysql</groupId>
         <artifactId>mysql-connector-java</artifactId>
         <version>8.0.13</version>
       </dependency>
   
       <dependency>
         <groupId>org.mybatis</groupId>
         <artifactId>mybatis</artifactId>
         <version>3.4.6</version>
       </dependency>
   
       <dependency>
         <groupId>org.projectlombok</groupId>
         <artifactId>lombok</artifactId>
         <version>1.16.20</version>
       </dependency>
   ```

   

#### 2、从 XML 中构建 SqlSessionFactory

1. 新建resources文件夹，并将其设为Sources Root
2. 编写mybatis核心配置文件，新建.xml文件，命名为mybatis-cfg.xml，输入以下内容
3. 数据源改为自己的

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/database_name?serverTimezone=GMT"/>
                <property name="username" value="username"/>
                <property name="password" value="password"/>
            </dataSource>
        </environment>
    </environments>
    <mappers>
        <mapper resource="mapper/UserMapper.xml"/>
    </mappers>
</configuration>
```



#### 3、从 SqlSessionFactory 中获取 SqlSession

1. 在com.xxx下新建util文件夹并创建类MybatisUtil

2. resource的路径要正确 

3. 调用MybatisUtil.getSession()即可获得SqlSession

   ```java
   public class MybatisUtil {
       public static SqlSessionFactory getSqlSqlSessionFactory() throws IOException {
           String resource = "mybatis-cfg.xml";
           InputStream inputStream = Resources.getResourceAsStream(resource);
           SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
           return sqlSessionFactory;
       }
   
       public static SqlSession getSession() throws IOException{
           SqlSessionFactory sqlSessionFactory = getSqlSqlSessionFactory();
           return sqlSessionFactory.openSession();
       }
   }
   ```



#### 4、创建实体类

1. 在com.xxx下新建文件夹entity，并新建use.User实体类

```java
@Data
public class User {
    private int id;
    private String name;
    private String pwd;
}
```

#### 5、编写SQL语句的映射文件

1. 在resoureces文件新建mapper文件夹，新建UserMapper.xml文件并输入

   ```xml
   <?xml version="1.0" encoding="UTF-8" ?>
   <!DOCTYPE mapper
           PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
           "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
   <mapper namespace="com.jia.mapper.UserMapper">
       <select id="selectUser" resultType="com.jia.entity.user.User">
       select * from user where id = #{id}
     </select>
   </mapper>
   ```

   

   

#### 6、测试

1. 在main方法中写入：

   ```java
   public class App {
       public static void main( String[] args ) throws IOException {
           SqlSession sqlSession = MybatisUtil.getSession();
           User user = sqlSession.selectOne("com.jia.mapper.user.UserMapper.selectUser",1);
           System.out.println("id=" + user.getId() + "," +  "name=" + user.getName());
           sqlSession.close();
       }
   }
   ```

   

2. 如果数据库中有user表，表中有id为1、name为zhangsan的数据，则得到结果为

   ```
   id=1,name=zhangsan
   ```

   