# 一、简介

在很多网站上都具有一个全局搜索的功能，例如京东、淘宝等等大型的项目，而elasticsearch就是一种提供全局搜索的搜索引擎框架，并且提供了分布式，基于RESTful web接口

ElasticSearch有很多使用场景：维基百科的全文检索、电商网站的检索商品、日志数据分析使用ELK技术等等

在使用ElasticSearch之前先了解一些基本概念：

- Index：索引，相当于mysql中的数据库
- Type：类型，相当于mysql中的表
- Document：文档，相当于mysql中的某个table的内容

也就是说在mysql中要定位一个数据需要通过 数据库--> 表 --> 内容，那么在ElasticSearch中也类似，Index --> Type --> Document，这两者的概念非常相似，当然关于Type在高版本中的使用后面再说

ElasticSearch的主要功能便是搜索，而mysql中也可以使用搜索例如like等等模糊匹配，ElasticSearch的效率要比mysql中的高，因为mysql中的数据存在磁盘中，而ElasticSearch中的数据存在内存中这是其一，其二ElasticSearch中使用了倒排索引机制，使得ElasticSearch做搜索引擎效率较高

关于倒排索引的具体可以参考https://blog.csdn.net/andy_wcl/article/details/81631609这篇文章，本文只是初步使用不对其原理进行深入解析

# 二、安装

使用docker安装ElasticSearch非常方便，所以这里选择使用docker安装ElasticSearch，步骤如下

1. 下载镜像文件

```
docker pull elasticsearch:7.4.2 存储和检索数据
docker pull kibana:7.4.2 可视化检索数据
```

2. 创建实例

```shell
mkdir -p /mydata/elasticsearch/config
mkdir -p /mydata/elasticsearch/data
echo "http.host: 0.0.0.0" >> /mydata/elasticsearch/config/elasticsearch.yml
chmod -R 777 /mydata/elasticsearch/ 	#保证权限

docker run --name elasticsearch -p 9200:9200 -p 9300:9300 \
-e "discovery.type=single-node" \
-e ES_JAVA_OPTS="-Xms64m -Xmx512m" \
-v /mydata/elasticsearch/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml \
-v /mydata/elasticsearch/data:/usr/share/elasticsearch/data \
-v /mydata/elasticsearch/plugins:/usr/share/elasticsearch/plugins \
-d elasticsearch:7.4.2
```

3. 启动

```shell
docker run --name kibana -e ELASTICSEARCH_HOSTS=http://192.168.18.107:9200 -p 5601:5601 -d kibana:7.4.2
```

在kibana连接elasticsearch过程中，可能会报错，可以参考https://blog.csdn.net/whatday/article/details/107879989

使用浏览器访问192.168.18.107:9200，则说明搭建成功

![image-20210311091529907](http://cdn.noteblogs.cn/image-20210311091529907.png)

kibana客户端：192.168.18.107:5601

# 三、文档增删改查

### 3.1 _cat

- GET /_cat/nodes： 查看所有节点
- _GET /_cat/health： 查看 es 健康状况
- GET /_cat/master： 查看主节点_
- _GET /_cat/indices： 查看所有索引  show databases;  

### 3.2 保存一个文档

```
PUT customer/external/1
{
  "name":"zhangsan"
}
```

customer/external/1，索引/类型/文档，类似于mysql中的数据库/表/字段