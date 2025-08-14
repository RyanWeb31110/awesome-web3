# Foundry 框架下如何对 Solidity 智能合约进行测试

在区块链开发中，智能合约的安全性和正确性至关重要，而测试是保障合约质量的核心环节。Foundry 作为以太坊生态中主流的开发工具链，提供了强大的合约测试能力，支持单元测试、模糊测试、分叉测试等多种场景。本文将基于 Foundry 框架，详细讲解 Solidity 智能合约测试的全流程与核心技巧。

## 一、测试基础：从文件结构到测试用例编写

Foundry 的测试体系遵循简洁且规范的流程，从文件命名到测试函数定义均有明确约定，确保测试逻辑可维护、可扩展。

### 1. 测试文件规范

- **命名规则**：测试文件默认以`.t.sol`为后缀（如`Counter.t.sol`），也可使用`Test`后缀（如`CounterTest.sol`），便于 Foundry 识别测试目标。
- **目录结构**：通常将测试文件放在项目的`test`目录下，与源码目录`src`对应（如`test/Counter.t.sol`对应`src/Counter.sol`）。

### 2. 测试用例编写步骤

编写测试用例需遵循固定流程，确保能正确调用 Foundry 的测试功能：

1. **导入核心合约**：
   导入`forge-std/Test.sol`中的`Test`合约（提供日志和断言功能）和待测试的目标合约（如`Counter.sol`）。

   ```solidity
   import "forge-std/Test.sol";
   import "../src/Counter.sol"; // 导入目标合约
   ```

2. **继承 Test 合约**：
   测试合约需继承`Test`合约，以使用其内置的断言（如`assertEq`）、日志（如`console.log`）等功能。

   ```solidity
   contract CounterTest is Test { ... }
   ```

3. **初始化状态（可选）**：
   通过`setUp`函数在每个测试用例运行前执行初始化操作（如部署合约实例），确保测试环境一致。

   ```solidity
   Counter public counter;
   function setUp() public {
       counter = new Counter(); // 每个测试前部署新的Counter实例
   }
   ```

4. **定义测试函数**：

   - 以`test`为前缀的函数将作为独立测试用例执行（如`test_Increment`）。
   - 以`testFuzz`为前缀的函数为模糊测试用例，参数由 Foundry 随机生成（如`testFuzz_SetNumber`）。

### 3. 测试用例示例

以测试`Counter`合约的`increment`和`setNumber`函数为例：

```solidity
contract CounterTest is Test {
    Counter public counter;

    // 初始化：每个测试前部署Counter
    function setUp() public {
        counter = new Counter();
    }

    // 单元测试：测试increment功能
    function test_Increment() public {
        counter.increment();
        assertEq(counter.number(), 1); // 断言结果正确
    }

    // 模糊测试：随机输入测试setNumber
    function testFuzz_SetNumber(uint256 x) public {
        counter.setNumber(x);
        assertEq(counter.number(), x); // 断言设置结果正确
    }
}
```

**关键特性**：单元测试和模糊测试均为**无状态**，即一个测试用例对状态的修改不会影响其他测试（每个测试用例独立运行在`setUp`初始化的环境中）。

## 二、测试执行：命令与报告解析

Foundry 提供了丰富的命令用于执行测试、过滤用例和生成报告，满足不同场景的测试需求。

### 1. 基础测试命令

- **执行所有测试**：`forge test`
  该命令会自动编译测试文件（若未修改则跳过编译）并执行所有测试用例，输出通过 / 失败结果。

  ```bash
  > forge test
  Ran 2 tests for test/Counter.t.sol:CounterTest
  [PASS] testFuzz_SetNumber(uint256) (runs: 256, μ: 32120, ~: 32354)
  [PASS] test_Increment() (gas: 31852)
  Suite result: ok. 2 passed; 0 failed; 0 skipped
  ```

- **查看帮助**：`forge test -h`
  查看所有测试相关命令的参数说明（如过滤、日志级别等）。

### 2. 测试过滤与日志控制

- **过滤测试用例**：通过`--match-test`或`--match-contract`筛选特定测试：

  ```bash
  # 仅执行名称包含"Increment"的测试
  forge test --match-test "test_Increment"
  # 仅执行CounterTest合约中的测试
  forge test --match-contract "CounterTest"
  ```

- **日志详细程度**：通过`-v`系列参数控制输出详细度（`v`越多越详细）：

  - `-v`：基础日志；
  - `-vvv`：显示失败测试的堆栈跟踪；
  - `-vvvvv`：显示所有测试的堆栈跟踪和初始化日志。

