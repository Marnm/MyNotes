---
title: MongoDB_Notes
tags: []
---

## 连接数据库

	# mongo
	MongoDB shell version v3.6.13
	connecting to: test
	> 
	
默认shell连接的是本机localhost，`nonnecting to:`显示你正在使用的数据库名称，'test'是MongoDB的默认库，切换数据库可以使用`use 'dbName'`命令


## 插入记录

	> j = {name:"mongo"};
	{ "name" : "mongo" }
	> t={x:3}
	{ "x" : 3 }
	> db.things.save(j)
	WriteResult({ "nInserted" : 1 })
	> db.things.save(t)
	WriteResult({ "nInserted" : 1 })
	> db.things.find()
	{ "_id" : ObjectId("627e7e064b7e83be0e107f68"), "name" : "mongo" }
	{ "_id" : ObjectId("627e7e124b7e83be0e107f69"), "x" : 3 }
	
* 不需要预先创建一个集合，在第一次插入数据时候会自动创建.
* 在文档汇总其实可以存储任何结构的数据，当然在实际应用我们存储的还是相同类型文档的集合
* 每次插入数据的时候，集合中都会有 一个ID，名字叫`_id`.

**使用循环插入**
```js
> for(var i=1;i<=10;i++)db.c2.insert({name:"user"+i, age:18+i})
WriteResult({ "nInserted" : 1 })
> db.c2.find()
{ "_id" : ObjectId("627fd46019fb40ff16ec97ab"), "name" : "user1", "age" : 19 }
{ "_id" : ObjectId("627fd46019fb40ff16ec97ac"), "name" : "user2", "age" : 20 }
{ "_id" : ObjectId("627fd46019fb40ff16ec97ad"), "name" : "user3", "age" : 21 }
{ "_id" : ObjectId("627fd46019fb40ff16ec97ae"), "name" : "user4", "age" : 22 }
{ "_id" : ObjectId("627fd46019fb40ff16ec97af"), "name" : "user5", "age" : 23 }
{ "_id" : ObjectId("627fd46019fb40ff16ec97b0"), "name" : "user6", "age" : 24 }
{ "_id" : ObjectId("627fd46019fb40ff16ec97b1"), "name" : "user7", "age" : 25 }
{ "_id" : ObjectId("627fd46019fb40ff16ec97b2"), "name" : "user8", "age" : 26 }
{ "_id" : ObjectId("627fd46019fb40ff16ec97b3"), "name" : "user9", "age" : 27 }
{ "_id" : ObjectId("627fd46019fb40ff16ec97b4"), "name" : "user10", "age" : 28 }
>
```

## `_id` key

MongoDB支持的数据类型中，`_id`是其自有产物。

存储在MongoDB集合中的每个文档都有一个默认的**主键** `_id` ， 这个主键名称是固定的，它可以是MongoDB支持的任何数据类型，默认是`ObjectId`。

\_id key 举例说明

	 db.test.find()
	{ "_id" : ObjectId("627d3ef6de66cb02cb45d3e1"), "name" : "zhangya", "age" : "21", "addr" : "北京市朝阳区" }
	{ "_id" : ObjectId("627d3fffde66cb02cb45d3e2"), "name" : "zhang3", "age" : "24", "addr" : "上海浦东区" }
	>
	
在MongoDB中，每一个集合都必须有一个叫做**\_id**的字段，字段的默认类型是ObjectId，换句话说，字段类型可以不是ObjectId，例如：

`db.collection.insert`

