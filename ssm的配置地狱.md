# SpringMVC整合ssm步骤
1. 分析需求
2. 设计数据库
3. 设计业务
4. 前端界面

## 一、整合mybatis
1. 创建maven项目，导入依赖
```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.athos</groupId>
    <artifactId>book</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.encoding>UTF-8</maven.compiler.encoding>
        <java.version>11</java.version>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
    </properties>
    <dependencies>
<!--    配置log4j    -->
        <dependency>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
            <version>1.2.16</version>
        </dependency>
<!--    Junit    -->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
        </dependency>
<!--    数据库驱动    -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.21</version>
        </dependency>
<!--    数据库连接池druid    -->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid-spring-boot-starter</artifactId>
            <version>1.1.23</version>
        </dependency>
<!--    servlet、jsp、jstl    -->
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>servlet-api</artifactId>
            <version>2.5</version>
        </dependency>
        <dependency>
            <groupId>javax.servlet.jsp</groupId>
            <artifactId>jsp-api</artifactId>
            <version>2.2</version>
        </dependency>
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>jstl</artifactId>
            <version>1.2</version>
        </dependency>
<!--    mybatis    -->
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.5.5</version>
        </dependency>
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis-spring</artifactId>
            <version>2.0.5</version>
        </dependency>

<!--    spring    -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>5.2.8.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jdbc</artifactId>
            <version>5.2.8.RELEASE</version>
        </dependency>

    </dependencies>

<!--  maven 资源过滤设置  -->
<!--  静态资源导出问题-->
    <build>
        <resources>
            <resource>
                <directory>src/main/java</directory>
                <includes>
                    <include>**/*.properties</include>
                    <include>**/*.xml</include>
                </includes>
                <filtering>false</filtering>
            </resource>
            <resource>
                <directory>src/main/resource</directory>
                <includes>
                    <include>**/*.properties</include>
                    <include>**/*.xml</include>
                </includes>
                <filtering>false</filtering>
            </resource>
        </resources>
    </build>

</project>
```
2.连接数据库

- 使用idea右边侧边栏的database连接数据库
- 创建包路径：entity,mapper,service,controller...
- resource目录下创建文件：mybatis-config.xml,applicationContext.xml(spring核心配置文件),database.properties,spring-mapper.xml,spring-service.xml,spring-mvc.xml


applicationContext.xml

    ```

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                            http://www.springframework.org/schema/beans/spring-beans.xsd">
        <import resource="classpath:spring-dao.xml"/>
        <import resource="classpath:spring-service.xml"/>
        <import resource="classpath:spring-mvc.xml"/>
    </beans>
    ```

database.properties

    ```
    jdbc.driver=com.mysql.cj.jdbc.Driver
    jdbc.url=jdbc:mysql://localhost:3306/ssmbuild?useSSL=true&useUnicode=true&characterEncoding=utf8&serverTimezone=Asia/Shanghai
    jdbc.username=root
    jdbc.password=zzzs12138
    ```

mybatis-config.xml

    ```
    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
    <configuration>
    <!--  配置数据源，交给spring去做  -->
        <typeAliases>
            <package name="com.athos.entity"/>
        </typeAliases>
    <!--绑定mapper.xml-->
        <mappers>
            <mapper class="com.athos.mapper.BooksMapper"/>
        </mappers>
    </configuration>
    ```

## 二、整合spring层

spring-dao.xml

```
    <?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                            http://www.springframework.org/schema/beans/spring-beans.xsd
                            http://www.springframework.org/schema/context
                            https://www.springframework.org/schema/context/spring-context.xsd">

<!--  关联数据库配置文件  -->
    <context:property-placeholder location="classpath:database.properties"/>
<!--  连接池设置  -->
    <bean id="datasource" class="com.alibaba.druid.pool.DruidDataSource" init-method="init" destroy-method="clone">
        <property name="driverClassName" value="${jdbc.driver}"/>
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
        <!-- 配置初始化大小、最小、最大 -->
        <!-- 通常来说，只需要修改initialSize、minIdle、maxActive -->
        <!-- 初始化时建立物理连接的个数，缺省值为0 -->
        <property name="initialSize" value="5"/>
        <!-- 最小连接池数量 -->
        <property name="minIdle" value="5"/>
        <!-- 最大连接池数量，缺省值为8 -->
        <property name="maxActive" value="20"/>
        <!-- 获取连接时最大等待时间，单位毫秒。配置了maxWait之后，缺省启用公平锁，并发效率会有所下降，如果需要可以通过配置useUnfairLock属性为true使用非公平锁。 -->
        <property name="maxWait" value="60000"/>
        <!-- 添加此处作用是为了在SQL监控页面显示sql执行语句，不配置不显示 -->
        <property name="filters" value="stat,wall,log4j" />
    </bean>
<!--  SQLSessionFactory  -->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <!--    注入数据源    -->
        <property name="dataSource" ref="datasource"/>
        <!--    绑定mybatis配置文件    -->
        <property name="configLocation" value="classpath:mybatis-config.xml"/>
    </bean>
<!--  配置mapper接口扫描包,动态实现mapper接口注入到spring容器中-->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <!--    注入sqlSessionFactory    -->
        <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/>
        <!--    要扫描的mapper包    -->
        <property name="basePackage" value="com.athos.mapper"/>

    </bean>

 </beans>
```

spring-service.xml
```
    <?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                            http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">
    <!--1.扫描service的包-->
    <context:component-scan base-package="com.athos.service"/>
    <!--  2.将我们所有的业务类注入到spring中，可以通过配置也可以通过注解实现  -->
    <bean id="BookServiceImpl" class="com.athos.service.impl.BookServiceImpl">
        <property name="booksMapper" ref="booksMapper"/>
    </bean>
    <!--  3 声明事务配置  -->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <!--    注入数据源    -->
        <property name="dataSource" ref="datasource"/>
    </bean>
</beans>
```


## 三、整合springMVC层

1. 增加项目web支持-add framework support
2. 配置web.xml文件
```
    <?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
         http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">
<!--  1.DispatcherServlet  -->
    <servlet>
        <servlet-name>springmvc</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:applicationContext.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>springmvc</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>
<!--  2.乱码过滤  -->
<!--    <filter>-->
<!--        <filter-name>encodingFilter</filter-name>-->
<!--        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>-->
<!--        <init-param>-->
<!--            <param-name>encoding</param-name>-->
<!--            <param-value>utf-8</param-value>-->
<!--        </init-param>-->
<!--    </filter>-->
<!--    <filter-mapping>-->
<!--        <filter-name>encodingFilter</filter-name>-->
<!--        <url-pattern>/*</url-pattern>-->
<!--    </filter-mapping>-->
<!--      设置session过期时间:minutes  -->
    <session-config>
        <session-timeout>15</session-timeout>
    </session-config>
</web-app>
```

spring-mvc.xml

```
    <?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                            http://www.springframework.org/schema/beans/spring-beans.xsd
                            http://www.springframework.org/schema/mvc
                            http://www.springframework.org/schema/mvc/spring-mvc.xsd
                            http://www.springframework.org/schema/context
                            https://www.springframework.org/schema/context/spring-context.xsd">

<!--    1.注解驱动-->
    <mvc:annotation-driven/>
<!--    2.静态资源过滤-->
    <mvc:default-servlet-handler/>
<!--    3，扫描包controller-->
    <context:component-scan base-package="com.athos.controller"/>
<!--    4.视图解析器-->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/jsp/"/>
        <property name="suffix" value=".jsp"/>
    </bean>
</beans>
```



## 四、controller 层和jsp层交互起来就可以实现网站