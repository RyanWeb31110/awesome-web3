# Uniswap V2 技术实现详解

## 1. 架构概览

Uniswap V2 是一个去中心化的自动化做市商(AMM)协议，基于恒定乘积公式实现代币交换。其核心架构包括以下组件：

### 核心合约结构
- **UniswapV2Factory**: 工厂合约，负责创建和管理交易对
- **UniswapV2Pair**: 交易对合约，实现具体的AMM逻辑
- **UniswapV2Router**: 路由合约，提供用户友好的接口
- **UniswapV2Library**: 库合约，包含核心计算函数

## 2. 核心合约实现

### 2.1 Factory 合约

```solidity
contract UniswapV2Factory {
    mapping(address => mapping(address => address)) public getPair;
    address[] public allPairs;
    
    event PairCreated(address indexed token0, address indexed token1, address pair, uint);
    
    function createPair(address tokenA, address tokenB) external returns (address pair) {
        require(tokenA != tokenB, 'UniswapV2: IDENTICAL_ADDRESSES');
        (address token0, address token1) = tokenA < tokenB ? (tokenA, tokenB) : (tokenB, tokenA);
        require(token0 != address(0), 'UniswapV2: ZERO_ADDRESS');
        require(getPair[token0][token1] == address(0), 'UniswapV2: PAIR_EXISTS');
        
        bytes memory bytecode = type(UniswapV2Pair).creationCode;
        bytes32 salt = keccak256(abi.encodePacked(token0, token1));
        assembly {
            pair := create2(0, add(bytecode, 32), mload(bytecode), salt)
        }
        
        IUniswapV2Pair(pair).initialize(token0, token1);
        getPair[token0][token1] = pair;
        getPair[token1][token0] = pair;
        allPairs.push(pair);
        
        emit PairCreated(token0, token1, pair, allPairs.length);
    }
}
```

**关键实现细节:**

- 使用 CREATE2 确保交易对地址的确定性
- 维护双向映射以快速查找交易对
- 代币地址排序确保唯一性

### 2.2 Pair 合约核心逻辑

#### 恒定乘积公式实现

```solidity
contract UniswapV2Pair {
    uint112 private reserve0;
    uint112 private reserve1;
    uint32  private blockTimestampLast;
    
    uint256 public price0CumulativeLast;
    uint256 public price1CumulativeLast;
    uint256 public kLast;
    
    modifier lock() {
        require(unlocked == 1, 'UniswapV2: LOCKED');
        unlocked = 0;
        _;
        unlocked = 1;
    }
    
    function swap(uint amount0Out, uint amount1Out, address to, bytes calldata data) 
        external lock {
        require(amount0Out > 0 || amount1Out > 0, 'UniswapV2: INSUFFICIENT_OUTPUT_AMOUNT');
        
        (uint112 _reserve0, uint112 _reserve1,) = getReserves();
        require(amount0Out < _reserve0 && amount1Out < _reserve1, 'UniswapV2: INSUFFICIENT_LIQUIDITY');
        
        uint balance0;
        uint balance1;
        {
            address _token0 = token0;
            address _token1 = token1;
            require(to != _token0 && to != _token1, 'UniswapV2: INVALID_TO');
            
            if (amount0Out > 0) _safeTransfer(_token0, to, amount0Out);
            if (amount1Out > 0) _safeTransfer(_token1, to, amount1Out);
            if (data.length > 0) IUniswapV2Callee(to).uniswapV2Call(msg.sender, amount0Out, amount1Out, data);
            
            balance0 = IERC20(_token0).balanceOf(address(this));
            balance1 = IERC20(_token1).balanceOf(address(this));
        }
        
        uint amount0In = balance0 > _reserve0 - amount0Out ? balance0 - (_reserve0 - amount0Out) : 0;
        uint amount1In = balance1 > _reserve1 - amount1Out ? balance1 - (_reserve1 - amount1Out) : 0;
        
        require(amount0In > 0 || amount1In > 0, 'UniswapV2: INSUFFICIENT_INPUT_AMOUNT');
        
        {
            uint balance0Adjusted = balance0.mul(1000).sub(amount0In.mul(3));
            uint balance1Adjusted = balance1.mul(1000).sub(amount1In.mul(3));
            require(balance0Adjusted.mul(balance1Adjusted) >= uint(_reserve0).mul(_reserve1).mul(1000**2), 'UniswapV2: K');
        }
        
        _update(balance0, balance1, _reserve0, _reserve1);
        emit Swap(msg.sender, amount0In, amount1In, amount0Out, amount1Out, to);
    }
}
```