```js
> db.things.insert({_id:3,name:"Bill",age:34})
> db.things.find()
{ "_id" : ObjectId("627e7e064b7e83be0e107f68"), "name" : "mongo" }
{ "_id" : ObjectId("627e7e124b7e83be0e107f69"), "x" : 3 }
{ "_id" : ObjectId("627e811d4b7e83be0e107f6a"), "x" : 4, "j" : 1 }
{ "_id" : ObjectId("627e811d4b7e83be0e107f6b"), "x" : 4, "j" : 2 }
{ "_id" : ObjectId("627e811d4b7e83be0e107f6c"), "x" : 4, "j" : 3 }
{ "_id" : ObjectId("627e811d4b7e83be0e107f6d"), "x" : 4, "j" : 4 }
{ "_id" : ObjectId("627e811d4b7e83be0e107f6e"), "x" : 4, "j" : 5 }
{ "_id" : ObjectId("627e811d4b7e83be0e107f6f"), "x" : 4, "j" : 6 }
{ "_id" : ObjectId("627e811d4b7e83be0e107f70"), "x" : 4, "j" : 7 }
{ "_id" : ObjectId("627e811d4b7e83be0e107f71"), "x" : 4, "j" : 8 }
{ "_id" : ObjectId("627e811d4b7e83be0e107f72"), "x" : 4, "j" : 9 }
{ "_id" : 3, "name" : "Bill", "age" : 34 }
```

！虽然**\_id** 的类型可以自由指定，但是在同一个集合中必须唯一，如果插入重复值的话系统将会抛出异常

`db.collection.insertOne()`
```js
> db.test.insertOne({name:"fnaichu", age: 24})
{
        "acknowledged" : true,
        "insertedId" : ObjectId("627f80462c1c696497b0e31e")
}
> 
```
`db.collection.insertMany()`
```js
> db.test.insertMany([
... {name:"li4", age: 35, addr:"广州市"},
... {name:"wang5", age: 42, addr:"南京市"},
... {name:"qiang7", age: 28, addr:"上海市"}])
{
        "acknowledged" : true,
        "insertedIds" : [
                ObjectId("627f82648f2637720618c4f4"),
                ObjectId("627f82648f2637720618c4f5"),
                ObjectId("627f82648f2637720618c4f6")
        ]
}
> 
```


******

## 查询记录

### 普通查询

可以简单的通过`find()`来查询，它返回一个任意结构的集合

```js
> var cursor=db.things.find()
> while(cursor.hasNext()) printjson(cursor.next())
{ "_id" : ObjectId("627e7e064b7e83be0e107f68"), "name" : "mongo" }
{ "_id" : ObjectId("627e7e124b7e83be0e107f69"), "x" : 3 }
{ "_id" : ObjectId("627e811d4b7e83be0e107f6a"), "x" : 4, "j" : 1 }
{ "_id" : ObjectId("627e811d4b7e83be0e107f6b"), "x" : 4, "j" : 2 }
{ "_id" : ObjectId("627e811d4b7e83be0e107f6c"), "x" : 4, "j" : 3 }
{ "_id" : ObjectId("627e811d4b7e83be0e107f6d"), "x" : 4, "j" : 4 }
{ "_id" : ObjectId("627e811d4b7e83be0e107f6e"), "x" : 4, "j" : 5 }
{ "_id" : ObjectId("627e811d4b7e83be0e107f6f"), "x" : 4, "j" : 6 }
{ "_id" : ObjectId("627e811d4b7e83be0e107f70"), "x" : 4, "j" : 7 }
{ "_id" : ObjectId("627e811d4b7e83be0e107f71"), "x" : 4, "j" : 8 }
{ "_id" : ObjectId("627e811d4b7e83be0e107f72"), "x" : 4, "j" : 9 }
{ "_id" : 3, "name" : "Bill", "age" : 34 }
>
```
上面例子显示了游标风格的迭代输出。`hasNext()`函数告诉我们是否还有数据，如果有则可以调用`next()`函数.


