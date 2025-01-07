# Atomical ElectrumX  文档
---
![原子协议索引器代码结构图](./img/原子协议索引器代码结构图.png)

[项目结构](./nav.md)
# Atomical-ElectrumX 概述

## 简介
Atomical-ElectrumX 是一个基于 ElectrumX 服务器的扩展实现，主要增加了对 Atomicals 协议的支持。它作为比特币网络的轻节点服务器，为客户端提供高效的查询服务。

## 核心组件

### 区块处理系统
- **Block Processor（block_processor.py）**
 - 处理新区块的验证和链接
 - 维护 UTXO 集合
 - 处理 Atomicals 相关的交易和状态更新

### 数据存储层
- **Storage（storage.py）**
 - 管理区块数据、交易历史和 UTXO 集的持久化存储
 - 提供高效的数据检索机制
 - 支持 Atomicals 协议特有的数据结构存储

### 会话管理
- **Session Manager（session_manager.py）**
 - 管理客户端连接
 - 处理不同类型的会话（ElectrumX、HTTP、RPC）
 - 实现会话状态同步

### 内存池管理
- **Mempool（mempool.py）**
 - 跟踪未确认交易
 - 维护交易优先级队列
 - 处理 Atomicals 相关的未确认状态

## 主要特征

### Atomicals 协议支持
- 支持 Atomicals 协议的交易验证
- 提供 Atomicals 状态查询接口
- 实现 Atomicals 相关的索引构建

### 高性能设计
- 异步 I/O 操作
- 高效的内存管理
- 优化的数据库查询

### 多协议支持
- **支持多种连接协议：**
 - ElectrumX 协议
 - HTTP/REST API
 - JSON-RPC

### 可扩展性
- 模块化设计
- 插件系统支持
- 自定义索引能力

## 技术特点

### 安全性
- 实现交易验证
- 支持 SPV 验证
- 安全的会话管理

### 可靠性
- 数据一致性保证
- 故障恢复机制
- 状态同步机制

### 可维护性
- 清晰的代码结构
- 完善的日志系统
- 模块化的设计
---
## Atomical-ElectrumX 核心依赖包详解

```
aiorpcX[ws]>=0.23.0,<0.24
plyvel>=1.5.0,<1.6
cbor2>=5.6.0,<5.7
aiohttp>=3.9.0,<3.10

```

### aiorpcX[ws] (>=0.23.0,<0.24)
- **核心功能**: 异步RPC框架
- **主要用途**:
 - 处理服务器与客户端之间的通信
 - 提供 WebSocket 连接支持
 - 实现高并发的请求处理
- **关键特性**:
 - 异步I/O操作
 - 高效的消息序列化
 - 内置的 WebSocket 支持

### plyvel (>=1.5.0,<1.6)
- **核心功能**: LevelDB Python接口
- **主要用途**:
 - 区块数据永久存储
 - UTXO集管理
 - 交易历史索引
- **关键特性**:
 - 高性能键值存储
 - 原子性操作支持
 - 数据压缩能力

### cbor2 (>=5.6.0,<5.7)
- **核心功能**: CBOR编码实现
- **主要用途**:
 - Atomicals协议数据序列化
 - 二进制数据编码和解码
 - 网络数据传输
- **关键特性**:
 - 高效的数据编码
 - 类型安全
 - 兼容JSON数据结构

### aiohttp (>=3.9.0,<3.10)
- **核心功能**: 异步HTTP框架
- **主要用途**:
 - REST API实现
 - HTTP客户端功能
 - Web服务器功能
- **关键特性**:
 - 异步HTTP处理
 - WebSocket支持
 - 中间件系统


 ## 搭建流程

 ### 开发环境
 python>=3.10
 ```
 1. 安装依赖包
    pip install -r requirements.txt
 2. 配置数据库
    cp .env.example .env
 3. 启动服务
    ./run.py
 ```
 ### env配置
```
DAEMON_URL=http://user:pass@localhost:8332/ # 比特币fullnode
COIN=Bitcoin
# NET=testnet
REQUEST_TIMEOUT=90
DB_DIRECTORY=/home/ubuntu/electrumxdb # 数据库目录
DB_ENGINE=leveldb
SERVICES=tcp://0.0.0.0:50010,ws://:50020,rpc://:8000
HOST=""
ALLOW_ROOT=true
CACHE_MB=400
MAX_SEND=3000000
COST_SOFT_LIMIT=1000
COST_HARD_LIMIT=10000
REQUEST_SLEEP=100
INITIAL_CONCURRENT=10
MAX_TOKENS=10000
RATE_LIMIT_WINDOW_SECONDS=60
RATE_LIMIT_DELAY_AFTER=5
RATE_LIMIT_DELAY_MS=300
ENABLE_RATE_LIMIT=true
```
