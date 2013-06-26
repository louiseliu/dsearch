这里特别的罗嗦几句，在Solr4.0发布以后，官方取消了BaseTokenizerFactory接口，而直接使用Lucene Analyzer标准接口。因此IK分词器2012 FF版本也取消了org.wltea.analyzer.solr.IKTokenizerFactory类。那么原来的配置
    
    <fieldType name="iktext" class="solr.TextField" >
    
    <analyzer type="index">
    
    <charFilter class="solr.HTMLStripCharFilterFactory"/>
    
    <tokenizer class="org.wltea.analyzer.solr.IKTokenizerFactory" useSmart ="false"/>
    
    <filter class="solr.LowerCaseFilterFactory"/>
    
    <filter class="solr.NGramFilterFactory" minGramSize="1" maxGramSize="20" side="front"/>
    
    </analyzer>
    
    <analyzer type="query">
    
    <tokenizer class="org.wltea.analyzer.solr.IKTokenizerFactory" useSmart ="false"/>
    
    <filter class="solr.LowerCaseFilterFactory"/>
    
    </analyzer>
    
    </fieldType>


就无法使用，

解决方案 https://code.google.com/p/ik-analyzer/issues/detail?id=104
