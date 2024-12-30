# coins
## 依赖包用途和功能介绍

    ```
import re
import struct
from dataclasses import dataclass
from decimal import Decimal
from functools import partial
from hashlib import sha256
from typing import Sequence, Tuple

import electrumx.lib.tx as lib_tx
import electrumx.lib.tx_axe as lib_tx_axe
import electrumx.lib.tx_dash as lib_tx_dash
import electrumx.lib.util as util
import electrumx.server.block_processor as block_proc
import electrumx.server.daemon as daemon
from electrumx.lib.hash import (
    HASHX_LEN,
    Base58,
    double_sha256,
    hash_to_hex_str,
    hex_str_to_hash,
)
from electrumx.lib.script import OpCodes, Script, ScriptError, ScriptPubKey, _match_ops
from electrumx.lib.tx import Tx
from electrumx.server.session.electrumx_session import (
    AuxPoWElectrumX,
    DashElectrumX,
    ElectrumX,
    NameIndexAuxPoWElectrumX,
    NameIndexElectrumX,
    SmartCashElectrumX,
)

    ```

### 1. 标准库导入
- `re`: 正则表达式模块，用于字符串匹配和解析。
- `struct`: 用于处理 C 风格的二进制数据。
- `dataclasses`: 提供了一种方便的方式来定义不可变数据对象。
- `decimal.Decimal`: 用于精确的浮点运算，常见于处理金融数据。
- `functools.partial`: 用于固定函数的部分参数或关键字参数，生成新的函数对象。
- `hashlib.sha256`: 用于计算 SHA-256 哈希，区块链中广泛使用。

### 2. 类型注解
- `Sequence` 和 `Tuple`: 类型提示，用于函数参数和返回值的静态检查。

### 3. 外部模块（可能是 ElectrumX 项目）
- **ElectrumX 模块**:
  - `electrumx.lib.tx`, `electrumx.lib.tx_axe`, `electrumx.lib.tx_dash`: 这些模块可能包含处理交易（Transaction）的逻辑，分别针对比特币或 Altcoin（例如 Dash 和 Axe）。
  - `electrumx.server.block_processor`, `electrumx.server.daemon`: 可能是用于区块链同步和与区块链节点通信的模块。
  - `electrumx.server.session.electrumx_session`: 包含多个 ElectrumX 会话类（如 `AuxPoWElectrumX`, `DashElectrumX`, 等），可能负责不同类型链的会话处理。

### 4. 特定功能的导入
- **哈希相关**:
  - `HASHX_LEN`, `Base58`, `double_sha256`, `hash_to_hex_str`, `hex_str_to_hash`: 这些工具可能用于数据哈希或编码操作。
- **脚本相关**:
  - `OpCodes`, `Script`, `ScriptError`, `ScriptPubKey`, `_match_ops`: 这些功能可能用于比特币交易的脚本解析和操作。
- **交易类**:
  - `electrumx.lib.tx.Tx`: 可能是交易对象的核心类。


    ``` 
  @dataclass
class Block:
    __slots__ = "raw", "header", "transactions"
    raw: bytes
    header: bytes
    transactions: Sequence[Tuple[Tx, bytes]]
    ``` 

### 1. `Block` 类
- 使用了 `__slots__` 来限制实例属性为 `raw`, `header`, 和 `transactions`，以节省内存。
- 属性类型说明：
  - `raw` 和 `header`: 二进制数据，分别表示区块的原始数据和区块头。
  - `transactions`: 一个序列，包含交易对象和对应的附加数据。

### 2. `CoinError` 异常类

    ``` 
class CoinError(Exception):
    """Exception raised for coin-related errors."""
    ``` 
    
- 定义了一个基础异常类，用于处理与加密货币相关的错误。

### 3. Mixin 类

    ```
class CoinHeaderHashMixin:
    @classmethod
    def header_hash(cls, header):
        """Given a header return hash"""
        return double_sha256(header)
    ```

- **`CoinHeaderHashMixin`**:
  - 定义了静态方法 `header_hash`，利用 `double_sha256` 计算区块头的双重哈希。

    ```
class CoinShortNameMixin:
    SHORTNAME: str
    ```
