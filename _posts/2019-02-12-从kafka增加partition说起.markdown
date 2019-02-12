---
layout: post
title:  "从kafka增加partition说起"
date:   2019-02-12 20:52:06 +0800
categories: jekyll update
---


*该文章为原创文章，转载请注明出处https://tyxing.github.io/*



kafka增加partition实在是一个常见的场景。

众所周知，每一个consumer线程从一个partition读取数据，这是kafka高吞吐量的原因之一。

kafka的partition数目是建立topic之初，根据业务所需的吞吐量提前计算好的。这个改天开新坑写一下。


现在如果需要临时增加kafka的partitions数目会有什么问题？

这里要从消息的key说起。

众所周知，kafka的消息是键值对组成。key默认值可以设为null。

这个key的作用主要有2点。

1是给该条消息进行补充说明，

2是进行在分区时参与处理。

在key为null 的时候，默认采用 Round-robin 策略，也就是雨露均沾，风水轮流转，实现类是 DefaultPartitioner。

但我们实际应用中为保持相关消息按序到，就必须送到指定的 Partition，方法可以有:

1.指定 Partition 编号

2.指定 Key

3.自定义 Partitioner - 实现 org.apache.kafka.clients.producer.Partitioner, 并通过属性注册。

因此，我们有把数据写到同一个partition的需求时候，需要指定相同的key。

然后kafka的分区器，会把这个key哈希处理，根据哈希值发送到不同的partition上面。

因此想把数据顺序写入同一个partition的话，指定相同的key是可以的。


现在问题来了，我增加了partition的数目，这个key 哈希是会受到影响的。增加后，新来的数据会发送去其他partition。

也就是说，只有不改变topic的partition数目，同一个key哈希去的位置才会不变。

这里比较规范的做法是，在使用topic之前就根据吞吐量需求，计算好partition的数目，使用时不去进行改变。
