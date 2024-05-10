# mongoDB



# 1.简介

MongoDB是一个`文档数据库`，旨在方便应用开发和扩展。**是一个非关系型文档数据库**



## 1.1 特点

- 面向集合存储，易存储对象类型的数据
- 支持查询,以及动态查询
- 支持RUBY，PYTHON，JAVA，C++，PHP，C#等多种语言
- 文件存储格式为`BSON`（一种JSON的扩展）
- 支持复制和故障恢复和分片
- 支持事务支持
- 索引 聚合 关联…



## 1.2 应用场景

- **游戏应用**：使用云数据库MongoDB作为游戏服务器的数据库存储用户信息。用户的游戏装备、积分等直接以内嵌文档的形式存储，方便进行查询与更新。
- **物流应用**：使用云数据库MongoDB存储订单信息，订单状态在运送过程中会不断更新，以云数据库MongoDB内嵌数组的形式来存储，一次查询就能将订单所有的变更读取出来，方便快捷且一目了然。
- **社交应用**：使用云数据库MongoDB存储用户信息以及用户发表的朋友圈信息，通过地理位置索引实现附近的人、地点等功能。并且，云数据库MongoDB非常适合用来存储聊天记录，因为它提供了非常丰富的查询，并在写入和读取方面都相对较快。
- **视频直播**：使用云数据库MongoDB存储用户信息、礼物信息等。
- **大数据应用**：使用云数据库MongoDB作为大数据的云存储系统，随时进行数据提取分析，掌握行业动态。



# 2.安装

`docker run --name mongodb -v ~/mongodb/db:/data/db -p 27017:27017 -d mongo:5.0.5`



`-d`：创建守护式容器
`-p`：端口映射
`~/mongodb/db:/data/db` 数据卷
`--restart=always`：启动docker ，MongoDB 自动启动
`-e MONGO_INITDB_ROOT_USERNAME=root`：设置用户名 root
`-e MONGO_INITDB_ROOT_PASSWORD=root`：设置密码 root



进入mongo容器

`docker exec -it mongodb mongo `

- mongo 登录mongo-cli命令



# 3.核心概念

## 3.1 库<database>

 `mongodb中的库就类似于传统关系型数据库中库的概念，用来通过不同库隔离不同应用数据`。mongodb中可以建立多个数据库。每一个库都有自己的集合和权限，不同的数据库也放置在不同的文件中。默认的数据库为"test"，数据库存储在启动指定的data目录中。

## 3.2 集合<Collection>

 `集合就是 MongoDB 文档组，类似于 RDBMS （关系数据库管理系统：Relational Database Management System)中的表的概念`。

集合存在于数据库中，一个库中可以创建多个集合。每个集合没有固定的结构，这意味着你在对集合可以插入不同格式和类型的数据，但通常情况下我们插入集合的数据都会有一定的关联性。

## 3.3 文档<Document>

文档集合中一条条记录，是一组键值(key-value)对(即 BSON)。MongoDB 的文档不需要设置相同的字段，并且相同的字段不需要相同的数据类型，这与关系型数据库有很大的区别，也是 MongoDB 非常突出的特点。

一个简单的文档例子如下：

```json
{"site":"123", "name":"123"}
```

## 3.4 关系总结

| RDBMS  | MongoDB |
| :----: | :-----: |
| 数据库 | 数据库  |
|   表   |  集合   |
|   行   |  文档   |
|   列   |  字段   |



# 4.基本操作

## 4.1 库操作

