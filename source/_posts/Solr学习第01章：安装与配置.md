title: Solr学习第01章：安装与配置
date: 2015-10-28 10:01:31
categories: solr
tags: [solr,搜索]
---
# 一、Solr简介
Apache Solr官网
> Solr is highly reliable, scalable and fault tolerant, providing distributed indexing, 
replication and load-balanced querying, automated failover and recovery, centralized configuration and more. 
Solr powers the search and navigation features of many of the world's largest internet sites. --Apache Solr官网
百度百科
> Solr是一个高性能，采用Java5开发，基于Lucene的全文搜索服务器。同时对其进行了扩展，提供了比Lucene更为丰富的查询语言，
同时实现了可配置、可扩展并对查询性能进行了优化，并且提供了一个完善的功能管理界面，是一款非常优秀的全文搜索引擎。--百度百科

总而言之，Solr是一个开源的基于java的搜索解决方案，如果你想部署自己的搜索服务，不想受制于第三方搜索服务的限制，那么选择Solr吧，它一定不会让你失望
# 二、Solr的安装

## 官方下载地址
下载地址 `http://lucene.apache.org/solr/mirrors-solr-latest-redir.html`

## CentOS 7 下solr的安装部署
Solr版本 `5.7.1`
Solr在5.0版本之后取消了对于web容器的依赖，或者说，内置了Jetty容器而不是以war包的形式发布。5.0版本后的部署非常简单，达到了开箱即用的效果。
```
cd /opt
wget http://mirror.bit.edu.cn/apache/lucene/solr/5.3.1/solr-5.3.1.tgz
tar zxvf solr-5.3.1.tgz -C /opt
cd /opt/solr-5.3.1/bin
./solr start #启动
./solr stop #停止
./solr restart -p 8983 #重启8983端口的solr
./solr start -help #详细帮助，可以切换端口，重设solrhome等
```
启动后，会看到
```
Waiting up to 30 seconds to see Solr running on port 8983 [/]  
Started Solr server on port 8983 (pid=1248). Happy searching!
```
这时候访问 host:port/solr/（本地默认为http://localhost:8983/solr/ ） 如果看到solr的网页管理页面，大功告成！是不是很easy？

管理页面左侧有几个标签：
+ Dashboard 可以看的solr的基本信息、配置、内存使用等
+ Logging 网页版的日志输出，如果你在使用时遇到错误，可以在这里看到详细的报错信息
+ Core Admin 管理你的Core，比如你更新了Core的配置，不用重启solr实例，在这里refresh一下就好
+ Java Properties jvm的参数信息
+ Thread Dunp 所有的线程信息
+ Core Selector 选择一个Core，查看配置，进行自定义的查询等等

# 配置solrhome
如果保持默认设置，你的`{solrhome}`位于`{solr安装位置}/server/solr`，这个目录很重要，你的索引配置，搜索配置，数据等文件都位于这个目录下，我们接下来的工作，主要就在这个目录进行。为了方便，我们现在改变solrhome的位置，其实很简单，在启动solr的时候加入参数`-s <dir>`即可
在此之前，先在`/opt`下建立`mysolrhome`文件夹，而且`{solrhome}`要求必须含有一个`solr.xml`文件，我们直接从默认目录拷贝过来
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<solr>
  <solrcloud>
    <str name="host">${host:}</str>
    <int name="hostPort">${jetty.port:8983}</int>
    <str name="hostContext">${hostContext:solr}</str>
    <bool name="genericCoreNodeNames">${genericCoreNodeNames:true}</bool>
    <int name="zkClientTimeout">${zkClientTimeout:30000}</int>
    <int name="distribUpdateSoTimeout">${distribUpdateSoTimeout:600000}</int>
    <int name="distribUpdateConnTimeout">${distribUpdateConnTimeout:60000}</int>
  </solrcloud>
  <shardHandlerFactory name="shardHandlerFactory"
    class="HttpShardHandlerFactory">
    <int name="socketTimeout">${socketTimeout:600000}</int>
    <int name="connTimeout">${connTimeout:60000}</int>
  </shardHandlerFactory>
</solr>
```
然后在bin目录输入
```
./solr stop -p 8983 #停止时要用-p指明停止的是哪个端口的solr
./solr start -s /opt/mysolrhome
```
打开管理页面，看到运行信息中
```
-Dsolr.solr.home=/opt/mysolrhome
```
就说明成功了