Apache Lucene是一个基于Java、高性能的全文检索引擎，4.3版本的主要改进包括： 

+ 显著改善了最小匹配布尔查询的性能，查询速度提升了40倍
+ 新增了SortingAtomicReader（允许根据条件排序索引）和SortingMergePolicy（允许在段被合并之前排序文档）
+ DocIdSetIterator和Scorer现在有一个cost API，能够提供迭代器可能匹配的文档的最大数量，可在查询执行期间进行优化
+ Analyzing/FuzzySuggester现在允许记录任意byte[]作为有效负载。
+ 除了典型的Intersects外，Lucene Spatial模块现在还可以通过Within、Contains和Disjoint关系搜索索引。
+ PostingsHighlighter现在允许自定义passage scores、per-field BreakIterators
+ 在facet模块中新增了facet方法，用于计算facet数。
+ Apache Solr是基于Lucene的企业级搜索平台，Solr 4.3版本除了包含Lucene 4.3的改进和bug修复外，还针对企业级应用进行了优化，详细信息：Apache Solr 4.3 Changelog 

Lucene 4.2版本的主要亮点包括： 

+ 包含了一个新的默认编解码器（Lucene42Codec），带来了更高效的docvalues格式和更小的term vectors
+ 简化了doc值外部、codec API及其实现。
+ 重构了facet模块，显著增强了性能，某些case下性能可提升3.8倍。
+ facet模块中的DrillDownQuery现在支持多选
+ 一个新的DrillSideways类，可以计算facet标签数，以及单次查询中的命中数和几乎发生的失误数。
+ 一个新的docvalues类型SORTED_SET，支持多个值。
+ 一个新的classification模块



Lucene 4.1的主要新特性包括： 

+ 新的默认编解码器（Lucene41Codec），基于先前实验的“块”索引格式，在提高性能的同时，也合并了“Appending”、“Pulsing”功能。

+ 新的搜索建议实现AnalyzingSuggester和FuzzySuggester（允许对输入内容模糊匹配）

+ facet模块现在支持实时功能

+ highlighter模块中添加了新的高亮功能postingshighlighter

+ 在FilteredQuery中增加了FilterStrategy，以使过滤查询执行更加灵活。

+ 增加了CommonTermsQuery以加快高频术语的查询速度。

+ 优化、修复了4.0版本中的一些功能和bug。

Solr 4.1除了包含以上这些特性外，还增强了SolrCloud、添加了一些针对企业级搜索的新功能、改善了Admin UI、修复了一些兼容问题和bug。 


Apache Lucene 4.0的主要特新包括： 

+ 针对词（term）、文章列表、存储字段、词语向量（term vector）的索引格式可通过Codec API来实现定制。你可以从提供的实现中选择，也可以自定义索引格式。
+ 新的doc值，用于存储每个文档的类型值。
+ 现在当应用程序使用多线程进行索引时，IndexWriter同时flushes segments到磁盘，从而显著改善了性能。
添加了新的索引统计。
+ 新的默认词典/index（BlockTree）索引共享前缀。
+ 索引词语不再局限于UTF-16字符，可以是编码为字节数组的任意二进制值，默认情况下，被编码为UTF-8。
+ 显著改善了搜索中使用过滤器的性能。
+ 基于文件系统的目录能够限制合并线程的IO速率，以减少合并和搜索中的IO争用。
+ 添加了一些备用的编解码器和组件。
+ FuzzyQuery速度比之前版本快了100-200倍。
+ 添加了一个新的拼写检查器DirectSpellChecker。
+ 提供了一个模块化的API，重组了之前分散在Lucene核心、发布版本和Solr中的组件，如Analyzers、Queries等。



Lucene 3.6.2包含了一些优化改进和bug修复，主要包括： 

+ 修复了当内存中项目索引需要超过2.1GB RAM时的ArrayIndexOutOfBoundsException异常问题。
+ 修复了查询解析器中的一个布尔查询解析bug
+ 修复了BooleanScorer2中的bug，现在使用scorer visitor API时可返回正确的freq()
+ 修复了IndexWriter RAM计算bug
+ 修复了其他一些小的bug
 
Solr是基于Lucene的企业级搜索平台，因此Solr 3.6.2中包含了Lucene 3.6.2中的改进，除此之外，还包括： 

+ 修复了请求所有字段时ConcurrentModificationException异常的bug
+ 修复了edismax查询解析器的bug，为隐式布尔查询应用最小匹配
+ 修复了数据导入处理程序中的一些bug

