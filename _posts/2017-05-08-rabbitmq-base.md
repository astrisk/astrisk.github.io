---
title:  "RABBITMQ-基础知识"
date:   2017-05-08 15:27:44
categories: RABBITMQ
---

RabbitMQ系列blog为《RabbitMQ in Action》的读书笔记，记录相关知识摘要。同时也有些关于RabbitMQ集群以及相关应用的测试方法。

## **队列**

1. 队列概念
	- 为消息提供了住所，消息在此等待消费
	- 可以起到负载均衡的作用
	- 消息的最后终点（除非消息进入了“黑洞”）
2. 创建队列
	- 生产者和消费者都可以通过Queue.declare创建队列；如果消费者在同一channel上订阅了另一个队列，则无法再声明队列，必须首先取消队列，将channle置为“传输”模式
3. 队列设置相关参数
	- Exclusive --true时，队列为私有队列
	- Auto-delete--当最后一个消费者取消订阅时，队列就会自动移除。
4. 检测队列
	- Queue.declare passive-true,如果队列存在，则命令会成功返回，如果不存在，不会创建队列，会返回一个错误。
5. 谁创建队列
	- 生产者和消费者都可以创建队列。如果消息被路由到一个不存在的队列，RabbitMQ会忽视消息，消息会丢失。	
6. 订阅队列消息的两种方式
	Basic.consume - 接收模式
	Basic.get - 获取单条消息
	
7. 消息分发方式
	以循环方式发送给多个消息订阅者
	
8. 消息确认
	消费者通过basic.ack向rabbitMQ发送消息确认，如果订阅到队列时，auto_ack = true,一旦消费者接收到消息，会自动视为确认消息。
	消费者确认后，rabbitMQ才能安全地从队列中删除消息
	如果消费者没有确认，在确认消息之前，rabbitMQ不会再给该消费者再发送消息
	
9. 拒绝消息
	把消费者从rabbitMQ服务器断开，消息会重新入队列，并发送给另一个消费者
	rabbitMQ2.0或以上版本，basic.reject = true,消息会重新发给下一个订阅者；basic.reject=false消息会被从队列中移除，不会发送给新的消费者

## **交换器和绑定**

队列通过规则绑定（routing key）交换器，当消息发送给交换器时，把消息的routing key和交换器绑定的routing key进行进行匹配，如果匹配成功，消息就会进入对应的队列；如果匹配失败，消息进入“黑洞”。

### 交换器类型

1. Direct
	- 如果routing key匹配成功，消息就会被投递到对应的队列。服务器必须实现direct类型的交换器，包含一个空白字符串名称的默认交换器。当声明一个队列时，它会自动绑定到默认交换器，并以队列名称做为routing key。
	
2. Fanout
	- 该交换器收到消息后，会被广播到绑定到fanout交换器的队列中。属于1 VS N的模式。
	
3. Topic
	- 该交换器，可以使得来自不同源头的消息能够到达同一队列。通过在队列绑定时，通过不同位置和类型的通配符来实现。

		Topic exchange can't have an arbitrary routing_key, it must be a list of words,delimited by dots. Routing key example:"stock.usd.nyse", "nyse.vmw", "quick.orange.rabbit". There can bu as many words in the routing key as you like,up to the limit of 255bytes.
		
		Two cases for binding keys:
		*(star) can substitute for exactly one word.
		#(hash) can sbustitute for zero or more words.
		
		If we break our contract and send a message with one or four words,like "orange" or "quick.orange.male.rabbit", these messages won't match any bindings and will be be lost.
		
		On the other hand "lazy.orange.male.rabbit", even though it has four words, will match the queue "lazy.#" and will be delivered to the second queue.
		
		
		把msg-inbox模块的error , info , warnning logs投递到msg-inbox-log 队列
		$channel -> queue_bind('msg-inbox-log', 'logs-exchange', '*.msg-inbox')
		
		All-log队列将会接收所有从web应用程序发布的所有日志
		$channel -> queue_bind('all-log', 'logs-exchange', '#')
	
4. Header
	- 使用消息的header而非routing key进行消息路由，但是性能比较差，几乎用不到。

## **多租户模式**

Vhost: 每个vhost就像一个mini版的rabbitMQ，拥有自己的队列，交换器和绑定以及权限机制。默认vhost: "/"  default user : guest  passwd:guest
	
1. Vhost 创建：rabbitctl add_vhost [vhost_name]	
2. Vhost 删除：rabbitctl delete_vhost [vhost_name]	
3. Vhost 查看：rabbitctl list_vhosts

## **消息持久化**

队列和交换器的durable属性。该属性默认为false，它决定了rabbitMQ是否需要在崩溃或重启之后重新创建队列(交换器)。
	
1. 持久化消息
	能从服务器崩溃中恢复的消息。在消息发布前，通过将消息的“投递模式”（delivery mode）
	选项设置为2，同时消息还必须被发送到持久化的交换器中。
		§ 消息投递模式选项设置为2
		§ 发送到持久化的交换器
		§ 到达持久化的队列
	消息 --> 持久化交换器 -->持久化日志文件 -->持久化队列
	
	RabbitMQ内建集群环境中，持久化消息工作得并不好。如果集群节点crash，节点上的队列从集群消失，并且持久化队列无法重建，消息也丢失。
	
2. 事务
	
	发送方确认模式--在rabbitMQ2.3.1或更高版本上可用channel上发布的消息都有一个唯一ID，一旦消息到达所有匹配的队列后，channel会发送一个发送方确认模式给生产者app（包含消息的唯一ID）

