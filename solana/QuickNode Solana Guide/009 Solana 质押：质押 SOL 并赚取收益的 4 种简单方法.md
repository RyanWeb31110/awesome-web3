# 009 Solana 质押：质押 SOL 并赚取收益的 4 种简单方法

质押原生代币是区块链网络激励验证者维护网络安全的常用方式。验证者将获得部分网络原生代币作为其工作的奖励。质押也是代币持有者通过持有代币赚取被动收入的绝佳方式。在本指南中，我们将介绍在 Solana 上质押 SOL 的四种方式：

- 使用 Solana 钱包（例如 [Phantom](https://phantom.app/) 或 [Solflare](https://solflare.com/) ）
- 使用 Solana 的命令行界面 (Solana CLI)
- 使用 Solana 的 JavaScript API
- 使用 Liquid Staking

### What You Will Need

- 已安装 [Solana CLI](https://docs.solana.com/cli/install-solana-cli-tools) 最新版本
- 已安装 [Nodejs](https://nodejs.org/en/) （16.15 或更高版本）
- 基本的 JavaScript 经验
- 现代网络浏览器（例如 [Google Chrome](https://www.google.com/chrome/) ）
- Solana 钱包（例如 [Phantom](https://phantom.app/) 或 [Squad](https://squads.so/) ）
- 一些 SOL 代币用于质押和账户租金、交易费



## 什么是 Staking 质押？

作为 Solana 安全和激励机制不可或缺的一部分，质押功能使验证者能够将一定数量的 SOL 代币锁定在质押账户中，并通过质押证明 (PoS) 机制参与共识。验证者参与交易确认和区块生产，并获得与其质押金额成比例的奖励。这种机制通过让任何恶意行为的验证者面临代币损失的风险来增强网络的安全性。

在 Solana 上创建一个质押账户需要生成一个新的质押账户，然后将 SOL 代币从已充值的钱包转移到该账户。在创建时可以分配质押和提现权限，也可以稍后修改这些权限以获得最大的灵活性。每个质押账户只能委托给一个验证者；不过，用户/权限拥有者可以拥有多个质押账户，从而将他们的质押分散到多个验证者。对质押的更改（例如激活、委托和取消激活）将在下一个周期（一个周期约为 2 天）生效。这种渐进的过程有助于网络的稳定性和可预测性，防止质押分配出现突然且剧烈的变化。

有关质押的更多信息，请查看 [Solana 文档 ](https://docs.solana.com/staking)。如果您需要帮助寻找验证者，以下是一些资源可以帮助您找到：

- https://stakewiz.com/
- https://www.validators.app/

让我们开始质押一些 SOL！



## Method 1 - Using a Wallet

质押 SOL 最简单的方法之一就是直接在钱包中质押。大多数主流钱包都提供质押界面，只需点击几下即可将 SOL 委托给验证者。以下是一些常用钱包的操作方法：

- [Phantom ](https://www.quicknode.com/guides/solana-development/getting-started/how-to-stake-sol#stake-with-phantom)
- [Solflare ](https://www.quicknode.com/guides/solana-development/getting-started/how-to-stake-sol#stake-with-solflare)
- [Backpack ](https://www.quicknode.com/guides/solana-development/getting-started/how-to-stake-sol#stake-with-backpack)
- [Squads ](https://www.quicknode.com/guides/solana-development/getting-started/how-to-stake-sol#stake-with-squads)

### Stake with Phantom

在 Phantom 中质押非常简单。打开你的 Phantom 钱包扩展程序：

![Phantom Wallet](https://www.quicknode.com/guides/assets/images/phantom-1-3fb8b6de382f3fe1b27937f384c986ac.png)

您应该会看到一些关于 Solana 的详细信息。滚动到看到“开始赚取 SOL”按钮。点击它即可开始质押。

![Phantom Staking](https://www.quicknode.com/guides/assets/images/phantom-2-c4329cdd2a3a8f50c0006c0f73e58806.png)

从呈现给您的选项中选择“Native Staking”：

![Phantom Staking Options](https://www.quicknode.com/guides/assets/images/phantom-3-1d3f3e4a3892194ffb06ba6462815958.png)

在搜索栏中搜索“QuickNode”即可找到我们的 validator 验证器（或您选择的验证器名称）。 *您也可以通过验证器地址（例如 `5s3vajJvaAbabQvxFdiMfg14y23b2jvK6K2Mw4PYcYK` ）进行搜索。* 点击您想要的验证器。系统将提示您输入质押金额。输入金额后，点击“质押”确认。片刻之后，您将看到“SOL 质押成功！”消息：

![Phantom Staking Validator](https://www.quicknode.com/guides/assets/images/phantom-4x-f6dabe41aa763a461f84d813f5403d72.png)

您的质押将在下一个周期开始时完成激活。您可以再次点击您的 Solana 余额，在钱包（或任何主要的 Solana 浏览器）中查看您的质押状态。您应该会在“Staking”下看到您的质押：

![Phantom Staking Success](https://www.quicknode.com/guides/assets/images/phantom-7-a635f7c48d84cf795c82dfb29212c307.png)

您可以打开它来查看有关您的质押的更多详细信息。您还可以查看您在 [Solana Beach 的](https://solanabeach.io/address/YOUR_WALLET/stakes) stake。

![Phantom Staking Details](https://www.quicknode.com/guides/assets/images/phantom-8-f1d5de41f4146f77876e39e2c3172afb.png)

就这样！您现在就可以使用 Phantom 质押 SOL 了。



### Stake with Solflare 

Solflare 让质押变得轻而易举。打开您的 Solflare 钱包，然后点击您的 Solana 余额。您将被引导至 Solana 详情页面：

![Solflare Wallet](https://www.quicknode.com/guides/assets/images/sf-0-793e74558fb52860f0e656225fd91e0b.png)

您应该会看到一个“Stake SOL” 模块。点击“Stake”：

![Solflare Wallet Stake Button](https://www.quicknode.com/guides/assets/images/sf-1-4bb90b20dd97d0acb0cc7ab78923c616.png)

搜索您选择的验证器（例如“QuickNode”或“5s3vajJvaAbabQvxFdiMfg14y23b2jvK6K2Mw4PYcYK”），然后从下拉列表中选择所需的验证器：

![Solflare Wallet](https://www.quicknode.com/guides/assets/images/sf-2-8a082adb6eb21cee0e18cfda78578911.png)

输入您想质押的金额后，点击“Stake”。系统将提示您确认交易：

![Solflare Wallet](https://www.quicknode.com/guides/assets/images/sf-3-45380c0876aa9b449a9b421361704eee.png)

然后，您可以返回到 Solana 余额详情屏幕，您将看到一个新的“Staking”余额：

![Solflare Wallet](https://www.quicknode.com/guides/assets/images/sf-4-fa525431ff47b176f3a6de290623d3b7.png)

单击它可查看正在激活的质押的详细信息：

![Solflare Wallet](https://www.quicknode.com/guides/assets/images/sf-5-9c14b3e24aac2fb9c2bad9b11fc7913e.png)

太棒了！您现在正在 Solflare 质押 SOL。🔥



### Stake with Backpack

Backpack 是另一款流行的 Solana 钱包，它允许您在不离开钱包的情况下质押您的 SOL。打开您的 Backpack 钱包。在主屏幕上点击您的 Solana 余额：

![Backpack Wallet](https://www.quicknode.com/guides/assets/images/bp-1-8539dff482845704bfc83a5d396ae3eb.png)

您将看到您的 SOL 余额，并带有“ ⭐️ Stake SOL to Earn ”选项。点击此按钮：

![Backpack SOL Balance](https://www.quicknode.com/guides/assets/images/bp-2-541a5d6d9d68eaa480c662834a4a6afa.png)

您将看到一个选项供您选择所需的验证器。点击“Other Validators”即可搜索特定的验证器：

![Backpack Validator Search](https://www.quicknode.com/guides/assets/images/bp-3-4f4eaa1e61e2828ad22375151a8fce31.png)

搜索您想要的验证器（例如“QuickNode”或“5s3vajJvaAbabQvxFdiMfg14y23b2jvK6K2Mw4PYcYK”）并选择它：

![Backpack Validator Search Results](https://www.quicknode.com/guides/assets/images/bp-4-7a0c626f5a1f4c92a02afb2afdef56f1.png)

您将返回到“New Stake”页面，其中包含所选的验证器。输入您想要质押的金额，然后点击“Stake”：

![Backpack QuickNode Validator](https://www.quicknode.com/guides/assets/images/bp-5-40649a76c3abfe8fc833a7609044ef2f.png)

您将被重定向到 SOL 余额页面，在该页面中您将看到“Staking”下列出您的新帐户：

![Backpack SOL Balance](https://www.quicknode.com/guides/assets/images/bp-6-a22944082b84739ab0513598ff10320d.png)

您的 bags（🎒）现在已经装满了质押的 SOL！



### Stake with Squads 

[Squads](https://www.quicknode.com/guides/solana-development/3rd-party-integrations/multisig-with-squads) 是 Solana 上的开源智能合约钱包标准，允许团队和个人安全地管理其数字资产。Squads 用户可以直接在 Squads 用户界面 (UI) 中质押他们的 SOL。让我们来了解一下整个流程。

打开你的 Squads 钱包，并使用授权发起人的钱包连接到 Squads。点击左侧边栏的“Stake”标签。屏幕底部有一个搜索栏，你可以在其中搜索验证者。搜索你想要的验证者（例如，“QuickNode”或“5s3vajJvaAbabQvxFdiMfg14y23b2jvK6K2Mw4PYcYK”）：

![Squads Wallet](https://www.quicknode.com/guides/assets/images/squads-1-0c6f697e633d1bfd9508a77b4338c5e1.png)

点击您想要的验证者。系统将提示您输入想要质押的金额：

![Squads Stake Validators](https://www.quicknode.com/guides/assets/images/squads-2-b3ae96fa6d51b4a8bae982a5d3e03123.png)

输入金额后，点击“Stake”。随后系统会提示您发起交易，以获得您的 Squads 的批准：

![Squads Confirmation](https://www.quicknode.com/guides/assets/images/squads-3-fd65a24eb81a822714f7013bfe7ea169.png)

前往“Transactions”选项卡查看正在进行的交易。将提案与您的 Squads 成员分享以获取批准。当达到您的投票门槛时，您将能够在您的 Squads 中“Execute”该交易：

![Squads Transaction](https://www.quicknode.com/guides/assets/images/squads-4-a1c4b6309069e4fb24ac47a8f10eab95.png)

您现在应该可以返回“Stake”选项卡并查看您质押的 SOL：

![Squads Staked SOL](https://www.quicknode.com/guides/assets/images/squads-5-38d1ecae02ee06e3d689e79157d0bd6e.png)

现在，您正在保护自己的资产，并利用“Squads”来保障Solana网络的安全！



## Method 2 - Using Solana CLI

让我们首先创建一个项目目录，其中包含一个用于 CLI 的文件夹和一个用于 JavaScript API 的文件夹（我们稍后会使用）：

```text
mkdir -p solana-staking/cli && mkdir -p solana-staking/web3js
cd solana-staking/cli
```

要使用 Solana CLI 进行质押，您需要一个 paper wallet 纸钱包。如果您还没有纸钱包，可以在终端中输入以下命令创建一个：

```sh
solana-keygen new --no-passphrase -o wallet.json
```

这将创建一个新的密钥对并将其保存到名为 **wallet.json** 的文件中。新的公钥将显示在你的终端中：

```sh
Generating a new keypair
Wrote new keypair to wallet.json
============================================================================
pubkey: Fzts...kjKb
===========================================================================
```

确保你的纸钱包已存入 SOL。如果你刚刚创建了纸钱包，你可以将资金从任何钱包转移到终端显示的地址。

接下来，我们需要创建一个质押账户。让我们创建另一个名为 **stake-account.json** 的密钥对：

```sh
solana-keygen new --no-passphrase -s -o stake-account.json 
```

注意：我们在这里使用 `-s` 标志，因为我们不需要此帐户的助记词。

现在我们可以使用 `create-stake-account` 命令，使用 **stake-account.json** 密钥对创建一个质押账户：

```sh
solana create-stake-account stake-account.json 100 \
    ----from wallet.json \
    --stake-authority wallet.json --withdraw-authority wallet.json \
    --fee-payer wallet.json \
    --url mainnet-beta  # or testnet, devnet, localhost, or your QuickNode URL
```

请确保将 `--url` 标志替换为正确的网络。如果您使用 **mainnet-beta** ，则真实资金将从您的钱包中转出。为了获得最佳效果，我们建议您使用 QuickNode 端点——您可以[在此处](https://www.quicknode.com/signup?utm_source=internal&utm_campaign=guides&utm_content=how-to-stake-sol)注册免费帐户。
您现在应该能够通过运行以下命令来验证您的帐户是否已创建：

```sh
solana stake-account stake-account.json
```

您应该会在终端中看到您的stake账户信息：

```sh
Balance: 100 SOL
Rent Exempt Reserve: 0.00228288 SOL
Stake account is undelegated
Stake Authority: FztsbEJLCmdaeQWuJKZrQ8MjN1j3yVMftynzA5e8kjKb
Withdraw Authority: FztsbEJLCmdaeQWuJKZrQ8MjN1j3yVMftynzA5e8kjKb
```

您会注意到，我们的stake仍然需要委托给验证者 - 让我们委托它。

首先，我们需要找到一个验证者的地址。您可以[在这里](https://www.validators.app/validators)找到验证者列表，或者在终端中输入 `solana validators -um` （或 `-ud` 表示 devnet）。找到一个验证者并复制他们的**投票账户**地址。

![Vote Account](https://www.quicknode.com/guides/assets/images/vote-d230647c48f750f40371d5fdafb58ce0.png)

我们将使用 QuickNode 的验证器（ *主网上*的 *5s3vajJvaAbabQvxFdiMfg14y23b2jvK6K2Mw4PYcYK* ）。

现在您可以通过调用 `solana delegate-stake stake-account.json <VOTE_ACCOUNT>` 委托您的质押：

```sh
solana delegate-stake stake-account.json <VOTE_ACCOUNT> \ # replace <VOTE_ACCOUNT> with your validator's address
    --stake-authority wallet.json \ 
    --fee-payer wallet.json \
    --url https://example.solana-mainnet.quiknode.io/12345 \ # Replace with your Quicknode Endpoint or testnet, mainnet-beta, localhost
    --with-memo "Delegating stake test."
```

您应该会在终端中看到一个签名，确认委托成功。您可以再次运行 `solana stake-account stake-account.json -ud` 来验证您的质押是否已委托：

```sh
Balance: 0.1 SOL
Rent Exempt Reserve: 0.00228288 SOL
Delegated Stake: 0.09771712 SOL
Active Stake: 0 SOL
Activating Stake: 0.09771712 SOL
Stake activates starting from epoch: 529
Delegated Vote Account Address: 5s3vajJvaAbabQvxFdiMfg14y23b2jvK6K2Mw4PYcYK
Stake Authority: FztsbEJLCmdaeQWuJKZrQ8MjN1j3yVMftynzA5e8kjKb
Withdraw Authority: FztsbEJLCmdaeQWuJKZrQ8MjN1j3yVMftynzA5e8kjKb
```

干得好！你的质押应该已经激活，并将在下一个周期开始时开始累积奖励。

**您的质押激活后** ，您可能想要取消质押或从账户中提取部分 SOL。您可以使用 `deactivate-stake` 和 `withdraw-stake` 命令来执行此操作。

要停用您的质押，请运行：

```sh
solana deactivate-stake stake-account.json \
    --stake-authority wallet.json \
    --fee-payer wallet.json \
    --url https://example.solana-mainnet.quiknode.io/12345 \
```

要从您的质押账户中提取部分 SOL，请运行：

```sh
solana withdraw-stake stake-account.json <DESTINATION> <AMOUNT> \
    --withdraw-authority wallet.json  \
    --fee-payer wallet.json \
    --url https://example.solana-mainnet.quiknode.io/12345 \
```

确保将您的 `--url` 替换为您正在使用的适当网络（testnet、mainnet-beta、localhost、devnet 或您的 QuickNode URL）。

现在，您已经掌握了从 Solana 命令行质押 SOL 的所有基础知识。还有一些其他命令我们这里没有介绍。您可以在终端中运行 `solana -h` 来独立探索这些命令，查看所有可用的命令。



## Method 3 - Staking SOL with Solana's JavaScript API 

要开始使用 JavaScript，请导航到我们之前创建的 JS 子目录。在 `cli` 目录中，运行：

```sh
cd ../web3js
```

创建一个启用 .json 导入的 **tsconfig.json** ：

```sh
tsc -init --resolveJsonModule true
```

 安装 SolanaWeb3.js：

```sh
npm install @solana/web3.js@1 # or yarn add @solana/web3.js
```

创建一个名为 **app.ts** 的新文件：

```sh
echo > app.ts
```

让我们遵循上一节中的命名约定，并创建一个名为 **wallet.json** 的新密钥对：

```sh
solana-keygen new --no-passphrase -o wallet.json 
```

我们将在脚本中使用密钥对生成器来创建多个质押账户，而不是为我们的质押账户设置单个密钥对。



### Structure your project 

在您最喜欢的编辑器中打开 `app.ts` 并添加以下代码：

```ts
import { Connection, PublicKey, Keypair, StakeProgram, LAMPORTS_PER_SOL, Authorized, TransactionSignature, TransactionConfirmationStatus, SignatureStatus } from '@solana/web3.js'
import walletSecret from './wallet.json'

const connection = new Connection('http://127.0.0.1:8899', 'confirmed');
const wallet = Keypair.fromSecretKey(new Uint8Array(walletSecret));
const stakeAccount = Keypair.generate();
const validatorVoteAccount = new PublicKey("TBD");

async function confirmTransaction(
    connection: Connection,
    signature: TransactionSignature,
    desiredConfirmationStatus: TransactionConfirmationStatus = 'confirmed',
    timeout: number = 30000,
    pollInterval: number = 1000,
    searchTransactionHistory: boolean = false
): Promise<SignatureStatus> {
    const start = Date.now();

    while (Date.now() - start < timeout) {
        const { value: statuses } = await connection.getSignatureStatuses([signature], { searchTransactionHistory });

        if (!statuses || statuses.length === 0) {
            throw new Error('Failed to get signature status');
        }

        const status = statuses[0];

        if (status === null) {
            await new Promise(resolve => setTimeout(resolve, pollInterval));
            continue;
        }

        if (status.err) {
            throw new Error(`Transaction failed: ${JSON.stringify(status.err)}`);
        }

        if (status.confirmationStatus && status.confirmationStatus === desiredConfirmationStatus) {
            return status;
        }

        if (status.confirmationStatus === 'finalized') {
            return status;
        }

        await new Promise(resolve => setTimeout(resolve, pollInterval));
    }

    throw new Error(`Transaction confirmation timeout after ${timeout}ms`);
}

async function main() {
    try {
        // Step 1 - Fund the wallet
        console.log("---Step 1---Funding wallet");
        await fundAccount(wallet, 2 * LAMPORTS_PER_SOL);
        // Step 2 - Create the stake account
        console.log("---Step 2---Create Stake Account");
        await createStakeAccount({ wallet, stakeAccount, lamports: 1.9 * LAMPORTS_PER_SOL });
        // Step 3 - Delegate the stake account
        console.log("---Step 3---Delegate Stake Account");
        await delegateStakeAccount({ stakeAccount, validatorVoteAccount, authorized: wallet });
        // Step 4 - Check the stake account
        console.log("---Step 4---Check Stake Account");
        await getStakeAccountInfo(stakeAccount.publicKey);
    } catch (error) {
        console.error(error);
        return;
    }
}

async function fundAccount(accountToFund: Keypair, lamports = LAMPORTS_PER_SOL) {

}
async function createStakeAccount({ wallet, stakeAccount, lamports }: { wallet: Keypair, stakeAccount: Keypair, lamports?: number }) {

}
async function delegateStakeAccount({ stakeAccount, validatorVoteAccount }: { stakeAccount: Keypair, validatorVoteAccount: PublicKey }) {

}
async function getStakeAccountInfo(stakeAccount: PublicKey) {

}

main();
```


我们在这里所做的是：

- 从 SolanaWeb3.js 和我们的 wallet.json 文件导入必要的包
- 创建与本地节点的连接（如果愿意，您可以使用不同的集群）
- 为我们的质押账户创建新的密钥对
- 为我们的  validator vote account 验证器投票账户创建一个新的公钥（我们暂时将其保留为“TBD”——当我们启动本地验证器时，我们需要替换它）
- 创建辅助函数来确认交易
- 创建一个 `main` 函数，按顺序运行所有函数
- 创建四个函数，用于为我们的质押账户提供资金、创建、委托和获取相关信息

下面让我们逐一构建我们的功能。



### Step 1 - Fund the wallet

我们需要一些测试 SOL 来实现我们的交易。让我们使用以下代码更新 `fundAccounts` 函数：

```ts
async function fundAccount(accountToFund: Keypair, lamports = LAMPORTS_PER_SOL) {
    const { blockhash, lastValidBlockHeight } = await connection.getLatestBlockhash();
    try {
        const signature = await connection.requestAirdrop(accountToFund.publicKey, lamports);
        const result = await confirmTransaction(connection, signature, 'finalized');
        if (result.value.err) {
            throw new Error(`Failed to confirm airdrop: ${result.value.err}`);
        }
        console.log("Wallet funded", signature);
    }
    catch (error) {
        console.error(error);
    }
    return;
}
```

我们只需传递一个*密钥对* ，并使用 `requestAirdrop` 方法请求 1 SOL（或用户指定的金额）的空投。我们正在等待网络确认（ *finalized 最终确定* ），以确保我们的资金可用于后续步骤。



### Step 2 - Create the stake account 

接下来，我们将创建质押账户。使用以下代码更新 `createStakeAccount` 函数：

```ts
async function createStakeAccount({ wallet, stakeAccount, lamports }: { wallet: Keypair, stakeAccount: Keypair, lamports?: number }) {
    const transaction = StakeProgram.createAccount({
        fromPubkey: wallet.publicKey,
        stakePubkey: stakeAccount.publicKey,
        authorized: new Authorized(wallet.publicKey, wallet.publicKey),
        lamports: lamports ?? LAMPORTS_PER_SOL
    });
    try {
        const { blockhash, lastValidBlockHeight } = await connection.getLatestBlockhash();
        transaction.feePayer = wallet.publicKey;
        transaction.recentBlockhash = blockhash;
        transaction.lastValidBlockHeight = lastValidBlockHeight;
        transaction.sign(wallet, stakeAccount);
        const signature = await connection.sendRawTransaction(transaction.serialize());
        const result = await confirmTransaction(connection, signature, 'finalized');
        if (result.value.err) {
            throw new Error(`Failed to confirm airdrop: ${result.value.err}`);
        }
        console.log("Stake Account created", signature);
    }
    catch (error) {
        console.error(error);
    }
    return;
}
```


让我们分析一下这里的代码：

1. 首先，我们使用 *StakeProgram* 类中的 `createAccount` 方法定义交易。我们传入钱包的公钥、之前生成的新质押账户的公钥、作为授权质押者的钱包公钥，以及我们想要质押的 SOL 数量（默认为 1 SOL）。 *注意：如果您之前没有使用过 `StakeProgram` 类，我们建议您浏览[文档](https://solana-labs.github.io/solana-web3.js/classes/StakeProgram.html)以了解更多可用的方法。*
2. 接下来，我们获取网络的最新区块哈希值和最新有效区块高度。然后，我们在交易中设置费用支付者、最新区块哈希值和最新有效区块高度。
3. 我们使用钱包和质押账户密钥对签署、发送和确认交易。

现在我们有了一个充值好的质押账户！让我们进入下一步，将质押账户委托给验证者。



### Step 3 - Delegate the stake account 

现在我们有了一个已注资的质押账户，我们需要将其委托给验证者。使用以下代码更新 `delegateStakeAccount` 函数：

```ts
async function delegateStakeAccount({ stakeAccount, validatorVoteAccount, authorized }: { stakeAccount: Keypair, validatorVoteAccount: PublicKey, authorized: Keypair }) {
    const transaction = StakeProgram.delegate({
        stakePubkey: stakeAccount.publicKey,
        authorizedPubkey: authorized.publicKey,
        votePubkey: validatorVoteAccount
    });
    try {
        const { blockhash, lastValidBlockHeight } = await connection.getLatestBlockhash();
        transaction.feePayer = authorized.publicKey;
        transaction.recentBlockhash = blockhash;
        transaction.lastValidBlockHeight = lastValidBlockHeight;
        transaction.sign(authorized);
        const signature = await connection.sendRawTransaction(transaction.serialize());
        const result = await confirmTransaction(connection, signature, 'finalized');
        if (result.value.err) {
            throw new Error(`Failed to confirm airdrop: ${result.value.err}`);
        }
        console.log("Stake Account delegated to vote account", signature);
    }
    catch (error) {
        console.error(error);
    }
    return;
}
```

与上一步类似，我们利用 *StakeProgram* 类创建交易。这次我们调用 `delegate` 方法，并传入质押账户的公钥、授权质押者的公钥以及验证者投票账户的公钥。然后，我们使用授权质押者的密钥对对交易进行签名、发送和确认。

我们现在应该有一个已注资并委托的质押账户！让我们再添加一个函数来验证质押账户是否已激活。



### Step 4 - Verify the stake account is activating 

我们可以通过调用 *Connection* 类的 `getStakeAccountInfo` 函数来验证质押账户是否已激活。使用以下代码更新 `getStakeAccountInfo` 函数：

```ts
async function getStakeAccountInfo(stakeAccount: PublicKey) {
    try {
        const info = await connection.getAccountInfo(stakeAccount);
        console.log(`Stake account exists.`);
    } catch (error) {
        console.error(error);
    }
    return;
}
```

我们只需传入质押账户的公钥并记录质押账户的状态即可。现在我们可以调用此函数来验证质押账户是否已激活。



### Run the script 

现在我们已经定义了所有函数，让我们测试一下。

首先，确保本地集群正在运行。如果您在 CLI 质押练习后关闭了集群，或者未完成 CLI 质押练习，则可以通过运行以下命令重新启动集群：

```bash
solana-test-validator
```

在运行脚本之前，我们必须找到验证者的投票账户公钥。我们可以通过运行以下命令来完成此操作：

```bash
solana validators -ul
```

`-ul` 标志将确保我们正在查看本地集群。复制  Vote Account 投票账户公钥：

![Vote Account](https://www.quicknode.com/guides/assets/images/voteAccount-35949a94bd97a166aa7f0e231cf79afd.png)

这是我们本地验证者的投票账户。让我们更新之前定义的 `validatorVoteAccount` 变量：

```ts
const validatorVoteAccount = new PublicKey("YOUR_VALIDATOR_VOTE_ACCOUNT");
```

在第二个终端中，运行以下命令来执行脚本：

```bash
ts-node app.ts
```

您应该看到以下输出：

![Stake Account Activating](https://www.quicknode.com/guides/assets/images/activating-851b680481a8b37b77b046e607cd6374.png)


恭喜！您已成功使用 Solana Web3.js 库创建、注资、委托并激活质押账户。



## Method 4 - Liquid Staking 

流动性质押 (Liquid Staking) 是一种允许用户质押其 SOL 并保持流动性的概念。用户可以质押其 SOL 来换取多种流动性质押代币（例如 mSOL、jitoSOL 等）。这使得用户在获得质押奖励的同时，仍然可以在其他应用中使用其 SOL。想要深入了解流动性质押，请查看我们的[指南 ](https://www.quicknode.com/guides/solana-development/getting-started/liquid-staking)。



## Resources 资源

原文链接：https://www.quicknode.com/guides/solana-development/getting-started/how-to-stake-sol

- [Solana Validator Metrics
  Solana 验证器指标](https://www.validators.app/)
- [StakeWiz](https://stakewiz.com/)
- [Solana Beach Validator Explorer
  索拉纳海滩验证器探索器](https://solanabeach.io/validators)
- [Solana CLI Documentation
  Solana CLI 文档](https://docs.solana.com/cli/delegate-stake)
- [*StakeProgram* Class JS Documentation
  *StakeProgram* 类 JS 文档](https://solana-labs.github.io/solana-web3.js/classes/StakeProgram.html)