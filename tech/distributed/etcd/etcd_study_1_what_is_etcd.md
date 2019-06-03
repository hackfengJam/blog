# What is etcd ?

### What is etcd ? 

以下内容，译自 [etcd 官网](https://etcd.io/)

--- 

#### 目录

- [1. Project](#1-Project)
- [2. Technical overview](#2-Technical-overview)
- [3. Adopters](#3-Adopters)


#### 1. Project

1.1 etcd 是一个强一致的的分布式 键-值 存储，它提供了一种可靠的方式来存储需要由分布式系统机器或集群访问的数据
    它优雅地处理网络分区期间的领导者选举，并且可以容忍机器故障，即使在领导者节点中也是如此。

<details>
<summary>原文</summary>

etcd is a strongly consistent, distributed key-value store 
that provides a reliable way to store data 
that needs to be accessed by a distributed system or cluster of machines. 
It gracefully handles leader elections during network partitions and can tolerate machine failure, even in the leader node.

</details>


1.2 从简单的 Web应用程序 到 Kubernetes，任何复杂的应用程序都可以读取数据并将数据写入 etcd 。


<details>
<summary>原文</summary>

Applications of any complexity, from a simple web app to [Kubernetes](https://kubernetes.io/), can read data from and write data into etcd.

</details>

1.3 您的应用程序可以读取和写入 etcd 中的数据。
    一个简单的用例是将 etcd 中的数据库连接详细信息或功能标记存储为键值对。可以监视这些值，允许您的应用在更改时重新配置。
    高级用法利用 etcd 的一致性保证来实现数据库领导者选举或跨工作集群执行分布式锁定。


<details>
<summary>原文</summary>

Your applications can read from and write data into etcd . 
A simple use case is storing database connection details or feature flags in etcd as key-value pairs. 
These values can be watched, allowing your app to reconfigure itself when they change. 
Advanced uses take advantage of etcd ’s consistency guarantees 
to implement database leader elections or perform distributed locking across a cluster of workers.

</details>

1.4 etcd 是开源的，可以在 GitHub 上获得，并由 Cloud Native Computing Foundation 支持。

<details>
<summary>原文</summary>

etcd is open source, available [on GitHub](https://github.com/etcd-io/etcd), and backed by the [Cloud Native Computing Foundation](https://www.cncf.io/).

</details>

#### 2. Technical overview

2.1 etcd 是用 Go 编写的，它具有出色的跨平台支持，小型二进制文件和背后的优秀社区。
    etcd 机器之间的通信通过 Raft 一致性算法处理。

<details>
<summary>原文</summary>

etcd is written in [Go](https://golang.org/), which has excellent cross-platform support, small binaries and a great community behind it. 
Communication between etcd machines is handled via the Raft consensus algorithm.

</details>

2.2 来自 etcd 领导者的延迟是最重要的跟踪指标，内置仪表板具有专用于此的视图。
    在我们的测试中，严重的延迟会在集群内引入不稳定性，因为 Raft 的速度与大多数机器中的最慢机器一样快（/因为Raft只有大多数机器中最慢的机器速度）。
    您可以通过正确调整集群来缓解此问题。etcd 已经在具有高度可变网络的云提供商上进行了预调整。

<details>
<summary>原文</summary>

Latency from the etcd leader is the most important metric 
to track and the built-in dashboard has a view dedicated to this. 
In our testing, severe latency will introduce instability within the cluster 
because Raft is only as fast as the slowest machine in the majority. 
You can mitigate this issue by properly tuning the cluster. 
etcd has been pre-tuned on cloud providers with highly variable networks.

</details>

#### 3. Adopters

<details>
<summary>Adopters List</summary>

3.1 [Kubernetes (K8s)](https://kubernetes.io/)

etcd 是服务发现的后端，存储集群状态和配置

<details>
<summary>原文</summary>

etcd is the backend for service discovery and stores cluster state and configuration

</details>

3.2 [Rook](https://rook.io/)

etcd 作为 Rook 的编排引擎

<details>
<summary>原文</summary>

etcd serves as the orchestration engine for Rook

</details>

3.3 [CoreDNS](https://coredns.io/)

CoreDNS 使用 etcd 作为可选后端

<details>
<summary>原文</summary>

CoreDNS uses etcd as an optional backend

</details>


3.4 [M3](https://eng.uber.com/m3/)

M3 是 Uber 创建的 Prometheus 大型指标平台，使用 etcd 进行规则存储和其他功能

<details>
<summary>原文</summary>

M3, a large-scale metrics platform for Prometheus created by Uber, uses etcd for rule storage and other functions

</details>

3.5 [OpenStack](https://www.openstack.org/)

OpenStack 支持 etcd 作为配置存储，分布式密钥锁定等的可选提供者

<details>
<summary>原文</summary>

OpenStack supports etcd as an optional provider of configuration storage, distributed key locking, and more

</details>

3.6 [Patroni](https://github.com/zalando/patroni)

带有ZooKeeper，etcd 或 Consul 的 PostgreSQL HA模板

<details>
<summary>原文</summary>

A template for PostgreSQL HA with ZooKeeper, etcd, or Consul

</details>

3.7 [Trillian](https://github.com/google/trillian/)

由 Google 创建的透明，高度可扩展且可加密验证的数据存储

<details>
<summary>原文</summary>

A transparent, highly scalable and cryptographically verifiable data store, created by Google

</details>
</details>

## 感谢

[etcd 官网](https://etcd.io/)

[etcd 官方文档](https://etcd.io/docs/v3.3.12/)
