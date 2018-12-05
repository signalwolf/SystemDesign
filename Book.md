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
    1.  Relational Database: data is organized into relations (tables in SQL) where each relation is an 
    unordered collection of tuples(rows in SQL)
        1.  Disadvantage:
            1.  Object relation Mismatch: 在程序处理数据的时候往往是以object的形式处理的，但是存的时候就要考虑怎么修改
            成为relational model了；--> impedance mismatch
            2.  不适合于 one to many relationship: 对于 one to many的处理就是需要使用另一个table来记录；例如在linkedin
            上的一个人是有多个工作经历的；这样的map就是一对多；那么在relational database的结构中就产生了问题：我的主表是user表
            user -> name, summary, location等的map，但是 job/education则是要去从整个的 job的表中去找，找user ID == curr
            requested user ID 的情况，然后再 join 到给user的feedback中，这样的过程非常的久。
    2.  Document Database
        1.  Advantage: 
            1.  greater scalability: support very large datasets or very high write throughput.
            2.  Free open source software. 
            3.  Better locality: 对应于relational database的第二个disadvantage. 由于document database使用的是JSON/XML
            file的存储方式，故而其本身就是一个Tree的结构，多份工作或者多份学历只是将Tree Node的 child增大而已。
        2.  Disadvantage:
            1.  Many to one/many problem: many people live in one city; many people work in one company, etc...
            对于relational database来说，由于本身这些city, company信息就是有自己的单独的表格，因此处理方式就是找到job表中 city =
            requested city or company = requested company的user id 然后再populate；
            但是在document database中，这些city/company只是user的一个entity，故而需要新建一个document, 并且让user的city/
            company中增加一个reference ID 去指向这些
              
    reference: 
        1.  MongoDB one to one:
        https://docs.mongodb.com/manual/tutorial/model-embedded-one-to-one-relationships-between-documents/
        2.  MongoDB one to many:
        https://docs.mongodb.com/manual/tutorial/model-embedded-one-to-many-relationships-between-documents/  

10. Backup:
    key value; column family
    Graph data;
    Cypher, Neo4j, datalog,datomic, SPARQL, OrientDB, 