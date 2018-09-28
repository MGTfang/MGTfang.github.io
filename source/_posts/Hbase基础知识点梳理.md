title: Hbase基础知识点梳理
author: Peng Fang
tags:
  - Hbase
categories:
  - 大数据
date: 2018-09-10 20:09:00
---
### 整体架构
从HBase部署架构来说，HBase有两种服务器：Master服务器和RegionServer服务器。   
一般一个HBase集群有一个Master服务器和几个RegionServer服务器。   
Master服务器负责维护表结构信息，Master挂到后还能查询数据，只是不能新建表或修改表。   
实际的数据存储在RegionServer服务器上，RegionServer保存的表数据是直接存储在Hadoop的HDFS上。客户端获取数据是由客户端直接连接RegionServer服务器。   
RegionServer非常依赖ZooKeeper服务。ZooKeeper在HBase中扮演管家角色，管理了HBase所有RegionServer的信息，包括具体的数据段存放在哪个RegionServer上。   
客户端每次与HBase连接，先与ZooKeeper通信，查询出需要与哪个RegionServer连接，然后再连接RegionServer。   
整体架构图如下：   
![整体架构图](http://207.148.69.56/pub/imgs/arch/full_arch.png)

#### Region是什么
Region就是一段数据的集合。HBase中的表一般拥有一个到多个Region。Region有几个特性：
- Region不能跨服务器
- 当数据量大的时候，HBase会拆分region
- HBase负载均衡的时候，可能会把Region从一个RegionServer移动到另一个上
- Region是基于HDFS的，所有数据存取操作都是调用了HDFS的客户端接口来实现的

#### RegionServer是什么
RegionServer是存放Region的容器，直观来说就是服务器上的一个服务。一般来说，一个服务器只会安装一个RegionServer服务。
当客户端从ZooKeeper获取RegionServer的地址后，它会直接从RegionServer获取数据，插入、删除等数据操作也是经过它。

#### Master是什么
HBase中Master角色不像领导，更像是打杂的。Master负责各种协调工作，比如建表、删表、移动Region、合并等操作。它们的共性就是需要跨RegionServer，由哪个RegionServer
来执行都不合适，所以HBase将这些操作放到Master上。这种结构大大降低了集群对Master的依赖，即使Master宕机了，也不影响数据操作。

### 存储架构
HBase最基本的存储单位是列。行跟行的列可以完全不一样，行与行的数据，甚至同一行的列可以存储在不同的机器上。若干个列被归类为一个列族。   
存储架构图如下：
![存储架构图](http://207.148.69.56/pub/imgs/arch/storage_arch.png)

#### 行键是什么
rowkey是由用户指定的一串不重复字符串。rowkey是决定row存储顺序的唯一凭证。HBase是根据字典排序的。比如：row-1, row-11, row-2。

#### 列族是什么
建表的时候不需要指定列，但需要确定列族，这就是说一个表有几个列族是一开始确定好的。此外，表的很多属性，比如过期时间、数据块缓存以及是否压缩等都是定义在列族上，而不是定义在表上或者列上。   
同一个表里的不同列族可以有完全不同的属性配置，但是同一个列族内的所有列都会有相同属性。
列名称规范是列族：列名，如brother:age。   
列族的意义是：HBase会把相同的列族尽量放在同一台机器上，所以说，如果想让某一个列被放在一起，就给他们定义相同的列族。   
一个表设置多少个列族合适？官方的建议是：越少越好，因为Hbase虽然是分布式数据库，但是数据在同一台机器上仍然会加速数据的查询过程。所以请根据实际需要制定列族，列族太多会极大降低数据库性能。

#### 单元格是什么
一个列上可以存储多个版本的值，多个版本的值被存储在多个单元格里面，多个版本之间用版本号（Version）来区分。所以唯一确定一条结果的表达式应该是行键：列族：列：版本号(rowkey:column family:column:version)。不过版本号是可以省略的，如果不写版本号，HBase默认获取最后一个版本的数据并返回。每个列或者单元格的值都被赋予一个时间戳，这个时间戳默认是由系统指定的，也可以由用户显示指定。

#### Region跟行的关系
一个Region就是多个行的集合。在Region中行的排序按照行键（rowkey）字典排序。   
表结构图如下：
![表结构示意图](http://207.148.69.56/pub/imgs/arch/table_struct.png)

传统数据库insert语句可以把整行的数据一次性写在行语句里面，而在HBase里面如果一行有10列，那么存储一行的数据得写10行的语句。存储语句必须精确写出数据被存储到哪个单元格，根据表：列族：行：列来定义。行的概念被弱化到只有一个抽象的存在，rowkey是其唯一的体现。

### 讲讲ZooKeeper
ZooKeeper不知Hadoop的HA模式（HA的作用是保证一个namenode挂掉的时候，另外一个namenode能立即启动将其代替，这样就不会发生单点故障的问题）用到，HBase也会用到。ZooKeeper是Apache旗下的一个开源项目，它是一个开源的分布式应用程序协调服务，它可以为分布式应用提供一致性服务，提供的功能包括：配置服务、域名服务、分布式同步、组服务等。ZooKeeper负责维护HBase的所有节点，如果ZooKeeper宕掉了，集群的一个节点都连不上。   
ZooKeeper最大的功能之一就是知道某个节点是否宕机了，那么ZooKeeper是如何知道某个节点宕机的呢？   
答案是，每一个机器在ZooKeeper中都有一个会话（Session），如果某个机器宕机了，这个会话（Session）就会过期，与此同时，ZooKeeper就知道该节点已宕机。   
多少个节点最好？节点越多容灾能力就越强，不过节点数最好是奇数个，这样有利于仲裁。   
ZooKeeper的容灾机制：集群中只要有过半的机器是正常工作的，那么整个集群对外就是可用的。如果有5个节点，有2个集群宕掉，集群依然可以运行。这是我们称该集群容灾能力是2。目前节点数是6，只要有3个机器宕掉那么集群就宕掉了，此时的容灾能力仍是2。   
ZooKeeper应该存储在一个独立的磁盘上，避免当磁盘出现IO问题的时候ZooKeeper被波及。

### HBase常用命令
- list   
查看数据库中有哪些表

- describe   
查看表属性

- create & alter   
create 'test', 'cf' # 建立一个叫test的表，这个表有一个列族叫cf
alter 'test', 'cf2' # 新增一个列族
在生产环境更改表之前，最好先停用（disable）这个表。因为对列族的所有操作都会同步到所有拥有这个表的RegionServer上。当有很多客户端都在连着的时候，直接新增一个列族对性能的影响较大。

- put  
put 'test', 'row1', 'cf:name', 'jack'
意思是往test表插入一个单元格，这个单元格的rowkey为row1，也就是说它是属于row1这个行中的一个列。该单元格的列族是cf，单元格的列名是name，数据值是jack。   
    - 关于时间戳
    每一个单元格可以存储多个版本的值。它用timestamp来存储该条记录的时间戳，这个时间戳就用来当版本号使用。timestamp虽然是时间的标定，其实可以输入任意的数字，比如1，2，3等。当用scan命令的时候HBase会显示拥有最大（最新）的timestamp的数据版本。   
    创建表时默认版本是1，也就是说在一个单元格插入多个版本的数据的时候，HBase只会保留最后一个版本。能够在scan操作的结果集看到多个版本的前提是建表时设定的VERSIONS参数要大于1，这样表才能保存设定VERSIONS数个历史记录。修改版本数的命令：   
    alter 'test', {NAME=>'cf',VERSION=>5} # NAME为要修改的列族的名称，后面跟要修改的属性，我们修改VERSION为5   
    put 'test','row2','cf:name','ted'   
    put 'test','row2','cf:name','billy',2222222222222 #自定义时间戳晚于当前时间   
    scan 'test' # 只能看到时间戳最大的那条结果   
    get 'test','row2',{COLUMN=>'cf:name', VERSIONS=>3} #查询所有版本的数据   

- scan   
scan '表名'，这样输入的话，会从第一条数据开始把所有数据全部显示一遍。   
在HBase中我们用起始行（STARTROW）和结束行（ENDROW）来限制记录的条数。   
scan 'test',{STARTROW=>'row3'} # 显示所有rowkey大于等于row3的记录   
scan 'test',{ENDROW=>'row4'} # 小于row4(不包括)的记录   
两个参数一起用就是显示 >= STARTROW 并且 <ENDROW中的那段数据   
scan '表名',{COLUMNS=>['列1','列2',...]} # 只遍历指定列   
scan '表名',{STARTROW=>'起始行键',ENDROW=>'结束行键'} # 指定行键范围   
scan '表名',{LIMIT=>行数量} # 指定最大返回行数量   
scan '表名',{TIMERANGE=>[最小时间戳,最大时间戳]} # 找出单元格的历史版本数据   
scan ’表名',{VERSION=>'版本数'} # 指定版本数，显示单元格多个版本值  
scan '表名',{FILTER=>'过滤器'} # scan 'table1' { FILTER=> "PrefixFilter('row1')" }   
- get   
scan可以查询表的多条数据，get只能查询一个单元格的记录。在数据量大的时候，get的查询熟读远远高于scan。   
get 'test','row7','cf:name' # 查询某个单元格的记录   
get 'test','row7',{COLUMN=>'cf:name',VERSION=>5} # 把查询版本数设定为5   
scan 'test',{VERSION=>5} # scan跟上VERSIONS参数也可以查询出多个版本的数据   
- count   
计算表的行数  
- delete   
delete '表名','行键','列名'   
delete '表名','行键','列名',时间戳   
- split   
拆分(split)指定的Region，除了可以等到Region大小达到阈值后触发自动拆分机制来拆分Region，还可以手动拆分指定的Region，如 split 'region名'。   
- merge_region   
合并（merge）两个Region和一个Region   
- compact   
调用指定表的所有Region或者指定列族的所有Region的合并（compact）机制。通过compact机制可以合并该Region或者Region的列族下的所有HFile（StoreFile），以此来提高读取性能。   
compact跟合并（merge）并不一样。merge操作是合并2个Region为1个，而compact操作更新的单元StoreFile，一个Region可以含有一个或多个StoreFile，compact操作的目的在于减少StoreFile的数量以增加读取性能。
- 快照   
快照就是表在某个时刻的结构和数据。可以使用快照来将表恢复到某个时刻的结构和数据，恢复过程很快，往往只有数秒。HBase是怎么做到的呢？其实快照并不直接复制数据，而是保持一份文件列表，通过修改表所链接的文件夹来改变表的数据，好处是：速度极快；不额外占用磁盘空间。   
HBase要求在恢复快照之前必须要先停用（disable）需要恢复的表，才能恢复快照，避免不必要的麻烦。先停用，然后恢复快照，再启用（enable）。   
通过snapshot命令可以创建指定表的快照。   
 disable 'mgdmcredit:userblack_temp'   
 snapshot 'mgdmcredit:userblack_temp', 'userblack_temp_snapshot'   
 disable 'mgdmcredit:userblack'  
 drop 'mgdmcredit:userblack'   
 clone_snapshot 'userblack_temp_snapshot' 'mgdmcredit:userblack'   
 enable 'mgdmcredit:userblack_temp'   
 truncate 'mgdmcredit:userblack_temp'   
 delete_snapshot 'userblack_temp_snapshot'   
 clone_snapshot使用快照的数据创建一张新表，创建的过程很快，因为使用的方式不是复制数据，并且修改新表的数据不会影响旧表的数据。   

 ### 客户端API
 在HBase中有一个理念：所有数据皆为bytes。在HBase中数据最终都会被序列化为bytes[]保存。
 - put方法
 ``` java
try(Connection connection = ConnectionFactory.createConnection(config)) {
    Table table = connection.getTable(TableName.valueOf("myTable"));
    Put put = new Put(Bytes.toBytes("row1"));
    put.addColumn(Bytes.toBytes("mycf"), Bytes.toBytes("name"), Bytes.toBytes("ted"));
    table.put(put);
}
 ```
 新增和修改类型，修改就是往同一个rowkey再执行一次put操作，将之前的数据覆盖掉。
 JDK的try-with-resources，上面例子中看到try(...){....}，{....}中的代码执行完毕会自动释放资源，不需要手动写finally语句块了。

- get方法
get不像scan，并不能用多种条件去查找，只能用行键去查找。不过HBase一行有可能很大，我们可以通过设置参数让get只获取其中一部分的数据，以提高查询性能。   
``` java
Get get = new Get(Bytes.toBytes("row1"));
get.setMaxVersions(10);
Result result = table.get(get);
List<Cell> cells = result.getColumnCells(Bytes.toBytes("mycf"), Bytes.toBytes("name"));
for(Cell c: cells) {
    // 用CellUtil.cloneValue来获取数据而不是getValue
    bytes[] cValue = CellUtil.cloneValue(c);
    System.out.println(Bytes.toString(cValue));
}
```
用CellUtil.cloneValue来获取数据而不用getValue的原因：根据目前getValue的实现代码，每次调用它都会获取整个Cell的数组备份，比较消耗性能。所以当需要获取Cell中的值时，请使用CellUtil.cloneValue方法。   

- Scan扫描   
查询多条数据我们会想到用scan。默认的scan是从表头一直遍历到表尾，非常耗时好性能，实际工作中，我们至少要定义一下遍历的起始rowkey，如果知道遍历的结束rowkey就好了。   
举个栗子：
``` java
Scan scan = new Scan(Bytes.toBytes("row1"));
ResultScanner rs = table.getScanner(scan);
...
rs.close(); // 使用完了关闭
```
这里为什么是getScanner()而不是scan()？因为Table通过传入scan之后返回的结果扫描器（ResultScanner）并不是实际的查询结果。获取结果扫描器（ResultScanner）的时候并没有实际去查询数据。真正要获取数据的时候要打开扫描器，然后遍历它，这个时候才真正地查询了数据。
这个ResultScanner就像关系型数据库中的ResultSet一样是需要持续占用资源的，所以用完后务必记得关闭它。

- 缓存   
HBase在扫描的时候已经默认开启了缓存。具体来讲，每一次的next()操作都会产生一次完整的RPC请求，而这次RPC请求可以获取多少数据是通过hbase-site.xml中的hbase.client.scanner.caching参数配置的。比如配置为1，遍历了10个结果就会发送10次请求。
可以在表的层面修改缓存条数，也可以在扫描层面去修改。表的层面修改时通过hbase-site.xml中的配置:   
``` xml
<property>
<name>hbase.client.scanner.caching</name>
<value>1000</value>
</property>
```
hbase.client.scanner.caching的默认配置是100.   
在扫描层面修改缓存可以使用Scan.setCaching(int caching)方法设置一次next获取的数据条数，这个配置的优先级比配置文件内的hbase.client.scanner.caching高。   

### HBase二层查询架构
从0.96版本之后三层查询架构被改成了二层查询架构。直接把.META.表所在的RegionServer信息存储到zk中的/hbase/meta-region-server去了。   
![二级查询架构图](http://207.148.69.56/pub/imgs/arch/query_arch.png)
上图可以总结成以下流程：   
1. 客户端先通过ZooKeeper的/hbase/meta-region-server节点查询到哪台RegionServer上有hbase:meta表。  
2. 客户端连接含有hbase:meta表的RegionServer。hbase:meta表存储了所有Region的行键范围信息，通过这个表可以查询出你要存放的rowkey属于哪个Region的范围里面，以及这个Region又属于哪个RegionServer。
3. 获取这些信息后，客户端就可以直连其中一台用于你要存取的rowkey的RegionServer，并直接对其操作。
4. 客户端会把meta信息缓存起来，下次操作就不需要进行以上加载habse:meta的步骤了。

### 过滤器
过滤器就是在Get或Scan的时候过滤结果用的，可以把他看成SQL中的where语句。HBase中过滤器被用户创建出来以后会被序列化我可以网络传输的格式，然后被分发到各个RegionServer。在RegionServer中Filter被还原出来。这样在Scan的遍历过程中，不满足过滤条件的结果将不会被返回客户端。

#### 前缀过滤器
这种过滤器可以根据行键的前缀匹配同样是这个前缀的行。运用它能提高查询性能，因为前缀过滤器在遇到扫描的行键的前缀大于所指定的前缀时，立即停止扫描。
就算用了前缀过滤器也依然要结合上STARTROW使用，否则scan还是会从第一条记录开始扫描，浪费了大量的性能。
当我们用Scan扫描数据的时候，如果使用STOPROW来指定终止行，结果集中并不会包含终止行。如果想在结果中包含终止行可以有两种方式：
（1）在终止行的rowkey上增加一个字节的数据，然后把增加一个字节的rowkey作为STOPROW
（2）使用包含结尾过滤器（InclusiveStopFilter）

### Region的拆分
通过查询hbase:meta我们可以形象地看到，一个Region就是一个表的一段rowkey的数据集合。
当Region太大的时候HBase会拆分它。   
为什么要拆分Region？因为当某个Region太大的时候读取效率太低了。   
我们可以想想为什么从MySQL、Oracle转移到NoSQL来？最根本的原因就是这些关系型数据库把数据放到一个地方，而当数据量增大到上亿的时候同一个磁盘已经无法应付这些数据的读取了，因为遍历一遍数据的时间实在太长了。用NoSQL的理由就是其能把大数据分拆到不同的机器上，然后像查询一个完整的数据一样查询他们。当Region太大的时候，一样会遇到跟传统数据库一样的问题，所以要拆分Region。   

#### IncreasingToUpperBounRegionSplitPolicy策略（默认）
计算公式：  
Math.min(tableRegionsCount^3 * initialSize, defaultRegionMaxFileSzie)   
- tableRegionsCount：表在所有RegionServer上拥有的Region数量总和
- initialSize：如果定义了hbase.increasing.policy.initial.size，则使用这个数值，否则，就使用memstore的刷写大小的2倍，即hbase.hregion.memstore.flush.size *2
- defaultRegionMaxFileSzie：ConstantSizeRegionSplitPolicy所用到的hbase.hregion.max(一般是10G)

一开始可以先定义拆分点，但是当书记开始工作起来后会出现热点不均的情况，所以推荐的方法是：
1. 用预拆分导入初始数据
2. 然后用自动拆分来让HBase自动管理Region
Region的拆分对性能的影响还是很大的，默认的策略已经适用于大多数情况。可以尝试哪种策略最好。

### Region的合并
Region的合并（merge）并不是为了性能考虑，更多地出于维护的目的。   
啥时候才会用到合并呢？比如删了大量的数据，每个Region都变小了，这个时候分成这么多个Region就有点浪费，可以把Region合并起来。   
通过Merge类来合并叫冷合并，就是要把HMaster和所有的HRegionServer全部停掉，再执行才可以；   
通过online_merge命令叫做热合并   

### HFile的合并
除了Region会合并和拆分，在Region中的单个Store中也会发生合并（compaction）。   
HFile为什么要合并？但凡存储在磁盘上的东西都涉及到一个操作：寻址。传统硬盘是磁头的移动寻址，是一个很慢的动作。当HFile一多，每次读取数据的时候寻址动作就多了，效率就降低了。所以我们要适当减少碎片文件，进而需要合并操作。HFile的合并操作就是在一个store里面找到需要合并的HFile，把他们合并起来，再把之前的碎文件移除。   

#### 0.96版本之后的合并算法

##### 待合并文件挑选条件
该文件 < （所有文件大小总和 - 该文件大小）* 比例因子   
如果该文件大小小于最小合并大小(minCompactSize)，直接进入待合并列表。最小合并大小配置项：hbase.hstore.compaction.min.size，如果没有设定该项，则使用hbase.hregion.memstore.flush.size.

##### 以组合作为计算单元
新的算法不再按文件为单元进行比较了，而是挑出多个文件组合。   
挑选组合的条件：
被挑选的文件必须能通过以上提到的筛选条件，并且组合内含有的文件数必须大于hbase.hstore.compaction.min，小于hbase.hstore.compaction.max.   
挑选完组合后，比较哪个文件组合包含的文件更多，则合并哪个组合。如果出现平局，就挑选哪个文件尺寸总和更少的组合。

调整HFile合并策略，让HFile的数量尽量减小，以减少每次Scan的跨HFile的次数，但同时又要保证该合并策略适用于场景，并且不用太频繁。   

### 读取性能优化
性能有两方面的提高空间：
- 调整对于API的用法   
- 调整系统配置   

#### 参考
《Mysql不睡觉书》
