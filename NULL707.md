---
timezone: UTC+8
---

> 请在上边的 timezone 添加你的当地时区(UTC)，这会有助于你的打卡状态的自动化更新，如果没有添加，默认为北京时间 UTC+8 时区


# 你的名字

1. 自我介绍
web3小白
2. 你认为你会完成本次残酷学习吗？
会
3. 你的联系方式（推荐 Telegram）
nullqaq707

## Notes

<!-- Content_START -->
### 2025.03.15
## 零知识证明
零知识证明是指向他人证明某个命题为真，但又不透露“该命题为真”之外的任何信息。

零知识证明具备以下性质：

- 完备性（Completeness）：若命题为真，任何证明者可以向验证者提出令人信服的证据，即“真的可被验证”。
- 可靠性（Soundness）：若命题为假，则不存在不诚实的证明者能骗过验证者，即“假的会被发现”。
- 零知识性（Zero-knowledge）：证明某个命题为真，但又不透露“该命题为真”之外的其他任何信息。

工程实践上我们还要求零知识证明的算法拥有以下性质：

- 简洁性（Succinctness）[1]：证明很小且验证速度快。
- 零知识（Zero Knowledge）：可以隐藏计算的输入信息。

ZK-Rollup 会周期性向主网上传 3 种数据：

- 状态根：状态根可以快速确认 Layer 2 小账本的内容是否被篡改。
- 交易数据：经过压缩和聚合的交易数据，例如将多个交易合并为一批次的状态变化结果。通过使用零知识证明保证交易的安全性，可以舍弃一些不必要的信息，例如前面提到的“用户签名”。
- 有效性证明：即零知识证明，让 Layer 1 的智能合约在经过简单验证后，就能确认交易已经被正确执行。

## ZK-Rollup 的基础组件

- KZG 多项式承诺：因两个多项式最多拥有 n2 个交点，而定义域却存在极多的点，那么我们只需要检查有限的若干次，就能确信对方确实以正确的多项式进行了计算。若将信息编码在多项式中，则靠多次确认多项式在特定点上的输出结果，即可确认交易已被正确验证（确认过程原本是需要交互的，但可用其他方法变为非交互式）。
- 哈希算法：能将任意长度的数据映射为固定长度的哈希值，用于压缩证明。
- 椭圆曲线加密：可以将椭圆曲线上的两个点用难以预测的方式映射起来，用于构建证明系统。可用来进行一些复杂的证明，比如在不公开哈希值的情况下证明两个哈希相等。
- 随机数等其它组件：用于随机数来确认起始需要检查的点，并用类似“上一个哈希影响下一个哈希”的方式确认一连串需要检查的点，以确保检查点的随机性与非交互性。

目前零知识证明主要有两种技术路线，SNARK 与 STARK。SNARK 出现更早，更加成熟，目前被更多的项目方采用。
### 2025.03.14
## 压缩方法

以 OP-Rollup 的为例，我们要向 Layer 1 上传一段时间内的所有交易详情，如果不对这部分数据进行高度压缩，那分担负载的效果就非常小了。我们以单笔交易为例，它身上其实有许多可改进的空间。

比如一笔常见的转账交易，它的原数交易数据可能是以下这样的：

4232f4610000000000000000000000007ea2be2df7ba6e54b1a9c70676f668455e329d29000000000000000000000000d548a5e31de2b4c2681a58a3be5302abcae4bc5700000000000000000000000000000000000000000000000000000000000186a0

（Method ID / 填充的 0 / 代币合约地址 / 填充的 0 / 收款的账户地址 / 填充的 0 / 提币数量）
原始交易数据可以通过以下手段压缩：

- 用科学计数法把转账数量压缩成 64 位数据，并删除不必要的 0。（数量的精度会略微下降，但实践中影响不大）
  
- 调用的方法如果很常见，可以删除所调用的 Method ID，因为如“转账一笔 ERC20 [3] 代币”的交易，可以通过交易内容的特征推测
  
