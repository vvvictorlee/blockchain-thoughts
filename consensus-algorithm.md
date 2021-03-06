## 拜占庭容错共识算法总结

拜占庭容错共识算法分为两大类：基于区块链的共识算法、传统拜占庭容错算法(classic BFT)

基于区块链的拜占庭共识算法基于同步模型(synchronous)设计，安全性(safety)和可用性(liveness)均依赖于同步环境。

传统的拜占庭共识算法基于半同步模型(partial synchronous)设计，安全性(safety)在异步环境下也可得到保证，而可用性(liveness)则需要在同步环境下才可得到保证。

**注：同步环境指的是在消息传输延迟有一个上界(upper bound)**

以中本聪设计的比特币工作量证明算法(Nakamoto POW Consensus)为例，它假设新区快的传输延迟有一个上界，即 10 分钟，如果该同步假设不成立，会发生什么呢？来看一个极端的例子，假设整个比特币网络被分为 A、B 两个区域。A 和 B 两个网络分区内的区块、交易均能及时达到，而由 A(B) 分区产生的区块、交易需要花费 1 年的时间才能到达 B(A) 分区，显然，A 和 B 网络分区将会产生两条互不汇聚的两条链，由此安全性将得不到保证。

如果极端的网络分区发生在其它的拜占庭容错共识算法系统里，由于任何分区都不会同时得到 2f+1 个节点的响应，因此系统将会阻塞(网络分区均无法形成大多数)或某一分区运行其它分区阻塞(至多有一个网络分区形成大多数)，但是不会损害系统的安全性，只是影响了系统的可用性。

### PBFT

### Tendermint

### Casper

### Thunderella

Thunderella 核心思想是结合快速、异步的路径(asynchronous path - BFT)与缓慢的、同步的"回退(fall-back)"路径(仅仅当出错时，才会执行该条路径，区块链式共识算法)来实现状态机复制(SMR, state machine replication，也称共识算法)。

在系统中超大多数(super majority，这里 thunderella 选择是 3/4)节点是诚实的情况下，Thunderella 使用异步路径来快速达成共识，这称之为"乐观响应"(optimistic responsiveness)，但是只保证一致性，不保证可用性。当系统不可用时，执行回退路径，即由底层支撑的区块链式共识算法接管。

一个共识算法被认为具有"响应"特性，当且仅当输入到系统中诚实节点的请求/交易能够及时被确认，仅依赖于实际的网络延迟，而不是事先预知的网络延迟上界。

Thunderella 共识算法的简单描述如下：

- 存在一个领导者(leader/accelerator)。

- 交易被发送至 leader 节点处，leader 对交易进行签名，并赋予一个递增的序列号(可以理解成是一个包含很多交易的区块)，然后将签好名的交易发送给系统选择出的委员会成员。

- 委员会成员(committee members)确认来自 leader 处的签名交易，规则是每一个序列号最多一个交易，即诚实的委员会成员不会对相同序号不同交易进行确认。

- 如果一笔交易得到了超过 3/4 委员会成员的签名确认，则称该笔交易为 notarized。参与者可直接输出最长序列公证过的交易(notarized transactions)，所有的这些交易都是确认的。

Thunderella 算法假设 1/2 的委员会成员是诚实的(条件 W')，因此选择的确认阈值是 3/4。证明如下：假设存在两组超过 3/4 的委员会针对相同序列号不同交易的签名，那么 (>3/4)+(>3/4) - 1 > 2/4，此时系统中将会存在多于 1/2 的恶意委员会成员，与算法的假设相矛盾，得证。在条件 O = "leader 是诚实的，以及 3/4 的委员会成员是诚实"的条件下，该算法满足可用性以及乐观响应特性。但是值得注意的是，此时的共识算法在 W' 条件下是只满足安全性(safety)，却不满足可用性(liveness)，如果 leader 节点是恶意的，或者宕机，或者有超过 1/4 的恶意委员会成员故意不发消息，导致无法形成 3/4 大多数签名投票，协议就会中止运行，因此无法满足可用性。

为了克服上述问题，Thunderella 使用底层的区块链协议来解决异步路径中止的问题，
区块链协议在大多数节点诚实(条件W)的前提下，可实现安全性与可用性。简单来说，如果参与者注意到交易没有得到委员会确认，则将一些“证据”发送至底层区块链，然后系统进入到一个冷却器(cool-down)，在此期间，委员会不再接受确认来自 leader 发送的消息，但是其仍可以将自己目前所知的公证过(notarized)的交易进行广播。在冷却器结束以后，系统进入到“慢速模式(slow mode)”，交易在底层区块链中得到确认，接下来，可以使用区块链来轮换 leader，开启新一轮的乐观协议。

冷却期的存在是为了确保参与者在进入到“慢速模式”以前，对已经公证过的交易达成共识。**冷却期以区块数量为计量单位，是系统需要选取的一个安全参数，需能够保证公证过的交易能到达全网大部分节点。**

**证据的收集**  如果一个参与者注意到它的交易没有及时被 leader/committee 确认，则将该笔交易发送至底层区块链，leader 被设计成需要确认在区块链中看到的所有交易，因此当参与者观察到某笔交易在一定时间内(例如 n 个区块)没有被公证，则进入慢速模式。

**委员会选取**  到目前为止，构建的 thunderella 算法在 W 与 W' 条件同时成立的情况下(系统的参与者大多数是诚实的，与委员会成员大多数是诚实的)，能够保证安全性与可用性q





### Algorand

### Hot-Stuff

（待完成）
