## ELK（Elasticsearch、Logstash、Kibana）--搜索引擎Elasticsearch
版本7.6.2，**开源高扩展、分布式全文搜索引擎、近乎实时的存储检索数据。**
- doug cutting
1. 安装
2. 生态圈
3. 分词器 --ik
4. restFul操作es
5. 进阶CRUD
6. springboot集成ES(从原理开始)
7. 爬虫爬取数据（安全允许范围内）
8. 实战全文检索

**以后项目只要使用搜索，就可以使用ES，大数据的情况下**
### ES安装 head安装 kibana安装
jdk1.8起，11最好，java开发，es版本和我们之后的java的核心jar包，版本对应，jdk环境正常
Es windows下解压即可使用
- 安装可视化界面 head插件 解压
```
https://github.com/mobz/elasticsearch-head
```
- 安装node.js
- head文件夹下npm install下载head所需依赖
- npm run start启动
- 配置elasticsearch跨域
```
elasticearch.yml
在配置文件末尾添加如下内容，重新启动elasticsearch服务

http.cors.enabled: true
http.cors.allow-origin: "*"
```
- 重启es服务再次连接
- 创建索引，难以理解的时候可以将es看做是一个数据库（可以建索引（库），文档（库中的数据）），head 看做是一个数据展示工具，后续所有查询在kibana中做。
- 下载kibana 官网下载即可，版本和es版本号保持一致即可，解压很慢，是一个标准的工程，解压即用，启动.bat 访问localhost：5601
- 开发工具：post、curl、谷歌、kibana测试，之后的操作都在kibana的Dev tools操作
- 汉化：kiban.yml-il8n.locale: "zh-CN"，重启即可
#### 核心概念-面向文档、一切都是json
**1. 索引 建库**
**2. 字段类型 建立数据库对应字段规则**
**3. 文档 填充数据进去**

---

数据库-索引
行-文档
字段-json
**集群、节点、索引、类型、文档、分片、映射**
- es一个人就是一个集群；默认名字就是elasticsearch，起来就是分片
- 将索引划分多个分片，每个分片在集群的不同服务器之间迁移；
##### 文档
- 一个索引类型中有多个文档，索引一片文档时，可以通过索引-类型-文档-文档id，索引到具体的文档**ID不一定是整数，实际上是一个字符串**
- 最小单位是就是文档，包含key-value，**就是一条条数据**比如用户表中的1号用户2号用户那一行。
- 层次性，就是一个json对象（fastJason、Jackson自动转换）
##### 类型（很少用了），类似于数据库表
##### 索引-数据库
映射类型的容器，索引是一个非常大的文档集合，索引存储映射类型的其他字段和其他设置，然后存储到各个分片上
**一个集群至少有一个节点，一个节点就是一个es进程，节点可以有多个索引默认的，如果你创建索引，那么就会有5个分片（五合一叫主分片）构成，一个主分片有一个复制分片**
- 倒排索引：采用lucence倒排索引作为底层，这种结构适合快速的全文搜索，一个索引由文档中所有不重复的列表组成，对于每一个词，都有一个包含它的文档列表；针对相应的关键词显示内容，如果没有切合关键字，就不会显示相应的内容。**分片之后关键词作key，分片id作value**
- 比如说两句话，存在两个不同的文档中，每个不重复的词是一个列表，在进行检索的时候，就看检索的关键词，在文档中占有的比例或者说权重大，就优先显示相应的文档，1文档中包含两个关键词，2文档只包含一个，就会优先显示1文档

## IK分词器插件

分词：将要搜索的中文或别的划分成一个个关键字，我们在搜索的时候会将自己的信息进行分词，会将数据库中或者索引库中的数据进行分词没然后进行一个匹配操作；默认的中文分词将每一个子看成一个词，但是这不符合要求，所以我们需要安装中文分词器：ik分词器解决这个问题

---

IK提供两个分词算法：ik_smart , ik_max_word；其中ik_smart最少切分，ik_max_word最细粒度划分

1. 下载安装，版本号必须与es版本号一致。解压到elasticsearch下的plugin文件夹即可
2. 重启观察es，也可以通过elasticsearch-plugin list查看加载的插件
2. 使用kibana测试
![avatar](/photos/分词器效果.png)

3. 分词器字典中有时候没有需要的词，会将我们需要的关键词差分开，就需要将关键词加到自己的分词器配置中去：ik分词器中config-添加自己的dic然后重启即可

- restful风格
1. 基于索引的操作

创建规则：添加索引

```

PUT /test2
{
  "mappings": {
    "properties": {
      "name": {
        "type": "text"
      },
      "age":{
        "type": "integer"
      },
      "birthday":{
        "type": "date"
      }
    }
  }
}

GET test2 得到具体的信息
GET _cat/indices?v 获得当前索引信息
```

修改索引：put覆盖；覆盖后version号会增加
```
 "_index" : "test3",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 2,
  "result" : "updated",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 1,
  "_primary_term" : 1

```
post方式修改:
```
POST /test3/_doc/1/_update
{
  "doc": {
    "name": "Athos123"
  }
}
```

删除：delete

```
POST /test3/_doc/1/_update
{
  "doc": {
    "name": "Athos123"
  }
}
```