- 常用行为设置绿色通道（Helper ID）：大部分发送代币的行为都是如 USDC、WETH 等常用代币，可以用更短的一个 Helper ID，来表示调用方法是“发送”，发送的代币是“USDC”这两个信息。
  
- 登记一个“电话簿”，记录收款人地址，将 40 位的地址压缩为第 XXX 页的第 X 个地址。

- 如果发送的是 ETH，连 Helper ID 都可以省掉。
 

最终我们需要上传至 Layer 1 的数据从一段非常长的信息

4232f4610000000000000000000000007ea2be2df7ba6e54b1a9c70676f668455e329d29000000000000000000000000d548a5e31de2b4c2681a58a3be5302abcae4bc5700000000000000000000000000000000000000000000000000000000000186a0

变为了

059c570186a0

（收款账户“电话簿”编号 / 提币数量）

## Optimistic Rollup

Optimistic Rollup 是“乐观的打包”，它假设绝大部分的参与者都是诚实的，允许一批数量较少的验证者节点（Validator），对交易进行收集、排序、验证。同时还设置了挑战者的角色（Challenger），其职责是监督验证者提交的信息是否诚实。

OP-Rollup 会定期向主网上传两种数据：

- 状态根（State Root）: 状态根可以快速确认 Layer 2 小账本的内容是否被篡改。

- 压缩后的全部交易数据：包含各种交易细节，比如交易附带的“用户签名”。

虽然上传了近段时间的全部交易详情，但以太坊主网并不负责直接验证这些交易，只起到一个公示的作用。

与 Plasma 类似，OP-Rollup 也使用默克尔树的形式保存了一个“小账本”，记录了全体账户的所有状态（账户余额）。如果我们相信目前的交易验证者（Validator）都是诚实的，那么状态根能快速确认当前 Layer 2 的小账本记录的内容是否被篡改，确保安全性。

反之，假如我们对目前的交易合法性产生质疑，任何第三方都可以在主网获取最近一段时间内所有交易的副本，重新验证之后，将自己验证的结果与 Layer 2 的小账本的记录做对比，确认小账本上的记录均为合法。如果发现作恶，挑战者就可以在以太坊一层提交欺诈证明来改写二层的状态。挑战成功后，不诚实的验证者将受到惩罚，挑战者将获得奖励。同时，受影响的交易将被回滚，进行重新验证。

这个过程中，负责监督的挑战者（Challenger）是直接与 Layer 1 的智能合约交互的，一层对二层的状态有着最终裁决权。

这种设计之下，即使只有一个诚实的挑战者，也足以确保整个 Layer 2 的交易安全。不过代价是 OP-Rollup 必须提供一个退出窗口期，让挑战者有时间去检验并提交欺诈证明，因此使用官方桥从 OP-Rollup 网络提款往往需要 7 - 14 天的等待期。


### 2025.03.13
## 状态通道

假设 Alice 经常在一家咖啡店消费，如果每次买一杯 5 美元的咖啡，都需要支付 0.5 美元的手续费，这也太痛苦了。那么假如 Alice 和咖啡店能达成共识，每次买咖啡时付给咖啡店一张签了名的欠条，一段时间后咖啡店攒了足够多的欠条，将欠条算好总金额一次性兑现，这样交易成本就可以大幅降低，对双方都有利。这种思路就是最早的二层网络，也就是状态通道的原理。

状态通道使用了多签技术，允许两个个体之间提前存入一笔资金锁定在智能合约中，建立一个内部通道，然后双方可以在通道内进行多笔小额转账，速度极快，成本极低，再在一段时间后用转账证明一次性提款。状态通道也是比特币的 Lightning Network（闪电网络），以太坊的 Raiden Network（雷电网络）背后的底层技术。

## 侧链

侧链可以理解为一条相对独立的区块链，它们往往采用与主链（一般是以太坊）类似的架构，方便主链上的项目迁移至侧链。

我们可以在主链的智能合约内锁定一定量的资产，然后在侧链上铸造等量资产，实现“原子交换”。用这种方式将资产存入侧链，在侧链上进行各种交易，然后在必要时转移回主链。

