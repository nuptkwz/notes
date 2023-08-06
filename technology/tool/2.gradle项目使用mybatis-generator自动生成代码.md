@[toc]
# 前言
> mybatis-generator工具是用来生成mybatis的model,mapper,dao持久层代码。本文结合现在主流的构建工具是gradle，连接数据库自动生成相应代码。虽然mybatis-generator没有提供gradle的插件，但是可以用gradle调用ant任务，因此，gradle也能间接启动mybatis-generator。

# 环境
 - **JDK 1.8**
 - **IntelliJ IDEA 2017.2.2**
 - **Gradle 3.5.0**
 - **mysql**

# 配置
以shopmall-order订单服务为例，通过配置shopmall-order.gradle和generator.xml两个文件，来自动生成相应entity、mapper、xml等。

### shopmall-order.gradle
```
description = '''shopmall-order'''

dependencies {
    compile project(":shopmall-order-api")
    compile project(':shopmall-account-api')

}

//mybatis-generator.xml 配置路径
// mac下是找不到 ./src 路径的，需要全路径，如下配置。windows则为src/main/resources/generator.xml
mybatisGenerator {
    verbose = true
    configFile = 'src/main/resources/generator.xml'
}
```

 ### generator.xml
 ```
 <?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
  PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
  "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
<generatorConfiguration>
  <context id="mybatis" targetRuntime="MyBatis3">

    <!--自动实现Serializable接口-->
    <plugin type="org.mybatis.generator.plugins.SerializablePlugin"></plugin>

    <!-- 去除自动生成的注释 -->
    <commentGenerator>
      <property name="suppressAllComments" value="true"/>
    </commentGenerator>

    <!--数据库基本信息-->
    <jdbcConnection driverClass="com.mysql.jdbc.Driver"
      connectionURL="jdbc:mysql://10.223.25.82:3306/shopmall"
      userId="root"
      password="123456">
    </jdbcConnection>

    <javaTypeResolver>
      <property name="forceBigDecimals" value="true"/>
      <property name="useJSR310Types" value="true"/>
    </javaTypeResolver>

    <!--生成实体类的位置以及包的名字-->
    <!--同样Mac用户：targetProject 为全路径-->
    <javaModelGenerator targetPackage="com.ireeder.order.domain.entity"
      targetProject="src/main/java">
      <!-- enableSubPackages:是否让schema作为包的后缀 -->
      <property name="enableSubPackages" value="true"/>
      <!-- 从数据库返回的值被清理前后的空格 -->
      <property name="trimStrings" value="true"/>
    </javaModelGenerator>

    <!--生成映射文件存放位置-->
    <!--同样Mac用户：targetProject 为全路径-->
    <sqlMapGenerator targetPackage="mapper"
      targetProject="src/main/resources">
      <!-- enableSubPackages:是否让schema作为包的后缀 -->
      <property name="enableSubPackages" value="false"/>
    </sqlMapGenerator>

    <!--生成Dao类存放位置,mapper接口生成的位置-->
    <!--同样Mac用户：targetProject 为全路径-->
    <javaClientGenerator type="XMLMAPPER" targetPackage="com.shopmall.order.domain.mapper"
      targetProject="src/main/java">
      <!-- enableSubPackages:是否让schema作为包的后缀 -->
      <property name="enableSubPackages" value="false"/>
    </javaClientGenerator>

    <table schema="shopmall" tableName="coupon_item"
      domainObjectName="CouponItemEntity" enableCountByExample="false"
      enableDeleteByExample="false" enableSelectByExample="false"
      selectByPrimaryKeyQueryId="true"
      enableUpdateByExample="false">
      <columnOverride column="updated_at" javaType="java.time.LocalDateTime"/>
      <columnOverride column="created_at" javaType="java.time.LocalDateTime"/>
      <columnOverride column="validity" javaType="java.lang.Boolean"/>
    </table>
  </context>
</generatorConfiguration>
```