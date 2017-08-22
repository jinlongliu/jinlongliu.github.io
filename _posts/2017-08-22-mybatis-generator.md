---
layout: post
title: "Mybatis generator使用"
description: ""
category: Java 
tags: [Spring]
---
{% include JB/setup %}
**<font color="red">版权声明：本文为博主原创文章，未经博主授权不得转载。</font>**

### 配置Mybatis-generator 插件
{% highlight bash %}
#maven依赖，插件部分
    <build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
			<plugin>
				<groupId>org.mybatis.generator</groupId>
				<artifactId>mybatis-generator-maven-plugin</artifactId>
				<version>1.3.5</version>
			</plugin>
		</plugins>
	</build>
	
#使用MySQL数据库需要添加驱动依赖
	<dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <scope>runtime</scope>
    </dependency>
{% endhighlight %}

### mybatis-generator配置文件，存放路径spring-boot的resources
{% highlight bash %}

<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">

<generatorConfiguration>
    <!--MBG运行时加载额外包的路径，比如JDBC drivers；jar，zip压缩文件或者是加入到classpath的路径-->
    <classPathEntry location="D:\02maven_repo\mysql\mysql-connector-java\5.1.43\mysql-connector-java-5.1.43.jar" />

    <!--context 严格顺序-->
    <!--(property*,plugin*,commentGenerator?,(connectionFactory|jdbcConnection),javaTypeResolver?,
    javaModelGenerator,sqlMapGenerator?,javaClientGenerator?,table+)-->
    <context id="MySQLTables" targetRuntime="MyBatis3">

        <!--插件-->
        <plugin type="org.mybatis.generator.plugins.EqualsHashCodePlugin"></plugin>
        <plugin type="org.mybatis.generator.plugins.SerializablePlugin"></plugin>
        <plugin type="org.mybatis.generator.plugins.ToStringPlugin"></plugin>
        <!--分页-->
        <plugin type="org.mybatis.generator.plugins.RowBoundsPlugin"></plugin>

        <!--取消注释-->
        <commentGenerator>
            <property name="suppressAllComments" value="true"/>
        </commentGenerator>

        <!--数据库连接信息-->
        <jdbcConnection driverClass="com.mysql.jdbc.Driver"
                        connectionURL="jdbc:mysql://172.18.60.121:3306/liujinlong"
                        userId="root"
                        password="123qwe!@#QWE">
        </jdbcConnection>

        <javaTypeResolver >
            <property name="forceBigDecimals" value="false" />
        </javaTypeResolver>

        <!--Model生成包路径，项目路径-->
        <javaModelGenerator targetPackage="org.playchain.demo4.model" targetProject="D:\01workspace\demo4\src\main\java">
            <property name="enableSubPackages" value="true" />
            <property name="trimStrings" value="true" />
            <property name="constructorBased" value="true" />
        </javaModelGenerator>

        <!--mapper XML生成信息-->
        <sqlMapGenerator targetPackage="mapper"  targetProject="D:\01workspace\demo4\src\main\resources">
            <property name="enableSubPackages" value="true" />
        </sqlMapGenerator>

        <!--mapper 接口文件-->
        <javaClientGenerator type="XMLMAPPER" targetPackage="org.playchain.demo4.mapper"  targetProject="D:\01workspace\demo4\src\main\java">
            <property name="enableSubPackages" value="true" />
        </javaClientGenerator>

        <table schema="DB2ADMIN" tableName="user" domainObjectName="User" >
            <property name="useActualColumnNames" value="true"/>
            <!--主键-->
            <generatedKey column="id" sqlStatement="MySql" identity="true" />
            <!--重写，忽略字段-->
            <columnOverride column="DATE_FIELD" property="startDate" />
            <!--忽略字段-->
            <ignoreColumn column="FRED" />
            <columnOverride column="LONG_VARCHAR_FIELD" jdbcType="VARCHAR" />
        </table>

    </context>
</generatorConfiguration>


{% endhighlight %}

### 应用新增扫描注解
{% highlight bash %}

@SpringBootApplication
@MapperScan("org.playchain.demo4.mapper")
public class DemoApplication {
	public static void main(String[] args) {
		SpringApplication.run(DemoApplication.class, args);
	}
}

{% endhighlight %}

{% include JB/setup %}


