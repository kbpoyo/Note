---
number headings: first-level 1, max 6, 1.1
---

# 1 安装MySql

## 1.1 下载

**下载地址**
[mysql下载](https://downloads.mysql.com/archives/community/)
![](http://imgs.kbpoyo.top/imgs/数据库_202206241406218.png)

## 1.2 解压

**解压到自选目录**
![](http://imgs.kbpoyo.top/imgs/数据库_202206241408819.png)

# 2 配置MySql

## 2.1 添加环境变量

**进入环境变量设置窗口**
![](http://imgs.kbpoyo.top/imgs/数据库_202206241410721.png)

**在系统变量中新建变量<u> MYSQL_HOME</u>**
变量值为解压路径
![](http://imgs.kbpoyo.top/imgs/数据库_202206241413835.png)

**在path中添加bin目录的路径**
选中*path*
![](http://imgs.kbpoyo.top/imgs/数据库_202206241416737.png)

添加 *%MYSQL_HOME%\\bin*
![](http://imgs.kbpoyo.top/imgs/数据库_202206241418051.png)

## 2.2 新建配置文件

**新建一个文本**
```config
[mysqld]
port = 3306
basedir=E:\mysql-5.7.24-winx64
datadir=E:\mysql-5.7.24-winx64\data
max_connections=200
character-set-server=utf8
default-storage-engine=INNODB
sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES

[mysql]
default-character-set=utf8

```

> [!info]+ 关键词含义
> - \[mysqld]
> 	- **port**: 开放数据库程序端口(默认3306)
> 	- **basedir**: MySql的根目录   
> 	- **datadir**: data数据库生成路径
> 	- **max_connections**: 与客户端的最大连接数
> 	- **character-set-server**: 设置mysql内部字符集
> 	- **default-storage-engine**: 设置默认存储引擎
> - \[mysql]
> 	- **default-character-set**: 设置默认字符集

保存文件名为*my.ini*，存放发在*MySql*的根目录(解压目录)
> [!important]  
> **basedir** 和 **datadir** 需要根据自己的安装目录进行修改


## 2.3 初始化MySql

在命令行提示符中输入指令：**mysqld --initialize-insecure**
![](http://imgs.kbpoyo.top/imgs/数据库_202206241554876.png)


## 2.4 注册MySql服务

输入命令：**mysql -install**
![](http://imgs.kbpoyo.top/imgs/数据库_202206241558723.png)


## 2.5 启动MySql服务

输入命令： **net start mysql**
![](http://imgs.kbpoyo.top/imgs/数据库_202206241600840.png)


## 2.6 修改默认账户密码

输入命令： **mysqladmin -u root password 1234**
1234指设置默认管理员的密码为1234，可自行更改
![](http://imgs.kbpoyo.top/imgs/数据库_202206241602644.png)


# 3 登录和退出

## 3.1 登录

![](http://imgs.kbpoyo.top/imgs/数据库_202206241604463.png)


## 3.2 退出

**退出命令**
- exit
- quit


# 4 MySql数据库模型

## 4.1 关系型数据库

![](http://imgs.kbpoyo.top/imgs/数据库_202206241611748.png)


## 4.2 数据模型

**数据库和表**
- 我们可以在客户端上，用*sql*语句通过*DBMS*创建数据库和数据表
- 一个数据库对应磁盘上的一个文件夹，一张表则对应一个文件
- 一个数据库下可以创建多张表
![](http://imgs.kbpoyo.top/imgs/数据库_202206241616390.png)



## 4.3 存储形式

**MySql中自带的mysql数据库**
![](http://imgs.kbpoyo.top/imgs/数据库_202206241621477.png)

- MySql中可以创建多个数据库，每个数据库对应磁盘上的一个文件夹
- 每个数据库中可以创建多张表，每张表对应到磁盘上的一个 **.frm**文件
- 每张表可以存储多条数据，数据会被存储到磁盘中的 **.MYD**文件中
- **.frm**：表文件
- **.MYD**：数据文件


# 5 卸载MySql

**卸载步骤**
> 1. 停止mysql：net stop mysql
> 2. 卸载mysql：mysqld -remove mysql
> 3. 删除MySql目录及其相关的环境变量


![](http://imgs.kbpoyo.top/imgs/数据库_202206241630419.png)