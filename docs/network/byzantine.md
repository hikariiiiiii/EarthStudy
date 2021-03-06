# 拜占庭将军问题

## 什么是拜占庭将军问题

“拜占庭将军问题”也被称为“拜占庭容错”，是Leslie Lamport（2013年的图灵讲得主）用来为描述分布式系统一致性问题（Distributed Consensus）在论文中抽象出来一个著名的例子：

> 拜占庭帝国想要进攻一个强大的敌人，为此派出了10支军队去包围这个敌人。这个敌人虽不比拜占庭帝国，但也足以抵御5支常规拜占庭军队的同时袭击。这10支军队在分开的包围状态下同时攻击。他们任一支军队单独进攻都毫无胜算，除非有至少6支军队（一半以上）同时袭击才能攻下敌国。他们分散在敌国的四周，依靠通信兵骑马相互通信来协商进攻意向及进攻时间。困扰这些将军的问题是，他们不确定他们中是否有叛徒，叛徒可能擅自变更进攻意向或者进攻时间。在这种状态下，拜占庭将军们才能保证有多于6支军队在同一时间一起发起进攻，从而赢取战斗？

注：“  拜占庭将军问题中并不去考虑通信兵是否会被截获或无法传达信息等问题，即消息传递的信道绝无问题。

## 通俗分析

先看在没有叛徒情况下，假如一个将军A提一个进攻提议（如：明日下午1点进攻，你愿意加入吗？）由通信兵通信分别告诉其他的将军，如果幸运中的幸运，他收到了其他6位将军以上的同意，发起进攻。如果不幸，其他的将军也在此时发出不同的进攻提议（如：明日下午2点、3点进攻，你愿意加入吗？），由于时间上的差异，不同的将军收到（并认可）的进攻提议可能是不一样的，这是可能出现A提议有3个支持者，B提议有4个支持者，C提议有2个支持者等等。

再加一点复杂性，在有叛徒情况下，一个叛徒会向不同的将军发出不同的进攻提议（通知A明日下午1点进攻， 通知B明日下午2点进攻等等），一个叛徒也会可能同意多个进攻提议（即同意下午1点进攻又同意下午2点进攻）。

叛徒发送前后不一致的进攻提议，被称为“拜占庭错误”，而能够处理拜占庭错误的这种容错性称为「Byzantine fault tolerance」，简称为BFT。

## 问题抽象

求解拜占庭将军问题，隐含要满足以下两个条件：

1. 每个忠诚的将军必须收到相同的命令值vi（vi是第i个将军的命令）。
2. 如果第i个将军是忠诚的，那么他发送的命令和每个忠诚将军收到的vi相同。

于是，拜占庭将军问题的可以描述为：一个发送命令的将军要发送一个命令给其余n-1个将军，使得：

IC1. 所有忠诚的接收命令的将军遵守相同的命令；（一致性）

IC2. 如果发送命令的将军是忠诚的，那么所有忠诚的接收命令的将军遵守所接收的命令。（正确性）　

Lamport对拜占庭将军问题的研究表明，当**n>3m**时，即叛徒的个数m小于将军总数n的1/3时，通过口头**同步通信**（假设通信是可靠的），可以构造同时满足IC1和IC2的解决方案，即将军们可以达成一致的命令。但如果通信是可认证、防篡改伪造的（如采用PKI认证，消息签名等），则在任意多的叛徒（至少得有两个忠诚将军）的情况下都可以找到解决方案。

而Fischer-Lynch-Paterson定理证明了，**异步通信**情况下，只要有一个叛徒存在，拜占庭将军问题就**无解**。翻译成分布式计算语言，在一个多进程异步系统中，只要有一个进程不可靠，那么就不存在一个协议，此协议能保证有限时间内使所有进程达成一致。

拜占庭将军问题在一个分布式系统中，是一个非常有挑战性的问题。因为分布式系统不能依靠同步通信，否则性能和效率将非常低。因此寻找一种实用的解决拜占庭将军问题的算法一直是分布式计算领域中的一个重要问题。

在这里，我们先给出分布式计算中有关拜占庭缺陷和故障的两个定义：

> **定义1**：拜占庭缺陷（Byzantine Fault）：任何观察者从不同角度看，表现出不同症状的缺陷。
>
> **定义2：**拜占庭故障（Byzantine Failure）：在需要共识的系统中由于拜占庭缺陷导致丧失系统服务。

在分布式系统中，不是所有的缺陷或故障都能称作拜占庭缺陷或故障。像死机、丢消息等缺陷或故障不能算为拜占庭缺陷或故障。拜占庭缺陷或故障是最严重缺陷或故障，拜占庭缺陷有不可预测、任意性的缺陷，例如遭黑客破坏，中木马的服务器就是一个拜占庭服务器。

在一个有拜占庭缺陷存在的分布式系统中，所有的进程都有一个初始值。在这种情况下，共识问题（Consensus Problem），就是要寻找一个算法和协议，使得该协议满足以下三个属性。

1. **一致性（Agreement）**：所有的非缺陷进程都必须同意同一个值。
2. **正确性（Validity）**：如果所有的非缺陷的进程有相同的初始值，那么所有非缺陷的进程所同意的值必须是同一个初始值。
3. **可结束性（Termination）**：每个非缺陷的进程必须最终确定一个值。

根据Fischer-Lynch-Paterson的理论，在异步通信的分布式系统中，只要有一个拜占庭缺陷的进程，就不可能找到一个共识算法，可同时满足上述要求的一致性、正确性和可结束性要求。在实际情况下，根据不同的假设条件，有很多不同的共识算法被设计出来。这些算法各有优势和局限。算法的假设条件有以下几种情况：

1. 故障模型：非拜占庭故障/拜占庭故障。
2. 通信类型：同步/异步。
3. 通信网络连接：节点间直连数。
4. 信息发送者身份：实名/匿名。
5. 通信通道稳定性：通道可靠/不可靠。
6. 消息认证性：认证消息/非认证消息。