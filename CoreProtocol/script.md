# script

![script-flow](../img/script-flow.png)

## 核心功能解析

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



---
```python
class ScriptError(Exception):
    """Exception used for script errors."""


OpCodes = Enumeration(
    "Opcodes",
    [
        ("OP_0", 0),
        ("OP_PUSHDATA1", 76),
        "OP_PUSHDATA2",
        "OP_PUSHDATA4",
        "OP_1NEGATE",
        "OP_RESERVED",
        "OP_1",
        "OP_2",
        "OP_3",
        "OP_4",
        "OP_5",
        "OP_6",
        "OP_7",
        "OP_8",
        "OP_9",
        "OP_10",
        "OP_11",
        "OP_12",
        "OP_13",
        "OP_14",
        "OP_15",
        "OP_16",
        "OP_NOP",
        "OP_VER",
        "OP_IF",
        "OP_NOTIF",
        "OP_VERIF",
        "OP_VERNOTIF",
        "OP_ELSE",
        "OP_ENDIF",
        "OP_VERIFY",
        "OP_RETURN",
        "OP_TOALTSTACK",
        "OP_FROMALTSTACK",
        "OP_2DROP",
        "OP_2DUP",
        "OP_3DUP",
        "OP_2OVER",
        "OP_2ROT",
        "OP_2SWAP",
        "OP_IFDUP",
        "OP_DEPTH",
        "OP_DROP",
        "OP_DUP",
        "OP_NIP",
        "OP_OVER",
        "OP_PICK",
        "OP_ROLL",
        "OP_ROT",
        "OP_SWAP",
        "OP_TUCK",
        "OP_CAT",
        "OP_SUBSTR",
        "OP_LEFT",
        "OP_RIGHT",
        "OP_SIZE",
        "OP_INVERT",
        "OP_AND",
        "OP_OR",
        "OP_XOR",
        "OP_EQUAL",
        "OP_EQUALVERIFY",
        "OP_RESERVED1",
        "OP_RESERVED2",
        "OP_1ADD",
        "OP_1SUB",
        "OP_2MUL",
        "OP_2DIV",
        "OP_NEGATE",
        "OP_ABS",
        "OP_NOT",
        "OP_0NOTEQUAL",
        "OP_ADD",
        "OP_SUB",
        "OP_MUL",
        "OP_DIV",
        "OP_MOD",
        "OP_LSHIFT",
        "OP_RSHIFT",
        "OP_BOOLAND",
        "OP_BOOLOR",
        "OP_NUMEQUAL",
        "OP_NUMEQUALVERIFY",
        "OP_NUMNOTEQUAL",
        "OP_LESSTHAN",
        "OP_GREATERTHAN",
        "OP_LESSTHANOREQUAL",
        "OP_GREATERTHANOREQUAL",
        "OP_MIN",
        "OP_MAX",
        "OP_WITHIN",
        "OP_RIPEMD160",
        "OP_SHA1",
        "OP_SHA256",
        "OP_HASH160",
        "OP_HASH256",
        "OP_CODESEPARATOR",
        "OP_CHECKSIG",
        "OP_CHECKSIGVERIFY",
        "OP_CHECKMULTISIG",
        "OP_CHECKMULTISIGVERIFY",
        "OP_NOP1",
        "OP_CHECKLOCKTIMEVERIFY",
        "OP_CHECKSEQUENCEVERIFY",
    ],
)
```
在你提供的代码片段中，OpCodes 是一个枚举类型，定义了比特币脚本中常见的操作码（OpCodes）。这些操作码是比特币脚本虚拟机的指令，用于描述如何处理交易的输入和输出。以下是代码的主要组成部分和功能：
1. Enumeration: 这是一个用于定义枚举的构造，允许你为一组相关的常量赋予名称。这里的名称是操作码的名称，值是对应的整数。
操作码列表: 列表中包含了比特币脚本中使用的各种操作码，例如：
OP_0 对应的值是 0
OP_PUSHDATA1 对应的值是 76
其他操作码如 OP_DUP、OP_HASH160 等也在列表中定义。
用途: 这些操作码用于比特币脚本的执行，指示虚拟机执行特定的操作，比如数据推送、条件判断、签名验证等。
通过使用枚举，代码的可读性和可维护性得到了提升，因为你可以使用操作码的名称而不是直接使用数字值，这样更容易理解代码的意图。
---         
```python   
# 这些断言用于确保特定的操作码被正确地映射到预期的值上，以避免创建无效的脚本。
# 这些值来自比特币脚本的定义，用于确保脚本的正确执行。
assert OpCodes.OP_DUP == 0x76
assert OpCodes.OP_HASH160 == 0xA9
assert OpCodes.OP_EQUAL == 0x87
assert OpCodes.OP_EQUALVERIFY == 0x88
assert OpCodes.OP_CHECKSIG == 0xAC
assert OpCodes.OP_CHECKMULTISIG == 0xAE
```
---


