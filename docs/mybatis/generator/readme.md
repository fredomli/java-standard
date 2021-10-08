# Mybatis Generator的使用

## 1. Mybatis 的诞生背景

虽然MyBatis是一个简单易学的框架，但是配置XML文件也是一件相当繁琐的一个过程，而且会出现很多不容易定位的错误。当在工作中需要生成大量对象的时候，有太多的重复劳动，简直是生无可恋。所以，官方开发了 MyBatis Generator。它只需要很少量的简单配置，就可以完成大量的表到Java对象的生成工作，拥有零出错和速度快的优点，让开发人员解放出来更专注于业务逻辑的开发。


## 2. Mybatis Generator简介
官网的Mybatis Generator使用介绍，点击下方链接：

[http://mybatis.org/generator/index.html](http://mybatis.org/generator/index.html)

Mybatis Generator，简称MBG，发布时间为：2019年11月24日。

Mybatis Generator生成的文件包含三类：
* Model实体文件，一个数据库表对应生成一个Model实体文件。
* Mapper接口文件，数据操作方法都在此接口中定义。
* Mapper XML配置文件。

## 3.Mybatis Generator的使用介绍
下载Mybatis Generator之后，配置`generatorConfig.xml`文件，然后通过命令行就可以运行
Mybatis Generator如下所示：
```shell
java -jar mybatis-generator-core-x.x.x.jar -configfile \temp\generatorConfig.xml -overwrite
```
当然，MyBatis Generator的使用还可以有其他多种方式，如下所示：

```text
MyBatis Generator (MBG) can be run in the following ways:

From the command prompt with an XML configuration
As an Ant task with an XML configuration
As a Maven Plugin
From another Java program with an XML configuration
From another Java program with a Java based configuration
As an Eclipse Feature
```

具体的介绍，请移步官方文档：[http://mybatis.org/generator/running/running.html](http://mybatis.org/generator/running/running.html

## 4.MyBatis generatorConfig 配置文件介绍：  
[http://mybatis.org/generator/configreference/xmlconfig.html](http://mybatis.org/generator/configreference/xmlconfig.html)  

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
  PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN""http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">

<generatorConfiguration>
  <classPathEntry location="/Program Files/IBM/SQLLIB/java/db2java.zip" />

  <context id="DB2Tables" targetRuntime="MyBatis3">
    <jdbcConnection driverClass="COM.ibm.db2.jdbc.app.DB2Driver"
        connectionURL="jdbc:db2:TEST"
        userId="db2admin"
        password="db2admin">
    </jdbcConnection>

    <javaTypeResolver >
      <property name="forceBigDecimals" value="false" />
    </javaTypeResolver>

    <javaModelGenerator targetPackage="test.model" targetProject="\MBGTestProject\src">
      <property name="enableSubPackages" value="true" />
      <property name="trimStrings" value="true" />
    </javaModelGenerator>

    <sqlMapGenerator targetPackage="test.xml"  targetProject="\MBGTestProject\src">
      <property name="enableSubPackages" value="true" />
    </sqlMapGenerator>

    <javaClientGenerator type="XMLMAPPER" targetPackage="test.dao"  targetProject="\MBGTestProject\src">
      <property name="enableSubPackages" value="true" />
    </javaClientGenerator>

    <table schema="DB2ADMIN" tableName="ALLTYPES" domainObjectName="Customer" >
      <property name="useActualColumnNames" value="true"/>
      <generatedKey column="ID" sqlStatement="DB2" identity="true" />
      <columnOverride column="DATE_FIELD" property="startDate" />
      <ignoreColumn column="FRED" />
      <columnOverride column="LONG_VARCHAR_FIELD" jdbcType="VARCHAR" />
    </table>

  </context>
</generatorConfiguration>
```
### Mybatis 中文文档
[https://mybatis.org/mybatis-3/zh/index.html](https://mybatis.org/mybatis-3/zh/index.html)  

*(☞ﾟヮﾟ)☞[返回首页README.md address](https://github.com/fredomli/java-standard)*
