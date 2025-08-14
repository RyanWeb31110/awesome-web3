# 001 Solana 基础知识参考指南

Solana 区块链是一个强大的工具，每秒可处理数千笔交易，且几乎不收取任何交易费用。本指南将帮助您了解 Solana 的基本原理，并开启您的 Solana 开发之旅。如果您已经在 Solana 上进行开发，请将本指南作为参考，继续加深您对 Solana 基础知识的理解。
在本指南中，我们将介绍 Solana 的基本构造，具体包括：**Accounts** 帐户、 **Programs** 程序、**Instructions** 指令、**Transactions** 交易、**RPCs** 和 **Subscriptions** 订阅。

## TLDR

- **Accounts** 账户就像传统操作系统中的 **files** —— Solana 上的大多数东西都是一个账户：user wallets 用户钱包、**programs** 程序、**data logs** 数据日志，甚至 **system programs** 系统程序
- **Programs** 程序（在其他链上称为 smart contracts 智能合约）是一种特殊类型的“可执行”账户——它们是无状态的，仅根据用户输入的参数执行代码。程序不会自我更新，而是拥有更新其“拥有”的其他账户数据（或状态）的特定权限。而是拥有特定权限的账户来更新它们的数据（或状态）。
- 用户通过 RPC 签署并提交 **transactions** 交易（**bundles of program-specific instructions** 程序特定指令包）到 Solana 集群，以告诉程序该做什么。
- Solana 使用一种称为历史证明（**Proof of History PoH**）的独特共识机制，该机制依靠时间和加密技术来对发送到网络的交易进行时间戳标记和顺序排序。
- 程序和交易签名者授权对链上的某些数据账户进行更改，以更新网络状态。

## Solana 简介

Solana 是一个开源区块链，针对速度和成本进行了优化，允许信息和状态在全球范围内同步。

Solana 采用一种独特的权益证明 (**PoS**) 共识机制，称为历史证明 (**Proof of History PoH**)，该机制使用一种精细的可验证延迟函数，有效地对发送到网络的交易进行时间戳标记和排序。

