# Chapter I: Reliable, Scalable and Maintainable Applications
1.  现在的时代从 compute intensive过度到了 data intensive. 在data intensive的情况下有几种模块：
    1.  Database: store the data and retrive the data
    2.  Caches: remember the result of an expensive operation to speed up the read.
    3.  Search index: allow users ot search data by keyword or filter it.
    4.  Stream processing: sent message to another process to be handled asynchronously.
    5.  Batch processing: Periodically crunch(紧缩) a large amount of accumulated data.

# Chapter II: Data Models and Query Languages:
1.  应用的结构：
    1.  定义data model: 需要存什么内容，例如人体信息的身高体重，可能有些可以丢弃。相当于定义database 的 schema.
    2.  保存data 在 database中：怎么存的问题，relational database的table; document database 的 JSON/XML
    3.  保存data 在电脑硬件（磁盘、内存等）中：怎么存的问题，在网络上用large distributed system的问题
2.  Database的几种形式：
    1.  Relational Database: 
    data is organized into relations (tables in SQL) where each relation is an 
    unordered collection of tuples(rows in SQL)
        1.  Disadvantage:
            1.  Object relation Mismatch: 在程序处理数据的时候往往是以object的形式处理的，但是存的时候就要考虑怎么修改
            成为relational model了；--> impedance mismatch
            2.  不适合于 one to many relationship: relational database 需要多层SQL table跳跃:
            对于 one to many的处理就是需要使用另一个table来记录；例如在linkedin
            上的一个人是有多个工作经历的；这样的map就是一对多；那么在relational database的结构中就产生了问题：我的主表是user表
            user -> name, summary, location等的map，但是 job/education则是要去从整个的 job的表中去找，找user ID == curr
            requested user ID 的情况，然后再 join 到给user的feedback中，这样的过程非常的久。
            3.  Add one more column is a big disadvantage: 在database中增加一栏需要改动schema，并且对之前所有的内容都要修改。
            在高峰期时非常的危险的。
            4.  由于relational database本身的size相比document的大，故而同样大小的内容需要分裂到不同的computer上，然后造成了读
            的速度不如一台电脑的document database; 同时，由于one to many的事情导致有多张表，这样也造成了我要读一个内容需要从多个
            database上读取。这样也是更慢
        2.  Advantage:
        3.  Schema on write:
            在写的时候就必须要有schema，并且按照schema去写。
            
    2.  Document Database
        1.  Advantage: 
            1.  greater scalability: support very large datasets or very high write throughput.
            2.  Free open source software. 
            3.  Better locality: 对应于relational database的第二个disadvantage. 由于document database使用的是JSON/XML
            file的存储方式，故而其本身就是一个Tree的结构，多份工作或者多份学历只是将Tree Node的 child增大而已。
        2.  Disadvantage:
            1.  Many to one problem: many people live in one city; many people work in one company, etc...
            对于relational database来说，由于本身这些city, company信息就是有自己的单独的表格，因此处理方式就是找到job表中 city =
            requested city or company = requested company的user id 然后再populate；
            但是在document database中，这些city/company只是user的一个entity，故而需要新建一个document, 并且让user的city/
            company中增加一个reference ID 去指向这公司，但是要找出在该公司的所有的员工，那就需要在Jobs的document中加入所有的user
            的信息；而这操作量巨大；
            2.  Many to many: 很多document database本身就不支持join，而many to many肯定需要的是join操作；Relational database
            可以使用join，而document就比较麻烦。
            3.  深层次的Nested 的structure: 因为document一层层去找是非常的耗时的
        3.  Schema on read:
            在读的时候才知道schema，写程序不需要知道schema但是读程序是知道schema的。优势是我们要新增内容的时候，我们不用管，直接push
            然后再在read的时候对这项缺失的内容进行添加。
              
    reference: 
        1.  MongoDB one to one:
        https://docs.mongodb.com/manual/tutorial/model-embedded-one-to-one-relationships-between-documents/
        2.  MongoDB one to many:
        https://docs.mongodb.com/manual/tutorial/model-embedded-one-to-many-relationships-between-documents/ 
        3.  SQL one to one; one to many; many to many:
        https://stackoverflow.com/questions/7296846/how-to-implement-one-to-one-one-to-many-and-many-to-many-relationships-while-de