当然我们可以使用的是JavaScript shell，可以用到JS的特性，forEach就可以输出游标了，下面的例子就是使用forEach()来循环输出的: forEach()必须定义一个函数供每个游标元素调用：
```js
> db.things.find().forEach(printjson)
{ "_id" : ObjectId("627e7e064b7e83be0e107f68"), "name" : "mongo" }
{ "_id" : ObjectId("627e7e124b7e83be0e107f69"), "x" : 3 }
{ "_id" : ObjectId("627e811d4b7e83be0e107f6a"), "x" : 4, "j" : 1 }
{ "_id" : ObjectId("627e811d4b7e83be0e107f6b"), "x" : 4, "j" : 2 }
{ "_id" : ObjectId("627e811d4b7e83be0e107f6c"), "x" : 4, "j" : 3 }
{ "_id" : ObjectId("627e811d4b7e83be0e107f6d"), "x" : 4, "j" : 4 }
{ "_id" : ObjectId("627e811d4b7e83be0e107f6e"), "x" : 4, "j" : 5 }
{ "_id" : ObjectId("627e811d4b7e83be0e107f6f"), "x" : 4, "j" : 6 }
{ "_id" : ObjectId("627e811d4b7e83be0e107f70"), "x" : 4, "j" : 7 }
{ "_id" : ObjectId("627e811d4b7e83be0e107f71"), "x" : 4, "j" : 8 }
{ "_id" : ObjectId("627e811d4b7e83be0e107f72"), "x" : 4, "j" : 9 }
{ "_id" : 3, "name" : "Bill", "age" : 34 }
>
```

在MongoDB shell里，也可以把游标当作数组来用：
```js
var cursor=db.things.find()
> printjson(cursor[1])
{ "_id" : ObjectId("627e7e124b7e83be0e107f69"), "x" : 3 }
> printjson(cursor[2])
{ "_id" : ObjectId("627e811d4b7e83be0e107f6a"), "x" : 4, "j" : 1 }
> printjson(cursor[0])
{ "_id" : ObjectId("627e7e064b7e83be0e107f68"), "name" : "mongo" }
> printjson(cursor[12])
undefined
```

使用游标的时候请注意占用内存的问题，特别是很大的游标对象，有可能会内存溢出。所以应该用迭代的方式输出。把游标转换成真实的数组类型：
```js
> var arr=db.things.find().toArray()
> arr[0]
{ "_id" : ObjectId("627e7e064b7e83be0e107f68"), "name" : "mongo" }
> arr[11]
{ "_id" : 3, "name" : "Bill", "age" : 34 }
> arr[12]
>

```
请注意这些特性知识在MongoDB shell里使用，而不是所有的其他应用程序驱动都支持。MongoDB游标对象不是没有快照，如果有其他用户在集合里第一次或者最后一次调用`next()`，你可能得不到游标里的数据。所以要明确的锁定你要查询的游标。



### 条件查询

一个类似SQL的查询，是怎么在MongoDB里实现的？这里是在MongoDB shell里查询，当然你也可以用其他的应用程序驱动或者语言来实现：
| |
| --- |
| **SQL：** |
| SELECT * FROM things WHERE name="mongo"; |
| **MongoDB：** |
| db.things.find({name:"mongo"}).forEach(printjson) |

| |
| --- |
| **SQL：** |
| SELECT * FROM things WHERE x=4; |
| **MongoDB：** |
| db.things.find({x:4}).forEach(printjson) |

查询条件时`{a:A, b:B}`类似于`where a==A and b==B and ...`

上面显示的是所有的元素，当然我们也可以返回特定的元素，类似于返回表里的某字段的值，只需要在find({x:4})里指定元素的名字

|  |
| --- |
| **SQL：** |
| SELECT j FROM things WHERE x=4 |
| **MongoDB：** |
| db.things.find({x:4},{j:true}) |


### `findOne()`

为了方便考虑，MongoDB shell避免游标可能带来的开销，提供一个`findOne()`函数。这个函数和find()函数一样，不过它返回的是游标里的第一条数据，如果没有找到内容则返回null值。

```sql
> printjson(db.things.findOne({name:"mongo"}))
{ "_id" : ObjectId("627e7e064b7e83be0e107f68"), "name" : "mongo" }
>
```

### `limit`

如果需要限制结果集的长度，那么可以调用`limit`方法，这是强烈推荐解决性能问题的方法，就是通过限制条数来减少网络传输，例如
```js
> db.things.find().limit(4)
{ "_id" : ObjectId("627e7e064b7e83be0e107f68"), "name" : "mongo" }
{ "_id" : ObjectId("627e7e124b7e83be0e107f69"), "x" : 3 }
{ "_id" : ObjectId("627e811d4b7e83be0e107f6a"), "x" : 4, "j" : 1 }
{ "_id" : ObjectId("627e811d4b7e83be0e107f6b"), "x" : 4, "j" : 2 }
>
```

