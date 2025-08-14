# 008 创建 Solana WebSocket 订阅

创建事件监听器是一种有效的方法，可以提醒您的应用或用户您正在监控的内容发生了变化。Solana 内置了几个便捷的事件监听器（也称为订阅），让您可以轻松监听 Solana 区块链上的变化。以下是一些可能有用的示例：一个用于与链上程序（例如销售机器人）交互的 Discord 机器人，一个用于检查用户交易错误的 dApp，或者当用户钱包余额发生变化时发送的电话通知。

在本指南中，您将学习如何使用 Solana 的多种事件监听器方法和 QuickNode 的 Websocket 端点 (WSS://) 来监听链上的变化。具体来说，您将创建一个简单的 TypeScript 应用程序来跟踪账户（或钱包）中的更改。然后，您将学习如何使用 Solana 的取消订阅方法从应用程序中移除监听器。我们还将介绍 Solana 其他监听器方法的基础知识。

#### **What You Will Need**

- [Nodejs](https://nodejs.org/en/) 安装（16.15 或更高版本）
- 已安装 npm 或 yarn（我们将使用 yarn 来初始化我们的项目并安装必要的软件包。如果您喜欢使用 npm，也可以使用 npm）
- Typescript 和 [ts-node](https://www.npmjs.com/package/ts-node) 安装
- [Solana Web3](https://www.npmjs.com/package/@solana/web3.js)

## Set Up Your Environment

使用以下命令在终端中创建一个新的项目目录：

```sh
mkdir solana-subscriptions
cd solana-subscriptions
```

创建文件 **app.ts** ：

```sh
echo > app.ts
```

使用“--yes”标志初始化您的项目以使用新包的默认值：

```sh
yarn init --yes
#or
npm init --yes
```

#### **Install Solana Web3 dependencies :**

```sh
yarn add @solana/web3.js@1
#or
npm install @solana/web3.js@1
```


在您喜欢的代码编辑器中打开 **app.ts** ，并在第 1 行从 Solana Web3 库导入 **Connection** 、 **PublicKey** 和 **LAMPORTS_PER_SOL** ：

```javascript
import { Connection, PublicKey, LAMPORTS_PER_SOL } from "@solana/web3.js";
```


好了！一切准备就绪，可以出发了。



## Set Up Your Quicknode Endpoint 

要在 Solana 上构建，您需要一个 API 端点来连接网络。您可以选择使用公共节点，也可以部署和管理您自己的基础设施；但是，如果您希望响应速度提高 8 倍，您可以将繁重的工作交给我们。

我们将使用 Solana Devnet 节点。复制 HTTP Provider 和 WSS Provider链接：

![New Solana Endpoint](https://www.quicknode.com/guides/assets/images/0-9b8c136f77deeffa7ef3d7e66edc9bb8.png)



在 **app.js** 的第 3 行和第 4 行创建两个新变量来存储这些 URL：

```javascript
const WSS_ENDPOINT = 'wss://example.solana-devnet.quiknode.pro/000/'; // replace with your URL
const HTTP_ENDPOINT = 'https://example.solana-devnet.quiknode.pro/000/'; // replace with your URL
```



## Establish a Connection to Solana

在第 5 行，创建到 Solana 的新**连接** ：

```javascript
const solanaConnection = new Connection(HTTP_ENDPOINT,{wsEndpoint:WSS_ENDPOINT});
```

如果您以前创建过与 Solana 的连接实例，您可能会注意到我们的参数有所不同，尤其是 `{wsEndpoint:WSS\_ENDPOINT}` 。

**Connection** 类的构造函数允许我们传递一个可选的 **commitmentOrConfig 参数** 。 [**ConnectionConfig**](https://solana-labs.github.io/solana-web3.js/modules.html#ConnectionConfig)中可以包含一些有趣的选项，但今天我们将重点介绍可选参数 **wsEndpoint** 。该参数用于为完整节点 JSON RPC PubSub Websocket 端点提供端点 URL。在我们的例子中，它就是我们之前定义的 WSS 端点 **WSS_ENDPOINT** 。

如果不传递 **wsEndpoint** 会发生什么？Solana 使用 **makeWebsocketUrl** 函数来解决这个问题，该函数会将端点 URL 中的 *https* 替换为 *wss* 或将 *http* 替换为 *ws* ( [source](https://github.com/solana-labs/solana/blob/627d91fb20272afa5ce790fbec01b40a29fb9852/web3.js/src/connection.ts#L2452) )。由于所有 QuickNode HTTP 端点都具有一个具有相同身份验证令牌的对应 WSS 端点，因此可以省略此参数，除非您想为 Websocket 查询使用单独的端点。

让我们创建一些订阅！



## Create an Account Subscription 

要在 Solana 上追踪钱包，我们需要在 **solanaConnection** 上调用 **onAccountChange** 方法。我们将传递 **ACCOUNT_TO_WATCH** 、我们要搜索的钱包的公钥以及一个回调函数。我们创建了一个简单的日志，用于在检测到事件时提醒我们，并记录新的账户余额。请将此代码片段添加到代码中第 7 行 **solanaConnection** 声明之后：

```javascript
(async()=>{
    const ACCOUNT_TO_WATCH = new PublicKey('vines1vzrYbzLMRdu58ou5XTby4qAqVRLmqo36NKPTg'); // Replace with your own Wallet Address
    const subscriptionId = await solanaConnection.onAccountChange(
        ACCOUNT_TO_WATCH,
        (updatedAccountInfo) =>
            console.log(`---Event Notification for ${ACCOUNT_TO_WATCH.toString()}--- \nNew Account Balance:`, updatedAccountInfo.lamports / LAMPORTS_PER_SOL, ' SOL'),
        "confirmed"
    );
    console.log('Starting web socket, subscription ID: ', subscriptionId);
})()
```



## Build a Simple Test 

这段代码已经可以运行了，但我们需要添加一个功能来测试它是否正常工作。我们将添加一个 **sleep** 函数（用于添加时间延迟）和一个**空投**请求。在第 6 行的异步代码块之前，添加：

```javascript
const sleep = (ms:number) => {
    return new Promise(resolve => setTimeout(resolve, ms));
}
```


然后，在“启动 Web 套接字”日志后的异步代码块内，添加此空投调用：

```javascript
    await sleep(10000); //Wait 10 seconds for Socket Testing
    await solanaConnection.requestAirdrop(ACCOUNT_TO_WATCH, LAMPORTS_PER_SOL);
```

您的代码将在 socket 启动后等待 10 秒，以请求向钱包空投（注意：这仅适用于 devnet 和 testnet）。

我们的代码现在看起来像这样：

```javascript
import { Connection, PublicKey, LAMPORTS_PER_SOL, } from "@solana/web3.js";

const WSS_ENDPOINT = 'wss://example.solana-devnet.quiknode.pro/000/'; // replace with your URL
const HTTP_ENDPOINT = 'https://example.solana-devnet.quiknode.pro/000/'; // replace with your URL
const solanaConnection = new Connection(HTTP_ENDPOINT, { wsEndpoint: WSS_ENDPOINT });
const sleep = (ms:number) => {
    return new Promise(resolve => setTimeout(resolve, ms));
}

(async () => {
    const ACCOUNT_TO_WATCH = new PublicKey('vines1vzrYbzLMRdu58ou5XTby4qAqVRLmqo36NKPTg');
    const subscriptionId = await solanaConnection.onAccountChange(
        ACCOUNT_TO_WATCH,
        (updatedAccountInfo) =>
            console.log(`---Event Notification for ${ACCOUNT_TO_WATCH.toString()}--- \nNew Account Balance:`, updatedAccountInfo.lamports / LAMPORTS_PER_SOL, ' SOL'),
        "confirmed"
    );
    console.log('Starting web socket, subscription ID: ', subscriptionId);
    await sleep(10000); //Wait 10 seconds for Socket Testing
    await solanaConnection.requestAirdrop(ACCOUNT_TO_WATCH, LAMPORTS_PER_SOL);
})()
```



## Start Your Socket! 

让我们继续测试一下。在终端中，你可以输入 **ts-node app.ts** 来启动你的 Web socket！大约 10 秒后，你应该会看到如下终端回调日志：

```sh
solana-subscriptions % ts-node app.ts
Starting web socket, subscription ID:  0
---Event Notification for vines1vzrYbzLMRdu58ou5XTby4qAqVRLmqo36NKPTg--- 
New Account Balance: 88790.51694709  SOL
```


太棒了！你应该注意到，即使在事件通知之后，你的应用仍然保持打开状态。这是因为我们的订阅仍在监听账户的变化。我们需要一种方法来**取消**订阅监听器。按 Ctrl^C 即可停止该进程。



## Unsubscribe from Account Change Listener 

Solana 创建了一个内置方法 **removeAccountChangeListener** ，我们可以使用它来取消订阅我们的帐户变更监听器。该方法接受一个有效的 **subscriptionId** (number) 作为其唯一参数。在异步代码块中，在**空投**之后添加另一个 **sleep 函数** ，以便留出时间处理交易，然后调用 **removeAccountChangeListener** ：

```javascript
    await sleep(10000); //Wait 10 for Socket Testing
    await solanaConnection.removeAccountChangeListener(subscriptionId);
    console.log(`Websocket ID: ${subscriptionId} closed.`);
```

现在再次运行您的代码，您应该看到相同的序列，然后关闭 Websocket：

```sh
solana-subscriptions % ts-node app.ts
Starting web socket, subscription ID:  0
---Event Notification for vines1vzrYbzLMRdu58ou5XTby4qAqVRLmqo36NKPTg--- 
New Account Balance: 88791.51694709  SOL
Websocket ID: 0 closed.
```

干得好！如你所见，如果你想在某些事件发生后（例如，时间流逝、达到特定阈值、通知数量等）禁用监听器，这将非常有用。

我们已将该脚本的最终代码发布到[我们的 Github 仓库](https://github.com/quiknode-labs/technical-content/blob/main/solana/websockets/app.ts)供您参考。



## Other Solana Websocket Subscriptions 

Solana 还有其他几种类似的 Websocket 订阅/取消订阅方法，它们也非常有用。我们将在本节中简要介绍它们。

- **onProgramAccountChange** ：注册一个回调函数，当指定程序拥有的账户发生变化时调用。传入程序的公钥和一个可选的账户过滤器数组。此外，该函数还可以方便地跟踪用户代币账户的变更。使用 **removeProgramAccountChangeListener** 取消订阅。
- **onLogs** ：注册一个回调函数，每当日志发出时都会调用该回调函数——其用法类似于 **onAccountChange** ，只需传入一个有效的 *PublicKey* 即可。该回调函数将返回最近的交易 ID。使用 **removeOnLogsListener** 取消订阅。
- **onSlotChange** ：注册一个在插槽更改时调用的回调函数。此方法仅接受回调函数。使用 **removeSlotChangeListener** 取消订阅。
- **onSignature** ：注册一个回调函数，在签名更新时通过传递有效的*交易签名 (Transaction Signature)* 进行调用。无论交易是否发生错误，回调函数都会返回结果。使用 **removeSignatureListener** 取消订阅 **。**
- **onRootChange** ：注册一个回调函数，在根更改时进行调用。此方法仅接受回调函数。使用 **removeRootChangeListener** 取消订阅。

**注意：所有[取消订阅方法](https://www.quicknode.com/guides/solana-development/getting-started/how-to-create-websocket-subscriptions-to-solana-blockchain-using-typescript#unsubscribe-methods)都需要一个 subscriptionId  (number) 参数。 您可以在 [quicknode.com/docs/solana 的](https://www.quicknode.com/docs/solana?utm_source=internal&utm_campaign=guides&utm_content=how-to-create-websocket-subscriptions-to-solana-blockchain-using-typescript)文档中找到更多关于这些方法的信息。您可以随意修改 **app.js 文件来尝试其他订阅功能。



## Billing and Optimization

Websocket 方法的计费基于收到的响应数量，而不是订阅数量。例如，如果您开设了一个 `accountChange` 订阅并收到 100 个响应，则您的帐户将被收取 5,000 个积分（[ 每个响应 50 个积分 ](https://www.quicknode.com/api-credits/sol#:~:text=Value-,accountsubscribe,-50)x 100 个响应）。

为了优化您的订阅并确保您不会因不必要的订阅或不相关的回复而被收费，您应该考虑以下事项：

- 当您不再对接收数据感兴趣时，使用*取消订阅*方法来删除监听器，
- 利用适用方法的过滤器仅接收相关数据。



### Unsubscribe Methods

以下方法用于从订阅中删除监听器：

- [`accountUnsubscribe`](https://www.quicknode.com/docs/solana/accountUnsubscribe?utm_source=internal&utm_campaign=guides&utm_content=how-to-create-websocket-subscriptions-to-solana-blockchain-using-typescript) : 取消订阅帐户以接收帐户更新。
- [`logsUnsubscribe`](https://www.quicknode.com/docs/solana/logUnsubscribe?utm_source=internal&utm_campaign=guides&utm_content=how-to-create-websocket-subscriptions-to-solana-blockchain-using-typescript):  取消接收日志。
- [`programUnsubscribe`](https://www.quicknode.com/docs/solana/programUnsubscribe?utm_source=internal&utm_campaign=guides&utm_content=how-to-create-websocket-subscriptions-to-solana-blockchain-using-typescript):  取消订阅接收程序帐户更新。
- [`slotUnsubscribe`](https://www.quicknode.com/docs/solana/slotUnsubscribe?utm_source=internal&utm_campaign=guides&utm_content=how-to-create-websocket-subscriptions-to-solana-blockchain-using-typescript):  取消接收插槽更新。
- [`signatureUnsubscribe`](https://www.quicknode.com/docs/solana/signatureUnsubscribe?utm_source=internal&utm_campaign=guides&utm_content=how-to-create-websocket-subscriptions-to-solana-blockchain-using-typescript):  取消接收签名更新。
- [`rootUnsubscribe`](https://www.quicknode.com/docs/solana/rootUnsubscribe?utm_source=internal&utm_campaign=guides&utm_content=how-to-create-websocket-subscriptions-to-solana-blockchain-using-typescript):  取消接收根更新。



### Using Filters with logsSubscribe 

[`logsSubscribe`](https://www.quicknode.com/docs/solana/logsSubscribe?utm_source=internal&utm_campaign=guides&utm_content=how-to-create-websocket-subscriptions-to-solana-blockchain-using-typescript) 方法允许您过滤事务日志，从而显著减少收到的响应数量。以下是示例：

```javascript
// To subscribe to all logs (expensive and not recommended):
const subscriptionId = await connection.onLogs(
  'all',
  (logs) => {
    console.log('New log:', logs);
  }
);

// To subscribe only to logs mentioning a specific address:
const EXAMPLE_PUBLIC_KEY = new PublicKey('YOUR_ADDRESS_HERE');
const specificAddressSubscriptionId = await connection.onLogs(
  EXAMPLE_PUBLIC_KEY,
  (logs) => {
    console.log('Log mentioning specific address:', logs);
  }
);
```



### Using Filters with programSubscribe 

[`programSubscribe`](https://www.quicknode.com/docs/solana/programSubscribe?utm_source=internal&utm_campaign=guides&utm_content=how-to-create-websocket-subscriptions-to-solana-blockchain-using-typescript) 方法功能强大，但可能会生成许多响应。使用过滤器可以缩小接收数据的范围：

```javascript
const filters = [
  {
    memcmp: {
      offset: 0, // Specify the offset in the account data
      bytes: 'base58_encoded_bytes_here' // Specify the bytes to compare
    }
  },
  {
    dataSize: 128 // If your account data is fixed, you can filter by data size - update this value to match your account's data size
  }
];

const subscriptionId = await connection.onProgramAccountChange(
  new PublicKey('Your_Program_ID_Here'),
  (accountInfo) => {
    console.log('Account changed:', accountInfo);
  },
  'confirmed',
  filters
);
```

 在此示例中，我们同时使用了 `memcmp` 和 `dataSize` 过滤器。 `memcmp` 会比较账户数据中特定偏移量的一系列字节，而 `dataSize` 则会根据账户的数据大小进行过滤。这种组合可以显著减少您收到的响应数量，从而仅关注符合您特定条件的账户。有关使用 *GetProgramAccountsFilter* 的更多信息，请参阅 [Solana RPC 文档 ](https://www.quicknode.com/guides/solana-development/spl-tokens/how-to-get-all-tokens-held-by-a-wallet-in-solana/?utm_source=internal&utm_campaign=guides&utm_content=how-to-create-websocket-subscriptions-to-solana-blockchain-using-typescript)。



### 最佳实践

1. **过滤器越具体** ，收到的不相关回复就越少。
2. **根据您的用例使用适当的commitment levels** ：更高的commitment levels（如“finalized”）可能会导致更少但更确定的更新。
3. **定期审查并删除未使用的订阅** ：审核您的有效订阅并删除不再需要的订阅。
4. **监控您的使用情况** ：跟踪您收到的回复数量，并在必要时调整您的过滤器或订阅策略（ [QuickNode 仪表板 ](https://dashboard.quicknode.com/?utm_source=internal&utm_campaign=guides&utm_content=how-to-create-websocket-subscriptions-to-solana-blockchain-using-typescript)）。

通过实施这些优化策略，您可以确保只接收所需的数据，从而最大限度地减少不必要的计费信用使用，同时保持 Solana 应用程序的有效性。



## 替代解决方案

QuickNode 提供了多种从 Solana 获取实时数据的解决方案。请查看以下选项，找到适合您用例的工具：

- [WebSockets](https://www.quicknode.com/docs/solana/accountSubscribe?utm_source=internal&utm_campaign=guides&utm_content=how-to-create-websocket-subscriptions-to-solana-blockchain-using-typescript) ：如本指南所述，WebSockets 提供与 Solana 节点的直接连接以实现实时更新——这对于简单应用程序和快速开发来说非常理想。如果您想将 Solana WebSockets 与 Solana Kit 结合使用，请参阅我们的[指南：使用 WebSockets 和 Solana Kit 监控 Solana 帐户，](https://www.quicknode.com/guides/solana-development/tooling/web3-2/subscriptions?utm_source=internal&utm_campaign=guides&utm_content=how-to-create-websocket-subscriptions-to-solana-blockchain-using-typescript) 了解如何使用 Solana Kit 在 Solana 中设置 WebSocket 订阅。
- [Yellowstone Geyser gRPC](https://marketplace.quicknode.com/add-on/yellowstone-grpc-geyser-plugin?utm_source=internal&utm_campaign=guides&utm_content=how-to-create-websocket-subscriptions-to-solana-blockchain-using-typescript) ：Yellowstone Geyser gRPC 为流式传输 Solana 数据提供了强大的 gRPC 接口，并具有内置过滤和历史数据支持。
- [Streams](https://www.quicknode.com/streams/solana?utm_source=internal&utm_campaign=guides&utm_content=how-to-create-websocket-subscriptions-to-solana-blockchain-using-typescript) ：用于处理 Solana 数据并将其路由到多个目的地的托管解决方案，具有内置过滤和历史数据支持。



## 原文链接

https://www.quicknode.com/guides/solana-development/getting-started/how-to-create-websocket-subscriptions-to-solana-blockchain-using-typescript