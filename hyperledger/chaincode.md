# 链码

### **生命周期**

- 打包：智能合约代码编码
- 安装：上传到背书节点
- 实例化：执行Init()
- 升级：对链码进行版本升级
- 交互：查询写入

![8](8.png)



### **5种系统链码**

- **LSCC（Lifecycle System Chaincode）生命周期管理链码**，安装、实例化、升级

- **CSCC（Configuration System Chaincode）配置管理链码**，如允许节点加入链

- **QSCC（Query System Chaincode）账本查询链码**，区块索引的外部服务

- **ESCC（Endorsement System Chaincode）交易背书链码**，对交易模拟结果进行封装、签名

- **VSCC（Validation System Chaincode）交易验证链码**

 

### **链码编程接口（shim.Chaincode）**

链码要继承shim.Chaincode接口

```
  Init(stub shim.ChaincodeStubInterface) pb.Response      //用于链码初始化

  Invoke(stub shim.ChaincodeStubInterface) pb.Response    //应用程序与链码交互的入口
```

stub shim.ChaincodeStubInterface是链码SDK接口

 

### **链码SDK接口（shim.ChaincodeStubInterface）**

- 参数解析

  ```
  GetArgs() [][]byte               //以byte数组的数组的形式获得传入的参数列表
  
  GetStringArgs() []string         //以字符串数组的形式获得传入的参数列表
  
  GetFunctionAndParameters() (string, []string) 
  //将字符串数组的参数分为两部分，数组第一个字是Function，剩下的都是Parameter
  
  GetArgsSlice() ([]byte, error)   //以byte切片的形式获得参数列表
  ```

- 交易信息

  ```
  GetTxID() string       //获取交易ID
  ```

- 状态操作

  ```
  GetState(key string) ([]byte, error)       //获取状态
  
  PutState(key string, value []byte) error   //更新状态
  
  DelState(key string) error                 //删除状态
  
  GetStateByRange(startKey, endKey string) (StateQueryIteratorInterface, error)    //获取状态区间
  
  GetStateByPartialCompositeKey(objectType string, keys []string) (StateQueryIteratorInterface, error)    //部分复合键的查询
  
  CreateCompositeKey(objectType string, attributes []string) (string, error)      //生成复合键
  
  SplitCompositeKey(compositeKey string) (string, []string, error)     //拆分复合键
  
  GetQueryResult(query string) (StateQueryIteratorInterface, error)    //富查询
  
  GetHistoryForKey(key string) (HistoryQueryIteratorInterface, error)  //历史数据查询
  ```

- 链码互操作(只读)

  ```
  InvokeChaincode(chaincodeName string, args [][]byte, channel string) pb.Response   //调用另外的链码
  ```

- 事件相关

  ```
  SetEvent(name string, payload []byte) error   //发送事件
  ```

- 其他

  ```
  GetCreator() ([]byte, error)    //获取当前用户的用户证书
  
  GetSignedProposal() (*pb.SignedProposal, error)   //获得签名的提案
  
  GetTransient() (map[string][]byte, error)         //获得Transient对象
  
  GetTxTimestamp() (*timestamp.Timestamp, error)    //获得交易时间戳
  
  GetBinding() ([]byte, error)    //获得Binding对象
  ```

  

### 交互方法套路

1. 检查参数的个数
2. 验证参数的正确性
3. 验证数据是否存在 应该存在 or 不应该存在
4. 其他逻辑验证
5. 写入状态



### **链码编程禁忌**

1. 分布式系统、多节点隔离执行，必须保证执行结果一致
2. 随机函数
3. 系统时间
4. 不稳定的外部依赖

 

### **应用开发流程**

1. 需求整理（数据上链、交互方法）
2. 链码编写
3. 链码交互

![10](10.png)