****

## 修改记录

用数据库的描述方式就是：将name字段，值为mongo的一条内容修改为mongo_new：
```js
> db.things.update({name:"mongo"},{$set:{name:"mong_new"}})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
```


## 删除记录

将用户name是mong_new的记录删除:
```js
> printjson(db.things.findOne({name:"mong_new"}))
{ "_id" : ObjectId("627e7e064b7e83be0e107f68"), "name" : "mong_new" }

> db.things.remove({name:"mong_new"})
WriteResult({ "nRemoved" : 1 })
> db.things.findOne({name:"mong_new"})
null

```


## 高级查询

## 条件操作符

`<`、`<=`、`>`、`>=`

```js
	db.collection.find({"fields": {$gt: value}})
	db.collection.find({"fields": {$lt: value}})
	db.collection.find({"fields": {$gte: value}})
	db.collection.find({"fields": {$lge: value}})
	// 多个条件
	db.collection.find({"fields":{$gt:value1, $lt: value2}});
```

**`$exists`** 判断字段是否存在

```js
//查询存在age字段的记录
db.c1.find({age:{$exists:true}})

// 查询name字段不存在的记录
db.c2.find({name:{$exists:false}})
```

**NULL** 值处理
```js
// 查询记录排查null值
db.c1.find({"age":"$in":[null],"$exists":true})
```

**`$mod`** 取模运算

```js
// 查询age字段，取模6等于1的数据
> db.c2.find({age:{$mod:[6,1]}})
{ "_id" : ObjectId("627fd46019fb40ff16ec97ab"), "name" : "user1", "age" : 19 }
{ "_id" : ObjectId("627fd46019fb40ff16ec97b1"), "name" : "user7", "age" : 25 }
// 查询age字段，取模2等于0的数据
> db.c2.find({age:{$mod:[2,0]}})
{ "_id" : ObjectId("627fd46019fb40ff16ec97ac"), "name" : "user2", "age" : 20 }
{ "_id" : ObjectId("627fd46019fb40ff16ec97ae"), "name" : "user4", "age" : 22 }
{ "_id" : ObjectId("627fd46019fb40ff16ec97b0"), "name" : "user6", "age" : 24 }
{ "_id" : ObjectId("627fd46019fb40ff16ec97b2"), "name" : "user8", "age" : 26 }
{ "_id" : ObjectId("627fd46019fb40ff16ec97b4"), "name" : "user10", "age" : 28 }
>
```

**`$ne`** 不等于

```js
// 查询x不等于3的记录
> db.things.find({x:{$ne:3}})
{ "_id" : ObjectId("627e811d4b7e83be0e107f6a"), "x" : 4, "j" : 1 }
{ "_id" : ObjectId("627e811d4b7e83be0e107f6b"), "x" : 4, "j" : 2 }
{ "_id" : ObjectId("627e811d4b7e83be0e107f6c"), "x" : 4, "j" : 3 }
{ "_id" : ObjectId("627e811d4b7e83be0e107f6d"), "x" : 4, "j" : 4 }
{ "_id" : ObjectId("627e811d4b7e83be0e107f6e"), "x" : 4, "j" : 5 }
{ "_id" : ObjectId("627e811d4b7e83be0e107f6f"), "x" : 4, "j" : 6 }
{ "_id" : ObjectId("627e811d4b7e83be0e107f70"), "x" : 4, "j" : 7 }
{ "_id" : ObjectId("627e811d4b7e83be0e107f71"), "x" : 4, "j" : 8 }
{ "_id" : ObjectId("627e811d4b7e83be0e107f72"), "x" : 4, "j" : 9 }
{ "_id" : 3, "name" : "Bill", "age" : 34 }
>
```

**`$in` 和 `$nin` **，包含和不包含
```js
> db.c2.find({"name":{"$in":[20,21,22]}})

> db.c2.find({"name":{"$nin":[20]}})
```

**`count()`** 查询记录条数
```js
db.things.find().count()
```