- **`CoinShortNameMixin`**:
  - 定义了一个 `SHORTNAME` 属性，用于表示加密货币的简短名称。

  ### 4. `Coin` 类

    ```


class Coin(CoinHeaderHashMixin, CoinShortNameMixin):
    """Base class of coin hierarchy."""

    REORG_LIMIT = 200
    # Not sure if these are coin-specific
    RPC_URL_REGEX = re.compile(".+@(\\[[0-9a-fA-F:]+\\]|[^:]+)(:[0-9]+)?")
    VALUE_PER_COIN = 100000000
    CHUNK_SIZE = 2016
    BASIC_HEADER_SIZE = 80
    STATIC_BLOCK_HEADERS = True
    SESSIONCLS = ElectrumX
    DEFAULT_MAX_SEND = 1000000
    DESERIALIZER = lib_tx.Deserializer
    DAEMON = daemon.Daemon
    BLOCK_PROCESSOR = block_proc.BlockProcessor
    HEADER_VALUES = (
        "version",
        "prev_block_hash",
        "merkle_root",
        "timestamp",
        "bits",
        "nonce",
    )
    HEADER_UNPACK = struct.Struct("< I 32s 32s I I I").unpack_from
    P2PKH_VERBYTE = bytes.fromhex("00")
    P2SH_VERBYTES = (bytes.fromhex("05"),)
    XPUB_VERBYTES = bytes("????", "utf-8")
    XPRV_VERBYTES = bytes("????", "utf-8")
    WIF_BYTE = bytes.fromhex("80")
    ENCODE_CHECK = Base58.encode_check
    DECODE_CHECK = Base58.decode_check
    GENESIS_HASH = "000000000019d6689c085ae165831e93" "4ff763ae46a2a6c172b3f1b60a8ce26f"
    GENESIS_ACTIVATION = 100_000_000

    MEMPOOL_HISTOGRAM_REFRESH_SECS = 500
    # first bin size in vbytes. smaller bins mean more precision but also bandwidth:
    MEMPOOL_COMPACT_HISTOGRAM_BINSIZE = 30_000

    # Peer discovery
    PEER_DEFAULT_PORTS = {"t": "50001", "s": "50002"}
    PEERS = []
    CRASH_CLIENT_VER = None
    BLACKLIST_URL = None
    ESTIMATEFEE_MODES = (None, "CONSERVATIVE", "ECONOMICAL")

    RPC_PORT: int
    NAME: str
    NET: str

    # only used for initial db sync ETAs:
    TX_COUNT_HEIGHT: int  # at a given snapshot of the chain,
    TX_COUNT: int  # there have been this many txs so far,
    TX_PER_BLOCK: int  # and from that height onwards, we guess this many txs per block

    @classmethod
    def lookup_coin_class(cls, name, net):
        """Return a coin class given name and network.

        Raise an exception if unrecognised."""
        req_attrs = ("TX_COUNT", "TX_COUNT_HEIGHT", "TX_PER_BLOCK")
        for coin in util.subclasses(Coin):
            if coin.NAME.lower() == name.lower() and coin.NET.lower() == net.lower():
                missing = [attr for attr in req_attrs if not hasattr(coin, attr)]
                if missing:
                    raise CoinError(f"coin {name} missing {missing} attributes")
                return coin
        raise CoinError(f"unknown coin {name} and network {net} combination")

    @classmethod
    def sanitize_url(cls, url):
        # Remove surrounding ws and trailing /s
        url = url.strip().rstrip("/")
        match = cls.RPC_URL_REGEX.match(url)
        if not match:
            raise CoinError(f'invalid daemon URL: "{url}"')
        if match.groups()[1] is None:
            url = f"{url}:{cls.RPC_PORT:d}"
        if not url.startswith(("http://", "https://")):
            url = f"http://{url}"
        return url + "/"

    @classmethod
    def max_fetch_blocks(cls, height):
        if height < 130000:
            return 1000
        return 100

    @classmethod
    def genesis_block(cls, block):
        """Check the Genesis block is the right one for this coin.

        Return the block less its unspendable coinbase.
        """
        header = cls.block_header(block, 0)
        header_hex_hash = hash_to_hex_str(cls.header_hash(header))
        if header_hex_hash != cls.GENESIS_HASH:
            raise CoinError(f"genesis block has hash {header_hex_hash} " f"expected {cls.GENESIS_HASH}")

        return header + b"\0"

    @classmethod
    def hashX_from_script(cls, script):
        """Returns a hashX from a script."""
        return sha256(script).digest()[:HASHX_LEN]

    @staticmethod
    def lookup_xverbytes(verbytes):
        """Return a (is_xpub, coin_class) pair given xpub/xprv verbytes."""
        # Order means BTC testnet will override NMC testnet
        for coin in util.subclasses(Coin):
            if verbytes == coin.XPUB_VERBYTES:
                return True, coin
            if verbytes == coin.XPRV_VERBYTES:
                return False, coin
        raise CoinError("version bytes unrecognised")

    @classmethod
    def address_to_hashX(cls, address):
        """Return a hashX given a coin address."""
        return cls.hashX_from_script(cls.pay_to_address_script(address))

    @classmethod
    def hash160_to_P2PKH_script(cls, hash160):
        return ScriptPubKey.P2PKH_script(hash160)

    @classmethod
    def hash160_to_P2PKH_hashX(cls, hash160):
        return cls.hashX_from_script(cls.hash160_to_P2PKH_script(hash160))

    @classmethod
    def pay_to_address_script(cls, address):
        """Return a pubkey script that pays to a pubkey hash.

        Pass the address (either P2PKH or P2SH) in base58 form.
        """
        raw = cls.DECODE_CHECK(address)

        # Require version byte(s) plus hash160.
        verbyte = -1
        verlen = len(raw) - 20
        if verlen > 0:
            verbyte, hash160 = raw[:verlen], raw[verlen:]
        if verbyte == cls.P2PKH_VERBYTE:
            return cls.hash160_to_P2PKH_script(hash160)
        if verbyte in cls.P2SH_VERBYTES:
            return ScriptPubKey.P2SH_script(hash160)

        raise CoinError(f"invalid address: {address}")

    @classmethod
    def privkey_WIF(cls, privkey_bytes, compressed):
        """Return the private key encoded in Wallet Import Format."""
        payload = bytearray(cls.WIF_BYTE + privkey_bytes)
        if compressed:
            payload.append(0x01)
        return cls.ENCODE_CHECK(payload)

    @classmethod
    def header_prevhash(cls, header):
        """Given a header return previous hash"""
        return header[4:36]

    @classmethod
    def static_header_offset(cls, height):
        """Given a header height return its offset in the headers file.

        If header sizes change at some point, this is the only code
        that needs updating."""
        assert cls.STATIC_BLOCK_HEADERS
        return height * cls.BASIC_HEADER_SIZE

    @classmethod
    def static_header_len(cls, height):
        """Given a header height return its length."""
        return cls.static_header_offset(height + 1) - cls.static_header_offset(height)

    @classmethod
    def block_header(cls, block, height):
        """Returns the block header given a block and its height."""
        return block[: cls.static_header_len(height)]

    @classmethod
    def block(cls, raw_block, height):
        """Return a Block namedtuple given a raw block and its height."""
        header = cls.block_header(raw_block, height)
        txs = cls.DESERIALIZER(raw_block, start=len(header)).read_tx_block()
        return Block(raw_block, header, txs)

    @classmethod
    def decimal_value(cls, value):
        """Return the number of standard coin units as a Decimal given a
        quantity of smallest units.

        For example 1 BTC is returned for 100 million satoshis.
        """
        return Decimal(value) / cls.VALUE_PER_COIN

    @classmethod
    def warn_old_client_on_tx_broadcast(cls, _client_ver):
        return False

    @classmethod
    def bucket_estimatefee_block_target(cls, n: int) -> int:
        """For caching purposes, it might be desirable to restrict the
        set of values that can be queried as an estimatefee block target.
        """
        return n
    ```
