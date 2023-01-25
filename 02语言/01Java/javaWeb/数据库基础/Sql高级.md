# 约束

> [!info]+ 约束的概念
> - 作用于表中列上的规则，用于限制加入表的数据
> 	- 例如：给id列加入约束，使其值不能重复，不能为null值
> - 保证了数据库中数据的正确性，有效性和完整性
> 	- 例如：年龄为3000，成绩为-5分这样的无效数据可以得到限制
> 

## 分类

- **非空约束**
	- 非空约束用于保证列中的所有数据不能为null值
	- ![](http://imgs.kbpoyo.top/imgs/数据库_202207062248124.png)
- **唯一约束**
	- 保证列中的所有数据各不相同
	- ![](http://imgs.kbpoyo.top/imgs/数据库_202207062252885.png)
- **主键约束**
	- 主键是一行数据的唯一标识，要求非空且唯一，一张表只能有一个主键
	- ![](http://imgs.kbpoyo.top/imgs/数据库_202207062256799.png)
- **外键约束**
	- ![](http://imgs.kbpoyo.top/imgs/数据库_202207062303838.png)
	- ![](http://imgs.kbpoyo.top/imgs/数据库_202207062305498.png)
	- ![](http://imgs.kbpoyo.top/imgs/数据库-202207062308488.png)
- **默认约束**
	- 保存数据时，未指定值则采用默认值
	- ![](http://imgs.kbpoyo.top/imgs/数据库_202207062310930.png)
- **检查约束**


# 多表查询

- **连接查询**
	- *内连接查询*
		- ![](http://imgs.kbpoyo.top/imgs/数据库_202207062316541.png)
		- 案例
			-  ![](http://imgs.kbpoyo.top/imgs/数据库_202207062318864.png)
			- ![](http://imgs.kbpoyo.top/imgs/数据库_202207062318275.png)
	- *外连接查询*
		- ![](http://imgs.kbpoyo.top/imgs/数据库_202207062320777.png)
		- 案例
			- ![](http://imgs.kbpoyo.top/imgs/数据库_202207062321482.png)
			- ![](http://imgs.kbpoyo.top/imgs/数据库_202207062321097.png)
- **子查询**
	- 概念
		- ![](http://imgs.kbpoyo.top/imgs/数据库_202207062323165.png)
	- 子查询结果不同，作用不同
		- 结果为单行单列，子查询语句作为条件值，使用 *= != > <* 等进行条件判断
		- 结果为多行单列，子查询语句作为条件值，使用 *in*等关键字进行条件判断
		- 结果为多行多列，子查询语句作为虚拟表
	- 案例
		- ![](http://imgs.kbpoyo.top/imgs/数据库_202207062328747.png)


# 数据库设计

**概念**
- 根据需求，构造最优的数据库存储模型
- 建立数据库中的表结构和表与表之间的联系

**软件研发步骤**
![](http://imgs.kbpoyo.top/imgs/数据库_202207062332559.png)

**设计步骤**
![](http://imgs.kbpoyo.top/imgs/数据库_202207062335107.png)

**表关系**
- 一对一
	- ![](http://imgs.kbpoyo.top/imgs/数据库_202207062336345.png)
- 一对多
	- ![](http://imgs.kbpoyo.top/imgs/数据库_202207062336314.png)
- 多对多
	- ![](http://imgs.kbpoyo.top/imgs/数据库-202207062337602.png)


# 事务

**概念**
![](http://imgs.kbpoyo.top/imgs/数据库_202207062341898.png)

**语法**
- 开启事务
	- *start transaction* 或者 *begin*
- 提交事务
	- *commit*
- 回滚事务
	- *rollback*

**案列**
![](http://imgs.kbpoyo.top/imgs/数据库_202207062344987.png)

**四大特征**
![](http://imgs.kbpoyo.top/imgs/数据库_202207062346416.png)

**修改默认方式**
- *select @@autocommit*：查询默认提交方式，1 为自动提交，0 为手动提交
- *set @@autocommit = 0*：设置提交方式
