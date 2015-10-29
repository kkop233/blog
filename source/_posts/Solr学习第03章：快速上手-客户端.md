title: Solr学习第03章：快速上手-客户端
date: 2015-10-29 11:23:30
categories: solr
tags: [solr,搜索]
---
# SolrJ使用
上一章我们配置了索引，导入了数据，并且实现了再solr的管理页面搜索数据。但是我们现在面临一个问题，虽然可以用http的方式与solr交互了，但非常的不方便。请求必须自己拼凑字符串，response也必须自己手动处理长长的json字符串，成功失败的状态也必须自己解析等等。
好在Solr的开发者已经帮助我们开发好了各种语言使用的客户端，这个客户端可以类比为连接MySQL的JDBC。
我们先建立个简单的Java项目，拷贝`solr-5.3.1\dist`目录下的`solr-solrj-5.3.1.jar`，以及`solr-5.3.1\dist\solrj-lib`下的所有jar引入工程，可能还缺少一个commons-logging.jar，百度下载即可。
jar包导入后编写代码：
Student.java
```java
package solrj;
//省略import
public class Student {
    @Field
    private int id;
    @Field
    private String name;
    @Field
    private Date birthday;
    @Field
    private double avg_score;
    @Field
    private List<String> hobby;
    @Field
    private String motto;
    public Student() {
    }
    public Student(int id, String name, Date birthday, double avg_score,
            List<String> hobby) {
        this.id = id;
        this.name = name;
        this.birthday = birthday;
        this.avg_score = avg_score;
        this.hobby = hobby;
    }
    
    @Override
    public String toString() {
        return "Student [id=" + id + ", name=" + name + ", birthday="
                + birthday + ", avg_score=" + avg_score + ", hobby=" + hobby
                + ", motto=" + motto + "]";
    }
    
      // 省略get set方法
}
```
注意Student中的field必须用@Field注解
Main.java
```java
package solrj;
//省略import
public class Main {
    
    static String url="http://host:port/solr/student";   //注意改成你的地址
    
    private static void add() throws SolrServerException, IOException {
        List<String> list=new ArrayList<String>();
        list.add("暴走");
        list.add("漫画");
        list.add("篮球");
        SolrInputDocument doc=new SolrInputDocument();
        doc.setField("id", 666);
        doc.setField("name", "王尼玛");
        doc.setField("birthday", new Date());
        doc.setField("avg_score", 9.9);
        doc.setField("hobby", list); // list可以是继承Collection的对象
        doc.setField("motto", "fxxk my life");
        
        SolrClient server = new    HttpSolrClient(url);
        UpdateResponse response=server.add(doc);
        server.commit();
        server.close();
        System.out.println(response.toString());
    }
    
    private static void search() throws SolrServerException, IOException {
        SolrClient server = new    HttpSolrClient(url);
        SolrQuery query=new SolrQuery();
        query.set("q", "hobby:篮球 AND id:[2 TO *]");
        QueryResponse response=server.query(query);
        List<Student> list=response.getBeans(Student.class);
        server.close();
        for (Student student : list) {
            System.out.println(student.toString());
        }
    }
    
    public static void main(String[] args) throws SolrServerException, IOException {
        add();
        search();
    }
}
```
注意事项：
+ add是用solrj工具添加了一条记录，search是搜索hobby含有篮球并且id大于2的对象
+ SolrClient 对象在使用完后需要关闭，在实际使用中，一个SolrClient对象可以被多线程复用
+ Java bean里面需要存储的字段注意使用注解标注

运行结果：
```bash
{responseHeader={status=0,QTime=58}}
Student [id=3, name=李明, birthday=Tue Jan 01 08:00:00 CST 1991, avg_score=111.0, hobby=[篮球, 看电影, 夜跑], motto=null]
Student [id=666, name=王尼玛, birthday=Thu Oct 29 11:32:22 CST 2015, avg_score=9.9, hobby=[暴走, 漫画, 篮球], motto=fxxk my life]  
```