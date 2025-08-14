# 004 Solana Airdropping Test SOL 

在 Solana 上构建代码非常有趣。为了在生产环境之前测试代码，通常最佳做法是在 Localhost、Devnet 或 Testnet 上运行。这些环境可以安全地测试代码，而不会花费真实代币。但是，要在这些环境中运行，您需要在该集群上拥有 SOL 代币。在本指南中，我们将介绍五种将测试 SOL 空投到您钱包的方法：

- 使用 Solana 的命令行界面 (Solana CLI)
- 使用 Solana 的 JavaScript API
- 使用 QuickNode 的多链水龙头
- 使用 Solana 水龙头

#### What You Will Need 

- 已安装 [Solana CLI](https://docs.solana.com/cli/install-solana-cli-tools) 最新版本
- 已安装 [Nodejs](https://nodejs.org/en/) （16.15 或更高版本）
- 基本的 JavaScript 经验
- 现代网络浏览器（例如 [Google Chrome](https://www.google.com/chrome/) ）
- Solana 钱包（例如 [Phantom](https://phantom.app/) ）



## Clusters 集群

在开始空投之前，我们先快速回顾一下 Solana 的集群，因为我们在申请空投时需要指定一个特定的集群。
“Solana 集群是一组协同工作的验证器，用于处理客户端交易并维护账本的完整性。多个集群可以共存。”
事实上，Solana 维护着三个集群，每个集群都有不同的用途：

- **Mainnet Beta 主网 Beta 版** ：生产环境，无需许可，使用真实代币（注意：**无法在主网 Beta 集群上请求 SOL 空投**）
- **Devnet 开发网**：应用开发者测试 Solana 应用的平台（Devnet 上的代币并非真实存在，且不具备任何经济价值）。Devnet 通常运行与主网 Beta 相同的软件版本。
- **Testnet 测试网** ：Solana 核心贡献者和验证者对新更新和功能进行压力测试的环境，重点是测试网络性能（测试网上的代币不是真实的，也没有财务价值）

[来源：docs.solana.com/clusters](https://docs.solana.com/clusters)


除了上面概述的三个集群之外，Solana CLI（我们稍后会详细介绍）还允许您启动本地测试验证器，以在您的机器上运行完整的区块链集群。本地集群将使您能够快速部署和测试程序，并且与 Devnet 和 Testnet 一样，Localhost 代币并非真实存在。如果您想了解更多信息，请查看 [Solana 的本地开发快速入门指南 ](https://docs.solana.com/getstarted/local)。

要在任何集群上执行交易，您需要该集群上的 SOL 来支付交易费和租金。



## 方法 1 - 通过 Solana CLI 进行空投测试 SOL

第一种方法是使用 Solana CLI 请求空投。如果您尚未安装，请按照 [docs.solana.com/cli/install-solana-cli-tools](https://docs.solana.com/cli/install-solana-cli-tools) 上适合您操作环境的说明进行操作。为确保安装成功，请打开一个新终端并输入：

```sh
solana --version
```

您应该看到类似这样的内容：

![img](https://www.quicknode.com/guides/assets/images/0-6f22c4aaec6101b1a57667174096cae6.png)

一切准备就绪！你只需要准备好你的钱包地址——你可以直接从 Phantom 复制：

![img](https://www.quicknode.com/guides/assets/images/1-783a21839c5d72146821890b11e6a024.png)

通过调用 **airdrop** 子命令请求空投 1 SOL，传入要空投的 SOL 数量和目标钱包地址：

```sh
solana airdrop 1 YOUR_PHANTOM_WALLET_ADDRESS -u devnet # for Devnet
# or 
solana airdrop 1 YOUR_PHANTOM_WALLET_ADDRESS -u testnet # for Testnet
# or 
solana airdrop 1 YOUR_PHANTOM_WALLET_ADDRESS -u localhost # for Localhost (local validator must be running)
# or 
solana airdrop 1 YOUR_PHANTOM_WALLET_ADDRESS -u https://example.solana.quiknode.pro/000000/
```

注意：目前，devnet 上的空投限制为每次请求 2 sol，testnet 上的空投限制为每次请求 1 sol - 两者都可能受到每日限制

您应该看到交易和签名确认如下：

![img](https://www.quicknode.com/guides/assets/images/2-1200470bf71ebf4bdf6832617cfd6f58.png)

您可以在 Phantom 中查看余额，点击左上角的图标，选择“开发者设置”，然后选择“更改网络”。然后选择您请求的相同网络（例如“Devnet”）：

![img](https://www.quicknode.com/guides/assets/images/3-faa99307e1046fb531438eedb4ddaf64.png)

![img](https://www.quicknode.com/guides/assets/images/4-a01ae1cd641af1382c3a91c23ddcf620.png)

如果您在使用 Solana CLI 时遇到困难，请尝试运行 **solana airdrop -h** 来查看帮助选项菜单。



## 方法 2 - 使用 Solana 的 JavaScript API 进行空投测试 SOL

Solana Web3 允许您使用 JavaScript、cURL 或 Python 进行 RPC 调用。Solana Web3 包含一个方法 **requestAirdrop** ，用于将开发令牌投放到适用的集群上。我们将在[此处](https://www.quicknode.com/docs/solana/requestAirdrop)介绍 JavaScript 调用——如果您想使用 Python 或 cURL 进行操作，请查看我们的 Solana 文档 。

在终端中创建一个新的项目目录和文件 **airdrop.js** ，使用以下命令：

```sh
mkdir airdrop_sol
cd airdrop_sol
echo > airdrop.js
```

安装 Solana Web3 依赖项：

```sh
yarn init -y
yarn add @solana/web3.js@1
```

或者

```sh
npm init -y
npm install --save @solana/web3.js@1
```

在选择的代码编辑器中打开 **airdrop.js** ，并添加以下代码：

```javascript
const SOLANA = require('@solana/web3.js');
const { Connection, PublicKey, LAMPORTS_PER_SOL, clusterApiUrl } = SOLANA;
const SOLANA_CONNECTION = new Connection(clusterApiUrl('devnet'));
const WALLET_ADDRESS = 'YOUR_PHANTOM_WALLET_ADDRESS'; //👈 Replace with your wallet address
const AIRDROP_AMOUNT = 1 * LAMPORTS_PER_SOL; // 1 SOL 

(async () => {
    console.log(`Requesting airdrop for ${WALLET_ADDRESS}`);

    const signature = await SOLANA_CONNECTION.requestAirdrop(
        new PublicKey(WALLET_ADDRESS),
        AIRDROP_AMOUNT
    );

    console.log(`Tx Complete: https://explorer.solana.com/tx/${signature}?cluster=devnet`)
})();
```

让我们来看看这个脚本：

1. 导入 Solana Web3 库并解构它以获取必要的类、方法和常量。
2. 创建一个新变量 **SOLANA_CONNECTION** 来建立与 Devnet 集群的连接。
3. 粘贴您的钱包地址。
4. 在 lampors 中将 **AIRDROP_AMOUNT** 设置为等于 1 SOL（使用 **LAMPORTS_PER_SOL** 常量）。
5. 使用 Solana Web3 的 **requestAirdrop** 方法将指定数量的 Lamport 投递到我们定义的 **WALLET_ADDRESS** 。此调用将返回一个我们定义为**签名的**交易 ID。
6. 我们在 Solana 浏览器上记录了该交易的链接。

注意：您可以选择在连接上使用 `getSignatureStatuses` 方法来轮询交易状态并等待其完成。

从您的终端运行您的脚本：

```sh
node airdrop
```

您应该看到类似这样的内容：

![img](https://www.quicknode.com/guides/assets/images/5-ab0c55ae965b183af60fe5651e9ece0c.png)



## 方法 3 - 通过 QuickNode 水龙头空投测试 SOL

如果您不想运行自己的脚本，不用担心！QuickNode 最近推出了一款易于使用的多链水龙头。您只需前往 [QuickNode 多链水龙头](https://faucet.quicknode.com/solana/devnet?utm_source=internal&utm_campaign=guides)并连接您的钱包（或粘贴您的钱包地址）。请确保已选择 Solana 开发网或测试网：

![img](https://www.quicknode.com/guides/assets/images/6-70707cc9e43e3ae31690df3e6fa5565d.png)

您可以选择发推文申请更大额度的空投，或者直接点击“不用了，谢谢，只给我 1 个 SOL”。就这样！稍等片刻，您的交易就会确认，现在您有一个无需代码的解决方案，可以在您的钱包里获得一些开发/测试 SOL！



## 方法4  - 通过 Solana 水龙头空投测试 SOL

Solana 发布了 Solana Faucet，旨在让 Devnet SOL 更容易获取。该水龙头允许用户每小时最多领取 5 个 Devnet SOL（2 次）。请访问 https://faucet.solana.com/ 领取您的 Devnet SOL。

![Solana Faucet](https://www.quicknode.com/guides/assets/images/7-0949014af812f7acf8aa546dc65e5318.png)

## 原文链接

https://www.quicknode.com/guides/solana-development/getting-started/a-complete-guide-to-airdropping-test-sol-on-solana