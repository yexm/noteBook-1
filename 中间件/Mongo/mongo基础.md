# SQL术语Mongo基础

服务端程序：mongod

客户端程序：mongo

| SQL术语   | MongoDB 术语 |
| --------- | ------------ |
| databases | databases    |
|table|colliection|
|row|document|
|column|field|
|index|index|
|table joins||
|primary key|primary key|


MongoDB连接方式，使用mongo客户端启动后

e.g.

```bash
mongodb://admin:admin@localhost/test
```

## 创建数据库
```bash
use DATABASE_NAME
```
## 删除

#### 删除当前数据库

```bash
db.dropDatabase()
```

#### 删除集合

```bash
db.collection.drop()
```

#### 删除文档

```bash
db.collection.remove(
   <query>,
   {
     justOne: <boolean>,
     writeConcern: <document>
   }
)
```

### 更新数据

```bash
db.collection.update(
   <query>,                                                         
 #查询kv
   <update>,                                                      
 #更新kv
   {
     upsert: <boolean>,                                     
#如果kv不存在是否插入
     multi: <boolean>,
#多值更新，默认只更新找到的第一条记录，设置为true，将更新所有匹配到的kv
     writeConcern: <document>
#抛出异常的级别
   }
)
```

### 插入数据

```bash
db.runoob.insert({name:"runoob"})
```

数据可赋值给一个变量，将变量传递给插入函数，以下实例为向集合中插入文档

```bash
db.col.insert({title: 'MongoDB 教程', 
   description: 'MongoDB 是一个 Nosql 数据库',
   by: '菜鸟教程',
   url: 'http://www.runoob.com',
   tags: ['mongodb', 'database', 'NoSQL'],
   likes: 100
})
```

or

```bash
document=({title: 'MongoDB 教程', 
   description: 'MongoDB 是一个 Nosql 数据库',
   by: '菜鸟教程',
   url: 'http://www.runoob.com',
   tags: ['mongodb', 'database', 'NoSQL'],
   likes: 100
});

db.col.insert(document)
WriteResult({ "nInserted" : 1 })
```

插入的数据被自动添加_id，不指定该值save()与insert()类似，指定_id save()方法将重置_id

### 查询

```bash
db.col.find()
db.col.find().pretty()
```

二者的区别自己猜吧，很明显

### mongoDB  中$操作符

$lt    小于

$gt    大于

$lte/gte    略

$ne    不等于

and条件使用','隔开即可

or条件使用实例：

```bash
db.col.find(
   {
      $or: [
         {key1: value1}, {key2:value2}
      ]
   }
).pretty()
```

$type    指定v的数据类型，实例如下：

```bash
db.col.find({title : {$type : 2}})
```