```python
def is_unspendable_legacy(script):
    # 检查脚本是否以OP_FALSE OP_RETURN或OP_RETURN开头
    # OP_FALSE OP_RETURN对应的字节码是b"\x00\x6a"
    # OP_RETURN对应的字节码是b"\x6a"
    return script[:2] == b"\x00\x6a" or (script and script[0] == 0x6A)
``` 
---         

```python
def is_unspendable_genesis(script):
    # 检查脚本是否以OP_FALSE OP_RETURN开头
    # OP_FALSE OP_RETURN对应的字节码是b"\x00\x6a"
    return script[:2] == b"\x00\x6a"
```
---         

```python
def _match_ops(ops, pattern):
    # 检查操作码是否与模式匹配
    if len(ops) != len(pattern):
        return False
    for op, pop in zip(ops, pattern, strict=False):
        if pop != op:
            # -1 表示 '数据推送'，其操作码是 (op, data) 元组
            if pop == -1 and isinstance(op, tuple):
                continue
            return False

    return True
```
---         



```python
class ScriptPubKey:
    """ScriptPubKey是一个处理交易输出脚本的类，该脚本提供了必要的条件以便进行消费。
    """

    TO_ADDRESS_OPS = (
        OpCodes.OP_DUP,
        OpCodes.OP_HASH160,
        -1,
        OpCodes.OP_EQUALVERIFY,
        OpCodes.OP_CHECKSIG,
    )
    TO_P2SH_OPS = (OpCodes.OP_HASH160, -1, OpCodes.OP_EQUAL)
    TO_PUBKEY_OPS = (-1, OpCodes.OP_CHECKSIG)

    @classmethod
    def P2SH_script(cls, hash160):
        """P2SH_script是一个类方法，用于生成P2SH脚本。
        """
        return bytes((OpCodes.OP_HASH160,)) + Script.push_data(hash160) + bytes((OpCodes.OP_EQUAL,))

    @classmethod
    def P2PKH_script(cls, hash160):
        """P2PKH_script是一个类方法，用于生成P2PKH脚本。
        """
        return (
            bytes((OpCodes.OP_DUP, OpCodes.OP_HASH160))
            + Script.push_data(hash160)
            + bytes((OpCodes.OP_EQUALVERIFY, OpCodes.OP_CHECKSIG))
        )
```
---         



```python
class Script:
    @classmethod
    def get_ops(cls, script):
        # 初始化操作码列表
        ops = []

        # 尝试解析脚本，可能会抛出异常
        try:
            n = 0
            while n < len(script):
                # 获取当前位置的操作码
                op = script[n]
                n += 1

                # 如果操作码小于或等于OP_PUSHDATA4，则表示后续有原始字节数据
                if op <= OpCodes.OP_PUSHDATA4:
                    # 根据操作码确定数据长度
                    if op < OpCodes.OP_PUSHDATA1:
                        dlen = op
                    elif op == OpCodes.OP_PUSHDATA1:
                        dlen = script[n]
                        n += 1
                    elif op == OpCodes.OP_PUSHDATA2:
                        (dlen,) = unpack_le_uint16_from(script[n : n + 2])
                        n += 2
                    else:
                        (dlen,) = unpack_le_uint32_from(script[n : n + 4])
                        n += 4
                    # 检查是否会超出脚本长度
                    if n + dlen > len(script):
                        raise IndexError
                    # 将操作码和数据作为元组存储
                    op = (op, script[n : n + dlen])
                    n += dlen

                # 将操作码添加到列表中
                ops.append(op)
        except Exception:
            # 如果解析过程中出现异常，可能是脚本被截断了
            raise ScriptError("truncated script")

        # 返回解析出的操作码列表
        return ops

    @classmethod
    def push_data(cls, data):
        """Returns the opcodes to push the data on the stack."""
        # 确保数据是bytes或bytearray类型
        assert isinstance(data, (bytes, bytearray))

        # 获取数据的长度
        n = len(data)
        # 根据数据长度选择合适的操作码
        if n < OpCodes.OP_PUSHDATA1:
            return bytes((n,)) + data
        if n < 256:
            return bytes((OpCodes.OP_PUSHDATA1, n)) + data
        if n < 65536:
            return bytes((OpCodes.OP_PUSHDATA2,)) + pack_le_uint16(n) + data
        return bytes((OpCodes.OP_PUSHDATA4,)) + pack_le_uint32(n) + data

    @classmethod
    def opcode_name(cls, opcode):
        # 根据操作码返回其名称
        if OpCodes.OP_0 < opcode < OpCodes.OP_PUSHDATA1:
            return f"OP_{opcode:d}"
        try:
            return OpCodes.whatis(opcode)
        except KeyError:
            return f"OP_UNKNOWN:{opcode:d}"

    @classmethod
    def dump(cls, script):
        # 获取脚本的操作码和数据
        opcodes, datas = cls.get_ops(script)
        # 遍历操作码和数据，打印详细信息
        for opcode, data in zip(opcodes, datas, strict=False):
            name = cls.opcode_name(opcode)
            if data is None:
                print(name)
            else:
                print(f"{name} {data.hex()} ({len(data):d} bytes)")
```
---         

