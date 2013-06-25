solr4.0的IK中文分词、高亮、suggest、spellcheck、realtime get等  

http://sling2007.blog.163.com/blog/static/8473271320121136143011/

一、中文分词

 
mmseg
ik
Imdict
paoding
算法
mmseg算法
（正向最大匹配或者最细分词）
正向迭代最细粒度切分算法（正向最大匹配并且最细分词）
                   已经不继续更新
速度
complex 1200kb/s左右,
simple 1900kb/s左右
低于50W字/秒
50万字/秒
自定义词典
支持、不支持中英文混合
支持但不支持重加载（只能重启）
支持混合词
停止词
不支持
支持
官网
http://code.google.com/p/ik-analyzer/
http://code.google.com/p/mmseg4j/
http://code.google.com/p/imdict-chinese-analyzer/
http://code.google.com/p/paoding/

功能上，mmseg不支持中英文混合词，不支持停止词，只会对文档的每个字做一次分词，如果需要多种分词的话，需要额外一个字段。
而ik，不支持词典的重加载（只能初始化的时候加载一次），但是支持中英文混合词，会对文档进行所有分词。
当然这些只能自己去改代码了。。。
分词上，如果对“中华人民共和国”进行分词，
mmseg的complex方式的结果是“中华人民共和国”
mmseg的maxword方式的结果是“中华、华人、人民、共和、国”，其实maxword就是对complex进一步分成2个一组的词。如果需要用3个字的最小maxword方式的词，也需要改代码。
ik的分词结果是“中华人民共和国、中华人民、中华、华人、人民、人民共和国、共和国、共和、国”
可以看出ik的最细分词。
两者的利弊就看自己需求吧。。。。。。

        在solr中，加入IK分词器等中文分词的时候，和lucene类似。比如在solr的主节点加入mmseg4j分词的步骤如下：
        1、下载mmseg4j分词包mmseg4j-*。jar，上传到solr的war包解压后的WEB-INF/lib目录下，注意版本，早期的不支持solr。
         2、在需要使用mmseg分词的core的schema.xml文件中，加入fieldtype的定义
de>         <fieldtype name="textComplex" class="solr.TextField"
			positionIncrementGap="100">
			<analyzer>
				<tokenizer class="com.chenlb.mmseg4j.solr.MMSegTokenizerFactory"
					mode="complex" dicPath="dic/">
				</tokenizer>
			</analyzer>
	 </fieldtype>
	其中mode有simple、max-word、complex三种，
            simple和complex都是正向最大分词，simple效率高，complex准确率高； max-word则是最多分词。
            如果需要可以用copy字段，以对同一个文本，用2种分词，以保证搜索效果。
        dicPath是词典存放的路径，这里的相对路径以solrhome/solrcore为基础，建立dic文件夹后，把分词jar包下的*.dic加入即可。
        words-my.dic就是自定义词典的位置，需要用utf-8编码,my可以随便换成自己想用的名字。
   这样字段被索引的时候，就可以使用mmseg分词了。
如果自定义词典中需要增加新词，那么修改words-my.dic后，用一个自定义的handler即可触发。
<requestHandler name="/mmseg4j" class="com.chenlb.mmseg4j.solr.MMseg4jHandler"> 
<lst name="defaults"> 
<str name="dicPath">dic/</str> 
</lst> 
</requestHandler>de>
 需要重新加载词典的时候，访问一下http://localhost/solr/core0/mmseg4j即可。
MMseg4jHandler这个重新加载词典的逻辑大致是，先查看词典文件是否改变，然后把所有词（不只是自定义词典，还有chars.dic、unit.dic、word.dic）重加载。所以如果需要可以自己修改一下这个类，只加载words-my.dic即可，也可以dic目录下只放自定义词典，其他词典则会从mmseg4j的jar包中加载。

二、高亮
     在百度、谷歌搜索的时候，会发现自己输入的关键词在返回结果页中是红色高亮显示的，比如：


solr也提供了这个功能。可以在solrconfig.xml中配置默认的高亮，也可以在本文最下面的第五部分的查询代码中，显示指定高亮的字段、样式等。
java调用类似如下：
          String channelid = "0001";
        SolrSearcher searcher = new SolrSearcher(BaseSolrCreator.core_photoset_ + channelid);
                 String q = "text:刘来";
                searcher.setCommonCondition(q, null, null, "setid desc", 0, 1);
                searcher.setHighlightCondition("setname", "<em>", "</em>", 1);
                searcher.search();
                ArrayList<Map<String, Object>> resultList = searcher.getResultsWithHightlight();
                System.out.println(resultList);
三、suggest
          suggest可以用edgengram来分词实现，请参照http://wiki.apache.org/solr/AnalyzersTokenizersTokenFilters#solr.EdgeNGramFilterFactory，
ngram可以用back和front两种方式，最小分词的长度可以控制。
比如对“中华民国”分词，结果就是"中、中华、中华民、中华民国"，所以只需输入前缀，那么就能搜索出这个词条。

