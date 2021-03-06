

# BookStore Restful API

###功能

1. 对“书名”进行模糊查询
2. 删除
3. 新增（待） 
3. 更新（待）

###工具

1. mongodb
  
    1. 命令：use dbName、show collections、db.collectionName.insert({})/find({})、db.collectionName.drop()/remove()

2. monk —— 读取mongodb

        var monk = require('monk');
        var db = monk('localhost:27017/BookStore');
        db.get('books');
        booksCollection.find({})  // 等同mongodb查询集合中所有文档的命令
        /*find({},{})第一个参数：配置查询匹配该字段的文档，
        第二个参数：配置查询结果如文档某字段显隐、分页等*/
        booksCollection.find({},{limit:limit,skip:(curPage-1)*limit})  
        find(searchStr,{"_id":0,limit:limit,skip:(curPage-1)*limit})
        booksCollection.remove({})  //删除集合中匹配该字段的文档

3. express

    配置路由、端口、http请求

4. http请求

    get（读取）、post（创建）、put更新、delete（删除）
    
###细节

1. 获取请求参数

        req.params.paramsName
        
2. monk读取的数据库中集合返回值都是Promise类型，可通过then((res)=>{})来获取返回值

        booksCollection
          .find()/.remove({})
          .then((docs)=>{
              //[]或[{},{}]
          });

3. 返回JSON字符串格式数据

        res.json(docs); 
        
4. 跨域支持

        res.header("Access-Control-Allow-Origin", "*");
        
5. 模糊查询

    用正则表达式匹配name字段
    
        searchStr = function () {
            var obj = {};
            if(req.params.search){
                obj = {name:new RegExp(req.params.search)}  //对name字段支持“模糊查询”
            }
            return obj;
        }();
        
6. 分页查询

    find()第二个参数配置limit和skip字段
    
        booksCollection.find({},{limit:limit,skip:(curPage-1)*limit})
        
7. 删除

        booksCollection.remove({id:parseInt(req.params.id)})
        
8. 新增

        var bodyParser=require('body-parser');
        app.use(bodyParser.json()); //需使用该方法，否则req.body为空对象
        booksCollection
          .findOne({},{sort: {id: -1}})  //找到当前最书籍中最大ID，sort:{id:-1}表示降序排序
          .then((docs)=>{
              var book = req.body;  //获取post的对象
              book.id = (parseInt(docs.id)+1); //自动生成新增ID:最大id基础上加1
              return booksCollection.insert(book)
          }) 
          
9. 更新

        var book=req.body; ////获取put的对象
        //执行更新,第1个参数是要更新的图书查找条件，第2个参数是要更新的对象
        booksCollection
            .update({"id":book.id}, book)
            .then((docs)=>{
                //返回更新完成后的对象
                return booksCollection.find({"id":book.id})
            })
            .then((docs)=>{
                res.json(docs[0]);
            }) 