### 1. 静态常量与配置
- **链参数**:
  - `REORG_LIMIT`: 最大重组限制。
  - `CHUNK_SIZE`: 区块分片的大小。
  - `BASIC_HEADER_SIZE`: 区块头的固定大小。
  - `STATIC_BLOCK_HEADERS`: 是否使用固定大小的区块头。

- **哈希与编码**:
  - `GENESIS_HASH`: 创世区块的哈希值。
  - `P2PKH_VERBYTE`, `P2SH_VERBYTES`: 地址类型的版本字节。
  - `WIF_BYTE`: 私钥导入格式 (WIF) 的前缀字节。

- **交易与区块处理**:
  - `DESERIALIZER`: 用于交易反序列化。
  - `BLOCK_PROCESSOR`: 用于区块处理的类。

- **其他**:
  - `RPC_URL_REGEX`: 用于匹配和验证 RPC URL 的正则表达式。
  - `MEMPOOL_COMPACT_HISTOGRAM_BINSIZE`: 内存池直方图的最小分片大小。

### 2. 方法详解
#### **查找和验证**
- **`lookup_coin_class`**:
  - 根据币种名称和网络查找对应的子类。
  - 验证子类是否包含必要的属性 (`TX_COUNT`, `TX_COUNT_HEIGHT`, `TX_PER_BLOCK`)。
  - 未找到时抛出 `CoinError`。

- **`sanitize_url`**:
  - 清理 RPC URL 并添加默认端口号和协议。
  #### **区块和交易操作**
- **`block_header`**:
  - 从原始区块中提取区块头。
- **`block`**:
  - 返回一个包含区块原始数据、头部和交易信息的 `Block` 对象。
- **`genesis_block`**:
  - 验证创世区块的哈希是否匹配。
  - 返回去掉不可消费 coinbase 的创世区块。

  #### **地址和脚本**
- **`pay_to_address_script`**:
  - 根据 Base58 地址返回支付脚本，支持 P2PKH 和 P2SH 类型。
- **`address_to_hashX`**:
  - 从地址生成 `hashX`，用于优化地址查找。
- **`privkey_WIF`**:
  - 将私钥转换为 Wallet Import Format (WIF)。

#### **哈希与计算**
- **`header_hash`**:
  - 计算区块头的双重哈希。
- **`decimal_value`**:
  - 将最小单位值（如 satoshi）转换为标准单位值（如 BTC）。

#### **内存池和费用估算**
- **`MEMPOOL_HISTOGRAM_REFRESH_SECS`**:
  - 内存池直方图刷新间隔。
- **`bucket_estimatefee_block_target`**:
  - 限制费用估算的区块目标范围。

#### **版本与兼容性**
- **`lookup_xverbytes`**:
  - 根据版本字节查找对应的子类。
- **`warn_old_client_on_tx_broadcast`**:
  - 返回是否对旧客户端的交易广播进行警告。

  ## `AtomicalsCoinMixin` 是一个 Mixin 类
    ```
  class AtomicalsCoinMixin:
    ATOMICALS_ACTIVATION_HEIGHT: int
    ATOMICALS_ACTIVATION_HEIGHT_DMINT: int
    ATOMICALS_ACTIVATION_HEIGHT_COMMITZ: int
    ATOMICALS_ACTIVATION_HEIGHT_DENSITY: int
    ATOMICALS_ACTIVATION_HEIGHT_DFT_BITWORK_ROLLOVER: int
    ATOMICALS_ACTIVATION_HEIGHT_CUSTOM_COLORING: int
    ```


- **`ATOMICALS_ACTIVATION_HEIGHT`**:
  - 主激活高度，可能用于启用 Atomicals 功能的基础规则。

- **`ATOMICALS_ACTIVATION_HEIGHT_DMINT`**:
  - `DMINT` 的激活高度，具体功能可能涉及某种特定的链上操作或逻辑（如代币铸造）。

- **`ATOMICALS_ACTIVATION_HEIGHT_COMMITZ`**:
  - `COMMITZ` 的激活高度，可能与某种承诺（commitment）或交易确认逻辑有关。

- **`ATOMICALS_ACTIVATION_HEIGHT_DENSITY`**:
  - 用于控制密度相关逻辑的激活高度，可能影响链上资源分配或验证规则。

- **`ATOMICALS_ACTIVATION_HEIGHT_DFT_BITWORK_ROLLOVER`**:
  - 激活高度，可能与区块工作量（bitwork）或哈希难度调整规则相关。

- **`ATOMICALS_ACTIVATION_HEIGHT_CUSTOM_COLORING`**:
  - 自定义着色（custom coloring）的激活高度，可能用于启用某种标记或链上可视化特性。

## `AuxPowMixin` 是一个 Mixin 类

    ```

class AuxPowMixin:
    STATIC_BLOCK_HEADERS = False
    DESERIALIZER = lib_tx.DeserializerAuxPow
    SESSIONCLS = AuxPoWElectrumX
    TRUNCATED_HEADER_SIZE = 80
    # AuxPoW headers are significantly larger, so the DEFAULT_MAX_SEND from
    # Bitcoin is insufficient.  In Namecoin mainnet, 5 MB wasn't enough to
    # sync, while 10 MB worked fine.
    DEFAULT_MAX_SEND = 10000000

    @classmethod
    def header_hash(cls, header):
        """Given a header return hash"""
        return double_sha256(header[: cls.BASIC_HEADER_SIZE])

    @classmethod
    def block_header(cls, block, height):
        """Return the AuxPow block header bytes"""
        deserializer = cls.DESERIALIZER(block)
        return deserializer.read_header(cls.BASIC_HEADER_SIZE)

    ```
