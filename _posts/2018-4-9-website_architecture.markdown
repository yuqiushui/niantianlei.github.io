---
layout:     post
title:      "大型网站架构"
subtitle:   " \"website architecture\""

author:     "Nian Tianlei"
header-img: "img/post-bg-2016.jpg"
header-mask: 0.4
catalog:    true
tags:
    - 架构设计
---


### 大型互联网系统特点
高并发，瞬时大流量：淘宝双十一、京东六一八，需要面对高并发用户       
高可用：系统7*24小时不间断地提供服务，有瞬间宕机就会成为焦点，极大影响用户体验  
海量数据：管理海量数据，需要大量服务器  
用户分步广泛，网络场景复杂：巴西，美东等各地网络情况千差万别，中美光缆数次故障，很多国外用户多的网站要考虑在海外建立数据中心  
安全环境恶劣：互联网容易受到攻击，大型网站几乎每天都会受到黑客攻击  
需求快速变更，频繁上线：用户总会有各种各样的需求，大型产品每天每周都有新版本发布  
渐进式发展：几乎所有大型网站都是从一个小网站开始的，好的互联网产品都是慢慢运营起来的，不是一开始就开发好的。  
大型网站最难的地方还是在于庞大的用户群体，高并发的访问和海量的数据。  

## 架构设计
**1.服务器分离**  
随着业务的发展，数据越来越多，导致存储空间不足。所以需要将应用和数据分离，分成三台服务器，应用服务器，数据库服务器和文件服务器。  
其中，应用服务器需要更快的业务处理，所以需要更优秀的CPU。  
数据库服务器需要更快的数据检索和数据缓存，所以需要更快的磁盘和更多的内存。  
文件服务器需要存放更多文件，所以需要更大的硬盘。  

网站并发能力得到很大改善，不过可能用户数量急剧增多，数据库压力急剧增大，导致访问延迟，从而影响整个网站的性能。  

**2.缓存技术**  
大部分的访问集中在少部分的数据上。  
我们可以将使用较频繁的数据缓存起来。提高访问速度。  

缓存分为两种，本地缓存和远程缓存。本地缓存就是直接缓存在本地应用服务器上，速度更快不过可能与本地的应用程序争抢宝贵的内存资源。远程缓存就是远程分布式缓存，可以使用集群的方式，部署大内存的服务器作为专门的缓存服务器。这种方式理论上不受内存空间大小限制。  

数据访问压力得到有效缓解，但是单一应用服务器处理的请求数量有限，应用服务器作为瓶颈。  

**3.应用服务器集群**  
对于大型网站而言，当一台服务器的能力不足时，不要试着换更强大的服务器，再强大的服务器的能力也无法满足持续增长的用户请求。  
这种情况下就是增加一台服务器分担原有服务器的访问及存储压力。  
于是可以不断增加服务器不断改善系统性能，从而实现系统的可伸缩性。  

通过负载均衡调度服务器，可将用户访问的请求发送到应用服务器集群中的任何一台服务器上，若是有更多的用户那就加入更多的应用服务器，使应用服务器的负载压力不再成为瓶颈。  

**4.数据库读写分离**  
网站使用缓存后，使得大部分读操作不需要经过数据库，但是小部分读操作和全部写操作都需要经过数据库，网站用户到达一定规模后，数据库操作就成为网站瓶颈。  

目前大部分的主流数据库都支持主从热备功能，就是主数据库修改数据，从数据库的数据会自动更新同步。  
通过配置这种关系，可以将一台数据库的数据同步更新到另一台服务器上。  
可以实现读写分离，从而改善数据库负载压力。  

有个主数据库用来写数据，从数据库用来读数据。主数据库更改了数据后，从数据库就可以立即同步， 这样就实现了读写分离。  

**5.反向代理和CDN加速响应**  
为加快网站访问速度，主要手段有 CDN和反向代理。  
CDN和反向代理的本质都是缓存。都是为了尽早返回数据给用户，一方面加快用户访问速度，另一方面减轻后端服务器的负载压力。  

CDN代理在用户通过域名解析ip地址时，通过解析用户ip地址来将请求转发到距离用户距离最近的应用服务器上。用户就可以就近取得所需数据。  

反向代理部署在网站的中心机房，可以将请求转发给真实服务器，反向代理自己也可以缓存一些数据，若是用户需要就直接返回给用户。  

**6.分布式文件系统和分布式数据库系统**  
过去我们读写分离，将一台服务器拆成两台，但若是性能依然无法满足需求，那么就使用分布式数据库。文件系统也一样，需要使用分布式文件系统。  

分布式数据库是网站数据库拆分的最后手段，只有在单表数据规模非常庞大的时候才使用。  
网站往往是业务分库，也就是说不同业务的数据放在不同的数据库上。  

**7.使用NoSQL和搜索引擎**  
网站业务越来越复杂，对数据存储和检索的需求也越来越复杂。  
一些非关系数据库技术如NoSQL和非数据库查询技术如搜索引擎。  
NoSQL和搜索引擎对分布式的支持会更好，我们可以通过统一的模块去访问各种数据，减轻多数据源的麻烦。  

**8.业务拆分**  
将不同业务拆分为成不同的应用。应用之间通过超链接建立关系。比如说在导航栏每个链接都指向不同的应用地址。也可以通过消息队列进行数据分发。  

**9.分布式服务**  
由于数据库连接的资源也是有限的，每一个应用服务器都要和数据库建立连接，很容易导致数据库连接资源不足，而拒绝服务。