de><fieldType name="text_general_edge_ngram" class="solr.TextField" positionIncrementGap="100">
   <analyzer type="index">
      <tokenizer class="solr.LowerCaseTokenizerFactory"/>
      <filter class="solr.EdgeNGramFilterFactory" minGramSize="1" maxGramSize="80" side="front"/>
   </analyzer>
   <analyzer type="query">
      <tokenizer class="solr.LowerCaseTokenizerFactory"/>
   </analyzer>
</fieldType>de>


四、spellcheck、相关搜索
在谷歌、百度查询的时候，输入词后会有一个相关搜索，如下

在solr中可以通过一个词典文件或者索引文件，来达到这个效果。
在solrconfig.xml中需要配置大致如下，以开启spellcheck功能：
spellcheck源数据可以是索引也可以是文件，索引可以是主索引，也可单独建一个索引。
spellcheck的字段最好是schema中定义的string类型，如果用复杂类型，这里反而会失控。
我测试中，中文的spellcheck效果很差。不建议使用。
  <searchComponent name="spellcheck" class="solr.SpellCheckComponent">
   <lst name="spellchecker">
      <str name="name">index_dic</str> 
      <str name="classname">solr.IndexBasedSpellChecker</str>
      <str name="field">search_text</str>
      <str name="spellcheckIndexDir">./indexspellchecker</str>
      <str name="buildOnOptimize">true</str>
   </lst>

   <lst name="spellchecker">
      <str name="classname">solr.FileBasedSpellChecker</str>
      <str name="name">file_dic</str>
      <str name="sourceLocation">spellings_dic.txt</str>
      <str name="characterEncoding">UTF-8</str>
      <str name="spellcheckIndexDir">./filespellchecker</str>
      <str name="buildOnOptimize">true</str>
   </lst>

  <str name="queryAnalyzerFieldType">text_ik</str>
  </searchComponent>

  <requestHandler name="/suggest" class="solr.SearchHandler" startup="lazy">
    <lst name="defaults">
     <str name="df">search_text</str>
    
      <str name="spellcheck.dictionary">index_dic</str>
      <str name="spellcheck">on</str>
      <str name="spellcheck.extendedResults">false</str>       
      <str name="spellcheck.count">10</str>
      <str name="spellcheck.alternativeTermCount">11</str>
      <str name="spellcheck.maxResultsForSuggest">1000000</str>       
      <str name="spellcheck.collate">true</str>
      <str name="spellcheck.collateExtendedResults">false</str>  
      <str name="spellcheck.maxCollationTries">5</str>
      <str name="spellcheck.maxCollations">5</str>         
    </lst>
    <arr name="last-components">
      <str>spellcheck</str>
    </arr>
  </requestHandler>

下面的代码就会查询出“江西“的搜索结果、以及江西的查询建议
String channelid = "0001";
        SolrSearcher searcher = new SolrSearcher(BaseSolrCreator.core_photoset_ + channelid);
                String input = "江西";       
        String q = "search_text:江西";
        String dict = "index_dic";
        String accuracy = "0.1";
        String suggestionCount = "12";
        String maxCollations = "5";
        String maxResultForSuggest = "10000000";

        searcher.setCommonCondition(q, null, "channelid, setname", null, 0, 5);
        //searcher.addTermRegexCondition("setname_suggest", ".*" + input + ".*",
        //      TermsParams.TermsRegexpFlag.CASE_INSENSITIVE.name(), null, null, 5);
        searcher.addSpellCheckCondition(input, dict, accuracy, suggestionCount, maxCollations, maxResultForSuggest,
                false, false, false);
        searcher.search();
        //System.out.println(searcher.getTermResults());
        // System.out.println(searcher.getResults());
        System.out.println(searcher.getSpellCheckResults(true));

五、realtimeget实时索引
lucene是不直接支持实时索引的，因为只有被commit的索引才能被搜索出来，但commit磁盘的话，带来的消耗很大，所以不可用实时commit。但可以间接实现，比如一个方案是：用multisercher，先搜索原来被commit的索引，然后再把更新的数据放到RAMdirectroy中，作为另一个searcher，这样既可实现。
solr4.0后则提供了realtimeget功能，具体请参见http://wiki.apache.org/solr/RealTimeGet
需要在solrconfig.xml中启用，updateLog和/get两个模块既可。
    <updateLog>
      <str name="dir">${solr.data.dir:}</str>
    </updateLog>

  <requestHandler name="/get" class="solr.RealTimeGetHandler">
     <lst name="defaults">
       <str name="omitHeader">true</str>
       <str name="wt">json</str>
       <str name="indent">true</str>
     </lst>
  </requestHandler>

另外：实时索引必须有主键，而且这个只能基于主键查询，也就是不支持其它字段的搜索了。。。。。
比如用延迟commit测试既可