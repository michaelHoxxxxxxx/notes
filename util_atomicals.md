# Atomicals 协议工具类文档
`util_atomicals.py` 是 Atomicals 协议的核心工具类文件,提供了协议所需的基础功能实现。    
## 主要功能模块
### 1. 验证模块
#### 1.1 工作量证明验证
```python
def is_proof_of_work_prefix_match(tx_hash, powprefix, powprefix_ext=None)
```
验证交易哈希是否满足指定的工作量证明要求。
参数:
- tx_hash: 交易哈希 
- powprefix: 工作量证明前缀
- powprefix_ext: 扩展难度值(可选)
返回值:
- bool: 是否满足工作量证明要求
#### 1.2 Bitwork 字符串验证
```python
def is_valid_bitwork_string(bitwork)
```
验证 bitwork 字符串格式是否合法。
参数:
- bitwork: bitwork 字符串
返回值:
- bool: 是否合法
### 2. 名称系统
#### 2.1 Ticker 验证
```python
def is_valid_ticker_string(ticker)
```
验证代币简称(ticker)是否合法。
参数:
- ticker: 代币简称
返回值:
- bool: 是否合法

#### 2.2 Realm 验证
```python
def is_valid_realm_string_name(realm_name)
```
验证 realm 名称是否合法。
参数:
- realm_name: realm 名称
返回值:
- bool: 是否合法

### 3. 状态管理
#### 3.1 状态更新
```python
def apply_set_state_mutation(current_object, state_mutation_map, is_top_level)
```
应用状态变更。
参数:
- current_object: 当前对象
- state_mutation_map: 状态变更映射
- is_top_level: 是否为顶级对象
返回值:
- 更新后的对象
### 4. 操作解析
#### 4.1 协议操作解析
```python
def parse_protocols_operations_from_witness_array(tx, tx_hash, allow_args_bytes)
```
从交易见证数据中解析 Atomicals 协议操作。
参数:
- tx: 交易对象
- tx_hash: 交易哈希
- allow_args_bytes: 是否允许 args_bytes
返回值:
- 解析后的操作列表

## 错误处理
大多数函数在遇到错误时会:
1. 返回 False/None 表示验证失败
2. 抛出异常并附带详细错误信息

## 函数解析

```python
def pad_bytes_n(val, n):
    padlen = n
    if len(val) > padlen:
        raise ValueError("pad_bytes_n input val is out of range")
    new_val = val
    extra_bytes_needed = padlen - len(val)
    new_val = new_val + bytes(extra_bytes_needed)
    return new_val
```
--- 
- pad_bytes_n 函数的作用是将输入的字节序列 val 填充到指定的长度 n。如果 val 的长度超过了 n，则会抛出一个 ValueError。以下是函数的详细解释：
1. 参数：
 - val: 需要填充的字节序列。
 - n: 目标长度，函数会将 val 填充到这个长度。
2. 逻辑：
 - 首先，函数检查 val 的长度是否超过了 n。如果超过，则抛出一个 ValueError，提示输入的字节序列超出了范围。
 - 接着，计算需要填充的字节数 extra_bytes_needed，即 n 减去 val 的当前长度。
 - 然后，使用 bytes(extra_bytes_needed) 创建一个指定长度的空字节序列，并将其附加到 val 的末尾。
 - 最后，返回填充后的字节序列。
 - 这个函数通常用于确保字节序列的长度符合某个特定的要求，例如在加密或数据传输中，可能需要固定长度的字节序列。
--- 

```python
def pad_bytes64(val):
    return pad_bytes_n(val, 64)
```
--- 
- pad_bytes64 函数的作用是将输入的字节序列 val 填充到 64 字节长度。
- 这个函数通常用于确保字节序列的长度符合某个特定的要求，例如在加密或数据传输中，可能需要固定长度的字节序列。
--- 

```python
def unpack_mint_info(mint_info_value):
    if not mint_info_value:
        raise IndexError("unpack_mint_info mint_info_value is null. Index error.")
    return loads(mint_info_value)
```
--- 
- unpack_mint_info 函数的作用是将输入的 mint_info_value 解码为字典。
- 这个函数通常用于将字节序列解码为字典，例如在加密或数据传输中，可能需要将字节序列解码为字典。  
--- 

