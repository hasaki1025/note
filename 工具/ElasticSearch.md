# ElasticSearch

- ElasticSearch支持Restful风格
  - ElasticSearch只能使用get,put,delete,head方式的请求

  - ElasticSearch的基本操作

    - 创建一个索引

      - 使用Postman发送一个put请求创建一个索引
  
        ```url
        http://43.138.191.71:9200/gg
        ```

        - 返回结果
  
          ```json
          {
              "acknowledged": true,
              "shards_acknowledged": true,
              "index": "gg"
          }
          ```

        - 如果重复创建会导致以下结果
  
          ```json
          {
              "error": {
                  "root_cause": [
                      {
                          "type": "resource_already_exists_exception",
                          "reason": "index [gg/8QJuZPNGTu6hBTGIvs0Jdg] already exists",
                          "index_uuid": "8QJuZPNGTu6hBTGIvs0Jdg",
                          "index": "gg"
                      }
                  ],
                  "type": "resource_already_exists_exception",
                  "reason": "index [gg/8QJuZPNGTu6hBTGIvs0Jdg] already exists",
                  "index_uuid": "8QJuZPNGTu6hBTGIvs0Jdg",
                  "index": "gg"
              },
              "status": 400
          }
          ```
  
    - 获取一个索引
    
      - 使用postman发送get请求
    
        ```url
        http://43.138.191.71:9200/gg
        ```
    
      - 获取到gg索引有关信息
    
        ```json
        {
            "gg": {
                "aliases": {},
                "mappings": {},
                "settings": {
                    "index": {
                        "routing": {
                            "allocation": {
                                "include": {
                                    "_tier_preference": "data_content"
                                }
                            }
                        },
                        "number_of_shards": "1",
                        "provided_name": "gg",
                        "creation_date": "1652965618126",
                        "number_of_replicas": "1",
                        "uuid": "8QJuZPNGTu6hBTGIvs0Jdg",
                        "version": {
                            "created": "8020099"
                        }
                    }
                }
            }
        }
        ```
    
      - 获取所有索引有关信息
    
        ```url
        http://43.138.191.71:9200/_cat/indices?v
        ```
    
        - 得到信息
    
          ```json
          health status index uuid                   pri rep docs.count docs.deleted store.size pri.store.size
          yellow open   gg    8QJuZPNGTu6hBTGIvs0Jdg   1   1          0            0       225b           225b
          
          ```
    
    - 而删除的操作和get以及put相同
    
    - 向索引中添加数据（使用post请求）
    
      ```json
      http://43.138.191.71:9200/gg/_doc
      
      //数据格式采用json放在请求体中
      {
          "_index": "gg",
          "_id": "s09N34ABbDOAJUDkIvzA",
          "_version": 1,
          "result": "created",
          "_shards": {
              "total": 2,
              "successful": 1,
              "failed": 0
          },
          "_seq_no": 0,
          "_primary_term": 2
      }
      ```
    
      - 返回的数据
    
        ```json
        {
            "_index": "gg",
            "_id": "s09N34ABbDOAJUDkIvzA",
            "_version": 1,
            "result": "created",
            "_shards": {
                "total": 2,
                "successful": 1,
                "failed": 0
            },
            "_seq_no": 0,
            "_primary_term": 2
        }
        ```
    
        - 其中_id是该记录的唯一标识，不指定的话有ES自动生成，可以在请求路径中指定
    
          ```url
          http://43.138.191.71:9200/gg/_doc/1001
          ```
    
        - 在此基础上如果使用put方式则进行的是修改操作，但是也可以使用http://43.138.191.71:9200/gg/_create/1001创建
    
      - 对比
    
        ```url
        http://43.138.191.71:9200/gg/_doc/1001 #post 有则修改无则创建
        http://43.138.191.71:9200/gg/_create/1001 #put 创建，如果有重复会报错
        http://43.138.191.71:9200/gg/_create/1001 #post 创建，如果有重复会报错
        http://43.138.191.71:9200/gg/_doc/1001 #post 有则修改无则创建
        ```
    
      - 以上展示的都是全量更新，不能对于局部的某些属性进行单独更新
    
      - 局部更新需要采用post(使用put会报错)
    
        ```url
        http://43.138.191.71:9200/gg/_update/1001 #post
        
        
        //以下为请求体
        {
            "doc":{
                "name":"杀死一只知更鸟"
            }
        }
        ```
    
      - 删除数据(使用delete方法)
    
        ```url
        http://43.138.191.71:9200/gg/_doc/1001
        ```
    
        
    
    - 查询
    
      - 查询一条数据
    
        ```url
        http://43.138.191.71:9200/gg/_doc/1001 #get
        ```
    
      - 查询一个索引下的所有数据
    
        ```url
        http://43.138.191.71:9200/gg/_search 
        ```
    
        - 请求体方式
    
          ```json
          {
              "query":{
                  "match_all":{
                      
                  }
              }
          }
          ```
    
          
    
      - 条件查询(自适应模糊查询)
    
        - 第一种方式（url上填写参数）
    
          ```url
          http://43.138.191.71:9200/gg/_search?q=name:乡土中国
          ```
    
        - 第二种方式（在请求体上填写参数）
    
          ```json
          http://43.138.191.71:9200/gg/_search
          
          //请求体
          {
              "query":{
                  "match":{
                      "name":"乡土中国"
                  }
              }
          }
          
          ```
    
      - 分页查询
    
        ```json
        {
            "query":{
                "match_all":{
        
                }
            },
            "from":2,
            "size":2
        }
        ```
    
        - from=(需要显示的页码-1)*每页的大小
        - size=每页的大小
    
      - 限制显示查询(只想要显示某个字段)
    
        ```json
        {
            "_source":["name"]
        }
        ```
    
      - 排序查询
    
        ```json
        {   
            "sort":{
                "price":{
                    
                }
            }
        }
        //对price字段排序,默认升序
        
        
        {   
            "sort":{
                "price":{
                    "order":"desc"//指定排序规则 desc 为降序，asc为升序
                }
            }
        }
        ```
    
      - 多条件查询
    
        ```json
        {
            "query":{
                "bool":{
                    "must":[
        
                        {
                            "match":{
                            "name":"乡土中国"
                            }
                        },
        
                        {
                            "match":{
                            "price": 1
                            }
                        }
                        
                    ]
                }
            }
        }
        
        //将must改为should代表or，must代表and
        ```
    
      - 过滤器
    
        ```json
        {
            "query":{
                "bool":{
                
                    "filter":{
                        "range":{
                            "price":{
                                "gt":3000
                            }
                        }
                    }
                }
            }
        }
        
        //gt大于，ge大于或等于，ne是不等于，eq是等于，lt小于，le小于或等于
        ```
    
      - 非全文检索
      
        ```json
        {
            "query":{
                "match_phrase":{
                    "name":"乡土"
                }
            }
        }
        ```
      
        - 数据存储在文档中会进行分词，使用match的查询会对关键字每个进行拆分并单个查询，而采用match_phrase则不会分词但是依旧能实现模糊查询的功能
      
      - 聚合查询（将相同字段的相同的值进行统计）
      
        ```json
        {
            "aggs":{
                "自定义组名price_group":{
                    "terms":{
                        "field":"price"
                    }
                }
            }
        }
        ```
      
        - 返回结果
      
        ```json
        {
            "aggregations": {
                "自定义组名price_group": {
                    "doc_count_error_upper_bound": 0,
                    "sum_other_doc_count": 0,
                    "buckets": [
                        {
                            "key": 1,
                            "doc_count": 1
                        },
                        {
                            "key": 123,
                            "doc_count": 1
                        },
                        {
                            "key": 12311,
                            "doc_count": 1
                        },
                        {
                            "key": 100123,
                            "doc_count": 1
                        },
                        {
                            "key": 1223411,
                            "doc_count": 1
                        },
                        {
                            "key": 12234112981291,
                            "doc_count": 1
                        }
                    ]
                }
            }
        }
        ```
      
      - 查询平均值
      
        ```json
        {
            "aggs":{
                "price_avg":{
                    "avg":{
                        "field":"price"
                    }
                }
            }
        }
        ```
      
      - 设置值映射
      
        ```json
        {
            "properties":{
                "name":{
                    "type":"text",
                    //text代表可以分词
                    "index":true
                },
                "sex":{
                    "type":"keyword",
                    //keyword代表不能分词
                    "index":true
                },
                "email":{
                    "type":"keyword",
                    "index":false
                    //index为false代表无法索引查询
                }
            }
        }
        ```
    
  - 使用Springboot整合Elasticsearch
  
    - 导入依赖
  
      ```xml
       <dependency>
                  <groupId>org.elasticsearch.client</groupId>
                  <artifactId>elasticsearch-rest-high-level-client</artifactId>
              </dependency>
      ```
  
    - 创建配置类（自定义的）
  
      ```java
      @Configuration
      public class ElasticsearchUtil {
      
        
          @Value("${Elasticsearch.host}")
          String host;
      
          @Value("${Elasticsearch.port}")
          Integer port;
      
          @Bean
          RestHighLevelClient createRestClient()
          {
              RestClientBuilder builder= RestClient.builder(new HttpHost(host,port));
              return new RestHighLevelClient(builder);
          }
      
      }
      
      ```
  
    - 测试创建索引
  
      ```java
       @Test
          void testES() throws IOException {
              CreateIndexRequest request=new CreateIndexRequest("es");
              CreateIndexResponse response = client.indices().create(request, RequestOptions.DEFAULT);
              System.out.println(response.isShardsAcknowledged());
          }
      ```
  
    - 获取索引
  
      ```java
        @Test
          void testGetSearch() throws IOException {
              GetIndexRequest request = new GetIndexRequest("gg");
              GetIndexResponse response = client.indices().get(request, RequestOptions.DEFAULT);
              System.out.println(response.getAliases());
              System.out.println(response.getDataStreams());
              System.out.println(Arrays.toString(response.getIndices()));
              System.out.println(response.getMappings());
              System.out.println(response.getSettings());
      
          }
      
      //结果
      {gg=[]}
      {}
      [gg]
      {gg=org.elasticsearch.cluster.metadata.MappingMetadata@b2d8563c}
      {gg={"index.creation_date":"1653039694552","index.number_of_replicas":"1","index.number_of_shards":"1","index.provided_name":"gg","index.uuid":"69jqXw9XSNC5MNu2-mjYWg","index.version.created":"7090399"}}
      
      ```
  
      - 与直接通过postman获取到的信息差不多
  
        ```json
        {
            "gg": {
                "aliases": {},
                "mappings": {
                    "properties": {
                        "author": {
                            "type": "text",
                            "fields": {
                                "keyword": {
                                    "type": "keyword",
                                    "ignore_above": 256
                                }
                            }
                        },
                        "name": {
                            "type": "text",
                            "fields": {
                                "keyword": {
                                    "type": "keyword",
                                    "ignore_above": 256
                                }
                            }
                        },
                        "price": {
                            "type": "long"
                        }
                    }
                },
                "settings": {
                    "index": {
                        "creation_date": "1653039694552",
                        "number_of_shards": "1",
                        "number_of_replicas": "1",
                        "uuid": "69jqXw9XSNC5MNu2-mjYWg",
                        "version": {
                            "created": "7090399"
                        },
                        "provided_name": "gg"
                    }
                }
            }
        }
        ```
  
      - 删除索引和查询和创建差不多
  
    - 使用索引
  
      - 插入文档
  
        ```java
        @Test
        void testinsert() throws IOException {
        
            IndexRequest request=new IndexRequest();
            User user = new User();
            user.setUName("zhangsan");
            user.setUPassword("sksksksk");
            User one = userService.findOne(user);
        
            IndexRequest gg = request.index("gg").id("1010").source(new ObjectMapper().writeValueAsString(user), XContentType.JSON);
            IndexResponse response = client.index(gg, RequestOptions.DEFAULT);
            System.out.println(response);
        }
        ```
  
      - 查询数据
  
        ```java
        @Test
        void testsearch() throws IOException {
        
            GetRequest request = new GetRequest();
            GetResponse response = client.get(request.index("gg").id("1010"), RequestOptions.DEFAULT);
            System.out.println(response);
        }
        ```
  
      - 批量添加
  
        ```java
        @Test
        void testBuchadd() throws IOException {
            BulkRequest request = new BulkRequest();
            List<Book> book = bookService.list(null);
            for (Book book1 : book) {
                request.add(new IndexRequest().index("gg").id(String.valueOf(book1.getBId())).source(new ObjectMapper().writeValueAsString(book1),XContentType.JSON));
            }
        
            BulkResponse responses = client.bulk(request, RequestOptions.DEFAULT);
            for (BulkItemResponse item : responses.getItems()) {
                System.out.println(item);
            }
        }
        ```
        
      - 全量查询
      
        ```java
        @Test
        void testsearch2() throws IOException {
        
            SearchRequest request=new SearchRequest();
            SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();
        
            searchSourceBuilder.query(QueryBuilders.matchAllQuery());
            request.indices("gg").source(searchSourceBuilder);
            SearchResponse search = client.search(request, RequestOptions.DEFAULT);
            System.out.println(search.getTook());//时间
            for (SearchHit hit : search.getHits()) {
                System.out.println(hit.getSourceAsString());//查询的记录
            }
        }
        ```
      
      - 条件查询
      
        ```java
        @Test
        void testtermsearch() throws IOException {
            SearchSourceBuilder sourceBuilder=new SearchSourceBuilder().query(QueryBuilders.matchQuery("name","乡土中国"));
            SearchRequest request=new SearchRequest("gg").source(sourceBuilder);
            SearchResponse response = client.search(request, RequestOptions.DEFAULT);
            for (SearchHit hit : response.getHits()) {
                System.out.println(hit.getSourceAsString());
            }
        
        }
        ```
      
      - 条件查询和排序和size限制和查询限制
      
        ```java
        @Test
        void testtermsearch() throws IOException {
            SearchSourceBuilder sourceBuilder=new SearchSourceBuilder().query(QueryBuilders.matchQuery("name","乡土中国"));
            //SearchSourceBuilder sourceBuilder=new SearchSourceBuilder().query(QueryBuilders.termQuery("name","乡土中国"));
        
            sourceBuilder.sort("price", SortOrder.ASC);
            sourceBuilder.fetchSource(new String[]{"name","price"},new String[]{"author"});
            sourceBuilder.from(3);
            sourceBuilder.size(2);
            SearchRequest request=new SearchRequest("gg").source(sourceBuilder);
            SearchResponse response = client.search(request, RequestOptions.DEFAULT);
            for (SearchHit hit : response.getHits()) {
                System.out.println(hit.getSourceAsString());
            }
        
        }
        ```
      
      - 组合条件查询
      
        ```java
        @Test
        void testsearch3() throws IOException {
        
        
            SearchSourceBuilder sourceBuilder=new SearchSourceBuilder();
        
            //编写组合条件
            BoolQueryBuilder boolQueryBuilder = QueryBuilders.boolQuery();
            boolQueryBuilder.should(QueryBuilders.matchQuery("name","乡土中国"));
            boolQueryBuilder.should(QueryBuilders.matchQuery("bname","杀死一只知更鸟"));
        
            sourceBuilder.query(boolQueryBuilder);
            SearchRequest request=new SearchRequest("gg").source(sourceBuilder);
        
            SearchResponse response = client.search(request, RequestOptions.DEFAULT);
            for (SearchHit hit : response.getHits()) {
                System.out.println(hit.getSourceAsString());
            }
        }
        ```
      
      - 查询平均值
      
        ```java
        @Test
        void testsearch5() throws IOException {
            AggregationBuilder aggregation= AggregationBuilders.max("price").field("price");
        
            SearchSourceBuilder sourceBuilder=new SearchSourceBuilder().aggregation(aggregation);
        
            SearchRequest request=new SearchRequest("gg").source(sourceBuilder);
            SearchResponse response = client.search(request, RequestOptions.DEFAULT);
        
            for (SearchHit hit : response.getHits()) {
                System.out.println(hit.getSourceAsString());
            }
        }
        ```
      
      - 分组查询
      
        ```java
        @Test
        void testsearch6() throws IOException {
            AggregationBuilder aggregation= AggregationBuilders.terms("price").field("price");
        
            SearchSourceBuilder sourceBuilder=new SearchSourceBuilder().aggregation(aggregation);
        
            SearchRequest request=new SearchRequest("gg").source(sourceBuilder);
            SearchResponse response = client.search(request, RequestOptions.DEFAULT);
        
            for (SearchHit hit : response.getHits()) {
                System.out.println(hit.getSourceAsString());
            }
        }
        ```

