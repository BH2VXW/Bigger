我之前在《[.Net Core系列教程（三）——使用Mongodb][1]》中，说过在.Net Core下，怎样使用Mongodb数据库，这篇文章就作为它的延伸，来说下在.Net中，怎样使用Mongodb Driver来进行最常用的增、删、改、查操作。当然，我这个的环境是在.Net Core下，在其他版本的.Net环境下，相差不大。我这实例中使用的驱动是2.4.0版的，而现在最新版本是2.4.4。
闲言少叙，还是撸代码吧。
先按《[.Net Core系列教程（三）——使用Mongodb][1]》文章中的方法，设置好数据库的相关配置，之后取得数据库：
<!--more-->
```csharp
    var db = client.GetDatabase("database");
```
这个代码可以按照自己的实际需求来写，比如这样：

```csharp
    var db = client.GetDatabase(MongoUrl.Create(settings.Value.MongodbConnection).DatabaseName);
```
取得一个collection，这里以news表为例，Models.News是News的实例类：

```csharp
    var collection = db.GetCollection<Models.News>("news");
```
我们再准备一下具体要操作的数据：

```csharp
    var request=new Models.News(){title:"新闻测试",body:"这里是新闻测试的内容",author:"张三","status":True};
```

这些前提准备好了之后，再开始具体的数据库操作了

1.增加操作：

```csharp
	collection.InsertOne(request);
```

2.修改操作：

```csharp
    var query = new BsonDocument("_id", new ObjectId(id));
    var dict = new Dictionary<string, object> {
        { "title",request.title},
        { "body",request.body},
        { "author",request.author},
        { "status",request.status}
    };
    var data = new BsonDocument(dict);
    collection.UpdateOne(query, new BsonDocument("$set", data));
```

3.查询操作：

```csharp
    //查列表
    int page=1;  //当前页号
    int pagesize=50;  //每页50条记录
    BsonDocument query = new BsonDocument(){"author":"张三","status":True};
    int total = Convert.ToInt32(collection.Count(query));  //数据总记录数
    var list = collection.Find(query).Sort(new BsonDocument("_id", -1)).Limit(pagesize).Skip((page-1)*pagesize).ToList();  //带分页查询，按_id倒序排序
    
    //查单条
    BsonDocument query = new BsonDocument("_id", _id);
    var data = collection.Find(query).FirstOrDefault();
```

4.删除操作：

```csharp
    ObjectId _id = new ObjectId(id);
    collection.DeleteOne(new BsonDocument("_id", _id));
```

以上只是其中几个比较简单的用法，其实还有很多实现方法，比如异步方法、插入多条等等，其他的等有时间再整理吧。
  [1]: 15.html
