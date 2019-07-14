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

1. 配置crypto-config.yaml，并使用crytogen生成公私钥和证书文件

2. 配置configtx.yaml，并使用configtxgen生成系统链创世区块、生成通道创世交易、生成锚节点交易（可选）

3. 创建docker-compose.yaml文件，配置order容器、各peer容器、cli容器

4. 启动docker-compose

   ```bash
   docker exec -it cli bash       //进入客户端容器(cli)控制台
   
   peer                                   //与peer节点交互的工具
   ```

5. 利用peer工具：

   - 创建通道

     ```bash
     peer channel create
     ```

   - 将节点加入通道

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