**核心机制分析:**

1. **恒定乘积验证**: `x * y = k` 公式确保流动性守恒
2. **手续费机制**: 0.3% 交易费通过调整后余额计算实现
3. **重入保护**: `lock` 修饰符防止重入攻击
4. **闪电贷支持**: 通过回调机制实现原子性操作

## 3. AMM 数学模型

### 3.1 恒定乘积公式推导

基本公式: `x * y = k`

其中:
- x, y: 两种代币的储备量
- k: 恒定乘积常数

**交换计算:**

给定输入量 `Δx`，输出量 `Δy` 计算：

```
(x + Δx) * (y - Δy) = k = x * y
Δy = y - (x * y) / (x + Δx)
Δy = y * Δx / (x + Δx)
```

**含手续费的计算:**

实际实现中考虑 0.3% 手续费：

```
Δy = y * (Δx * 0.997) / (x + Δx * 0.997)
```

### 3.2 价格计算机制

```solidity
function getAmountOut(uint amountIn, uint reserveIn, uint reserveOut) 
    internal pure returns (uint amountOut) {
    require(amountIn > 0, 'UniswapV2Library: INSUFFICIENT_INPUT_AMOUNT');
    require(reserveIn > 0 && reserveOut > 0, 'UniswapV2Library: INSUFFICIENT_LIQUIDITY');
    
    uint amountInWithFee = amountIn.mul(997);
    uint numerator = amountInWithFee.mul(reserveOut);
    uint denominator = reserveIn.mul(1000).add(amountInWithFee);
    amountOut = numerator / denominator;
}
```

## 4. 流动性管理机制

### 4.1 添加流动性

```solidity
function mint(address to) external lock returns (uint liquidity) {
    (uint112 _reserve0, uint112 _reserve1,) = getReserves();
    uint balance0 = IERC20(token0).balanceOf(address(this));
    uint balance1 = IERC20(token1).balanceOf(address(this));
    uint amount0 = balance0.sub(_reserve0);
    uint amount1 = balance1.sub(_reserve1);
    
    bool feeOn = _mintFee(_reserve0, _reserve1);
    uint _totalSupply = totalSupply;
    
    if (_totalSupply == 0) {
        liquidity = Math.sqrt(amount0.mul(amount1)).sub(MINIMUM_LIQUIDITY);
        _mint(address(0), MINIMUM_LIQUIDITY);
    } else {
        liquidity = Math.min(amount0.mul(_totalSupply) / _reserve0, amount1.mul(_totalSupply) / _reserve1);
    }
    
    require(liquidity > 0, 'UniswapV2: INSUFFICIENT_LIQUIDITY_MINTED');
    _mint(to, liquidity);
    
    _update(balance0, balance1, _reserve0, _reserve1);
    if (feeOn) kLast = uint(reserve0).mul(reserve1);
    
    emit Mint(msg.sender, amount0, amount1);
}
```

**流动性代币计算:**

初始流动性: `L = √(x * y) - MINIMUM_LIQUIDITY`

后续流动性: `L = min(Δx * L_total / x, Δy * L_total / y)`

### 4.2 移除流动性

