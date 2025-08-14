# 002 面向 EVM 开发人员的 Solana 开发

多年来，ETH虚拟机 (EVM) 一直是去中心化应用程序 (dApp) 开发的基础，但 Solana 的创新架构提供了独特的优势，例如高吞吐量、低延迟和成本效益。本指南专为希望在 Solana 上进行构建的 EVM 开发者而设计。本指南将利用您现有的ETH和 EVM 知识，帮助您了解 Solana 的关键概念、工具和工作流程。

读完本指南后，您将深入了解 Solana 的架构及其与ETH的区别。您将了解账户、执行模型和交易处理等关键概念与 EVM 的比较，从而帮助您将开发思维从基于 Solidity 的智能合约转变为 Solana 基于 Rust 的无状态程序。

### Why Solana? 

作为ETH开发者，探索 Solana 因其技术优势和显著的生态系统增长提供了独特的机会。

#### 技术优势

- **并行执行实现高吞吐量** ：Solana 利用 [Sealevel](https://solana.com/news/sealevel---parallel-processing-thousands-of-smart-contracts) 并行执行引擎，该引擎允许同时处理多个交易，这与ETH的顺序执行模型不同。这可以实现更高的吞吐量，而不会造成网络拥堵。
- **无状态程序** ：与存储自身状态的ETH智能合约不同，Solana 程序是无状态的，并与外部账户交互以进行数据存储。这种代码和状态的分离提高了效率并降低了链上存储成本。
- **单体式设计** ：与ETH的模块化路线图不同，Solana 的设计旨在实现原生扩展，而无需依赖 Rollup 或分片。这意味着更简单的开发、更少的跨链复杂性、不存在流动性碎片化，以及更佳的全网用户体验。
- **成本效益：** Solana 的 Layer 1 架构旨在降低交易费用，为开发人员和用户提供经济可行的环境，而无需依赖额外的扩展解决方案。

#### 生态系统增长和开发者机会

- **链上 GDP 增长：** 2024 年第四季度，Solana 的应用总收入（称为链上 GDP）环比增长 213%，从 2.68 亿美元增至 8.4 亿美元。值得注意的是，仅 11 月份，Solana 的应用收入就达到了 3.67 亿美元。
- **DeFi 总锁定价值 (TVL)：** Solana 成为 DeFi TVL 规模第二大的区块链，截至 2024 年第四季度末达到 86 亿美元，环比增长 213%。
- **去中心化交易所 (DEX) 占据主导地位：** Solana 平台上领先的 DEX [Raydium](https://raydium.io/) 在 2024 年第四季度的日均交易量达 32 亿美元，较上一年大幅增长。Raydium 在全球 DEX 交易量中的份额环比增长超过 66%，使其成为交易量份额排名第二的 DEX，仅次于 Uniswap。
- **流动性质押增长：** Solana 的流动性质押率环比增长 33%，达到 11.2%。领先的流动性质押提供商 Sanctum 现已支持超过 100 种流动性质押代币 (LST)。

Solana 网络的技术能力及其不断扩展的生态系统，使其成为开发者的理想选择。通过学习如何在 Solana 上进行构建，您可以利用这些机会，为网络的发展做出贡献。

有关详细分析，请参阅 Messari 的 [《2024 年第四季度索拉纳状况》报告 ](https://messari.io/report/state-of-solana-q4-2024)。



## ETH 和 Solana 之间的主要架构差异

在开始之前，熟悉ETH和 Solana 术语之间的差异至关重要。

### ETH与 Solana：术语比较表

| **Ethereum (EVM) **                         | **Solana (SVM) **                                            | **Description**                                              |
| ------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **Smart Contracts 智能合约**                | **Programs 程序**                                            | 在ETH中，智能合约既存储逻辑，也存储状态。在 Solana 中，程序是包含可执行代码但**无状态的**账户，数据存储在单独的账户中。 |
| **Wei **                                    | **Lamports**                                                 | 在ETH中， **wei** 是 **ETH** 的最小单位，其中 **1 ETH = 10¹⁸ wei** 。<br />在 Solana 中， **lampor** 是 **SOL** 的最小单位，其中 **1 SOL = 1,000,000,000 个 lampor** 。 |
| **Gas**                                     | **Compute Units **                                           | 在ETH中，gas 衡量交易所需的计算工作量，其中 `Fee = gas used × (base + priority)` 。<br />在 Solana 中，计算单元 (CU) 起着类似的作用，其中 `Fee = fixed base (per signature) + (CU × CU price)` 。 |
| **ABI<br />(Application Binary Interface)** | **IDL <br />(Interface Description Language) IDL<br />（接口描述语言）** | 在ETH中，ABI（应用程序二进制接口）定义了合约如何与外部应用程序交互。<br />在 Solana 中，IDL（接口描述语言）起着同样的作用，它定义了程序如何公开函数以供客户端交互。 |
| **Proxy Contracts 代理合约**                | **Programs 程序**                                            | 在ETH中，使用代理合约来实现智能合约的升级。<br />在 Solana 中，程序**默认是可升级的** 。 |
| **Nonce <br />随机数**                      | **Blockhash / Durable Nonce <br />区块哈希/持久随机数**      | ETH交易使用**每个账户递增的随机数**来防止重放攻击。<br />Solana 主要使用**最近的区块哈希**进行交易验证，但也支持**[持久随机数 (Durable Nonces)](https://www.quicknode.com/guides/solana-development/transactions/how-to-send-offline-tx?utm_source=internal&utm_campaign=guides&utm_content=solana-development-for-evm-developers)** ，以实现离线签名和更长的交易有效期。 |
| **Transaction 交易**                        | **Instruction 指令**                                         | 在ETH中，一笔交易通常调用**单个合约函数** ，即使它涉及复杂的合约逻辑。<br />在 Solana 中，一笔交易由**一条或多条指令**组成，每条指令都调用一个程序来执行特定的操作。 |
| **ERC20 Token ERC20 代币**                  | **SPL Token SPL 代币**                                       | 在ETH中，ERC-20 代币是为每个代币部署的**单独智能合约** 。<br />在 Solana 中，SPL 代币由**单一共享代币计划**管理，无需为每个代币部署单独的合约。 |



### Account Model: Stateful vs. Stateless 

在ETH上，智能合约**会存储自己的状态** ，这意味着存储和执行是捆绑在一起的。
在 Solana 中， **程序**根本不存储任何状态。所有状态都存储在与合约本身分离的账户中。

#### Ethereum’s Account Model 

ETH账户有两种形式：

- **EOAs (Externally Owned Accounts)**：外部拥有账户，具有私钥的用户控制账户。
- **CAs (Contract Accounts)**：合约账户，存储**代码和状态的**智能合约账户。

当你在ETH上部署智能合约时，它会存储在**CAs 合约账户**中，并且所有状态修改都发生在该账户内。

#### Solana’s Account Model Solana

在 Solana 上， **一切都是账户 everything is an account** 。但它们分为两类：

- **Program Accounts (Executable：**程序账户（可执行） ，仅包含代码，类似于ETH智能合约，但没有状态。
- **Data Accounts (Non-Executable)**：数据账户（不可执行），存储所有链上数据，作为程序的持久存储。

#### Solana 中的账户如何运作

- 每个帐户都有**一个所有者** 。所有者程序是唯一被允许修改帐户数据的实体。
- 默认情况下，新账户由**System Program 系统程序**拥有，该系统程序处理一些操作，如转移代币、分配账户数据和分配账户所有权。
- 要创建帐户，您只需将 SOL 发送到该地址。
- 一旦将帐户分配给程序，程序本身就会控制该帐户可以发生的情况。
- 账户需要持有一定数量的 SOL（Lamports）来支付[租金 ](https://www.quicknode.com/guides/solana-development/getting-started/understanding-rent-on-solana?utm_source=internal&utm_campaign=guides&utm_content=solana-development-for-evm-developers)，这是一种确保区块链资源高效利用的机制。它要求账户维持最低余额才能保持活跃。如果账户关闭，可以收回这笔租金。

程序账户可以通过从其他程序分配账户或使用**程序派生地址[ ( Program Derived Addresses  PDA) ](https://www.quicknode.com/guides/solana-development/anchor/how-to-use-program-derived-addresses?utm_source=internal&utm_campaign=guides&utm_content=solana-development-for-evm-developers) **模型将其数据存储在数据账户中。开发者可以通过将程序设置为账户所有者并使用种子派生地址来生成这些数据账户。


从下图可以看出，一个 Account 有 5 个字段： `lamports` 、 `owner` 、 `executable` 、 `rent_epoch` 和 `data` 。

![Program Account and Data Account](https://www.quicknode.com/guides/assets/images/program-account-data-account-15cf0cd574012926bb1f2ad9f75dc66c.png)*[Image Source 图片来源](https://solanacookbook.com/assets/account_example.5b70d95a.jpeg)*



#### Program Derived Addresses (PDAs) 程序派生地址

**程序派生地址 (PDA)** 是一种特殊类型的账户地址，程序无需私钥即可控制。它们支持程序签名。如果 PDA 是从 Program ID 派生的，程序就可以为 PDA 签名。此外，它们允许确定性数据存储，确保相同的seed值始终生成相同的 PDA。

PDA 可以用作链上账户的地址（唯一标识符），从而提供一种轻松存储、映射和获取程序状态的方法。[ *来源*](https://solana.com/docs/core/pda)
开发者可以使用种子(seed)和碰撞 (bump) 来生成 PDA，确保它们不会与用户控制的地址冲突。碰撞是一个附加值（介于 0 到 255 之间），它会被添加到种子中，以生成唯一的地址。

![PDA Generation](https://www.quicknode.com/guides/assets/images/pda-derivation-ef07d3b6af6d718c549a8b49a4539c3b.svg)[*Image Source 图片来源*](https://solana.com/docs/core/pda#generating-pdas)

例如，假设我们的程序有一个 `increment` 函数，用于增加计数器。我们可以为每个用户创建一个单独的 PDA，而不是将所有用户计数器存储在一个帐户中，其中 PDA 的seed基于用户的公钥。

- 如果用户 A 调用 `increment` ，程序将定位并更新用户 A 的计数器 PDA。
- I如果用户 B 调用 `increment` ，程序将定位并更新用户 B 的计数器 PDA。
- 由于程序拥有 PDA，因此只有程序可以修改它——用户无法直接控制数据。但程序也可以通过在 PDA 数据中存储 `authority` 字段来设计权限检查，以便根据程序的逻辑赋予用户间接控制权。

这消除了每次写入时明确授权的需要，从而使 Solana 程序变得非常高效。

Solana 的账户模型引入了一些关键优势，可以改善用户体验，并简化在 Solana 上的构建。让我们来探索一下其中的一些优势。



### Parallel Execution and Localized Fee Markets 并行执行和本地化费用市场

Solana 的账户模型通过要求每笔交易预先定义其读取和写入的账户来实现**并行执行** 。这使得非冲突交易能够并行处理，从而提高网络效率并减少拥堵。

**工作原理** ：

- 如果两笔交易仅从同一个账户读取，则它们可以同时执行。
- 如果一个人从同一个帐户写入而另一个人读取，则必须按顺序执行以保持数据一致性。
- 如果多笔交易尝试写入同一账户，则系统会逐笔处理。处理方式由**本地费用市场**决定，优先级高（费用高）的交易将优先执行。

[Solana Priority Fee API](https://marketplace.quicknode.com/add-on/solana-priority-fee?utm_source=internal&utm_campaign=guides&utm_content=solana-development-for-evm-developers) SOLANA 优先费用 API

QuickNode 的 [Solana 优先费用 API](https://marketplace.quicknode.com/add-on/solana-priority-fee?utm_source=internal&utm_campaign=guides&utm_content=solana-development-for-evm-developers) 允许开发人员检索最近费用数据的优先费用。

| Transaction 1          | Transaction 2                      | Execution Model on Solana |
| ---------------------- | ---------------------------------- | ------------------------- |
| Alice sends SOL to Bob | Alice sends SOL to Charlie         | Sequential<br /> 顺序     |
| Alice sends SOL to Bob | Charlie sends SOL to Carol Charlie | Parallel <br />平行线     |

**为什么重要** ：

- 不参与高需求操作（例如，流行的代币铸造）的用户不会受到不相关交易费用飙升的影响。
- 程序设计会影响性能。例如，写入竞争激烈的自动做市商 (AMM) 和代币铸造程序可能会遇到执行瓶颈。设计程序以最大限度地减少写入竞争可以提高性能。

### Program Reusability 程序可重用性

Solana 账户模型的其他主要优势包括程序可重用性。Solana 允许多个应用程序与共享的链上程序进行交互。这减少了代码重复、部署成本和执行效率低下的问题，从而使 Solana 更具可扩展性和可组合性。

#### Ethereum: Smart Contracts Are Isolated —— ETH的智能合约是孤立的

在ETH上，每个新的用例（代币、NFT、DeFi 协议）通常需要：

- 单独的智能合约部署，导致整个网络的逻辑冗余。
- 由于重复合同，在合同创建时会产生不必要的交易费用。
- 存储和执行独立，使得合约难以无缝交互。

例如，如果您创建一个新的 ERC-20 代币，则必须部署一个全新的智能合约，即使大多数 ERC-20 合约的功能相同。

#### Solana: Reusable Programs —— Solana的程序可重用

Solana 程序旨在共享和重用。开发人员无需为每个用例部署新程序，而是可以在现有程序下创建新帐户。
例如， [Solana Token Program](https://spl.solana.com/token)管理所有 SPL 代币（Solana 上的代币标准），从而无需单独的代币合约：

- I无需部署合约，而是在代币计划下创建一个铸币账户。
- 该铸币账户定义了代币的属性，例如供应、小数和权限。
- 代币程序处理所有操作（minting 铸造、burning 销毁、transfers 转移），而不需要每个代币单独创建合约。

这些示例可以扩展到其他用例，如 DAO、NFT 等。

> 本指南无法涵盖 Solana 账户模型的所有细节。如果有兴趣了解更多关于 Solana 账户模型的信息，请查看 [《Solana 账户模型简介》](https://www.quicknode.com/guides/solana-development/getting-started/solana-fundamentals-reference-guide#accounts?utm_source=internal&utm_campaign=guides&utm_content=solana-development-for-evm-developers) 指南。

### Gas Fees 

在ETH和 Solana 之间转换时，了解交易费用的运作方式至关重要，因为它们的架构选择差异很大。虽然它们有一些相似之处，例如都设有优先费用和基础费用，但费用结构却有所不同。

#### Ethereum Fees 

- **Global Fee Market：全球费用市场，** ETH采用通用费用市场，某个领域（例如 NFT mint）的高需求可能会推高所有交易的 Gas 价格。Gas 衡量的是计算工作量，而您设置的 Gas 价格决定了交易的优先级，这会导致拥堵期间费用飙升和成本增加。
- **Base Fee：基础费用，** ETH上的每个区块都有一个基础费用，该费用由当前区块之前的区块决定。
- **Priority Fee：优先费用，** 优先费用是一种激励验证者优先处理交易的方式。用户可以支付更高的优先费用，以确保其交易优先于其他交易被打包进区块，这可能会导致在拥堵期间费用飙升。

Gas 衡量计算工作量，类似于 Solana 的计算单位，总费用计算为 `gas used * gas price` ，而 gas price 由基本费用和优先费用组成。

#### Solana Fees 

Solana 的费用结构确保只有与相同账户交互的交易才会竞争优先权，而其他交易则不受高需求操作的影响。

**Base Fee 基本费用：**

- 取决于交易需要的签名数量。
- 每个签名需要花费 5000 个 lamport（0.000005 SOL）。
- 一半的费用被烧毁，另一半归 validators 验证者所有。

**Priority Fee 优先费用：**

- 用户可以竞标额外的费用来优先处理他们的交易，类似于ETH的优先费用。
- Validators 验证者使用这笔费用来确定在高需求情况下首先包含哪些交易。
- 一半的费用被烧毁，另一半归验证者所有。

总费用取决于用户愿意为优先权支付的价格和交易需要的总计算量（复杂的交易需要更多的计算单元）。

**Rent (Storage Cost)  租金（仓储费用）：**

- Solana 要求账户存入 SOL 才能保持活跃状态 —— 这可以防止不活跃的账户占用不必要的存储空间。
- 这并非费用，而是可退还的押金。账户关闭时，租金将按照 program owner（例如，通常是账户授权方）的规定收回。
- 有关租金的更多详细信息，请参阅 [Rent on Solana](https://www.quicknode.com/guides/solana-development/getting-started/understanding-rent-on-solana?utm_source=internal&utm_campaign=guides&utm_content=solana-development-for-evm-developers) 指南。

### Transaction Processing 交易处理

Solana 和 ETH 都支持原子交易，这意味着如果交易的任何部分失败，整个交易都会被撤销。
然而，Solana 和 ETH 的交易处理略有不同。

#### Ethereum Transactions

在ETH上，一笔交易通常是对智能合约的一次函数调用。因此，开发者可能需要创建自定义函数来处理特定的用例。

ETH使用 incremental nonces 增量随机数（与您的账户绑定的计数器）来确保交易的唯一性并防止重放攻击。交易在内存池中等待，由验证者提取，但这可能会导致网络拥堵时出现抢先交易或更高的 Gas 费用。

#### Solana Transactions

Solana 在处理交易方面采用了不同的方法。一笔交易可以捆绑多条 [instructions 指令 ](https://www.quicknode.com/guides/solana-development/getting-started/solana-fundamentals-reference-guide#instructions?utm_source=internal&utm_campaign=guides&utm_content=solana-development-for-evm-developers)*（可以将它们视为多个微函数调用），* 从而允许你一次性完成一系列操作（例如，转移代币和更新余额）。

但是，单个交易中可以包含的指令数量是有限制的：

- **Transaction Size Limit 交易大小限制** ：每笔交易 1232 字节
- **Compute Unit Limit 计算单元限制** ：交易必须保持在最大计算预算内，这意味着复杂的交易可能需要拆分

 Solana 上的计算单元

| Limits                                                       | Compute Units     |
| ------------------------------------------------------------ | ----------------- |
| Max Compute per block 每个块的最大计算量                     | 48 million 4800万 |
| Max Compute per account per block 每个账户每个区块的最大计算量 | 12 million 1200万 |
| Max Compute per transaction 每笔交易的最大计算量             | 1.4 million 140万 |
| Transaction Default Compute 交易默认计算                     | 200,000 20万      |

随着您使用 Solana 的经验越来越丰富，您可能会遇到这些交易限制。有一些工具，例如 [Lil' JIT Jito Bundles](https://marketplace.quicknode.com/add-on/lil-jit-jito-bundles-and-transactions?utm_source=internal&utm_campaign=guides&utm_content=solana-development-for-evm-developers) ，可以实现多个交易的原子批处理，从而允许执行超出标准 CU 上限的复杂操作。
来源：[ 优化 Solana 交易的策略](https://www.quicknode.com/docs/solana/transactions)*

Solana 使用最近的 `blockhash` （而不是 `nonce` 来验证交易。此区块哈希的有效期为 150 个区块，超过 150 个区块后，交易将过期。
另外，Solana 支持 [Durable Nonces](https://www.quicknode.com/guides/solana-development/transactions/how-to-send-offline-tx?utm_source=internal&utm_campaign=guides&utm_content=solana-development-for-evm-developers) ，允许离线签名交易并随时提交。Durable Nonces 是唯一且连续的，可以防止重放攻击并确保延迟交易的安全执行。
虽然没有内存池，但优先费用仍然在交易排序中发挥作用。交易直接发送给 validators 验证者和 leaders 领导者，从而降低了开销并减少了延迟，但验证者需要将交易转发给领导者，直到交易被处理完毕。

领导者负责生产区块，每 4 个区块轮换一次。

Solana 交易由以下部分组成：

- an array of instructions 一系列指令
- an array of accounts to read or write from 读取或写入的账户数组
- an array of signatures 一组签名
- a recent blockhash 最近的区块哈希值

有关 Solana 交易的更多详细信息，请查看[指南 ](https://www.quicknode.com/guides/solana-development/getting-started/solana-fundamentals-reference-guide#transactions?utm_source=internal&utm_campaign=guides&utm_content=solana-development-for-evm-developers)。



### Development Tools 

下表比较了ETH（EVM）和 Solana（SVM）之间的一些常见开发工具。

| **Ethereum Tool**    | **Solana Equivalent**                                        | **Description**                                              |
| -------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **Solidity**         | **Rust, C, TypeScript, Assembly**                            | Rust（无论是否使用 Anchor）是 Solana 的标准。Solana 也使用 C、Assembly 和 TypeScript。 |
| **Hardhat/Foundry**  | **[Solana Test Validator](https://docs.solana.com/developing/test-validator)**, **[LiteSVM](https://github.com/LiteSVM/litesvm)**, **[Luzid](https://luzid.app/)** | 用于测试程序和账户的本地区块链，类似于 Hardhat 的节点。      |
| **Hardhat/Foundry ** | **[Program-test](https://docs.rs/solana-program-test/latest/solana_program_test/)**, **[BankRun.js](https://kevinheavey.github.io/solana-bankrun/)** ** | Rust 和 JS 程序测试框架                                      |
| **Ethers.js / Viem** | **[@solana/kit](https://www.npmjs.com/package/@solana/kit) （以前称为 Solana Web3.js 2.x）、 [Solana Web3.js 1.x（已弃用）](https://www.npmjs.com/package/@solana/web3.js)** | 用于客户端 dApp 交互的 JavaScript SDK，类似于 Ethers.js。    |
| **Remix **           | **[Solana Playground](https://beta.solpg.io/)**              | 基于 Web 的 IDE，用于编写、测试和部署 Rust 程序，与 Remix 的易用性相似。 |
| **ABI**              | **[Codama](https://github.com/codama-idl/)**, **[Shanks](https://github.com/metaplex-foundation/shank)**, **[Anchor](https://www.anchor-lang.com/docs/basics/idl)** | 标准化的 IDL 和客户端生成工具，替代ETH的 ABI 进行程序交互。  |
| **Etherscan**        | **[SolanaFM](https://solana.fm/)**, **[Solana Explorer](https://explorer.solana.com/)**, **[Solscan](https://solscan.io/)** | 用于检查账户和交易的区块链浏览器，类似于 Etherscan。         |
| **scaffold-eth**     | **[create-solana-dapp](https://github.com/solana-developers/create-solana-dapp)** | 用于快速设置 dApp 的样板生成器，与 scaffold-eth 相当。       |
| **RainbowKit**       | **[Solana Wallet Adapter](https://github.com/anza-xyz/wallet-adapter)** | 处理钱包连接的框架，类似于ETH中的 wagmi 或 RainbowKit。      |

如果您还没有准备好深入学习 Rust，可以使用 Neon（一款部署在 Solana 程序中的 EVM）来开发 Solana。它允许您编写 Solidity 合约并将其部署到 Solana 上。您可以[在此处](https://neon.com/)找到更多关于 Neon 的信息，并查看 [Neon 指南 ](https://www.quicknode.com/guides/tags/neon?utm_source=internal&utm_campaign=guides&utm_content=solana-development-for-evm-developers)。



### Smart Contract (Program) Development 

在智能合约（程序）开发中，Solana 采取了不同的方法，其中程序是无状态的，数据存储在单独的账户中。

本节将帮助您理解ETH和 Solana 智能合约开发之间的主要区别。

####  智能合约结构：代码与状态分离

##### ETH：智能合约同时拥有代码和状态

- 在ETH上，智能合约是一个包含逻辑（代码）和持久存储（状态）的单一单元。
- 它作为合约账户（Contract Account CA）部署，与用户控制的外部拥有账户（Externally Owned Accounts EOA）分开。
- **示例：** 如果您检索代币余额，它将直接从合约的存储中获取。
- 升级需要代理模式（例如，Transparent Proxy 透明代理、UUPS）。

##### Solana：程序是无状态的；状态是外部的。

- Solana 程序不存储任何状态。相反，所有数据都存储在与程序交互的账户中。
- 程序部署到可执行帐户中，状态存储在不可执行帐户中（例如，PDAs - Program Derived Addresses 程序派生地址）。
- **示例：** 代币余额不存储在智能合约中，而是存储在Token Program拥有的账户中。
- Programs 程序默认可升级（但可以锁定以实现不变）。

![System Program - Solana SPL Token Program](https://www.quicknode.com/guides/assets/images/program-token-1c8984f3e0f41276b7414fd36f1996b0.png)

#### Data Storage: Mappings vs. Accounts 数据存储：映射与账户

##### Ethereum: Uses Mappings for Storage 

- **Solidity allows mappings Solidity 允许映射** （例如 `mapping(address => uint256) balances;` ）在合约内存储数据。

##### Solana: Uses Accounts Instead of Mappings 

- Solana 中没有与映射直接对应的机制。相反，PDAs（程序派生地址）充当存储结构化数据的确定性账户。
- **示例：** 您不需要将代币余额映射到地址，而是创建一个存储用户余额的 PDA 帐户。

> 不要从映射的角度来思考，而要从创建单独的帐户的角度来思考，这些帐户保存每个用户或实体的必要数据。

#### Function Execution: Message Sender vs. Explicit Accounts 函数执行：消息发送者 vs. 显式账户

##### Ethereum: msg.sender and tx.origin Handle Authorization 

- 在 ETH 中，合约会自动知道谁通过 `msg.sender` 调用该函数，以及谁是通过 `tx.origin` 发起交易的用户。

##### Solana: Explicit Account Passing Require 需要明确传递账户

- Solana 程序没有内置的 msg.sender 等效项。相反，您必须明确传递程序将与之交互的帐户。
- **示例：** 要将代币从一个用户转移到另一个用户，交易必须包括：
  - **The Token Program accoun  代币程序账户** ：这是执行代币转移逻辑的程序。
  - **The sender’s token account 发送者的代币账户** ：该账户持有发送者的代币，并将被扣款。
  - **The recipient’s token account 接收者的代币账户** ：该账户将接收转移的代币。
  - **The sender’s main wallet account 发送者的主钱包账户** ：该账户支付交易费用，通常是交易的签署者。
- 这种明确性确保 Solana 运行时确切知道涉及哪些帐户以及它们扮演什么角色（例如，可读、可写或签名者）。

> 在设计 Solana 程序时，请务必考虑账户的概念。您必须传入程序与之交互的每个账户。



#### Tokens 代币标准：ERC-20、ERC-721 等与 SPL 代币

##### Ethereum: Each Token Has Its Own Contract  每个代币都有自己的合约

- ETH对每种代币都有不同的代币标准（例如 ERC-20、ERC-721 等）。每种代币都遵循相应的标准，但每个代币都是一个单独的智能合约，需要为每个新代币部署新的合约。

##### Solana: One Token Program, Many Tokens  一个代币程序，多种代币

- Solana 有一个集中式[Token Program](https://github.com/solana-program/token/tree/main/program)，用于管理所有 SPL（Solana Program Library）代币。
- 您无需为每个代币部署新合约（如ETH上的 ERC-20），而是在代币程序下创建一个 mint account 铸币账户。
- Token Program处理所有核心代币功能，包括：
  - Creating new tokens (mint accounts) 创建新代币（铸币账户）
  - Minting new tokens into accounts 将新代币铸造到账户中
  - Burning tokens 销毁代币
  - Transferring tokens between accounts 在账户之间转移代币
- 每个用户的代币余额都存储在一个专用代币账户中——*[ 关联代币账户 (Associated Token Account ATA)，](https://www.quicknode.com/guides/solana-development/spl-tokens/how-to-transfer-spl-tokens-on-solana#associated-token-accounts?utm_source=internal&utm_campaign=guides&utm_content=solana-development-for-evm-developers)* 与用户的钱包地址和代币的铸币地址绑定。
- Solana 还推出了 [SPL Token 2022](https://www.quicknode.com/guides/solana-development/spl-tokens/token-2022/overview?utm_source=internal&utm_campaign=guides&utm_content=solana-development-for-evm-developers) ，它扩展了原始 Token 程序，增加了metadata 元数据、transfer guards, 转移保护、transfer fees 转移费用等高级功能。

虽然本指南并未涵盖所有差异，但从ETH过渡到 Solana 时需要牢记以下一些关键区别。

接下来，我们将探索 Solana 入门的基础知识，包括钱包和 CLI 工具。



## Wallets & CLI Basics 钱包和 CLI 基础知识

虽然可以使用库以编程的方式创建钱包，但ETH开发者通常使用基于浏览器的钱包（例如 MetaMask 或 Rabby）来创建钱包。这些钱包会生成一个外部拥有账户 (Externally Owned Account  EOA)，允许用户签署交易并管理资金。

在 Solana 上，您可以使用浏览器扩展程序（例如 [Phantom](https://phantom.com/) 、 [Solflare](https://solflare.com/) ）或通过命令行（ [Solana CLI](https://solana.com/docs/intro/installation#install-the-solana-cli) ）创建钱包。

### Creating a Wallet on Solana 

#### 选项 1：使用浏览器钱包（Phantom、Solflare、Backpack）

- 安装钱包。查看[本指南](https://www.quicknode.com/guides/solana-development/wallets/intro-to-solana-wallets?utm_source=internal&utm_campaign=guides&utm_content=solana-development-for-evm-developers)了解更多详情。
- 创建一个新的钱包并备份您的助记词。
- 使用此钱包签署交易并与 Solana dApps 交互。

####  选项 2：使用 Solana CLI 创建钱包

如果您想要一个用于开发目的的钱包，您可以使用 CLI 生成密钥对：

```bash
solana-keygen new --outfile ~/.config/solana/id.json
```

这将创建一个密钥对 JSON 文件 ( `id.json` )，其中包含数组格式的私钥。此外，您还将获得一个助记词，可用于恢复密钥对。

一定不要泄漏你的密钥。

确保您的 `id.json` 文件安全且私密。切勿分享其内容或助记词，因为它们会授予您钱包和资金的完全访问权限。



### Importing & Exporting Wallets 

I如果您在浏览器扩展程序（即 Phantom）中有一个钱包，但想要在 CLI 中使用它，则需要从扩展程序中导出其私钥。

我们将介绍如何将 Phantom 钱包导出到 Solana CLI 以及反之亦然。

#### 将 Phantom 钱包导出到 Solana CLI

1. 打开 Phantom 并找到导出助记词的选项。

2. 复制短语（一系列单词）。

3. 运行以下命令：

   ```bash
   solana-keygen recover 'prompt://?key=0/0' -o my-wallet.json
   ```

4. 出现提示时输入您的助记词。

5. 密钥对将被导入到 Solana CLI。

#### 将 Solana CLI 钱包导入 Phantom

1. 在文本编辑器中打开 `id.json` 文件。
2. 复制文件的内容（类似于 `[12, 34, 56, ...]` ）。
3. 在 Phantom 中，找到**导入私钥**并粘贴复制的值。

这会将您的 CLI 生成的钱包恢复到 Phantom。



### 设置 Solana CLI 和 RPC 配置

开发者通过 RPC 端点与 Solana 网络交互。您可以配置与哪个网络交互（主网、测试网或开发网）。

为了加快开发速度，QuickNode 为 Solana 提供了 RPC 端点。您可以使用这些 RPC 与 Solana 网络进行交互，而无需设置自己的节点或使用公共 RPC。要获取您的 RPC 端点，请[在此处](https://www.quicknode.com/signup?utm_source=internal&utm_campaign=guides&utm_content=solana-development-for-evm-developers)注册一个免费帐户。

检查当前配置：

```bash
solana config get
```

设置你的 RPC 端点（例如，devnet）：

```bash
solana config set --url devnet
```

验证您的钱包地址：

```bash
solana address
```

空投测试 SOL 用于开发：

```bash
solana airdrop 2
solana balance
```

这将为您提供 2 个 SOL，用于在 Devnet 上进行测试。您也可以使用 [QuickNode Faucet](https://faucet.quicknode.com/drip?utm_source=internal&utm_campaign=guides&utm_content=solana-development-for-evm-developers) 获取测试 SOL。

欲了解更多信息，请查看 [Solana 上空投测试 SOL 的完整指南 ](https://www.quicknode.com/guides/solana-development/getting-started/a-complete-guide-to-airdropping-test-sol-on-solana?utm_source=internal&utm_campaign=guides&utm_content=solana-development-for-evm-developers)。



## 原文链接

https://www.quicknode.com/guides/solana-development/getting-started/solana-development-for-evm-developers