# 介绍
Paxos 算法是第一个被证明完备的分布式系统共识算法。共识算法的作用是让分布式系统中的多个节点之间对某个提案（Proposal）达成一致的看法。提案的含义在分布式系统中十分宽泛，像哪一个节点是 Leader 节点、多个事件发生的顺序等等都可以是一个提案。

兰伯特当时提出的 Paxos 算法主要包含 2 个部分:
- **Basic Paxos 算法**：描述的是多节点之间如何就某个值(提案 Value)达成共识。
- **Multi-Paxos 思想**：描述的是执行多个 Basic Paxos 实例，就一系列值达成共识。Multi-Paxos 说白了就是执行多次 Basic Paxos ，核心还是 Basic Paxos 。

由于 Paxos 算法在国际上被公认的非常难以理解和实现，因此不断有人尝试简化这一算法。到了 2013 年才诞生了一个比 Paxos 算法更易理解和实现的共识算法—[Raft 算法open in new window](https://javaguide.cn/distributed-system/theorem&algorithm&protocol/raft-algorithm.html) 。更具体点来说，Raft 是 Multi-Paxos 的一个变种，其简化了 Multi-Paxos 的思想，变得更容易被理解以及工程实现。

针对没有恶意节点的情况，除了 Raft 算法之外，当前最常用的一些共识算法比如 **ZAB 协议**、 **Fast Paxos** 算法都是基于 Paxos 算法改进的。

针对存在恶意节点的情况，一般使用的是 **工作量证明（POW，Proof-of-Work）**、 **权益证明（PoS，Proof-of-Stake ）** 等共识算法。这类共识算法最典型的应用就是区块链，就比如说前段时间以太坊官方宣布其共识机制正在从工作量证明(PoW)转变为权益证明(PoS)。

区块链系统使用的共识算法需要解决的核心问题是 **拜占庭将军问题** ，这和我们日常接触到的 ZooKeeper、Etcd、Consul 等分布式中间件不太一样。

**下面我们来对 Paxos 算法的定义做一个总结：**
- Paxos 算法是兰伯特在 **1990** 年提出了一种分布式系统共识算法。
- 兰伯特当时提出的 Paxos 算法主要包含 2 个部分: Basic Paxos 算法和 Multi-Paxos 思想。
- Raft 算法、ZAB 协议、 Fast Paxos 算法都是基于 Paxos 算法改进而来。

# Basic Paxos 算法
Basic Paxos 中存在 3 个重要的角色：
1. **提议者（Proposer）**：也可以叫做协调者（coordinator），提议者负责接受客户端的请求并发起提案。提案信息通常包括提案编号 (Proposal ID) 和提议的值 (Value)。
2. **接受者（Acceptor）**：也可以叫做投票员（voter），负责对提议者的提案进行投票，同时需要记住自己的投票历史；
3. **学习者（Learner）**：如果有超过半数接受者就某个提议达成了共识，那么学习者就需要接受这个提议，并就该提议作出运算，然后将运算结果返回给客户端。
![[Pasted image 20240722223055.png]]

为了减少实现该算法所需的节点数，一个节点可以身兼多个角色。并且，一个提案被选定需要被半数以上的 Acceptor 接受。这样的话，Basic Paxos 算法还具备容错性，在少于一半的节点出现故障时，集群仍能正常工作。

# Multi Paxos 思想
Basic Paxos 算法的仅能就单个值达成共识，为了能够对一系列的值达成共识，我们需要用到 Multi Paxos 思想。

⚠️**注意**：Multi-Paxos 只是一种思想，这种思想的核心就是通过多个 Basic Paxos 实例就一系列值达成共识。也就是说，Basic Paxos 是 Multi-Paxos 思想的核心，Multi-Paxos 就是多执行几次 Basic Paxos。

由于兰伯特提到的 Multi-Paxos 思想缺少代码实现的必要细节(比如怎么选举领导者)，所以在理解和实现上比较困难。

不过，也不需要担心，我们并不需要自己实现基于 Multi-Paxos 思想的共识算法，业界已经有了比较出名的实现。像 Raft 算法就是 Multi-Paxos 的一个变种，其简化了 Multi-Paxos 的思想，变得更容易被理解以及工程实现，实际项目中可以优先考虑 Raft 算法。