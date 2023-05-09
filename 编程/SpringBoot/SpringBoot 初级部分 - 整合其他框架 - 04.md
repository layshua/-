### 文章目录

*   [SpringBoot - 整合其他框架 - 04](#SpringBoot04_1)
*   *   [1.SpringBoot 整合 Junit](#1SpringBootJunit_3)
    *   [2.SpringBoot 整合 Redis](#2SpringBootRedis_49)
    *   [3.SpringBoot 整洁 MyBatis](#3SpringBootMyBatis_97)

# SpringBoot - 整合其他[框架](https://so.csdn.net/so/search?q=%E6%A1%86%E6%9E%B6&spm=1001.2101.3001.7020) -04

该文章参考：[黑马 SpringBoot](https://www.bilibili.com/video/BV1Lq4y1J77x?spm_id_from=333.999.0.0)

## 1.SpringBoot 整合 [Junit](https://so.csdn.net/so/search?q=Junit&spm=1001.2101.3001.7020)

1.  搭建 SpringBoot 工程
    
2.  引入 starter-test 起步依赖
    
    ```
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
    ```
    
3.  编写测试类
    
    ```
    @Service
    public class UserService {
        public void add(){
            System.out.println("add...");
        }
    }
    ```
    
4.  添加测试相关注解 `@SpringBootTest`
    
5.  编写测试方法
    
    ```
    @SpringBootTest(classes = SpringbootTestApplication.class)
    //这里加不加classes取决于当前测试类包所在的位置
    //在引导类所在包的子包或同级则不需要加(会自动找引导类)，否则要加
    class SpringbootRedisApplicationTests {
    
        @Autowired
        UserService userService;
        @Test
        void contextLoads() {
            userService.add();
        }
    }
    ```
    

## 2.SpringBoot 整合 Redis

1.  搭建 SpringBoot 工程
    
2.  引入 redis 起步依赖
    
    ```
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-redis</artifactId>
    </dependency>
    ```
    
3.  配置 redis 相关属性
    
    ```
    spring:
     redis:
     host: 127.0.0.1 #redis的主机ip
     port: 6379
    ```
    
4.  注入`RedisTemplate`模板
    
5.  编写测试方法，测试（记得打开本机的 redis）
    
    ```
    @SpringBootTest
    class SpringbootRedisApplicationTests {
    
        @Autowired
        private RedisTemplate redisTemplate;
    
        @Test
        public void testSet(){
            //存数剧
            redisTemplate.boundValueOps("name").set("zhangsan");
        }
        @Test void testGet(){
            //获取数据
            Object name = redisTemplate.boundValueOps("name").get();
            System.out.println(name);
        }
    }
    ```
    

## 3.SpringBoot 整洁 [MyBatis](https://so.csdn.net/so/search?q=MyBatis&spm=1001.2101.3001.7020)

1.  搭建 SpringBoot 工程
    
2.  引入 mybatis 起步依赖，添加 mysql 驱动
    
    ```
    <dependency>
        <groupId>org.mybatis.spring.boot</groupId>
        <artifactId>mybatis-spring-boot-starter</artifactId>
        <version>2.2.0</version>
    </dependency>
    
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <scope>runtime</scope>
    </dependency>
    ```
    
3.  编写 DataSource 和 MyBatis 相关配置
    
    DataSource 配置信息：
    
    ```
    spring:
     datasource:
     url: jdbc:mysql:///mzz
     username: root
     password: root
     driver-class-name: com.mysql.jdbc.Driver
    ```
    
    用注解开发可以不用写 MyBatis 的配置
    
    xml 开发 MyBatis 相关配置：
    
    ```
    mybatis:
     mapper-locations: classpath:mapper/*Mapper.xml #mapper映射文件路径
     type-aliases-package: com.itheima.springbootmybatis.domain #配置别名
    #config-location: 指定mybatis的核心配置文件
    ```
    
4.  定义表和实体类
    
    ```
    USE `mzz`;
    
    DROP TABLE IF EXISTS `t_user`;
    
    CREATE TABLE `t_user` (
      `id` int(11) NOT NULL AUTO_INCREMENT,
      `username` varchar(32) COLLATE utf8mb4_unicode_ci DEFAULT NULL,
      `password` varchar(32) COLLATE utf8mb4_unicode_ci DEFAULT NULL,
      PRIMARY KEY (`id`)
    ) ENGINE=InnoDB AUTO_INCREMENT=3 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
    
    insert  into `t_user`(`id`,`username`,`password`) values (1,'zhangsan','123'),(2,'lisi','234');
    ```
    
    ![](<images/1683635603970.png>)
    
    ```
    @Data
    @ToString
    @AllArgsConstructor
    @NoArgsConstructor
    public class User {
        private Integer id;
        private String username;
        private String password;
    }
    ```
    
5.  编写 dao 和 mapper 文件 / 纯注解开发 、
    
    xml 开发：
    
    ```
    @Mapper
    @Repository
    public interface UserXmlMapper {
    
        public List<User> findAll();
    }
    ```
    
    ```
    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="com.itheima.springbootmybatis.mapper.UserXmlMapper">
        <select id="findAll" resultType="user">
            select * from t_user
        </select>
    </mapper>
    ```
    
    纯注解开发：
    
    ```
    @Mapper
    @Repository//这里可加可不加，mybatis提供@Mapper可以代替
    public interface UserMapper {
        @Select("select * from t_user")
        public List<User> findAll();
    }
    ```
    
6.  测试
    
    ```
    @SpringBootTest
    class SpringbootMybatisApplicationTests {
    
        @Autowired
        private UserMapper userMapper;
    
        @Autowired
        private UserXmlMapper userXmlMapper;
    
        @Test
        void testFindAll() {
            List<User> res = userMapper.findAll();
            for (User user : res) {
                System.out.println(user);
            }
        }
    
        @Test
        void testFindAll1() {
            List<User> res = userXmlMapper.findAll();
            for (User user : res) {
                System.out.println(user);
            }
        }
    }
    ```
    
    最后喜欢的小伙伴别忘了一键三连哦🎈🎈🎈  
    
    ![](<images/1683635604031.png>)