侧链是否属于 Layer 2 存在一定争议，因为侧链虽然受主链的影响，但不继承主链的安全性这一点存在很大隐患。

侧链安全性由自身的共识节点负责，因此侧链本身的安全性，以及侧链与主链之间的通讯桥都可能出现问题。根据木桶原理，安全性取决于所有环节里最短的木板，侧链出现问题会影响整个生态的安全。

## Plasma

Plasma 会在 Layer 1 用智能合约锁住一笔资金，然后依赖一个节点运营商在 Layer 2 进行交易。运营商以默克尔树（Merkle Tree，又名：哈希树）的形式保存一个小账本，记录了 Layer 2 账户的状态信息，同时负责保存并公开所有的交易记录。

Plasma 运营商需要定期向 Layer 1 上传默克尔树的根证明，即“状态根”（State Root），这样所有人都能检查 Layer 2 的账本是否被篡改。

此外，Plasma 还支持链中链的特性，即不向 Layer 1 定期提交根证明，而是向另一条 Plasma 链定期提交根证明，如此一层叠一层，形成“巢状区块链”（Nested Blockchain）的形态。

## Rollup

Rollup 意为“打包”，也就是将一段时间内发生的交易先进行压缩，再进行打包，然后周期性地上传至主网。目前主流的 Rollup 方案分为两大路线，分别为 Optimistic Rollup（乐观的 Rollup）与 Zero-knowledge Rollup（零知识证明的 Rollup）。

- Optimistic Rollup：OP-Rollup 是将一段时间内的所有交易细节全部压缩打包，定期发送至 Layer 1。这种机制乐观地相信大部分交易都是诚实的，继承了 Plasma 挑战期和欺诈证明机制。
- Zero-knowledge Rollup：ZK-Rollup 一般是将一段时间内的交易计算完成后，将状态变化的结果压缩打包，并附上交易已经在 Layer 2 被正确执行的零知识证明，定期发送至 Layer 1。用零知识证明代替监督者，依赖数学而非验证者（Rely on Math, not Validators）。

Rollup 的二层解决方案，略微降低了去中心化程度，但获得了以下优点：

- 大幅分担了 Layer 1 层的计算负担，有效实现扩容；
- 压缩交易数据，节省了 Layer 1 的存储资源；
- 向 Layer 1 上传必要的关键信息，且一层对二层的状态有最终裁决权，从而最大程度地继承了 Layer 1 的安全性
### 2025.03.12
## 区块链提高性能的思路

- 增大单个区块的大小，容纳更多的交易

这样做会引起区块账本的快速膨胀，参与验证的机器性能要求越来越高，提高了参与门槛，导致整个网络去中心化程度和安全性渐渐降低。从 BTC 分叉出来的 BCH（Bitcoin Cash） 将区块大小从 1MB 提升至 32MB，BSV（Bitcoin Satoshi's Vision） 则是更激进地取消了区块大小上限，允许无限多的信息融入一个区块。

- 降低出块的时间，追求一定时间内出更多的块来处理更多的交易

这样对节点的网络条件提出了更高要求，提高了参与门槛。并且影响了全网数据同步的稳定性，因为物理上相隔较远的节点集群容易对最新的区块产生分歧，导致分叉。分叉链总需要竞争出新的最长链，抛弃其中的一条分支，导致过去一段时间内的许多交易被重写，这就是“区块重组”现象，Polygon 在 2023 年发生过 157 个区块的重组事件。

- 使用数量更少的超级节点通讯

超级节点的性能更强大，网络带宽更好更稳定，因此彼此之间能实现超高速的通讯，但这显然降低了去中心化程度。如 Fanton 有 51 个共识节点，BSC、EOS、TRON 则仅有 21 个超级节点。

- 用特殊的共识机制提升性能

