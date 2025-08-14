# Foundry Console.log

在 Foundry 框架中，`console.log` 是一个非常实用的调试工具，它允许你在智能合约执行过程中打印变量值和调试信息。这个功能通过 `console.sol` 库实现，让你可以在测试或脚本执行期间查看合约内部状态，而不必依赖事件日志或部署合约。

### 核心功能与用法

`console.log` 的基本用法类似于 JavaScript 或其他语言中的日志函数，但有一些 Solidity 特有的考虑：

```solidity
import "forge-std/console.sol";

contract MyContract {
    function myFunction() external {
        uint256 value = 42;
        console.log("Value is:", value); // 打印变量值
        console.log("Current block number:", block.number); // 打印全局变量
    }
}
```

### 支持的数据类型

`console.log` 支持多种 Solidity 数据类型：

1. **基本类型**：`uint`、`int`、`address`、`bool`、`bytes`、`string` 等
2. **复杂类型**：支持打印数组和结构体，但需要注意格式
3. **自定义日志**：支持格式化字符串和多个参数

### 实用技巧

1. **打印复杂数据结构**：

   ```solidity
   uint[] memory arr = new uint[](3);
   arr[0] = 1;
   arr[1] = 2;
   arr[2] = 3;
   console.logArray(arr); // 专门用于打印数组
   ```

2. **格式化输出**：

   ```solidity
   address addr = msg.sender;
   console.log("Sender address: %s", addr); // 使用格式化占位符
   ```

3. **条件日志**：

   ```solidity
   if (someCondition) {
       console.log("Condition is true!");
   }
   ```

4. **性能考量**：

   - 测试环境中使用 `console.log` 不会产生 gas 成本
   - 正式部署的合约应移除所有 `console` 调用，避免增加部署和执行成本（会产生 gas 成本）

5. **调试工具链整合**：

   - 使用 `forge test --vvvv` 命令查看详细日志输出
   - 在 `foundry.toml` 配置文件中设置默认日志级别

### 注意事项

1. **仅用于开发环境**：`console.sol` 不是以太坊虚拟机 (EVM) 的标准组件，因此不能在生产环境中使用。
2. **导入路径**：根据你使用的 Foundry 版本，导入路径可能有所不同。最新版本通常使用 `forge-std/console.sol`。
3. **兼容性**：某些特殊类型（如映射或嵌套复杂结构）可能无法直接打印，需要手动分解为基本类型。

### 示例合约

下面是一个完整的示例，展示如何在测试中使用 `console.log`：

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.17;

import "forge-std/Test.sol";
import "forge-std/console.sol";

contract MyContract {
    uint256 public counter;

    function increment() external {
        counter++;
    }
}

contract MyContractTest is Test {
    MyContract public contractInstance;

    function setUp() public {
        contractInstance = new MyContract();
    }

    function testIncrement() public {
        uint256 initialValue = contractInstance.counter();
        console.log("Initial counter value:", initialValue);

        contractInstance.increment();
        uint256 newValue = contractInstance.counter();
        console.log("New counter value:", newValue);

        assertEq(newValue, initialValue + 1);
    }
}
```

运行这个测试时，使用 `forge test --vvvv` 命令可以看到详细的日志输出，帮助你理解合约执行过程中的状态变化。

通过合理使用 `console.log`，你可以大大提高调试效率，更快地定位和解决智能合约中的问题。

### 在 foundry.toml 中配置日志级别

在 Foundry 框架中，你可以通过修改 `foundry.toml` 配置文件来设置默认日志级别，这样就不需要每次运行测试时都手动添加 `--vvvv` 等参数。这对于提高开发效率非常有帮助。

### 日志级别概述

Foundry 提供了 5 个日志级别，从低到高分别是：

- `error` (0)：只显示错误信息
- `warn` (1)：显示警告和错误信息
- `info` (2)：显示信息、警告和错误信息（默认级别）
- `debug` (3)：显示调试、信息、警告和错误信息
- `trace` (4)：显示所有信息，包括最详细的跟踪信息

要设置默认日志级别，需要在项目根目录下的 `foundry.toml` 文件中添加或修改 `[profile.default]` 部分：

```toml
[profile.default]
verbosity = 4  # 设置为4表示trace级别，显示最详细的日志
```

你也可以为不同的配置文件设置不同的日志级别：

```toml
[profile.default]
verbosity = 2  # 默认使用info级别

[profile.debug]
verbosity = 4  # 调试配置使用trace级别
```

### 按命令类型设置日志级别

你还可以针对特定命令设置日志级别，例如只为测试命令设置详细日志：

```toml
[profile.default]
# 其他配置...

[profile.default.test]
verbosity = 4  # 只对forge test命令使用trace级别

[profile.default.build]
verbosity = 2  # 对forge build使用info级别
```

### 覆盖配置文件的设置

即使在配置文件中设置了默认日志级别，你仍然可以在命令行中使用 `-v` 或 `--verbosity` 参数临时覆盖配置：

```bash
# 临时将日志级别设置为debug
forge test --verbosity 3
```

### 日志级别与 console.log 的关系

日志级别直接影响 `console.log` 输出的可见性：

- `error` (0)：不显示任何 `console.log` 输出
- `warn` (1)：不显示 `console.log` 输出
- `info` (2)：显示 `console.log` 输出（默认行为）
- `debug` (3)：显示所有 `console.log` 输出，包括详细调试信息
- `trace` (4)：显示最详细的 `console.log` 输出，包括合约内部状态变化

### 推荐配置

对于日常开发，推荐使用以下配置：

```toml
[profile.default]
# 其他配置...
verbosity = 2  # 默认使用info级别，显示console.log输出

[profile.trace]
verbosity = 4  # 专门用于深度调试的配置
```

这样，你可以通过以下命令在需要时切换到详细日志模式：

```bash
forge test --profile trace
```

通过合理配置 `foundry.toml` 中的日志级别，你可以在保持开发效率的同时，确保在需要时能够获得足够的调试信息。

参考文档：

https://getfoundry.sh/reference/forge-std/console-log/