### 1. **属性定义**

- **`STATIC_BLOCK_HEADERS = False`**:
  - 标记是否使用固定大小的区块头。在 AuxPoW 中，区块头大小通常动态变化，因此设置为 `False`。

- **`DESERIALIZER = lib_tx.DeserializerAuxPow`**:
  - 定义反序列化器，用于解析 AuxPoW 区块数据。

- **`SESSIONCLS = AuxPoWElectrumX`**:
  - 定义会话类，可能用于网络交互或客户端通信。

- **`TRUNCATED_HEADER_SIZE = 80`**:
  - AuxPoW 区块的截断头部大小。80 字节是比特币等区块链的标准区块头大小。

- **`DEFAULT_MAX_SEND = 10000000`**:
  - 默认的最大数据发送大小为 10 MB，适应 AuxPoW 较大的区块头。在某些链（如 Namecoin）中，5 MB 可能不足以同步数据。

### 2. **方法详解**

#### **`header_hash(cls, header)`**
- **功能**:
  - 计算给定区块头的哈希值。
- **实现**:
  - 使用 `double_sha256` 计算双重哈希。
  - 仅对区块头的标准部分（`BASIC_HEADER_SIZE`）计算哈希，忽略扩展部分。

#### **`block_header(cls, block, height)`**
- **功能**:
  - 返回 AuxPoW 区块头部的字节表示。
- **实现**:
  - 使用反序列化器 (`cls.DESERIALIZER`) 读取区块的头部数据。
  - 截取 `BASIC_HEADER_SIZE` 部分作为头部。

## `EquihashMixin` 是一个 Mixin 类
    ```
    class EquihashMixin:
    STATIC_BLOCK_HEADERS = False
    BASIC_HEADER_SIZE = 140  # Excluding Equihash solution
    DESERIALIZER = lib_tx.DeserializerEquihash
    HEADER_VALUES = (
        "version",
        "prev_block_hash",
        "merkle_root",
        "reserved",
        "timestamp",
        "bits",
        "nonce",
    )
    HEADER_UNPACK = struct.Struct("< I 32s 32s 32s I I 32s").unpack_from

    @classmethod
    def block_header(cls, block, height):
        """Return the block header bytes"""
        deserializer = cls.DESERIALIZER(block)
        return deserializer.read_header(cls.BASIC_HEADER_SIZE)

    ```

### 1. **属性定义**

- **`STATIC_BLOCK_HEADERS = False`**:
  - 表明区块头的大小不是固定的，因为 Equihash 的解决方案部分（solution）通常是动态大小。

- **`BASIC_HEADER_SIZE = 140`**:
  - 定义了区块头的基本大小，不包含 Equihash 解决方案部分。

- **`DESERIALIZER = lib_tx.DeserializerEquihash`**:
  - 自定义的反序列化器，专用于处理 Equihash 区块数据。

- **`HEADER_VALUES`**:
  - 描述区块头字段的结构，包括：
    - `version`: 协议版本。
    - `prev_block_hash`: 前一个区块的哈希。
    - `merkle_root`: Merkle 树根。
    - `reserved`: 预留字段，通常为 32 字节。
    - `timestamp`: 区块时间戳。
    - `bits`: 用于目标难度的压缩表示。
    - `nonce`: 随机值，用于工作量证明。

- **`HEADER_UNPACK`**:
  - 使用 `struct.Struct` 定义区块头的二进制解析格式。
  - 格式说明：
    - `<`: 小端字节序。
    - `I`: 4 字节无符号整数。
    - `32s`: 32 字节字符串（如哈希值）。
    - `32s`: 另一个 32 字节字符串（如 Merkle 根）。
### 2. **方法详解**

#### **`block_header(cls, block, height)`**
- **功能**:
  - 返回区块头的字节表示，不包括 Equihash 解决方案部分。
- **实现**:
  - 使用自定义的 `DESERIALIZER` 解析区块数据。
  - 读取区块的前 `BASIC_HEADER_SIZE` 字节作为头部数据。

## `ScryptMixin` 是一个专为支持 Scrypt 哈希算法的区块链设计的 Mixin 类。

    ```
class ScryptMixin(CoinHeaderHashMixin):
    DESERIALIZER = lib_tx.DeserializerTxTime
    HEADER_HASH = None

    @classmethod
    def header_hash(cls, header):
        """Given a header return the hash."""
        if cls.HEADER_HASH is None:
            # Requires OpenSSL 1.1.0+
            from hashlib import scrypt

            cls.HEADER_HASH = lambda x: scrypt(x, salt=x, n=1024, r=1, p=1, dklen=32)

        (version,) = util.unpack_le_uint32_from(header)
        if version > 6:
            return super().header_hash(header)
        else:
            return cls.HEADER_HASH(header)


    ```

## 类结构与功能

### 1. **属性定义**

- **`DESERIALIZER = lib_tx.DeserializerTxTime`**:
  - 指定了用于解析包含时间戳的交易数据的反序列化器。

- **`HEADER_HASH = None`**:
  - 用于存储 Scrypt 哈希函数的引用。
  - 延迟初始化方式确保只有在首次使用时才加载 `scrypt` 哈希函数，减少资源消耗。
### 2. **方法详解**

#### **`header_hash(cls, header)`**
- **功能**:
  - 返回给定区块头的哈希值。
  - 支持基于版本的条件逻辑：
    - 如果版本号大于 6，调用父类的 `header_hash` 方法（通常是双重 SHA-256）。
    - 如果版本号小于或等于 6，使用 Scrypt 哈希算法。