共识机制决定了全网节点对出块方式如何达成共识，一些特殊的机制也许可以提高出块速度，但共识机制越复杂，就对机器性能要求越高，也更容易出现单点故障导致整个系统出错。如 Solana，全网节点依赖随机选出的单个 Leader 节点来协调，因此获得了极高的理论 TPS 上限，但对节点性能要求变得非常高，并多次发生全网宕机的安全性事故。
## 不可能三角
区块链不可能三角（Blockchain Trilemma）是指在设计区块链系统时，存在三个目标之间的矛盾，这三个目标分别是去中心化、安全性和可扩展性（性能）。

- 去中心化（Decentralization）：指的是在区块链系统中，所有的节点都具有相同的权力，没有单一的中心化权威节点进行控制。这个目标是区块链的核心特性，也是保证系统安全性和抗攻击性的基础。
- 安全性（Security）：指的是在区块链系统中，保证交易的真实性、完整性、不可篡改性和抗攻击性等方面的安全。这个目标是区块链系统的重要保障，也是确保系统可靠性和信任度的基础。
- 可扩展性（Scalability）：可扩展性即性能，指的是在区块链系统中，支持足够大量的交易、节点和用户等系统扩展。这个目标是区块链系统的重要需求，也是确保系统能够满足现实需求的基础。


追求去中心化程度和安全性，采用更多的节点和更公平的出块方式，值得信赖。但为了允许低性能节点参与验证，协调全球网络延迟，导致每秒可处理的交易数较低，牺牲了性能。

    代表区块链：BTC、ETH 追求了极致的安全可靠和去中心化，但处理交易的速度较低，BTC 约为 7 笔/秒，ETH 约为 10 - 20 笔/秒。

追求安全性和可扩展性（即性能），往往采用少数超级节点进行通讯，超级节点拥有更强的性能和更好的网络环境，彼此之间能实现超高速的通讯。但参与门槛过高，牺牲了去中心化程度。

    代表区块链：BSC、EOS、TRON 等区块链采用了少数高性能节点维护网络，仅有 21 个超级节点进行记账。

追求可扩展性（即性能）和去中心化程度，为保证去中心化采用了较多验证节点，为了追求性能提高了出块速度，或采用了特殊的共识机制。但提高出块速度容易导致大规模区块重组，更复杂的共识机制容易导致全网宕机等安全事故，牺牲了安全性。

    代表区块链：Polygon 在 2023 年发生了 157 个区块的大规模重组；Solana 多次出现全网宕机的事故。

### 2025.03.11
## Layer2 是什么？
Layer2 解决方案是建立在现有区块链之上的技术，旨在提高其扩展性和效率，而不改变其底层结构。

随着区块链的普及，交易量激增，导致网络拥堵和交易费用上升。Layer2 解决方案旨在解决这些问题，提供更快、更便宜的交易。

## Layer2 的种类

- 状态通道（State Channels）： 允许两方在链下进行多次交易，然后将结果提交到主链。
- 侧链（Sidechains）： 是与主链并行运行的独立链，允许资产和数据在两者之间转移。
- Plasma： 是一个框架，允许创建多个子链，每个子链都与主链相互作用。
- Rollups： 通过在链下处理交易并将其结果打包到主链来提高效率。

## 跨链桥 是什么？

跨链桥是一种允许在不同的区块链之间转移资产和数据的技术。随着多个区块链平台的出现，互操作性成为一个关键问题。跨链桥提供了一种方法，使资产和数据能够在不同的链之间自由流动。

## 跨链桥 的种类

- 简单支付验证（SPV）桥： 通过验证另一个链上的交易来工作。
- 联邦桥： 由一组验证者管理，负责在两个链之间转移资产。
- TSS（阈值签名方案）桥： 使用多方计算来创建跨链交易的签名。

## Layer2 和跨链桥的重要性

- 扩展性： Layer2 解决方案提供了一种方法，使区块链能够处理更多的交易，满足日益增长的需求。
- 互操作性： 跨链桥确保了区块链的互操作性，使得不同的链可以互相通信和互相支持。
- 创新： 这两种技术为开发者提供了更多的工具和机会，推动了区块链领域的创新。
### 2025.03.10

## Layer1 是什么？
Layer1 是区块链的第一层，是所有区块链活动发生的地方，包括交易的处理、验证和记录。