```python
def is_sanitized_dict_whitelist_only(d: dict, allow_bytes=False):
    if not isinstance(d, dict):
        return False
    for _, v in d.items():
        if isinstance(v, dict):
            return is_sanitized_dict_whitelist_only(v, allow_bytes)
        if not allow_bytes and isinstance(v, bytes):
            return False
```
--- 
- is_sanitized_dict_whitelist_only 函数的作用是检查输入的字典 d 是否只包含允许的键和值类型。
- 这个函数通常用于确保字典中的键和值类型符合某个特定的要求，例如在加密或数据传输中，可能需要确保字典中的键和值类型符合某个特定的要求。
---

```python
def is_integer_num(n):
    if isinstance(n, int):
        return True
    return False
```
---  
- is_integer_num 函数的作用是检查输入的 n 是否为整数。
- 这个函数通常用于确保输入的 n 是整数，例如在加密或数据传输中，可能需要确保输入的 n 是整数。
---  

```python
def is_hex_string(value):
    if not isinstance(value, str):
        return False
    try:
        int(value, 16)  # Throws ValueError if it cannot be validated as hex string
        return True
    except (ValueError, TypeError):
        pass
    return False
```
---
- is_hex_string 函数的作用是检查输入的 value 是否为十六进制字符串。
- 这个函数通常用于确保输入的 value 是十六进制字符串，例如在加密或数据传输中，可能需要确保输入的 value 是十六进制字符串。
--- 

```python
def is_hex_string_regex(value):
    if not isinstance(value, str):
        return False
    m = re.compile(r"^[a-z0-9]+$")
    if m.match(value):
        return True
    return False
```
--- 
- is_hex_string_regex 函数的作用是检查输入的 value 是否为十六进制字符串。
- 这个函数通常用于确保输入的 value 是十六进制字符串，例如在加密或数据传输中，可能需要确保输入的 value 是十六进制字符串。
--- 

```python
def is_atomical_id_long_form_string(value):
    if not value:
        return False
``` 
--- 
- is_atomical_id_long_form_string 函数的作用是检查输入的 value 是否为 Atomical ID 长格式字符串。
- 这个函数通常用于确保输入的 value 是 Atomical ID 长格式字符串，例如在加密或数据传输中，可能需要确保输入的 value 是 Atomical ID 长格式字符串。
--- 

```python
def is_atomical_id_short_form_string(value):
    if not value:
        return False
``` 
--- 
- is_atomical_id_short_form_string 函数的作用是检查输入的 value 是否为 Atomical ID 短格式字符串。
- 这个函数通常用于确保输入的 value 是 Atomical ID 短格式字符串，例如在加密或数据传输中，可能需要确保输入的 value 是 Atomical ID 短格式字符串。
--- 

```python
def is_atomical_id_long_form_bytes(value):
    if not isinstance(value, bytes):
        return False
``` 
--- 
- is_atomical_id_long_form_bytes 函数的作用是检查输入的 value 是否为 Atomical ID 长格式字节序列。
- 这个函数通常用于确保输入的 value 是 Atomical ID 长格式字节序列，例如在加密或数据传输中，可能需要确保输入的 value 是 Atomical ID 长格式字节序列。
--- 

```python
def is_compact_atomical_id(value):
    """Whether this is a compact atomical id or not"""
    if isinstance(value, int):
        return False
``` 
--- 
- is_compact_atomical_id 函数的作用是检查输入的 value 是否为 Atomical ID 短格式字符串。
- 这个函数通常用于确保输入的 value 是 Atomical ID 短格式字符串，例如在加密或数据传输中，可能需要确保输入的 value 是 Atomical ID 短格式字符串。
---

```python
def compact_to_location_id_bytes(value):
    """Convert the 36 byte atomical_id to the compact form with the "i" at the end"""
    if not value:
        raise TypeError("value in compact_to_location_id_bytes is not set")
```
--- 
- compact_to_location_id_bytes 函数的作用是将输入的 value 转换为 Atomical ID 短格式字符串。
- 这个函数通常用于将 Atomical ID 长格式字节序列转换为 Atomical ID 短格式字符串，例如在加密或数据传输中，可能需要将 Atomical ID 长格式字节序列转换为 Atomical ID 短格式字符串。
--- 