**`skip()`** 限制返回记录的起点
```js
// 从第31条记录开始，返回10条记录
db.userProfile.find().skip(31).limit(10)
```

### 排序

**以年龄升序ASC**
```js
db.c2.find().sort({age: 1})
```
**以年龄降序DESC**
```js
db.c2.find().sort({age:-1})
```


## 游标

像大多数数据库产品一样，MongoDB也是用游标来循环处理每一条结果数据
```js
> for(var c=db.c2.find();c.hasNext();){
... printjson(c.next())}
{
        "_id" : ObjectId("627fd46019fb40ff16ec97ab"),
        "name" : "user1",
        "age" : 19
}
{
        "_id" : ObjectId("627fd46019fb40ff16ec97ac"),
        "name" : "user2",
        "age" : 20
}
{
        "_id" : ObjectId("627fd46019fb40ff16ec97ad"),
        "name" : "user3",
        "age" : 21
}
{
        "_id" : ObjectId("627fd46019fb40ff16ec97ae"),
        "name" : "user4",
        "age" : 22
}
{
        "_id" : ObjectId("627fd46019fb40ff16ec97af"),
        "name" : "user5",
        "age" : 23
}
{
        "_id" : ObjectId("627fd46019fb40ff16ec97b0"),
        "name" : "user6",
        "age" : 24
}
{
        "_id" : ObjectId("627fd46019fb40ff16ec97b1"),
        "name" : "user7",
        "age" : 25
}
{
        "_id" : ObjectId("627fd46019fb40ff16ec97b2"),
        "name" : "user8",
        "age" : 26
}
{
        "_id" : ObjectId("627fd46019fb40ff16ec97b3"),
        "name" : "user9",
        "age" : 27
}
{
        "_id" : ObjectId("627fd46019fb40ff16ec97b4"),
        "name" : "user10",
        "age" : 28
}
// 另一种方式
> db.c2.find().forEach(function(u){printjson(u)})
{
        "_id" : ObjectId("627fd46019fb40ff16ec97ab"),
        "name" : "user1",
        "age" : 19
}
{
        "_id" : ObjectId("627fd46019fb40ff16ec97ac"),
        "name" : "user2",
        "age" : 20
}
{
        "_id" : ObjectId("627fd46019fb40ff16ec97ad"),
        "name" : "user3",
        "age" : 21
}
{
        "_id" : ObjectId("627fd46019fb40ff16ec97ae"),
        "name" : "user4",
        "age" : 22
}
{
        "_id" : ObjectId("627fd46019fb40ff16ec97af"),
        "name" : "user5",
        "age" : 23
}
{
        "_id" : ObjectId("627fd46019fb40ff16ec97b0"),
        "name" : "user6",
        "age" : 24
}
{
        "_id" : ObjectId("627fd46019fb40ff16ec97b1"),
        "name" : "user7",
        "age" : 25
}
{
        "_id" : ObjectId("627fd46019fb40ff16ec97b2"),
        "name" : "user8",
        "age" : 26
}
{
        "_id" : ObjectId("627fd46019fb40ff16ec97b3"),
        "name" : "user9",
        "age" : 27
}
{
        "_id" : ObjectId("627fd46019fb40ff16ec97b4"),
        "name" : "user10",
        "age" : 28
}
>
```


## 存储过程

MongoDB同样支持存储过程。关于存储过程你需要知道的第一件事就是它是用JavaScript来写的。  
MongoDB存储过程是存储在db.system.js表中。

MongoDB的存储过程创建过程如下：
```js
// 创建一个存储过程
> db.system.save({_id:"addNumbers",value:function(x,y){return x+y}})
WriteResult({ "nMatched" : 0, "nUpserted" : 1, "nModified" : 0, "_id" : "addNumbers" })

//可以以下方法检查存储
> db.system.js.find()
{ "_id" : "addNumbers", "value" : { "code" : "function (x,y){return x+y}" } }

// 调用存储过程
> db.eval('addNumber(3,4.2)')
WARNING: db.eval is deprecated
7.2
```

## 参考：
https://www.mongodb.com/docs/v4.2/reference/method/js-collection/
