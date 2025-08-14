# 查找 Solana 钱包和 Mint 关联代币地址的五种方法

## Overview 概述

无论是查询代币账户余额还是构建代币转账指令，了解如何查找用户的代币账户地址都是任何 Solana 开发人员的一项基本技能。在本指南中，将介绍Solana 代币账户的基础知识，以及五种简单的方法来获取与 Solana 关联的代币账户的地址：

使用命令行...

- 使用 [Solana 的 SPL-Token 命令行界面 (SPL-Token CLI)](https://www.quicknode.com/guides/solana-development/spl-tokens/how-to-look-up-the-address-of-a-token-account?utm_source=qn-youtube&utm_campaign=MAY2025UNDERSTANDINGSOLANATOKENACCOUNTS&utm_content=sign-up&utm_medium=generic#link-cli)
- 使用 [cURL 脚本](https://www.quicknode.com/guides/solana-development/spl-tokens/how-to-look-up-the-address-of-a-token-account?utm_source=qn-youtube&utm_campaign=MAY2025UNDERSTANDINGSOLANATOKENACCOUNTS&utm_content=sign-up&utm_medium=generic#link-curl)

使用 JavaScript/TypeScript

- 使用 [Solana 套件](https://www.quicknode.com/guides/solana-development/spl-tokens/how-to-look-up-the-address-of-a-token-account?utm_source=qn-youtube&utm_campaign=MAY2025UNDERSTANDINGSOLANATOKENACCOUNTS&utm_content=sign-up&utm_medium=generic#link-web3-2)
- 使用 [Solana 的旧版 web3.js 库](https://www.quicknode.com/guides/solana-development/spl-tokens/how-to-look-up-the-address-of-a-token-account?utm_source=qn-youtube&utm_campaign=MAY2025UNDERSTANDINGSOLANATOKENACCOUNTS&utm_content=sign-up&utm_medium=generic#link-web3)

使用 Rust

- 使用 [`spl_associated_token_account`](https://www.quicknode.com/guides/solana-development/spl-tokens/how-to-look-up-the-address-of-a-token-account?utm_source=qn-youtube&utm_campaign=MAY2025UNDERSTANDINGSOLANATOKENACCOUNTS&utm_content=sign-up&utm_medium=generic#link-rust)

## 了解 Solana 上的代币账户

首先回顾一下什么是**代币账户**以及它与 Solana **钱包地址**有何不同。
当在 Phantom 或 Solflare 等 Solana 钱包中查看代币余额时，通常会查看**关联的代币账户**及其余额：

![Solflare Token Balances](https://www.quicknode.com/guides/assets/images/8-ca113374a2292f93b3c95d7fd89d6231.png)

首先，获取钱包地址——可以直接从 Phantom 或任何其他钱包复制它：

![&#39;Phantom Wallet Copy Button&#39;](https://www.quicknode.com/guides/assets/images/1-f4ee8632fbe6a77395b96c4a53856a50.png)
访问 [Solana 浏览器](https://explorer.solana.com/)并粘贴钱包地址。向下滚动并点击 **“Tokens”** 选项卡，将看到**代币账户**及其余额的列表。其中不仅显示同质化代币（类似货币的代币，例如 USDC 或 BONK），还显示非同质化代币 (NFT)。点击 **“详细信息”** 按钮，获取有关每个代币账户的更多信息：

![Solana Explorer Token Account Details](https://www.quicknode.com/guides/assets/images/9-50196941631d32af03bb190cc05dc498.png)
Explorer 中的代币账户和余额与 Solana 钱包应用中看到的一致。
可以看到，每个代币（USDC、PyUSD、BONK 等）都有一个单独的**代币账户地址** 。这使得涉及不同代币账户的多个交易可以并行运行——可以同时发送 USDC 并接收 PyUSD，因为它们存储在不同的代币账户中。这也是 Solana 速度快的原因之一。

## Mint Addresses 铸币地址

可以看到每个**代币账户**都有一个**代币铸造地址** 。
**代币铸造地址**唯一地定义了一个代币。代币的创建者会在其网站或社交媒体上发布铸造地址：

**USDC** 由 Circle 创建，USDC 铸币地址（ `EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v` ）已[在 Circle 的网站上公布 ](https://www.circle.com/multi-chain-usdc/solana)。

![USDC Mint Address](https://www.quicknode.com/guides/assets/images/11-a036774ce84e25e59e9f661412a1c6cb.png)

**PyUSD** 由 PayPal 创建，PyUSD 铸币地址（ `2b1kV6DkPAnxd5ixfnxCpjxmKwqjjaYmCZfHsFu24GXo` ）已[在 PayPal 网站上公布 ](https://www.paypal.com/us/cshelp/article/what-is-paypal-usd-pyusd-help1005)。

![PyUSD Mint Address](https://www.quicknode.com/guides/assets/images/12-1a18a3a09c37926e392264f63d0bf4ba.png)

**BONK** 由社区创建，BONK 铸币地址（ `DezXAZ8z7PnrnRJjz3wXBoRgixCa6xjnB7YaB1pPB263` ）已[在 @bonk_inu X 个人资料上发布 ](https://x.com/bonks__lnu)。

![BONK Mint Address](https://www.quicknode.com/guides/assets/images/13-70b9e87ca6bfa464cac2733e03b2d011.png)


有人可能会创建一个看起来像 USDC 或 PyUSD 的新代币，但如果铸造地址不同，它就是完全不同的代币。因此许多像 Solana Explorer 这样的应用程序在显示不知名的代币时都会显示警告：

![Solana Explorer Token Warning](https://www.quicknode.com/guides/assets/images/10-0f84d1201f5885336077133d0a418967.png)


Solana Explorer 还可以搜索知名代币，如 USDC、BONK 和 PyUSD，并查看正确的铸币地址。



## The Associated Token Account program 关联代币账户计划

在浏览器中查看钱包时显示的代币账户大多是**关联代币账户** 。它们是与特定**钱包地址**和**代币铸造地址**相关联的代币账户。
虽然将代币存储在单独的账户中很快，但这意味着我们还需要弄清楚如何在给定特定钱包地址的情况下找到每个代币账户地址。

- 如果想向 `bob.sol` 发送一些 USDC，需要知道 `bob.sol` 存储其 USDC 的代币账户。
- 如果想要知道 `alice.sol` 是否有任何 BONK，需要知道 Alice 存储她的 BONK 的代币账户。
  这就是***关联代币账户***发挥作用的地方。


[关联代币计划 (Associated Token Program)](https://spl.solana.com/associated-token-account) 是一种基于钱包地址和代币铸造情况，确定特定钱包账户存储代币位置的方法。例如：

- 为了找到 `bob.sol` 的 USDC 关联的代币账户，我们将他的钱包地址和 USDC 铸币地址结合起来。
- 要找到 `alice.sol` 的 BONK 关联的代币账户，我们需要钱包地址和 BONK 铸币地址。

这些账户由代币计划 ( `TokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DA` ) 或较新的代币扩展计划 ( `TokenzQdBNbLqP5VEhdkAS6EPFLC1PHnBqCXEpPxuEb` ) 所有，并由用户的钱包地址控制。

本指南将展示如何获取与指定钱包和铸币地址关联的代币账户的地址。
请记住： **<u>关联代币账户是使用由钱包地址和代币铸币地址生成的地址的代币账户</u>** 。



## 方法 1 - 使用 SPL-Token CLI 的命令行

| Dependency    | Version |
| ------------- | ------- |
| spl-token-cli | 5.1.0   |

检查钱包余额的第一种方法是使用 Solana SPL-Token CLI。请按照 [spl.solana.com/token](https://spl.solana.com/token#setup) 上针对操作环境的说明进行操作。为确保安装成功，请打开一个新终端并输入：

```sh
spl-token --version
```

应该看到类似这样的内容：

![&#39;Solana CLI Version Check&#39;](https://www.quicknode.com/guides/assets/images/0-66de2a09d0235c732b507095001d46df.png)


在终端中，输入以下内容获取代币帐户：

```sh
spl-token address \
  --owner WALLET_WALLET_ADDRESS  \
  --token TOKEN_MINT_ADDRESS \
  --verbose \
  --url mainnet-beta
```

只需传递 `--owner` 标志（包含要查询的钱包地址）、 `--token` 标志（包含要查询的代币的铸币地址）以及 `--verbose` 标志（用于获取更详细的响应，此查询必需）。- `-um` 标志告知 CLI 使用 Solana 主网集群（ CLI 工具会验证铸币地址是否确实是该集群上的有效铸币地址）。

应该看到类似这样的内容：

![&#39;Solana CLI Results&#39;](https://www.quicknode.com/guides/assets/images/2-993164719cfb7dc0045dfe1aaff52d87.png)



## 方法 2 - 使用 cURL 的命令行

| Dependency | Version |
| ---------- | ------- |
| curl       | 8.7.1   |


cURL 是一个用于通过 URL 传输数据的命令行工具和库。大多数基于 *nix 的系统都预装了 cURL 支持。运行以下命令检查是否支持 cURL：

```sh
curl -V
```

If you don't have it installed, head to [curl.se](https://curl.se/) to set it up.
如果尚未安装，请前往 [curl.se](https://curl.se/) 进行设置。

Once you are ready to go, make a file called `get-associated-token-account.bash` with the following contents:
准备就绪后，请创建一个名为 `get-associated-token-account.bash` 的文件，其中包含以下内容：

```sh
WALLET_ADDRESS="YOUR WALLET ADDRESS"
MINT_ADDRESS="YOUR MINT ADDRESS"

curl https://api.mainnet-beta.solana.com -X POST -H "Content-Type: application/json" -d '
  {
    "jsonrpc": "2.0",
    "id": 1,
    "method": "getTokenAccountsByOwner",
    "params": [
      "'$WALLET_ADDRESS'",
      {
        "mint": "'$MINT_ADDRESS'"
      },
      {
        "encoding": "jsonParsed"
      }
    ]
  }
' | jq;

echo "✅ Completed successfully. Check the output above."
```


在终端中运行：

```sh
bash get-associated-token-account.bash 
```

应该看到类似这样的内容：

![&#39;Solana cURL Results&#39;](https://www.quicknode.com/guides/assets/images/6-3f2a95787a8e9a1dd2e56a7f130d4367.png)


请注意， `result.value[0].pubkey` 字段中返回了相同的地址🙌。有关 `getTokenAccountsByOwner` 方法的更多信息，请查看 [Solana 文档 ](https://www.quicknode.com/docs/solana/getTokenAccountsByOwner)。



## 方法 3 - 使用 Solana Kit 的 TypeScript

| Dependency            | Version |
| --------------------- | ------- |
| node.js               | 22.14.0 |
| @solana/kit           | 2.1.1   |
| @solana-program/token | 0.5.1   |
| esrun                 | 3.2.26  |

使用以下命令在终端中创建一个新的项目主管：

```sh
mkdir token-address-solana-kit && cd token-address-solana-kit
```

安装 Solana Kit 依赖项：

```sh
npm init -y
npm install --save @solana/kit @solana-program/token esrun
```

然后创建一个名为 `get-associated-token-account.ts` 的新文件，其中包含以下内容：

```typescript
import { address } from "@solana/kit";
import { findAssociatedTokenPda } from "@solana-program/token";

const WALLET = address("YOUR_WALLET_ADDRESS"); // e.g., dDCQNnDmNbFVi8cQhKAgXhyhXeJ625tvwsunRyRc7c8
const MINT = address("YOUR_MINT_ACCOUNT"); // e.g., EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v
const TOKEN_PROGRAM_ADDRESS = address(
  "TokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DA"
);

const [associatedTokenAddress] = await findAssociatedTokenPda({
  mint: MINT,
  owner: WALLET,
  tokenProgram: TOKEN_PROGRAM_ADDRESS,
});

console.log(
  "✅ Got associated token account address using Solana Kit: ",
  associatedTokenAddress
);
```

运行代码。在终端中输入：

```sh
npx esrun address.ts
```

应该会在终端中看到返回的相关代币地址。

![Solana Kit Results](https://www.quicknode.com/guides/assets/images/14-0ccc464b008c2d113783690cfdec28e4.png)
或者也可以使用 `rpc` 类向 Solana 集群发送 [`getTokenAccountsByOwner`](https://www.quicknode.com/docs/solana/getTokenAccountsByOwner) 请求。这需要网络请求，因此速度较慢，但它也能找到在关联代币程序之外创建的代币账户，这在某些（罕见）情况下可能会有用！



## 方法 4 - 使用 Solana web3.js 的 TypeScript

| Dependency      | Version |
| --------------- | ------- |
| node.js         | 22.14.0 |
| @solana/web3.js | 1.98.2  |
| esrun           | 3.2.26  |

使用以下命令在终端中创建一个新的项目目录：

```sh
mkdir token-address-legacy-web3 && cd token-address-legacy-web3
```

安装 Solana web3.js 依赖项：

```sh
npm init -y
npm install --save @solana/web3.js@1
```

创建一个名为 `get-associated-token-account-legacy-web3.js` 的新文件，其中包含以下内容：在所选的代码编辑器中打开 **address.js** ，并在第 1 行引入 **@solana/web3.js** 。将从此包中导入 *PublicKey* 类。

```typescript
import { PublicKey } from '@solana/web3.js';

const WALLET = new PublicKey('YOUR_WALLET_ADDRESS'); // e.g., E645TckHQnDcavVv92Etc6xSWQaq8zzPtPRGBheviRAk
const MINT = new PublicKey('YOUR_MINT_ADDRESS');    // e.g., EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v
const TOKEN_PROGRAM_ID = new PublicKey('TokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DA');
const ASSOCIATED_TOKEN_PROGRAM_ID = new PublicKey('ATokenGPvbdGVxr1b2hvZbsiqW5xWH25efTNsLJA8knL');

const [address] = PublicKey.findProgramAddressSync(
  [WALLET.toBuffer(), TOKEN_PROGRAM_ID.toBuffer(), MINT.toBuffer()],
  ASSOCIATED_TOKEN_PROGRAM_ID
);

console.log(
  "✅ Got associated token account address using Solana web3.js: ",
  address.toBase58()
);
```

`findProgramAddressSync` 函数根据给定的种子和程序 ID 确定性地查找程序地址。在本例中，将钱包地址、代币程序 ID 和铸币地址作为种子，并将关联的代币程序 ID 作为程序 ID 传递（这些种子及其顺序[由关联的代币程序定义 ](https://github.com/solana-labs/solana-program-library/blob/74ab831642efb6f12216a3ca700a39c057634de4/associated-token-account/program/src/lib.rs#L62-L76)）。
运行代码。在终端中输入：

```sh
npx esrun get-associated-token-account-legacy-web3.js
```

应该看到类似这样的内容：

![&#39;Solana-Web3.js Results&#39;](https://www.quicknode.com/guides/assets/images/3-dd576cba71a2f6399ec54537acabc28f.png)

注意，可能需要使用 `@solana/spl-token` 中的 `getAssociatedTokenAddressSync` 函数——它使用起来更方便一些。只需安装 `@solana/spl-token` 包即可：

```sh
npm install --save @solana/spl-token
然后像这样使用它：
```

```typescript
import { PublicKey } from "@solana/web3.js";
import { getAssociatedTokenAddressSync } from "@solana/spl-token";

const WALLET = new PublicKey("YOUR_WALLET_ADDRESS"); // e.g., E645TckHQnDcavVv92Etc6xSWQaq8zzPtPRGBheviRAk
const MINT = new PublicKey("YOUR_MINT_ADDRESS"); // e.g., EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v
const address = getAssociatedTokenAddressSync(WALLET, MINT);

console.log(
  "✅ Got associated token account address using Solana web3.js: ",
  address.toBase58()
);
```



## 方法 5 - Rust

| Dependency                   | Version |
| ---------------------------- | ------- |
| rust                         | 1.86.0  |
| solana-sdk                   | 2.2.22  |
| spl-associated-token-account | 7.0.0   |

如果是 Rust 开发人员，还可以使用 [Solana Rust SDK](https://docs.rs/solana-sdk/latest/solana_sdk/) 。在项目文件夹中，创建一个新项目并切换到它：

```sh
cargo new get-associated-token-address-rust; cd get-associated-token-address-rust
```

添加必要的依赖项：

```sh
cargo add solana-sdk spl-associated-token-account
```

打开 **src/main.rs** ，在第 1 行导入必要的包：

```rust
use solana_sdk::pubkey::Pubkey;
use spl_associated_token_account::get_associated_token_address;
use std::str::FromStr;
```

然后修改 `main` 函数，通过将所有者和铸币者的公钥传递到 `get_associated_token_address` 方法来获取地址：

```rust
fn main() {
    let wallet = Pubkey::from_str("YOUR_WALLET_ADDRESS").unwrap();
    let mint = Pubkey::from_str("YOUR_TOKEN_MINT_ADDRESS").unwrap();
    let associated_token_address = get_associated_token_address(&wallet, &mint);

    println!(
        "✅ Got associated token account address using Rust: {}",
        associated_token_address
    );
}
```

编译并运行代码。在终端中输入：

```sh
cargo run
```

应该会看到返回相同的代币地址：

![&#39;Solana Rust Results&#39;](https://www.quicknode.com/guides/assets/images/7-b4f3776f88d44a70aa02dc5b2969c8b7.png)



## 参考

- [Solana Fundamentals Solana 基础知识](https://www.quicknode.com/guides/solana-development/getting-started/solana-fundamentals-reference-guide)
- [Solana Docs Solana 文档](https://www.quicknode.com/docs/solana)

