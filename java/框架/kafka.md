### 一.kafka简介

​		kafka主要是一个**分布式流式处理平台**,主要的功能点在于三个方面:

​		1.**消息队列**:主要就是发布和订阅消息,类似于其他的消息队列.

​		2.**容错的持久方式存储记录消息流**:kafka支持消息持久化到磁盘中,防止消息的丢失

​		3.**流式处理平台**:在消息发布的时候进行处理,kafka提供了很多的流式处理类库API

### 二.kafka的相关构成概念

​		1.**producer**:生产者,消息的发起者,也是生产者.

​		2.**consumer**:消费者,会消息这些消息数据.

​		3.**broker**:代理,其实代理的意思就是要一个kafka实例,多个kafka broker就会组成kafka cluster(集群)

​		4.**topic**:主题,消息生产者发送给kafka的消息将会存储在主题当中,而消费者则会通过订阅主题的方式从主题中去获取想要的消息进行消费.

​		5.**partion**:分区,其实主题topic是一个大的酒箱子,而partion则是里面的酒瓶子,真正存储消息的是partion.通常意义上讲的消息队列的队列通道,则就是kafka里面的partion分区的概念.在后面会进一步讲解partion的概念.

### 三.kafka的模型设计

#### 		1.早期队列设计模型

​		![图片](https://mmbiz.qpic.cn/mmbiz_png/iaIdQfEric9Tw5HhjBWfamF35XaNsxW3GH7q2gyBOE7YfUs03gX0YAhzJuOslcqewZC4KZw12a8usWAfrBAcjoPw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

​		使用队列queue来作为通讯通道,生产者将消息源源不断的发送给消息队列,然后消费者自己去队列中获取自己需要的消息来进行消费.但是这样有个弊端,就是队列有个性质就是先进先出,假如现在有100个生产者发送消息,然后队列里面就会存100个消息,有2个消费者来消费,每次消费者都会从队列中获取一条消息进行消费,那么后面进来的消息则会一直排队等候,可能会出现超时的情况.效率明显看起来会偏低.而且理想情况下,消费者挨个消费消息,那么2个消费者则会各消费50个消息.

​		假如现在要求,有100个消息,有2个消费者,必须让每个消费者都消费这100个消息.上面的队列设计模型就不太好处理,可以强行给每个消费者建立单独的队列通道,然后让生产者发送多份的消息到不同的队列通道中去,这明显不合理.而且很浪费资源,

#### 		2.发布-订阅设计模型

​		![图片](https://mmbiz.qpic.cn/mmbiz_png/iaIdQfEric9Tw5HhjBWfamF35XaNsxW3GH7CcZ39jmlbghrJ6qF7fUFFIoAia2xGQaNia0a4JtMmIrpWoib79wkPVXg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

​		简单描述下该模式,多个消费者向同一个topic主题进行订阅操作.生产者发送消息给topic主题,然后topic主题通过**广播模式**将topic中的消息传递给**所有的消费者**.假如在topic进行广播传递消息的过程中,又来了一个订阅该主题的消费者,那么这个消费者将收不到此次的消息.只能等到下次生产者重新发送消息给topic主题.

​		假如在某个场景下,只有一个消费者订阅了一个主题,那么,就会形成一个生产者,一个topic,一个消费者的局面,这不就是早期的消息队列模式了么.所以订阅-发布模式是可以很好兼容队列设计模型.

#### 		3.健全版的kafka发布-订阅模型

![图片](https://mmbiz.qpic.cn/mmbiz_png/iaIdQfEric9Tw5HhjBWfamF35XaNsxW3GHibA0TZlicjZY0AD34aCyiboEDXuYvLVUAPVB0icKe4MDVW9kictUibRP41DQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

​		在一个kafka cluster集群中,部署了两个kafka实例broker.在实例broker1中,设置了两个主题topic,而topic1主题中设置了两个partion分区.注意,此时在topic1和topic2中,都有partion1分区.这说明一个topic主题可以有多少不同的partion分区,而相同的partion分区又可以存在于不同的topic主题当中.甚至partion2还能在topic2中一起分布到broker2中.

### 四.kafka的特性

#### 		1.**kafka的高伸缩性**

​		那是一个topic主题可以跨多个kafka实例,而相同的partion又可以归属到不同topic上,只需要配置好生产者能将对应的消息发送到对应的主题上,高伸缩性就是这样体现出来的.

#### 		2.**kafka的负载均衡**

​		假设kafka不支持多主题多分区的扩展到多实例上,那么单个机器上的kafka实例压力会特别大,一旦被压崩溃,后果很严重.一旦支持了,生产者可以将消息发送到指定的topic主题上面,topic将消息存入对应的分区中,然后实现多台机器多实例的部署,这样整个集群的压力会较为均衡.

#### 		3.**kafka的容错性**

​		kafka对于partion分区的设定还有一个多副本(Replica)的机制,也就是kafka可以对某个partion分区设置一个Replica(复制)参数,该参数会让对应的partion有多个副本的存在.那么这么多的副本当中就会有一个leader的出现,其他的副本均是该leader副本的追随者(follower).生产者和消费者只能与leader副本进行交互,也就是说生产者发送的消息都是直接存在leader副本上的,而follower副本会去同步leader副本上的数据,消费者消费消息同理.

​		正是因为有多个follower副本的存在,假如leader副本挂掉了,那么kafka会通过选举的方式从完好的follower副本中重新选举出一个新的leader副本来进行数据直接交互.其实能参与竞选的follower副本是有要求的,也就是follower里面的数据要尽可能和原先leader副本的数据保持一致,一旦缺失的数据太多,该follower副本没有机会参加竞选.

​		总之,正是因为有多个follower副本的存在,保证了kafka的容错性,提高了容灾能力,但是也因为有多个follower副本的存在,提高了存储成本.

#### 		4.kafka的持久性(未完...)






