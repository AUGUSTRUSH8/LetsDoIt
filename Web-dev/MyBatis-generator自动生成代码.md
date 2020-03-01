### MyBatis-generator 自动生成 MyBatis 代码

#### 第一步

IDEA创建maven工程

#### 第二步

在maven项目的pom.xml 添加mybatis-generator-maven-plugin 插件和MySQL数据库驱动依赖

```xml
<build>
  <plugins>
    <plugin>
      <groupId>org.mybatis.generator</groupId>
      <artifactId>mybatis-generator-maven-plugin</artifactId>
      <version>1.3.5</version>
      <configuration>
        <verbose>true</verbose>
        <overwrite>true</overwrite>
      </configuration>
    </plugin>
  </plugins>
</build>

<dependency>
  <groupId>mysql</groupId>
  <artifactId>mysql-connector-java</artifactId>
  <version>5.1.34</version>
</dependency>
```

#### 第三步

在maven项目下的src/main/resources 目录下建立名为 generatorConfig.xml的配置文件，作为mybatis-generator-maven-plugin 插件的执行目标，模板如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">

<generatorConfiguration>
    <!-- 配置mysql 驱动jar包路径.用了绝对路径 -->
    <classPathEntry location="D:/apache-maven-3.5.4/localRepository/mysql/mysql-connector-java/5.1.39/mysql-connector-java-5.1.39.jar" />

    <context id="wangyongzhi_mysql_tables" targetRuntime="MyBatis3">
        <!-- 防止生成的代码中有很多注释，加入下面的配置控制 -->
        <commentGenerator>
            <property name="suppressAllComments" value="true" />
            <property name="suppressDate" value="true" />
        </commentGenerator>

        <!-- 数据库连接 -->
        <jdbcConnection driverClass="com.mysql.jdbc.Driver"
                        connectionURL="jdbc:mysql://127.0.0.1:3306/admin?useUnicode=true&amp;characterEncoding=UTF-8"
                        userId="root"
                        password="123">
        </jdbcConnection>

        <javaTypeResolver >
            <property name="forceBigDecimals" value="false" />
        </javaTypeResolver>

        <!-- 数据表对应的model层  -->
        <javaModelGenerator targetPackage="com.admin.generator.domain" targetProject="src">
            <property name="enableSubPackages" value="true" />
            <property name="trimStrings" value="true" />
        </javaModelGenerator>

        <!-- sql mapper 映射配置文件 -->
        <sqlMapGenerator targetPackage="com.admin.generator.mapper"  targetProject="src">
            <property name="enableSubPackages" value="true" />
        </sqlMapGenerator>

        <!-- mybatis3中的mapper接口 -->
        <javaClientGenerator type="XMLMAPPER" targetPackage="com.admin.generator.dao"  targetProject="src">
            <property name="enableSubPackages" value="true" />
        </javaClientGenerator>

        <!-- 数据表进行生成操作 schema:相当于库名; tableName:表名; domainObjectName:对应的DO -->
        <table schema="admin" tableName="info" domainObjectName="InvoiceInfo"
               enableCountByExample="false" enableUpdateByExample="false"
               enableDeleteByExample="false" enableSelectByExample="false"
               selectByExampleQueryId="false">
        </table>

    </context>
</generatorConfiguration>
```

#### 第四步

使用maven运行mybatis-generator-maven-plugin插件：工程名->Plugins->mybatis-generator->mybatis-generator:generate->Run Maven Build

**All Done!**