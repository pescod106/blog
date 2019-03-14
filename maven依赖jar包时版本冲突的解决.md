# maven依赖jar包时版本冲突的解决

共有四种解决方式：

## 声明优先原则

在pom.xml配置文件中，如果有两个名称相同版本不同的依赖声明，那么先写的会生效。 
所以，先声明自己要用的版本的jar包即可。

## 路径近者优先

直接依赖优先于传递依赖，如果传递依赖的jar包版本冲突了，那么可以自己声明一个指定版本的依赖jar，即可解决冲突。

## 排出原则

传递依赖冲突时，可以在不需要的jar的传递依赖中声明排除，从而解决冲突。
例子：

```
<dependency>
    <groupId>org.apache.struts</groupId>
    <artifactId>struts2-spring-plugin</artifactId>
    <version>2.3.24</version>
    <exclusions>
      <exclusion>
        <groupId>org.springframework</groupId>
        <artifactId>spring-beans</artifactId>
      </exclusion>
    </exclusions>
</dependency>
```

## 版本锁定原则（最常使用）

在配置文件pom.xml中先声明要使用哪个版本的相应jar包，声明后其他版本的jar包一律不依赖。解决了依赖冲突。
例子：

```
<properties>
    <spring.version>4.2.4.RELEASE</spring.version>
    <hibernate.version>5.0.7.Final</hibernate.version>
    <struts.version>2.3.24</struts.version>
</properties>
    <!-- 锁定版本，struts2-2.3.24、spring4.2.4、hibernate5.0.7 -->
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>${spring.version}</version>
        </dependency>
    </dependencies>
</dependencyManagement>
```