2. 基于文档的基本操作（重点）

    1. 基本操作
        1. 添加数据：
        ```
            PUT /athos/usr/1
            {
            "name": "athos",
            "age": 26,
            "desc": "一顿操作猛如虎，一看战绩0-5",
            "tags": ["暖男","直男","游戏男"]
            }

            PUT /athos/usr/3
            {
            "name": "lisi",
            "age": 34,
            "desc": "一顿操作猛如虎，一看战绩0-5",
            "tags": ["老男人","直男","死肥宅"]
            }
        ```

        2. 搜索：GET /athos/usr/3，简单的条件查询：GET /athos/usr/_search?q=name:athos，搜索都用get

        ```
            {
            "took" : 56,
            "timed_out" : false,
            "_shards" : {
                "total" : 1,
                "successful" : 1,
                "skipped" : 0,
                "failed" : 0
            },
            "hits" : {
                "total" : {
                "value" : 1,
                "relation" : "eq"
                },
                "max_score" : 0.9808291,
                "hits" : [
                {
                    "_index" : "athos",
                    "_type" : "usr",
                    "_id" : "1",
                    "_score" : 0.9808291,// 未来如果有很多查询结果，这个score就是结果的匹配度，匹配度越高分值越高
                    "_source" : {
                    "name" : "athos",
                    "age" : 26,
                    "desc" : "一顿操作猛如虎，一看战绩0-5",
                    "tags" : [
                        "暖男",
                        "直男",
                        "游戏男"
                                ]
                            }
                        }
                    ]
                }
            }
        ```

        3. 更新使用put或者post _update（推荐）,因为put方式在更新的时候如果有字段没有更新，原有的数据就会被清空
    2. 重点操作:排序，分页，模糊，精准查询，高亮

    java中对象与方法就是es中的key like source、query。。。。
    ```
        GET /athos/usr/_search
        {
            "query":{
                //模糊查询match
                "match": {
                "tags": "渣男"
                }
            },
             "_source": ["name","tags"]
             //通过source属性设置只显示的内容，结果的过滤；类似于select* 和select name，tags；

            //排序，根据年级降序排序
             "sort":{
                "age":{
                "order": "desc"
                }
            },
            //分页查询，下标从0开始
            //前端代码就是:/search/{current}/{pagesize}
            "from": 0,//数据起始位置
            "size": 1//返回数据条数
        }

        结果：
                #! Deprecation: [types removal] Specifying types in search requests is deprecated.
        {
        "took" : 2,
        "timed_out" : false,
        "_shards" : {
            "total" : 1,
            "successful" : 1,
            "skipped" : 0,
            "failed" : 0
        },
        //hits:索引和文档的信息；查询的结果总数以及具体的文档，遍历的结果；其中通过score判断谁符合结果；
        "hits" : {
            "total" : {
            "value" : 4,
            "relation" : "eq"
            },
            "max_score" : 0.6827427,
            "hits" : [
            {
                "_index" : "athos",
                "_type" : "usr",
                "_id" : "2",
                "_score" : 0.6827427,
                "_source" : {
                "name" : "athos1",
                "age" : 26,
                "desc" : "一顿操作猛如虎，一看战绩0-5",
                "tags" : [
                    "渣男",
                    "直男",
                    "游戏男"
                ]
                }
            },
            {
                "_index" : "athos",
                "_type" : "usr",
                "_id" : "4",
                "_score" : 0.6657745,
                "_source" : {
                "name" : "athos12",
                "age" : 3,
                "desc" : "一顿操作猛如虎，一看战绩0-5",
                "tags" : [
                    "渣男",
                    "猛男",
                    "死肥宅"
                ]
                }
            },
            {
                "_index" : "athos",
                "_type" : "usr",
                "_id" : "1",
                "_score" : 0.13755092,
                "_source" : {
                "name" : "athos",
                "age" : 26,
                "desc" : "一顿操作猛如虎，一看战绩0-5",
                "tags" : [
                    "暖男",
                    "直男",
                    "游戏男"
                ]
                }
            },
            {
                "_index" : "athos",
                "_type" : "usr",
                "_id" : "3",
                "_score" : 0.116015166,
                "_source" : {
                "name" : "lisi",
                "age" : 34,
                "desc" : "一顿操作猛如虎，一看战绩0-5",
                "tags" : [
                    "老男人",
                    "直男",
                    "死肥宅"
                ]
                }
            }
            ]
        }
        }
    ```
- bool值查询：must（and）必须同时满足条件；should（or）只要满足其中一个都会显示

同时满足多个条件的数据显示
![avatar](/photos/bool查询-must.png)


 满足其中一个条件的数据显示

![avatar](/photos/bool查询-should.png)

 除了条件以外的全部显示

![avatar](/photos/bool查询-mustnot.png)

过滤器自定义：gt大于，gte大于等于，lt小于，lte小于等于，配合使用可以实现区间查询

![avatar](/photos/过滤.png)

匹配多个条件查询：多个条件使用空格隔开，在match中的条件
 
- 精确查询trem，使用倒排索引查询，直接查询精确的数据
- match会使用分词器解析，先分析文档，然后通过分析的文档进行查询

- 两个类型： text类型和keyword类型；text类型可以被分词器解析，keyword则不行