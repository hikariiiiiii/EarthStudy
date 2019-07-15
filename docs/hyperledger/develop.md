# 开发

## 环境准备

**Docker环境**（Docker、Docker Compose）

**GO语言环境**

**Fabric组件Docker镜像**

​       fabric-peer、fabric-orderer、fabric-tools、fabric-baseos

``` bash
# docker pull hyperledger/fabric-peer:latest
# docker pull hyperledger/fabric-orderer:latest
# docker pull hyperledger/fabric-tools:latest
# docker pull hyperledger/fabric-ccenv:latest
# docker pull hyperledger/fabric-baseos:latest
# docker pull hyperledger/fabric-kafka:latest
# docker pull hyperledger/fabric-zookeeper:latest
# docker pull hyperledger/fabric-couchdb:latest
# docker pull hyperledger/fabric-ca:latest
```

**Fabric源码库** -> release-1.0（go get）

**cryptogen、configtxgen工具编译**（go install）

**官方示例 fabric-samples**（go get）

​	first-network/  ->

​	./byfn.sh -m generate -c hikari          //生成相应证书和创世区块

​	./byfn.sh -m up -c hikari                   //通过docker compose启动网络



## **网络启动**

1. 配置crypto-config.yaml设置组织、节点、成员结构

2. 使用crytogen生成公私钥和证书文件

3. 配置configtx.yaml设置联盟、系统链、组织的结构、组织和锚节点的访问地址和MSP地址

4. 使用configtxgen生成系统链创世区块、生成通道创世交易、生成锚节点交易（可选）

5. 创建docker-compose.yaml文件，配置order节点容器、各peer节点容器、cli容器

6. 启动docker-compose

   ```bash
   docker-compose up -d           //关闭是 docker-compose down
   
   docker exec -it cli bash       //进入客户端容器(cli)控制台
   
   peer                           /与peer节点交互的工具
   ```

7. 利用peer工具：

   - 利用通道创世交易创建通道

     ```bash
     peer channel create
     ```

   - 将当前cli连接的peer节点加入通道

     ```bash
     peer channel join
     ```

   - 设置主节点

     ```bash
     peer channel update
     ```

   - 安装链码

     ```bash
     peer chaincode install
     ```

   - 实例化链码

     ```bash
     peer chaincode instantiate
     ```



## 指令笔记


### 设置工作路径
``` bash
export FABRIC_CFG_PATH=$GOPATH/src/github.com/hyperledger/fabric/imocc/deploy
```

### 环境清理
``` bash
rm -fr config/*
rm -fr crypto-config/*
```

### 生成证书文件
``` bash
cryptogen generate --config=./crypto-config.yaml
```

### 生成创世区块
``` bash
configtxgen -profile OneOrgOrdererGenesis -outputBlock ./config/genesis.block
```

### 生成通道的创世交易
``` bash
configtxgen -profile TwoOrgChannel -outputCreateChannelTx ./config/mychannel.tx -channelID mychannel
configtxgen -profile TwoOrgChannel -outputCreateChannelTx ./config/assetschannel.tx -channelID assetschannel
```

### 生成组织关于通道的锚节点（主节点）交易
``` bash
configtxgen -profile TwoOrgChannel -outputAnchorPeersUpdate ./config/Org0MSPanchors.tx -channelID mychannel -asOrg Org0MSP
configtxgen -profile TwoOrgChannel -outputAnchorPeersUpdate ./config/Org1MSPanchors.tx -channelID mychannel -asOrg Org1MSP
```

### 启动容器
``` bash
docker-compose up -d
```

### 进入客户端容器cli控制台
``` bash
docker exec -it cli bash
```

### 创建通道
``` bash
peer channel create -o orderer.imocc.com:7050 -c mychannel -f /etc/hyperledger/config/mychannel.tx
peer channel create -o orderer.imocc.com:7050 -c assetschannel -f /etc/hyperledger/config/assetschannel.tx
```

### 加入通道
``` bash
peer channel join -b mychannel.block
peer channel join -b assetschannel.block
```