```solidity
function burn(address to) external lock returns (uint amount0, uint amount1) {
    (uint112 _reserve0, uint112 _reserve1,) = getReserves();
    address _token0 = token0;
    address _token1 = token1;
    uint balance0 = IERC20(_token0).balanceOf(address(this));
    uint balance1 = IERC20(_token1).balanceOf(address(this));
    uint liquidity = balanceOf[address(this)];
    
    bool feeOn = _mintFee(_reserve0, _reserve1);
    uint _totalSupply = totalSupply;
    
    amount0 = liquidity.mul(balance0) / _totalSupply;
    amount1 = liquidity.mul(balance1) / _totalSupply;
    
    require(amount0 > 0 && amount1 > 0, 'UniswapV2: INSUFFICIENT_LIQUIDITY_BURNED');
    
    _burn(address(this), liquidity);
    _safeTransfer(_token0, to, amount0);
    _safeTransfer(_token1, to, amount1);
    
    balance0 = IERC20(_token0).balanceOf(address(this));
    balance1 = IERC20(_token1).balanceOf(address(this));
    
    _update(balance0, balance1, _reserve0, _reserve1);
    if (feeOn) kLast = uint(reserve0).mul(reserve1);
    
    emit Burn(msg.sender, amount0, amount1, to);
}
```

## 5. 价格预言机实现

### 5.1 时间加权平均价格 (TWAP)

```solidity
function _update(uint balance0, uint balance1, uint112 _reserve0, uint112 _reserve1) private {
    require(balance0 <= uint112(-1) && balance1 <= uint112(-1), 'UniswapV2: OVERFLOW');
    
    uint32 blockTimestamp = uint32(block.timestamp % 2**32);
    uint32 timeElapsed = blockTimestamp - blockTimestampLast;
    
    if (timeElapsed > 0 && _reserve0 != 0 && _reserve1 != 0) {
        price0CumulativeLast += uint(UQ112x112.encode(_reserve1).uqdiv(_reserve0)) * timeElapsed;
        price1CumulativeLast += uint(UQ112x112.encode(_reserve0).uqdiv(_reserve1)) * timeElapsed;
    }
    
    reserve0 = uint112(balance0);
    reserve1 = uint112(balance1);
    blockTimestampLast = blockTimestamp;
    
    emit Sync(reserve0, reserve1);
}
```

**TWAP 计算:**

```
TWAP = (价格累积值_结束 - 价格累积值_开始) / (时间_结束 - 时间_开始)
```

### 5.2 固定点数学库 (UQ112x112)

```solidity
library UQ112x112 {
    uint224 constant Q112 = 2**112;
    
    function encode(uint112 y) internal pure returns (uint224 z) {
        z = uint224(y) * Q112;
    }
    
    function uqdiv(uint224 x, uint112 y) internal pure returns (uint224 z) {
        z = x / uint224(y);
    }
}
```

## 6. Router 合约功能

### 6.1 多跳交换路径

```solidity
function swapExactTokensForTokens(
    uint amountIn,
    uint amountOutMin,
    address[] calldata path,
    address to,
    uint deadline
) external ensure(deadline) returns (uint[] memory amounts) {
    amounts = UniswapV2Library.getAmountsOut(factory, amountIn, path);
    require(amounts[amounts.length - 1] >= amountOutMin, 'UniswapV2Router: INSUFFICIENT_OUTPUT_AMOUNT');
    
    TransferHelper.safeTransferFrom(
        path[0], msg.sender, UniswapV2Library.pairFor(factory, path[0], path[1]), amounts[0]
    );
    
    _swap(amounts, path, to);
}

function _swap(uint[] memory amounts, address[] memory path, address _to) internal {
    for (uint i; i < path.length - 1; i++) {
        (address input, address output) = (path[i], path[i + 1]);
        (address token0,) = UniswapV2Library.sortTokens(input, output);
        uint amountOut = amounts[i + 1];
        
        (uint amount0Out, uint amount1Out) = input == token0 ? 
            (uint(0), amountOut) : (amountOut, uint(0));
            
        address to = i < path.length - 2 ? 
            UniswapV2Library.pairFor(factory, output, path[i + 2]) : _to;
            
        IUniswapV2Pair(UniswapV2Library.pairFor(factory, input, output))
            .swap(amount0Out, amount1Out, to, new bytes(0));
    }
}
```

