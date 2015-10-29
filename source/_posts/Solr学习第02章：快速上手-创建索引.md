title: Solr学习第02章：快速上手-创建索引
date: 2015-10-29 11:23:16
categories: solr
tags: [solr,搜索]
---
快速上手旨在最快速的建立一个可用的Demo工程，对solr的整个使用过程做一个概况的了解，忽略详细难以理解的超多配置，对进一步的学习起到构建大致框架的作用。
# 一、Core的建立
在Solr中，一个core就可以简单的类比为关系数据库中的表，一个core可以对应web开发中的java bean，例如，我们可以建立一个“学生”core：
## 1. 手动建立core目录和配置文件
打开{solrhome}，以`student` core 为例
    + 建立`student/conf`文件夹
    + 在`conf`下创建`schema.xml`和`solrconfig.xml`文件
这两个文件的sample在你下载的solr程序的`example`文件夹下有，带很多注释，我们编辑后的两个文件为：
schema.xml
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<schema name="student" version="1.5">
<field name="id" type="int" indexed="true" stored="true" required="true" multiValued="false" />
<field name="name" type="string" indexed="true" stored="true" required="true" multiValued="false" />
<field name="birthday" type="date" indexed="true" stored="true" required="true" multiValued="false" />
<field name="avg_score" type="double" indexed="true" stored="true" required="true" multiValued="false" />
<field name="hobby" type="string" indexed="true" stored="true" required="true" multiValued="true" />
<field name="motto" type="string" indexed="false" stored="true" required="false" multiValued="false" />
<uniqueKey>id</uniqueKey>
<field name="_version_" type="long"     indexed="true"  stored="true"/>
<fieldtype name="string"  class="solr.StrField"/>
<fieldType name="int" class="solr.TrieIntField" precisionStep="0" positionIncrementGap="0"/>
    <fieldType name="double" class="solr.TrieDoubleField" precisionStep="0" positionIncrementGap="0"/>
<fieldType name="date" class="solr.TrieDateField" precisionStep="0" positionIncrementGap="0"/>
<fieldType name="long" class="solr.TrieLongField" precisionStep="0" positionIncrementGap="0"/>
</schema>
```
一个学生我们定义了6个字段，其中值得注意的是：
    + 有个主键“id”，没什么特殊需求的话这个字段必须有，用uniqueKey来指出，也可以是string
    + hobby（爱好）multiValued=true，这个字段可以有多个值
    + motto（座右铭）indexed=false，这个字段不能被用于检索和排序
solrconfig.xml 这个文件很长，但是刚开始不用每一项都搞清楚，直接使用默认配置就可以
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<config>
  <luceneMatchVersion>5.2.0</luceneMatchVersion>
  <dataDir>${solr.data.dir:}</dataDir>
  <directoryFactory name="DirectoryFactory" 
                    class="${solr.directoryFactory:solr.NRTCachingDirectoryFactory}">
  </directoryFactory> 
  <codecFactory class="solr.SchemaCodecFactory"/>
  <schemaFactory class="ClassicIndexSchemaFactory"/>
  <indexConfig>
    <lockType>${solr.lock.type:native}</lockType>
    <infoStream>true</infoStream>
  </indexConfig>
  <updateHandler class="solr.DirectUpdateHandler2">
    <updateLog>
      <str name="dir">${solr.ulog.dir:}</str>
    </updateLog>
    <autoCommit> 
       <maxTime>${solr.autoCommit.maxTime:1500}</maxTime> 
       <openSearcher>true</openSearcher> 
    </autoCommit>
  </updateHandler>
  <query>
    <maxBooleanClauses>1024</maxBooleanClauses>
  </query>
  <requestHandler name="/select" class="solr.SearchHandler">
    <lst name="defaults">
       <str name="echoParams">explicit</str>
       <int name="rows">10</int>
       <str name="df">text</str>
     </lst>
    </requestHandler>
  <requestHandler name="/query" class="solr.SearchHandler">
     <lst name="defaults">
       <str name="echoParams">explicit</str>
       <str name="wt">json</str>
       <str name="indent">true</str>
       <str name="df">text</str>
     </lst>
  </requestHandler>
  <admin>
    <defaultQuery>*:*</defaultQuery>
  </admin>
</config>
```
为了方便，autocommit设置为1.5秒提交一次，同时opwensearcher=true保证每次提交后新数据的可见性
## 2. 在管理页面建立core
打开solr管理页面，在`Core Admin`选项里选择`Add Core`，这里会有几个属性：

+ **name** core的名称，和你的core文件夹名相同，这里是student
+ **instanceDir** core文件夹相对于solrhome的位置，这里就在solrhome下，直接填student，如果位于子目录下，就填比如`collection1/student`这种形式
+ **dataDir** core数据储存位置，这里默认
+ **config** solrconfig位置，默认
+ **schema** schema位置，默认

如果在建立core的时候发生错误，管理页面会一直有个错误提示去不掉，你可以完全删除core文件夹，重启solr再重复建core操作，即可解决。

# 二、数据导入
使用upload方法导入一批数据。打开管理页面 core-selector->student->Documents，选择json方式分别导入了以下三条数据：
```json
  {
    "id": 1,
    "name": "James",
    "birthday": "1991-01-01T00:00:00Z",
    "avg_score": 111,
    "hobby": [
      "篮球"
    ],
    "motto": "业精于勤,荒于嬉.行成于思,毁于随"
  }
  {
    "id": 2,
    "name": "小王",
    "birthday": "1991-01-01T00:00:00Z",
    "avg_score": 111,
    "hobby": [
      "唱K"
    ],
    "motto": "To be, or not to be: that is the question"
  }
  {
    "id": 3,
    "name": "李明",
    "birthday": "1991-01-01T00:00:00Z",
    "avg_score": 111,
    "hobby": [
      "篮球",
      "看电影",
      "夜跑"
    ]
  }
```
这样一来，索引就算是创建好了，而且我们也导入了需要搜索的数据，打开Query界面，可以在这里进行简单的查询，看看我们刚刚导入的数据如何。
修改"q"字段为 `hobby:篮球`，点击查询，可以看到id=1,3的doc被查询了出来，说明我们导入的数据已经被成功索引了。注意，solr还帮我们在数据上方生成了这一次索引的请求url，http://182.92.158.145:8983/solr/student/select?q=hobby%3A%E7%AF%AE%E7%90%83&wt=json&indent=true