3.  Query Language:
    1.  Declarative query: SQL:
        1.  Advantage: 
            1.  SQL hide the implementation details of database engine. 
            2.  It lend themselves to parallel execution. 对于多线程，因为可以有人在做database 的software可以直接处理
            相比而言，document database就需要用户自己调整做parallel compute.
    2.  Imperative query: most of programming language
        1.  

4.  各种database的对比:
    1.  关系型：MySQL, Oracle, MS SQL, PostgreSQL
        1.  有schema的要求 or schema on write. 因此更新起来是非常的麻烦的，需要整个system down，然后加入新的column
        2.  但是在ACID的满足上是非常的好的，不需要很多的extra work
    2.  文件型：Document: MongoDB, CouchDB
    3.  Key-value: Redis, MemCache
        1.  
    4.  Column: HBase, Cassandra:
        1.  相比于SQL的行式的database, 行式的database的储存是按照列来进行的。也就是说之前我们将一行的data存在一个地方，现在我们将一列的
        data存在一起。
        2.  HBase相比于Cassandra:
            1.  他们都是基于google big table的列族式的数据模型:
            2.  Cassandra只有一种节点，而HBase有多种不同角色，除了处理读写请求的region server之外，
            其架构在一套完整的HDFS分布式文件系统之上，并需要ZooKeeper来同步集群状态，部署上Cassandra更简单。
            3.  Cassandra通过一致性哈希来决定一行数据存储在哪些节点，靠概率上的平均来实现负载均衡；
            而HBase每段数据(region)只有一个节点负责处理，由master来动态分配一个region是否大到需要拆分成两个，
            同时会将过热的节点上的一些region动态的分配给负载较低的节点，因此实现动态的负载均衡。
            3.  因为每个region同时只能有一个节点处理，一旦这个节点无响应，在系统将这个节点的所有region转移到其他节点之前这些数据便无法读写，
            加上master也只有一个节点，备用master的恢复也需要时间，因此HBase在一定程度上有单点问题；而Cassandra无单点问题。
            4.  Cassandra的读写性能优于HBase:
                1.  在读和写的Tput上，Cassandra在大集成的网络情况下都更优；
                2.  在读的速度(latency)上，HBase更优, 但是在写的时候 Cassandra更优秀
            5.  HBase 是 Master slave的结构，它可以和Hadoop MapReduce完美结合
        2.  优势：当我们只需要对某列进行修改的时候，例如facebook的message，那么我们只需要修改 content部分就好。这样的写是非常的快的。
        3.  行式的database更适合OLAP: 因为数据以列来存储，他们在磁盘的一个连续的地方，故而一次性就读完了整条信息，但是如果要做一个entity/cokumn上
        range范围内的average等，那么就很慢，因为如果我们要读整个data 出来到memory然后还要再去找我们的这个entity的话，这样的操作时间快一点但是memory
        的要求就很大；如果是每次都在disk上连续的读，每次都是一个I/O操作，故而时间上很久
        4.  列式的database更适合OLTP: 因为数据以列存储，一次直接读一块到memory中然后再操作是非常快的


10. Backup:
    key value; column family
    Graph data;
    Cypher, Neo4j, datalog,datomic, SPARQL, OrientDB, 
    
