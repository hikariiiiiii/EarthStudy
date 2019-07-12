# 开发



### **区块链结构**

魔法数 Magic no

区块大小 Blocksize

​		区块头 Blockheader：

​		版本号 Version

​		前区块Hash值 hashPrevBlock

​		Merkle根节点Hash值 hashMerkleRoot

​		时间戳 Time

​		当前目标Hash值 Bits

​		随机数 Nonce

交易数量 Transaction counter

交易 Transactions

​       	版本号

​       	输入数量

​       	输入列表

​       	输出数量

​       	输出列表

​       	锁定时间lock_time



### **调用比特币API的方式**

1. 通过bitcoind-qt
2. 通过bitcoind-cli
3. 通过curl
4. 通过语言库调用