```python
def location_id_bytes_to_compact(location_id):
    (digit,) = unpack_le_uint32_from(location_id[32:])
    return f"{hash_to_hex_str(location_id[:32])}i{digit}"
```
--- 
- location_id_bytes_to_compact 函数的作用是将输入的 location_id 转换为 Atomical ID 短格式字符串。
- 这个函数通常用于将 Atomical ID 长格式字节序列转换为 Atomical ID 短格式字符串，例如在加密或数据传输中，可能需要将 Atomical ID 长格式字节序列转换为 Atomical ID 短格式字符串。
--- 

```python
def compact_to_location_id_bytes(value):
    """Convert the 36 byte atomical_id to the compact form with the "i" at the end"""
    if not value:
        raise TypeError("value in compact_to_location_id_bytes is not set")

    index_of_i = value.index("i")
    if index_of_i != 64:
        raise TypeError(f"{value} should be 32 bytes hex followed by i<number>")

    raw_hash = hex_str_to_hash(value[:64])

    if len(raw_hash) != 32:
        raise TypeError(f"{value} should be 32 bytes hex followed by i<number>")

    num = int(value[65:])

    if num < 0 or num > 100000:
        raise TypeError(f"{value} index output number was parsed to be less than 0 or greater than 100000")

    return raw_hash + pack_le_uint32(num)
```
--- 
- compact_to_location_id_bytes 函数的作用是将输入的 value 转换为 Atomical ID 长格式字节序列。
- 这个函数通常用于将 Atomical ID 短格式字符串转换为 Atomical ID 长格式字节序列，例如在加密或数据传输中，可能需要将 Atomical ID 短格式字符串转换为 Atomical ID 长格式字节序列。
--- 

```python
def location_id_bytes_to_compact(location_id):
    (digit,) = unpack_le_uint32_from(location_id[32:])
    return f"{hash_to_hex_str(location_id[:32])}i{digit}"
```
--- 
- location_id_bytes_to_compact 函数的作用是将输入的 location_id 转换为 Atomical ID 短格式字符串。
- 这个函数通常用于将 Atomical ID 长格式字节序列转换为 Atomical ID 短格式字符串，例如在加密或数据传输中，可能需要将 Atomical ID 长格式字节序列转换为 Atomical ID 短格式字符串。
--- 

```python
def get_tx_hash_index_from_location_id(location_id):
    (output_index,) = unpack_le_uint32_from(location_id[32:36])
    return location_id[:32], output_index
```
--- 
- get_tx_hash_index_from_location_id 函数的作用是从输入的 location_id 中提取出 tx_hash 和 output_index。
- 这个函数通常用于从 Atomical ID 短格式字符串中提取出 tx_hash 和 output_index，例如在加密或数据传输中，可能需要从 Atomical ID 短格式字符串中提取出 tx_hash 和 output_index。
--- 

```python
def is_valid_dmt_op_format(tx_hash, dmt_op):
    if not dmt_op or dmt_op["op"] != "dmt" or dmt_op["input_index"] != 0:
        return False, {}
    payload_data = dmt_op["payload"]
    # Just validate the properties are not set, or they are dicts
    # Do nothing with them for a DMT mint
    # This is just a data sanitation concern only
    metadata = payload_data.get("meta", {})
    if not isinstance(metadata, dict):
        return False, {}
    args = payload_data.get("args", {})
    if not isinstance(args, dict):
        return False, {}
    ctx = payload_data.get("ctx", {})
    if not isinstance(ctx, dict):
        return False, {}
    init = payload_data.get("init", {})
    if not isinstance(init, dict):
        return False, {}
    ticker = args.get("mint_ticker", None)
    if is_valid_ticker_string(ticker):
        return True, {"payload": payload_data, "$mint_ticker": ticker}
    return False, {}
```
--- 
- is_valid_dmt_op_format 函数的作用是检查输入的 dmt_op 是否为有效的分布式铸造(dmt)操作。
- 这个函数通常用于确保输入的 dmt_op 是有效的分布式铸造(dmt)操作，例如在加密或数据传输中，可能需要确保输入的 dmt_op 是有效的分布式铸造(dmt)操作。
--- 
    