- **实现**:
  1. 延迟加载 Scrypt 哈希函数：
     - 如果 `HEADER_HASH` 尚未定义，则动态引入 `hashlib.scrypt`。
     - 定义 Scrypt 哈希的参数：`n=1024`, `r=1`, `p=1`, `dklen=32`。
  2. 根据区块头的版本号决定使用哪种哈希算法：
     - 使用 `util.unpack_le_uint32_from` 从头部提取版本号。
     - 低版本区块使用 Scrypt 哈希。
     - 高版本区块使用父类的哈希逻辑（通常是双重 SHA-256）。

## `KomodoMixin` 是一个为 Komodo 区块链设计的 Mixin 类
定义了与 Komodo 区块链特性相关的基本参数和功能。Komodo 是基于 Zcash 分叉的区块链平台，因此此类继承了一些 Zcash 的特性。
    
    ```
class KomodoMixin:
    P2PKH_VERBYTE = bytes.fromhex("3C")
    P2SH_VERBYTES = (bytes.fromhex("55"),)
    WIF_BYTE = bytes.fromhex("BC")
    GENESIS_HASH = "027e3758c3a65b12aa1046462b486d0a" "63bfa1beae327897f56c5cfb7daaae71"
    DESERIALIZER = lib_tx.DeserializerZcash

    ```


### 1. **属性定义**

- **`P2PKH_VERBYTE = bytes.fromhex("3C")`**:
  - Komodo 的 Pay-to-PubKey-Hash (P2PKH) 地址版本字节。通常用于生成以特定前缀开头的地址。

- **`P2SH_VERBYTES = (bytes.fromhex("55"),)`**:
  - Komodo 的 Pay-to-Script-Hash (P2SH) 地址版本字节。支持脚本化交易，例如多签名交易。

- **`WIF_BYTE = bytes.fromhex("BC")`**:
  - Wallet Import Format (WIF) 的版本字节，用于私钥导入和导出。

- **`GENESIS_HASH = "027e3758c3a65b12aa1046462b486d0a63bfa1beae327897f56c5cfb7daaae71"`**:
  - Komodo 的创世区块哈希值，用于验证链的起始点是否正确。

- **`DESERIALIZER = lib_tx.DeserializerZcash`**:
  - 定义了区块和交易的反序列化器，使用的是 `DeserializerZcash`，表明 Komodo 区块链在设计上继承了 Zcash 的数据结构。

---

## `BitcoinMixin` 是一个 Mixin 类，为比特币（Bitcoin）链定义了一些基础属性和参数。
    ```

class BitcoinMixin(CoinShortNameMixin):
    SHORTNAME = "BTC"
    NET = "mainnet"
    XPUB_VERBYTES = bytes.fromhex("0488b21e")
    XPRV_VERBYTES = bytes.fromhex("0488ade4")
    RPC_PORT = 8332

    ```
### 1. **属性定义**

- **`SHORTNAME = "BTC"`**:
  - 定义比特币的简称，通常用于显示或标识比特币链。

- **`NET = "mainnet"`**:
  - 指定当前链的网络类型为主网（`mainnet`）。可能在其他环境（如测试网 `testnet` 或回归测试网 `regtest`）中被子类覆盖。

- **`XPUB_VERBYTES = bytes.fromhex("0488b21e")`**:
  - 表示扩展公钥（Extended Public Key）的版本字节，用于生成符合 BIP-32 标准的比特币扩展公钥地址。

- **`XPRV_VERBYTES = bytes.fromhex("0488ade4")`**:
  - 表示扩展私钥（Extended Private Key）的版本字节，用于生成符合 BIP-32 标准的比特币扩展私钥。

- **`RPC_PORT = 8332`**:
  - 指定比特币节点的远程过程调用（RPC）默认端口号，主网中默认为 `8332`。

---

##  `NameMixin` 是一个用于解析和处理区块链脚本中包含的“名称前缀”（name prefix）的 Mixin 类。

    ```
class NameMixin:
    DATA_PUSH_MULTIPLE = -2

    @classmethod
    def interpret_name_prefix(cls, script, possible_ops):
        """Interprets a potential name prefix

        Checks if the given script has a name prefix.  If it has, the
        name prefix is split off the actual address script, and its parsed
        fields (e.g. the name) returned.

        possible_ops must be an array of arrays, defining the structures
        of name prefixes to look out for.  Each array can consist of
        actual opcodes, -1 for ignored data placeholders, -2 for
        multiple ignored data placeholders and strings for named placeholders.
        Whenever a data push matches a named placeholder,
        the corresponding value is put into a dictionary the placeholder name
        as key, and the dictionary of matches is returned."""

        try:
            ops = Script.get_ops(script)
        except ScriptError:
            return None, script

        name_op_count = None
        for pops in possible_ops:
            # Start by translating named placeholders to -1 values, and
            # keeping track of which op they corresponded to.
            template = []
            named_index = {}

            n = len(pops)
            offset = 0
            for i, op in enumerate(pops):
                if op == cls.DATA_PUSH_MULTIPLE:
                    # Emercoin stores value in multiple placeholders
                    # Script structure: https://git.io/fjuRu
                    added, template = cls._add_data_placeholders_to_template(ops[i:], template)
                    offset += added - 1  # subtract the "DATA_PUSH_MULTIPLE" opcode
                elif isinstance(op, str):
                    template.append(-1)
                    named_index[op] = i + offset
                else:
                    template.append(op)
            n += offset

            if not _match_ops(ops[:n], template):
                continue

            name_op_count = n
            named_values = {key: ops[named_index[key]] for key in named_index}
            break

        if name_op_count is None:
            return None, script

        name_end_pos = cls.find_end_position_of_name(script, name_op_count)

        address_script = script[name_end_pos:]
        return named_values, address_script

    @classmethod
    def _add_data_placeholders_to_template(cls, opcodes, template):
        num_dp = cls._read_data_placeholders_count(opcodes)
        num_2drop = num_dp // 2
        num_drop = num_dp % 2

        two_drops = [OpCodes.OP_2DROP] * num_2drop
        one_drops = [OpCodes.OP_DROP] * num_drop

        elements_added = num_dp + num_2drop + num_drop
        placeholders = [-1] * num_dp
        drops = two_drops + one_drops

        return elements_added, template + placeholders + drops

    @classmethod
    def _read_data_placeholders_count(cls, opcodes):
        data_placeholders = 0

        for opcode in opcodes:
            if isinstance(opcode, tuple):
                data_placeholders += 1
            else:
                break

        return data_placeholders

    @staticmethod
    def find_end_position_of_name(script, length):
        """Finds the end position of the name data

        Given the number of opcodes in the name prefix (length), returns the
        index into the byte array of where the name prefix ends."""
        n = 0
        for _i in range(length):
            # Content of this loop is copied from Script.get_ops's loop
            op = script[n]
            n += 1

            if op <= OpCodes.OP_PUSHDATA4:
                # Raw bytes follow
                if op < OpCodes.OP_PUSHDATA1:
                    dlen = op
                elif op == OpCodes.OP_PUSHDATA1:
                    dlen = script[n]
                    n += 1
                elif op == OpCodes.OP_PUSHDATA2:
                    (dlen,) = struct.unpack("<H", script[n : n + 2])
                    n += 2
                else:
                    (dlen,) = struct.unpack("<I", script[n : n + 4])
                    n += 4
                if n + dlen > len(script):
                    raise IndexError
                n += dlen

        return n
    ```