### 6.2 滑点保护机制

Router 合约通过以下方式实现滑点保护：

1. **最小输出保证**: `amountOutMin` 参数
2. **最大输入限制**: `amountInMax` 参数  
3. **截止时间**: `deadline` 参数防止长时间挂起

## 7. 闪电贷实现

### 7.1 闪电贷机制

```solidity
// 在 swap 函数中的闪电贷实现
if (data.length > 0) IUniswapV2Callee(to).uniswapV2Call(msg.sender, amount0Out, amount1Out, data);
```

**闪电贷流程:**

1. 用户调用 `swap` 函数，指定输出金额和回调数据
2. 合约先转出代币给用户
3. 调用用户合约的回调函数
4. 用户在回调中执行自定义逻辑
5. 回调完成后验证恒定乘积公式

### 7.2 闪电贷示例实现

```solidity
contract FlashLoanExample is IUniswapV2Callee {
    function executeFlashLoan(address pair, uint amount0, uint amount1) external {
        bytes memory data = abi.encode(msg.sender);
        IUniswapV2Pair(pair).swap(amount0, amount1, address(this), data);
    }
    
    function uniswapV2Call(address sender, uint amount0, uint amount1, bytes calldata data) external {
        address token0 = IUniswapV2Pair(msg.sender).token0();
        address token1 = IUniswapV2Pair(msg.sender).token1();
        
        // 执行套利逻辑
        performArbitrage(token0, token1, amount0, amount1);
        
        // 计算还款金额（包含手续费）
        uint amount0Repay = amount0 > 0 ? amount0 * 1000 / 997 + 1 : 0;
        uint amount1Repay = amount1 > 0 ? amount1 * 1000 / 997 + 1 : 0;
        
        // 还款
        if (amount0Repay > 0) IERC20(token0).transfer(msg.sender, amount0Repay);
        if (amount1Repay > 0) IERC20(token1).transfer(msg.sender, amount1Repay);
    }
}
```

## 8. 安全机制与最佳实践

### 8.1 重入攻击防护

```solidity
uint private unlocked = 1;
modifier lock() {
    require(unlocked == 1, 'UniswapV2: LOCKED');
    unlocked = 0;
    _;
    unlocked = 1;
}
```

### 8.2 整数溢出防护

```solidity
using SafeMath for uint;
require(balance0 <= uint112(-1) && balance1 <= uint112(-1), 'UniswapV2: OVERFLOW');
```

### 8.3 最小流动性锁定

```solidity
uint public constant MINIMUM_LIQUIDITY = 10**3;
```

防止流动性池被完全抽干的保护机制。

## 9. Gas 优化技术

### 9.1 存储优化

```solidity
struct Reserves {
    uint112 reserve0;
    uint112 reserve1;
    uint32 blockTimestampLast;
}
```

将三个变量打包到一个存储槽中，减少 SSTORE 操作成本。

### 9.2 计算优化

- 使用位运算替代除法运算
- 批量更新状态变量
- 优化循环结构

## 10. 总结

Uniswap V2 通过以下核心创新实现了高效的去中心化交易：

1. **恒定乘积公式**: 确保流动性守恒的数学基础
2. **自动做市**: 消除传统订单簿模式的复杂性
3. **流动性代币**: 激励用户提供流动性
4. **价格预言机**: 提供可靠的链上价格数据
5. **闪电贷**: 支持无抵押的原子性操作
6. **模块化设计**: Factory、Pair、Router 的清晰分工

这些技术创新为 DeFi 生态奠定了坚实基础，成为后续 AMM 协议发展的重要参考。