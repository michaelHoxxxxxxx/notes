核心功能解析
OpCodes 枚举
使用 Enumeration 定义了比特币脚本中常见的操作码（例如：OP_DUP、OP_HASH160）。
**用途：**这些操作码是比特币脚本虚拟机的指令，用于描述如何处理交易的输入和输出。
ScriptError 异常
用于捕获和标记脚本中的错误。
脚本相关的实用函数
is_unspendable_legacy(script) 和 is_unspendable_genesis(script)

判断脚本是否为不可花费的（OP_RETURN 类型）。
**用途：**标识无法花费的比特币输出（通常用于存储数据）。
_match_ops(ops, pattern)

用于匹配脚本的操作码序列是否符合某种模式。
ScriptPubKey 类
代表交易输出脚本 (tx output script)。
定义了几种常见的输出类型：
P2SH_script 和 P2PKH_script
构造 P2SH（Pay-to-Script-Hash）和 P2PKH（Pay-to-PubKey-Hash）类型的脚本。
常见的操作码模式
TO_ADDRESS_OPS：匹配地址相关的输出（典型的 P2PKH）。
TO_P2SH_OPS：匹配 P2SH 的输出。
Script 类
处理通用脚本的解析与操作。

get_ops(script)

从脚本中提取操作码序列（可能包含数据推送）。
**用途：**将比特币交易脚本解析为操作码和数据。
push_data(data)

根据数据长度选择合适的操作码，将数据推送到栈中。
**用途：**生成脚本时需要动态处理数据推送。
opcode_name(opcode)

根据操作码返回其名称（如 OP_DUP 或 OP_UNKNOWN）。
**用途：**为调试或日志生成可读脚本信息。
dump(script)

打印脚本的详细信息，包括每个操作码和对应的数据。
**用途：**调试脚本。
核心功能展示
脚本的解析与操作
get_ops 是脚本解析的核心函数，处理数据推送（如 PUSHDATA）并验证脚本完整性。
P2SH_script 和 P2PKH_script 用于生成标准脚本，方便钱包或节点创建交易。
常见操作码断言
assert OpCodes.OP_DUP == 0x76 等断言用于确保操作码的正确性。
**用途：**防止因枚举定义错误而导致脚本解析失败。
