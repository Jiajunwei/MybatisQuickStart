## 基本的crud操作

#### 1、搭建mybatis框架

- 参考Mybatis Quick Fast

#### 2、编写实体类（entity.xxx.Xxx，与数据库保持一致（提前创建好数据库并））

- 创建entity.user.User类

```java
@Data	//使用lombok
public class User {
    private int id;
    private String name;
    private String pwd;
}
```



#### 3、编写mapper映射文件（mapper.xxx.Xxx.xml）

- 在resources中创建mapper.user.UserMapper.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper .dtd">
<mapper namespace="mapper.user.UserMapper">
    <!-- 查询单个用户 -->
    <select id="selectUser" resultType="com.jia.entity.user.User">
      select * from user where id = #{id}
    </select>

    <!-- 查询所有用户 -->
    <select id="selectAllUser" resultType="com.jia.entity.user.User">
      select * from user
    </select>

    <!-- 添加用户 -->
    <!--返回值是int类型，不用写-->
    <!--useGeneratedKeys，使用自增，数据库主键要设置为自增-->
    <insert id="addUser" parameterType="com.jia.entity.user.User" useGeneratedKeys="true">
      insert into user (name, pwd) values(#{name}, #{pwd})
    </insert>

    <!-- 更新用户 -->
    <update id="updateUser" parameterType="com.jia.entity.user.User">
      update user set name = #{name}, pwd = #{pwd} where id = #{id}
    </update>

    <!-- 删除用户 -->
    <delete id="deleteUser" parameterType="java.lang.Integer">
        delete from user where id = #{id}
    </delete>
</mapper>
```



#### 4、编写核心配置文件（配置数据库连接的相关信息以及配置mapper映射文件）

- 要想使用某个mapper映射文件，需要将其加入到核心配置文件的<mappers> </mappers>中，参考下列代码

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
                <property name="url" value="jdbc:mysql://localhost:3306/shop?serverTimezone=GMT"/>
                <property name="username" value="root"/>
                <property name="password" value="root"/>
            </dataSource>
        </environment>
    </environments>
    <mappers>
        <mapper resource="mapper/user/UserMapper.xml"/>
        <mapper resource="mapper/user/UserMapperTestNameSpace.xml"/>
        <mapper resource="mapper/shop/BookPubMapper.xml"/>
    </mappers>
</configuration>
```



#### 5、编写MybatisUtil类以方便获取 SqlSession

- resource的值为上述的核心配置文件的路径

```java
package com.jia.util;

import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;

import java.io.IOException;
import java.io.InputStream;

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



#### 6、编写dao操作（dao.xxx.Xxx）

- sqlSession.selectOne()中，第一个参数是mapper映射文件中的namespace名+id名 ，比如mapper.user.UserMapper.selectUser，namespace名为mapper.user.UserMapper，id名为selectUser；第二个参数是一个对象。
- 关于namespace名，需要注意的是该mapper映射文件一定要加入到了核心配置文件的<mappers></mappers>中
- 其命名可以是随意的，但是要保持其在唯一性，一般为了方便寻找，我们对其命名与实体类名字相关联，比如mapper.user.UserMapper

```java
package com.jia.dao;

import com.jia.entity.user.User;
import com.jia.util.MybatisUtil;
import org.apache.ibatis.session.SqlSession;

import java.io.IOException;
import java.util.List;

public class UserDao {
    public User getById(int id) throws IOException {
        SqlSession sqlSession = MybatisUtil.getSession();
        User user = sqlSession.selectOne("mapper.user.UserMapper.selectUser",id);
        sqlSession.close();
        return user;
    }

    public List<User> getAllUser() throws IOException {
        SqlSession sqlSession = MybatisUtil.getSession();
        List<User> userList = sqlSession.selectList("mapper.user.UserMapper.selectAllUser");
        sqlSession.close();
        return userList;
    }

    public int addUser(User user) throws IOException {
        SqlSession sqlSession = MybatisUtil.getSession();
        int result = sqlSession.insert("mapper.user.UserMapper.addUser",user);
        sqlSession.commit();//手动提交：插入语句是一个事务，在jdbc中是默认提交，在这里需要手动提交
        sqlSession.close();
        return result;
    }

    public int updateUser(User user) throws IOException {
        SqlSession sqlSession = MybatisUtil.getSession();
//        //这里也可以用insert，因为inset、update底层实现都是update
//        int result = sqlSession.insert("testNameSpace.updateUser",user);
        int result = sqlSession.update("mapper.user.UserMapper.updateUser",user);
        sqlSession.commit();
        sqlSession.close();
        return result;
    }

    public int deleteUser(int id) throws IOException {
        SqlSession sqlSession = MybatisUtil.getSession();
        int result = sqlSession.delete("mapper.user.UserMapper.deleteUser",id);
        sqlSession.commit();
        sqlSession.close();
        return result;
    }
}
```



#### 7、测试

```java
//查询单个用户
UserDao userDao = new UserDao();
System.out.println( userDao.getById(1));

//添加用户
UserDao userDao = new UserDao();
User user = new User();
user.setName("LiSi");
user.setPwd("123");
System.out.println(userDao.addUser(user));

//更新用户信息
UserDao userDao = new UserDao();
User user =userDao.getById(3);
user.setPwd("33333");
System.out.println(userDao.updateUser(user));

//删除用户
UserDao userDao = new UserDao();
System.out.println(userDao.deleteUser(4));

//查询所有用户
UserDao userDao = new UserDao();
List<User> users = userDao.getAllUser();
for (User u : users){
    System.out.println(u);
}
```



#### 8、mapper代理方法

- 为什么要用mapper代理方法

1. 在com.xxx目录下创建mapper接口（mapper.xxx.XxxMapper）

   - mapper.user.UserMapper

   ```java
   public interface UserMapper {
       //查询单个用户的接口
       User getUser(Integer id);
   
       //查询所有用户的接口
       List<User> getAllUser();
   
       //添加用户的接口
       int addUser(User user);
   
       //更新用户信息的接口
       int updateUser(User user);
   
       //删除用户的接口
       int deleteUser(int id);
   }
   ```

   

2. 创建mapper映射文件

   - namespace必须要指向上述mapper接口，且id需要与接口中的函数名一一对应

   ```xml
   <?xml version="1.0" encoding="UTF-8" ?>
   <!DOCTYPE mapper
           PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
           "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
   <mapper namespace="com.jia.mapper.user.UserMapper">
       <!-- 查询单个用户 -->
       <select id="getUser" resultType="com.jia.entity.user.User">
         select * from user where id = #{id}
       </select>
   
       <!-- 查询所有用户 -->
       <select id="getAllUser" resultType="com.jia.entity.user.User">
         select * from user
       </select>
   
       <!-- 添加用户 -->
       <!--返回值是int类型，不用写-->
       <!--useGeneratedKeys，使用自增，数据库主键要设置为自增-->
       <insert id="addUser" parameterType="com.jia.entity.user.User" useGeneratedKeys="true">
         insert into user (name, pwd) values(#{name}, #{pwd})
       </insert>
   
       <!-- 更新用户 -->
       <update id="updateUser" parameterType="com.jia.entity.user.User">
         update user set name = #{name}, pwd = #{pwd} where id = #{id}
       </update>
   
       <!-- 删除用户 -->
       <delete id="deleteUser" parameterType="java.lang.Integer">
           delete from user where id = #{id}
       </delete>
   </mapper>
   ```

   

3. 测试

```java
 //mapper代理方法
SqlSession sqlSession = MybatisUtil.getSession();
UserMapper mapper = sqlSession.getMapper(UserMapper.class);

//查询单个用户
User user = mapper.getUser(1);
System.out.println(user);

//查询所有用户
List<User> userList = mapper.getAllUser();
for (User u : userList){
    System.out.println(u);
}

//添加用户
User user = new User();
user.setName("LiSi");
user.setPwd("123");
mapper.addUser(user);

//更新用户信息
User user =mapper.getUser(3);
user.setPwd("33333");
mapper.updateUser(3);

//删除用户
mapper.deleteUser(3);

//关闭session
sqlSession.close();
```