[官方文档](https://www.mongodb.com/docs/v5.0/reference/method/)

### 4.1.1 查看所有库

```sql
show databases; | show dbs;
```

![image-20230222204626145](img.asstes\image-20230222204626145.png)

注意:

- `admin`：从权限的角度来看，这是"root"数据库。要是将一个用户添加到这个数据库，这个用户自动继承所有数据库的权限。一些特定的服务器端命令也只能从这个数据库运行，比如列出所有的数据库或者关闭服务器。
- `local`：这个数据永远不会被复制，可以用来存储限于本地单台服务器的任意集合
- `config`：当Mongo用于分片设置时，config数据库在内部使用，用于保存分片的相关信息



### 4.1.2 创建数据库

```sql
use 库名
```

`注意: use 代表创建并使用,当库中没有数据时默认不显示这个库`



### 4.1.3 查看当前数据库

```sql
db
```



### 4.1.4 删除数据库

`默认删除当前选中的库`

```sql
db.dropDatabase()
```



## 4.2 集合操作

### 4.2.1 查看库中所有集合

```sql
show collections; | show tables;
```



### 4.2.2 创建集合

```sql
db.createCollection('集合名称', [options])
```

```
options可以是如下参数：
```

|   字段 | 类型 | 描述                                                         |
| -----: | ---- | ------------------------------------------------------------ |
| capped | 布尔 | （可选）如果为 true，则创建固定集合。固定集合是指有着固定大小的集合，当达到最大值时，它会自动覆盖最早的文档。 **当该值为 true 时，必须指定 size 参数。** |
|   size | 数值 | （可选）为固定集合指定一个最大值，即字节数。 **如果 capped 为 true，也需要指定该字段。** |
|    max | 数值 | （可选）指定固定集合中包含文档的最大数量。                   |

```sql
db.createCollection('集合名称', {max:100,capped:true,size:5000})
```

`注意:当集合不存在时,向集合中插入文档也会自动创建该集合。`



### 4.2.3 删除集合

```sql
 db.集合名称.drop();
```



## 4.3 文档操作

### 4.3.1 插入文档

`注意：在 mongodb 中每个文档都会有一个_id作为唯一标识,_id默认会自动生成如果手动指定将使用手动指定的值作为_id 的值。`

#### 4.3.1.1 单条文档

```sql
db.集合名称.insert({"name":"xh","age":23,"bir":"2022-12-12"});
```



#### 4.3.1.2 多条文档

```sql
db.集合名称.insertMany(
   [ <document 1> , <document 2>, ... ],
   {
 			writeConcern: 1,//写入策略，默认为 1，即要求确认写操作，0 是不要求。
      		ordered: true //指定是否按顺序写入，默认 true，按顺序写入。
   }
)


db.集合名称.insertMany([
  	{"name":"A","age":23,"bir":"2022-12-12"},
  	{"name":"B","age":25,"bir":"2022-12-12"}
]);

db.集合名称.insert([
  	{"name":"A","age":23,"bir":"2022-12-12"},
  	{"name":"B","age":25,"bir":"2022-12-12"}
]);
```



#### 4.3.1.3 脚本插入

```sql
for(let i=0;i<100;i++){
    db.users.insert({"_id":i,"name":"xh_"+i,"age":23});
}
```



### 4.3.2 查询所有文档

```sql
db.集合名称.find();
```



### 4.3.3 删除文档

```sql
db.集合名称.remove(
   <query>,
   {
     justOne: <boolean>,
     writeConcern: <document>
   }
)
```

**参数说明：**

- `query`：可选删除的文档的条件。
- `justOne`：`可选`如果设为 true 或 1，则只删除一个文档，如果不设置该参数，或使用默认值 false，则删除所有匹配条件的文档。
- `writeConcern`：`可选`抛出异常的级别。

```sql
db.xh.remove({_id:3}); #删除_id为3的文档
```

`db.xh.remove({ })--删除所有文档，{ }必须写` 



### 4.3.4 更新文档

```sql
db.集合名称.update(
   <query>,
   <update>,
   {
     upsert: <boolean>,
     multi: <boolean>,
     writeConcern: <document>
   }
);
```

**参数说明：**

- `query`：update的查询条件，类似sql update查询内where后面的。
- `update`：update的对象和一些更新的操作符（如,$,$inc…）等，也可以理解为sql update查询内set后面的
- `upsert`：`可选`，这个参数的意思是，如果不存在update的记录，是否插入objNew，true为插入，默认是false，不插入。
- `multi`：`可选`，mongodb 默认是false，只更新找到的第一条记录，如果这个参数为true,就把按条件查出来多条记录全部更新。
- `writeConcern`：`可选`，抛出异常的级别。

```sql
db.集合名称.update({"name":"zhangsan"},{name:"11",bir:new date()})  
		`这个更新是将符合条件的全部更新成后面的文档,相当于先删除在更新`(_id不发生改变)

db.集合名称.update({"name":"xiaohei"},{$set:{name:"mingming"}}) 
		`保留原来数据更新,但是只更新符合条件的第一条数据`

db.集合名称.update({name:”小黑”},{$set:{name:”小明”}},{multi:true}) 
		`保留原来数据更新,更新符合条件的所有数据`

db.集合名称.update({name:”小黑”},{$set:{name:”小明”}},{multi:true,upsert:true}) 
		`保留原来数据更新,更新符合条件的所有数据 没有条件符合时插入数据`
```



### 4.3.5 条件查询文档

**MongoDB 查询文档使用 find() 方法。find() 方法以`非结构化`的方式来显示所有文档。**

```sql
db.集合名称.find(query, projection)
```

- **query**：可选，使用查询操作符指定查询条件
- **projection**：可选，使用投影操作符指定返回的键。查询时返回文档中所有键值，只需省略该参数即可（默认省略）。

如果你需要以易读的方式来读取数据，可以使用 pretty() 方法，语法格式如下：

```sql
 db.集合名称.find().pretty()
```

`注意: pretty() 方法以格式化的方式来显示所有文档。`

#### 4.3.6.1 MongoDB 与 RDBMS Where 语句比较

如果你熟悉常规的 SQL 数据，通过下表可以更好的理解 MongoDB 的条件语句查询：

| 操作       | 格式                     | 范例                                       | RDBMS中的类似语句   |
| :--------- | :----------------------- | :----------------------------------------- | :------------------ |
| 等于       | `{<key>:<value>`}        | `db.xh.find({"by":"A"}).pretty()`          | `where by = 'A'`    |
| 小于       | `{<key>:{$lt:<value>}}`  | `db.xh.find({"likes":{$lt:50}}).pretty()`  | `where likes < 50`  |
| 小于或等于 | `{<key>:{$lte:<value>}}` | `db.xh.find({"likes":{$lte:50}}).pretty()` | `where likes <= 50` |
| 大于       | `{<key>:{$gt:<value>}}`  | `db.xh.find({"likes":{$gt:50}}).pretty()`  | `where likes > 50`  |
| 大于或等于 | `{<key>:{$gte:<value>}}` | `db.xh.find({"likes":{$gte:50}}).pretty()` | `where likes >= 50` |
| 不等于     | `{<key>:{$ne:<value>}}`  | `db.xh.find({"likes":{$ne:50}}).pretty()`  | `where likes != 50` |