Anatoly Yakovenko 在 [Solana 白皮书](https://solana.com/solana-whitepaper.pdf)中指出：“它使用一种加密安全函数，其编写方式使得输出无法根据输入预测，并且必须完全执行才能生成输出”(3)。“领导者 (Leader) 对用户消息进行排序，以便系统中的其他节点能够高效地处理它们，从而最大化吞吐量”(2)。

![img](https://www.quicknode.com/guides/assets/images/0-c4654c0741499040f14c2b28ad76d580.png)

本文对 Solana 运行时的深入探讨到此为止，接下来我们将重点介绍如何使用 Solana 进行交互和构建。如果您想深入了解 Solana 的架构及其部分设计特性背后的理论，以下是一些不错的资源：

- [Solana Whitepaper Solana 白皮书](https://solana.com/solana-whitepaper.pdf)
- [Anatoly Yakovenko's (Solana Co-Founder) Medium
  Anatoly Yakovenko 的（Solana 联合创始人）Medium](https://medium.com/@anatolyyakovenko)
- [Solana.com](https://docs.solana.com/cluster/synchronization)

## Accounts 账户

与操作系统中的 **files** 文件一样，帐户用于运行系统、存储数据或状态以及执行代码。

Solana 上有 3 种主要类型的账户：

1. **Data accounts** 数据账户用于存储信息并管理网络状态。数据账户分为两种类型：

   - **System-owned accounts** 系统拥有的账户（通常被称为钱包） *[例如：*[ *用户钱包* ](https://explorer.solana.com/address/E645TckHQnDcavVv92Etc6xSWQaq8zzPtPRGBheviRAk)*]*
   - **Program-derived accounts (often referred to as PDAs)** 
     程序衍生账户（通常称为 PDA）——一个常见的例子是用户针对特定代币的代币账户。它由 Solana SPL 代币程序衍生而来。 *[示例：*[ *用户的代币账户* ](https://explorer.solana.com/address/HmXbXp3q1NCuT8DT3EiTbcDcJti4w3aTUd1S7dWkV7wi)*]*

2. **Program accounts that hold executable code**
   持有可执行代码的程序账户。我们将在下一节中详细讨论这些账户，但这些账户是无状态的。 *[示例：* [*Candy Machine v2 程序账户* ](https://explorer.solana.com/address/cndy3Z4yapfJBmL3ShUp5exZKqR3z33thTzeNMm2gRZ)*]*

3. **Native System Accounts**
   原生系统账户（例如，System Program 系统程序、Voting 投票、Staking 质押、BPF LoaderBPF 加载器）

    *[示例：* [*Solana 系统程序* ](https://explorer.solana.com/address/11111111111111111111111111111111)*]*

> 本文将重点介绍第 1 点和第 2 点。如果您想了解更多关于第 3 点（原生系统帐户）的信息，请访问 [docs.solana.com](https://docs.solana.com/developing/runtime-facilities/programs) 。

简而言之，Solana 上的几乎所有东西都是一个账户。最常见的账户类型是用户钱包，这是一种管理用户 SOL 余额状态的数据账户。

每个帐户都有一个唯一的 address 地址或 public key 公钥（很像文件路径）和一组相关元数据：

- **lamports** ：此帐户拥有的 lampors 数量（64-bit unsigned integer 64 位无符号整数）*
- **owner**: The program owner this account is assigned to (string address of the owner Program). Owner programs control who has write access to accounts--other programs can read the data of another account, but if they tried to modify that data, the transaction would fail.
  **owner** ：此账户被分配给的程序所有者（所有者程序的字符串地址）。所有者程序控制着谁拥有对账户的写入权限 —— 其他程序可以读取另一个账户的数据，但如果它们试图修改该数据，交易将会失败。
- **executable** ：此帐户是否可以执行 instructions 指令（boolean 布尔值）。可执行值为 *true* 的帐户实际上是一个 Program 程序（或 Smart Contract 智能合约）。
- **数据** ：与帐户相关的任何其他数据，存储为raw data byte array 原始数据字节数组
- **rent_epoch** ：此账户下一个需要支付租金的时期（64-bit unsigned integer 64 位无符号整数）


\* **lampors** 是 Solana 原生代币 SOL 的最小面额（代表十亿分之一 SOL，10^9）


任何account的元数据都可以通过运行 [getAccountInfo JSON RPC 查询](https://www.quicknode.com/docs/solana/getAccountInfo)或在 [Solana Explorer](https://explorer.solana.com/) 上搜索来读取。Solana Explorer 允许您搜索任何account，并查看其account类型以及与哪个Program关联的基本详细信息。以下是一个Program拥有的 data account 数据帐户（ user's wallet 用户钱包）的示例：

![img](https://www.quicknode.com/guides/assets/images/1-b6dad4771c0bb328f571df02e01607c7.png)



让我们直观地看一下所有account类型的示例：

![img](https://www.quicknode.com/guides/assets/images/2-16c19fb6aaba7e517e667514ac52c108.png)


这里有很多内容，但您在图中看到的所有内容都是一个账户。在左侧，有一个名为“System Program”的 System Account，它管理所有用户钱包。我们还有一个可执行的Program Account，名为 Solana SPL Token Program，用户可以使用它来铸造、销毁和转移 SPL 代币。最后，我们有Program Derived Accounts 程序派生账户。在本例中，这些是Token Program拥有的token accounts，并与用户账户关联。请注意，一个用户钱包可以拥有多个代币账户。


请记住，您可以在 Solana Explorer 中搜索任何帐户地址以获取其 metadata 元数据并了解其帐户类型以及与哪个Program相关联。

> **Lamports 和 Rent Epoch** 账户和账户数据存储在验证者的内存中，需要以 Lamports 的形式支付“租金”（在 accounts **Lamports** 元数据字段中表示）才能保留在那里。
>
> 账户中存储的数据越多，所需的租金就越高 *（注意：当前账户最大大小为 10 MB）* 。
>
> 您可以使用 JSON RPC 方法 [getMinimumBalanceForRentExemption](https://www.quicknode.com/docs/solana/getMinimumBalanceForRentExemption) 计算租金需求。
>
> 验证者负责定期扫描所有账户并收取租金——账户会在 **rent_epoch** 元数据字段中指定的 **epoch** 进行检查 。
>
> 如果账户的 **Lamport** 余额降至零，网络将清除该账户，数据将丢失。
>
> 在撰写本文时，所有新账户都必须保持免租状态，即在账户中持有相当于两年租金的金额即可获得免租状态。
>
> 来源：* [*https*](https://docs.solana.com/developing/programming-model/accounts) ://docs.solana.com/developing/programming-model/accounts 
>
> Epoch 是 Solana 上的时间度量，目前大约代表 2.5 天。



## Programs 程序

Programs 程序就是 Solana 所说的 Smart Contracts 智能合约——它们是处理信息和请求的引擎：涵盖从代币转账、 Candy Machine mints，到 “hello world” 日志记录以及去中心化金融（DeFi）托管治理等各类操作。

- 请记住，Solana programs 是标记为“executable 可执行”的无状态帐户：“程序被视为无状态，因为存储在程序帐户中的主要数据是编译的 BPF 代码。”
- 程序可以使用“Cross-Program Invocation 跨程序调用”来使用或构建其他 Solana 程序。这通常被称为 composability 可组合性。
- 与许多其他流行的区块链不同，programs是可升级的。
- 大多数Programs都是用户生成的，但 Solana Labs 已经构建并维护了许多链上程序，称为 [Solana 程序库 ](https://spl.solana.com/)。


实际上，用户必须向程序提供特定于程序的 instructions 指令，程序处理这些指令，然后程序可以调用其他程序和/或更新其拥有的程序派生帐户 program-derived accounts PDAs。



### Program Derived Address Accounts PDAs 程序派生地址账户


程序派生地址 (Program Derived Address PDA) 是由特定程序创建和控制的帐户。PDA 具有以下功能：

- 允许程序控制特定地址（称为 program addresses 程序地址），因此没有外部用户可以为这些地址生成带有签名的有效交易。
- 允许程序以编程的方式对跨程序调用指令中存在的程序地址进行签名。Solana 根据程序定义的 seeds 种子和 program ID 程序 ID 确定性地派生 PDA。有关生成的程序地址的更多信息，请访问 [docs.solana](https://docs.solana.com/developing/programming-model/calling-between-programs#hash-based-generated-program-addresses) 。

*Source:* [_https://docs.solana.com/developing/programming-model/calling-between-programs#program-derived-addresses](https://docs.solana.com/developing/programming-model/calling-between-programs#program-derived-addresses)


这使得程序能够提供无需信任的服务，例如托管账户，用于安全地管理交易、投注或 DeFi 协议。参考上面的示意图，SPL Token Program序会根据用户的钱包和特定代币的 mint address 铸币地址，为用户生成一个 PDA。该 PDA 将存储用户所持有代币数量的状态或数据。



## Instructions 指令

Instructions 指令的核心是告诉Program执行某项操作。Solana 将指令定义为“程序中最小的连续执行逻辑单元”。

(来源： [docs.solana.com/terminology](https://docs.solana.com/terminology#instruction) )

一条指令由三部分组成：

1. 您将要调用的程序的公钥，The Public Key of the Program
2. 它将读取或修改的一组 Accounts 帐户
3. Program 执行所需的任何附加数据的 byte array  字节数组（program-specific）


最重要的一点—— **指令是特定于程序的** 。如果您输入了错误的Program ID 或不知道要传递给程序的预期数据结构，交易将失败。


在 Javascript 中，交易指令如下所示：

```javascript
new TransactionInstruction({
	programId: new PublicKey(PROGRAM_ID),
	keys: [
		{ pubkey: signerKeypair.publicKey, isSigner: true, isWritable: true },
		{ pubkey: pda, isSigner: false, isWritable: true },
		{ pubkey: SystemProgram.programId, isSigner: false, isWritable: false },
	],
	data: Buffer.from(SOME_ENCODED_DATA),
})
```



Logs for Simplified Debugging 用于简化调试的日志

您现在可以访问 RPC endpoints的日志，从而更有效地解决问题。如果您在 RPC 调用过程中遇到问题，只需检查 QuickNode 仪表板中的日志即可快速识别并解决问题。 在[我们的定价页面上了解更多关于日志历史记录限制的信息。](https://www.quicknode.com/pricing#features)

这里值得一提的是交易指令所需的账户信息。我们必须提供账户的公钥，无论该账户是否是交易的签名者，或者该账户是否会被写入交易（这有助于提高 Solana 的快速执行能力）。

在上面的例子中，用户对钱包进行签名，数据被写入 PDA。通常还需要包含系统程序账户（例如，用于创建新账户）。最后，我们传递 **SOME_ENCODED_DATA** 。所需数据类型在程序内部定义。程序必须反序列化输入数据才能正确处理。



## Transactions 交易

交易是向网络上的程序传递 instructions 指令的方式。交易可以将多条指令打包在一起，以便同时处理多个操作。

例如，假设您有三条不同的指令：一条指令要求钱包 A 向钱包 B 发送 1 美元 SOL，一条指令要求钱包 B 向钱包 A 发送 33 美元 USDC，以及一条指令要求在交易中记录一条备忘录，内容为“钱包 A 于 YYYY-MM-DD 将 1 美元 SOL 兑换为 33 美元 USDC”。虽然这些指令可以分别发送到网络，但如果一条指令失败，而其他指令成功，会发生什么情况？

您将遇到这种情况：一个人获得了他们的代币，而另一个人没有。通过将指令打包成一个包，我们可以确保所有交易同时成功或失败。

目前交易限制为 1,232 byes。每笔交易必须包含以下内容：

- 一个或多个交易指令的数组 array of  transaction instruction
- 程序将与之交互的所有帐户地址的数组 array of all account addresses 
- 一组签名 array of signatures：我们传入交易的部分账户也必须包含签名。“这些签名向链上程序发出信号，表明账户持有人已授权该交易。通常，程序会使用该授权来允许从账户扣款或修改其数据。”（来源： [docs.solana.com](https://docs.solana.com/developing/programming-model/transactions#signatures) ）例如，对于 NFT 的创建者来说，这也可能是验证其真实性的必要条件。
- a最近的 blockhash 区块哈希：区块哈希本质上是一个 PoH 时间戳。Solana 默认会删除“旧”交易，以防止重复计算，允许 validators 验证器仅根据最近的批次检查交易，并让用户快速确定他们的交易是否已被处理（或失败，以便他们可以重试）。在提交交易之前获取最近的区块哈希，可以让网络知道交易的年龄以及何时应该过期（如果需要）。


一旦网络收到交易，它就会返回一个 transaction ID（base-58 编码的字符串），可用于查找交易详情。



## RPC 远程过程调用

JSON-RPC 是一种以 JSON 编码的远程过程调用协议，允许用户向 Solana 集群发送信息并从网络获取响应。Solana 目前维护三个网络集群：

- Devnet：测试应用程序和方法功能的网络集群
- Testnet：测试网：用于对系统所有最新计划升级进行压力测试的开发环境
- Mainnet Beta：主网 Beta：生产环境


客户端可以通过 Solana 节点使用 JSON-RPC 请求向三个集群中的任何一个发出读取或写入请求。节点就像“空中交通管制”一样，负责处理网络请求。客户端通过 Solana Web3 Connection class 连接到 Solana 节点：

```javascript
const HTTP_ENDPOINT = 'https://example.solana-devnet.quiknode.pro/000/'
const SOLANA_CONNECTION = new Connection(HTTP_ENDPOINT)
```


在上面的示例中， **HTTP_ENDPOINT** 是一个指向 Solana 节点的 HTTP URL，该节点最终会将您的请求路由到三个集群之一。


每个请求都需要一个 API endpoint 来连接到网络。您可以选择使用公共节点，也可以部署和管理您自己的基础设施；但是，如果您希望响应速度提高 8 倍，您可以将繁重的工作交给QuickNode。


您现在可以**使用 Solana 上的 USDC 支付 QuickNode 计划** 。作为首家接受 Solana 付款的多链提供商，我们正在简化开发者的付款流程——无论您是创建新帐户还是管理现有帐户。[ 点击此处了解更多关于使用 Solana 付款的信息 ](https://blog.quicknode.com/quicknode-now-accepts-solana-payments/)。

![img](https://www.quicknode.com/guides/assets/images/3-433d7a5e2adf0f9a7a1ad730df9348ba.png)


建立与 Solana 集群的连接后，您可以对网络执行多种读取或写入方法。QuickNode[ 文档中](https://www.quicknode.com/docs/solana)提供了完整的 RPC 方法列表和文档。虽然每种方法都有其用途，但本指南中与讨论相关的一些方法如下：

- [sendTransaction](https://www.quicknode.com/docs/solana/sendTransaction) - 向 Solana 网络发送交易

- [getAccountInfo](https://www.quicknode.com/docs/solana/getAccountInfo) - 获取与任何帐户相关的元数据

- [getLatestBlockhash](https://www.quicknode.com/docs/solana/getLatestBlockhash) - 返回网络中最新的区块哈希

  

HTTP 调用允许我们告诉网络做某事（例如，处理交易及其指令，返回有关帐户的当前状态信息等），但如果我们希望网络提醒我们帐户状态的变化怎么办？



## Subscriptions 订阅

Subscriptions 订阅是一种特殊的 RPC 请求，我们通过 WebSocket (WSS) 发出请求，而不是使用 HTTP 端点。Solana Web3 JSON RPC 包含多种订阅方法（例如 [accountSubscribe](https://www.quicknode.com/docs/solana/accountSubscribe) ，它将监听帐户的 lamports 或数据的更改）。然后，我们可以使用回调函数订阅这些事件，从而允许在网络发生变化时执行某些所需的操作。

Connection class的构造函数允许我们传递一个可选的 *commitmentOrConfig* 。在其中，我们可以传递一个 *wsEndpoint* 。用于提供指向完整节点 JSON RPC Websocket 端点的端点 URL。如果您已经创建了 QuickNode 端点，您应该会在[端点页面](https://www.quicknode.com/endpoints)的右侧看到此选项。

```javascript
const HTTP_ENDPOINT = 'https://example.solana-devnet.quiknode.pro/000/'
const WSS_ENDPOINT = 'wss://example.solana-devnet.quiknode.pro/000/'
const solanaConnection = new Connection(HTTP_ENDPOINT, {
	wsEndpoint: WSS_ENDPOINT,
})
```


如果不传递 *wsEndpoint* 会发生什么？Solana 提供了一个 **makeWebsocketUrl** 函数来解决这个问题，该函数会将端点 URL 中的 *https* 替换为 *wss* 或将 *http* 替换为 *ws* ( [source](https://github.com/solana-labs/solana/blob/627d91fb20272afa5ce790fbec01b40a29fb9852/web3.js/src/connection.ts#L2452) )。由于所有 QuickNode HTTP 端点都具有一个具有相同身份验证令牌的对应 WSS 端点，因此可以省略此参数，除非您希望为 Websocket 查询使用单独的端点。


要深入了解如何创建 WebSocket 订阅，请查看[指南：如何创建 Solana WebSocket 订阅 ](https://www.quicknode.com/guides/solana-development/how-to-create-websocket-subscriptions-to-solana-blockchain-using-typescript)。



## Pull it All Together

先快速回顾一下：

- Solana 上的几乎每个东西都是一个帐户：user wallets 用户钱包、programs 程序、system programs 系统程序、program data 程序数据等。
- Programs 程序是无状态的——它们处理 instructions 指令并可以调用其他程序并改变它们拥有的 data accounts 数据账户的状态。
- Instructions are program-specific and tell a program what to do.
  指令是特定于程序的，它告诉程序要做什么。
- 交易将一条或多条 instructions 指令与 signature 签名打包在一起，以授权程序执行一项或多项任务。
- RPC 协议允许客户端通过节点（如 QuickNode）向 Solana 的三个集群之一发送读/写请求。
- 客户端可以通过使用 WebSocket 请求订阅网络变化。

让我们来看一个示例。假设有一个名为“GM”的程序，它有一个计数器，每当用户向该程序发送“GM”消息时，它都会记录下来。

![img](https://www.quicknode.com/guides/assets/images/4-b82268881919612c1bc1af4ea333d150.png)


在上面的示例中，客户端创建了一个包含多条交易指令的交易。这些指令是特定于程序的，因此它们需要与GM Program的预期结构相匹配。

客户端使用 *Connection* 类，通过 RPC 提供程序（例如 QuickNode）对交易进行签名并将其发送到 Solana 集群。

RPC 提供程序将请求路由到集群。Program account执行并处理收到的输入指令。

程序账户是无状态的，不会更改，但会授权更改 PDA 的数据以增加 GM 计数器的值。

客户端创建的订阅会监听 PDA 的更改；然后，网络会提醒客户端账户已更改。



## 参考资料和来源

- 原文链接：https://www.quicknode.com/guides/solana-development/getting-started/solana-fundamentals-reference-guide#instructions
- https://solanacookbook.com/core-concepts/accounts.html#deep-dive
- https://www.quicknode.com/guides/solana-development/an-introduction-to-the-solana-account-model
- https://medium.com/@anatolyyakovenko
- https://docs.solana.com/developing/programming-model/accounts
- https://docs.solana.com/developing/clients/jsonrpc-api#getaccountinfo