然而对数据库的操作也就那么多种，我们可以将每种操作提取出来，独立部署。

然后应用系统就可以调用这些公共服务，来完成具体业务操作。


## 架构模式
大型企业对于构建高性能，高可用，易伸缩，可扩展的应用有很多解决方案。这些方案被其它网站重复使用，从而形成了大型网站架构模式。  

**1.分层**  
应用层：负责具体业务和视图展示。  
服务层：为应用层提供服务支持。  
数据层：提供数据存储访问服务。比如数据库，缓存，文件。  

各层都有一定的独立性，只要维持接口不变，各层可以根据具体问题进行独立修改。对其他层影响较小。  

分层架构是逻辑上的， 他们可以部署在同一个服务器上或不同服务器上。  

一般情况下，上一层可以调用下一层的服务，但是下一层却不可以调用上一层的接口，这样为了降低耦合。上层依赖下层，避免循环依赖。  
也禁止跨层调用，应用层的不可以直接调用数据层的服务。也是为了降低依赖。  

分层的初衷是便于开发，便于维护。实际上，分层有利于成为分布式应用。  



**2.分割**  
网站功能越复杂，服务和数据处理种类越多，将这些功能全都分割开来，分割成高内聚低耦合的模块。  

有利于分布式部署（将不同的模块部署到不同的服务器上），也有利于开发和维护（扩展性更好）。  

对于复杂的大型网站而言，分割为很多下模块，以降低复杂度。  

**3.分布式**   
分层与分割都有利于分布式部署，分布式部署就意味着可以允许更大量的并发。我们可以将不同的层部署在不同的服务器上，这样就可以根据每一层的使用情况来调整服务器的数量，更有效地利用服务器资源。   

常用的分布式方案有以下几种：
分布式应用和服务：将分层和分割后的应用和服务模块分块部署（高内聚低耦合），可以使不同的应用复用共同的服务。  

分布式静态资源：网站的静态资源，如JS，CSS，图片等独立部署（使用另外的域名），可以减轻应用服务器压力。去请求其它服务器，而不是请求应用服务器。  

分布式数据和存储：海量数据，一方面可以对关系型数据进行分布式部署，另一方面使用NoSQL数据库，NoSQL数据库基本都是分布式的。  

分布式计算：网站可以将一些计算量巨大的任务化为很多小任务，然后可以将这些小任务交给不同的服务器，最后获得计算结果，比如说搜索引擎。  

但分布式也带来了不少问题，通过远程接口调用来使用其他服务器上的服务，通过了网络，可能性能受到影响。    

服务器越多，服务器宕机的可能性就越大。而且一台服务器宕机带来的影响也很大，导致很多应用不可用，网站可用性降低。另外，数据在分布式环境中保持数据一致性也非常困难，分布式事务也难以保证。  


**4.集群**  
对于用户访问集中的模块还是要放在多个服务器上，使用负载均衡，当某个服务器出现错误，会将请求转发到正常的服务器上，提高可用性。  

对于较小的分布式应用和服务，也至少部署两台服务器，当某台服务器出现问题时，有一个备用服务器，不会报错至少用户的体验不会很差。  
使用集群后，更加灵活，可以通过增加服务器数量来提高并发能力。  

注：分布式和集群的区别：  
分布式侧重于将不同的服务交给不同的服务器（多个服务器配合完成一个工作），而集群侧重于增加相同功能的服务器（将很多相同任务分给多个服务器，每个服务器独立处理几个任务）。  

**5.缓存**  
较大的软件中，缓存几乎无处不在。  

CDN: 使用到缓存，在网络提供商那里缓存一些静态资源，可以直接返回给用户。  
反向代理：用户直接与反向代理交互，反向代理也可以缓存一些静态资源。  
本地缓存：用户程序会缓存较热的数据，使得不用访问数据库。  
分布式缓存：将各种缓存放到分布式缓存集群中。  

**6.异步**  
为了降低耦合性，就是把模块之间依赖降低。异步是实现解耦的一个重要手段，因为它们不是直接调用的。  
在同一台服务器中，我们通过多线程，一个线程将数据放在一个队列中，另一个线程从队列中获取数据并进行操作。   
在分布式系统中，多个服务器通过分布式消息队列（就是普通队列的分布式部署）来实现异步。   
异步是典型的生产者消费者模式。  
彼此功能随意实现而不相互影响，十分有利于扩展功能。  

使用异步的好处有：①提高系统的健壮性，若是消费者服务器暂时出现问题，但生产者依然会表现出正常的样子（处理业务），消息队列不断增加。等到消费者服务器恢复，又可以从消息队列中取出数据。   
②有利于缓解高并发，当数据量很大，消费者会逐一从消息队列中取出消息，而不是直接宕机。  

**7.冗余**  
服务器需要7*24小时运行，当服务器很多时，服务器宕机是必然事件。  
所以说，要保证某个服务器宕机后，不丢失任何数据。所以就要保证数据冗余备份。  

因此，即使负载很小，也必须部署两台服务器，以实现数据备份。数据库除了定期备份，还要实现主从分离，实现热备份。  

为了抵御不可抗拒因素，某些大型网站会对整个数据中心进行备份。  

**8.自动化**  
发布过程的自动化可以有效减少故障。  
比如自动化代码管理，自动化测试，自动化安全监测，自动化部署。  

**9.安全**  
登陆，交易等重要操作需要对网络通信进行加密，并对用户提交的数据进行编码。常见的用于攻击网站的XSS攻击，SQL注入。对于交易转账等重要操作进行风险控制。 