# Chapter III: Storage and retrieval:
1.  Easy and simplest way: 每次都append在当前文件的最后 O(1) write; 然后每次read都需要遍历来找O(n) read（注意是O(n)的I/O process）
2.  Optimization1: 增加hash indexes来加快read的速度；O(1 +) write due to index adding; O(1) for key searching in memory
    1.  Write:
        1.  给data增加一个index来记录其号码；
        2.  在memory中记录 index: byte offset，相当于做 hash map，key是data的index，value是它在硬盘中的位置
        3.  然后再在相应的位置写入 key:value pair.
    2.  Read:
        1.  hash中寻找requested key
        2.  然后通过key找到其硬盘上的位置
    3.  Delete:
        1.  append a special deletion record to data file. In read, the key value set map key to deletion record
        it is deletion record, so not return anything. But those data still in disk
        2.  Those record get deleted during the compacting data process, when merge things and get the last record 
        is deletion record, removed everything before it. 
    3.  Problem: 
        1.  Keys must fit into RAM. 如果data非常多就有问题了
        2.  需要compact data，对于新来的data，必须要compact data, 否则disk就fill非常快了
            1.  Compact的过程就是丢弃掉duplicated key in log, and keeping only the most recent update for each key.
            2.  首先将要储存的文件都分成段(segement), 每个segement都有upper limitation，这样当写满的了时候就可以开始写新的内容；
            3.  当file被不断修改的时候，data file其实是在disk上存了多次；然而人在memory中的key value pair中只有最后的一个；
            于是需要清除掉duplicate的内容。在单机上可能少，但是在network上非常的多。
        3.  Range request 非常麻烦，需要扫描整个 hashmap.
        4.  Crash recovery 也是一个问题，毕竟 hash index 都存在于 memory中，而断电后，memory被清空。因此在disk存的data也需要
        加入index，这样才能recovery，只是如果data 很多的话，需要遍历一遍较久；
        另外的优化就是将部分的内容存入到disk上，然后在丢失memory后从disk读。
    4.  由于这些因素，hash index适合于key 非常的少，并且 write非常的多的情况；
    
