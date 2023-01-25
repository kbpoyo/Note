# Sql简介

![[数据库基础概念#^5e64ac]]

# 通用语法

**Sql语句可以单行或多行书写，以英文分号结尾**
![](http://imgs.kbpoyo.top/imgs/数据库_202206241646248.png)

**MySql中Sql语句不区分大小写，关键字建议大写**
![](http://imgs.kbpoyo.top/imgs/数据库_202206241652177.png)

**注释**
- <u>--</u>：语句末尾注释，需要有一个空格
- <u>#</u>：语句末尾注释，不需要空格

![](http://imgs.kbpoyo.top/imgs/数据库_202206241705557.png)


# 语句分类

## DDL

### 简介

**DDL：** （Data Definition Language）用来操作数据库对象：数据库，表，列等
![](http://imgs.kbpoyo.top/imgs/数据库_202206241733743.png)

### 操作数据库

> [!info]  sql语句
> - 查询所有数据库：**show databases**
> - 创建数据库：**create database (if not exists)** + 数据库名称 
> - 删除数据库：**drop database (if exists)** + 数据库名称
> - 使用数据库：**use** + 数据库名称
> - 查看当前使用的数据库：**select database()**


### 操作表

> [!info]  sql语句
> - 查询表：**show tables**
> - 查询表结构：**DESC** + 表名称
> - 创建表
> > [!note]+ 创建表语法
> > ```sql
> > CREATE TABLE 表名 (
> >字段名1    数据类型1，
> >字段名2   数据类型2，
> > ·····
> > 字段名n    数据类型n
> > )；
> > 
> > ```
> > >  **最后一行末尾，不加逗号**
> > 
> > > [!info] 数据类型
> > > - 数值
> > > 	- **tinyint**：小整数型，占一个字节
> > > 	- **int**：大整数型，占四个字节
> > > 		- eg：age *int*
> > > 	- **double**：浮点类型，占八个字节
> > > 		- 使用格式：字段名 *double*(总长度，小数点后保留的位数)
> > > 		- eg：score *double*(5,2)
> > > - 日期
> > > 	- **date**：日期值，只包含年月日
> > > 		- eg：birthday *date*
> > > 	- **datetime**：混合日期和时间值，包含年月日时分秒
> > > - 字符串
> > > 	- **char**：定长字符串
> > > 		- 优点：存储性能高
> > > 		- 缺点：浪费空间
> > > 		- eg：name *char*(10) \#如果存储的数据字符个数不足10个，也会占用10个空间
> > > 	- **varchar**：变长字符串
> > > 		- 优点：节约空间
> > > 		- 缺点：存储性能低
> > > 		- eg：name *varchar*(10) \#如果存储的数据字符个数不足10个，那数据字符个数是几就占几个空间
> > 
> - 删除表
>  	- **DROP TABLE** + 表名
>  	- **DROP TABLE IF EXISTS** + 表名
>  - 修改表
> 	 - 修改表名
> 		 - **ALTER TABLE** + 表名 + **RENAME TO** + 新的表名
> 			 - eg：*alter table* student *rename to* stu
> 	- 添加一列
> 		- **ALTER TABLE** + 表名 + **ADD** + 列名 + 数据类型
> 			- eg：*alter table* stu *add* address *varchar*(50)
> 	- 修改数据类型
> 		- **ALTER TABLE** + 表名 + **MODIFY** + 列名 + 新数据类型
> 			- eg：*alter table* stu *modify* address *char*(50)
> 	- 修改列名和数据类型
> 		- **ALTER TABLE** + 表名 + **CHANGE** + 列名 + 新列名 + 新数据类型
> 			- eg：*alter table* stu *change* address addr *varchar*(50)
> 	- 删除列
> 		- **ALTER TABLE** + 表名 + **DROP** + 列名
> 			- eg：*alter table* + stu *drop* addr


## DML

### 简介

**DML**：（Data Manipulation Language）数据操作语言，用来对数据库中的表的数据进行增删改
![](http://imgs.kbpoyo.top/imgs/数据库_202206261437666.png)


### 添加数据

- 给指定列添加数据
	- **INSERT INTO** + 表名(列名1，列名2，.......) + **VALUES**(值1，值2，.......)
- 给全部列添加数据
	- **INSERT INTO** + 表名 + **VALUES**(值1，值2，.......)
- 批量添加数据
	- **INSERT INTO** + 表名(列名1，列名2，.......) + **VALUES**(值1，值2，.......)，(值1，值2，.......)，.......
	- **INSERT INTO** + 表名 + **VALUES**(值1，值2，.......)，(值1，值2，.......)，.......

### 修改数据

- **UPDATE**  +  表名 + **SET** + 列名1=值1，列名2=值2，.......(**WHERE** + 条件)
>**注意**：
>- 修改语句如果不加*where*条件，则所有的数据都将被修改
>- 括号中的内容可以省略，但要明确语句的影响结果

### 删除数据

- **DELETE FROM** + 表名 (**WHERE** + 条件)
>**注意**：
>若不加*where*条件，则所有数据都将被删除



## DQL

### 简介

**DQL**：（Data Query Language）数据查询语言，用来查询数据库中表所记录的数据

### 查询的完整语法

**格式**
```sql
select
	字段列表
from
	表名列表
where
	条件列表
group by
	分组字段
having
	分组后条件
order by
	排序字段
limit
	分页限定
```

### 基础查询语法

- 查询多个字段
	- **select** + 字段列表 + **from**  +表名
	- **select** + \* + **from** + 表名 \#查询所有字段数据
-  去除重复记录
	- **select distinct** + 字段列表 + **from** + 表名
- 起别名
	- **as**：as可以省略
		- eg：*select* stu *as* stu_name *from* stu_table


### 条件查询

#### 语法格式

**格式**
>**select** + 字段列表 + **from** + 表名 + **where** + 条件列表

#### 查询条件

**符号与功能**
![](http://imgs.kbpoyo.top/imgs/数据库_202206261531013.png)


### 排序查询

**格式**
>**select** + 字段列表 + **from** + 表名 + **order by** + 排序字段名1 (排序方式1)， 排序字段名2(排序方式2)，........

**排序方式**
> - **asc**：升序(默认值)
> - **desc**：降序

 **注意**
 >有多个排序条件时，当前边的条件值一样时，才会根据第二条条件进行排序


### 聚合函数

#### 简介

>**对一列数据作为一个整体，进行纵向运算，相当于查询表中某一字段的属性**

![[Pasted image 20220626155242.png]]

**注意**
>若位进行分组查询，则查询结果多为一行，即查询字段的属性

#### 分类

| 函数名       | 功能                 |
|:----------|:-------------------|
| count(列名) | 统计数量(一般选用不为null的列） |
| max(列名)   | 列最大值               |
| min(列名)   | 列最小值               |
| sum(列名)   | 列求和                |
| avg(列名)   | 列求平均值              |  

#### 语法格式

**格式**
>**select** + 聚合函数名(列名) + **from** + 表

**注意**
>*null*值不参与所有聚合函数的运算


## 分组查询

**格式**
>**select** + 字段列表 + **from** + 表名 + (**where** + 分组前条件查询) **group by** + 分组字段名 + (**having** + 分组后条件过滤)

**注意**
>分组之后，查询的字段为聚合函数和分组字段，查询其他字段无任何意义

**实列**
```sql
#查询男同学和女同学各自的数学平均分

select sex, avg(math) from stu group by sex;


#查询男女同学各自的数学平均分，以及人数

select sex, avg(math), count(*) from stu group by sex;


#查询男女同学各自的数学平均分，人数，且要求分数低于70分的不参与分组

select sex, avg(math), count(*) from stu where math > 70 group by sex;


#查询男女同学各自的数学平均分，人数，要求分数低于70分的不参与分组，且分组后人数要大于2个人

select sex, avg(math), count(*) as '人数' from stu where math > 70 group by sex having '人数' > 2
```

**where 和 having 的区别**
- 执行时机不一样
	- *where*是分组之前进行限定，满足条件的才能参与分组
	- *having*是分组之后对结果进行过滤
- 可判断的条件不一样
	- *where*不能对聚合函数进行判断
	- *having*可以对聚合函数进行判断


## 分页查询

**语法格式**
>**select** + 字段列表 + **from** + 表名 + **limit** + 起始索引，查询条目数
>>起始索引从0开始

**示例**
```sql
#每页显示3条数据，查询第一页
select * from stu limit 0,3;

#每页显示3条数据，查询第二页
select * from stu limit 3,3;

#每页显示3条数据，查询第三页
select * from stu limit 6,3;
```

**起始索引公式**
>起始索引 = (当前页码 - 1) * 每页显示的条数



