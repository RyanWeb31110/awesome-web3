# 006 Solana 账户模型简介

[Solana](https://solana.com/) 账户模型是 Solana 生态系统的关键组成部分，对于开发者来说，尤其是从其他区块链过渡过来的开发者来说，这可能是最难理解的概念之一。为了高效地在 Solana 区块链上工作，充分理解账户模型至关重要。首先，让我们定义什么是账户，并探索不同类型的账户以及如何创建和与它们交互。

## What is an Account? 

账户是指 Solana 区块链上存储数据的位置。与以太坊等其他区块链相比，Solana 区块链的独特之处在于其数据的存储和管理方式。以下是不同的账户类别：

\- **Program Account 程序账户** - 这些账户存储可执行代码，和以太坊智能合约的等效内容。
\- **Storage Account 存储帐户** - 这些帐户存储与程序相关的数据。
\- **Token Account  代币账户** - 这些账户跟踪代币的账户余额并允许在账户之间转移或接收代币。

在 Solana 区块链中，程序与程序的数据和状态是分离的。两者被分配了独立的账户，但又相互关联。（与以太坊相比，智能合约及其数据在区块链上位于同一位置。）如果您要编写一个程序来统计程序进行的代币转账次数，则需要创建用于进行转账的程序以及另一个用于存储转账次数的账户。

![Two boxes with one box titled Program account and another titled Data Account with an arrow from Data Account to Program Account. ](https://www.quicknode.com/guides/assets/images/0-769417a502e1675a4ba31cca4075e6d4.png)

[Image Source 图片来源](https://solanacookbook.com/assets/account_example.5b70d95a.jpeg)

举一个传统金融领域的例子，你可以把程序想象成一张借记卡，你的银行账户是一个存储账户，你的账户余额是一个代币账户。虽然它们彼此关联，但它们位于不同的位置。如果你丢失了借记卡，你不会丢失你的银行账户（但可能会丢失里面的资金）。你的卡也是唯一一张可以通过购物来更改账户余额的卡。同样，与存储账户关联的程序也是唯一可以更改数据状态的程序。



## Types of Accounts

Solana 区块链上有两种类型的账户： **可执行账户**和**不可执行账户** 。程序账户是可执行账户，存储不可变代码。程序代码首先用 Rust 或 C/C++ 编写，然后通过 [LLVM compiler infrastructure LLVM 编译器基础结构](https://llvm.org/)编译为字节码。

数据和代币余额存储在不可执行账户中，因为其数据可以被更改。为了控制谁可以更改这些数据，不可执行账户会分配一个所有者程序地址。其他程序可以读取其他账户的数据，但如果它们尝试修改该数据，交易就会失败。



## Rent 租金

将所有这些数据存储在单独的账户中并非免费，而且会产生一些费用。对于开发者来说，这些费用（称为[**租金** ](https://docs.solana.com/implemented-proposals/rent)）可以通过 Lamport 支付。Lamport 是 Solana 代币 SOL 的一小部分，用于在 Solana 区块链上进行小额支付。租金根据账户的存储大小计算。存储的数据量越大，租金越高。

Solana 区块链的每个周期 (epoch) 结束时都会收取租金。周期是指 the leading validator 领先验证者仍然有效并能够生成交易区块的时间。您可以在 [Solana 浏览器 (Explorer)](https://explorer.solana.com/) 中查看当前和过去周期的数据。

在撰写本文时，一个 epoch 大约持续 2 天。与现实生活中的情况类似，如果一个账户余额为零，且无力支付租金，该账户就会被从区块链中移除。

账户持有至少相当于两年租金的代币余额即可免租。估算租金成本的一种简单方法是通过 Solana CLI 使用 [solana rent](https://docs.solana.com/cli/usage#solana-rent) 命令。输入账户大小（以字节为单位），您将看到每字节、每个周期的租金以及账户免租的最低金额：

![A terminal window showing the Solana rent command being run](https://www.quicknode.com/guides/assets/images/1-976dc6adb75f1d8f3faaca4bb93843d0.png)

## How to Create an Account 

要在 Solana 上创建帐户，客户端需要生成一对密钥（公钥和私钥）。然后，客户端使用 [SystemProgram::CreateAccount](https://docs.solana.com/developing/programming-model/accounts#creating) 调用注册公钥，并分配该帐户所需的数据存储大小。目前，此大小无法更改，大小限制为 10 MB。如果需要更多存储空间，程序可以将数据从一个帐户复制到另一个具有更大容量的帐户。

创建帐户时，需要为该帐户指定一个所有者。只有帐户所有者才能修改帐户中存储的数据。帐户创建后的默认所有者称为“ System Program 系统程序”。

系统程序是 Solana 上的[ native program 原生程序 ](https://docs.solana.com/developing/runtime-facilities/programs)，负责创建帐户、分配帐户数据以及将帐户所有权分配给连接的程序。原生程序是 Solana 上所有验证器都必须运行的程序。

系统程序还负责为其指定所有者的账户进行 Lamport 转账。如果用户创建一个账户来存储代币余额，则该代币的转账将由系统程序处理。用户将使用其私钥签署转账指令，系统程序将处理从发送方扣除代币并记入接收方账户的操作。



## Interacting with Accounts 

由于程序代码和程序存储的数据是独立的帐户，因此任何程序都可以读取其他帐户的数据。任何程序都可以向帐户添加 Lamport，但只有所有者可以删除它们。这在构建可能需要与不属于自己的帐户交互的程序时非常有用。

当读取帐户时，您将看到返回以下数据：

![Solana account command run in the Terminal](https://www.quicknode.com/guides/assets/images/2-8f036bd92bb272d4b3e8fd76969362a9.png)

[Image Source 图片来源](https://medium.com/@jorge.londono_31005/understanding-solanas-mint-account-and-token-accounts-546c0590e8e)

以下是该数据的细分：
\- **Public Key  公钥** - 分配给此帐户的公钥
\- **Balance 余额** - 此账户拥有的 SOL 数量
\- **Owner 所有者** - 拥有此帐户所有权的程序的地址
\- **Executable 可执行** - 这是否是一个可执行帐户
\- **rent_epoch** -  此账户将支付租金的下一个时期
\- **Length 长度** - 账户的大小



## Conclusion 

在 Solana 上，创建账户并与之交互是一切操作的根本。程序与其使用的数据分离，是代码和状态管理的独特方法。理解所有权规则以及程序如何在这些规则下运作，对于在 Solana 上成功开发至关重要。为了将这些概念付诸实践，请查看我们的[其他 Solana 教程 ](https://www.quicknode.com/guides/)。



## 原文链接：

https://www.quicknode.com/guides/solana-development/getting-started/an-introduction-to-the-solana-account-model