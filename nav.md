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
 [atomicals_blueprint_builder.py](./CoreProtocol/atomicals_blueprint_builder.md)  # Atomicals协议核心实现，负责构建和验证Atomicals交易<br>
 [coins.py](./CoreProtocol/coins.md)                        # 加密货币相关实现，定义各种币种的参数和规则<br>
 [tx.py](./CoreProtocol/tx.md)                          # 比特币交易的核心处理逻辑，包括交易解析和验证<br>
 [tx_dash.py](./CoreProtocol/tx_dash.md)                     # DASH特定的交易处理逻辑<br>
 [tx_axe.py](./CoreProtocol/tx_axe.md)                      # AXE特定的交易处理逻辑<br>
 [script.py](./CoreProtocol/script.md)                      # 比特币脚本引擎，处理锁定和解锁脚本<br>
 [psbt.py](./CoreProtocol/psbt.md)                        # 部分签名比特币交易(PSBT)的实现，支持多重签名<br>

## 2. 区块链数据处理 (Blockchain Processing)
server/
 [block_processor.py](./BlockchainProcessing/block_processor.md)             # 区块处理的核心组件，负责区块的解析、验证和链接<br>
 [mempool.py](./BlockchainProcessing/mempool.md)                     # 内存池管理，处理未确认交易<br>
 [history.py](./BlockchainProcessing/history.md)                     # 交易历史记录的处理和存储<br>
 [storage.py](./BlockchainProcessing/storage.md)                     # 区块数据持久化存储管理<br>

## 3. 网络通信 (Networking)
server/session/
 [session_manager.py](./Networking/session_manager.md)             # 管理所有类型会话的核心组件<br>
 [electrumx_session.py](./Networking/electrumx_session.md)           # 处理Electrum协议的客户端连接<br>
 [http_session.py](./Networking/http_session.md)                # 处理HTTP API请求<br>
 [rpc_session.py](./Networking/rpc_session.md)                 # 处理RPC调用<br>
 [shared_session.py](./Networking/shared_session.md)              # 会话间共享数据的实现<br>

## 4. 数据存储 (Data Storage)
server/
 [db.py](./DataStorage/db.md)                          # 数据库操作的核心实现，管理区块链数据的存储和检索<br>
 [storage.py](./DataStorage/storage.md)                     # 存储系统的抽象层，处理数据的持久化<br>

## 5. 工具类 (Utilities)
lib/
 [hash.py](./Utilities/hash.md)                        # 各种哈希算法的实现，包括SHA256、RIPEMD160等<br>
 [merkle.py](./Utilities/merkle.md)                      # 默克尔树的实现，用于区块头验证<br>
 [text.py](./Utilities/text.md)                        # 文本处理工具，处理编码转换等<br>
 [util_atomicals.py](./Utilities/util_atomicals.md)              # Atomicals专用工具函数，处理Atomicals特定的数据结构<br>
 [util.py](./Utilities/util.md)                        # 通用工具函数集合<br>

## 6. 地址处理 (Address Handling)
lib/
 [script2addr.py](./AddressHandling/script2addr.md)                 # 将脚本转换为地址的工具<br>
 [segwit_addr.py](./AddressHandling/segwit_addr.md)                 # 隔离见证地址的编码和解码实现<br>

## 7. 服务器管理 (Server Management)
server/
 [controller.py](./ServerManagement/controller.md)                  # 服务器主控制器，协调各个组件的工作<br>
 [daemon.py](./ServerManagement/daemon.md)                      # 守护进程管理，确保服务持续运行<br>
 [peers.py](./ServerManagement/peers.md)                       # P2P网络中的节点管理<br>
 [http_middleware.py](./ServerManagement/http_middleware.md)             # HTTP请求的中间件处理<br>
## 8. 环境配置 (Environment)
lib/
 [env_base.py](./Environment/env_base.md)                    # 基础环境配置类<br>
 [server_base.py](./Environment/server_base.md)                 # 服务器基础配置类<br>
server/
 [env.py](./Environment/env.md)                         # 服务器环境配置实现<br>
## 9. 基础设施 (Infrastructure)
lib/
 [enum.py](./Infrastructure/enum.md)                        # 系统中使用的枚举类型定义<br>
 [`__init__.py](./Infrastructure/__init__.md)                    # 包初始化文件<br>
server/
 [__init__.py](./Infrastructure/server/__init__.md)                    # 服务器包初始化文件<br>
 session/__init__.py            # 会话管理包初始化文件<br>