共识机制是 Layer1 的核心。它就像是城市中的交通信号灯，协调着所有数据流动和交易处理的顺序。

Layer1 的网络结构决定了信息在区块链中的传播方式。它就像是城市的道路系统，决定了数据如何高效、安全地流动。有的网络设计重视速度和效率，而有的则更加强调安全和去中心化。

## Layer1 的实际应用

### 公链：开放的数字大道

公链是 Layer1 技术的最广泛应用形式，代表着区块链技术的核心特质：开放性、去中心化和透明性。公链（如比特币和以太坊）就像是开放给所有人的大道。在这些网络中，任何人都可以参与验证交易，加强了网络的去中心化和透明性。

**公链的代表**

比特币（Bitcoin）：作为第一个和最著名的公链，比特币引领了数字货币的革命。

以太坊（Ethereum）：不仅作为货币，还通过其智能合约功能扩展了区块链的应用范围。

莱特币（Litecoin）、卡尔达诺（Cardano）等：这些公链通过不同的技术和特性，进一步丰富了公链生态系统。
### 联盟链：协作的桥梁
联盟链是一种介于公链和私链之间的区块链形式，它在特定的组织群体之间建立起了信任和合作的桥梁。

**联盟链的应用**

R3 Corda：金融服务行业中的联盟链，旨在提高银行间交易的效率。

Hyperledger Fabric：由Linux基金会发起，用于企业级区块链解决方案。

Quorum：由摩根大通开发，是一个基于以太坊的企业焦点的区块链平台。

**联盟链的优势**

高效率和低成本：较少的节点数量意味着更快的交易处理速度和更低的运行成本。

更好的隐私保护：适用于需要保护敏感数据的场景。
### 私链：专有的数字领地
私链更加封闭，像是个人的秘密花园，只有拥有者才能访问和控制。

私链在 Layer1 的世界里就像是被高墙围绕的私人领地。在这种类型的区块链中，访问和参与都受到严格的控制。私链主要用于特定的组织或企业内部，提供了一种既安全又高效的数据管理方式。

**私链的特点**
-  受限访问：只有经过授权的个人或实体才能参与网络。

   ---这种受限的参与方式保障了网络的私密性和专属性。
-  更高的效率：参与者数量有限。

   ---私链通常能够更快速地处理交易和数据。
-  定制化的控制：组织可以根据自己的需要定制网络规则和协议。

   ---这种灵活性使私链成为特定需求的理想选择。
  
**私链的应用场景**
- 企业数据管理：利用私链安全地存储和管理敏感数据。

  ---特别适用于对数据保密性有高要求的企业。
- 内部记录保持: 记录内部交易和日常操作。

  ---确保数据不被外界访问，增加业务操作的透明度。
- 供应链跟踪：虽然公链和联盟链在此领域也有应用，但私链提供了额外的安全性。

  ---对于需要严格数据控制和隐私的企业尤其重要。
- 私链的挑战：去中心化程度较低，控制集中在单一实体手中，降低了去中心化程度。

  ---这可能影响网络的透明度和公正性。
- 安全性考量：尽管控制严格，但私链可能更容易受到内部威胁的影响。

  ---需要强化内部安全措施和协议，以防止内部风险。
### 2025.03.09
### 四类共识算法

工作量证明(Proof of Work, PoW)类的共识算法；

Po*的凭证类共识算法；

拜占庭容错(Byzantine Fault Tolerance, BFT)类算法；

结合可信执行环境的共识算法。

