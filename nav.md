```
atomical-electrumx/
├── lib/
│   ├── atomicals_blueprint_builder.py  
│   ├── coins.py                        
│   ├── enum.py                         
│   ├── env_base.py                     
│   ├── hash.py                         
│   ├── __init__.py                     
│   ├── merkle.py                       
│   ├── peer.py                        
│   ├── psbt.py                         
│   ├── script2addr.py                  
│   ├── script.py                      
│   ├── segwit_addr.py                 
│   ├── server_base.py                 
│   ├── text.py                         
│   ├── tx_axe.py                       
│   ├── tx_dash.py                      
│   ├── tx.py                           
│   ├── util_atomicals.py              
│   └── util.py                         
│
├── server/
│   ├── block_processor.py              
│   ├── controller.py                   
│   ├── daemon.py                       
│   ├── db.py                           
│   ├── env.py                          
│   ├── history.py                      
│   ├── http_middleware.py              
│   ├── __init__.py                     
│   ├── mempool.py                      
│   ├── peers.py                        
│   └── storage.py                      
│
├── server/session/
    ├── electrumx_session.py            
    ├── http_session.py                 
    ├── __init__.py                     
    ├── rpc_session.py                  
    ├── session_base.py                
    ├── session_manager.py              
    ├── shared_session.py               
    └── util.py                        

```
--- 

## 1. 核心协议实现 (Core Protocol)

lib/
├── [atomicals_blueprint_builder.py](./CoreProtocol/atomicals_blueprint_builder.md)  # Atomicals协议核心实现，负责构建和验证Atomicals交易
├── [coins.py](./CoreProtocol/coins.md)                        # 加密货币相关实现，定义各种币种的参数和规则
├── [tx.py](./CoreProtocol/tx.md)                          # 比特币交易的核心处理逻辑，包括交易解析和验证
├── [tx_dash.py](./CoreProtocol/tx_dash.md)                     # DASH特定的交易处理逻辑
├── [tx_axe.py](./CoreProtocol/tx_axe.md)                      # AXE特定的交易处理逻辑
├── [script.py](./CoreProtocol/script.md)                      # 比特币脚本引擎，处理锁定和解锁脚本
└── [psbt.py](./CoreProtocol/psbt.md)                        # 部分签名比特币交易(PSBT)的实现，支持多重签名

## 2. 区块链数据处理 (Blockchain Processing)
server/
├── block_processor.py             # 区块处理的核心组件，负责区块的解析、验证和链接
├── mempool.py                     # 内存池管理，处理未确认交易
├── history.py                     # 交易历史记录的处理和存储
└── storage.py                     # 区块数据持久化存储管理

## 3. 网络通信 (Networking)
server/session/
├── session_manager.py             # 管理所有类型会话的核心组件
├── electrumx_session.py           # 处理Electrum协议的客户端连接
├── http_session.py                # 处理HTTP API请求
├── rpc_session.py                 # 处理RPC调用
└── shared_session.py              # 会话间共享数据的实现
## 4. 数据存储 (Data Storage)
server/
├── db.py                          # 数据库操作的核心实现，管理区块链数据的存储和检索
└── storage.py                     # 存储系统的抽象层，处理数据的持久化
## 5. 工具类 (Utilities)
lib/
├── hash.py                        # 各种哈希算法的实现，包括SHA256、RIPEMD160等
├── merkle.py                      # 默克尔树的实现，用于区块头验证
├── text.py                        # 文本处理工具，处理编码转换等
├── util_atomicals.py              # Atomicals专用工具函数，处理Atomicals特定的数据结构
└── util.py                        # 通用工具函数集合
## 6. 地址处理 (Address Handling)
lib/
├── script2addr.py                 # 将脚本转换为地址的工具
└── segwit_addr.py                 # 隔离见证地址的编码和解码实现
## 7. 服务器管理 (Server Management)
server/
├── controller.py                  # 服务器主控制器，协调各个组件的工作
├── daemon.py                      # 守护进程管理，确保服务持续运行
├── peers.py                       # P2P网络中的节点管理
└── http_middleware.py             # HTTP请求的中间件处理
## 8. 环境配置 (Environment)
lib/
├── env_base.py                    # 基础环境配置类
└── server_base.py                 # 服务器基础配置类
server/
└── env.py                         # 服务器环境配置实现
## 9. 基础设施 (Infrastructure)
lib/
├── enum.py                        # 系统中使用的枚举类型定义
└── __init__.py                    # 包初始化文件
server/
├── __init__.py                    # 服务器包初始化文件
└── session/__init__.py            # 会话管理包初始化文件