#### 4.3.6.2 AND

MongoDB 的 find() 方法可以传入多个键(key)，每个键(key)以逗号隔开，即常规 SQL 的 AND 条件。

```sql
db.xh.find({key1:value1, key2:value2}).pretty()
```

`若查询key有多个相同字段，则只查询最后一个条件`

```sql
db.xh.find(
	{
		age:24, 
		age:{$gt:25}
	}
).pretty() #只查询age>25
```



#### 4.3.6.3 OR

MongoDB OR 条件语句使用了关键字 **$or**,语法格式如下

```sql
db.xh.find(
   {
      $or: [
         {key1: value1}, {key2:value2}
      ]
   }
).pretty()
```



#### 4.3.6.4 数组查询

```sql
{ "_id" : 4, "name" : "qqqq", "age" : 200, "bir" : "3033", "like" : [ "play", "game", "eat", "shopping" ] }
```

查询:`mongoDB支持数组查询`

```sql
db.xh.find({like:"game"}
```

##### $size

`$size`按照数组的长度查询

```sql
db.xh.find({like:{$size:4}})
```



#### 4.3.6.5 模糊查询

`mongoDB可以使用正则表达式进行模糊查询`

```sql
db.xh.find({name:/xh/}) #/正则表达式/
```

`也可以在数组中使用`

```sql
db.xh.find({like:/a/})
```



#### 4.3.6.6 排序sort

在 MongoDB 中使用 `sort()` 方法对数据进行排序，sort() 方法可以通过参数指定排序的字段，并使用 1 和 -1 来指定排序的方式，其中 `1 为升序排列`，而 `-1` 是用于`降序排列`。

```sql
db.COLLECTION_NAME.find().sort({KEY:1})
```



#### 4.3.6.7 limit

```sql
db.COLLECTION_NAME.find().limit(NUMBER)
```



#### 4.3.6.8 skip

```sql
db.COLLECTION_NAME.find().limit(NUMBER).skip(NUMBER)
```



#### 4.3.6.9 分页

skip 和 limit 结合就能实现分页。

```sql
db.COLLECTION_NAME.find().skip(10).limit(100)
```

读取从 10 条记录后 100 条记录，相当于 sql 中limit (10,100)。



`skip(), limilt(), sort()三个放在一起执行的时候，执行的顺序是先 sort(), 然后是 skip()，最后是显示的 limit()。`



#### 4.3.6.10 总条数

```sql
db.集合名称.count();
```



#### 4.3.6.11 去重

```sql
db.集合名称.distinct('字段')
```



#### 4.3.6.12 指定返回字段

```sql
db.集合名称.find({条件},{name:1,age:1}) 
```

参数: 1 返回  0 不返回    `注意:1和0不能同时使用`



### 4.3.6  $type

