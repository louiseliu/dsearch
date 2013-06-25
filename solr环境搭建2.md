（转）http://www.cnblogs.com/reveewu/archive/2013/03/28/2987168.html

按照我的理解，使用solr搜索的大致流程为：配置tomcat--->配置solr---->配置中文分词---->引用solrnet生成索引---->访问索引服务


步骤一：安装tomcat

　　版本：tomcat 7.0

　　下载地址：http://mirror.bjtu.edu.cn/apache/tomcat/tomcat-7/v7.0.39/bin/apache-tomcat-7.0.39.exe

　　这个是安装版的，很简单，当然安装之前别忘了先准过java环境哈

　　访问http://localhost:8080/即可显示tomcat主页

步骤二：安装solr

　　版本：4.2

　　下载地址：http://www.apache.org/dyn/closer.cgi/lucene/solr/4.2.0

　　解压后得到如下目录

 　　

　　1.把/dist/solr-4.2.0.war拷贝到tomcat目录/webapps/下面，更改文件名为solr.war，重启tomcat后会自动生成solr文件夹

　　2.我们要使用多个core的solr服务，所以拷贝/example/multicore/文件夹到D盘根目录，并更改文件名为solr，作为SolrHome使用，完整路径为d:/solr-4.2.0/solr

　　3.在tomcat目录/conf/Catalina/localhost/下添加配置文件solr.xml，内容如下：

<Context docBase="D:/Tomcat 7.0/webapps/solr" reloadable="true" > 
<Environment name="solr/home" type="java.lang.String" value="D:/solr-4.2.0/solr" override="true" /> 
</Context>
　　4.重启tomcat，访问http://localhost:8080/solr/，不出意外可以显示如下界面

　　

　　5.默认自带了core0 core1两个core，现在我们自己创建一个名为test的core，可以直接复制core0文件，更改为test，然后使用solr管理界面中的core admin 添加就可以了，或者直　　　　接solr.xml重启tomcat也可以

步骤三：安装IK中文分词器

　　版本：2012-FF hotfix 1

　　下载地址：https://code.google.com/p/ik-analyzer/downloads/detail?name=IK%20Analyzer%202012FF_hf1.zip&can=2&q=

　　1.将 IKAnalyzer.cfg.xml，IKAnalyzer2012FF_u1.jar，stopword.dic 拷贝到tomcat的/webapps/solr/WEB-INF/lib/下面

　　2.为刚才名为test的core配置IK分词，打开test/conf/schema.xml,加入以下配置：

　　

<!-- ik analyzer -->
    <fieldType name="text_ik" class="solr.TextField">    
        <analyzer type="index" isMaxWordLength="false" class="org.wltea.analyzer.lucene.IKAnalyzer"/>    
        <analyzer type="query" isMaxWordLength="true" class="org.wltea.analyzer.lucene.IKAnalyzer"/>    
    </fieldType>
　　3.重启tomcat，没有意外的话，IK已经可以初步使用了，我们在solr管理中打开test core，点击analysis,field type选择text_ik，然后输入一串中文，就可以显示分词结果