**POW共识算法**：这类共识算法的核心思想实际是所有节点竞争记账权，而对于每一批次的记账(或者说，挖出一个区块)都赋予一个 难题，要求只有能够解出这个难题的节点挖出的区块才是有效的。同时，所有节点都不断地通过试图解决难题来产生自己的区块并将自己的区块追加在现有的区块链之后，但全网络中只有最长的链才被认为是合法且正确的。它采用的 难题 具有难以解答，但很容易验证答案的正确性的特点，同时这些难题的 难度，或者说全网节点平均解出一个难题所消耗时间，是可以很方便地通过调整难题中的部分参数来进行控制的，因此它可以很好地控制链增长的速度。通过控制区块链的增长速度，它还保证了若有一个节点成功解决难题完成了出块，该区块能够以(与其他节点解决难题速度相比)更快的速度在全部节点之间传播，并且得到其他节点的验证的特性，一旦解决了 难题 并生成了区块，就会在很快的时间内告知全网其他节点，而全网的其他节点在验证完毕该区块后，便会基于该区块继续解下一个难题以生成后续的区块。PoW 类算法给参与节点带来的计算开销，除了延续区块链生长外无任何其他意义，却需要耗费巨大的能源，并且该开销会随着参与的节点数目的上升而上升，是对能源的巨大浪费。


**Po*的凭证类共识算法**：根据每个节点的某些属性(拥有的币数、持币时间、可贡献的计算资源、声誉等)，定义每个节点进行出块的难度或优先级，并且取凭证排序最优的节点，或是取凭证最高的小部分节点进行加权随机抽取某一节点，进行下一段时间的记账出块。这种类型的共识算法在一定程度上降低了整体的出块开销，同时能够有选择地分配出块资源，即可根据应用场景选择 凭证 的获取来源，是一个较大的改进。然而，凭证的引入提高了算法的中心化程度，一定程度上有悖于区块链 去中心化 的思想，且多数该类型的算法都未经过大规模的正确性验证实验，部分该类算法的矿工激励不够明确，节点缺乏参与该类共识的动力。

**POW共识算法**：采取了不同的思路，它希望所有节点协同工作，通过协商的方式来产生能被所有(诚实)节点认可的区块。BFT 类共识算法一般都会定期选出一个领导者，由领导者来接收并排序区块链系统中的交易，领导者产生区块并递交给所有其他节点对区块进行验证，进而其他节点“举手”表决时接受或拒绝该领导者的提议。如果大部分节点认为当前领导者存在问题，这些节点也可以通过多轮的投票协商过程将现有领导者推翻，再以某种预先定好的协议协商产生出新的领导者节点。BFT 类算法一般都有完备的安全性证明，能在算法流程上保证在群体中恶意节点数量不超过三分之一时，诚实节点的账本保持一致。然而，这类算法的协商轮次也很多，协商的通信开销也比较大，导致这类算法普遍不适用于节点数目较大的系统。业界普遍认为，BFT 算法所能承受的最大节点数目不超过100。

**结合可信执行环境的共识算法**：软硬件结合的共识算法，可信执行环境是一类能够保证在该类环境中执行的操作绝对安全可信、无法被外界干预修改的运行环境，它与设备上的普通操作系统(Rich OS)并存，并且能给Rich OS提供安全服务。可信执行环境所能够访问的软硬件资源是与Rich OS完全分离的，从而保证了可信执行环境的安全性。利用可信执行环境，可以对区块链系统中参与共识的节点进行限制，很大程度上可以消除恶意节点的不规范或恶意操作，从而能够减少共识算法在设计时需要考虑的异常场景，一般来说能够大幅提升共识算法的性能。

### 智能合约

智能合约是一种在满足一定条件时，就自动执行的计算机程序。一个基于区块链的智能合约需要包括事务处理机制、数据存储机制以及完备的状态机，用于接收和处理各种条件。并且事务的触发、处理及数据保存都必须在链上进行。当满足触发条件后，智能合约即会根据预设逻辑，读取相应数据并进行计算，最后将计算结果永久保存在链式结构中。
### 2025.03.08
区块链(blockchain) 是一种数据以 区块(block) 为单位产生和存储，并按照时间顺序首尾相连形成 链式(chain) 结构，同时通过密码学保证不可篡改、不可伪造及数据传输访问安全的去中心化分布式账本。区块链中所谓的账本，其作用和现实生活中的账本基本一致，按照一定的格式记录流水等交易信息。特别是在各种数字货币中，交易内容就是各种转账信息。只是随着区块链的发展，记录的交易内容由各种转账记录扩展至各个领域的数据。比如，在供应链溯源应用中，区块中记录了供应链各个环节中物品所处的责任方、位置等信息。