$type操作符是基于`BSON`类型来检索集合中匹配的数据类型，并返回结果。

| **类型**                | **数字** | **备注**         |
| :---------------------- | :------- | :--------------- |
| Double                  | 1        |                  |
| String                  | 2        |                  |
| Object                  | 3        |                  |
| Array                   | 4        |                  |
| Binary data             | 5        |                  |
| Undefined               | 6        | 已废弃。         |
| Object id               | 7        |                  |
| Boolean                 | 8        |                  |
| Date                    | 9        |                  |
| Null                    | 10       |                  |
| Regular Expression      | 11       |                  |
| JavaScript              | 13       |                  |
| Symbol                  | 14       |                  |
| JavaScript (with scope) | 15       |                  |
| 32-bit integer          | 16       |                  |
| Timestamp               | 17       |                  |
| 64-bit integer          | 18       |                  |
| Min key                 | 255      | Query with `-1`. |
| Max key                 | 127      |                  |

如果想获取 "xh" 集合中 id 为 string 的数据，可以使用以下命令：

```sql
db.xh.find({"title" : {$type : 2}})
或
db.xh.find({"title" : {$type : 'string'}})
```

如果想获取 "xh" 集合中 like 为 array 的数据，可以使用以下命令：

```sql
db.xh.find({like:{$type:'array'}})
```



### 4.3.7 多表连接查询

mongodb可以使用`aggregate`函数来实现连接查询

| **命令**      | **功能描述**                               |
| ------------- | ------------------------------------------ |
| $project      | 指定输出文档里的字段.                      |
| $match        | 选择要处理的文档，与fine()类似。           |
| $limit        | 限制传递给下一步的文档数量。               |
| $skip         | 跳过一定数量的文档。                       |
| $unwind       | 扩展数组，为每个数组入口生成一个输出文档。 |
| $group        | 根据key来分组文档。                        |
| $sort         | 排序文档。                                 |
| $geoNear      | 选择某个地理位置附近的的文档。             |
| $out          | 把管道的结果写入某个集合。                 |
| $redact       | 控制特定数据的访问。                       |
| ***$lookup*** | ***多表关联（3.2版本新增）***              |

#### 4.3.7.1 $lookup的功能及语法

**1. 主要功能** 是将每个输入待处理的文档，经过$lookup 阶段的处理，输出的新文档中会包含一个新生成的数组列（户名可根据需要命名新key的名字 ）。数组列存放的数据 是 来自 被Join 集合的适配文档，如果没有，集合为空（即 为[ ]）

```sql
{ $lookup: {
    from:<String>, 
    localField:<String>, 
    foreignField:<String>, 
    as:<String>
} }
```

| **语法值**     | **解释说明**                                                 |
| -------------- | ------------------------------------------------------------ |
| `from`         | 要连接的目标表，类似于sql中的join                            |
| `localField`   | 作为连接参照的本表字段                                       |
| `foreignField` | 作为连接参照的目标表字段，localField和foreignField类似sql多表连接中join后的等值设置。 |
| `as`           | 当连接查询查询出来之后，外表的查询结果会在本表查询结果中开辟一个新的字段进行存储，而字段名就是这个字段设置的，只有一个存储结果时为单独的对象字段，多个结果时为对象数组 : |

```sql
db.messgae.aggregate([
    {
        //临时字段保存id，转换为字符串
        $set: {
            "id": {$toString: "$_id"}
        }
    },
    //join条件
    {
        $lookup: {
            //要连接的表
            form: "message_ref",
            //要要连接的字段和目标字段，相当于ON
            localField: "id",
            foreignField: "messageId",
            //查询完毕存储字段
            as: "ref"
        }
    },
    //查询条件，类似于sql查询语句中的where部分
    {$match: {"ref.receiverId": 1}},
    //排序
    {$sort: {sendTime: -1}},
    //跳过
    {$skip: 0},
    //偏移量
    {$limit: 50}
])
```



## 5.索引

索引通常能够极大的提高查询的效率，如果没有索引，MongoDB在读取数据时必须扫描集合中的每个文件并选取那些符合查询条件的记录。这种扫描全集合的查询效率是非常低的，特别在处理大量的数据时，查询可以要花费几十秒甚至几分钟，这对网站的性能是非常致命的。索引是特殊的数据结构，索引存储在一个易于遍历读取的`数据集合`中，索引是对数据库表中一列或多列的值进行排序的一种结构。



### 5.1 创建索引

createIndex()方法基本语法格式如下所示：

```sql
db.collection.createIndex({keys：1/-1},{options})
```

