# spring boot 问题随笔
## 在使用 maven 时出现 Invalid packaging for parent pom.xml, must be _pom_ but is _xxx 类问题的处理
 pom 文件中添加 ```<packaging>pom</packaging>```这行代码指定打包方式即可,但是这个只是存在于父模块的 pom 文件中,子模块还是要jar 或者 war ,或者直接不指定设置默认即可

## There was an unexpected error (type=Not Found, status=404).
包路径不对,application.java 文件的包必须是项目下的父路径，其他类的包路径必须是其子路径.

## maven 默认过滤静态资源,如果不配置,就会导致status = 500 报错
1. 静态资源过滤需要在 pom 文件的 build 标签添加<br>
```
<!--        maven静态资源过滤设置-->
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
                <directory>src/main/resources</directory>
                <includes>
                    <include>**/*.properties</include>
                    <include>**/*.xml</include>
                    <include>**/*.html</include>
                </includes>
                <filtering>false</filtering>
            </resource>
        </resources>
```

<br>

2. 如果第一条添加以后还是找不到,确认是否子模块中 pom 文件中指定打包方式为 pom.如果是,删掉即可