**根据 B 站 up 主 MarkerHub 视频制作的一个笔记**

[**我的博客**](http://linbookz.com/blogView/blog/6)

**B 站博主链接：**  
[https://www.bilibili.com/video/BV1PQ4y1P7hZ?p=1](https://www.bilibili.com/video/BV1PQ4y1P7hZ?p=1)

**博主的开发文档:**  
[https://juejin.cn/post/6844903823966732302](https://juejin.cn/post/6844903823966732302)  
**部署视频**  
[https://www.bilibili.com/video/BV17A411E7aE?p=1](https://www.bilibili.com/video/BV17A411E7aE?p=1)  
**up 主的部署文档**  
[https://juejin.im/post/6886061338804617229/](https://juejin.im/post/6886061338804617229/)

**开源项目源码 :**  
up 主的：  
[https://github.com/MarkerHub/vueblog](https://github.com/MarkerHub/vueblog)  
我的：  
[https://gitee.com/hntianshu/vueblog](https://gitee.com/hntianshu/vueblog)

**热部署配置:**  
[https://www.cnblogs.com/erlongxizhu-03/p/12193646.html](https://www.cnblogs.com/erlongxizhu-03/p/12193646.html)

**idea 报 Could not autowired 解决办法:**  
[https://blog.csdn.net/yxm234786/article/details/81460752](https://blog.csdn.net/yxm234786/article/details/81460752)

**ElementUI 官方文档**  
[https://element.eleme.cn/#/zh-CN/component/quickstart](https://element.eleme.cn/#/zh-CN/component/quickstart)

**PostMan 安装包下载**  
[https://blog.csdn.net/weixin_43184774/article/details/100578557](https://blog.csdn.net/weixin_43184774/article/details/100578557)

**VsCode 保存自动格式化样式**  
[https://blog.csdn.net/wang0112233/article/details/90608328](https://blog.csdn.net/wang0112233/article/details/90608328)

**Redis 的安装和启动**  
[https://www.cnblogs.com/pretty-sunshine/p/10615287.html](https://www.cnblogs.com/pretty-sunshine/p/10615287.html)

**搞定 Shiro 集成 redis 实现会话共享**  
[https://blog.csdn.net/m0_46995061/article/details/106751848](https://blog.csdn.net/m0_46995061/article/details/106751848)

**nginx 官网**  
[http://nginx.org/en/download.html](http://nginx.org/en/download.html)

### 文章目录

*   [简易博客项目 (springboot+jwt+shiro+vue+elementUI+axios+redis+mysql)](#springbootjwtshirovueelementUIaxiosredismysql_45)
*   *   [第一章 整合新建 springboot, 整合 mybatisplus](#_springbootmybatisplus_47)
    *   *   [第一步 创建项目 (第八步骤就行)+ 数据库:](#__48)
        *   [配置分页 MybatisPlusConfig + 生成代码 (dao 、service、serviceImpl 等)](#MybatisPlusConfigdao_serviceserviceImpl_213)
        *   [第三步 做测试](#__361)
    *   [第二章 统一结果封装](#__403)
    *   [第三章 Shiro 整合 jwt 逻辑分析](#_Shirojwt_505)
    *   [第四章 Shiro 逻辑开发](#_Shiro_906)
    *   [第五章 异常处理](#__990)
    *   [第六章 实体校验](#__1037)
    *   [第七章 跨域问题](#__1129)
    *   [第七章 登录接口开发](#__1191)
    *   [第八章 博客接口的开发](#__1309)
    *   [后端总结](#_1464)
    *   [第九章 Vue 前端页面开发](#_Vue_1466)
    *   *   [前言](#_1467)
        *   [环境准备](#_1476)
        *   [新建项目](#_1497)
        *   [项目的架构](#_1521)
        *   [vscode 前端常用插件推荐，搭建 JQuery、Vue 等开发环境](#vscodeJQueryVue_1572)
        *   *   [** 使用 vscode 打开项目并且运行 **](#vscode_1576)
        *   [安装 element-ui](#elementui_1585)
        *   [补充配置：](#_1675)
        *   [安装 axios](#axios_1682)
        *   [页面路由](#_1703)
        *   [路由中心配置](#_1722)
        *   [登录页面开发](#_1779)
        *   *   [登录页面制作](#_1780)
            *   [配置 store](#store_1872)
            *   [登录发起请求 (配置完善 store)](#_store_1955)
            *   [** 配置 axios 拦截 **](#axios_2152)
            *   [公共组件 Header](#Header_2224)
        *   [博客编辑（发表）](#_2418)
        *   *   [安装 mavon-editor](#mavoneditor_2419)
        *   [博客详情和编辑按钮的权限](#_2594)
        *   [路由权限拦截](#_2679)
    *   [第十章 其他](#__2838)
    *   [增加删除文章功能](#_2839)

# 简易博客项目 (springboot+jwt+[shiro](https://so.csdn.net/so/search?q=shiro&spm=1001.2101.3001.7020)+vue+elementUI+axios+redis+mysql)

## 第一章 整合新建 springboot, 整合 mybatisplus

### 第一步 创建项目 (第八步骤就行)+ 数据库:

[https://blog.csdn.net/weixin_43247803/article/details/113622480](https://blog.csdn.net/weixin_43247803/article/details/113622480)  
**修改 pom.xml 借鉴**

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <!-- lookup parent from repository -->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.2.6.RELEASE</version>
        <relativePath/>
    </parent>
    <groupId>com.vueblog</groupId>
    <artifactId>vueblog</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>vueblog</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <!-- web -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <!-- devtools -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>
        <!-- mysql -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <scope>runtime</scope>
        </dependency>
        <!-- lombok -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <!-- testing -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <!--mybatis-plus-->
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
            <version>3.2.0</version>
        </dependency>
        <!-- framework: mybatis-plus代码生成需要一个模板引擎 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-freemarker</artifactId>
        </dependency>
        <!--mp代码生成器-->
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-generator</artifactId>
            <version>3.2.0</version>
        </dependency>
        <!-- hutool -->
        <dependency>
            <groupId>cn.hutool</groupId>
            <artifactId>hutool-all</artifactId>
            <version>5.3.3</version>
        </dependency>
        <!-- jwt -->
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt</artifactId>
            <version>0.9.1</version>
        </dependency>
        <!-- shiro-redis -->
    <!--    <dependency>
            <groupId>org.crazycake</groupId>
            <artifactId>shiro-redis-spring-boot-starter</artifactId>
            <version>3.2.1</version>
        </dependency>-->
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

**修改配置文件**

```
# DataSource Config
spring:
 datasource:
 driver-class-name: com.mysql.cj.jdbc.Driver
 url: jdbc:mysql://localhost:3306/vueblog?useUnicode=true&useSSL=false&characterEncoding=utf8&serverTimezone=Asia/Shanghai
 username: root
 password: 123456
mybatis-plus:
 mapper-locations: classpath*:/mapper/**Mapper.xml
server:
 port: 8081
```

**创建数据库 vueblog 然后执行下面命令生成表**

```
DROP TABLE IF EXISTS `m_blog`;
/*!40101 SET @saved_cs_client     = @@character_set_client */;
 SET character_set_client = utf8mb4 ;
CREATE TABLE `m_blog` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `user_id` bigint(20) NOT NULL,
  `title` varchar(255) NOT NULL,
  `description` varchar(255) NOT NULL,
  `content` longtext,
  `created` datetime NOT NULL ON UPDATE CURRENT_TIMESTAMP,
  `status` tinyint(4) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=21 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;
/*!40101 SET character_set_client = @saved_cs_client */; 

DROP TABLE IF EXISTS `m_user`;
/*!40101 SET @saved_cs_client     = @@character_set_client */;
 SET character_set_client = utf8mb4 ;
CREATE TABLE `m_user` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `username` varchar(64) DEFAULT NULL,
  `avatar` varchar(255) DEFAULT NULL,
  `email` varchar(64) DEFAULT NULL,
  `password` varchar(64) DEFAULT NULL,
  `status` int(5) NOT NULL,
  `created` datetime DEFAULT NULL,
  `last_login` datetime DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `UK_USERNAME` (`username`) USING BTREE
) ENGINE=InnoDB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8;
/*!40101 SET character_set_client = @saved_cs_client */; 

INSERT INTO `vueblog`.`m_user` (`id`, `username`, `avatar`, `email`, `password`, `status`, `created`, `last_login`) VALUES ('1', 'markerhub', 'https://image-1300566513.cos.ap-guangzhou.myqcloud.com/upload/images/5a9f48118166308daba8b6da7e466aab.jpg', NULL, '96e79218965eb72c92a549dd5a330112', '0', '2020-04-20 10:44:01', NULL);
```

### 配置分页 MybatisPlusConfig + 生成代码 (dao 、service、serviceImpl 等)

**1.0 配置分页**  
**创建 MybatisPlusConfig 类 (创建路径 com/vueblog/config)**

```
package com.vueblog.config;

import com.baomidou.mybatisplus.extension.plugins.PaginationInterceptor;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.transaction.annotation.EnableTransactionManagement;

@Configuration
@EnableTransactionManagement
@MapperScan("com.markerhub.mapper")
public class MybatisPlusConfig {
    @Bean
    public PaginationInterceptor paginationInterceptor() {
        PaginationInterceptor paginationInterceptor = new PaginationInterceptor();
        return paginationInterceptor;
    }
}
```

**2.0 生成代码**  
**创建 CodeGenerator(在 com.vueblog 包下面)**  
** 修改对应的数据库: 账号密码、数据库名、包配置 (ctrl+f 可找到对应位置) **  
**然后运行输入俩表 , 号隔开是多表**

```
package com.vueblog;

import com.baomidou.mybatisplus.core.exceptions.MybatisPlusException;
import com.baomidou.mybatisplus.core.toolkit.StringPool;
import com.baomidou.mybatisplus.core.toolkit.StringUtils;
import com.baomidou.mybatisplus.generator.AutoGenerator;
import com.baomidou.mybatisplus.generator.InjectionConfig;
import com.baomidou.mybatisplus.generator.config.*;
import com.baomidou.mybatisplus.generator.config.po.TableInfo;
import com.baomidou.mybatisplus.generator.config.rules.NamingStrategy;
import com.baomidou.mybatisplus.generator.engine.FreemarkerTemplateEngine;

import java.util.ArrayList;
import java.util.List;
import java.util.Scanner;
// 演示例子，执行 main 方法控制台输入模块表名回车自动生成对应项目目录中
public class CodeGenerator {

    /**
     * <p>
     * 读取控制台内容
     * </p>
     */
    public static String scanner(String tip) {
        Scanner scanner = new Scanner(System.in);
        StringBuilder help = new StringBuilder();
        help.append("请输入" + tip + "：");
        System.out.println(help.toString());
        if (scanner.hasNext()) {
            String ipt = scanner.next();
            if (StringUtils.isNotEmpty(ipt)) {
                return ipt;
            }
        }
        throw new MybatisPlusException("请输入正确的" + tip + "！");
    }

    public static void main(String[] args) {
        // 代码生成器
        AutoGenerator mpg = new AutoGenerator();

        // 全局配置
        GlobalConfig gc = new GlobalConfig();
        String projectPath = System.getProperty("user.dir");
        gc.setOutputDir(projectPath + "/src/main/java");
        // gc.setOutputDir("D:\\test");
        gc.setAuthor("anonymous");
        gc.setOpen(false);
        // gc.setSwagger2(true); 实体属性 Swagger2 注解
        gc.setServiceName("%sService");
        mpg.setGlobalConfig(gc);

        // 数据源配置 数据库名 账号密码
        DataSourceConfig dsc = new DataSourceConfig();
        dsc.setUrl("jdbc:mysql://localhost:3306/vueblog?useUnicode=true&useSSL=false&characterEncoding=utf8&serverTimezone=UTC");
        // dsc.setSchemaName("public");
        dsc.setDriverName("com.mysql.cj.jdbc.Driver");
        dsc.setUsername("root");
        dsc.setPassword("123456");
        mpg.setDataSource(dsc);

        // 包配置
        PackageConfig pc = new PackageConfig();
        pc.setModuleName(null);
        pc.setParent("com.vueblog");
        mpg.setPackageInfo(pc);

        // 自定义配置
        InjectionConfig cfg = new InjectionConfig() {
            @Override
            public void initMap() {
                // to do nothing
            }
        };

        // 如果模板引擎是 freemarker
        String templatePath = "/templates/mapper.xml.ftl";
        // 如果模板引擎是 velocity
        // String templatePath = "/templates/mapper.xml.vm";

        // 自定义输出配置
        List<FileOutConfig> focList = new ArrayList<>();
        // 自定义配置会被优先输出
        focList.add(new FileOutConfig(templatePath) {
            @Override
            public String outputFile(TableInfo tableInfo) {
                // 自定义输出文件名 ， 如果你 Entity 设置了前后缀、此处注意 xml 的名称会跟着发生变化！
                return projectPath + "/src/main/resources/mapper/"
                        + "/" + tableInfo.getEntityName() + "Mapper" + StringPool.DOT_XML;
            }
        });

        cfg.setFileOutConfigList(focList);
        mpg.setCfg(cfg);

        // 配置模板
        TemplateConfig templateConfig = new TemplateConfig();

        templateConfig.setXml(null);
        mpg.setTemplate(templateConfig);

        // 策略配置
        StrategyConfig strategy = new StrategyConfig();
        strategy.setNaming(NamingStrategy.underline_to_camel);
        strategy.setColumnNaming(NamingStrategy.underline_to_camel);
        strategy.setEntityLombokModel(true);
        strategy.setRestControllerStyle(true);
        strategy.setInclude(scanner("表名，多个英文逗号分割").split(","));
        strategy.setControllerMappingHyphenStyle(true);
        strategy.setTablePrefix("m_");
        mpg.setStrategy(strategy);
        mpg.setTemplateEngine(new FreemarkerTemplateEngine());
        mpg.execute();
    }
}
```

**效果如下:**  

![](<images/1683768209108.png>)

### 第三步 做测试

**1.0 UserController 类 (ctrl+R 可全局搜索类)**

```
package com.vueblog.controller;

import com.vueblog.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;

import org.springframework.web.bind.annotation.RestController;

/**
 * <p>
 *  前端控制器
 * </p>
 *
 * @author 深林中的书海
 * @since 2021-02-05
 */
@RestController
@RequestMapping("/user")
public class UserController {
    @Autowired
    private UserService userService;

    @GetMapping("/index")
    public  Object index(){
        return userService.getById(1);
    }

}
```

**2.0 运行项目 查看效果**  

![](<images/1683768209143.png>)

## 第二章 统一结果封装

**这里我们用到了一个 Result 的类，这个用于我们的异步统一返回的结果封装。一般来说，结果里面有几个要素必要的**

*   是否成功，可用 code 表示（如 200 表示成功，400 表示异常）
*   结果消息
*   结果数据

*   **Result 类 (路径 com.vueblog.common.lang;)**

```
package com.vueblog.common.lang;

import lombok.Data;

import java.io.Serializable;

@Data
public class Result implements Serializable { //序列化
    private int code; //200是正常 400表示异常
    private String msg;
    private Object data;//返回数据
    //成功
    public static Result succ( Object data){

        return succ(200,"操作成功",data);
    }
    //成功
    public static Result succ(int code,String msg,Object data){
        Result r = new Result();
        r.setCode(code);
        r.setMsg(msg);
        r.setData(data);
        return r;
    }
    //失败
    public static Result fail(String msg){

        return fail(400,msg,null);
    }
    //失败
    public static Result fail(String msg,Object data){

        return fail(400,msg,data);
    }
    //失败
    public static Result fail(int code,String msg,Object data){
        Result r = new Result();
        r.setCode(code);
        r.setMsg(msg);
        r.setData(data);
        return r;
    }

}
```

*   **在 UserController 中引用测试**

```
package com.vueblog.controller;

import com.vueblog.common.lang.Result;
import com.vueblog.entity.User;
import com.vueblog.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;

import org.springframework.web.bind.annotation.RestController;

/**
 * <p>
 *  前端控制器
 * </p>
 *
 * @author 深林中的书海
 * @since 2021-02-05
 */
@RestController
@RequestMapping("/user")
public class UserController {
    @Autowired
    private UserService userService;

    @GetMapping("/index")
    public  Object index(){
        User user = userService.getById(1);
        return Result.succ(200,"操作成功",user);
    }

}
```

*   **页面运行效果图 (http://localhost:8081/user/index)**

![](<images/1683768209180.png>)

## 第三章 Shiro 整合 jwt 逻辑分析

考虑到后面可能需要做集群、负载均衡等，所以就需要会话共享，而 shiro 的缓存和会话信息，我们一般考虑使用 redis 来存储这些数据，所以，我们不仅仅需要整合 shiro，同时也需要整合 redis。在开源的项目中，我们找到了一个 starter 可以快速整合 shiro-redis，配置简单，这里也推荐大家使用。  
而因为我们需要做的是[前后端分离](https://so.csdn.net/so/search?q=%E5%89%8D%E5%90%8E%E7%AB%AF%E5%88%86%E7%A6%BB&spm=1001.2101.3001.7020)项目的骨架，所以一般我们会采用 token 或者 jwt 作为跨域身份验证解决方案。所以整合 shiro 的过程中，我们需要引入 jwt 的身份验证过程。  
那么我们就开始整合：  
我们使用一个 shiro-redis-spring-boot-starter 的 jar 包，具体教程可以看官方文档：[github.com/alexxiyang/…](https://github.com/alexxiyang/shiro-redis/blob/master/docs/README.md#spring-boot-starter)

*   **导入 shiro-redis 的 starter 包：还有 jwt 的工具包，以及为了简化开发，引入 hutool 工具包。**  
    pom.xml 中导入:

```
<!-- shiro-redis -->
      <dependency>
            <groupId>org.crazycake</groupId>
            <artifactId>shiro-redis-spring-boot-starter</artifactId>
            <version>3.2.1</version>
        </dependency> 
		 <!-- hutool工具类 -->
        <dependency>
            <groupId>cn.hutool</groupId>
            <artifactId>hutool-all</artifactId>
            <version>5.3.3</version>
        </dependency>
            <!-- jwt 生成工具 校验工具-->
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt</artifactId>
            <version>0.9.1</version>
        </dependency>
```

*   **创建 ShiroConfig**  
    **文件路径：com/vueblog/config**

```
package com.vueblog.config;

import com.vueblog.shiro.AccountRealm;
import com.vueblog.shiro.JwtFilter;
import org.apache.shiro.mgt.DefaultSessionStorageEvaluator;
import org.apache.shiro.mgt.DefaultSubjectDAO;
import org.apache.shiro.mgt.SecurityManager;
import org.apache.shiro.session.mgt.SessionManager;
import org.apache.shiro.spring.web.ShiroFilterFactoryBean;
import org.apache.shiro.spring.web.config.DefaultShiroFilterChainDefinition;
import org.apache.shiro.spring.web.config.ShiroFilterChainDefinition;
import org.apache.shiro.web.mgt.DefaultWebSecurityManager;
import org.apache.shiro.web.session.mgt.DefaultWebSessionManager;
import org.crazycake.shiro.RedisCacheManager;
import org.crazycake.shiro.RedisSessionDAO;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import java.util.HashMap;
import java.util.LinkedHashMap;
import java.util.Map;
import javax.servlet.Filter;

/**
 * shiro启用注解拦截控制器
 */
@Configuration
public class ShiroConfig {
    @Autowired
    private JwtFilter jwtFilter;

    @Bean
    public SessionManager sessionManager(RedisSessionDAO redisSessionDAO) {
        DefaultWebSessionManager sessionManager = new DefaultWebSessionManager();
        sessionManager.setSessionDAO(redisSessionDAO);
        return sessionManager;
    }
    @Bean
    public DefaultWebSecurityManager securityManager(AccountRealm accountRealm,
                                                     SessionManager sessionManager,
                                                     RedisCacheManager redisCacheManager) {
        DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager(accountRealm);
        securityManager.setSessionManager(sessionManager);
        securityManager.setCacheManager(redisCacheManager);
        /*
         * 关闭shiro自带的session，详情见文档
         */
        DefaultSubjectDAO subjectDAO = new DefaultSubjectDAO();
        DefaultSessionStorageEvaluator defaultSessionStorageEvaluator = new DefaultSessionStorageEvaluator();
        defaultSessionStorageEvaluator.setSessionStorageEnabled(false);
        subjectDAO.setSessionStorageEvaluator(defaultSessionStorageEvaluator);
        securityManager.setSubjectDAO(subjectDAO);
        return securityManager;
    }
    @Bean
    public ShiroFilterChainDefinition shiroFilterChainDefinition() {
        DefaultShiroFilterChainDefinition chainDefinition = new DefaultShiroFilterChainDefinition();
        Map<String, String> filterMap = new LinkedHashMap<>();
        filterMap.put("/**", "jwt"); // 主要通过注解方式校验权限
        chainDefinition.addPathDefinitions(filterMap);
        return chainDefinition;
    }
    @Bean("shiroFilterFactoryBean")
    public ShiroFilterFactoryBean shiroFilterFactoryBean(SecurityManager securityManager,
                                                         ShiroFilterChainDefinition shiroFilterChainDefinition) {
        ShiroFilterFactoryBean shiroFilter = new ShiroFilterFactoryBean();
        shiroFilter.setSecurityManager(securityManager);

        Map<String, Filter> filters = new HashMap<>();
        filters.put("jwt", jwtFilter);
        shiroFilter.setFilters(filters);

        Map<String, String> filterMap = shiroFilterChainDefinition.getFilterChainMap();

        shiroFilter.setFilterChainDefinitionMap(filterMap);

        return shiroFilter;
    }



}
```

*   **创建 MybatisPlusConfig**

**路径: com.vueblog.config**

```
package com.vueblog.config;

import com.baomidou.mybatisplus.extension.plugins.PaginationInterceptor;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.transaction.annotation.EnableTransactionManagement;

@Configuration
@EnableTransactionManagement
@MapperScan("com.markerhub.mapper")
public class MybatisPlusConfig {
    @Bean
    public PaginationInterceptor paginationInterceptor() {
        PaginationInterceptor paginationInterceptor = new PaginationInterceptor();
        return paginationInterceptor;
    }
}
```

*   **创建 AccountRealm**  
    **路径： com.vueblog.shiro**

```
package com.vueblog.shiro;

import org.apache.shiro.authc.AuthenticationException;
import org.apache.shiro.authc.AuthenticationInfo;
import org.apache.shiro.authc.AuthenticationToken;
import org.apache.shiro.authz.AuthorizationInfo;
import org.apache.shiro.realm.AuthorizingRealm;
import org.apache.shiro.subject.PrincipalCollection;
import org.springframework.stereotype.Component;

@Component
public class AccountRealm extends AuthorizingRealm {
    //为了让realm支持jwt的凭证校验
    @Override
    public boolean supports(AuthenticationToken token) {
        return token instanceof JwtToken;
    }

    //权限校验
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {

        return null;
    }
    //登录认证校验
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
       JwtToken jwtToken = (JwtToken) token;

        return null;
    }
}
```

*   **注意事项如果启动不了在 VueblogApplication 加入 @MapperScan 扫描 mapper**

```
@SpringBootApplication
@MapperScan(basePackages = "com.vueblog.mapper")
public class VueblogApplication {

    public static void main(String[] args) {
        SpringApplication.run(VueblogApplication.class, args);
    }

}
```

*   **创建 JwtFilter**  
    **路径：com.vueblog.shiro;**

```
package com.vueblog.shiro;

import cn.hutool.http.server.HttpServerRequest;
import cn.hutool.json.JSONUtil;
import com.baomidou.mybatisplus.core.toolkit.StringUtils;
import com.vueblog.common.lang.Result;
import com.vueblog.util.JwtUtils;
import io.jsonwebtoken.Claims;
import org.apache.shiro.authc.AuthenticationException;
import org.apache.shiro.authc.AuthenticationToken;
import org.apache.shiro.authc.ExpiredCredentialsException;
import org.apache.shiro.web.filter.authc.AuthenticatingFilter;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

import javax.servlet.ServletRequest;
import javax.servlet.ServletResponse;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

@Component
public class JwtFilter extends AuthenticatingFilter {
    @Autowired
    JwtUtils jwtUtils;
    //验证token
    @Override
    protected AuthenticationToken createToken(ServletRequest servletRequest, ServletResponse servletResponse) throws Exception {
        HttpServletRequest request = (HttpServletRequest)servletRequest;
        //获取头部token
        String jwt = request.getHeader("Authorization");
        if(StringUtils.isEmpty(jwt)){
                return null;
        }
        return new JwtToken(jwt);
    }

    @Override
    protected boolean onAccessDenied(ServletRequest servletRequest, ServletResponse servletResponse) throws Exception {
        HttpServletRequest request = (HttpServletRequest)servletRequest;
        //获取头部token
        String jwt = request.getHeader("Authorization");
        if(StringUtils.isEmpty(jwt)){
            return true;
        }else{
//            校验jwt
            Claims claims = jwtUtils.getClaimByToken(jwt);
            //校验是否为空和时间是否过期
            if(claims == null || jwtUtils.isTokenExpired(claims.getExpiration())){
                throw new ExpiredCredentialsException("token已失效,请重新登录");

            }
            //执行登录
            return executeLogin(servletRequest,servletResponse);
        }
    }
    //捕捉错误重写方法返回Result
    @Override
    protected boolean onLoginFailure(AuthenticationToken token, AuthenticationException e, ServletRequest request, ServletResponse response) {
        HttpServletResponse httpServletResponse = (HttpServletResponse) response;

        Throwable throwable = e.getCause() == null ? e : e.getCause();

        Result result = Result.fail(throwable.getMessage());
        //返回json
        String json = JSONUtil.toJsonStr(result);
        try {
            //打印json
            httpServletResponse.getWriter().print(json);
        }catch (IOException ioException){

        }

        return false;
    }



}
```

*   **创建 JwtToken**

**路径：com.vueblog.shiro**

```
package com.vueblog.shiro;

import org.apache.shiro.authc.AuthenticationToken;

public class JwtToken implements AuthenticationToken {
    private String token;

    public JwtToken(String jwt){
        this.token = jwt;
    }

    @Override
    public Object getPrincipal() {
        return token;
    }

    @Override
    public Object getCredentials() {
        return token;
    }
}
```

*   **创建 JwtUtils**

路径：com.vueblog.util

```
package com.vueblog.util;

import io.jsonwebtoken.Claims;
import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.SignatureAlgorithm;
import lombok.Data;
import lombok.extern.slf4j.Slf4j;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

import java.util.Date;

/**
 * jwt工具类
 */
@Slf4j
@Data
@Component
@ConfigurationProperties(prefix = "markerhub.jwt")
public class JwtUtils {

    private String secret;
    private long expire;
    private String header;

    /**
     * 生成jwt token
     */
    public String generateToken(long userId) {
        Date nowDate = new Date();
        //过期时间
        Date expireDate = new Date(nowDate.getTime() + expire * 1000);

        return Jwts.builder()
                .setHeaderParam("typ", "JWT")
                .setSubject(userId+"")
                .setIssuedAt(nowDate)
                .setExpiration(expireDate)
                .signWith(SignatureAlgorithm.HS512, secret)
                .compact();
    }

    public Claims getClaimByToken(String token) {
        try {
            return Jwts.parser()
                    .setSigningKey(secret)
                    .parseClaimsJws(token)
                    .getBody();
        }catch (Exception e){
            log.debug("validate is token error ", e);
            return null;
        }
    }

    /**
     * token是否过期
     * @return  true：过期
     */
    public boolean isTokenExpired(Date expiration) {
        return expiration.before(new Date());
    }
}
```

*   **创建 spring-devtools.properties**

**路径：resources/WETA-INF/**

```
restart.include.shiro-redis=/shiro-[\\w-\\.]+jar
```

## 第四章 Shiro 逻辑开发

**小提示: 登录调用 AccountRealm 类下面的 doGetAuthenticationInfo**

**创建类 AccountProfile 用于传递数据**  
**路径：com.vueblog.shiro**

```
package com.vueblog.shiro;

import lombok.Data;

import java.io.Serializable;

@Data
public class AccountProfile implements Serializable {
    private Long id;

    private String username;

    private String avatar;

    private String email;

}
```

**完善 AccountRealm**

```
package com.vueblog.shiro;

import cn.hutool.core.bean.BeanUtil;
import com.vueblog.entity.User;
import com.vueblog.service.UserService;
import com.vueblog.util.JwtUtils;
import org.apache.shiro.authc.*;
import org.apache.shiro.authz.AuthorizationInfo;
import org.apache.shiro.realm.AuthorizingRealm;
import org.apache.shiro.subject.PrincipalCollection;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

@Component
public class AccountRealm extends AuthorizingRealm {
    @Autowired
    JwtUtils jwtUtils;
    @Autowired
    UserService userService;
    //为了让realm支持jwt的凭证校验
    @Override
    public boolean supports(AuthenticationToken token) {
        return token instanceof JwtToken;
    }

    //权限校验
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {

        return null;
    }
    //登录认证校验
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
       JwtToken jwtToken = (JwtToken) token;
       //获取userId
        String userId = jwtUtils.getClaimByToken((String) jwtToken.getPrincipal()).getSubject();
       //获取用户内容
        User user = userService.getById(Long.valueOf(userId));
        if(user == null){
            throw new UnknownAccountException("账户不存在");
        }
        if(user.getStatus() == -1){
            throw new LockedAccountException("账户不存在");
        }

        AccountProfile profile = new AccountProfile();
        BeanUtil.copyProperties(user,profile);//将user数据转移到profile
        //用户信息  密钥token 用户名字
        return new SimpleAuthenticationInfo(profile,jwtToken.getCredentials(),getName());
    }
}
```

## 第五章 异常处理

**创建 GlobalExceptionHandler 类**  
捕获全局异常

```
package com.vueblog.common.exception;

import com.vueblog.common.lang.Result;
import lombok.extern.slf4j.Slf4j;
import org.apache.shiro.ShiroException;
import org.springframework.http.HttpStatus;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.ResponseStatus;
import org.springframework.web.bind.annotation.RestControllerAdvice;

/**
 * 日志输出
 * 全局捕获异常
 */
@Slf4j
@RestControllerAdvice
public class GlobalExceptionHandler {
    @ResponseStatus(HttpStatus.UNAUTHORIZED) //因为前后端分离 返回一个状态 一般是401 没有权限
    @ExceptionHandler(value =  ShiroException.class)//捕获运行时异常ShiroException是大部分异常的父类
    public Result handler(ShiroException e){
        log.error("运行时异常：-----------------{}",e);
        return Result.fail(401,e.getMessage(),null);
    }


    @ResponseStatus(HttpStatus.BAD_REQUEST) //因为前后端分离 返回一个状态
    @ExceptionHandler(value =  RuntimeException.class)//捕获运行时异常
    public Result handler(RuntimeException e){
        log.error("运行时异常：-----------------{}",e);
        return Result.fail(e.getMessage());
    }
}
```

**然而我们运行测试发现并没有拦截**  

![](<images/1683768209205.png>)

  
**因为我们没有进行登录拦截**  
**@RequiresAuthentication// 登录拦截注解**  

![](<images/1683768209228.png>)

  
运行效果：  
提示 401 登录异常  

![](<images/1683768209554.png>)

## 第六章 实体校验

**当我们表单数据提交的时候，前端的校验我们可以使用一些类似于 jQuery Validate 等 js 插件实现，而后端我们可以使用 Hibernate validatior 来做校验。  
我们使用 springboot 框架作为基础，那么就已经自动集成了 Hibernate validatior。(校验登录非空等等)**

*   **User 实体类中**

```
import javax.validation.constraints.Email;
import javax.validation.constraints.NotBlank;
....

@TableName("m_user")
public class User implements Serializable {
    private static final long serialVersionUID = 1L;
    @TableId(value = "id", type = IdType.AUTO)
    private Long id;
    @NotBlank(message = "昵称不能为空")
    private String username;
    @NotBlank(message = "邮箱不能为空")
    @Email(message = "邮箱格式不正确")
    private String email;
    
    ...
}
```

*   **在 userController 类中写一个方法测试**

```
/**
     *
     *@RequestBody主要用来接收前端传递给后端的json字符串中的数据的(请求体中的数据的)；
     * GET方式无请求体，所以使用@RequestBody接收数据时，
     * 前端不能使用GET方式提交数据，
     * 而是用POST方式进行提交。在后端的同一个接收方法里，
     * @RequestBody与@RequestParam()可以同时使用，@RequestBody最多只能有一个，
     * 而@RequestParam()可以有多个。
     *
     * @Validated注解用于检查user中填写的规则  如果不满足抛出异常
     * 可在GlobalExceptionHandler中捕获此异常 进行自定义 返回数据信息
     */
    @PostMapping("/save")
    public  Result save(@Validated @RequestBody User user){

        return Result.succ(user);
    }
```

![](<images/1683768209573.png>)

![](<images/1683768209609.png>)

  
**启动 postMan 测试  
如果 postman 没法启动  
可以新建一个环境变量 POSTMAN_DISABLE_GPU = true。  
1. 打开高级系统设置；  
2. 在 “高级” 选项卡中，单击“环境变量”；  
3. 添加一个新的系统变量；  
4. 关闭 Postman 并重新打开**  

![](<images/1683768209629.png>)

**定义捕获异常返回处理**  
在捕获异常 GlobalExceptionHandler 类中修改如下：

```
/**
     * 实体校验异常
     * MethodArgumentNotValidException捕获实体校验异常
     */
    @ResponseStatus(HttpStatus.BAD_REQUEST) //因为前后端分离 返回一个状态
    @ExceptionHandler(value =  MethodArgumentNotValidException.class)//捕获运行时异常
    public Result handler(MethodArgumentNotValidException e){
        log.error("实体捕获异常  ：-----------------{}",e);
        BindingResult bindingException = e.getBindingResult();
        //多个异常顺序抛出异常
        ObjectError objectError = bindingException.getAllErrors().stream().findFirst().get();
        return Result.fail(objectError.getDefaultMessage());
    }
```

![](<images/1683768209865.png>)

  
**效果如下：(变得简短了)**  

![](<images/1683768209902.png>)

  
**输入正确的格式如下：**  
返回了我们需要的信息  

![](<images/1683768209920.png>)

## 第七章 跨域问题

因为是前后端分析，所以跨域问题是避免不了的，我们直接在后台进行全局跨域处理:  
**路径: com.vueblog.config**  
**注意：此配置是配置到 confroller 的，在 confroller 之前是经过 jwtFilter，所以在进行访问之前配置一下 Filter 的跨域问题**

```
package com.vueblog.config;

import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.CorsRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

/**
 * 解决跨域问题
 */
@Configuration
public class CorsConfig implements WebMvcConfigurer {
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")
                .allowedOrigins("*")
                .allowedMethods("GET", "HEAD", "POST", "PUT", "DELETE", "OPTIONS")
                .allowCredentials(true)
                .maxAge(3600)
                .allowedHeaders("*");
    }
}
```

**jwtFilter 进行跨域处理:**

*   **路径: com.vueblog.shiro**

```
/**
     * 对跨域提供支持
     * @param request
     * @param response
     * @return
     * @throws Exception
     */
    @Override
    protected boolean preHandle(ServletRequest request, ServletResponse response) throws Exception {
        HttpServletRequest httpServletRequest = WebUtils.toHttp(request);
        HttpServletResponse httpServletResponse = WebUtils.toHttp(response);
        httpServletResponse.setHeader("Access-control-Allow-Origin", httpServletRequest.getHeader("Origin"));
        httpServletResponse.setHeader("Access-Control-Allow-Methods", "GET,POST,OPTIONS,PUT,DELETE");
        httpServletResponse.setHeader("Access-Control-Allow-Headers", httpServletRequest.getHeader("Access-Control-Request-Headers"));
        // 跨域时会首先发送一个OPTIONS请求,这里我们给OPTIONS请求直接返回正常状态
        if (httpServletRequest.getMethod().equals(RequestMethod.OPTIONS.name())) {
            httpServletResponse.setStatus(org.springframework.http.HttpStatus.OK.value());
            return false;
        }
        return super.preHandle(request, response);
    }
```

**---- 基本框架已经搭建完成 -----**

## 第七章 登录接口开发

*   **创建 LoginDto**
*   **路径： com.vueblog.common.dto**

```
package com.vueblog.common.dto;

import lombok.Data;

import javax.validation.constraints.NotBlank;
import java.io.Serializable;

@Data
public class LoginDto implements Serializable {

    @NotBlank(message = "用户名不能为空")
    private String username;
    @NotBlank(message = "密码不能为空")
    private String password;
}
```

*   **在 GlobalExceptionHandler 类中增加断言异常**  
    路径：com.vueblog.common.exception

```
/**
     * 断言异常
     * @param e
     * @return
     */
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    @ExceptionHandler(value = IllegalArgumentException.class)
    public Result handler(IllegalArgumentException e){
        log.error("Assert异常:------------------>{}",e);
        return Result.fail(e.getMessage());
    }
```

*   **创建 AccountController 类**  
    登录和退出逻辑

```
package com.vueblog.controller;

import cn.hutool.core.lang.Assert;
import cn.hutool.core.map.MapUtil;
import cn.hutool.crypto.SecureUtil;
import com.baomidou.mybatisplus.core.conditions.query.QueryWrapper;
import com.vueblog.common.dto.LoginDto;
import com.vueblog.common.lang.Result;
import com.vueblog.entity.User;
import com.vueblog.service.UserService;
import com.vueblog.util.JwtUtils;
import org.apache.shiro.SecurityUtils;
import org.apache.shiro.authz.annotation.RequiresAuthentication;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.validation.annotation.Validated;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import javax.servlet.http.HttpServletResponse;

@RestController
public class AccountController {

    @Autowired
    UserService userService;

    @Autowired
    JwtUtils jwtUtils;

    @RequestMapping("/login")
    public Result login(@Validated @RequestBody LoginDto loginDto, HttpServletResponse response){
        User user = userService.getOne(new QueryWrapper<User>().eq("username", loginDto.getUsername()));
        Assert.notNull(user,"用户不存在");//断言拦截
        //判断账号密码是否错误 因为是md5加密所以这里md5判断
        if(!user.getPassword().equals(SecureUtil.md5(loginDto.getPassword()))){
            //密码不同则抛出异常
            return Result.fail("密码不正确");
        }
        String jwt = jwtUtils.generateToken(user.getId());

        //将token 放在我们的header里面
        response.setHeader("Authorization",jwt);
        response.setHeader("Access-control-Expose-Headers","Authorization");

        return Result.succ(MapUtil.builder()
                .put("id",user.getId())
                .put("username",user.getUsername())
                .put("avatar",user.getAvatar())
                .put("email",user.getEmail()).map()

        );
    }

    //需要认证权限才能退出登录
    @RequiresAuthentication
    @RequestMapping("/logout")
    public Result logout() {
        //退出登录
        SecurityUtils.getSubject().logout();
        return null;
    }



}
```

运行效果:  

![](<images/1683768210173.png>)

![](<images/1683768210203.png>)

## 第八章 博客接口的开发

*   **创建工具类 ShiroUtil**  
    路径 om.vueblog.util，可于判断等等

```
package com.vueblog.util;

import com.vueblog.shiro.AccountProfile;
import org.apache.shiro.SecurityUtils;

public class ShiroUtil {
    public static AccountProfile getProfile(){

        return (AccountProfile) SecurityUtils.getSubject().getPrincipal();
    }
}
```

*   **完善 BlogController 类**

```
package com.vueblog.controller;

import cn.hutool.core.bean.BeanUtil;
import cn.hutool.core.lang.Assert;
import com.baomidou.mybatisplus.core.conditions.query.QueryWrapper;
import com.baomidou.mybatisplus.core.metadata.IPage;
import com.baomidou.mybatisplus.extension.plugins.pagination.Page;
import com.vueblog.common.lang.Result;
import com.vueblog.entity.Blog;
import com.vueblog.service.BlogService;
import com.vueblog.util.ShiroUtil;
import org.apache.shiro.authz.annotation.RequiresAuthentication;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.validation.annotation.Validated;
import org.springframework.web.bind.annotation.*;

import java.time.LocalDateTime;

/**
 * <p>
 *  前端控制器
 * </p>
 *
 * @author 深林中的书海
 * @since 2021-02-05
 */
@RestController
public class BlogController {

    @Autowired
    BlogService blogService;
    //木有值默认为1
    @GetMapping("/blogs")
    public Result list(@RequestParam(defaultValue = "1") Integer currentPage){
        Page page = new Page(currentPage, 5);
        IPage pageData = blogService.page(page, new QueryWrapper<Blog>().orderByDesc("created"));
        return Result.succ(pageData);
    }

    //@PathVariable动态路由
    @GetMapping("/blog/{id}")
    public Result detail(@PathVariable Long id){
        Blog blog = blogService.getById(id);
        //判断是否为空 为空则断言异常
        Assert.notNull(blog,"该博客已被删除");
        return Result.succ(blog);
    }
    //@Validated校验
    //@RequestBody
    //添加删除  木有id则添加 有id则编辑
    @RequiresAuthentication  //需要认证之后才能操作
    @PostMapping("/blog/edit")
    public Result edit(@Validated @RequestBody Blog blog){

        //一个空对象用于赋值
        Blog temp = null;
        //如果有id则是编辑
        if(blog.getId() != null){
            temp = blogService.getById(blog.getId());//将数据库的内容传递给temp
            //只能编辑自己的文章
            Assert.isTrue(temp.getUserId().longValue()== ShiroUtil.getProfile().getId().longValue(),"没有编辑权限");

        } else {
            temp = new Blog();
            temp.setUserId(ShiroUtil.getProfile().getId());
            temp.setCreated(LocalDateTime.now());
            temp.setStatus(0);
        }
        //将blog的值赋给temp 忽略 id userid created status 引用于hutool
        BeanUtil.copyProperties(blog,temp,"id","userId","created","status");
        blogService.saveOrUpdate(temp);

        return Result.succ(null);
    }

    //@PathVariable动态路由
    @RequiresAuthentication  //需要认证之后才能操作
    @PostMapping("/blogdel/{id}")
    public Result del(@PathVariable Long id){
        boolean b = blogService.removeById(id);
        //判断是否为空 为空则断言异常
      if(b==true){

          return Result.succ("文章删除成功");
      }else{
          return Result.fail("文章删除失败");
      }
    }

}
```

*   **运行程序**  
    **查询测试:**

![](<images/1683768210233.png>)

  

![](<images/1683768210258.png>)

  
**新增编辑测试:**  
得到 token  

![](<images/1683768210322.png>)

  
选中 header=> 填写 token  

![](<images/1683768210372.png>)

  
选中 body=>raw=>json 填写请求

新增

```
{
	"title":"标题测试",
	"description":"描述测试哦奥德萨",
	"content":"内容测试阿斯顿撒哦i等哈送到哈桑"
}
```

![](<images/1683768210610.png>)

  
修改

```
{
	"id":21,
	"title":"标题测试修改",
	"description":"描述测试哦奥德萨",
	"content":"内容测试阿斯顿撒哦i等哈送到哈桑"
}
```

![](<images/1683768210644.png>)

  
**文章删除**  

![](<images/1683768210674.png>)

## 后端总结

**后端的一个骨架基本完成然后开始我们的前端开发**

## 第九章 Vue 前端页面开发

### 前言

**接下来，我们来完成 vueblog 前端的部分功能。可能会使用的到技术如下：  
vue  
element-ui  
axios  
mavon-editor  
markdown-it  
github-markdown-css**

### 环境准备

*   **node.js 安装:**

[https://nodejs.org/zh-cn/](https://nodejs.org/zh-cn/)  

![](<images/1683768210690.png>)

**安装完成之后检查下版本信息：**  

![](<images/1683768210726.png>)

*   **接下来，我们安装 vue 的环境**

```
# 安装淘宝npm
npm install -g cnpm --registry=https://registry.npm.taobao.org
# vue-cli 安装依赖包
cnpm install --g vue-cli
```

### 新建项目

**选中要建立的文件 cmd 打开**  

![](<images/1683768210757.png>)

  
**进入你的项目目录，创建一个基于 webpack 模板的新项目: vue init webpack 项目名**

![](<images/1683768210797.png>)

**输入:**  
**vue init webpack xx**  
**全部 enter 即可**

![](<images/1683768210816.png>)

  
**完成**  

![](<images/1683768210833.png>)

  
**运行项目**  
进入 demo(项目) 目录 运行

```
cd demo
npm run dev
```

![](<images/1683768210862.png>)

  
**浏览器打开**  

![](<images/1683768210895.png>)

### 项目的架构

```
├── README.md            项目介绍
├── index.html           入口页面
├── build              构建脚本目录
│  ├── build-server.js         运行本地构建服务器，可以访问构建后的页面
│  ├── build.js            生产环境构建脚本
│  ├── dev-client.js          开发服务器热重载脚本，主要用来实现开发阶段的页面自动刷新
│  ├── dev-server.js          运行本地开发服务器
│  ├── utils.js            构建相关工具方法
│  ├── webpack.base.conf.js      wabpack基础配置
│  ├── webpack.dev.conf.js       wabpack开发环境配置
│  └── webpack.prod.conf.js      wabpack生产环境配置
├── config             项目配置
│  ├── dev.env.js           开发环境变量
│  ├── index.js            项目配置文件
│  ├── prod.env.js           生产环境变量
│  └── test.env.js           测试环境变量
├── mock              mock数据目录
│  └── hello.js
├── package.json          npm包配置文件，里面定义了项目的npm脚本，依赖包等信息
├── src               源码目录 
│  ├── main.js             入口js文件
│  ├── app.vue             根组件
│  ├── components           公共组件目录
│  │  └── title.vue
│  ├── assets             资源目录，这里的资源会被wabpack构建
│  │  └── images
│  │    └── logo.png
│  ├── routes             前端路由
│  │  └── index.js
│  ├── store              应用级数据（state）状态管理
│  │  └── index.js
│  └── views              页面目录
│    ├── hello.vue
│    └── notfound.vue
├── static             纯静态资源，不会被wabpack构建。
└── test              测试文件目录（unit&e2e）
  └── unit              单元测试
    ├── index.js            入口脚本
    ├── karma.conf.js          karma配置文件
    └── specs              单测case目录
      └── Hello.spec.js
```

**集成开发工具 Visual Studio Code(VsCode)**

*   **VsCode 的安装：**  
    [https://code.visualstudio.com/](https://code.visualstudio.com/)  
    
    ![](<images/1683768210916.png>)
    

### vscode 前端常用插件推荐，搭建 JQuery、Vue 等开发环境

**汉化插件：chinese**  
**插件如何导入和一些推荐插件：**  
[https://blog.csdn.net/jiandan1127/article/details/85957003/](https://blog.csdn.net/jiandan1127/article/details/85957003/)

#### **使用 vscode 打开项目并且运行**

**ctrl+`(~ 键) 打开终端**  
**输入：**  
**npm run dev**

![](<images/1683768210950.png>)

  
效果如下则成功：  

![](<images/1683768210973.png>)

### 安装 element-ui

*   **官方文档：**

[https://element.eleme.cn/#/zh-CN/component/installation](https://element.eleme.cn/#/zh-CN/component/installation)

**ctrl+`(~ 键) 打开终端输入安装命令**

```
# 安装element-ui
cnpm install element-ui --save
```

**然后我们打开项目 src 目录下的 main.js，引入 element-ui 依赖。**

```
import Element from 'element-ui'
import "element-ui/lib/theme-chalk/index.css"
Vue.use(Element)
```

![](<images/1683768210993.png>)

*   **测试 elementUi 是否引入成功**

```
import Vue from 'vue'
import Router from 'vue-router'
import HelloWorld from '@/components/HelloWorld'
import Demo from '@/views/Demo' 

Vue.use(Router)

export default new Router({
  routes: [
    {
      path: '/',
      name: 'HelloWorld',
      component: HelloWorld
    }, {
      path: '/Demo',
      name: 'Demo',
      component: Demo
    }
  ]
})
```

*   **新建 Demo.vue**

**src/views/demo**  
**引入 button 按钮 (官方直接复制)**

```
<template>
    <div class="Demo">
        {{msg}}
         <el-button type="primary">主要按钮</el-button>
    </div>
</template>
<script> export default{
        name: "Demo",
        data() {
            return {
                msg: "测试"
            };
        }
    } </script>
```

![](<images/1683768211065.png>)

*   **App.vue 添加：**

```
<div id="nav">
      <router-link to="/demo">Demo页面</router-link>
    </div>
```

![](<images/1683768211096.png>)

*   **运行项目浏览器打开项目**

![](<images/1683768211116.png>)

*   **显示下方则成功**

![](<images/1683768211179.png>)

### 补充配置：

**autoOpenBrowser 修改为 true。**

```
autoOpenBrowser:true
```

![](<images/1683768211211.png>)

### 安装 [axios](https://so.csdn.net/so/search?q=axios&spm=1001.2101.3001.7020)

接下来，我们来安装 axios（[www.axios-js.com/](http://www.axios-js.com/)），axios 是一个基于 promise 的 HTTP 库，这样我们进行前后端对接的时候，使用这个工具可以提高我们的开发效率。

*   **安装命令:**

```
cnpm install axios --save
```

![](<images/1683768211230.png>)

*   **然后同样我们在 main.js 中全局引入 axios。**

```
import axios from 'axios'
//引用全局
Vue.prototype.$axios = axios
```

![](<images/1683768211277.png>)

### 页面路由

接下来，我们先定义好路由和页面，因为我们只是做一个简单的博客项目，页面比较少，所以我们可以直接先定义好，然后在慢慢开发，这样需要用到链接的地方我们就可以直接可以使用：  
我们在 views 文件夹下定义几个页面：

**BlogDetail.vue（博客详情页）  
BlogEdit.vue（编辑博客）  
Blogs.vue（博客列表）  
Login.vue（登录页面）**  
**可以配置插件: VueHelper(新建 vue 项目 最上方输入 vuet 按键 tab 可直接生成模板)**  

![](<images/1683768211308.png>)

  

![](<images/1683768211334.png>)

  
**建立页面完成**  

![](<images/1683768211359.png>)

```
注意:每个页面下方<template><temlate> 里面只能有一个<div></div>
```

![](<images/1683768211388.png>)

### 路由中心配置

路径：**rountr/index.js**

```
import Vue from 'vue'
import Router from 'vue-router'
import HelloWorld from '@/components/HelloWorld'
import Demo from '@/views/Demo' 
import Login from '@/views/Login' 
import Blogs from '@/views/Blogs' 
import BlogEdit from '@/views/BlogEdit' 
import BlogDetail from '@/views/BlogDetail' 

Vue.use(Router)

export default new Router({
  routes: [
    {
      path: '/',
      name: 'Index',
      redirect:{name : "Blogs"}
    },
    {
      path: '/blogs',
      name: 'Blogs',
      component: Blogs
    },{
    path: '/Login',
    name: 'Login',
    component: Login
    },{
      path: '/blog/add',
      name: 'BlogEdit',
      component: BlogEdit
    }, {
      path: '/Demo',
      name: 'Demo',
      component: Demo
    },{
      path: '/blog/:blogid',
      name: 'BlogDetail',
      component: BlogDetail
    } ,{
      path: '/blog/:blogid/edit',
      name: 'BlogEdit', 
      component: BlogEdit
    }
]})
```

**打开终端 npm run dev 启动项目**  
可自行测试其他路由  

![](<images/1683768211432.png>)

  
**测试完我们可以删掉 App.vue 中的测试和图片**  

![](<images/1683768211450.png>)

### 登录页面开发

#### 登录页面制作

**Login.vie**

```
<template>
  <div>
    <el-container>
      <el-header>
        <img class="mlogo" src="../assets/logo.jpg" alt="logo图片" />
      </el-header>
      <el-main>
        <el-form :model="ruleForm" :rules="rules" ref="ruleForm" label-width="100px" class="demo-ruleForm">
          <el-form-item label="用户姓名" prop="username">
            <el-input v-model="ruleForm.username"></el-input>
          </el-form-item>
          <el-form-item label="用户密码" prop="password">
            <el-input v-model="ruleForm.password"></el-input>
          </el-form-item>
          <el-form-item>
            <el-button type="primary" @click="submitForm('ruleForm')">立即创建</el-button>
            <el-button @click="resetForm('ruleForm')">重置</el-button>
          </el-form-item>
        </el-form>
      </el-main>
    </el-container>
  </div>
</template>
<script> export default {
  name: "Login",
  data () {
    return {
      ruleForm: {
        username: '',
        password: ''
      },
      rules: {
        username: [
          { required: true, message: '请输入用户名称', trigger: 'blur' },
          { min: 3, max: 15, message: '长度在 3 到 15 个字符', trigger: 'blur' }

        ],
        password: [
          { required: true, message: '请选择密码', trigger: 'blur' }
        ]
      }
    };
  },
  methods: {
    submitForm (formName) {
      this.$refs[formName].validate((valid) => {
        if (valid) {
          alert('submit!');
        } else {
          console.log('error submit!!');
          return false;
        }
      });
    },
    resetForm (formName) {
      this.$refs[formName].resetFields();
    }
  }
} </script>

<style scoped> .el-header,
.el-footer {
  background-color: #b3c0d1;
  color: #333;
  text-align: center;
  line-height: 60px;
}

.el-main {
  /* background-color: #e9eef3; */
  color: #333;
  text-align: center;
  line-height: 160px;
}
.mlogo {
  height: 80%;
}

.demo-ruleForm {
  max-width: 500px;
  margin: 0 auto;
} </style>
```

![](<images/1683768211480.png>)

#### 配置 store

**在 src 包下面创建 stroe 文件夹  
然后创建 index.js**  

![](<images/1683768211498.png>)

```
import Vue from 'vue'
import Vuex from 'vuex';

Vue.use(Vuex)

export default new Vuex.Store({
  state: {
    token:'',
    userInfo:{}
  },
  mutations: {
    
  },
  actions: {
    
  },
  modules: {
    
  }
})
```

**然后我们给 main.js 中引入 store**

```
// 存储
import store from './store'
//在new Vue({}) 里面加入store
new Vue({
  el: '#app',
  router,
  store ,
  components: { App },
  template: '<App/>'
})
```

```
// The Vue build version to load with the `import` command
// (runtime-only or standalone) has been set in webpack.base.conf with an alias.
import Vue from 'vue'
import App from './App'
// 存储
import store from './store'
// 路由
import router from './router'
// 引入element-ui依赖
import Element from 'element-ui'
import "element-ui/lib/theme-chalk/index.css"
// 引入axios依赖
import axios from 'axios'

//引用全局
Vue.prototype.$axios = axios 

Vue.use(Element)
Vue.config.productionTip = false

/* eslint-disable no-new */
new Vue({
  el: '#app',
  router,
  store ,
  components: { App },
  template: '<App/>'
})
```

** npm run dev 启动测试 **

#### 登录发起请求 (配置完善 store)

**配置完善 store/index.js**

在 store 下面的 index.js 添加 *  
**mutationsx 相当于 java 中实体类的 set  
getters 相当于 get**

```
> userInfo可以存入会话的sessionStorage里面
> **sessionStorage中只能存字符串 不能存入对象所以我们存入序列化 jons串:**
> **sessionStorage.setItem("userInfo",JSON.stringify(userInfo))**
> 会话获取
> **sessionStorage.getInte(userInfo)**
```

```
> token可以存入浏览器的localStorage里面
 > localStorage.setItem("token",token)
 > token获取：
 > localStorage.getItem("token")
```

```
import Vue from 'vue'
import Vuex from 'vuex'

Vue.use(Vuex)

export default new Vuex.Store({
  //定义全局参数 其他页面可以直接获取state里面的内容
  state: {
    token: '', //方法一 localStorage.getItem("token") 
    //反序列化获取session会话中的 userInfo对象
    userInfo:JSON.parse(sessionStorage.getItem("userInfo"))
  },
  mutations: {
    //相当于实体类的set
    SET_TOKEN:(state,token)=>{
      state.token=token//将传入的token赋值 给state的token
      //同时可以存入浏览器的localStorage里面
      localStorage.setItem("token",token)
    },
    SET_USERINFO:(state,userInfo)=>{
      state.userInfo=userInfo//将传入的tuserInfo赋值 给state的userInfo
      //同时可以存入会话的sessionStorage里面 sessionStorage中只能存字符串 不能存入对象所以我们存入序列化 jons串
      sessionStorage.setItem("userInfo",JSON.stringify(userInfo))
    },
    //删除token及userInfo
    REMOVE_INFO:(state)=>{
      state.token = '';
      state.userInfo = {};
      localStorage.setItem("token",'')
      sessionStorage.setItem("userInfo",JSON.stringify(''))
    }
  },
  getters: {
    //相当于get
    //配置一个getUser可以直接获取已经反序列化对象的一个userInfo
   getUser: state=>{
     return state.userInfo;
   },getToken: state=>{
    return state.token;
  }
  },
  actions: {
    
  },
  modules: {
    
  }
})
```

**在 Login.vue 中添加**

```
_this.$axios.post("http://localhost:8081/login", _this.ruleForm).then(res => {

            console.log(res)
            const jwt = res.headers['authorization'];
            const userInfo = res.data.data
            _this.$store.commit('SET_TOKEN', token)
            _this.$store.commit('SET_USERINFO', userInfo)
          })
```

```
<template>
  <div>
    <el-container>
      <el-header>
        <img class="mlogo" src="../assets/logo.jpg" alt="logo图片" />
      </el-header>
      <el-main>
        <el-form :model="ruleForm" :rules="rules" ref="ruleForm" label-width="100px" class="demo-ruleForm">
          <el-form-item label="用户姓名" prop="username">
            <el-input v-model="ruleForm.username"></el-input>
          </el-form-item>
          <el-form-item label="用户密码" prop="password">
            <el-input v-model="ruleForm.password"></el-input>
          </el-form-item>
          <el-form-item>
            <el-button type="primary" @click="submitForm('ruleForm')">立即创建</el-button>
            <el-button @click="resetForm('ruleForm')">重置</el-button>
          </el-form-item>
        </el-form>
      </el-main>
    </el-container>
  </div>
</template>
<script> export default {
  name: "Login",
  data () {
    return {
      ruleForm: {
        username: 'admin',
        password: '111111'
      },
      rules: {
        username: [
          { required: true, message: '请输入用户名称', trigger: 'blur' },
          { min: 3, max: 15, message: '长度在 3 到 15 个字符', trigger: 'blur' }

        ],
        password: [
          { required: true, message: '请选择密码', trigger: 'blur' }
        ]
      }
    };
  },
  methods: {
    submitForm (formName) {
      this.$refs[formName].validate((valid) => {
        if (valid) {
          const _this = this;//获取整个vue的this
          console.log("校验成功")
          //  alert('submit!');
          _this.$axios.post("http://localhost:8081/login", _this.ruleForm).then(res => {

            console.log(res)
            const jwt = res.headers['authorization'];
            const userInfo = res.data.data
            //存储(共享)全局变量jwt和userInfo
            _this.$store.commit("SET_TOKEN", jwt);
            _this.$store.commit("SET_USERINFO", userInfo);

            //获取token和getUser
            // console.log(_this.$store.getters.getToken)
            // console.log(_this.$store.getters.getUser) 
          })

        } else {
          console.log('error submit!!');
          return false;
        }
      });
    },
    resetForm (formName) {
      this.$refs[formName].resetFields();
    }
  }
} </script>

<style scoped> .el-header,
.el-footer {
  background-color: #b3c0d1;
  color: #333;
  text-align: center;
  line-height: 60px;
}

.el-main {
  /* background-color: #e9eef3; */
  color: #333;
  text-align: center;
  line-height: 160px;
}
.mlogo {
  height: 80%;
}

.demo-ruleForm {
  max-width: 500px;
  margin: 0 auto;
} </style>
```

*   **打开页面点击登录**  
    **可查看我们的信息已经存入浏览器中**  
    
    ![](<images/1683768211529.png>)
    

#### **配置 axios 拦截**

*   **src 下面创建 axios.js**

![](<images/1683768211560.png>)

*   **main.js 中引入**

```
// 引入自定义axios.js
import "./axios.js"
```

![](<images/1683768211578.png>)

*   **完善 axios.js**

```
// 引入axios依赖
import axios from 'axios'
// 引入element-ui依赖
import Element from 'element-ui'
import store from './store';
// 引入路由
import router from './router'

var s=window.location.toString();   
		    var s1=s.substr(7,s.length);
		    var s2=s1.indexOf("/");
		    s=s.substr(0,8+s2);
	var a = "http://localhost:8081/";//获取连接前缀相当于 http://localhost:8081/

  //配置默认前缀
axios.defaults.baseURL= a

//配置前置拦截
axios.interceptors.request.use(config => { 
  return config
})

//配置后置拦截
axios.interceptors.response.use(response=>{
    let res = response.data; 
    if(res.code == 200){
      return response
    }else{ 
        Element.Message.error("操作错了哦",{duration : 2*1000}) 
        //返回一个异常提示就不会继续往下走了 不+的话 res=>的里面 还是会继续走的
        return Promise.reject(response.data.msg)
    }
     // 捕获并处理后台异常信息
  },error=>{
     // 使得异常信息更加友好
    console.log(error) 
    if (error.response.data) { //data不为空时
      error.message = error.response.data.msg
      console.log("-------------------------")
      console.log(error.message)
      console.log("-------------------------")
  }
    if(error.response.status == 401){
      store.commit('REMOVE_INFO')//清空token userinfo
      router.push("/login")  //跳转登录页面
    } 
    Element.Message.error(error.message)
    
    return Promise.reject(error)
  }

)
```

*   **npm run dev 运行效果**

![](<images/1683768211609.png>)

#### 公共组件 Header

**打开 AccountController**  

![](<images/1683768211642.png>)

**要先打开在 logout 方法里面返回一个 Result 不然会报错**  

![](<images/1683768211663.png>)

  
**如果报错了这个需要先安装 redis 并且启动**

[https://www.cnblogs.com/pretty-sunshine/p/10615287.html](https://www.cnblogs.com/pretty-sunshine/p/10615287.html)

![](<images/1683768211693.png>)

  
**完善配置 Header 组件**

```
<template>
  <div class="m-content">
    <h3>欢迎来到我的博客世界</h3>
    <div class="block">
      <div class="block">
        <el-avatar :size="50" :src="user.avatar"></el-avatar>
      </div>
      <div>{{user.username}}</div>
      <div class="maction">
        <span>
          <el-link>主页</el-link>
        </span>
        <el-divider direction="vertical"></el-divider>
        <span>
          <el-link type="success">发表博客</el-link>
        </span>

        <el-divider direction="vertical"></el-divider>
        <span v-show="!hasLogin">
          <el-link type="primary" @click="login">登录</el-link>
        </span>
        <span v-show="hasLogin">
          <el-link type="danger" @click="logout">退出</el-link>
        </span>
      </div>
    </div>
  </div>
</template>

<script> export default {
  name: "Header"
  , data () {
    return {
      user: {
        username: '请先登录',
        avatar: '../assets/logo.png'
      },
      hasLogin: false
    }
  },
  methods: {
    //退出操作
    logout () {
      const _this = this
      //首先调用后端logout接口(因该接口需要认证权限,所以需要传入token)
      //其次调用$store清除用户信息及token
      _this.$axios.get("/logout", {
        headers: {
          "Authorization": localStorage.getItem("token")
        }
      }).then(res => {
        _this.$store.commit("REMOVE_INFO")
        _this.$router.push("/login")
      })
    },
    login () { //跳转登录页面进行登录
      this.$router.push("/login")
    }
  },
  //页面创建时即会调用,进而获取用户信息
  created () {
    console.log("=======123=")
    console.log(this.$store.getters.getUser)
    if (this.$store.getters.getUser.username) {//如果username不为空
      this.user.username = this.$store.getters.getUser.username
      this.user.avatar = this.$store.getters.getUser.avatar
      //判断是登录状态还是非登录显示 退出按钮或者登录按钮
      this.hasLogin = true;
    }
  }
} </script>

<style> .m-content {
  max-width: 960px;
  margin: 0 auto;
  text-align: center;
}
.maction {
} </style>
```

**运行项目可以自行测试登录退出功能**

![](<images/1683768211724.png>)

  

![](<images/1683768211744.png>)

*   **完善 blogs.vue**  
    **先给实体类 Blog 加上 JsonFormat**

```
@JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss")
    private LocalDateTime created;
```

![](<images/1683768211771.png>)

*   **完善 Blogs.vue**

```
<template>
  <div class="mcontaner">
    <Header></Header>
    <div class="block">
      <el-timeline>
        <el-timeline-item :timestamp="blog.created" placement="top" v-for="(blog,key) in blogs" :key=key>
          <el-card>

            <h4>
              <router-link :to="{  name: 'BlogDetail', params: {blogid: blog.id}}">{{blog.title}}</router-link>
            </h4>

            <p>{{blog.description}}</p>
          </el-card>
        </el-timeline-item>

      </el-timeline>

      <el-pagination class="mage" background layout="prev, pager, next" :current-page="currentPage"
        :page-size="pageSize" :total="total" @current-change=page>
      </el-pagination>
    </div>
  </div>
</template>

<script> //引入header组件
import Header from "../components/Header";

export default {
  data () {
    return {
      blogs: {},
      currentPage: 1,
      total: 0,
      pageSize: 5
    }
  },
  name: "Blogs",
  components: { Header },

  methods: {
    page (currentPage) {
      const _this = this
      _this.$axios.get("/blogs?currentPage=" + currentPage).then(res => {
        var data = res.data.data
        _this.blogs = data.records
        _this.currentPage = data.current
        _this.pageSize = data.size
        _this.total = data.total

      })
    }
  },
  created () {
    this.page(1)
  }
} </script>

<style scoped> .block {
  margin: 20px;
}

.mage {
  margin: 0 auto;
} </style>
```

![](<images/1683768211801.png>)

![](<images/1683768211818.png>)

![](<images/1683768211855.png>)

### 博客编辑（发表）

#### 安装 mavon-editor

*   **基于 Vue 的 markdown 编辑器 mavon-editor**

```
cnpm install mavon-editor --save
```

*   **然后在 main.js 中全局注册：**

```
// 全局注册 
import mavonEditor from 'mavon-editor'
import 'mavon-editor/dist/css/index.css'
// use
Vue.use(mavonEditor)
```

```
// The Vue build version to load with the `import` command
// (runtime-only or standalone) has been set in webpack.base.conf with an alias.
import Vue from 'vue'
import App from './App'
// 存储
import store from './store'
// 路由
import router from './router'
// 引入element-ui依赖
import Element from 'element-ui'
import "element-ui/lib/theme-chalk/index.css"
// 引入axios依赖
import axios from 'axios'   
// 引入自定义axios.js
import "./axios.js"
//mavonEditor
import mavonEditor from 'mavon-editor'
import 'mavon-editor/dist/css/index.css'

//引用全局
Vue.prototype.$axios = axios 

// use
Vue.use(mavonEditor)
Vue.use(Element)
Vue.config.productionTip = false

/* eslint-disable no-new */
new Vue({
  el: '#app',
  router,
  store ,
  components: { App },
  template: '<App/>'
})
```

**编写 BlogEdit.vue**

```
<template>
  <div>
    <Header></Header>
    <div class="m-content">
      <el-form :model="ruleForm" :rules="rules" ref="ruleForm" label-width="100px" class="demo-ruleForm">

        <el-form-item label="标题" prop="title">
          <el-input v-model="ruleForm.title"></el-input>
        </el-form-item>

        <el-form-item label="摘要" prop="description">
          <el-input type="textarea" v-model="ruleForm.description"></el-input>
        </el-form-item>

        <el-form-item label="内容" prop="content">
          <mavon-editor v-model="ruleForm.content"></mavon-editor>
        </el-form-item>

        <el-form-item>
          <el-button type="primary" @click="submitForm('ruleForm')">立即创建</el-button>
          <el-button @click="resetForm('ruleForm')">重置</el-button>
        </el-form-item>
      </el-form>
    </div>
  </div>
</template>

<script> import Header from "../components/Header";
export default {
  name: "BlogEdit",
  components: { Header },
  data () {
    return {
      ruleForm: {
        id: '',
        title: '',
        description: '',
        content: ''
      },
      rules: {
        title: [
          { required: true, message: '请输入标题', trigger: 'blur' },
          { min: 3, max: 5, message: '长度在 3 到 15 个字符', trigger: 'blur' }
        ],

        description: [
          { required: true, message: '请输入摘要', trigger: 'blur' },
        ],
        content: [
          { required: true, message: '请输入内容', trigger: 'blur' },
        ]
      }
    };
  },
  methods: {
    submitForm (formName) {
      this.$refs[formName].validate((valid) => {
        if (valid) {
          const _this = this
          this.$axios.post('/blog/edit', this.ruleForm, {
            headers: {
              "Authorization": localStorage.getItem("token")
            }
          }).then(res => {
            _this.$alert('操作成功', '提示', {
              confirmButtonText: '确定',
              callback: action => {
                _this.$router.push("/blogs")
              }
            });
          })
        } else {
          console.log('error submit!!');
          return false;
        }
      });
    },
    resetForm (formName) {
      this.$refs[formName].resetFields();
    }
  },
  created () {
    //获取动态路由的 blogid
    const blogId = this.$route.params.blogid
    console.log(blogId)
    const _this = this
    if (blogId) {
      this.$axios.get("/blog/" + blogId).then(res => {
        console.log(res)
        const blog = res.data.data
        _this.ruleForm.id = blog.id
        _this.ruleForm.title = blog.title
        _this.ruleForm.description = blog.description
        _this.ruleForm.content = blog.content

      })
    }

  }
} </script>

<style scoped> .m-content {
  text-align: center;
} </style>
```

**运行效果**  

![](<images/1683768211902.png>)

  

![](<images/1683768211921.png>)

**编辑：**

![](<images/1683768211961.png>)

### 博客详情和编辑按钮的权限

博客详情中需要回显博客信息，然后有个问题就是，后端传过来的是博客内容是 markdown 格式的内容，我们需要进行渲染然后显示出来，这里我们使用一个插件 markdown-it，用于解析 md 文档，然后导入 [github](https://so.csdn.net/so/search?q=github&spm=1001.2101.3001.7020)-markdown-c，所谓 md 的样式。

```
# 用于解析md文档
cnpm install markdown-it --save
# md样式
cnpm install github-markdown-css
```

然后就可以在需要渲染的地方使用：

*   views\BlogDetail.vue

```
<template>
  <div>
    <Header></Header>
    <div class="blog">
      <h2>{{blog.title}}</h2>
      <el-link icon="el-icon-edit" v-if="ownblog">
        <router-link :to="{name:'BlogEdit',params:{blogid:blog.id}}">
          编辑
        </router-link>

      </el-link>
      <el-divider></el-divider>
      <div class="markdown-body" v-html="blog.content"></div>
    </div>
  </div>
</template>

<script> import 'github-markdown-css'
import Header from "../components/Header";
export default {
  name: "BlogDetail",
  components: { Header },
  data () {
    return {
      blog: {
        id: '',
        title: '',
        content: '',
        description: ''

      },
      ownblog: false
    }
  },
  created () {
    //获取动态路由的 blogid
    const blogId = this.$route.params.blogid
    const _this = this
    if (blogId) {
      this.$axios.get("/blog/" + blogId).then(res => {
        const blog = res.data.data
        _this.blog.id = blog.id
        _this.blog.title = blog.title
        _this.blog.description = blog.description

        //MardownIt 渲染
        var MardownIt = require("markdown-it")
        var md = new MardownIt();
        var result = md.render(blog.content)
        _this.blog.content = result
        //查看是否是登录人 是则可以编辑
        _this.ownBlog = (blog.userId === _this.$store.getters.getUser.id)
      })
    }

  }
} </script>

<style scoped> .blog {
  margin-top: 10px;
  box-shadow: 0 2px 12px 0 rgba(0, 0, 0, 0.1);
  width: 100%;
  min-height: 700px;
  padding: 10px;
} </style>
```

### 路由权限拦截

**页面已经开发完毕之后，我们来控制一下哪些页面是需要登录之后才能跳转的，如果未登录访问就直接重定向到登录页面，因此我们在 src 目录下定义一个 js 文件：**

// 配置一个路由前置拦截 rounter 是路由

*   **src\permission.js**

```
import router from "./router";
// 路由判断登录 根据路由配置文件的参数
router.beforeEach((to, from, next) => {
  if (to.matched.some(record => record.meta.requireAuth)) { // 判断该路由是否需要登录权限
    const token = localStorage.getItem("token")
    console.log("------------" + token)
    if (token) { // 判断当前的token是否存在 ； 登录存入的token
      if (to.path === '/login') {
      } else {
        next()
      }
    } else {
      next({
        path: '/login'
      })
    }
  } else {
    next()
  }
})
```

**通过之前我们再定义页面路由时候的的 meta 信息，指定 requireAuth: true，需要登录才能访问，因此这里我们在每次路由之前（router.beforeEach）判断 token 的状态，觉得是否需要跳转到登录页面。**

*   **src/rouer/index.js**  
    添加：

meta: {  
requireAuth: true  
}

```
{
 path: '/blog/add', // 注意放在 path: '/blog/:blogId'之前
 name: 'BlogAdd',
 meta: {
 requireAuth: true
  },
 component: BlogEdit
},{
 path: '/blog/:blogid/edit',
 name: 'BlogEdit', 
 component: BlogEdit,
 meta: {
 requireAuth: true
      }
```

```
import Vue from 'vue'
import Router from 'vue-router'
import HelloWorld from '@/components/HelloWorld'
import Demo from '@/views/Demo' 
import Login from '@/views/Login' 
import Blogs from '@/views/Blogs' 
import BlogEdit from '@/views/BlogEdit' 
import BlogDetail from '@/views/BlogDetail' 

Vue.use(Router)

export default new Router({
  routes: [
    {
      path: '/',
      name: 'Index',
      redirect:{name : "Blogs"}
    },
    {
      path: '/blogs',
      name: 'Blogs',
      component: Blogs
    },{
    path: '/Login',
    name: 'Login',
    component: Login
    },{
      path: '/blog/add',
      name: 'BlogEdit',
      component: BlogEdit,
      meta: {
        requireAuth: true
      }
    }, {
      path: '/Demo',
      name: 'Demo',
      component: Demo
    },{
      path: '/blog/:blogid',
      name: 'BlogDetail',
      component: BlogDetail
    } ,{
      path: '/blog/:blogid/edit',
      name: 'BlogEdit', 
      component: BlogEdit,
      meta: {
        requireAuth: true
      }
    }
]})
```

**然后我们再 main.js 中 import 我们的 permission.js**

*   **src/main.js**

添加: import ‘./permission.js’ // 路由拦截

```
// The Vue build version to load with the `import` command
// (runtime-only or standalone) has been set in webpack.base.conf with an alias.
import Vue from 'vue'
import App from './App'
// 存储
import store from './store'
// 路由
import router from './router'
// 引入element-ui依赖
import Element from 'element-ui'
import "element-ui/lib/theme-chalk/index.css"

// 引入axios依赖
import axios from 'axios'   

// 引入自定义axios.js
import "./axios.js"
import './permission.js' // 路由拦截

//mavonEditor
import mavonEditor from 'mavon-editor'
import 'mavon-editor/dist/css/index.css'

//引用全局
Vue.prototype.$axios = axios 

// use
Vue.use(mavonEditor)
Vue.use(Element)
Vue.config.productionTip = false

/* eslint-disable no-new */
new Vue({
  el: '#app',
  router,
  store ,
  components: { App },
  template: '<App/>'
})
```

## 第十章 其他

## 增加删除文章功能

*   **前端**

**src/views/BlogDetail.vue**

```
<template>
  <div>
    <Header></Header>
    <div class="blog">
      <h2>{{blog.title}}</h2>
      <el-link icon="el-icon-edit" v-if="ownblog" class="linklist">
        <router-link :to="{name:'BlogEdit',params:{blogid:blog.id}}">
          编辑
        </router-link>
      </el-link>
      <el-link icon="el-icon-delete" v-if="ownblog" class="linklist">
        <el-button type="danger" round @click="delblog">删除</el-button>
      </el-link>
      <el-divider></el-divider>
      <div class="markdown-body" v-html="blog.content"></div>
    </div>
  </div>
</template>

<script> import 'github-markdown-css'
import Header from "../components/Header";
export default {
  name: "BlogDetail",
  components: { Header },
  data () {
    return {
      blog: {
        id: '',
        title: '',
        content: '',
        description: ''

      },
      ownblog: false
    }
  },
  methods: {
    delblog () {
      const blogId = this.$route.params.blogid
      const _this = this
      if (blogId) {
        this.$confirm('此操作将永久删除该文章, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          _this.$axios.post(`/blogdel/${blogId}`, null, {
            headers: {
              "Authorization": localStorage.getItem("token")
            }
          }).then(res => {
            this.$message({
              type: 'success',
              message: res.data.data
            });
            _this.$router.push("/blogs")
          })

        }).catch(() => {

          this.$message({
            type: 'info',
            message: '已取消删除'
          });
        });


      }
    }
  },
  created () {
    //获取动态路由的 blogid
    const blogId = this.$route.params.blogid
    const _this = this
    if (blogId) {
      this.$axios.get("/blog/" + blogId).then(res => {
        const blog = res.data.data
        _this.blog.id = blog.id
        _this.blog.title = blog.title
        _this.blog.description = blog.description

        //MardownIt 渲染
        var MardownIt = require("markdown-it")
        var md = new MardownIt();
        var result = md.render(blog.content)
        _this.blog.content = result

        //查看是否是登录人 是则可以编辑和删除
        _this.ownblog = (blog.userId === _this.$store.getters.getUser.id)
      })
    }

  }
} </script>

<style scoped> .blog {
  margin-top: 10px;
  box-shadow: 0 2px 12px 0 rgba(0, 0, 0, 0.1);
  width: 100%;
  min-height: 700px;
  padding: 10px;
}
.linklist {
  margin: 5px;
} </style>
```

*   **后端**  
    src/main/java/com/vuelog/controller/BlogController

```
//@PathVariable动态路由
    @RequiresAuthentication  //需要认证之后才能操作
    @PostMapping("/blogdel/{id}")
    public Result del(@PathVariable Long id){
        boolean b = blogService.removeById(id);
        //判断是否为空 为空则断言异常
      if(b==true){

          return Result.succ("文章删除成功");
      }else{
          return Result.fail("文章删除失败");
      }
    }
```

```
package com.vueblog.controller;

import cn.hutool.core.bean.BeanUtil;
import cn.hutool.core.lang.Assert;
import com.baomidou.mybatisplus.core.conditions.query.QueryWrapper;
import com.baomidou.mybatisplus.core.metadata.IPage;
import com.baomidou.mybatisplus.extension.plugins.pagination.Page;
import com.vueblog.common.lang.Result;
import com.vueblog.entity.Blog;
import com.vueblog.service.BlogService;
import com.vueblog.util.ShiroUtil;
import org.apache.shiro.authz.annotation.RequiresAuthentication;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.validation.annotation.Validated;
import org.springframework.web.bind.annotation.*;

import java.time.LocalDateTime;

/**
 * <p>
 *  前端控制器
 * </p>
 *
 * @author 深林中的书海
 * @since 2021-02-05
 */
@RestController
public class BlogController {

    @Autowired
    BlogService blogService;
    //木有值默认为1
    @GetMapping("/blogs")
    public Result list(@RequestParam(defaultValue = "1") Integer currentPage){
        Page page = new Page(currentPage, 5);
        IPage pageData = blogService.page(page, new QueryWrapper<Blog>().orderByDesc("created"));
        return Result.succ(pageData);
    }

    //@PathVariable动态路由
    @GetMapping("/blog/{id}")
    public Result detail(@PathVariable Long id){
        Blog blog = blogService.getById(id);
        //判断是否为空 为空则断言异常
        Assert.notNull(blog,"该博客已被删除");
        return Result.succ(blog);
    }
    //@Validated校验
    //@RequestBody
    //添加删除  木有id则添加 有id则编辑
    @RequiresAuthentication  //需要认证之后才能操作
    @PostMapping("/blog/edit")
    public Result edit(@Validated @RequestBody Blog blog){

        //一个空对象用于赋值
        Blog temp = null;
        //如果有id则是编辑
        if(blog.getId() != null){
            temp = blogService.getById(blog.getId());//将数据库的内容传递给temp
            //只能编辑自己的文章
            Assert.isTrue(temp.getUserId().longValue()== ShiroUtil.getProfile().getId().longValue(),"没有编辑权限");

        } else {
            temp = new Blog();
            temp.setUserId(ShiroUtil.getProfile().getId());
            temp.setCreated(LocalDateTime.now());
            temp.setStatus(0);
        }
        //将blog的值赋给temp 忽略 id userid created status 引用于hutool
        BeanUtil.copyProperties(blog,temp,"id","userId","created","status");
        blogService.saveOrUpdate(temp);

        return Result.succ(null);
    }

    //@PathVariable动态路由
    @RequiresAuthentication  //需要认证之后才能操作
    @PostMapping("/blogdel/{id}")
    public Result del(@PathVariable Long id){
        boolean b = blogService.removeById(id);
        //判断是否为空 为空则断言异常
      if(b==true){

          return Result.succ("文章删除成功");
      }else{
          return Result.fail("文章删除失败");
      }
    }

}
```