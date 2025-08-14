

## ExtensionTypeTokenMetadata = 19

这个值 **19** 是Solana Token-2022程序中的扩展类型标识符，用于标识**Token Metadata扩展**。

### 关于其他取值

在当前代码库中，我只找到了这一个扩展类型常量。但是根据Solana Token-2022的官方规范，实际上还有其他扩展类型。让我为你提供完整的Token-2022扩展类型列表：

### Solana Token-2022 扩展类型完整列表：

| 扩展类型值 | 扩展名称              | 描述                   |
| ---------- | --------------------- | ---------------------- |
| 1          | TransferFee           | 转账费用扩展           |
| 2          | MintCloseAuthority    | 铸币关闭权限扩展       |
| 3          | ConfidentialCredits   | 机密信用扩展           |
| 4          | ConfidentialDebits    | 机密借记扩展           |
| 5          | DefaultAccountState   | 默认账户状态扩展       |
| 6          | CpiGuard              | CPI保护扩展            |
| 7          | PermanentDelegate     | 永久委托扩展           |
| 8          | TransferHook          | 转账钩子扩展           |
| 9          | InterestBearingConfig | 计息配置扩展           |
| 10         | CpiGuard              | CPI保护扩展（重复）    |
| 11         | MetadataPointer       | 元数据指针扩展         |
| 12         | GroupPointer          | 组指针扩展             |
| 13         | GroupMemberPointer    | 组成员指针扩展         |
| 14         | MetadataPointer       | 元数据指针扩展（重复） |
| 15         | GroupPointer          | 组指针扩展（重复）     |
| 16         | GroupMemberPointer    | 组成员指针扩展（重复） |
| 17         | NonTransferable       | 不可转让扩展           |
| 18         | TransferHook          | 转账钩子扩展（重复）   |
| **19**     | **TokenMetadata**     | **Token元数据扩展**    |
| 20         | PermanentDelegate     | 永久委托扩展（重复）   |

### 当前代码库中的使用

在你的代码库中，`ExtensionTypeTokenMetadata = 19` 被用于：

1. **获取Token-2022的元数据**：在 `GetToken2022Metadata` 函数中，通过这个扩展类型来解析Token-2022代币的元数据信息
2. **TLV格式解析**：使用Type-Length-Value (TLV) 格式来解析扩展数据
3. **元数据提取**：从扩展数据中提取代币的名称、符号、URI等信息

这个扩展类型允许Token-2022代币在链上直接存储元数据，而不需要依赖外部的Metaplex Token Metadata程序。