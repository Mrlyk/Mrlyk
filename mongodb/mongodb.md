# mongodb

>  MongoDB是一个基于分布式文件存储的对象型数据库，由 C++ 编写。旨在为WEB应用提供可扩展的高性能数据存储解决方案。

中文文档：https://www.mongodb.org.cn/tutorial/

菜鸟教程：https://www.runoob.com/mongodb/mongodb-linux-install.html

mongodb 是对象型数据库，其中的集合`collction`对应传统关系型数据库中的表`table`，文档操作则对应着表操作

下面对他的基本使用做个记录

[toc]

## 数据库相关

- 查看当前所有的数据库：`show dbs` 
- 切换/创建 数据库: `use xxx` xxx 为要切换/创建的数据库名。数据库要有数据才能被看到
- 查看当前所在的数据库: `db` 

## 集合相关

- 查看当前数据库下的所有集合: `show collections`
- 查看集合中的所有数据: `show collection_name`

## CURD 文档操作

mongodb 的增删改查基本操作

#### Create

创建文档和插入文档相同，如果文档不存在时就是创建

**语法：** 

```text
db_name.collection_name.insert(
	<document or array of documents>,
	{
	    writeConcern: <document>,    // 
	    ordered: <boolean>    //可选字段
	}
)
```

**实例：** 

```shell
db.db_name.collection_name.insert(
	{"name": "test1"},
	{"name": "test1"},
)
```

插入数据时，如果没有指定唯一主键`_id`，则会自动生成

#### Read 数据查询

数据查询是最重要的一环，这里对使用做个基本说明

