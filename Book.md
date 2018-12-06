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
                完全丢失了，这样无法还原了；故而在我们对数进行操作之前，我们先commit log，记录下我们要做什么，如果
                
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
               
4.  B-Tree: Most widely used: relational and nonrelational both use this structure:
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
5.  B tree vs LSM tree:
    1.  Rule of thumb: LSM trees are faster on write; B trees are faster on read. 
    2.  Advantage of LSM tree:
        1.  Faster on write:
            1.  B tree 需要写两次，一次是记录下现在正在做的操作，一次是真正的修改。这是为了防止突然的failure而LSM tree不需要
            2.  这种行为叫做 write amplification, LSM的 write amplification 非常小。
            
# Chapter5 Replication:
1.  Considerations: sync or async replication, handle failed replicas. 
2.  Master slave model:
    1.  Usually have 3 replicas: 一般来说在一个data center有两个，这样一个leader failed了可以快速的切换另一个，由于在同一个datacenter，
    两个机器上的内容差异较小；但是相应的如果当前的datacenter failed了，如断电、地震等，那么就需要完了；故而一般会在另一个data center再有
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
            write后又会再向其他的node发送write。************
        
            
    