### Factom 简介

对于添加永久的条目，Factom 有着最小的规则集。Factom 把大部分数据验证任务放到了客户端，在 Factom 这端强制验证的是 Factom 协议运行所需要的：Factoids (Factom 原生加密货币) 交易，Factoids 到 Entry Credits 的转换，且保证条目所需的花费被支付，支付成功之后，保证条目被记录在相应的链中。**注意：**Factom 其实也没有能力去验证用户记录的条目内容的有效性，它只负责记录。

如果 Factom 被用来记录真实房产契约的转移，那么 Factom 能做的也只有记录这个转移过程的发生，因为真实房产转移的规则是非常复杂的，例如，如果买方是外国人，农民或兼职居民，当地管辖区可能对财产有特殊要求。所以在这个例子中，一个加密签名不足以验证所有权转移的有效性，因此 Factom 被用来记录发生的过程而不是验证转移的有效性。

比特币矿工执行两个首要的任务。第一，他们解决双花问题。当看到两笔花费同一个输出的交易时，他们来决定哪一笔被允许进入主链。第二，矿工执行审计，既然比特币矿工只包含有效的交易，所以被包含进区块链的交易可以被假定已经被审计。

Factom 将比特币矿工扮演的两个角色分割为两个任务：1. 记录条目到一个最终的顺序；2. 审计条目的有效性。

1. 
