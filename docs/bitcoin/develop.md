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



**bitcoin core：**比特币核心（前身是bitcoin-QT，比特币钱包），带有图形界面

**bitcoind：**比特币核心简洁版、命令行版

### **调用比特币API的方式**

1. 通过bitcoin core
2. 通过bitcoind-cli
3. 通过curl
4. 通过语言库调用



# **RPC方法**

## ListUnspent

The listunspent RPC returns an array of unspent transaction outputs belonging to this wallet. Note: as of Bitcoin Core 0.10.0, outputs affecting watch-only addresses will be returned; see the spendable field in the results described below.

#### 参数

- MinimumConfirmations：交易最少确认数，默认值：1
- MaximumConfirmations：交易最多确认数，默认值：9999999
- Addresses：交易输出中应当包含的地址数组

#### 返回值

`listunspent`调用返回UTXO描述数组，成员结构如下：

- txid：交易id
- vout：输出序号
- address：地址
- account：关联账户
- scriptPubKey：公钥脚本
- redeemScript：赎回脚本
- amount：金额
- confirmations：确认数
- spendable：是否可消费，当钱包中包含对应的私钥时，该值为true
- solvable：是否可用，钱包中包含私钥时该值被忽略

#### 示例代码

下面的命令返回节点钱包中包含6~9999999个确认并且支付到指定地址的UTXO：

```
~$ bitcoin-cli -testnet listunspent 6 99999999 '''
  [
    "mgnucj8nYqdrPFh2JfZSB1NmUThUGnmsqe"
  ]
'''
```

输出结果如下：

```
[
    {
        "txid" : "d54994ece1d11b19785c7248868696250ab195605b469632b7bd68130e880c9a",
        "vout" : 1,
        "address" : "mgnucj8nYqdrPFh2JfZSB1NmUThUGnmsqe",
        "account" : "test label",
        "scriptPubKey" : "76a9140dfc8bafc8419853b34d5e072ad37d1a5159f58488ac",
        "amount" : 0.00010000,
        "confirmations" : 6210,
        "spendable" : true,
        "solvable" : true
    }
]
```

## CreateRawTransaction

`createrawtransaction`调用创建一个未签名的序列化交易，该交易可以将一个UTXO 转让给指定的P2PKH地址或P2SH地址。该交易不会存储在钱包里，也不会发送到网络 中。

#### 参数

- Inputs：交易输入数组，每个成员对象的结构如下：
  - txid：UTXO的交易id
    - vout：UTXO的输出序号
    - Sequence：序列号，可选
- Outputs：交易输出对象，键为地址，值为金额
- Locktime：声明该交易生效的最早时间，可选

#### 返回值

`createrawtransaction`调用返回生成的未签名交易的序列化字符串。

#### 示例代码

下面的命令将返回一个未签名的裸交易字符串：

```
~$ bitcoin-cli -testnet createrawtransaction '''
  [
    {
      "txid": "1eb590cd06127f78bf38ab4140c4cdce56ad9eb8886999eb898ddf4d3b28a91d",
      "vout" : 0
    }
  ]''' '{ "mgnucj8nYqdrPFh2JfZSB1NmUThUGnmsqe": 0.13 }'
```

输出结果如下：

```
01000000011da9283b4ddf8d89eb996988b89ead56cecdc44041ab38bf787f12\
06cd90b51e0000000000ffffffff01405dc600000000001976a9140dfc8bafc8\
419853b34d5e072ad37d1a5159f58488ac00000000
```

## SignRawTransaction

`signrawtransaction`调用使用钱包中的私钥对指定的序列化交易字符串进行签名。

#### 参数

- Transaction：要签名的交易字符串
- Dependencies：依赖的UTXO详情数组，成员结构如下：
  - txid：UTXO所在的交易ID
    - vout：UTXO在输出中的序号
    - scriptPubKey：输出的公钥脚本
    - redeemScript：如果公钥脚本对应于脚本哈希，则本字段指向完整的赎回脚本
- PrivateKeys：用于签名的私钥数组
- SigHashType：签名哈希类型，可以是：
  - ALL
  - NONE
  - SINGLE
  - ALL | ANYONECANPAY
  - NONE | ANYONECANPAY
  - SINGLE | ANYONECANPAY

#### 返回值

`signrawtransaction`调用返回签名后的交易信息，结构如下：

- hex： 签名后的交易序列字符串，16进制表示
- complete：交易是否具备完整签名，false表示还需要更多的签名

#### 示例代码

下面命令使用指定的私钥进行裸交易签名：

```
~$ bitcoin-cli -testnet signrawtransaction 01000000011da9283b4ddf8d\
89eb996988b89ead56cecdc44041ab38bf787f1206cd90b51e0000000000ffff\
ffff01405dc600000000001976a9140dfc8bafc8419853b34d5e072ad37d1a51\
59f58488ac00000000
```

输出结果如下：

```
{
    "hex" : "01000000011da9283b4ddf8d89eb996988b89ead56cecdc44041ab38bf787f1206cd90b51e000000006a47304402200ebea9f630f3ee35fa467ffc234592c79538ecd6eb1c9199eb23c4a16a0485a20220172ecaf6975902584987d295b8dddf8f46ec32ca19122510e22405ba52d1f13201210256d16d76a49e6c8e2edc1c265d600ec1a64a45153d45c29a2fd0228c24c3a524ffffffff01405dc600000000001976a9140dfc8bafc8419853b34d5e072ad37d1a5159f58488ac00000000",
    "complete" : true
}
```

## SendRawTransaction

`sendrawtransaction`调用验证指定交易并将其广播到P2P网络中。

#### 参数

- Transaction：序列化的交易码流，16进制字符串
- AllowHighFees：是否允许高额手续费，可选，默认值：false

#### 返回值

成功时`sendrawtransaction`调用返回交易ID，否则返回错误信息。

#### 示例代码

下面的命令验证指定的序列化签名交易并将其广播至其他节点：

```
~$ bitcoin-cli -testnet sendrawtransaction 01000000011da9283b4ddf8d\
89eb996988b89ead56cecdc44041ab38bf787f1206cd90b51e000000006a4730\
4402200ebea9f630f3ee35fa467ffc234592c79538ecd6eb1c9199eb23c4a16a\
0485a20220172ecaf6975902584987d295b8dddf8f46ec32ca19122510e22405\
ba52d1f13201210256d16d76a49e6c8e2edc1c265d600ec1a64a45153d45c29a\
2fd0228c24c3a524ffffffff01405dc600000000001976a9140dfc8bafc84198\
53b34d5e072ad37d1a5159f58488ac00000000
```

输出结果如下：

```
f5a5ce5988cc72b9b90e8d1d6c910cda53c88d2175177357cc2f2cf0899fbaad
```

![12867278-9f907ea5664e70e5](12867278-9f907ea5664e70e5.webp)

