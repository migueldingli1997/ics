# 3：IBC 设计原则

**这是对 IBC“设计原则”的解释。**

**有关 IBC 规范中使用的术语的定义，请参见[此处](./1_IBC_TERMINOLOGY.md) 。**

**有关架构的概述，请参见[此处](./2_IBC_ARCHITECTURE.md) 。**

**有关一组示例用例，请参见[此处](./4_IBC_USECASES.md) 。**

**有关设计模式的讨论，请参见[此处](./5_IBC_DESIGN_PATTERNS.md) 。**

“区块链间通信协议”的设计空间很广，这个术语本身变得太包罗万象了。 “区块链间通信协议”（IBC）是该设计空间中的一个非常特殊的点，它被选择为预期的可互操作区块链的链间生态系统提供特定的多样性，局部性，模块化和高效等特性。本文档概述了 IBC 为什么可以提供这些特性，并列举了主要的高级设计目标。

## 多样性

IBC 被设计为*多样性*协议。该协议支持*异构*区块链，其状态机用不同的语言实现不同的语义。可以将在 IBC 之上编写的应用程序*组合*在一起，并且使用IBC协议实现*自动化* 。

### 异构性

IBC 可以通过任何满足基本需求集的共识算法和状态机来实现（快速最终性，常量大小的状态加密承诺和简洁的加密承诺证明）。该协议处理数据身份认证，传输和排序（任何多链应用程序的共同要求），但与应用程序本身的语义无关。通过 IBC 连接的异构链必须了解兼容的应用程序层“接口”（例如，用于转移通证的接口），但是一旦通过 IBC 接口处理程序后，状态机就可以支持任意定制的功能（例如，屏蔽的交易）。

### 可组合性

协议开发人员和用户都可以将在 IBC 之上编写的应用程序组合在一起。 IBC 定义了一组用于身份认证，传输和排序的原语，以及一组用于资产和数据语义的应用层标准。支持兼容标准的链可以连接在一起并和任何打开连接（或重用连接）的用户进行交互，资产和数据可以跨多个链自动（“多跳”）和手动（通过依次发送几个 IBC 中继交易）中继。

### 自动化

IBC 中的“用户”或“参与者”（可以启动连接，创建通道，发送数据包，报告拜占庭欺诈等），可以是但不是必须是人类，模块，智能合约和自动化的链下进程，都可以使用该协议（例如，要支付用于计算的 gas 费用成本），并可以自行采取行动或协同行动。设计跨多个链的复杂交互（例如建立连接时的三次握手或多跳通证转移），使得除了单个启动动作之外的所有动作都可以从用户那里抽象出来。最终，可能可以自动启动一个新的区块链（抛开配置物理基础架构），然后启动 IBC 连接，并完全自动使用新链的状态机和吞吐量。

## 模块化

IBC 被设计为*模块化*协议。该协议被构造为一系列具有明确安全属性和要求的分层组件。特定层上组件的实现可以有所不同（例如不同的共识算法或连接打创建过程），只要它们为更高层提供必要的属性（例如最终性，<1/3 拜占庭安全性或两个链上的嵌入式受信任状态）。状态机仅需要了解 IBC 协议的兼容子集（例如，用于彼此共识的轻客户端验证算法）即可安全地进行交互。

## 局部性

IBC 被设计为*局部*协议，这意味着仅需要两个连接着的链的信息就可以探讨 IBC 连接的安全性和正确性。身份认证原语的安全性要求仅涉及连接中的区块链的共识算法和验证人集合，并且维护一组 IBC 连接的区块链仅需要了解其所连接的链的状态（无论这些链又连接到哪个链）。

### 通讯和信息的局部性

IBC 对其运行所在的区块链网络的拓扑结构不做任何假设，也不依赖任何特征。无需查看全局的区块链网络拓扑：可以在两条链之间的单个连接级别上探讨安全性和正确性，也可以针对网络拓扑中的子图探讨安全性和正确性。用户和链可以只根据区块链网络图的一部分已知和假定正确（各种角度）的信息来探讨其假设和风险。

IBC 中没有必需的“根链”-全球网络的某些子图可能会发展成中心辐射状结构，某些子图可能保持紧密连接，某些子图可能采用更奇特的拓扑。通道是端到端的；在第一个版本中，IBC 仅支持单跳路径，但将来将支持多跳路径（尽管涉及共识算法正确性假设，自动路由不一定可行或安全）。

但是，应用程序数据可能具有协议用户需要注意的显著的非局部属性，例如通证的原始来源区域（可能已在复杂的多跳路径上发送了多次），原始权益和通过跨链验证提供其服务的验证人的身份，或与管理非同质化通证的特定对象功能密钥相关联的原始智能合约。这些非局部属性不需要由 IBC 协议本身理解，需要由用户和更高级别的应用来探讨。

### 正确假设和安全的局部性

IBC 的用户（在区块链级别以及在人类或智能合约级别上）自行选择哪些共识算法，状态机和验证人集合为“假定正确”（以特定方式运行，例如<1/3 拜占庭）并选择以哪种方式假设正确性。假设 IBC 协议正确实施，则用户永远不会因拜占庭行为或验证人集合或未明确决定假定为验证人的区块链提交的错误状态机转换而遭受应用级不变违规（例如资产通涨）的风险。这在互连的区块链的预期大型网络拓扑中尤为重要，在这种拓扑中，一些区块链和验证人集合有时可能会有意的进行拜占庭行为—  IBC 保守的实施，限制了风险并限制了可能造成的损害。

### 许可的局部性

IBC 中的操作（例如打开连接，创建通道或发送数据包）是需要涉及两条链之间特定连接的状态机和参与者局部许可的。各个链可以选择要求特定应用层操作（例如，委托安全性惩罚）的许可机制（例如，治理）的批准，但是对于基本协议，操作是无许可的（抛开 gas 费用和存储成本）-默认情况下，无需任何批准即可创建连接，创建通道和发送数据包。当然，用户自己必须检查每个 IBC 连接的状态和共识，并决定使用它是否安全（例如，其存储是否处于受信任状态）。

## 高效性

IBC 被设计为一种*高效的*协议：链间数据和资产中继的平摊成本应主要包括底层状态转换或与数据包相关的操作（例如，转移通证）的成本，以及一些较小的固定开销。