### 3. Gas 报告与快照

Foundry 可生成合约函数的 Gas 消耗报告，帮助优化合约性能：

- **生成 Gas 报告**：

  ```bash
  forge test --gas-report
  ```

  输出示例（包含部署成本和函数调用的 Gas 统计）：

  ```plaintext
  src/MyERC20.sol:MyERC20 Contract
  Deployment Cost: 954682 | Deployment Size: 6217
  Function Name | Min    | Avg    | Median | Max    | #Calls
  balanceOf     | 2851   | 2851   | 2851   | 2851   | 512
  transfer      | 51894  | 52015  | 51918  | 52218  | 256
  ```

- **Gas 快照与对比**：

  - 生成快照：`forge snapshot`（默认保存为`.gas-snapshot`）；
  - 对比快照：`forge snapshot --diff .gas-snap2`（对比当前快照与`.gas-snap2`的差异）。
  - 对比快照，并显示两者之间的差异：`forge snapshot --check .gas-snap2`

## 三、作弊码（Cheatcodes）：模拟复杂场景的核心工具

Foundry 通过`vm`（虚拟机器）提供了一系列 “作弊码”，用于在测试中模拟区块链环境（如区块高度、时间）、修改状态（如账户余额）或验证合约行为（如事件、回滚），是测试复杂场景的关键。

### 1. 作弊码分类

作弊码按功能分为 9 大类，覆盖测试中的常见需求：

- **环境类**：修改 EVM 状态（如区块号、时间戳）；
- **断言类**：验证合约行为（如预期回滚、事件触发）；
- **模糊测试类**：配置模糊测试参数；
- **分叉类**：与主网等链交互的分叉测试；
- 其他：外部交互、实用工具、快照、RPC、文件处理等。

### 2. 常用作弊码详解

以下为开发中高频使用的作弊码及示例：

#### （1）环境模拟

- **`vm.roll(uint256 blockNumber)`**：修改当前区块号。

  ```solidity
  function test_Roll() public {
      uint256 newBlockNumber = 100;
      vm.roll(newBlockNumber); // 模拟区块号变为100
      assertEq(block.number, newBlockNumber); // 验证区块号
  }
  ```

- **`vm.warp(uint256 timestamp)`**：修改区块时间戳；`skip(uint256)`：向前推进时间。

  ```solidity
  function test_Warp() public {
      uint256 newTimestamp = 1693222800;
      vm.warp(newTimestamp); // 直接设置时间戳
      assertEq(block.timestamp, newTimestamp);
      
      skip(1000); // 时间向前推进1000秒
      assertEq(block.timestamp, newTimestamp + 1000);
  }
  ```

#### （2）状态修改

- **`vm.prank(address sender)`**：修改下一次调用的`msg.sender`；`vm.startPrank`/`vm.stopPrank`：批量修改发送者。

  ```solidity
  function test_Prank() public {
      address alice = makeAddr("alice"); // 生成测试地址
      vm.prank(alice); // 下一次调用的发送者为alice
      Owner o = new Owner(); // 部署合约时，msg.sender为alice
      assertEq(o.owner(), alice);
  }
  ```

- **`vm.deal(address to, uint256 amount)`**：设置账户的 ETH 余额。

  ```solidity
  function test_Deal() public {
      address alice = makeAddr("alice");
      vm.deal(alice, 1 ether); // 设置alice的ETH余额为1 ETH
      assertEq(alice.balance, 1 ether);
  }
  ```

- **`deal(address token, address to, uint256 amount)`**：设置 ERC20 代币余额（需导入`StdCheats`）。

  ```solidity
  function test_Deal_ERC20() public {
      MyERC20 token = new MyERC20("OS6", "OS6");
      address alice = makeAddr("alice");
      deal(address(token), alice, 100 ether); // 设置alice的代币余额为100 ETH
      assertEq(token.balanceOf(alice), 100 ether);
  }
  ```

#### （3）行为验证

- **`vm.expectRevert()`**：验证下一次调用是否回滚（支持字符串、错误码或自定义错误）。

  ```solidity
  // 验证字符串回滚信息
  function test_Revert() public {
      vm.startPrank(bob); // 非所有者调用
      vm.expectRevert("Only the owner can transfer"); // 预期回滚信息
      o.transferOwnership(alice); // 触发回滚
  }
  
  // 验证自定义错误
  error NotOwner(address caller);
  function test_Revert_CustomError() public {
      vm.expectRevert(abi.encodeWithSignature("NotOwner(address)", bob));
      o.transferOwnership2(alice); // 触发NotOwner错误
  }
  ```

