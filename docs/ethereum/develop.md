# 开发

**Whisper** 以太坊开发人员开发的去中心化的通讯协议

**Swarm**    以太坊开发人员开发的去中心化的文件系统

 

**Truffle**：以太坊智能合约开发环境，测试框架，以太坊的资源管理通道

**Ganache**：以太坊节点仿真器软件

 

### **Truffle**

1. 安装Truffle

   ```bash
   npm install --global --production windows-build-tools  //Windows环境需要先安装依赖
   npm install -g truffle
   ```

2. 创建Truffle项目

   ```bash
   truffle unbox xxxxx
   //or
   truffle init
   ```

3. Truflle项目目录结构

   contracts/:智能合约目录，sol文件

   migrations/:用作移植的脚本文件，js文件

   test/: 测试文件目录，sol或js文件

   truffle-config.js: Truffle 配置文件

4. 编译

   ```bash
   truffle compile
   ```

5. 部署

   migrations/中编写移植脚本，然后：

   ```bash
   truffle migrate
   ```

6. 测试

   test/中编写测试代码

7. JS中Web3初始化

   设置provider，创建一个web3实例

   ```js
   	if (typeof web3 !== 'undefined') {
   		App.web3Provider = web3.currentProvider;
   	} else {
   		// set the provider you want from Web3.providers
   		App.web3Provider =
   		new Web3.providers.HttpProvider("http://localhost:8545");
   		web = new Web3(App.web3Provider);
   	}
   ```

8. JS中初始化合约

   获取build/contracts/中的合约json（data）

   ```js
   App.contracts.conxxxx = TruffleContract(data);
   App.contracts.conxxxx.setProvider(App.web3Provider);
   ```

9. 合约交互

   ```js
   	//获取当前以太坊账户
   	web3.eth.getAccounts(function(error, accounts){
   		var account = accounts[0];
   		//获取合约实例：
   		App.contracts.conxxxx.deployed().then(function(conxxxxInstance){
   			//调用合约方法：
   			return conxxxxInstance.funname.call();
   			//or return conxxxxInstance.funname(arg... , {from: account});
   		}).then(function(resulet){
           	//TODO
   		}).catch(function(err){
   			//console.log(err.message);
   		})
   	})
   ```