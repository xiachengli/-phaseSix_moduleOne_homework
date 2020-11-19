#### 第一节 RabbitMQ架构与实践

##### 1.1 RabbitMQ简介、概念、基本架构

###### 1.1.1 RabbitMQ 简介

​	遵循AMQP协议，自身采用Erlang语言编写的消息中间件，最初是为了解决电信行业系统之间的可靠通信而设计的。

 ###### 1.1.2 整体逻辑架构

![image-20201020135654964](C:\Users\x1850\AppData\Roaming\Typora\typora-user-images\image-20201020135654964.png)

###### 1.1.3 Exchange类型

​	RabbitMQ常用的交换器类型有：fanout、direct、topic、headers四种

fanout：把消息发送到与此交换器进行绑定的消息队列中

direct：将消息发送到binding key与rounting key完全匹配的队列中

topic：topic类型的交换器在direct匹配规则上进行了扩展，及其支持“*”，“#”两种通配符。\*用于匹配一个单词、#用于匹配0或多个单词

headers：根据消息中的headers属性进行匹配。在绑定队列和交换器时指定一组键值对，当发送的消息到交换器时，RabbitMQ会获取该消息的headers，对比其中的键值对看是否完全匹配队列和交换器绑定时指定的键值对，如果匹配，消息就会路由到该队列。headers类型的交换器性能很差，不实用

###### 1.1.4  RabbitMQ数据存储

RabbitMQ消息有两种类型：

1. 持久化消息

   持久化消息在达到队列时写入磁盘，同时会在内存中保存一份备份，当内存不足时，消息从内存中清除

2. 非持久化消息

   一般只存在内存中，当内存压力大时数据刷盘处理

RabbitMQ存储层包含两个部分：队列索引和消息存储

![image-20201020141010814](C:\Users\x1850\AppData\Roaming\Typora\typora-user-images\image-20201020141010814.png)

- 队列索引

  - 索引维护队列消息的信息，如存储地点、是否已被消费者接受、是否已被消费者确认等

  - 每个队列都有相应的索引
  - 索引使用顺序的段文件来存储，后缀为.idx，文件名从0开始累加。每个段文件中包含固定的segment_entry_count条记录，默认值为16384。每个index从磁盘中读取消息的时候，至少要在内存中维护一个段文件，所以设置queue_index_embed_msgs_below值时要谨慎

- 消息存储

  - 消息以键值对的形式存储在文件中，一个虚拟主机上的所有队列使用同一块存储
  - 存储分为持久化存储和短暂存储
  - 后缀为.rdq，经过store处理的所有消息都会以追加的方式写入到该文件中，当该文件的大小超过指定的限制（file_size_limit）后，将会关闭该文件并创建一个新的文件以供写入
  - 消息可以直接存储在index中，也可以存储在store中。最佳的方式是较小的消息存在index中，而较大的消息存在store中。这个消息大小的界定可以通过queue_index_embed_msgs_below来配置，默认值为4096B

