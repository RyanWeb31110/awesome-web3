# Four Ways to Check the Balance of a Solana Wallet Address 查看 Solana 钱包地址余额的四种方法

如果您刚刚开始使用 Solana，您可能需要查看账户余额。在本指南中，我们将介绍四种简单的方法来查看 Solana 钱包的余额：

- 使用 Solana 的命令行界面 (Solana CLI)（[ 跳转到 Solana CLI](https://www.quicknode.com/guides/solana-development/getting-started/how-to-look-up-the-balance-of-a-solana-wallet#link-cli) ）
- 使用 Solana 的 JavaScript API（[ 跳转到 Solana-Web3.js](https://www.quicknode.com/guides/solana-development/getting-started/how-to-look-up-the-balance-of-a-solana-wallet#link-web3) ）
- 使用 cURL 脚本（[ 跳转到 cURL](https://www.quicknode.com/guides/solana-development/getting-started/how-to-look-up-the-balance-of-a-solana-wallet#link-curl) ）
- 使用 Rust Solana SDK（[ 跳转到 Rust](https://www.quicknode.com/guides/solana-development/getting-started/how-to-look-up-the-balance-of-a-solana-wallet#link-rust) ）

#### What You Will Need 你需要什么

- 已安装 [Solana CLI](https://docs.solana.com/cli/install-solana-cli-tools) 最新版本
- 已安装 [Nodejs](https://nodejs.org/en/) （16.15 或更高版本）
- 基本的 JavaScript 经验
- 安装了 [cURL](https://curl.se/) 稳定版本
- 要查询的钱包（例如 `E645TckHQnDcavVv92Etc6xSWQaq8zzPtPRGBheviRAk` ）
- [Rust 安装 ](https://www.rust-lang.org/tools/install)（Rust 方法可选）

Solana  基础知识：集群

在查询钱包之前，我们先快速了解一下 Solana 的集群，因为我们在检查钱包余额时需要指定一个特定的集群。

“Solana 集群是一组协同工作的验证器，用于处理客户端交易并维护账本的完整性。多个集群可以共存。”

事实上，Solana 维护着三个集群，每个集群都有不同的用途：

- **Mainnet Beta 主网 Beta 版** ：生产环境，无需许可，使用真实代币
- **Devnet 开发网** ：应用开发者测试 Solana 应用的平台（Devnet 上的代币并非真实存在，且不具备任何经济价值）。Devnet 通常运行与主网 Beta 相同的软件版本。
- **Testnet 测试网** ：Solana 核心贡献者和验证者对新更新和功能进行压力测试的环境，重点是测试网络性能（测试网上的代币不是真实的，也没有财务价值）

（来源： [docs.solana.com/cluster/overview](https://docs.solana.com/cluster/overview) ）

本指南中使用 `Mainnet-Beta` ，但您应该知道钱包可以同时在每个集群上拥有余额。



## 方法 1 - Solana CLI

我们的第一种方法是使用 Solana CLI 检查钱包余额。如果您尚未安装，请按照 [docs.solana.com/cli/install-solana-cli-tools](https://docs.solana.com/cli/install-solana-cli-tools) 上适合您操作环境的说明进行操作。为确保安装成功，请打开一个新终端并输入：

```sh
solana --version
```

您应该看到类似这样的内容：

![&#39;Solana CLI Version Check&#39;](https://www.quicknode.com/guides/assets/images/0-6f22c4aaec6101b1a57667174096cae6.png)

一切准备就绪！你只需要准备好你的钱包地址——你可以直接从 Phantom 或任何其他钱包复制：

![image-20250731170601414](./../resources/image/image-20250731170601414.png)

In your terminal, fetch your wallet balance by entering:
在您的终端中，输入以下内容获取您的钱包余额：

```sh
solana balance YOUR_WALLET_ADDRESS -u mainnet-beta
```

您可以通过修改 `-u` （Solana JSON RPC 的 URL）选项来修改搜索条件，以获取不同集群上该钱包的余额。要获取开发网或测试网的余额，请尝试：

```sh
solana balance YOUR_WALLET_ADDRESS -u devnet # for Devnet
# or 
solana balance YOUR_WALLET_ADDRESS -u testnet # for Testnet
```

这些默认集群（主网-beta、测试网、开发网）是公共的 JSON RPC 端点。如果您计划在 Solana 上进行大量查询或构建，您可能需要一个自己的端点。您需要一个主网节点来查询钱包的真实余额：

![&#39;Solana Mainnet Endpoint&#39;](https://www.quicknode.com/guides/assets/images/solana-mainnet-6bbd732b3c3cba30c41fbf9357c517bc.png)

您现在可以修改查询以使用您的端点：

```sh
solana balance YOUR_WALLET_ADDRESS -u https://example.solana.quiknode.pro/000000/
```

![&#39;Solana CLI Results&#39;](https://www.quicknode.com/guides/assets/images/2-7d1eb7966abb42851beab78b8d6deea2.png)



## 方法 2 - Solana-Web3.js

在终端中创建一个新的项目目录和文件 **balance.js** ，使用以下命令：

```sh
mkdir sol_balance
cd sol_balance
echo > balance.js
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

在您选择的代码编辑器中打开 **balance.js** ，在第 1 行，引入 **@solana/web3.js** 并将其存储在常量 **SOLANA** 中。在第 2 行，解构 **SOLANA** 以获取一些必要的类、方法和常量。

```javascript
const SOLANA = require('@solana/web3.js');
const { Connection, PublicKey, LAMPORTS_PER_SOL, clusterApiUrl } = SOLANA;
```

使用您的 QuickNode 端点（或使用 `clusterApiUrl('CLUSTER_NAME')` 与公共集群）建立与集群的连接，并将您的钱包地址定义为 `WALLET_ADDRESS` 。

```javascript
const QUICKNODE_RPC = 'https://example.solana.quiknode.pro/000000/'; // 👈 Replace with your QuickNode Endpoint OR clusterApiUrl('mainnet-beta')
const SOLANA_CONNECTION = new Connection(QUICKNODE_RPC);
const WALLET_ADDRESS = 'YOUR_WALLET_ADDRESS'; //👈 Replace with your wallet address
```

如果您经常使用 **Solana-Web3.js** 进行操作，那么您会对 *Connection* 类非常熟悉。它实际上是用来建立与 JSON RPC 端点的连接的方法。
最后，通过调用 *Connection* 实例上的 `getBalance()` 方法获取余额：

```javascript
(async () => {
    let balance = await SOLANA_CONNECTION.getBalance(new PublicKey(WALLET_ADDRESS));
    console.log(`Wallet Balance: ${balance/LAMPORTS_PER_SOL}`)
})();
```


关于脚本的一些说明：

- 使用异步函数来允许 `getBalance` 从网络返回响应
- 请注意，首先创建的不是仅仅传递 `WALLET_ADDRESS` ，而是 *PublicKey* 的新实例。PublicKey 对于获取和发送信息到 Solana 集群至关重要。您可以在代码中通过将钱包的字符串地址传递给 `new PublicKey()` 构造函数来定义 PublicKey。
- 将 `balance` 除以 `LAMPORTS_PER_SOL` ，因为 Solana 用于跟踪 SOL 的native unit实际上是 lamports（1 SOL 的 1 / 10^9）。

运行你的代码。在终端中输入：

```sh
node balance
```

您应该看到类似这样的内容：

![&#39;Solana-Web3.js Results&#39;](https://www.quicknode.com/guides/assets/images/3-637cde75e0226da3826a2de7377245c9.png)



## 方法 3 - cURL

cURL 是一个用于通过 URL 传输数据的命令行工具和库。大多数基于 *nix 的系统都预装了 cURL 支持。运行以下命令检查是否支持 cURL：

```sh
curl -h
```

如果您尚未安装，请前往 [curl.se](https://curl.se/) 进行设置。

准备就绪后，您只需在终端中输入此 HTTP 请求（请确保替换您的终端节点和钱包地址）。在终端中输入：

```sh
curl https://example.solana.quiknode.pro/000000/ -X POST -H "Content-Type: application/json" -d '
  {
    "jsonrpc": "2.0", "id": 1,
    "method": "getBalance",
    "params": [
      "YOUR_WALLET_ADDRESS"
    ]
  }
'
```

应该看到类似这样的内容：

![&#39;Solana cURL Results&#39;](https://www.quicknode.com/guides/assets/images/4-d98ab75cc08dbf54941d1d2ca20e494f.png)

请注意，余额以 lamports 为单位返回在 `result.value` 字段中。



## 方法 4 - Rust

如果您是 Rust 开发人员，您还可以使用 [Solana Rust SDK](https://docs.rs/solana-sdk/latest/solana_sdk/) 。在项目文件夹中，使用以下命令启动一个新项目：

```sh
cargo new sol-balance
```

导航到新创建的目录：

```sh
cd sol-balance
```

将必要的依赖项添加到您的 **Cargo.toml** 文件：

```toml
[dependencies]
solana-sdk = "1.16.14"
solana-client = "1.6.14"
```

打开 **src/main.rs** ，导入必要的包：

```rust
use solana_sdk::pubkey::Pubkey;
use solana_client::rpc_client::RpcClient;
use std::str::FromStr;
```

定义您的所有者帐户地址和 `LAMPORTS_PER_SOL` 常量：

```rust
const OWNER: &str = "YOUR_WALLET_ADDRESS";
const LAMPORTS_PER_SOL: f64 = 1_000_000_000.0;
```

最后，创建 `main` 函数，通过将所有者钱包公钥传递到 `get_balance` 方法来获取您的钱包余额：

```rust
fn main() {
    let owner_pubkey = Pubkey::from_str(OWNER).unwrap();
    let connection = RpcClient::new("https://example.solana.quiknode.pro/000000/".to_string()); // 👈 Replace with your QuickNode Endpoint
    let balance = connection.get_balance(&owner_pubkey).unwrap() as f64;
    println!("Balance: {}", balance / LAMPORTS_PER_SOL);
}
```

编译并运行代码。在终端中输入：

```sh
cargo build
cargo run
```

你应该会看到钱包余额。



## 原文链接：

https://www.quicknode.com/guides/solana-development/getting-started/how-to-look-up-the-balance-of-a-solana-wallet

