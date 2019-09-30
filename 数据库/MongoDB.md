# MongoDB

## MongoDB简介

> MongoDB是一种NoSQL型数据库，存储的数据对象由键值对组成。MongoDB所有存储在集合中的数据，都是BSON格式。BSON格式是一种类似JSON的二进制形式的存储格式，是Binary JSON的简称。

```shell
{
	"_id": NumberLong("2001000000000010755"),
    "name": "xiaoming",
    "age": 20
}
```

### MongoDB和关系型数据库(MySQL)的对比

1. 术语概念对比

   | 关系型数据库 | MongoDB     | 说明                               |
   | ------------ | ----------- | ---------------------------------- |
   | database     | database    | 数据库                             |
   | table        | collection  | 数据库表/数据库集合                |
   | row          | document    | 记录行/文档                        |
   | column       | field       | 数据字段/域                        |
   | index        | index       | 索引                               |
   | primary key  | primary key | 主键，MongoDB自动将"_id"设置为主键 |

2. 数据形式对比

   **关系型数据库**

   | id   | name | age  | sex  |
   | ---- | ---- | ---- | ---- |
   | 1    | 张三 | 23   | 男   |
   | 2    | 李四 | 21   | 男   |

   **MongoDB**

   ```shell
   {
   	"_id": ObjectId("5d82ec3ac1380000450055bc"),
   	"name": "张三",
   	"age": 23,
   	"sex": "男"
   },
   {
   	"_id": ObjectId("5d82ec3ac1380000450055be"),
   	"name": "李四",
   	"age": 21,
   	"sex": "男"
   }
   ```

## MongoDB的基本操作

### db、集合的基本操作

```shell
# 命令行输入mongo进入MongoDB的命令交互模式
$ mongo
# 列出已有的db列表
$ show dbs
# 如果test存在，那么切换到test；如果不存在，则创建test数据库
$ use test
# 显示当前db
$ db
# 显示当前已有db，没有数据或者集合，不显示
$ show dbs
# 创建集合test
$ db.createCollection("test")
# 往集合test插入一条数据，如果集合不存在，则自动创建
$ db.test.insert({"name": "xiaoming"})
# 列出该db下的所有集合
$ show collections
# 列出该db下的所有集合，等效于show collections
$ show tables
# 删除集合
$ db.test.drop()
#删除当前数据库，执行前最好确认下当前数据库是不是你要删除的数据库
$ db.dropDatabase()
```

### 数据插入操作

插入数据有4种方法，insert、insert One、insert Many、save

```shell
# insert可以插入一条数据
$ db.test.insert({"name": "xiaoming"})
# insert也可以插入多条数据
$ db.test.insert([{"name": "zhangsan"}, {"name": "lisi"}]) 
# insertOne只能插入一条数据
$ db.test.insertOne({"name": "wangwu"}) 
# insertMany可以插入一条或多条数据，但是必须以列表(list)的形式组织数据
$ db.test.insertMany([{"name":"xiaoming"}, {"name": "xiaowang"}]) 
# 如果不指定_id，save的功能与insert一样
$ db.test.save([{"name":"xiaoming"},{"name":"xiaoli"}])
# 如果指定_id，mongodb就不为该条记录自动生成_id了；只有save可以指定_id，insert、insertOne、insertMany都不可以
$ db.test_collection.save({"_id":ObjectId("5d07461141623d5db6cd4d43"),"name":"xiaoming"})
```

### 数据修改

数据修改有2种方法，update、save

- update

  语法格式：

  ```shell
  db.collectionName.update(
  	<query>,  # update的查询条件，类似sql update语句的where部分
  	<update>, # update的对象和一些的更新的操作符等，也可以理解为sql update语句的set部分
  	{
  		upsert: <boolean>, # 可选，这个参数的意思是，如果不存在update记录，是否插入objNew，true为插入，false为不插入
  		multi: <boolean>,  # 可选，mongodb默认为false，只更新找到的第一条记录，如果这个参数为true，就把符合条件的多条记录全部更新
  		writeConcern: <document> # 可选，抛出异常的级别
  	}
  )
  ```
  
  示例：

  ```shell
  # 只能修改一条数据
  $ db.test.update({"_id": 123456789}, {"name": "newname"})
  # 若有多条数据，则修改多条数据
  $ db.test.update({"name": "newname"}, {$set:{"name": "newname1"}},{multi: true})  
  # 修改数据，若不存在则插入数据
  $ db.test.update({"name": "newname2"}, {$set:{"name": "newname1"}}, true)
  ```
  
- save

  save方法，是通过传入的新文档，替换已有的旧文档，如果原文档字段多于新文档，那么多出的字段就会丢失

  语法格式：

  ```shell
  db.collectionName.save(
  	<document>,  # 文档数据
  	{
  		writeConcern: <document>  # 可选，抛出的异常级别
  	}
  )
  ```

  示例：

  ```shell
  # 指定_id，新的文档会覆盖旧的文档，原文档会清空
  $ db.test_collection.save({"_id": ObjectId("5d82f6d3fcc62e38774e3248"), "name": "newName123"})
  ```

### 数据删除

数据删除有3种方法，deleteOne、deleteMany、remove

新版本的mongo，remove已过时，官方推荐deleteOne、deleteMany

- deleteOne

  ```shell
  # 删除name为xiaoming的一条记录，如果有多条符合条件，那么只删除第一条
  $ db.test.deleteOne({"name": "xiaoming"})
  ```

