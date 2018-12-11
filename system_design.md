## steps:
Step 1: Requirements clarifications: 搞清楚你需要解决的问题，问题的规模，问题需要的feature等    
Step 2: System interface definition: 解决问题需要实现的API   
Step 3: Back-of-the-envelope estimation: 预测问题的scale 
Step 4: Defining data model 
Step 5: High-level design   
Step 6: Detailed design 
Step 7: Identifying and resolving bottlenecks
Step 8: 

## 设计系统的主要的考虑：
1.  Scalability
2.  Reliability
3.  Availability
4.  Efficiency: delay (response time) and bandwidth (process volume)
5.  Serviceability or manageability


## System design moduels:
1.  Load Balancing:
    1.  就是负责分流的部分，负责观测其后的模块的健康程度。尽可能的将新来的流量给放到相较 load更小的模块中。它可以存在于Web server和
    application server间，也可以在Application server和database之间。
![image]()
    2.  使用 LB的好处:
        1.  更好的response time: 不会出现用户拥堵在一台机器上的情况。这样用户的回复时间比较均衡
        2.  更reliable 的service: 不会出现一台机器挂了之后用户还是被redirect到那台机器上造成大量用户等待的情况
    3.  LB 的算法：
        1.  Health check: 检查其身后的服务器的情况，并且由此来redirect
            1.  Least Connection method: 最少的connection的机器赢
            2.  Least response time method:
            3.  Least Bandwidth method: 其实应该是最小的Tput的算法，信息被引导到了Tput最小，即其已占用带宽最小的机器上
            4.  Round Robin method: cycles through each server till one machine's pool not full, otherwise, keep loop.
            5.  Weighted Round Robin method: 上面的方法的主要的问题是每台机器的配置不一样，因此加上weight对配置进行考虑
            6.  IP hash: hash it based on IP address.
    4.  LB 的备份：因为LB可能挂，如果挂了就整个不work了，因此需要多个LB.
    5.  Implementation of LB:
        1.  Smart Client: 用户端作为LB来查看哪台机器好，然后向相应的地方发送包裹
        2.  Hardware LB: Expensive but reliable, 一般只在对用户的一段加上HW LB. Citrix NetScaler
        3.  Software LB: HAProxy 
2.  Caching:
    1.  Cache的设计其实就是一个基本认知，最近刚用的很可能会再被用到。因此将一部分内容放在更快的cache中加快了操作的速度。
    2.  Type of cache: 基于cache的分布
        1.  Application server Cache: 每台机器上有自己的cache
            1.  在 local machine上加cache帮助处理的更快
            2.  但是问题是在与如果前面有load balancer的话，那么同一个用户的request 不一定还在当前AP server上因此造成了它变慢了
        2.  Distributed Cache: 整体作为一个大的cache，然后分布在不同的机器上
            1.  使用consistent hash来分配大的cache到每台机器上
            2.  优势是非常容易增加cache的容量，例如我们加一台机器的话，可以直接加到distributed cache上
            3.  劣势是如果cache挂了的话的处理，cache如果挂了的话，那么就无法恢复；故而需要做additional的cache的replica
        3.  Global Cache: 整个机器作为一个cache来处理数据，所有的服务器都认为它是cache
            1.  Global Cache有两种形式：
                1.  当server的request不在Cache中的时候，server不管，是Cache自己来处理这个问题。（常用）
                    1.  优势，如果大量的server都在要一个不在Cache中的数据的时候，按照法二，他们都向database request而形成了flood
                2.  当server的request不在cache中的时候，server自己去database中取，然后再考虑是否加入到cache中
                    2.  如果有大量的Cache missed的时候，不会有太大的delay
    3.  (Cache invalidation)Type of cache: 基于cache与database的沟通
        1.  Cache 的问题： data inconsistency:
            1.  Cache中的data如果和database中的不一样怎么办？Database可能被修改了，而cache因为没有读而造成了不一致
        2.  Write through cache:
            1.  在Cache 和database中都写入内容，缺点是要写两次故而造成了非常长的latency
        3.  Write around cache:
            1.  删除掉Cache中的内容，然后再到database中写入内容，这样就使得一致性非常好，只是最新修改的内容不在cache中
        4.  Write back cache:
            1.  只在cache中修改，然后直接回复修改完成。但是这样take risk
    4.  Cache eviction policies:
        1.  FIFO
        2.  LIFO
        3.  LRU/MRU: least/most recent use
        4.  LFU: least frequent use
        5.  RR: randomly replacement 
3.  Sharding:
    1.  Horizontal sharding (range based sharding)/vertical sharding
        1.  Horizontal sharding的问题是如果分布不均匀造成的hot spot
        2.  Vertical sharding的问题是如果用户增大，最后还是要horizontal sharding
    2.  Sharding method:
        1.  Consistent hash
        2.  List partition: each partition is assigned a list of values
        3.  Round robin partition: 找到当前最小的 server并加入到其中
    3.  Sharding后的问题：
        1.  Join 不能形成，Join只能处理在一台机器上的 join 而不能 cross machine
        2.  reference 到其他机器的也不能形成。
4.  Proxies: 代理:filter request
    1.  在Client和Server之间加入代理后，代理可以：
        1.  合并请求：如果说有100个request到同一个模块，那么代理可以合并为一个请求然后把它从database上提取到代理的cache中，这样的操作非常小了
        2.  合并相邻请求：如果说有100个request到相邻的模块，那么代理可以合并为一个请求然后将整个大模块提取到cache中
5.  Queue:
    1.  用于async task, 当用户请求来临的时候，如果不需要立即的同步，那么可以先放在queue里让后让用户继续操作。这样就不会影响用户的performance
6.  SQL vs NoSQL:
    1.  NoSQL 
        1.  Key value stroage: store data in key-value format: Redis, Dynamo
        2.  Document database: Group together in collections: MongoDB
        3.  Wide Column database: row, column, value
![image]()
        4.  Graph database: ignore for now
    2.  选择标准：
        1.  Scale: 相比于SQL, NoSQL更加的容易scale, 毕竟这一波的NoSQL database的兴起就是因为大规模的计算
        2.  ACID: SQL更老也更成熟，因此相比于NoSQL, ACID的performance肯定是更好的
        3.  如果data的结构可以肯定是不会变的，应该使用SQL, 其实差不多，只是SQL更老更stable
        4.  快速增长选NoSQL, 非常方便之后对data 进行修改
        5.  使用cloud compute建议使用NoSQL
7.  CAP: consistency, availability, Partition tolerance:
    1.  理论上只存在满足其中两个条件的系统，不存在三个都满足的系统
![image]()
8.  Long-Polling vs WebSockets vs Server-Sent Events:
    1.  Http的问题在于通信的发起者一定是 client, 不存在说server主动push data 的情况。因为如果 server要发东西就很麻烦，在传统的http的
    连接下，需要的是client不停的向 server发请求，无论server有无请求，都要回复，只是没有请求的时候发送的为空而已。但是这样非常的占用带宽。
    因此需要尝试去解决这个问题。
    2.  Long polling: 用户端发送一个消息，然后 server可以不回复。如果server 有update 或者当前的连接expired了，那么这个时候server再
    回复。这样 client - server之间的通信被大大的减少了。这个架构还是在http的架构之下，算是扩展
    3.  WebSockets (Full duplx): 新的架构，相当于 client 和 server 之间建立了一个单独的通道，如果server 要push data 就直接从socket 发，client
    也可以随时通过 Socket来发
    4.  Server sent Events: 还是 http的框架，只是client建立了http连接后不断开，不停的接收server的信息