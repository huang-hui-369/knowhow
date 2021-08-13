# 全文检索相关技术
Lucene：如果使用该技术实现，需要对Lucene的API和底层原理非常了解，而且需要编写大量的Java代码。
Solr：使用java实现的一个web应用，可以使用rest方式的http请求，进行远程API的调用。
ElasticSearch(ES)：可以使用rest方式的http请求，进行远程API的调用。

Solr和ElasticSearch底层都是基于Lucene

# Solr和ES的比较
ElasticSearch vs Solr 检索速度

当单纯的对已有数据（静态数据，不会发生变化的数据）进行搜索时，Solr更快。


当实时建立索引时, Solr会产生io阻塞，查询性能较差, Elasticsearch具有明显的优势。


随着数据量的增加，Solr的搜索效率会变得更低，而Elasticsearch却没有明显的变化。


大型互联网公司，实际生产环境测试，将搜索引擎从Solr转到Elasticsearch以后的平均查询速度有了50倍的提升。


# 总结：

二者安装都很简单；
Solr 利用 Zookeeper 进行分布式管理，而 Elasticsearch 自身带有分布式协调管理功能;
Solr 支持更多格式的数据，而 Elasticsearch 仅支持json文件格式；
Solr 官方提供的功能更多，而 Elasticsearch 本身更注重于核心功能，高级功能都由第三方插件提供；
Solr 在传统的搜索应用中表现好于 Elasticsearch，但在处理实时搜索应用时效率明显低于Elasticsearch。
最终的结论：
Solr 是传统搜索应用的有力解决方案，但 Elasticsearch 更适用于新兴的实时搜索应用。


原文链接：https://blog.csdn.net/weixin_41947378/article/details/109405386

