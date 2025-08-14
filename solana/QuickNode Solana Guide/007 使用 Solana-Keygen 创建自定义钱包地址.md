# 007 使用 Solana-Keygen 创建自定义钱包地址

您是否注意到某些 Solana 帐户地址具有自定义前缀或后缀（例如， *DUSTawucrTsGU8hcqRdHDCbuYhCPADMLM2VcCb8VnFnQ* ）？

![Example of Vanity URL - $DUST](https://www.quicknode.com/guides/assets/images/0-8232df710d97c1159c1885bfb152e06d.png)

Solana 的 CLI 有一些强大的工具可以生成您自己的自定义钱包地址。在本指南中，您将使用 Solana CLI 和 grind 工具创建您自己的自定义钱包地址。

#### What You Will Need 

- 命令行终端
- 已安装 [Solana CLI](https://docs.solana.com/cli/install-solana-cli-tools)

## Set Up Your Project 

使用以下命令在终端中创建一个新的项目目录：

```sh
mkdir custom-wallet
cd custom-wallet
```

检查你的 Solana CLI 工具是否已安装并更新。在终端中输入：

```sh
solana --version
```

要检查并安装更新，请在终端中输入：

```sh
solana-install update
```

您应该看到类似这样的内容：

![Solana CLI Update Complete](https://www.quicknode.com/guides/assets/images/1-f07be297ac880bede3ab94845426ffb1.png)

 

## Using Solana-Keygen

Solana CLI 有一个用于创建新密钥对的原生命令 **Solana-Keygen** 。 **Solana-Keygen** 命令遵循以下格式：

```sh
solana-keygen [OPTIONS] <SUBCOMMAND>
```

选项和子命令均在帮助菜单中列出。您可以在终端中输入 **help** 子命令来查看它们以及 **Solana-Keygen** 的所有功能：

```sh
solana-keygen --help
```

您应该会看到类似这样的帮助菜单：

![Solana CLI Help Menu](https://www.quicknode.com/guides/assets/images/2-41ece4665b99e8a1f1b4fd008f168a17.png)

您应该会看到一个子命令 **grind** 。我们将使用它来生成您的自定义钱包。此子命令会有效地生成密钥，直到找到符合您搜索条件的密钥。让我们测试一下！

```sh
solana-keygen grind --starts-with a23:1
```

您应该在终端中看到如下响应：

```sh
custom-wallet % solana-keygen grind --starts-with a23:1
Searching with 10 threads for:
        1 pubkey that starts with 'a23' and ends with ''
Wrote keypair to a23...mgr.json
```

太棒了！您刚刚创建了第一个自定义钱包地址，该地址以“a23”的第一个实例（由“:1”设置）开头。您应该在项目文件夹中看到一个 *.json* 文件，其中包含新地址的私钥。您可以使用 [Solana JS SDK](https://solana-labs.github.io/solana-web3.js/classes/Keypair.html#fromSeed) 将其导入到您的 Solana 项目中，方法如下：

```javascript
const keypair = Keypair.fromSecretKey(Uint8Array.from(ARRAY_IN_JSON FILE));
```

在查看 **solana-keygen** 的其他功能之前，让我们先来尝试一个以 *123456789* 开头的钱包。在终端中输入：

```sh
solana-keygen grind --starts-with 123456789:1
```

您的 CLI 应该每隔几秒钟向您显示一次更新，表明搜索仍在继续，并且未找到任何匹配项：

```sh
custom-wallet % solana-keygen grind --starts-with 123456789:1                          
Searching with 10 threads for:
        1 pubkey that starts with '123456789' and ends with ''
Searched 1000000 keypairs in 1s. 0 matches found.
Searched 2000000 keypairs in 3s. 0 matches found.
Searched 3000000 keypairs in 5s. 0 matches found.
etc.
```

按 **CTRL+C** 取消搜索，否则您可能需要等待很长时间。这是一个很好的示例，展示了 **grind 子**命令的工作原理。它会查找密钥对，检查其是否符合您的条件，然后不断尝试，直到满足您的条件为止。查找包含 9 个用户定义变量的地址比查找包含 3 个变量的地址要困难得多。也就是说，通常情况下，grind 地址通常只定义了 2-5 个字符。除此之外，您将耗费大量的计算资源并等待很长时间。

现在让我们探索一下 **solana-keygen** 的其他一些功能。在终端中输入：

```sh
solana-keygen grind --help
```

请注意上面的命令，因为如果您忘记如何使用该命令，它总是一个很好的参考点！

您将看到三个部分：**Usage** 用法、**FLAGS** 标志和 **OPTIONS** 选项（可以认为标志只是一种不同类型的选项，但 Solana CLI 将它们分开了）：
**Usage 用法**显示了我们命令的格式。我们始终以 **solana-keygen grind** 开头，后跟任何标志和选项 ： **solana-keygen grind [FLAGS] [OPTIONS] [Your Search Query]** 

- --starts-with PREFIX:COUNT，其中 *PREFIX* 是您正在搜索的自定义前缀， *COUNT* 是要生成的密钥对的数量（例如，--starts-with quick:2 将生成两个以 *quick* 开头的钱包地址的密钥对）。
- --ends-with SUFFIX:COUNT，其中 *SUFFIX* 是您正在搜索的自定义后缀， *COUNT* 是要生成的密钥对的数量（例如，--ends-with node:1 将生成一个以 *node* 结尾的钱包地址的密钥对）。
- --starts-and-ends-with PREFIX:SUFFIX:COUNT，其中 *PREFIX* 是您正在搜索的自定义前缀， *SUFFIX* 是您正在搜索的自定义后缀， *COUNT* 是要生成的密钥对的数量（例如，--starts-and-ends-with quick:node:3 将生成三个密钥对，其钱包地址以 *quick* 开头并以 *node* 结尾）。
- 注意：PREFIX 和 SUFFIX 必须是 Base58 类型（AZ、az、0-9），容易混淆的字符（零 (0)、大写 i (I) 和 o (O)、小写 l）除外。

**Other Options**

- --ignore-case 将使你的搜索不区分大小写（默认情况下区分大小写）
- --use-mnemonic 默认情况下，grind 函数会保存私钥，而不是助记词。此标志将使用助记词生成。此模式下，速度可能会显著下降。
- --no-outfile（必须与 --use-mnemonic 一起使用）。仅打印种子短语和公钥。不输出密钥对文件。
- --no-bip39-passphrase（必须与 --use-mnemonic 一起使用）。不提示输入 BIP39 密码（此密码可提高恢复种子短语的安全性，而非密钥对文件本身的安全性，因为密钥对文件本身以不安全的纯文本形式存储）。
- --word-count（必须与 --use-mnemonic 一起使用）。指定生成的种子短语的单词数（默认值：12）。
- --language `<LANGUAGE>` （必须与--use-mnemonic 一起使用）指定助记词的语言（英语、简体中文、繁体中文、日语、西班牙语、韩语、法语、意大利语）
- --num-threads `<NUMBER>` 指定研磨线程数 [默认值：10]
- -C, --config `<FILEPATH>` 要使用的配置文件

这里内容丰富，所以即使看不懂也不用担心。学习的最好方法就是亲身实践！



## Create a Custom Wallet 

是时候测试一下了！生成一个以您名字首字母开头、以您姓氏首字母结尾的单一地址（不区分大小写）。创建一个 24 个单词的日语助记符，该助记符不包含 *.json* 输出或 bip39 密码。

在您亲自尝试之后，您可以看看我们下面是如何做的：

```sh
solana-keygen grind --starts-and-ends-with A:M:1 --ignore-case --use-mnemonic --word-count 24 --language japanese --no-outfile --no-bip39-passphrase 
```

如果一切顺利，你应该会在终端中看到一条响应，显示“找到匹配的密钥”，其中包含你的钱包地址和种子短语！很酷吧？

如果您愿意，您可以使用以下命令对种子短语进行反向查找：

```sh
solana-keygen pubkey prompt://
```

我建议尝试每个选项，以了解一切如何运作，然后创建您梦想的地址！



## Stay Safe Out There! 

请记住，无论您如何创建新的种子或钱包，请务必遵循钱包安全最佳实践。我们发布了[指南：加密钱包简介及安全保障 ](https://www.quicknode.com/guides/knowledge-base/an-introduction-to-crypto-wallets-and-how-to-keep-them-secure)，以帮助您。

你的自定义钱包是否只是触及了你正在寻找的东西的表面？查看我们的[指南：如何使用 Solana 命名服务创建 .sol 域名 ](https://www.quicknode.com/guides/knowledge-base/how-to-create-a-sol-domain-using-solana-naming-service)，以创建自定义 .sol 域名。



## 原文链接：

https://www.quicknode.com/guides/solana-development/getting-started/how-to-create-a-custom-vanity-wallet-address-using-solana-cli