### 1. **属性定义**

- **`DATA_PUSH_MULTIPLE = -2`**:
  - 定义特殊的占位符，用于表示脚本中多个忽略的数据推送操作。

---

### 2. **方法详解**

#### **`interpret_name_prefix(cls, script, possible_ops)`**
- **功能**:
  - 检查并解析脚本中是否包含名称前缀。
  - 如果存在名称前缀，将其字段解析为字典，并返回解析后的名称字段和地址脚本。

- **参数说明**:
  - `script`:
    - 要解析的脚本数据。
  - `possible_ops`:
    - 描述可能的名称前缀结构的数组列表。

- **实现逻辑**:
  1. 调用 `Script.get_ops` 获取脚本的操作码序列。
  2. 遍历 `possible_ops`，尝试匹配脚本的结构：
     - 支持命名占位符（如字符串）和忽略占位符（如 `-1` 和 `-2`）。
     - 使用 `_add_data_placeholders_to_template` 和 `_read_data_placeholders_count` 动态处理 `DATA_PUSH_MULTIPLE` 类型。
  3. 如果匹配成功：
     - 返回名称字段字典和分离后的地址脚本。
  4. 如果匹配失败，返回 `None` 和原始脚本。

#### **`_add_data_placeholders_to_template(cls, opcodes, template)`**
- **功能**:
  - 处理脚本中 `DATA_PUSH_MULTIPLE` 类型的占位符，动态增加模板中的占位符和 `DROP` 操作。
- **实现**:
  - 读取占位符的数量。
  - 添加适当数量的 `-1`（占位符）以及对应数量的 `OP_DROP` 和 `OP_2DROP` 操作。

#### **`_read_data_placeholders_count(cls, opcodes)`**
- **功能**:
  - 读取脚本中连续数据推送操作的数量。
- **实现**:
  - 遍历脚本操作码，统计所有数据推送操作的数量，直到遇到非数据推送操作。

#### **`find_end_position_of_name(script, length)`**
- **功能**:
  - 确定脚本中名称前缀部分的结束位置。
- **实现**:
  - 遍历脚本的前 `length` 个操作码。
  - 根据操作码的类型计算数据长度，并更新索引位置。

---
## 特性分析

### 1. **动态脚本解析**
- 支持复杂脚本结构：
  - 通过 `possible_ops` 定义脚本模板，解析支持动态或多数据推送操作的脚本结构。
  - 兼容多种区块链脚本，具有较高的灵活性。

### 2. **名称分离**
- 提供了将名称字段和地址脚本分离的工具方法：
  - 在不修改脚本原有结构的情况下，分离出名称字段，便于后续处理。

### 3. **命名占位符**
- 支持命名占位符的功能，使得脚本解析结果更具语义化：
  - 通过 `possible_ops` 传入字符串占位符，最终返回的字典可直接反映字段的含义。

---

## `NameIndexMixin` 是一个扩展自 `NameMixin` 的 Mixin 类

    ```
    class NameIndexMixin(NameMixin):
    """Shared definitions for coins that have a name index

    This class defines common functions and logic for coins that have
    a name index in addition to the index by address / script."""

    BLOCK_PROCESSOR = block_proc.NameIndexBlockProcessor
    SESSIONCLS = NameIndexElectrumX
    NAME_EXPIRATION = None

    @classmethod
    def build_name_index_script(cls, name):
        """Returns the script by which names are indexed"""

        from electrumx.lib.script import Script

        res = bytearray()
        res.append(cls.OP_NAME_UPDATE)
        res += Script.push_data(name)
        res += Script.push_data(b"")
        res.append(OpCodes.OP_2DROP)
        res.append(OpCodes.OP_DROP)
        res.append(OpCodes.OP_RETURN)

        return bytes(res)

    @classmethod
    def split_name_script(cls, script):
        named_values, address_script = cls.interpret_name_prefix(script, cls.NAME_OPERATIONS)
        if named_values is None or "name" not in named_values:
            return None, address_script

        name_index_script = cls.build_name_index_script(named_values["name"][1])
        return name_index_script, address_script

    @classmethod
    def hashX_from_script(cls, script):
        _, address_script = cls.split_name_script(script)
        return super().hashX_from_script(address_script)

    @classmethod
    def address_from_script(cls, script):
        _, address_script = cls.split_name_script(script)
        return super().address_from_script(address_script)

    @classmethod
    def name_hashX_from_script(cls, script):
        name_index_script, _ = cls.split_name_script(script)
        if name_index_script is None:
            return None

        return super().hashX_from_script(name_index_script)
    ```


