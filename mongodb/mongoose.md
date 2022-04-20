# mongoose

mongoose 是一个让我们可以通过 Node 来操作 mongdb 的库，很多 node 框架都有对其包装实现。使用方法也大都相同，这里记录一下 node 实现的 mongoose 的基本使用方法。

mongoose 到数据库的映射关系如下图

![image-20220410172816385](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20220410172816385.png?x-oss-process=image/resize,w_800,m_lfit)

- **Schema**: 数据库中集合的结构描述，也可以说是一种约束。想操作集合就需要先建立集合的 schema
- **Model**: model 对应 mongodb 中的集合 collections，mongodb 中一个数据库只有一个 collections，但是可以有很多 collection。所以 Model 在创建时可以连接到具体的 collection，也可以不传第三个参数表示要新建 collection
- Document: document 对应集合中的具体文档。如果把 collection 看成一个对象的话，document 就表示其中的一个属性

Schema 生成 Model，Model 实例化生成 Document。Model 和 Document 都可以完成数据库操作

[toc]

#### 连接数据库

在引入 mongoose 后，使用`connect`命令即可连接数据：

```js
mongoose.connect('mongodb://user:password@127.0.0.1:27017/dbname')
```

其中 user/password 是可选的。

在实例化 Model 的时候会自动连接？？？（目前 egg-mongoose 是这样的）

## Schema

要生成一个 schema，只需对其进行初始化

```js
const UserSchema = new Schema({
  id: { type: String, unique: true},
  name: { type: String }
})
```

上面就获得了一个`UserSchema`，描述了 user 这个 document 的文档结构。

其中 type 类型有以下

 | 类型     | 作用         |
  | -------- | ------------ |
  | String   | 定义字符串   |
  | Number   | 定义数字     |
  | Date     | 定义日期     |
  | Buffer   | 定义二进制   |
  | Boolean  | 定义布尔值   |
  | Mixed    | 定义混合类型 |
  | ObjectId | 定义对象ID   |
  | Array    | 定义数组     |

#### schema 操作

- 添加字段: `schema.add({age: Number})`

- timestamps: 自动添加日期，配置该选项后会自动添加 createdAt 和 updatedA t这两个字段

  ```js
  const UserSchema = new Schema({
    // ...
    { timestamps: true }
  })
  ```
  

有了他之后我们就可以通过他来获取 Model

## Model

继续上面的 Schema，有了 Schema 之后我们就可以映射出我们的 Model

`mongoose.model(name[, Schema [, collection]])` 

- name: model 名，**Model 对象暴露出的 key** 
- Schema: 对应的模式
- collection: 要连接的集合

一旦创建好了Model对象，**就会自动和数据库中对应的集合建立连接**，以确保在应用更改时，集合已经创建并具有适当的索引，且设置了必须性和唯一性。

```js
const UserMoel = mongoose.model('User', UserSchema)
```

*注意：如果没有第三个参数，会自动创建对应的 collection，规则是把 name  `toLowerCase()`并且在末尾带上`s`*

#### Model 常用 API

- remove(conditions, callback)

- deleteOne(conditions, callback)

- deleteMany(conditions, callback)

- find(conditions, projection, options, callback)

  ```js
  
  ```

- findById(id, projection, options, callback)
- findOne(conditions, projection, options, callback)
- count(conditions, callback)
- create(doc, callback)
- update(conditions, doc, options, callback)

## Document

有了 model 就可以从 model 中实例化出 document。使用`new Model`会创建一个新的文档（具体的集合）。

也可以使用文档和数据库交互

```js
const doc = new UserModel({id: '1', name: 'test1'}) // 创建 document 

doc.methods.show = function() {} // 还可以给 document 加具体的方法（比 model 实用的地方）

doc.save((err, doc) => { // 和数据库交互，存储数据
  if (err) return console.error(err)
  doc.show()
})
```