3.  Optimization2: SSTable + LSM Tree: google file system:
    1.  SSTable: sorted string table; LSM: log structured merge tree:
        1.  在optimization1 中主要的问题是Key must fit into RAM, 也就是说当key多的时候就会出现问题；或者换一个比较大的memory
        或者减小disk的size；
        2.  原理：
            4.  write:
                1.  在memory中开一个red black tree, 然后不停的处理进入的data key, 做好mapping 
                2.  当memory中的data (Memtable)的size不断增大后，达到chunk size的时候将内容放入到disk上，成为SSTable.
            5.  read:
                1.  当有个需求request的时候，看它是否在memory中，binary search ( O(logn))
                2.  如果没有的话，去disk上找最后的SSTable, 看是否在其中，如果不在就继续向前找更old的data
            6.  Compact:
                1.  From time to time, run merging and compaction process in background. 类似于merge sort，产生一个新的SSTable
                    1.  Hash index segement merge: 
                        1.  O(2n), O(2n), use hash table to remember which one inside the two segment, if hit duplication,
                        update the table.
                        2.  需要push两个 segment到memory中才能进行这样的操作； 
                    2.  SSTable chunk merge: 
                        1.  merge sort O(2n), O(1) (don't consider the output memory use): two index tracking two segment
                        then move the index to find the data. 
                        2.  可以不用push segment到memory中,只要index 它
            7.  Delete:
                1.  Map the key to an deletion record, 当读的时候，肯定是先读最近的，然后回溯，这样就能先读到 deletion record从而
                停止了读的过程故而读出来的内容仍旧是对的。
            8.  其他：
                1.  在使用SSTable的时候，还需要注意的是对断电等突发事件的防御（reliability），毕竟如果突然断电，Memory里面的data就
                完全丢失了，这样无法还原了；故而在我们对Memory里的树进行操作之前，我们先commit log，记录下我们要做什么，如果断电了，我们
                还是可以从log里面再还原出memory里面的树
        3.  优点包括：
            1.  Merge segment 相比于 hash index更efficient，在hash index过程中需要将多个segment的内容merge是需要将它们都load
            到memory中，而在SSTable中，进行merge的是多个sorted的segment，这样进行merge sort的话只要indexing
            2.  不需要将所有的key 都存在memory中，只需要存一部分也就是那个Chunk size的内存
            3.  虽然相比于 hash index, 写是慢了些，但是相比于B tree, 那是非常的快了；
        4.  缺点包括：
            1.  读起来比较麻烦：因为每次的读都需要从memory中取然后再要从存在磁盘上的SSTable上去取。虽然每次都是O(logn)的process time
            但是不知道要取多少次 --> O(mlogn)，尤其是要找一个不存在的key，这样就使得每次找都非常的麻烦。
            --> 优化是使用 Bloom filter, 这样的话看一个SSTable的时候可以先用Bloom filter来查看在当前的Segment里面是否有我们要找的
            key，如果没有的话就可以直接pass了；Bloom filter可以快速看是否不存在key, 如果说不存在就一定不存在，如果说存在那不一定存在。
        5.  Reference:
            1.  http://www.ituring.com.cn/article/19383
            2.  https://www.cnblogs.com/fxjwind/archive/2012/08/14/2638371.html
               
4.  B-Tree: Most widely used: relational and nonrelational both use this structure:
    B tree其实就是多路平衡查找树
    1.  B tree的特性： Details 见reference1. 
        1.  B tree的生长是自下往上生长的，当下面的满了的时候将node push到上面的level，然后再看上层是否满了，
        如此往复循环直到整个B tree都满了为止。
        2.  定义： branching factor = m，那么有 m/2 向上取整个reference, m/2向下取整个 key/boundary. 也就是说对一个node来说
        最多有 m/2 向上取整个子树，或者 m/2 + 1个子树；除了根节点和叶子节点以外，
        3.  如果根节点不是叶子节点的话，那么它至少有两个子树
        4.  除了根节点外，其他节点都包含 n 个 key. 
    1.  Build the B tree:
        1.  B tree break the database down into fixed size blocks or pages.
        2.  Each page is identified using an address or location, so one page can refer to another like an pointer. 
        3.  Page saved to disk. Each page can have pointer point to other page or an key.
        4.  The left node on key is point to an page where each key in that page lower than current key; right node is point
        to a page larger than the key. 
        5.  The tree can keep going till the leaf child, it also have key to set the boundary but it don't have reference
        it only have values.
    2.  The number of reference to child pages in one page of B tree is called the branching factor, it typically several 
    hundred.
    3.  有算法在background running来让整个structure remain balanced:
        1.  4 level of 4KB pages with a braching factore of 500 can store up to 256 TB.
        2.  4 level of B tree --> one node have 500 reference or up to 500 child.
        3.  At the last level or the 4th level, there is power(500, 3) child
        4.  Each pages is 4KB --> 4kB * 500 * 500 * 500 / 2 (half used for boundary) = 25 * power(10, 7) KB = 250 GB
        但是不知道为什么书上说 256 TB.
    4.  B tree 的增加与修改：
        当B tree 增加一个value时候，如果下面的已经满了，就split 成两份然后range request 也相应的改变。
    5.  B tree的优点:
        1.  读起来被认为是更快的，B tree的读每次都只要找到一条path到 leaf node, 而root 到leaf也就是3-4 层。故而更快
        2.  只是每次的读都是从disk上读，也就是random的读，这样的读取是非常的缓慢的；
    6.  B tree的缺点：
        1.  B tree的写比较慢，首先写入要先做read，然后还要检查是否有这个key, 没有的话之后要检查是否满了。满了还要slpit并且改变整个的
        结构。故而时间会比较慢。
    5.  Reference:
        1.  https://zhuanlan.zhihu.com/p/24309634 插入
        2.  https://zhuanlan.zhihu.com/p/24350783 删除
        
5.  B tree vs LSM tree:
    1.  Rule of thumb: LSM trees are faster on write; B trees are faster on read. 
    2.  Advantage of LSM tree:
        1.  Faster on write:
            1.  B tree 需要写两次，一次是记录下现在正在做的操作，一次是真正的修改。这是为了防止突然的failure而LSM tree不需要
            2.  这种行为叫做 write amplification, LSM的 write amplification 非常小。
        2.  LSM tree 的size更小，故而同一台机器上可以存储更多的内容。SSTable的overhead 远比B tree page的小，毕竟B tree的上面的多层
        都只是一个reference而不是一个有用的page,就算在最后一层上，也有一半的space被用作 boundary. 
    3.  Disadvantage of LSM tree:
        1.  Compression process是消耗资源的，在idle状态下可能还好，但是如果在峰值的时候做这个操作就会造成load增加。毕竟memory是shared
        于新的读、写与compression的，于是就造成了其他的读写操作受影响；同样，当写到磁盘上的时候也会和写新的SSTable产生share 磁盘write
        bandwidth的情况，由此造成performance收到了影响
        2.  LSM tree的key不是唯一的，在LSM tree中 key肯定是有duplicate的，并且需要compaction来解决这个问题。而B tree中的key 是唯一的
        对于transactional semantics 是需要的（说是要lock key）
    
6.  Column orient database:
    1.  Row based 的 database 的问题：如果要查找multiple row上的一个column的结果做分析。那么时间上非常慢，因为需要将整个row上的所有
    的data 都load到memory中，然后再从中提取想要的data，再来才能process。
    2.  Column orient database or wide column database 就是尝试解决这个问题：
        1.  它的储存是按照column来的，因此by default就解决了这个问题。每次只需要load 这个column的data 到memory中就好
        2.  它的另一个优势是减少存储空间，使用bitmap来表示data 而不是原先的data格式，这样能省下来很多的时间
        3.  column的内容还可以sorted后存储，这样对于range request来说是非常的efficient的
            
            
# Chapter5 Replication: Keeping a copy of the same data on several different nodes:
1.  Considerations: sync or async replication, handle failed replicas. 
2.  Replica 的主要作用有：
    1.  High availabilty: 一个倒掉了不会全部掉，整个网络永远都可以access
    2.  Latency: 就近的处理信息，让网络延迟尽可能的少
    3.  Scalability: 分流掉了 read的request
2.  Master slave model: (most popular one since it don't have conflict resolution to handle)
    1.  Usually have 3 replicas: 一般来说在一个data center有两个，这样一个leader failed了可以快速的切换另一个，由于在同一个datacenter，
    两个机器上的内容差异较小；但是相应的如果当前的 data center failed了，如断电、地震等，那么就需要完了；故而一般会在另一个data center再有
    一个follower 来保证系统的运行；
    2.  Sync or async replication:
        1.  所谓sync or async，其实就是user在做update的时候，Master先被update了，然后Master会发送write请求给slave要求他们update。
        那么是否告诉user他的update已经完成？还是说等slave都update后再告诉user完成了？因为user的 write 肯定是在master上，
        但是user 的read 就可能发生在任意的slave机器上。
        2.  如果此时slave机器上并没有接受到write 的请求，那么user会发现update没有生效，这是user在master上update完成后就直接回复的缺点；
        3.  那么如果等所有的slave都write完成了的话，那么user的等待时间就会非常的长。毕竟有一个slave可能在很远的地方。更可怕的是，如果slave
        宕机了，那么不可能被update，user要无限制的等待下去。
        4.  In practical, 一般认为sync的情况不是等所有的slave都回复，而是说等一个slave回复就可以了，其他的仍旧是async
        5.  In practical, leader based replication is configured to be completely async. Async is widely used.
    3.  New follower的setup:
        1.  Master 每隔一段时间就保存一个snapshot（包括timestamp）, 然后当new slave setup的时候，先copy snapshot到自己的机器上
        并建立内容，然后再向master要这个时间点之后的所有的update，然后就catch up 了
    4.  handling node outage:
        1.  Slave failure: 
        类似于new slave的add，向leader request all change after the last stamp the failed slave processed. 也叫catch up
        2.  Leader failure: failover
        one of slave need to promoted to be the new leader.
            1.  两种方法：其一：election；其二：previous leader appoint
            2.  注意的是要将恢复了的leader变成slave，否则两个machine 都认为自己是leader会造成write的重复发送；例如1，2，3三台机器，
            1号作为master failed，2号promote 成了master，那么2号会接受所有的写，然后再去写1/3, 此时3起来了觉得自己是mater，在收到了
            write后又会再向其他的node发送write。
    5.  Replication 的log的design:
        1.  Statement based: 不常用。
            1.  例如说直接发送client的SQL语句给周围的node，但是这样有很严重的问题在处理 Now和random, 每个指令到machine的时间不同，now肯定
            不一样；而每个random出来的也肯定是不一样的；
            2.  另一个缺点就是每个指令到达的机器的时间不一样，如果在master上后发的先到了并且与前面的有冲突，那么无法处理了
        2.  Write ahead log shipping:
            1.  直接将log发送给其他的机器。例如说red black tree在存入到disk成为SSTable时时有log来防止memory丢失的。这就是log，将这个
            发送给其他的node就可以了。
            2.  缺点就是如果两台机器的database不同，version不同，都可能造成log的logging以及understanding都是不同的。此时往往需要downtime
            来同时upgrade整个database
        3.  Logical log replication:
            1.  这就是说从当前的data/log中建立一个逻辑log来表示操作。这样就明白就脱离了底层的database的实现的细节而直接可以直接在多台
            不同的机器上共同使用
        4.  Trigger based replication:
            1.  自己设计一个code来update，当有update来的时候，code对它进行处理后再写入到database当中。
    6.  Problem with replication lag:
        1.  用户update之后再看内容发现没update，因为Master上的update了但是slave没update，而用户读取的时候可能是任意的一台slave
            1.  优化1：常修改的部分只读于 Master
            2.  优化2：在timer(1mins)内的修改后的read都只读于Master
            3.  优化3：给用户的request中加入下上次修改这个内容的时间
        2.  多次读取消息不一致，由于replication lag导致slave上的内容稍有不同。譬如你看微博，第一次刷看到了一些内容，第二次刷就看不到了，
        丢失了。这是因为用户先连上了更新的server，然后再刷的时候连上了老的server
            1.  优化：Monotonic Read: 保证从不回看。一种方法是短期内的读只读与同一个replica
        3.  有时间上前后关系的内容被无order化，例如群聊的时候，A先说再是B，然后C在旁边看却看到B的回复先然后才是A的问题。这就很奇怪了。这是
        因为C连的服务器对A/B的data到来的顺序不一样。
            1.  优化：consistent prefix reads. 保证write的顺序不变。Write always hapend on same replica
    7.  多个的replica node非常适合于read 多写的少的情况。这样每个node都分流了，但是一定要考虑上述的replication lag的问题。

3. Multi-leader solution: More robust in presence of network interruptions and latency spikes but very weak consistency guarantees
    1.  Single leader的主要的问题在于说如果一个用户因为什么原因无法连上 Leader/Master 的话，那么就无法对 database进行任何的修改。
    2.  一般来说，在一个data center里面使用multi-leader是很少见的。一般的操作是在多个data center中留一个leader. 在data center内部仍旧
    使用regular的master-slave的模型。
    3.  但是因为multi-leader的存在，多个leader之间肯定是有conflict的，那么就需要一个additional的 conflict resolution 模块。
    4.  与single leader的对比以及使用场景
        1.  由于有多个leader，故而能分流写的操作。然后由于每个data center都在用户附近，故而写的操作也更快（网络延时更小）--> write heavy的应用
        2.  某个data center 的leader挂掉了并无关系，甚至整个data center都挂掉了也没有关系 --> reliability要求较高的情况
        3.  最大的问题就是 conflict resolution
        4.  线下处理的时候也一定是要用 multi-leader的，例如在手机上update了的东西和在电脑上应该要同步；但是用户的update可能只是在offline
        你需要保存offline的情况下的用户的update也要保持一个非update的version，这时候如果只是signle leader，那么它连接着某个device，在等待
        update，然而其他的device要access这个node的时候就会出问题
        5.  同时update一个文件的时候也需要 multi-leader，因为多人在对同一个database上的文件进行操作。
    5.  Conflict handling:
        1.  处理方式
            1.  写带上 unqiue ID(timestamp etc), 故而可以使用 last write wins的策略丢掉更靠前的data
            2.  写带上 replica ID，故而丢弃的时候可以根据replica ID来丢弃
            3.  merge both write
            4.  弹出提示，要求用户决定
        2.  处理情况：
            1.  在写的时候处理：当发现conflict 的时候就决定是丢弃掉哪一个
            2.  在读的时候处理：当发现conflict，都写入，然后再在读的时候进行处理
    6.  Replication topologies:
        1.  Circular topology: 作为环来处理，lead1 update 后传给leader2, 然后3，然后4 直至传回给leader1
        2.  Star topology: 类似于树的结构，改了自己后给所有的子树发修改要求
        --> 上面的两种方式有个严重的问题就在于说如果有一个node failed了，那么余下的都是问题。于是有了第三个方式
        3.  All to all topology: 向所有的node都发，每个包都包含其经过的node的ID，每个node会尝试解包，如果发现自己的node在，那就discard.
        --> all to all 的问题在于order 无法保证；假如client A在lead1上将 A 从1 改为0；C 在A改完了后的时间上在 lead3 上将 A 从 1 改为 2
        那么这时候二者都会发 change request to lead2. 如果 lead3的信息先到，那么 lead1的信息可能就会被 discard.
        4.  解决办法是 version vector.

4.  Concurrent handling:
    1.  Master slave model:
        1.  通过version ID来保存，每个用户端都保存一个local 的version ID, 而在服务器上也有一个 version ID. 每次在服务器上有修改
        服务器上的version ID + 1, 然后用户在再次写的时候，将自己的version ID发给服务器，如果服务器上的version ID更大的话，那么保留另一个
        version上的change到新的value上，然后一起返回给client, client再做merge操作并记为 server return的 version ID + 1
    2.  Other model:
        1.  类似于上述的情况，只是每个replica上都有自己的version, 故而需要一个 version vector instead of version number. 
        
        
# Chapter 6: Partitioning/Sharding: Splitting a big database into smaller subsets
1.  Partitioning is combined with replication, 假如说一个database被分成了三份，且replica了三份。这样的话，这样每台机器上都有一个完整
的database，但是并不是一台机器作为master来design，而是说将每台机器都作为其中一个partition 的master来design. 这样写也被分流了
2.  Skewed: partition的目标是平分来起到分流的作用，如果我们的parition是不公平的，那么肯定会造成局部的高流量。这个高流量区域叫做: hot spot
而造成了hot spot的partition被称作 skewed.
4.  要注意增加、减少机器的时候，数据迁移要尽可能的减少
3.  Partitioning 的方法：
    1.  Partitioning by key Range:
        1.  sort data based on key. 然后像排书架一样的将key插入到对应的partition中。例如根据第一个字母来插入，也可以根据一天中的时间来分区
        2.  每个partition的boundary可以是manuel chosen或者是 choose them automatically using rebalancing partitions.
        3.  问题：
            1.  开始的分区很难选择正确，而且随着时间的变动，分区的boundary也在不断的变化。非常容易造成hot spot
    2.  Partitioning by hash of key:
        1.  Hash后来处理。用Hash function 来 产生 random number between 0 - power(2, 32) - 1. 然后每个server都只负责一部分的hash
        number的量。这个hash number 到机器number的信息存储在 web server上。
        2.  Consistent Hashing
            1.  ``
        3.  问题：
            1.  在hashing过后，数据之间的order是没有的。
    3.  特别要考虑的地方：
        1.  Celebrity user: 当一个这样的user有更新后，大量的follower会去read那一个partition，造成了 hot spot. 修改的办法是在 user
        ID 去做hashing的时候在ID 前面和后面都加上一个或者多个random的number 然后再多次hashing，这样就能够hash到不同的partition上。
4.  Vertical Sharding vs Horizontal sharding:
    1.  Vertical sharding: 不同的服务内容的数据库分在不同的服务器上. 例如 user table (user ID + password) vs user profile table (background and photos)
        1.  缺点：不能解决某个table的load比较大的问题。
    2.  Horizontal sharding: Consistent hashing
        1.  
        