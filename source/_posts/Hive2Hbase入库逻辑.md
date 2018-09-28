title: Hive2Hbase入库逻辑
author: Peng Fang
tags:
  - 数据入库
  - Hbase
categories:
  - 大数据
date: 2018-09-19 20:21:00
---
### Parquet格式hdfs文件根据规则生成HFile
1. 创建和设置habse参数
2. 清楚输出目录下HFile文件
3. 创建mapreduce job
4. 设置mapper：job.setJarByClass, job.setMapperClass, job.setMapOutputKeyClass, job.setMapOutputValueClass；
5. 设置Parquet格式的解析器；
6. 设置合并方法：job.setCombinerClass(Combiner.class)，Combiner继承自Reducer，reduce方法用于数据去重；
7. 设置写HFile的reduce：HFileOutputFormat2.configureIncrementalLoad, HFileOutputFormat2.setOutputPath; job.setReducerClass(KeyValueSortReducer.class)：设置reducer class, 如KeyValue、Put、Text，以正确地对输入值进行排序。configureIncrementalLoad中会根据前面设置的MapOutputKeyClass来设置ReducerClass，见下面的源代码。
8. 提交job，轮询等到其完成，退出程序执行。   
第7步configureIncrementalLoad源代码：   
``` java
static void configureIncrementalLoad(Job job, HTableDescriptor tableDescriptor,
                                     RegionLocator regionLocator, Class<? extends OutputFormat<?, ?>> cls) throws IOException,
        UnsupportedEncodingException {
    Configuration conf = job.getConfiguration();
    job.setOutputKeyClass(ImmutableBytesWritable.class);
    job.setOutputValueClass(KeyValue.class);
    job.setOutputFormatClass(cls);

    // Based on the configured map output class, set the correct reducer to properly
    // sort the incoming values.
    // TODO it would be nice to pick one or the other of these formats.
    if (KeyValue.class.equals(job.getMapOutputValueClass())) {
        job.setReducerClass(KeyValueSortReducer.class);
    } else if (Put.class.equals(job.getMapOutputValueClass())) {
        job.setReducerClass(PutSortReducer.class);
    } else if (Text.class.equals(job.getMapOutputValueClass())) {
        job.setReducerClass(TextSortReducer.class);
    } else {
        LOG.warn("Unknown map output value type:" + job.getMapOutputValueClass());
    }

    conf.setStrings("io.serializations", conf.get("io.serializations"),
            MutationSerialization.class.getName(), ResultSerialization.class.getName(),
            KeyValueSerialization.class.getName());

    // Use table's region boundaries for TOP split points.
    LOG.info("Looking up current regions for table " + tableDescriptor.getTableName());
    List<ImmutableBytesWritable> startKeys = getRegionStartKeys(regionLocator);
    LOG.info("Configuring " + startKeys.size() + " reduce partitions " +
            "to match current region count");
    job.setNumReduceTasks(startKeys.size());

    configurePartitioner(job, startKeys);
    // Set compression algorithms based on column families
    configureCompression(conf, tableDescriptor);
    configureBloomType(tableDescriptor, conf);
    configureBlockSize(tableDescriptor, conf);
    configureDataBlockEncoding(tableDescriptor, conf);

    TableMapReduceUtil.addDependencyJars(job);
    TableMapReduceUtil.initCredentials(job);
    LOG.info("Incremental table " + regionLocator.getName() + " output configured.");
}
```

### 加载HFile到HBase表
1. 加载配置文件hbase-site.xml
2. 读取hdfs配置文件hbase-hdfs-site.xml，然后配置hbase-site.xml的一些参数：   
  ● 配置dfs.ha.namenodes.[nameservice ID]，让DataNode知道每个集群的所有NameNode   
  ● 配置dfs.client.failover.proxy.provider.[nameservice    ID]，该参数定义HDFS客户端用来和活动的namenode节目联系的java类。配置的java类是用来给HDFS客户端判断哪个namenode节点是活动的，当前是哪个namenode处理客户端的请求。
  ● 配置dfs.namenode.rpc-address.[nameservice ID].[name node ID] ， 即配置每个namenode监听的RPC地址。   
  ● 配置dfs.nameservices--自定义的HDFS服务名，在NameNode节点的集群中，对HDFS集群访问的入口是NameNode所在的服务器。但是在两个NameNode节点的HA集群中，无法配置单一服务器入口。所以需要指定一个逻辑上的服务名，这个服务名是自定义的。当外界访问HDFS集群时，入口就变为这个服务。用户不必关心当前具体是哪台服务器在提供服务（Active状态），只要访问这个服务就可以了。   
  ● 配置mapreduce.job.hdfs-servers.token-renewal.exclude为名字服务id的数组，ResourceManager为应用程序更新委托令牌。只有当令牌的更新者是非空字符串时，此行为才被更新为令牌。MapReduce作业可以指示ResourceManager通过指定具有配置mapreduce.job.hdfs-servers.token-renewal.exclude = <host1>，<host2>，..，<hostN>的主机来跳过从某些主机获取的令牌的更新。   
3. 向HBASE表加载数据
``` java
HTable table = new HTable(conf, tableB);
LoadIncrementalHFile loader = new LoadIncrementalHFile(conf);
loader.doBulkLoad(new Path(outputPathOnHbase), table);  // outputPathOnHbase是HBASE的hdfs上HFile的路径
```
API解释如下：
``` java
Class LoadIncrementalHFiles：Tool to load the output of HFileOutputFormat into an existing table.
public void doBulkLoad(org.apache.hadoop.fs.Path hfofDir,
              Admin admin,
              Table table,
              RegionLocator regionLocator)
                throws TableNotFoundException,
                       IOException
Perform a bulk load of the given directory into the given pre-existing table. This method is not threadsafe.
Parameters:
hfofDir - the directory that was provided as the output path of a job using HFileOutputFormat
table - the table to load into
Throws:
TableNotFoundException - if table does not yet exist
IOException
```

#### 参考：
https://blog.csdn.net/book_mmicky/article/details/25714339
https://segmentfault.com/a/1190000007239743
http://www.searchdoc.cn/hadoop/hadoop.apache.org/docs/r3.0.0-alpha1/hadoop-project-dist/hadoop-common/release/3.0.0-alpha1/RELEASENOTES.3.0.0-alpha1.com.coder114.cn.html
https://www.programcreek.com/java-api-examples/index.php?api=org.apache.hadoop.hbase.mapreduce.KeyValueSortReducer

