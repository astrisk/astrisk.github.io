---
title:  "RABBITMQ-消息生命周期"
date:   2017-05-08 16:09:51
categories: RABBITMQ
---

转自：http://jzhihui.iteye.com/blog/1567232

（注：分析代码基于RabbitMQ 2.8.2）
当客户端通过basic.publish命令（AMQP定义）发布一个消息时，rabbit需要经过以下几个步骤处理消息：

1. 根据客户端传来的消息内容及相关属性（目标exchange，routing keys，mandatory及immediate属性）构造一个消息实体；
2. 根据要投递的exchange及routing keys匹配消息的目标投递队列名称；
3. 根据队列名称找到对应的处理进程ID；
4. 通过deliver消息（Erlang消息）向目标进程投递消息实体；
5. 队列对应进程收到投递的消息后，会试图向关联到此队列的消费者投递消息，如果投递失败，或者直接丢弃，或者将消息入队列，入队列时会根据消息及队列的属性（是否durable）判断是否需要写入磁盘，；
6. 保存到队列的消息，在下次有消费者关联到对应队列时，试图重新投递，直到投递成功，或者因为过期被丢弃掉。

本文的目的主要是分析上面各个步骤中的主要逻辑。

**创建消息实体**

消息实体由结构#basic_message表示，包含exchange_name，content，id，is_persistent，routing_keys几个域组成。没什么复杂逻辑，主要就是从客户端拿到这些信息后，再组织成一个#basic_message结构。
在实际投递消息之前，其实还有一层封装：#delivery结构，它只是在#basic_message的基础上封装了几个根据投递行为相关的属性，包含：mandatory（此值影响消息在未能成功路由到一个队列时的应对策略，如果为true，则会向客户端返回一个“无法路由（unroutable）”的错误消息；如果为false，则会直接丢弃消息），immediate（此值影响消息无法立即投递到消费者时的应对策略，如果为true，则会向客户端返回一个“无法投递（undeliverable）”的错误消息；如果为false，则服务器会将消息入队列，但是不保证消息最终会被消费者消费），sender（处理此消息的channel进程），message（对应上面的basic_message），msg_seq_no（从1开始的序列，每收到一个消息加1，此值只有存在AMQP中定义的事务的时候才有效）。
 
**匹配目标队列**

主要完成由客户端指定的routing keys匹配到目标队列的功能。在创建消费者时，消费者会指定在目标exchange上的绑定关系（bindings，通过queue.bind命令创建），匹配主要就基于这种绑定关系。在AMQP中，有四种最基本的exchange：direct，fanout，topic，headers。在匹配目标队列时，分成三类：direct和fanout的匹配类似，topic与headers各为一类。

**direct与fanout匹配算法**

主要代码如下：

{% highlight c %}

match_routing_key(SrcName, [RoutingKey]) ->
    find_routes(#route{binding = #binding{source      = SrcName,
                                          destination = '$1',
                                          key         = RoutingKey,
                                          _           = '_'}},
                []);
match_routing_key(SrcName, [_|_] = RoutingKeys) ->
    find_routes(#route{binding = #binding{source      = SrcName,
                                          destination = '$1',
                                          key         = '$2',
                                          _           = '_'}},
                [list_to_tuple(['orelse' | [{'=:=', '$2', RKey} ||
                                               RKey <- RoutingKeys]])]).

find_routes(MatchHead, Conditions) ->
    ets:select(rabbit_route, [{MatchHead, Conditions, ['$1']}]).

{% endhighlight %}

（参见[$RABBIT_SRC/src/rabbit_router.erl --> match_routing_key/2]）
SrcName为目标exchange的名称；direct的exchange在匹配时，传入的第二个参数是客户端发送的routing keys，fanout传入的是[‘_’]。
从上面代码可以看出，匹配主要使用ets:select/2函数来完成。只有一个routing key时，match_routing_key函数完成的功能很简单：按照exchange名称（source），routing key（key）在rabbit_route表中进行匹配，并把匹配记录的队列名称（destination）返回。其中rabbit_route表是在客户端通过queue.bind命令（AMQP定义）绑定队列与exchange时，由rabbit写入的，参见[$RABBIT_SRC/src/rabbit_binding.erl --> add/2]。
fanout类型的exchange在匹配时传入[‘_’]，会匹配到关联到exchange的所有队列。

当有多个routing keys时，find_routing中最终传入的第二个参数会类似如下形式：
[‘orelse’, {‘=:=’, ‘$2’, RKey1}, {‘=:=’, ‘$2’, RKey2}, {‘=:=’, ‘$2’, RKey1}, {‘=:=’, ‘$2’, RKey3}]
什么意思呢，{‘=:=’, ‘$2’, RKey1}是说 RKey1与$2所代表的变量要精确相等，第一个元素’orelse’代表其它的元素是“或”关系，也就是说只要#route结构中的key匹配RKey1, RKey2, RKey3中的任意一个就会返回匹配的队列名称。