```python
def is_validate_pow_prefix_string(pow_prefix, pow_prefix_ext):
    if not pow_prefix:
        return False
    m = re.compile(r"^[a-f0-9]{1,64}$")
    if pow_prefix:
        if pow_prefix_ext:
            if isinstance(pow_prefix_ext, int) and pow_prefix_ext >= 0 and pow_prefix_ext <= 15 and m.match(pow_prefix):
                return True
            else:
                return False
        if m.match(pow_prefix):
            return True
    return False
```
--- 
- is_validate_pow_prefix_string 函数的作用是检查输入的 pow_prefix 是否为有效的 Pow 前缀。
- 这个函数通常用于确保输入的 pow_prefix 是有效的 Pow 前缀，例如在加密或数据传输中，可能需要确保输入的 pow_prefix 是有效的 Pow 前缀。
---     

```python
def is_valid_bitwork_string(bitwork):
    if not bitwork:
        return None, None

    if not isinstance(bitwork, str):
        return None, None

    if bitwork.count(".") > 1:
        return None, None

    splitted = bitwork.split(".")
    prefix = splitted[0]
    ext = None
    if len(splitted) > 1:
        ext = splitted[1]
        try:
            ext = int(ext)  # Throws ValueError if it cannot be validated as hex string
        except (ValueError, TypeError):
            return None, None

    if is_validate_pow_prefix_string(prefix, ext):
        return bitwork, {"prefix": prefix, "ext": ext}
    return None, None
```
--- 
- is_valid_bitwork_string 函数的作用是解析输入的 bitwork 字符串，并返回 bitwork 和对应的 prefix 和 ext。
- 这个函数通常用于解析输入的 bitwork 字符串，并返回 bitwork 和对应的 prefix 和 ext，例如在加密或数据传输中，可能需要解析输入的 bitwork 字符串，并返回 bitwork 和对应的 prefix 和 ext。
--- 

```python
def is_bitwork_const(bitwork_val):
    return bitwork_val == "any"
```
--- 
- is_bitwork_const 函数的作用是检查输入的 bitwork_val 是否为常量 "any"。
- 这个函数通常用于检查输入的 bitwork_val 是否为常量 "any"，例如在加密或数据传输中，可能需要检查输入的 bitwork_val 是否为常量 "any"。
---         

```python
def has_requested_proof_of_work(operations_found_at_inputs):    
    if not operations_found_at_inputs:
        return False, None

    payload_dict = operations_found_at_inputs["payload"]
    args = payload_dict.get("args")
    if not isinstance(args, dict):
        return False, None
```
--- 
- has_requested_proof_of_work 函数的作用是检查输入的 operations_found_at_inputs 是否包含 proof of work 请求。
- 这个函数通常用于检查输入的 operations_found_at_inputs 是否包含 proof of work 请求，例如在加密或数据传输中，可能需要检查输入的 operations_found_at_inputs 是否包含 proof of work 请求。
---     
```python
def get_if_parent_spent_in_same_tx(parent_atomical_id_compact, expected_minimum_total_value, atomicals_spent_at_inputs):
    parent_atomical_id = compact_to_location_id_bytes(parent_atomical_id_compact)
    id_to_total_value_map = {}
    for _idx, atomical_entry_list in atomicals_spent_at_inputs.items():
        for atomical_entry in atomical_entry_list:
            atomical_id = atomical_entry["atomical_id"]
            # Only sum up the relevant atomical
            if atomical_id != parent_atomical_id:
                continue
            id_to_total_value_map[atomical_id] = id_to_total_value_map.get(atomical_id) or 0
            input_value = unpack_le_uint64(
                atomical_entry["value"][HASHX_LEN + SCRIPTHASH_LEN : HASHX_LEN + SCRIPTHASH_LEN + 8]
            )
            id_to_total_value_map[atomical_id] += input_value
    total_sum = id_to_total_value_map.get(parent_atomical_id)
    if total_sum is None:
        return False

    if total_sum >= expected_minimum_total_value:
        return True
    else:
        return False
```
--- 
- get_if_parent_spent_in_same_tx 函数的作用是检查输入的 parent_atomical_id_compact 是否在 atomicals_spent_at_inputs 中被花费。
- 这个函数通常用于检查输入的 parent_atomical_id_compact 是否在 atomicals_spent_at_inputs 中被花费，例如在加密或数据传输中，可能需要检查输入的 parent_atomical_id_compact 是否在 atomicals_spent_at_inputs 中被花费。
--- 