语法中 Key 值为你要创建的索引字段，`1` 为指定按`升序`创建索引，按`降序`创建索引指定为 `-1` 

```sql
db.xh.createIndex({"title":1,"description":-1})
```

createIndex() 接收可选参数，可选参数列表如下：

| Parameter            | Type          | Description                                                  |
| :------------------- | :------------ | :----------------------------------------------------------- |
| `background`         | Boolean       | 建索引过程会阻塞其它数据库操作，background可指定以后台方式创建索引，即增加 "background" 可选参数。 "background" 默认值为**false**。 |
| `unique`             | Boolean       | 建立的索引是否唯一。指定为true创建唯一索引。默认值为**false**. |
| `name`               | string        | 索引的名称。如果未指定，MongoDB的通过连接索引的字段名和排序顺序生成一个索引名称。 |
| dropDups             | Boolean       | **3.0+版本已废弃。**在建立唯一索引时是否删除重复记录,指定 true 创建唯一索引。默认值为 **false**. |
| sparse               | Boolean       | 对文档中不存在的字段数据不启用索引；这个参数需要特别注意，如果设置为true的话，在索引字段中不会查询出不包含对应字段的文档.。默认值为 **false**. |
| `expireAfterSeconds` | integer       | 指定一个以秒为单位的数值，完成 TTL设定，设定集合的生存时间。`只对Date类型有效，不支持混合索引` |
| `v`                  | index version | 索引的版本号。默认的索引版本取决于mongod创建索引时运行的版本。 |
| weights              | document      | 索引权重值，数值在 1 到 99,999 之间，表示该索引相对于其他索引字段的得分权重。 |
| default_language     | string        | 对于文本索引，该参数决定了停用词及词干和词器的规则的列表。 默认为英语 |
| language_override    | string        | 对于文本索引，该参数指定了包含在文档中的字段名，语言覆盖默认的language，默认值为 language. |



### 5.2 查看索引

```sql
db.xh.getIndexes()
```



### 5.3 查看索引大小

```sql
db.xh.totalIndexSize()
```



### 5.4 删除所有索引

```sql
db.xh.dropIndexes()
```



### 5.5 删除指定索引

```sql
db.xh.dropIndex("索引名称")
```



### 5.6 复合索引

说明: 一个索引的值是由多个 key 进行维护的索引的称之为复合索引

```sql
db.集合名称.createIndex({"title":1,"description":-1})
```

`注意: mongoDB 中复合索引和传统关系型数据库一致都是左前缀原则`



## 6.聚合函数

MongoDB 中聚合(aggregate)主要用于处理数据(诸如统计平均值，求和等)，并返回计算后的数据结果。

有点类似 **SQL** 语句中的 **count(\*)**。

```sql
db.mycol.aggregate([{$group : {_id : "$by_user", num_tutorial : {$sum : 1}}}])
```

- _id:`聚合分组id`
- $by_user:`指定分组字段`
- num_tutorial:计算后结果名
- $sum : 1 ：求和，结果`x1`

| 表达式    | 描述                                                         | 实例                                                         |
| :-------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| $sum      | 计算总和。                                                   | db.mycol.aggregate([{$group : {_id : "$by_user", num_tutorial : {$sum : "$likes"}}}]) |
| $avg      | 计算平均值                                                   | db.mycol.aggregate([{$group : {_id : "$by_user", num_tutorial : {$avg : "$likes"}}}]) |
| $min      | 获取集合中所有文档对应值得最小值。                           | db.mycol.aggregate([{$group : {_id : "$by_user", num_tutorial : {$min : "$likes"}}}]) |
| $max      | 获取集合中所有文档对应值得最大值。                           | db.mycol.aggregate([{$group : {_id : "$by_user", num_tutorial : {$max : "$likes"}}}]) |
| $push     | 将值加入一个数组中，不会判断是否有重复的值。                 | db.mycol.aggregate([{$group : {_id : "$by_user", url : {$push: "$url"}}}]) |
| $addToSet | 将值加入一个数组中，会判断是否有重复的值，若相同的值在数组中已经存在了，则不加入。 | db.mycol.aggregate([{$group : {_id : "$by_user", url : {$addToSet : "$url"}}}]) |
| $first    | 根据资源文档的排序获取第一个文档数据。                       | db.mycol.aggregate([{$group : {_id : "$by_user", first_url : {$first : "$url"}}}]) |
| $last     | 根据资源文档的排序获取最后一个文档数据                       | db.mycol.aggregate([{$group : {_id : "$by_user", last_url : {$last : "$url"}}}] |