- **`vm.expectEmit(bool checkTopic1, ...)`**：验证事件是否按预期触发（参数控制是否检查索引 topics）。

  ```solidity
  event OwnerTransfer(address indexed caller, address indexed newOwner);
  function test_Emit() public {
      vm.expectEmit(true, true, false, false); // 检查前两个indexed topic
      emit OwnerTransfer(address(this), bob); // 预期触发的事件
      o.transferOwnership(bob); // 执行操作，触发事件
  }
  ```

## 四、分叉测试：与主网合约交互

当需要测试合约与主网等现有链上合约（如 Uniswap、USDC）的交互时，可使用分叉测试（Fork Testing），无需在测试网部署完整环境。

### 1. 分叉测试的两种方式

- **命令行模式**：通过`--fork-url`指定 RPC 节点，直接在命令中启用分叉。

  ```bash
  forge test --fork-url https://mainnet.infura.io/v3/... --match-test "test_MainnetInteraction"
  ```

- **作弊码模式**：在代码中通过`vm.createSelectFork`创建分叉，更灵活。

  ```solidity
  function setUp() public {
      // 基于Sepolia测试网的8219000区块创建分叉
      sepoliaForkId = vm.createSelectFork(vm.rpcUrl("sepolia"), 8219000);
  }
  
  function test_Something() public {
      vm.selectFork(sepoliaForkId); // 切换到分叉链
      // 与主网合约交互（如检查某地址的USDC余额）
      MyERC20 usdc = MyERC20(0x...); // 主网USDC地址
      assertGe(usdc.balanceOf(someAddress), 1e18);
  }
  ```

## 五、模糊测试：用随机输入验证合约健壮性

模糊测试（Fuzz Testing）通过随机生成大量输入数据，测试合约在极端或异常情况下的行为，暴露潜在漏洞（如整数溢出、逻辑漏洞）。

### 1. 基础用法

模糊测试函数以`testFuzz`为前缀，参数由 Foundry 自动生成（如`uint256 x`会被随机赋值）。

```solidity
function testFuzz_SetNumber(uint256 x) public {
    counter.setNumber(x); // 随机输入x调用setNumber
    assertEq(counter.number(), x); // 验证结果
}
```

执行时，Foundry 默认生成 256 组随机数据，输出平均 Gas 等统计：

```plaintext
[PASS] testFuzz_SetNumber(uint256) (runs: 256, μ: 35731, ~: 35970)
```

### 2. 输入控制

通过`vm.assume`和`bound`限制输入范围，避免无效测试（如地址为 0、金额超出余额）。

```solidity
function testFuzz_ERC20Transfer(address to, uint256 amount) public {
    vm.assume(to != address(0)); // 排除零地址
    vm.assume(to != address(this)); // 排除当前合约地址
    amount = bound(amount, 0, 10000 * 10 ** 18); // 限制金额范围
    token.transfer(to, amount); // 安全测试转账
    assertEq(token.balanceOf(to), amount);
}
```

## 六、不变量测试：验证核心逻辑的恒定性

不变量测试（Invariant Testing）用于验证合约在任意随机操作序列下，核心规则始终成立（如 “ERC20 总供应量不变”“AMM 的 x*y=K”）。

### 1. 实现方式

- 测试函数以`invariant`为前缀；
- 通过`targetContract`指定测试目标，`targetSelector`指定需测试的函数。

```solidity
contract InvariantTest is Test {
    MyERC20 public token;
    function setUp() public {
        token = new MyERC20("Test", "TST");
        targetContract(address(token)); // 指定测试目标
    }

    // 验证总供应量不变（假设无 mint/burn）
    function invariant_TotalSupply() public {
        assertEq(token.totalSupply(), 1000 ether);
    }
}
```

执行命令：

```bash
forge test test/Invariant.t.sol -vv
```

## 总结

Foundry 框架通过灵活的测试结构、强大的作弊码工具和多样化的测试模式（单元、模糊、分叉、不变量），为 Solidity 合约测试提供了全方位支持。掌握 Foundry 测试不仅能显著提升合约的安全ß性，还能通过 Gas 报告优化性能，是区块链开发者的核心技能之一。建议从基础测试入手，逐步实践复杂场景，最终构建覆盖全生命周期的测试体系。