区块是链式结构的基本数据单元，聚合了所有交易相关信息，主要包含区块头和区块主体两部分。区块头主要由父区块哈希值(Previous Hash)、时间戳(Timestamp)、默克尔树根(Merkle TreeRoot)等信息构成；区块主体一般包含一串交易的列表。每个区块中的区块头所保存的父区块的哈希值，便唯一地指定了该区块的父区块，在区块间构成了连接关系，从而组成了区块链的基本数据结构。

**内容改变的快速检测**

在区块链中，我们只需要保留对自己有用的交易信息，删除或者在其他设备备份其余交易信息。如果需要验证交易内容，只需验证默克尔树即可。若根哈希验证不通过，则验证两个叶子节点，再验证其中哈希验证不通过的节点的叶子节点，最终可以准确识别被篡改的交易。

**数字签名**

在密码学领域，一套数字签名算法一般包含签名和验签两种运算，数据经过签名后，非常容易验证完整性，并且不可抵赖。只需要使用配套的验签方法验证即可，不必像传统物理签名一样需要专业手段鉴别。数字签名通常采用非对称加密算法，即每个节点需要一对私钥、公钥密钥对。所谓私钥即只有本人可以拥有的密钥，签名时需要使用私钥。不同的私钥对同一段数据的签名是完全不同的，类似物理签名的字迹。数字签名一般作为额外信息附加在原消息中，以此证明消息发送者的身份。公钥即所有人都可以获取的密钥，验签时需要使用公钥。因为公钥人人可以获取，所以所有节点均可以校验身份的合法性。
### 2025.03.07
**山寨币**：最初用于指任何不是比特币的加密货币，现在山寨币可能指任何市值相对较小的新加密货币。

**Arbitrum**：一个以太坊第二层的扩展解决方案，通过计算以太坊主网之外的交易，减少费用和网络堵塞。Arbitrum的滚动使用多轮欺骗证明来验证一致性的正确执行。

**燃烧**：召回NFT。例如，如果原定由1万枚NFT组成的收藏品只售出5千枚，开发团队可能会决定“燃烧”剩余五千枚NFT。

**Bridge**：桥接。一种允许独立的区块链通信的协议，使数据、代币和其他信息在系统之间转移。

**共识机制**：共识机制。区块链上的节点就交易或网络的状态达成的过程。如工作证明(POW)、股权证明(POS)。

**Defi**：去中心化金融缩短，正在公共区块链上构建无边界、无信任、点对点金融工具的生态系统，无需使用银行。DeFi应用程序被构建为开放和互连的，允许它们相互结合使用。

**DAO**：去中心化的自治组织，一个基于开源代码并由其用户管理的组织。DAO通常专注于一个特定的项目或任务，将传统企业的等级制度换写成区块链上的基准。

**Dapp**：去中心化应用，去中心化的应用程序，一种建立在区块链上的开源代码上的应用程序。Dapp独立于中心化的群体或人物而存在，通常通过奖励代币来激励用户维护它们。

**FOMO**：Fear Of Missing Out，失败，一般指未能失败行情而盲目跟风。

**分叉**：分叉，对一个区块链协议的改变。当这些变化较小时，将导致软分叉。当这些变化根本上时，将会导致硬分叉，形成一个具有不同规则的独立链。

**FUD**：Fear，Uncertainty，Doubt。恐惧、不确定、怀疑。围绕某项资产的消息，看似是负面的，但结果是错误的或吹得不成比例。

**ICO**：Initial Coin Invention，首次代币发行，向公众出售代币，便于为一个基于加密货币的项目募集资金，是一种众筹方式。

**IEO**：Initial Exchange Offers，首次交易所发行。与ICO概念类似，首次交易所发行是一种出售代币认购资金的方式，但监管力度加大。与ICO直接向公众出售代币不同，IEO是由现有的加密货币交易所管理。

**L1**：layer1，扩容方案，又称链上扩容，指在区块链底层协议上实现的扩容解决方案。

