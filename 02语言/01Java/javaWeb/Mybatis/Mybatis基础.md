### 概述

**Mybatis概念**
>- MyBatis 是一款优秀的持久层框架，用于简化 JDBC 开发
>- MyBatis 本是 Apache 的一个开源项目iBatis, 2010年这个项目由apache software foundation 迁移到了google code，并且改名为MyBatis 。2013年11月迁移到Github
>- [官网 | 简介](https://mybatis.org/mybatis-3/zh/index.html)

**持久层**
- 负责将数据保存到数据库的那一层代码，而Mybatis就是对*jdbc*代码进行了封装
- JavaEE三层架构：表现层，业务层，持久层

### 基础操作

**创建模块，导入坐标**
在创建好的模块中的*pom.xml*配置文件中添加依赖的坐标
```xml
<!--mybatis 依赖-->
<dependency>
	<groupId>org.mybatis</groupId>
	<artifactId>mybatis</artifactId>
	<version>3.5.5</version>
</dependency>
```

**编写 Mybatis 核心配置文件**
在模块的 *resources* 目录下创建*Mybatis*的配置文件 *mybatis-config.xml*
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
		PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
		"http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>

	<typeAliases>
		<package name="com.itheima.pojo"/>
	</typeAliases>
	
	<!--
	environments：配置数据库连接环境信息。可以配置多个environment，通过default属性切换不同的
	environment
	-->
	<environments default="development">
	
		<environment id="development">
			<transactionManager type="JDBC"/>
			
			<dataSource type="POOLED">
			<!--数据库连接信息-->
				<property name="driver" value="com.mysql.jdbc.Driver"/>
				<property name="url" value="jdbc:mysql:///mybatis?useSSL=false"/>
				<property name="username" value="root"/>
				<property name="password" value="1234"/>
			</dataSource>
			
		</environment>
		
		<environment id="test">
			<transactionManager type="JDBC"/>
			
			<dataSource type="POOLED">
			<!--数据库连接信息-->
				<property name="driver" value="com.mysql.jdbc.Driver"/>
				<property name="url" value="jdbc:mysql:///mybatis?useSSL=false"/>
				<property name="username" value="root"/>
				<property name="password" value="1234"/>
			</dataSource>
			
		</environment>
	</environments>
	
	<mappers>
	<!--加载sql映射文件-->
	<mapper resource="UserMapper.xml"/>
	</mappers>
	
</configuration>
```

**注意**
如果Mapper接口名称和SQL映射文件名称相同，并在同一目录下，则可以使用包扫描的方式简化SQL映射文件的加载。也就是将核心配置文件的加载映射配置文件的配置修改为：
```xml
<mappers>
	<!--加载sql映射文件-->
	<!-- <mapper resource="com/itheima/mapper/UserMapper.xml"/>-->
	<!--Mapper代理方式-->
	<package name="com.itheima.mapper"/>
</mappers>
```

**编写SQL映射文件**
在模块的*resources*目录下创建映射配置文件*UserMapper.xml*,对*sql*语句进行统一管理
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="test">
	<select id="selectAll" resultType="com.itheima.pojo.User">
		select * from tb_user;
	</select>
</mapper>
```