### 1. **属性定义**

- **`BLOCK_PROCESSOR = block_proc.NameIndexBlockProcessor`**:
  - 指定用于处理区块的逻辑类 `NameIndexBlockProcessor`，表明该区块链在区块处理时需要考虑名称索引的逻辑。

- **`SESSIONCLS = NameIndexElectrumX`**:
  - 定义会话类为 `NameIndexElectrumX`，可能用于与客户端通信时的名称索引支持。

- **`NAME_EXPIRATION = None`**:
  - 该属性为 `None`，但可以在子类中定义名称的过期时间（例如 DNS 或其他基于名称的应用场景）。

---

### 2. **方法详解**

#### **`build_name_index_script(cls, name)`**
- **功能**:
  - 构建用于名称索引的脚本。
- **实现**:
  1. 脚本的核心结构：
     - 使用 `OP_NAME_UPDATE` 表示名称更新操作。
     - 将 `name` 和空数据段 `b""` 推入脚本。
     - 使用 `OP_2DROP` 和 `OP_DROP` 清理堆栈。
     - 以 `OP_RETURN` 终止脚本。
  2. 通过 `Script.push_data` 将数据段格式化为正确的脚本格式。

#### **`split_name_script(cls, script)`**
- **功能**:
  - 分离脚本中的名称字段和地址脚本。
- **实现**:
  1. 调用 `interpret_name_prefix` 方法解析脚本，提取名称字段。
  2. 如果名称字段存在，构建名称索引脚本。
  3. 返回名称索引脚本和剩余的地址脚本。

#### **`hashX_from_script(cls, script)`**
- **功能**:
  - 从脚本中解析地址部分，生成其对应的 `hashX`。
- **实现**:
  1. 调用 `split_name_script` 提取地址脚本部分。
  2. 调用父类方法 `hashX_from_script` 生成 `hashX`。

#### **`address_from_script(cls, script)`**
- **功能**:
  - 从脚本中解析地址部分，并生成对应的地址。
- **实现**:
  1. 调用 `split_name_script` 提取地址脚本部分。
  2. 调用父类方法 `address_from_script` 返回地址。

#### **`name_hashX_from_script(cls, script)`**
- **功能**:
  - 从脚本中提取名称部分并生成其 `hashX`。
- **实现**:
  1. 调用 `split_name_script` 提取名称索引脚本部分。
  2. 如果名称索引脚本存在，调用父类方法 `hashX_from_script` 生成其 `hashX`。


## `NameIndexAuxPoWMixin` 是一个结合了 `NameIndexMixin` 和 `AuxPowMixin` 的 Mixin 类
    ```
    class NameIndexAuxPoWMixin(NameIndexMixin, AuxPowMixin):
    SESSIONCLS = NameIndexAuxPoWElectrumX
    ```
### 1. **属性定义**

- **`SESSIONCLS = NameIndexAuxPoWElectrumX`**:
  - 指定会话类为 `NameIndexAuxPoWElectrumX`。
  - 表明该类的设计用途是为支持名称索引和 AuxPoW 功能的区块链提供服务。

  ---

## 继承特性分析

### 1. **从 `NameIndexMixin` 继承**
- **名称索引支持**:
  - 继承了 `NameIndexMixin` 中的功能，如：
    - `build_name_index_script`: 构建名称索引脚本。
    - `split_name_script`: 分离脚本中的名称字段和地址脚本。
    - `name_hashX_from_script`: 从脚本中提取名称并生成其 `hashX`。

- **名称与地址解析分离**:
  - 支持对名称和地址的独立解析，使其能够处理包含复杂名称前缀的脚本。

### 2. **从 `AuxPowMixin` 继承**
- **AuxPoW 支持**:
  - 继承了 `AuxPowMixin` 的功能，如：
    - `header_hash`: 为区块头计算哈希，支持动态区块头大小。
    - `block_header`: 返回 AuxPoW 区块头的字节表示。

- **动态区块头处理**:
  - 能够解析和处理 AuxPoW 特有的动态大小区块头，适配需要此功能的区块链。

### 3. **功能结合**
- 此类通过多重继承，将名称索引和 AuxPoW 特性结合，适用于既需要名称索引又支持 AuxPoW 的区块链。


## `PrimeChainPowMixin` 是一个 Mixin 类
    ```

class PrimeChainPowMixin:
    STATIC_BLOCK_HEADERS = False
    DESERIALIZER = lib_tx.DeserializerPrimecoin

    @classmethod
    def block_header(cls, block, height):
        """Return the block header bytes"""
        deserializer = cls.DESERIALIZER(block)
        return deserializer.read_header(cls.BASIC_HEADER_SIZE)
    ```

### 1. **属性定义**

- **`STATIC_BLOCK_HEADERS = False`**:
  - 表明区块头的大小是动态的，适配质数链 PoW 特有的区块头结构。

- **`DESERIALIZER = lib_tx.DeserializerPrimecoin`**:
  - 指定使用 `DeserializerPrimecoin` 来解析区块数据，表明此类主要用于类似 Primecoin 的区块链。

---
### 2. **方法详解**

#### **`block_header(cls, block, height)`**
- **功能**:
  - 返回区块头的字节表示。
- **实现**:
  1. 使用 `DESERIALIZER` 解析给定的区块数据。
  2. 调用 `read_header` 方法提取区块的头部字节。
  3. 返回解析出的区块头。

