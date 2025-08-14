# 005 Wrapping, Transferring, and Unwrapping SOL on Solana 

Wrapped SOL (wSOL) 是 Solana 区块链上的一种代币，以 SPL token的形式代表native SOL。

编写脚本以使用 Solana Web3.js、Solana CLI 和 Solana Rust SDK 执行以下操作：

1. Wrap SOL into wSOL 将 SOL 包装到 wSOL 中
2. Transfer wrapped SOL to another wallet 将包装好的 SOL 转移到另一个钱包
3. Unwrap SOL 解包 SOL

### What You Will Need

- 已安装 [Nodejs](https://nodejs.org/en/) （16.15 或更高版本）
- TypeScript 体验和 [ts-node](https://www.npmjs.com/package/ts-node) 安装
- 已安装 [Solana CLI](https://docs.solana.com/cli/install)
- 已安装 [SPL Token CLI](https://spl.solana.com/token)
- 了解 [Solana 基础](https://www.quicknode.com/guides/solana-development/getting-started/solana-fundamentals-reference-guide?utm_source=internal&utm_campaign=guides&utm_content=wrapped-sol-basics)知识
- 理解 [SPL 代币](https://www.quicknode.com/guides/solana-development/spl-tokens/how-to-look-up-the-balance-of-a-token-account)
- [Rust 安装 ](https://www.rust-lang.org/tools/install)（Rust 方法可选）

| Dependency        | Version |
| ----------------- | ------- |
| @solana/web3.js   | ^1.91.1 |
| @solana/spl-token | ^0.3.9  |
| typescrip         | ^5.0.0  |
| ts-node           | ^10.9.1 |
| solana cli        | 1.18.8  |
| spl-token cli     | >3.3.0  |
| rust              | 1.75.0  |



##  什么是 Wrapped SOL？

Wrapped SOL (wSOL) 是 Solana 区块链上的一种代币，以 SPL 代币（Solana 上的原生代币协议）的形式代表原生 SOL。它允许您在需要 SPL 代币的环境中使用 SOL，例如去中心化交易所或其他与代币池交互的 DeFi 应用。通过 Wrapped SOL，您可以将原生 SOL 与其他 SPL 代币无缝集成到您的应用程序中，从而在 Solana 生态系统中实现更广泛的功能和交互。这种灵活性使包装 SOL 成为在 Solana 上创建更复杂、更具互操作性的去中心化应用程序的重要工具。

要包装 SOL，您只需将 SOL 发送到 native mint ( `So11111111111111111111111111111111111111112` ：可通过 `@solana/spl-token` 包中的 `NATIVE_MINT` 常量访问) 上的[关联代币账户 ](https://www.quicknode.com/guides/solana-development/spl-tokens/how-to-look-up-the-address-of-a-token-account)，并从 SPL 代币程序中调用 syncNative 指令 ( [source](https://github.com/solana-labs/solana-program-library/blob/12446572af0a3cb41c1c11d4c77a752eaad5661e/token/program/src/instruction.rs#L378) ) ，`syncNative` 会更新代币账户上的 amount 字段，以匹配可用的 wrapped SOL 数量。只有通过关闭代币账户并选择所需的地址来发送代币账户的 lampports，才能检索该 SOL。

让我们创建一个示例脚本来演示 wrapped SOL 交易的完整生命周期。



## Solana-Web3.js

### Create a New Project 

首先，设置我们的项目：

```sh
mkdir wrapped-sol-demo && cd wrapped-sol-demo && echo > app.ts
```

使用您最喜欢的包管理器初始化一个新项目（在本指南中我们将使用 npm）：

```sh
npm init -y
```

安装必要的依赖项：

```sh
npm install @solana/web3.js@1 @solana/spl-token
```



### Set Up Local Environment 

#### Import Dependencies 

打开 `app.ts` 并导入所需的依赖项：

```typescript
import {
    NATIVE_MINT,
    createAssociatedTokenAccountInstruction,
    getAssociatedTokenAddress,
    createSyncNativeInstruction,
    getAccount,
    createTransferInstruction,
    createCloseAccountInstruction,
} from "@solana/spl-token";
import {
    Connection,
    Keypair,
    LAMPORTS_PER_SOL,
    SystemProgram,
    Transaction,
    sendAndConfirmTransaction,
    PublicKey,
} from "@solana/web3.js";
```

这些导入提供了与 Solana 区块链和 SPL 代币程序交互所需的功能和类型。

### Create Main Function

现在，继续定义一个用于协调我们操作的 `main` 函数。在 `app.ts` 的 imports 下方添加以下代码：

```typescript
async function main() {
    const connection = new Connection("http://127.0.0.1:8899", "confirmed");
    const wallet1 = Keypair.generate();
    const wallet2 = Keypair.generate();

    await requestAirdrop(connection, wallet1);

    const tokenAccount1 = await wrapSol(connection, wallet1);

    const tokenAccount2 = await transferWrappedSol(connection, wallet1, wallet2, tokenAccount1);

    await unwrapSol(connection, wallet1, tokenAccount1);

    await printBalances(connection, wallet1, wallet2, tokenAccount2);
}
```

此函数概述了我们的步骤，并调用了多个函数来执行每个步骤。我们将在下一节中定义每个函数。在本例中，我们将使用[ local test validator 本地测试验证器](https://www.quicknode.com/guides/solana-development/getting-started/start-a-solana-local-validator?utm_source=internal&utm_campaign=guides&utm_content=wrapped-sol-basics)来运行代码；但您可以修改连接，使其与开发网或主网兼容，以用于实际应用程序。

当您准备好开始与开发网或主网交互时，您将需要一个 RPC 端点。您可以在 [QuickNode.com](https://www.quicknode.com/signup?utm_source=internal&utm_campaign=guides&utm_content=a-complete-guide-to-wrapped-sol) 免费获取一个 RPC 端点，然后只需将连接更改为指向您的端点，如下所示：

```typescript
const connection = new Connection("https://example.solana-mainnet.quiknode.pro/123/", "confirmed");
```



### Create Wrapped SOL Operations

让我们为想要执行的每个操作创建函数。

#### 1. Request Airdrop 

添加以下函数来请求 SOL 空投：

```typescript
async function requestAirdrop(connection: Connection, wallet: Keypair): Promise<void> {
    const airdropSignature = await connection.requestAirdrop(
        wallet.publicKey,
        2 * LAMPORTS_PER_SOL
    );
  while (true) {
        const { value: statuses } = await connection.getSignatureStatuses([fromAirdropSignature]);
        if (!statuses || statuses.length === 0) {
            await new Promise(resolve => setTimeout(resolve, 1000));
            continue;
        }
        if (statuses[0] && statuses[0].confirmationStatus === 'confirmed' || statuses[0].confirmationStatus === 'finalized') {
            break;
        }
        await new Promise(resolve => setTimeout(resolve, 1000));
  }    console.log("✅ - Step 1: Airdrop completed");
}
```

该函数请求向指定钱包空投 2 SOL。



#### 2. Wrap SOL

接下来，让我们创建一个函数来包装 SOL：

```typescript
async function wrapSol(connection: Connection, wallet: Keypair): Promise<PublicKey> {
    const associatedTokenAccount = await getAssociatedTokenAddress(
        NATIVE_MINT,
        wallet.publicKey
    );

    const wrapTransaction = new Transaction().add(
        createAssociatedTokenAccountInstruction(
            wallet.publicKey,
            associatedTokenAccount,
            wallet.publicKey,
            NATIVE_MINT
        ),
        SystemProgram.transfer({
            fromPubkey: wallet.publicKey,
            toPubkey: associatedTokenAccount,
            lamports: LAMPORTS_PER_SOL,
        }),
        createSyncNativeInstruction(associatedTokenAccount)
    );
    await sendAndConfirmTransaction(connection, wrapTransaction, [wallet]);

    console.log("✅ - Step 2: SOL wrapped");
    return associatedTokenAccount;
}
```

让我们来看看这个函数：

1. 首先，我们定义一个与钱包公钥和 native mint 原生铸币相关的 ATA 代币账户：associatedTokenAccount。
2. 接下来，我们组装一个包含三条指令的交易：
   - 我们为 wrapped SOL 创建相关的代币账户。
   - 我们将 1 SOL 转移到新创建的 ATA。
   - 我们调用 `syncNative` 指令来更新 ATA 上的数量字段，以匹配可用的包装 SOL 数量。
3. 最后，我们发送交易并记录成功消息。



#### 3. Transfer Wrapped SOL

现在，让我们创建一个函数来传输 Wrapped SOL：

```typescript
async function transferWrappedSol(
    connection: Connection,
    fromWallet: Keypair,
    toWallet: Keypair,
    fromTokenAccount: PublicKey
): Promise<PublicKey> {
    const toTokenAccount = await getAssociatedTokenAddress(
        NATIVE_MINT,
        toWallet.publicKey
    );

    const transferTransaction = new Transaction().add(
        createAssociatedTokenAccountInstruction(
            fromWallet.publicKey,
            toTokenAccount,
            toWallet.publicKey,
            NATIVE_MINT
        ),
        createTransferInstruction(
            fromTokenAccount,
            toTokenAccount,
            fromWallet.publicKey,
            LAMPORTS_PER_SOL / 2
        )
    );
    await sendAndConfirmTransaction(connection, transferTransaction, [fromWallet]);

    console.log("✅ - Step 3: Transferred wrapped SOL");
    return toTokenAccount;
}
```

由于包装后的 SOL 本身就是一个 SPL 代币，因此此功能只是一次简单的 SPL 代币转移（[ 请查看我们的指南：Solana 上的 SPL 代币转移：完整指南 ](https://www.quicknode.com/guides/solana-development/spl-tokens/how-to-transfer-spl-tokens-on-solana)）。此函数为接收者创建一个 ATA（在 `NATIVE_MINT` 上进行播种），然后将一半的包装后的 SOL 转移给它。



#### 4. Unwrap SOL

让我们添加一个函数来解开 SOL。为此，我们需要做的就是关闭关联的代币账户，并选择所需的地址来发送代币账户的 Lamport。

```typescript
async function unwrapSol(
    connection: Connection,
    wallet: Keypair,
    tokenAccount: PublicKey
): Promise<void> {
    const unwrapTransaction = new Transaction().add(
        createCloseAccountInstruction(
            tokenAccount,
            wallet.publicKey,
            wallet.publicKey
        )
    );
    await sendAndConfirmTransaction(connection, unwrapTransaction, [wallet]);
    console.log("✅ - Step 4: SOL unwrapped");
}
```

此函数关闭已包装的 SOL 关联的代币账户，从而有效地解开 SOL。我们通过传入钱包的公钥来选择 Lamport 的目的地。



#### 5. Print Balances

最后，让我们添加一个打印余额的功能：

```typescript
async function printBalances(
    connection: Connection,
    wallet1: Keypair,
    wallet2: Keypair,
    tokenAccount2: PublicKey
): Promise<void> {
    const [wallet1Balance, wallet2Balance, tokenAccount2Info] = await Promise.all([
        connection.getBalance(wallet1.publicKey),
        connection.getBalance(wallet2.publicKey),
        connection.getTokenAccountBalance(tokenAccount2)
    ]);

    console.log(`   - Wallet 1 SOL balance: ${wallet1Balance / LAMPORTS_PER_SOL}`);
    console.log(`   - Wallet 2 SOL balance: ${wallet2Balance / LAMPORTS_PER_SOL}`);
    console.log(`   - Wallet 2 wrapped SOL: ${Number(tokenAccount2Info.value.amount) / LAMPORTS_PER_SOL}`);
}
```

此功能获取并显示两个钱包的 SOL 和 wrapped SOL 余额。



### Run the Script 运行脚本

最后，在 `app.ts` 文件底部添加对 `main` 的调用：

```typescript
main().catch(console.error);
```

要在本地网络上运行我们的代码，请打开终端并启动本地 Solana 验证器：

```sh
solana-test-validator -r
```

验证器启动后，在另一个终端中运行脚本：

```sh
ts-node app.ts
```

您应该在终端中看到每个步骤的输出。我们预计钱包 1 有 ~1.5 SOL（空投 2 SOL，减去 1 SOL 转移，加上 0.5 SOL unwrapped，减去租金和交易费），钱包 2 有 0 SOL（因为我们没有向钱包 2 发送任何 SOL），钱包 2 有 0.5 wrapped SOL ：

```sh
ts-node app
✅ - Step 1: Airdrop completed
✅ - Step 2: SOL wrapped
✅ - Step 3: Transferred wrapped SOL
✅ - Step 4: SOL unwrapped
   - Wallet 1 SOL balance: 1.49794572
   - Wallet 2 SOL balance: 0
   - Wallet 2 wrapped SOL: 0.5
```



## Solana CLI 

让我们尝试使用 Solana CLI 执行相同的操作。请确保您已安装最新版本的 Solana CLI 和 SPL Token CLI，请运行以下命令：

```sh
solana -V
```

```sh
spl-token -V
```

如果您对本节中的命令有任何问题，您可以随时运行 `solana COMMAND_NAME -h` 或 `spl-token COMMAND_NAME -h` 来使用 CLI 获取帮助。

### Set Up Local Environment 

首先，让我们确保您的 CLI 配置为使用 localnet：

```sh
solana config set -ul
```

现在，让我们为钱包 1 和钱包 2 创建密钥对（您可以在同一个 JS 项目目录中执行此操作）：

```sh
solana-keygen new --no-bip39-passphrase -s -o wallet1.json
```

```sh
solana-keygen new --no-bip39-passphrase -s -o wallet2.json
```

您应该看到在项目目录中创建了两个文件： `wallet1.json` 和 `wallet2.json` 。这两个文件包含我们将在本例中使用的两个钱包的私钥。

### Airdrop SOL

在您的终端中，将 2 个 SOL 空投到 `wallet1.json` ：

```sh
solana airdrop -k wallet1.json 2
```

由于我们使用的是相对路径，请确保您的终端与 `wallet1.json` 位于同一目录中（如果您按照本指南中的说明操作，则应该是这种情况）。

### Wrap SOL

要使用 CLI 包装 SOL，只需使用 `spl-token wrap` 命令。就像我们之前的示例一样，我们将 `wallet1.json` 中的 1 个 SOL 包装到 wSOL 中：

```sh
spl-token wrap 1 wallet1.json
```

此命令将为 wSOL 创建一个新的代币帐户，并在钱包中铸造 1 wSOL。

### Transfer Wrapped SOL

要使用 CLI 转移像 wSOL 这样的 SPL 代币，您必须首先为接收者创建一个 ATA 关联代币账户。这可以通过 `spl-token create-account` 命令完成。让我们为 `wallet2.json` 创建一个关联代币账户：

```sh
spl-token create-account So11111111111111111111111111111111111111112 --owner wallet2.json --fee-payer wallet1.json 
```

请注意，我们使用 `NATIVE_MINT` 作为关联代币账户的代币铸币方，并使用 `wallet1.json` 作为费用支付方。终端应该会输出新的关联代币账户的地址，例如 `Creating account 2MhZKqyeLY1X2g7jFb7CQ4D4ySHMvGkTfUYZFWGe7fXw` 。我们稍后会用到它。

现在，让我们将 0.5 wSOL 从 `wallet1.json` 转移到我们新创建的关联代币账户（ **将 `2MhZKqyeLY1X2g7jFb7CQ4D4ySHMvGkTfUYZFWGe7fXw` 替换为您新创建的关联代币账户的地址** ）：

```sh
spl-token transfer So11111111111111111111111111111111111111112 0.5 2MhZKqyeLY1X2g7jFb7CQ4D4ySHMvGkTfUYZFWGe7fXw --owner wallet1.json --fee-payer wallet1.json
```

此命令将 0.5 wSOL 从 `wallet1.json` 转移到新的关联代币账户。 `--owner` 标志指定拥有源 wSOL 账户的钱包， `--fee-payer` 标志指定支付交易费的钱包。

### Unwrap SOL 

要解包 wSOL，您可以使用 `spl-token unwrap` 命令。让我们通过关闭关联代币账户，从 `wallet1.json` 中解包 wSOL。在终端中运行：

```sh
spl-token unwrap --owner wallet1.json
```

此命令将从关联代币账户中解开 wSOL，并将 wrapped SOL 发送到拥有关联代币账户的钱包。

### Check Balances 

最后，我们可以再次检查余额，确保一切正常。在终端中运行：

```sh
solana balance -k wallet1.json
```

我们预计这将显示出~1.5 SOL 的平衡。

```sh
solana balance -k wallet2.json
```

我们预计其余额为 0。

```sh
spl-token balance --address 2MhZKqyeLY1X2g7jFb7CQ4D4ySHMvGkTfUYZFWGe7fXw
```

将地址替换为您创建的新关联代币账户的地址。我们预计该地址将显示余额 0.5。
Great job! 现在您知道如何使用 Solana CLI 来包装、转移、解包和检查余额了。



## Rust 

对于喜欢使用 Rust 的开发人员，以下是使用 Solana Rust SDK 执行相同的包装 SOL 操作的方法。

### Set Up the Project 

首先，创建一个新的 Rust 项目：

```sh
cargo new wrapped-sol-rust
cd wrapped-sol-rust
```

将以下依赖项添加到您的 `Cargo.toml` 文件：

```rust
[dependencies]
solana-sdk = "2.0.3"
solana-client = "2.0.3"
spl-token = "6.0.0"
spl-associated-token-account = "4.0.0"
```



### Create Main Function 创建主函数

让我们尝试用 Rust 重新创建 JavaScript 代码。首先导入必要的包并创建一个主函数，该函数概述了空投、包装、转移和解包 SOL 的步骤。在 `src/main.rs` 中，输入以下代码：

```rust
use solana_client::rpc_client::RpcClient;
use solana_sdk::{
    commitment_config::CommitmentConfig,
    pubkey::Pubkey,
    signature::{Keypair, Signer},
    system_instruction,
    transaction::Transaction,
};
use spl_associated_token_account::get_associated_token_address;
use spl_token::{instruction as spl_instruction, native_mint};

fn main() -> Result<(), Box<dyn std::error::Error>> {
    let rpc_client = RpcClient::new_with_commitment(
        "http://127.0.0.1:8899".to_string(),
        CommitmentConfig::processed(),
    );

    let wallet1 = Keypair::new();
    let wallet2 = Keypair::new();

    request_airdrop(&rpc_client, &wallet1)?;

    let token_account1 = wrap_sol(&rpc_client, &wallet1)?;

    let token_account2 = transfer_wrapped_sol(&rpc_client, &wallet1, &wallet2, &token_account1)?;

    unwrap_sol(&rpc_client, &wallet1, &token_account1)?;

    print_balances(&rpc_client, &wallet1, &wallet2, &token_account2)?;

    Ok(())
}
```

此函数概述了我们的主要流程。我们将在下一节中定义每个函数。在本例中，我们将使用[本地测试验证器](https://www.quicknode.com/guides/solana-development/getting-started/start-a-solana-local-validator?utm_source=internal&utm_campaign=guides&utm_content=wrapped-sol-basics)来运行代码；但是，您可以修改连接，使其与开发网或主网兼容，以用于实际应用程序。当您准备好开始与开发网或主网交互时，您将需要一个 RPC 端点。您可以在 [QuickNode.com](https://www.quicknode.com/signup?utm_source=internal&utm_campaign=guides&utm_content=a-complete-guide-to-wrapped-sol) 免费获取一个 RPC 端点，然后只需将连接更改为指向您的端点，如下所示：

```rust
let rpc_client = RpcClient::new_with_commitment(
    "https://example.solana-mainnet.quiknode.pro/123/".to_string(),
    CommitmentConfig::processed(),
);
```



### Create Wrapped SOL Operations 

让我们为想要执行的每个操作创建函数。

#### 1. Request Airdrop 

```rust
fn request_airdrop(
    rpc_client: &RpcClient,
    wallet: &Keypair,
) -> Result<(), Box<dyn std::error::Error>> {
    let airdrop_signature = rpc_client.request_airdrop(&wallet.pubkey(), 2 * 1_000_000_000)?;
    let recent_blockhash: solana_sdk::hash::Hash = rpc_client.get_latest_blockhash()?;

    rpc_client.confirm_transaction_with_spinner(
        &airdrop_signature,
        &recent_blockhash,
        CommitmentConfig::processed(),
    )?;
    println!("✅ - Step 1: Airdrop completed");
    Ok(())
}
```

我们只是使用 `request_airdrop` 函数向指定钱包请求空投 2 个 SOL 。我们将使用 `confirm_transaction_with_spinner` 函数在终端中以带有加载效果的方式确认此次空投交易。

#### 2. Wrap SOL

接下来，让我们创建一个函数来包装 SOL：

```rust
fn wrap_sol(
    rpc_client: &RpcClient,
    wallet: &Keypair,
) -> Result<Pubkey, Box<dyn std::error::Error>> {
    let associated_token_account =
        get_associated_token_address(&wallet.pubkey(), &native_mint::id());
    let instructions = vec![
        spl_associated_token_account::instruction::create_associated_token_account(
            &wallet.pubkey(),
            &wallet.pubkey(),
            &native_mint::id(),
            &spl_token::id(),
        ),
        system_instruction::transfer(&wallet.pubkey(), &associated_token_account, 1_000_000_000),
        spl_instruction::sync_native(&spl_token::id(), &associated_token_account)?,
    ];

    let recent_blockhash: solana_sdk::hash::Hash = rpc_client.get_latest_blockhash()?;
    let transaction = Transaction::new_signed_with_payer(
        &instructions,
        Some(&wallet.pubkey()),
        &[wallet],
        recent_blockhash,
    );
    rpc_client.send_and_confirm_transaction_with_spinner(&transaction)?;
    
    println!("✅ - Step 2: SOL wrapped");
    Ok(associated_token_account)
}
```

让我们来看看这个函数：

1. 首先，我们定义一个与钱包公钥和原生铸币相关的代币账户。
2. 接下来，我们组装一个包含三条指令的交易：
   - 我们为包装的 SOL 创建相关代币账户。
   - 我们将 1 SOL 转移到新创建的 ATA。
   - 我们调用 `sync_native` 指令来更新 ATA 上的数量字段，以匹配可用的 wrapped SOL 数量。
3. 最后，我们发送交易并记录成功消息。

#### 3. Transfer Wrapped SOL

现在，让我们创建一个函数来传输 wrapped SOL：

```rust
fn transfer_wrapped_sol(
    rpc_client: &RpcClient,
    from_wallet: &Keypair,
    to_wallet: &Keypair,
    from_token_account: &Pubkey,
) -> Result<Pubkey, Box<dyn std::error::Error>> {
    let to_token_account = get_associated_token_address(&to_wallet.pubkey(), &native_mint::id());

    let instructions = vec![
        spl_associated_token_account::instruction::create_associated_token_account(
            &from_wallet.pubkey(),
            &to_wallet.pubkey(),
            &native_mint::id(),
            &spl_token::id(),
        ),
        spl_instruction::transfer(
            &spl_token::id(),
            from_token_account,
            &to_token_account,
            &from_wallet.pubkey(),
            &[&from_wallet.pubkey()],
            500_000_000,
        )?,
    ];

    let recent_blockhash = rpc_client.get_latest_blockhash()?;
    let transaction = Transaction::new_signed_with_payer(
        &instructions,
        Some(&from_wallet.pubkey()),
        &[from_wallet],
        recent_blockhash,
    );

    rpc_client.send_and_confirm_transaction_with_spinner(&transaction)?;

    println!("✅ - Step 3: Transferred wrapped SOL");
    Ok(to_token_account)
}
```


由于 wrapped SOL 只是一个 SPL 代币，因此此函数只是一个简单的 SPL 代币转移。此函数为接收者创建一个 ATA（在 `native_mint` 上进行种子植入），然后将 wrapped SOL 的一半转移给它。

#### 4. Unwrap SOL 

让我们添加一个函数来解开 SOL。为此，我们需要做的就是关闭关联代币账户，并选择所需的地址来发送代币账户的 Lamport。

```rust
fn unwrap_sol(
    rpc_client: &RpcClient,
    wallet: &Keypair,
    token_account: &Pubkey,
) -> Result<(), Box<dyn std::error::Error>> {
    let instruction = spl_instruction::close_account(
        &spl_token::id(),
        token_account,
        &wallet.pubkey(),
        &wallet.pubkey(),
        &[&wallet.pubkey()],
    )?;

    let recent_blockhash = rpc_client.get_latest_blockhash()?;
    let transaction = Transaction::new_signed_with_payer(
        &[instruction],
        Some(&wallet.pubkey()),
        &[wallet],
        recent_blockhash,
    );

    rpc_client.send_and_confirm_transaction_with_spinner(&transaction)?;

    println!("✅ - Step 4: SOL unwrapped");
    Ok(())
}
```

此函数关闭 wrapped SOL 的关联代币账户，从而有效地解开 SOL。我们通过传入钱包的公钥来选择 Lamport 的目的地。



#### 5. Print Balances 

最后，让我们添加一个打印余额的功能：

```rust
fn print_balances(
    rpc_client: &RpcClient,
    wallet1: &Keypair,
    wallet2: &Keypair,
    token_account2: &Pubkey,
) -> Result<(), Box<dyn std::error::Error>> {
    let wallet1_balance = rpc_client.get_balance(&wallet1.pubkey())?;
    let wallet2_balance = rpc_client.get_balance(&wallet2.pubkey())?;
    let token_account2_balance = rpc_client.get_token_account_balance(token_account2)?;
    println!(
        "   - Wallet 1 SOL balance: {}",
        wallet1_balance as f64 / 1_000_000_000.0
    );
    println!(
        "   - Wallet 2 SOL balance: {}",
        wallet2_balance as f64 / 1_000_000_000.0
    );
    println!(
        "   - Wallet 2 wrapped SOL: {}",
        token_account2_balance.ui_amount.unwrap()
    );

    Ok(())
}
```

此功能获取并显示两个钱包的 SOL 和 wrapped SOL 余额。



### Run the Rust Script 

要运行 Rust 脚本，请确保您已运行本地 Solana 验证器：

```sh
solana-test-validator -r
```

然后，在单独的终端中，导航到您的 Rust 项目目录并运行：

```sh
cargo run
```

您应该看到类似于 JavaScript 版本的输出，显示空投、包装、转移和解包 SOL 的步骤，然后是最终余额：

```sh
cargo run
   Compiling wrapped-sol-rust v0.1.0 (/wrapped-sol-rust)
    Finished dev [unoptimized + debuginfo] target(s) in 2.45s
     Running `target/debug/wrapped-sol-rust`
✅ - Step 1: Airdrop completed
✅ - Step 2: SOL wrapped
✅ - Step 3: Transferred wrapped SOL
✅ - Step 4: SOL unwrapped
   - Wallet 1 SOL balance: 1.49794572
   - Wallet 2 SOL balance: 0
   - Wallet 2 wrapped SOL: 0.5
```

此 Rust 实现提供与 JavaScript 和 CLI 版本相同的功能，演示了如何使用 Solana Rust SDK 与包装后的 SOL 进行交互。



## Resources 

原文链接：https://www.quicknode.com/guides/solana-development/getting-started/a-complete-guide-to-wrapped-sol

- [如何查询 Solana 代币账户的余额](https://www.quicknode.com/guides/solana-development/spl-tokens/how-to-look-up-the-balance-of-a-token-account?utm_source=internal&utm_campaign=guides&utm_content=wrapped-sol-basics)
- [如何查找代币账户地址](https://www.quicknode.com/guides/solana-development/spl-tokens/how-to-look-up-the-address-of-a-token-account?utm_source=internal&utm_campaign=guides&utm_content=wrapped-sol-basics)