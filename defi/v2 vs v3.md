V2 自动复投这个，今天我想了下，应该可以不需要。
V3 的 LP fee 是隔离于 liquidity 之外的，所以需要把 LP 存到合约，定期取出 LP fee 再复投回去提供流动性。
V2 的 LP fee 是通过 reserve 的增长，增加 LP token 的价值来实现的。所以要捐献流动性，只需要简单的 burn 调 LP token（比如打到黑洞地址），LP token 无法取出，此时 LP 对应的 token，及之后收取的手续费，都会一直留在池子里提供流动性。


V2 protocol fee 和返佣相关的问题，@Max 老马 @rick 都有问到，我统一在这里回答下：
V2 protocol fee 的收取机制和 V3 是不同的。V3 是交易时收取 From 和 to 两种 token，记录到一个单独的数据结构里。V2 是直接在算 amount 时，扣除手续费后给产出，所有的手续费累计在 reserve 里。也就是说，交易时，从 1 种 token 换出另一种给到了用户，但是扣留了手续费的部分在 reserve 里，而此时 LP token 数量没变，所以 LP 持有的 LP token 的价值变高了，隐含的收取了手续费。这是 LP fee 的部分。protocol fee 的部分，是在 https://github.com/thetafunction/orion-contract-v2/blob/main/contracts/PancakePair.sol#L89-L108 这个链接对应的函数里处理的，每次有人添加/移除流动性时，如果开启了 protocol fee，会计算 liquidity 的差值，得到 protocol fee 对应的 liquidity，增发 LP token 给 protocol fee 地址。
protocol fee 地址提取手续费时，做的实际是撤池子操作，burn 掉他收到的 LP token，根据此时池子内的 token 比例获得 token。
因此，对于返佣逻辑，这块可能不太好计算。一笔 swap 带来的手续费是留在池子的里部分 token，protocol fee 地址收到 LP token 有延迟，protocol fee 地址把 LP token 换回 token0/token1 又有延迟，变动很大。