- deleteMany

  ```shell
  # 删除name为xiaoming的所有记录
  $ db.test.deleteMany({"name": "xiaoming"})
  # 删除该集合的所有数据
  $ db.test.deleteMany({})
  ```

- remove

  语法格式：

  ```shell
  db.collection.remove(
  	<query>,    # （可选）删除的文档的条件
  	{
  		justOne: <boolean>, # （可选）如果设为 true 或 1，则只删除一个文档，如果不设置该参数，或使用默认值 false，则删除所有匹配条件的文档。
  		writeConcern: <document> # （可选）抛出异常的级别。
  	}
  )
  ```

  示例：

  ```shell
  # 删除name为xiaoming的所有记录
  $ db.test.remove({"name": "xiaoming"})
  # remove方法并不会真正释放空间，需要继续执行db.repairDatabase()来回收磁盘空间
  $ db.repairDatabase()
  ```

### 数据查询

数据查询的方法有findOne、find，findOne只返回一条数据，find返回所有的数据

- SQL与mongo的对比

  |          | SQL                                | MongoDB                                   |
  | :------: | ---------------------------------- | ----------------------------------------- |
  |   等于   | select * from test where id = 1;   | db.test.find({"id": 1}).pretty()          |
  |   小于   | select * from test where id < 10;  | db.test.find({"id": {$lt: 10}}).pretty()  |
  | 小于等于 | select * from test where id <= 10; | db.test.find({"id": {$lte: 10}}).pretty() |
  |   大于   | select * from test where id > 1;   | db.test.find({"id": {$gt: 1}}).pretty()   |
  | 大于等于 | select * from test where id >= 1;  | db.test.find({"id": {$gte: 1}}).pretty()  |

  **注：pretty()能让查询结果以格式化的json形式打印出来，便于查看**

- 排序、limit、skip

  - sort：1为升序，-1为降序
  - limit：显示数据的条数
  - skip：跳过数据的条数
  ```shell
  db.test.find({}).sort({"name": -1}).limit(10).skip(10).pretty()
  ```

- 复合条件查询，and、or

  - and

    find方法能够传入多个键值对，用逗号 ',' 隔开

    ```shell
    db.test.find({"name": "xiaoming", "name": "xiaowang"}).pretty()
    db.test.find({"id": {$gt: 1, $lt: 10}}).pretty()
    ```

  - or

    关键字是 `$or`

    ```shell
    # 语法
    db.collectionName.find(
    	{
    		$or: [
    			{key1: value1}, {key2: value2}
    		]
    	}
    ).pretty()
    # 示例
    db.test.find({$or: [{"name": "xiaoming"}, {"name": "xiaowang"}]}).pretty()
    ```
  
  - and + or 复合查询
  
    ```shell
    db.test.find({"id": 10, $or: [{"name": "xiaoming"}, {"name": "xiaowang"}]}).pretty()
    ```
  
- 包含 in、不包含nin、全部all

  ```shell
  # 1. in 包含
  db.test.find({"name": {$in: ["xiaoming", "xiaowang", "xiaozhang"]}}).pretty()
  # 2. nin 不包含
  db.test.find({"name": {$nin: ["xiaoming", "xiaowang", "xiaozhang"]}}).pretty()
  # 3. all 全部
  db.test.find({"name": {$all: ["xiaoming", "xiaowang", "xiaozhang"]}}).pretty()
  ```

- exists 判断字段是否存在

  ```shell
  # 字段存在的话就用true，不存在的话就false
  db.test.find({"isDelete": {$exists: false}}).pretty()
  ```

- null空值处理

  ```shell
  db.test.find({"tel": null}).pretty()
  ```

- 取模运算

  ```shell
  db.test.find({"id": {$mod: [10, 0]}}).pretty()
  ```

- count获取查询结果数

  ```shell
  db.test.find().count()
  # 默认情况下，count方法仍然返回全部记录条数
  db.test.find().limit(1).count()
  # 返回限制之后的记录数量，要使用count(true) 或者 count(非0)
  db.test.find().limit(1).count(true)
  ```

- distinct根据条件查询 所有

  ```shell
  # 查出所有的name列
  db.test.distinct("name")
  ```

### 索引

索引能够极大的提高查询效率，没有索引的话，读取数据时，必须扫描整个集合，再选取符合条件的记录。
索引是一种特殊的数据结构，索引存储在一个易于便利读取的数据集合中，索引是对数据库表中一列或多列的值进行排序的一种结构。

```shell
db.collectionName.createIndex(
	{key1: option1, key2: option2}, #key为要创建索引的字段，option为创建索引的方式：1 为升序，-1 为降序，可以对多个字段创建索引，称为复合索引
	{
		background: <boolean>,  #background可选，建索引过程会阻塞其它数据库操作，background 设置为 true 可指定以后台方式创建索引，默认值为 false
		unique: <boolean>,  #unique可选，建立的索引是否唯一。指定为true创建唯一索引。默认值为false
		name: <boolean>,  #name可选，索引的名称。如果未指定，MongoDB的通过连接索引的字段名和排序顺序生成一个索引名称
		sparse: <boolean>  #sparse可选，对文档中不存在的字段数据不启用索引；这个参数需要特别注意，如果设置为true的话，在索引字段中不会查询出不包含对应字段的文档。默认值为 false
	}
)
```

