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
    3.  Problem: 
        1.  Keys must fit into RAM. 如果data非常多就有问题了
        2.  需要compact data，对于新来的data，必须要compact data, 否则disk就fill非常快了
        3.  Range request 非常麻烦，需要扫描整个 hashmap.
    4.  由于这些因素，hash index适合于key 非常的少，并且 write非常的多的情况；
    
3.  Optimization2: SSB
    