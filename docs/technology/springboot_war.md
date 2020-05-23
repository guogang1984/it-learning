# springboot工程 war打包发布
> springboot工程切换到war包方式，同时支持jar,war可通过mvn的profile来处理。

## 继承SpringBootServletInitializer
```java
import org.springframework.boot.builder.SpringApplicationBuilder;
import org.springframework.boot.web.servlet.support.SpringBootServletInitializer;

public class XxxServletInitializer extends SpringBootServletInitializer
{
    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder application)
    {
        return application.sources(XxxApplication.class);
    }
}
```
> 其中 XxxApplication.class是工程自动生成的main启动类
> 
> SpringBootServletInitializer实现初始化web.xml作用

## 调整 pom.xml 文件
> 新定义两个属性切换packaging和scope ， 例如： tomcat.packaging，tomcat.scope

```xml
<!-- 第一处修改 -->
<packaging>${tomcat.packaging}</packaging>

<!-- 第二处修改 -->
<properties>
    <!-- war properties -->
    <tomcat.packaging>jar</tomcat.packaging>
    <tomcat.scope>compile</tomcat.scope>
    <!-- other properties -->
    ...
</properties>

<!-- 第三处修改 -->
<dependencies>
    <!-- 新增如下依赖-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-tomcat</artifactId>
        <scope>${tomcat.scope}</scope>
    </dependency>
    <dependency>
        <groupId>org.apache.tomcat.embed</groupId>
        <artifactId>tomcat-embed-jasper</artifactId>
        <scope>${tomcat.scope}</scope>
    </dependency>
</dependencies>

<!-- 第四处修改 -->
<profiles>
    <profile>
        <id>jar</id>
        <activation>
            <activeByDefault>true</activeByDefault>
        </activation>
        <properties>
            <tomcat.packaging>jar</tomcat.packaging>
            <tomcat.scope>compile</tomcat.scope>
        </properties>
   </profile>
   <profile>
        <id>war</id>
        <properties>
            <tomcat.packaging>war</tomcat.packaging>
            <tomcat.scope>provided</tomcat.scope>
        </properties>
   </profile>
</profiles>
```

## 执行打包

> 默认打包jar

```bash
mvn clean package
```

> 打包war

```bash
mvn clean package -Pwar
```

## 其他举例：按照场景(dev, alpha, beta, prod)进行配置

### 调整 springboot 的各个场景配置

```yml
#################################################################
# 共4个配置
# dev   开发配置 一般用于开发人员自行调整
# alpha 内测配置 一般用于指使用公司内测环境(window),采用tomcat方式部署
# beta  公测配置 一般用于指使用客户公测环境(centos7),采用容器化方式部署
# prod  正式配置 一般用于指使用客户正式环境(centos7),采用容器化方式部署
# 命令
# eclipse 找到 启动类，增加 --spring.profiles.active=dev 进行启动
# mvn spring-boot:run -D"spring-boot.run.profiles"="dev"
# java -jar target/xxxx.jar --spring.profiles.active=prod
#################################################################

---
# dev   开发配置 一般用于开发人员自行调整
spring.profiles: dev
# spring.profiles.include: xxx, dev-xxx

---
# alpha 内测配置 
spring.profiles: alpha

---
# beta  公测配置 
spring.profiles: beta

---
# prod  正式配置 
spring.profiles: prod
 
```

### 调整profiles，对应 springboot 的各个场景
```xml
<profiles>
    <profile>
        <!-- 开发版 -->
        <id>dev</id>
        <properties>
            <spring.profiles.active>dev</spring.profiles.active>
            <!-- spring-boot generate fat jar -->
            <tomcat.packaging>jar</tomcat.packaging>
            <tomcat.scope>compile</tomcat.scope>
        </properties>
        <dependencies>
            <!-- spring-boot-devtools -->
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-devtools</artifactId>
                <optional>true</optional> <!-- 表示依赖不会传递 -->
            </dependency>
        </dependencies>
   </profile>
   <profile>
        <!-- 内测版 -->
        <id>alpha</id>
        <properties>
            <spring.profiles.active>alpha</spring.profiles.active>
            <!-- spring-boot generate war -->
            <tomcat.packaging>war</tomcat.packaging>
            <tomcat.scope>provided</tomcat.scope>
        </properties>
   </profile>
   <profile>
        <!-- 公测版 -->
        <id>beta</id>
        <properties>
            <spring.profiles.active>beta</spring.profiles.active>
            <!-- spring-boot generate war -->
            <tomcat.packaging>war</tomcat.packaging>
            <tomcat.scope>provided</tomcat.scope>
        </properties>
   </profile>
   <profile>
        <!-- 正式版 -->
        <id>prod</id>
        <activation>
            <activeByDefault>true</activeByDefault>
        </activation>
        <properties>
            <spring.profiles.active>prod</spring.profiles.active>
            <!-- spring-boot generate war -->
            <tomcat.packaging>war</tomcat.packaging>
            <tomcat.scope>provided</tomcat.scope>
        </properties>
   </profile>
</profiles>
```

### 写入spring.profiles.active
>保证war包或jar包默认的启动场景, pom.xml新增了spring.profiles.active 属性，可在打包后写入到yml,properties等spring配置文件。
>注意：spring-boot-maven-plugin 插件, 提供了mvn属性使用'@属性名@'写入方式。

```yml
spring.profiles.active: @spring.profiles.active@
```


### 修改资源插件，写入spring.profiles.active
>修改资源插件的delimiters方式

```xml
<plugin>
   <groupId>org.apache.maven.plugins</groupId>
   <artifactId>maven-resources-plugin</artifactId>
   <version>${maven-resources-plugin.version}</version>
   <executions>
      <execution>
         <id>default-resources</id>
         <phase>validate</phase>
         <goals>
            <goal>copy-resources</goal>
         </goals>
         <configuration>
            <outputDirectory>target/classes</outputDirectory>
            <useDefaultDelimiters>false</useDefaultDelimiters>
            <delimiters>
               <delimiter>#</delimiter>
            </delimiters>
            <resources>
               <resource>
                  <directory>src/main/resources/</directory>
                  <filtering>true</filtering>
                  <includes>
                     <include>**/*.xml</include>
                     <include>**/*.yml</include>
                  </includes>
               </resource>
               <resource>
                  <directory>src/main/resources/</directory>
                  <filtering>false</filtering>
                  <excludes>
                     <exclude>**/*.xml</exclude>
                     <exclude>**/*.yml</exclude>
                  </excludes>
               </resource>
            </resources>
         </configuration>
      </execution>
   </executions>
</plugin>
```

通过资源插件的重定义，我们可以使用'#属性名#'写入。
```yml
spring.profiles.active: #spring.profiles.active#
```