**topic匹配算法**

topic的exchange基于类似正则表达式的方式来说明routing pattern（同bindings）。AMQP对用于topic类型exchange的routing key（由生产者在发布消息时指定）有如下规定：由0个或多个以”.”分隔的单词；每个单词只能以一字母或者数字组成。routing pattern增加如下规则：*可以匹配单个单词；#可以匹配0个或者多个单词。例如：routing pattern “*.stock.#”会匹配到routing key “usd.stock”和“eru.stock.db”，但不会匹配“stock.nasdaq”。

rabbit在收到一个topic类型exchange的绑定请求时，会根据routing pattern生成一个Trie结构：其中的边为以“.”分隔的单词，结点唯一编号。一般的Trie结构会是个树形结构，但是在AMQP的场景下，退化成类似一个链表。对于“*.abc.xyz.#.end”会生成如下结构：
![base1]({{ site.url }}/rabbitmq/base1.png)

（代码参见[$RABBIT_SRC/src/rabbit_exchange_type_topic.erl --> internal_add_binding/1]）
在收到一个消息时，rabbit会根据绑定时创建的trie结构进行搜索，比如对于routing key为test.abc.xyz.123.456.end搜索路径如下：
![base1]({{ site.url }}/rabbitmq/base1.png)

从node1到node2的路径，rabbit会首先尝试用“test”匹配，发现没到直达路径，然后尝试以“*”匹配，成功，node2到node3以“abc”匹配成功，node3到node4同理，node4到node5以“#”号匹配，然后在node5结点要跳过任何不能匹配node5到node6路径的单词，这里是“456”，最后匹配到“end”。
rabbit在这个算法的实现上有点奇怪，例如从node2到node3的匹配，即使“abc”路径已经匹配，但还是会尝试通过“*”和“#”匹配，增加了很多无意义的比较。
（代码参见[$RABBIT_SRC/src/rabbit_exchange_type_topic.erl --> trie_match/2]）

**headers匹配算法**

AMQP协议对headers类型的exchange的匹配算法有如下规定：由“x-match”头来控制匹配模式，分两种，all和any，类似于布尔运算里的AND和OR：如果匹配模式是all，则目标消息中的头信息的值必须匹配所有在绑定时指定的头信息；如果为any，则只要有一个头信息的值匹配就可以。其中的匹配是指，要么绑定时指定的头信息值为空，要么目标消息中的头信息的值与绑定时指定的值完全一致。
rabbit在实现时，会对绑定时指定的头信息和目标消息中的头信息进行排序（以头信息的键升序排列），然后逐个比对。
想不通这里为什么进行字母序排序（代码里也称这里的匹配算法是Horrendous matching algorithm，具体参见[$RABBIT_SRC/src/rabbit_exchange_type_headers.erl --> headers_match/2]），基于hash map类的数据结构更高效一些。

**查找队列进程**

每一个队列在创建时，会在rabbit_queue数据表中写入#amqqueue，其中跟进程相关的两个域是pid和slave_pids（在HA策略下slave_pids才有效，代表各个slave结点上的镜像队列的进程ID），pid代表的进程由[$RABBIT_SRC/src/rabbit_amqqueue_process.erl]。
查找的过程其实很简单，就是从rabbit_queue数据表中，根据队列名称找到相应的#amqqueue记录，并将相应的pid和slave_pids全部返回。

**投递消息**

找到队列对应的处理进程后，通过代理的方式（见[$RABBIT_SRC/src/delegate.erl]）向各个队列进程（包含镜像进程）发送deliver消息。rabbit_amqqueue_process在收到deliver消息后，会尝试将消息投递到某个消费者（参见[$RABBIT_SRC/src/rabbit_amqqueue_process --> attempt_delivery/3]）。最终会通过[$RABBIT_SRC/src/rabbit_writer.erl --> send_command_and_notify/5]调用以basic.deliver命令（AMQP）将消息内容发送给消费者。如果投递失败，有两种可能的结果：一种直接丢弃，另一种会将消息保存在队列中，等待后续有新的消费者加入时重新投递。保存在队列中的消息会根据durable属性来判断是不是需要写入磁盘，一般情况下，此值为false，消息只保存在内存中，如果需要持久化，此值为true，消息会写入磁盘。

**队列机制**

跟这部分相关的涉及到rabbit_msg_store模块和rabbit_queue_index模块。而且backing queue为rabbit_mirror_queue_master的队列还涉及到GM（Guaranteed Multicast）相关的东西，所以打算专门写一篇分析这一块。





