##### Consensus algorithms and total order broadcast
最著名的容错共识算法是 **视图戳复制（VSR, Viewstamped Replication）**，例如 Paxos，Raft 和 Zab。

大多数这些算法实际上并不直接使用这里描述的形式化模型（提议与决定单个值，并满足一致同意、完整性、有效性和终止属性）。取而代之的是，它们决定了值的 **顺序（sequence）**，这使它们成为全序广播算法。

全序广播要求将消息按照相同的顺序，恰好传递一次，准确传送到所有节点。如果仔细思考，这相当于进行了几轮共识：在每一轮中，节点提议下一条要发送的消息，然后决定在全序中下一条要发送的消息。

全序广播相当于重复进行多轮共识（每次共识决定与一次消息传递相对应）：
* 由于 **一致同意** 属性，所有节点决定以相同的顺序传递相同的消息。
* 由于 **完整性** 属性，消息不会重复。
* 由于 **有效性** 属性，消息不会被损坏，也不能凭空编造。
* 由于 **终止** 属性，消息不会丢失。

Raft 和 Zab 直接实现了全序广播，因为这样做比重复 **一次一值（one value a time）** 的共识更高效。在 Paxos 的情况下，这种优化被称为 Multi-Paxos。


##### Single-leader replication and consensus
单主复制情况下：将所有的写入操作都交给主库，并以相同的顺序将它们应用到从库，从而使副本保持在最新状态。本质上也是一个全序广播。

单主复制不需要共识，主库是由运维人员手动选择和配置的，只有一个节点被允许接受写入（即决定写入复制日志的顺序），如果该节点发生故障，则系统将无法写入，直到运维手动配置其他节点作为主库。这样的系统在实践中可以表现良好，但它无法满足共识的 **终止** 属性，因为它需要人为干预才能取得 **进展**。

一些数据库会自动执行领导者选举和故障切换，如果旧主库失效，会提拔一个从库为新主库。这使我们向容错的全序广播更进一步，从而达成共识。