**L2**：layer2，扩容方案，又称链下扩容，指不改变区块链底层协议和基础规则，通过状态提高通道、侧链等方案交易处理速度。

**Merkle Root**：默克尔树根，是默克尔树的单一顶级哈希值。它验证了一个区块内的所有交易。

**Merkle Tree**：默克尔树，是区块链安全用于验证和汇总大型数据集的一种数据结构。

**ZK**：Zero-Knowledge Proof，零知识证明，是一种验证方法，其中“证明者”向“验证者”现实拥有秘密知识而不揭露敏感信息本身。可以确保公共区块链上的数据隐私和保密性。
### 2025.03.06
比特币私钥本质上就是一个随机数,是一个256位，由0和1组成的数字

0100101…01010100  (共256位)

聪哥发明了一种特殊编码（Base58）可以将一大串01010转化为较容易备份的样子

例如：

- KwYHFL7WfhJPkfQkp1LsUwHvy1Pd9KynuxjjVDMZvRSV5D9VJq3v

助记词本质也是一串随机数（128—256位）

比特币社区通过了BIP39协议，来允许将随机数通过特定编码转化为词库中的单词（小知识：比特币改进协议 Bitcoin improvement proposals 简称BIP，是为比特币社区提供规范，完善比特币及其运行进程和外部环境特性的设计指导文件）

助记词钱包还有一个好处是，一组助记词可以派生出N个私钥，每个私钥都可以对应一个币种。如果你有30个币种（BTC、ETH、LTC、EOS等等等等），你不需要每个币种都单独记录一下私钥，只需记录好一组助记词就可以掌控所有资产。

私钥/助记词的作用：

- 计算收币地址

- 签名授权交易

- 恢复钱包资产

私钥就是区块链世界的资产的唯一凭证，拥有了私钥的，便拥有了对应地址上的资产
### 2025.03.05
数字资产即是一种存在于数字形式的资产

数字资产的性质：

1、稀缺性

2、去中心化性

3、安全性

- 区块链钱包：谁持有私钥，谁就可以访问资金

- 软件钱包： 软件钱包可以是基于 Web 的应用、移动应用或桌面应用

- 硬件钱包：硬件钱包是一种类似于闪存驱动器的小型设备，可让您离线存储加密货币，私钥远离您的手机或计算机

- 纸钱包：是的，你没有听错！这是一个物理纸张，上面印有你的私钥和公钥

- 私钥：这是你的钱包的“密码”。它是一个秘密的数字组合，只有你知道。失去它就意味着失去了访问你钱包中资金的能力，所以务必小心保管！

- 公钥：你可以把它看作是你的“邮箱地址”。当别人想给你发送加密货币时，他们会发送到你的公钥地址。


### 2025.03.04
区块用于存放一系列的交易信息，连续排列的区块连接在一起构成了区块链。

每当有新的交易发生时，它都会被广播到网络中的所有节点。然后，这些交易会被加入到新的区块中，并通过一系列的复杂计算和验证，最终被添加到链上。区块链上的每笔交易都是公开的，任何人都可以查看。这确保了数据的真实性和透明性。此外，由于每个参与者都有数据的一份拷贝，因此很难对其进行篡改。

**WEB1.0**：数据只读

**WEB2.0**：数据读写

**WEB3.0**：数据去中心化，用户拥有数据。通过智能合约，用户能够直接和程序交互而无需中间人。

### 2025.03.03
WEB3代表了一个更加开放、自由、去中心化的互联网形态，用户所创造的数据信息与数据资产都将归自身所有数据是存储在一个去中心化的网络中，每一个参与节点都拥有所有的数据备份。

去中心化：

1、在去中心化的系统中，没有一个绝对的权威或控制中心。相反，权力、资源或决策被分散到多个节点或参与者手中。

2、由于没有单一的控制点，去中心化的系统通常更难被篡改或攻击。即使部分节点出现问题，整个系统仍能正常运行。
### 2025.07.11

笔记内容

### 2025.07.12

<!-- Content_END -->
