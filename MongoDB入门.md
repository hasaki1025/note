# MongoDB入门

- ## 基本操作

  - 数据库

    - 创建

      ```js
      use mydbname
      ```

      - 创建后自动切换到该数据库

    - 删除

      ```javascript
      db.dropDatabase()
      ```

      - 删除当前数据库

    - 查看所有数据库

      ```javascript
      show dbs
      ```

    - 查看当前数据库

      ```javascript
      db
      ```

  - 集合

    - 创建一个空的集合

      ```javascript
      db.createCollection(CollectionName)
      #db.createCollection("nihao")
      ```
    
    - 删除集合
    
      ```javascript
      db.CollectionName.drop()
      #db.nihao.drop()
      ```
      
    
  - 文档
  
    - 插入操作
  
      - 单条数据插入
  
        ```javascript
        db.mydb.insertOne({
            name:"123"
        })
        ```
  
      - 多条数据插入
  
        ```javascript
        db.mydb.insertMany([
            {name:"1"},
            {name:"2"}
        ])
        ```
  
    - 查询
  
      - 查询所有
  
        ```javascript
        db.CollectionName.find()
        ```
  
      - 查询附带简单条件
  
        ```javascript
        //查询name为123的文档
        db.CollectionName.find({name:"123"})
        //查询name为123且id为2的文档
        db.CollectionName.find({
            name:"123",
            id:2
        })
        ```
  
        - or条件
  
          ```java
          //查询name为123或者23的文档
          db.CollectionName.find({
              $or:[
                  //条件一
                  {name:"123"},
                  {name:"23"}
              ]
          })
          ```
  
        - in关键字
  
          ```javascript
          //查询name为123或者23的文档
          db.CollectionName.find(
              name:{
              	$in:["123","23"]
              }
          )
          ```
  
        - 点运算符
  
          ```javascript
          //假设集合中user字段下含有name字段，现查询user的name为123的文档
          db.Collection.find(
              "user.name":"123"
          )
          ```
  
      - 查询数组
  
        - 查询tags数组为red和blank的文档
  
          ```javascript
          db.Collection.find( { tags: ["red", "blank"] } )
          ```
  
          - 只有tags数组完全和["red", "blank"]相同才能匹配上（包括顺序）
  
        - 查询包含了red和blank的文档
  
          ```javascript
          db.Collection.find({
              tags:{
                  $all:["red","blank"]
              }
          })
          ```
  
          - 只要tags数组中包含了red和blank即可（无关顺序）
  
        - 对数组单个值限制
  
          ```javascript
          //只要dim_cm数组中含有大于25的元素即可匹配
          db.Collection.find(
              dim_cm:{
              	$gt:20
              }
          )
          //数组中至少含有一个数大于15或者小于20
          db.Collection.find(
              dim_cm:{
              	$gt:15,$lt:20
              }
          )
          //数组中至少含有一个数大于22且小于30
          db.Collection.find(
          	dim_cm:{
              	$elemMatch:{
              		$gt:22,$lt:30
              	}
              }
          )
          ```
  
        - 数组索引值条件
  
          ```javascript
          //条件：dim_cm数组中下标为2大于24
          db.Collection.find(
          	{
                  "dim_cm.2":{
                      $gt:24
                  }
              }
          )
          ```
  
        - 数组长度限制
  
          ```javascript
          //限制数组长度为3
          db.Collection.find(
          	{
                  dim_cm:{
                      $size:3
                  }
              }
          )
          ```
  
      - 查询嵌套在数组中的文档
  
        - 嵌套在数组中文档中的某个字段限制
  
          ```javascript
          //users数组至少含有一个文档中的name字段为123
          db.Collection.find(
          	{
                  "users.name":"123"
              }
          )
          ```
  
      - 指定查询返回字段
  
        - 返回所有字段
  
          ```javascript
          //将会返回status为good的文档的所有的字段
          db.Collection.find(
          	{
                  status:"good"
              }
          )
          ```
  
        - 返回指定字段
  
          ```javascript
          //将会返回status为good的文档的item字段、_id字段(默认返回)、status字段
          db.Collection.find(
          	{
                  status:"good"
              },
              {
                  item:1,
                  status:1
              }
          )
          //抑制_id字段
          db.Collection.find(
          	{
                  status:"good"
              },
              {
                  _id:0
                  item:1,
                  status:1
              }
          )
          ```
  
        - 排除指定字段
  
          ```javascript
          //返回除了status和item以外的所有字段
          db.Collection.find(
          	{
                  status:"good"
              },
              {
                  status:0,
                  item:0
              }
          )
          ```
  
        - 返回嵌入文档中的特殊字段
  
          ```javascript
          db.Collection.find(
          	{
                  status:"good"
              },
              {
                  "user.name":1
              }
          )
          ```
  
          - 从 MongoDB 4.4 开始，您还可以使用嵌套形式指定嵌入字段，例如
  
            ```java
            db.Collection.find(
            	{
                    status:"good"
                },
                {
                   user:{
                       name:1
                   }
                }
            )
            ```
  
          - 隐藏特殊字段同理
  
        - 返回嵌入式文档数组的指定字段
  
          ```java
          //users数组是嵌入式文档数组，其中的文档包括了name字段，以下语句设置只返回name字段
          db.Collection.find(
              {
                  status:"good"
              },
              {
                  "users.name": 1
              }
          )
          ```
  
        - 返回数组指定范围
  
          ```javascript
          //返回items数组前三个
          db.mydb6.find({
              status:'A'
          },{
              status:1,
              items:{
                  $slice:3
              }
          })
          //返回items数组后三个
          db.mydb6.find({
              status:'A'
          },{
              status:1,
              items:{
                  $slice:-3
              }
          })
          ```
  
        - 根据条件筛选数组并返回数组的第一个元素
  
          ```javascript
          //比如筛选出来的数组为[1,2,3,4,5,6,7,8],一下语句返回的就是3
          db.Collection.find(
          	{
                  status:"A",
                  //必须要有数组筛选条件
                  items:{
                      $gt:2
                  }
              },{
                  //显示符合条件的items数组中第一个符合条件的元素
                  "items.$":1
              }
          )
          ```
  
      - 查询空字段或者缺失字段
  
        - 查找值为null
  
          ````shell
          db.Collection.find(
          	{
          		status:null
          	}	
          )
          ````
  
        - 采用存在性查找null值
  
          ```shell
          db.Collection.find(
          	{
          		status:{
          			$exists:false
          		}
          	}
          )
          ```
  
          
  
        - 采用类型匹配查找null值(该查找方法在从 MongoDB 4.2 开始废弃)
  
          ```shell
          db.Collection.find(
          	{
          		status:{
          			$type:10
          		}
          	}
          )
          ```
  
          - 在mongodb中每个类型都有一个编号，10就代表是null类型，类型表如下
  
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
  
    - 更新操作
  
      - 更新单个文档
  
        ```shell
        db.inventory.updateOne(
        	#条件
           { item: "paper" },
           #操作
           {
           #set操作，如果该字段不存在则会创建
             $set: { "size.uom": "cm", status: "P" },
             #$currentDate关键字
             $currentDate: { lastModified: true }
           }
        )
        ```
  
        - $currentDate：如果当前文档中不存在lastModified字段则会创建并赋值为当前时间，如果存在则更新值为当前时间
        - _id字段无法修改且id字段永远是第一个字段
  
      - $rename关键字
  
        ```shell
        db.students.updateOne(
           { _id: 1 },
           #左旧右新：nickname-->alias
           { $rename: { 'nickname': 'alias', 'cell': 'mobile' } }
        )
        ```
  
        - rename关键字可能会改变字段顺序