---
## `Verge` 类继承自 `Coin` 类，定义了与 Verge (XVG) 区块链相关的具体参数和方法。

    ```

class Verge(Coin):
    NAME = "Verge"
    SHORTNAME = "XVG"
    NET = "mainnet"
    XPUB_VERBYTES = bytes.fromhex("022d2533")
    XPRV_VERBYTES = bytes.fromhex("0221312b")
    P2PKH_VERBYTE = bytes.fromhex("30")
    P2SH_VERBYTES = [bytes.fromhex("33")]
    WIF_BYTE = bytes.fromhex("9E")
    GENESIS_HASH = "00000fc63692467faeb20cdb3b53200d" "c601d75bdfa1001463304cc790d77278"
    RPC_PORT = 20102
    TX_COUNT = 500000
    TX_COUNT_HEIGHT = 3082138
    TX_PER_BLOCK = 1
    DESERIALIZER = lib_tx.DeserializerVerge

    @classmethod
    def header_hash(cls, header):
        """Given a header return the hash."""
        import scrypt

        return scrypt.hash(header, header, 1024, 1, 1, 32)
    ```

### 1. **属性定义**

- **链标识**
  - `NAME = "Verge"`:
    - 定义区块链名称为 Verge。
  - `SHORTNAME = "XVG"`:
    - 简称为 XVG，通常用于标识代币。

- **网络类型**
  - `NET = "mainnet"`:
    - 指定网络为主网（`mainnet`）。

- **扩展密钥支持**
  - `XPUB_VERBYTES = bytes.fromhex("022d2533")`:
    - Verge 扩展公钥（XPUB）的版本字节。
  - `XPRV_VERBYTES = bytes.fromhex("0221312b")`:
    - Verge 扩展私钥（XPRV）的版本字节。

- **地址支持**
  - `P2PKH_VERBYTE = bytes.fromhex("30")`:
    - 定义 Pay-to-PubKey-Hash (P2PKH) 地址的版本字节。
  - `P2SH_VERBYTES = [bytes.fromhex("33")]`:
    - 定义 Pay-to-Script-Hash (P2SH) 地址的版本字节。

- **私钥导入格式**
  - `WIF_BYTE = bytes.fromhex("9E")`:
    - 定义 Wallet Import Format (WIF) 私钥的版本字节。

- **创世区块**
  - `GENESIS_HASH = "00000fc63692467faeb20cdb3b53200d" "c601d75bdfa1001463304cc790d77278"`:
    - Verge 创世区块的哈希值，用于链起点的验证。

- **网络通信**
  - `RPC_PORT = 20102`:
    - Verge 节点的远程过程调用（RPC）默认端口号。

- **交易统计**
  - `TX_COUNT = 500000`:
    - 当前网络的总交易数量。
  - `TX_COUNT_HEIGHT = 3082138`:
    - 交易计数对应的区块高度。
  - `TX_PER_BLOCK = 1`:
    - 每个区块的平均交易数量。

- **反序列化**
  - `DESERIALIZER = lib_tx.DeserializerVerge`:
    - 使用 `DeserializerVerge` 解析 Verge 特有的区块和交易数据。

---

### 2. **方法详解**

#### **`header_hash(cls, header)`**
- **功能**:
  - 返回给定区块头的哈希值。
- **实现**:
  - 使用 Scrypt 算法计算哈希：
    - 参数：
      - `N = 1024`: CPU/内存成本因子。
      - `r = 1`: 块大小因子。
      - `p = 1`: 并行化因子。
      - `dklen = 32`: 派生密钥长度。
    - 输入为区块头本身。
  - 这是 Verge 使用 Scrypt 算法的典型实现。

---

## `HOdlcoin` 类继承自 `Coin` 类，定义了 HOdlcoin 区块链的具体参数和功能。

    ```
class HOdlcoin(Coin):
    NAME = "HOdlcoin"
    SHORTNAME = "HODLC"
    NET = "mainnet"
    BASIC_HEADER_SIZE = 88
    P2PKH_VERBYTE = bytes.fromhex("28")
    WIF_BYTE = bytes.fromhex("a8")
    GENESIS_HASH = "008872e5582924544e5c707ee4b839bb" "82c28a9e94e917c94b40538d5658c04b"
    DESERIALIZER = lib_tx.DeserializerSegWit
    TX_COUNT = 258858
    TX_COUNT_HEIGHT = 382138
    TX_PER_BLOCK = 5

    ```

### 1. **属性定义**

- **链标识**
  - `NAME = "HOdlcoin"`:
    - 区块链名称为 HOdlcoin。
  - `SHORTNAME = "HODLC"`:
    - 简称为 HODLC，用于标识代币。

- **网络类型**
  - `NET = "mainnet"`:
    - 定义当前网络为主网（`mainnet`）。

- **区块头**
  - `BASIC_HEADER_SIZE = 88`:
    - 定义 HOdlcoin 区块头的基础大小为 88 字节，表明其区块头结构可能与比特币略有不同。

- **地址支持**
  - `P2PKH_VERBYTE = bytes.fromhex("28")`:
    - Pay-to-PubKey-Hash (P2PKH) 地址的版本字节。
  - `WIF_BYTE = bytes.fromhex("a8")`:
    - Wallet Import Format (WIF) 私钥的版本字节。

- **创世区块**
  - `GENESIS_HASH = "008872e5582924544e5c707ee4b839bb82c28a9e94e917c94b40538d5658c04b"`:
    - HOdlcoin 的创世区块哈希值，用于链起点的验证。

- **交易支持**
  - `DESERIALIZER = lib_tx.DeserializerSegWit`:
    - 使用 SegWit（隔离见证）交易反序列化器，表明 HOdlcoin 支持 SegWit。

- **交易统计**
  - `TX_COUNT = 258858`:
    - 当前网络的总交易数量。
  - `TX_COUNT_HEIGHT = 382138`:
    - 交易计数对应的区块高度。
  - `TX_PER_BLOCK = 5`:
    - 每个区块的平均交易数量。