### 设置主节点
``` bash
peer channel update -o orderer.imocc.com:7050 -c mychannel -f /etc/hyperledger/config/Org1MSPanchors.tx
```

### 链码安装
``` bash
peer chaincode install -n badexample -v 1.0.0 -l golang -p github.com/chaincode/badexample
peer chaincode install -n assets -v 1.0.0 -l golang -p github.com/chaincode/assetsExchange
```

### 链码实例化
``` bash
peer chaincode instantiate -o orderer.imocc.com:7050 -C mychannel -n badexample -l golang -v 1.0.0 -c '{"Args":["init"]}'
peer chaincode instantiate -o orderer.imocc.com:7050 -C assetschannel -n assets -l golang -v 1.0.0 -c '{"Args":["init"]}'
```

### 链码交互
``` bash
peer chaincode invoke -C assetschannel -n assets -c '{"Args":["userRegister", "user1", "user1"]}'
peer chaincode invoke -C assetschannel -n assets -c '{"Args":["assetEnroll", "asset1", "asset1", "metadata", "user1"]}'
peer chaincode invoke -C assetschannel -n assets -c '{"Args":["userRegister", "user2", "user2"]}'
peer chaincode invoke -C assetschannel -n assets -c '{"Args":["assetExchange", "user1", "asset1", "user2"]}'
peer chaincode invoke -C assetschannel -n assets -c '{"Args":["userDestroy", "user1"]}'
```

### 链码升级
``` bash
peer chaincode install -n assets -v 1.0.1 -l golang -p github.com/chaincode/assetsExchange
peer chaincode upgrade -C assetschannel -n assets -v 1.0.1 -c '{"Args":[""]}'
```

### 链码查询
``` bash
peer chaincode query -C mychannel -n badexample -c '{"Args":[]}'
peer chaincode query -C assetschannel -n assets -c '{"Args":["queryUser", "user1"]}'
peer chaincode query -C assetschannel -n assets -c '{"Args":["queryAsset", "asset1"]}'
peer chaincode query -C assetschannel -n assets -c '{"Args":["queryUser", "user2"]}'
peer chaincode query -C assetschannel -n assets -c '{"Args":["queryAssetHistory", "asset1"]}'
peer chaincode query -C assetschannel -n assets -c '{"Args":["queryAssetHistory", "asset1", "all"]}'
```

### 命令行模式的背书策略

在链码实例化peer chaincode instantiate时，添加-P 参数值：

```
EXPR(E[,E...])
EXPR = OR AND
E = EXPR(E[,E...])
MSP.ROLE
MSP 组织名 org0MSP org1MSP
ROLE admin member
```

示例：

```bash
-P “OR('org0MSP.member','org1MSP.admin')”
```

### 打印日志

```bash
docker ps -a        //查看容器名

docker logs 容器名
```

### 链码调试

dev模式省去了编译链码镜像的过程

在docker-compose.yaml中的peer.base里添加dev模式配置：

- command:  peer node start --peer-chaincodedev=true
- environment:  - CORE_CHAINCODE_MODE=dev

```bash
CORE_CHAINCODE_ID_NAME=assets:1.0.1 CORE_PEER_ADDRESS=0.0.0.0:27051 CORE_CHAINCODE_LOGGING_LEVEL=DEBUG go run -tags=nopkcs11 assetsExchange.go
```

## 外部服务剖析

### 如何提供服务？
决定于应用场景 终端用户
* 智能硬件 socket/tcp 太阳能发电
* 游戏、电商、社交 web/app http
* 企业内部 rpc(grpc)

### 如何选择SDK?
* nodejs  4星
* java       3星
* golang  1星

构造交易、发送交易、数据查询

### SDK的模块

#### 区块链管理
* 通道创建&加入
* 链码的安装、实例化、升级等
* admin | 云服务提供商

#### 数据查询
* 区块
* 交易
* 区块浏览器 ethscan eospark block-explorer

#### 区块链交互
* 发起交易 invoke query

#### 事件监听
* 业务事件 SendEvent
* 系统事件 block/trancastion

