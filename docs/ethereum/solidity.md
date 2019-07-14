# Solidity

中文文档：[Solidity]( https://solidity-cn.readthedocs.io/zh/develop/index.html	"Solidity中文文档")



### 语言依赖

**NodeJS**是对Google V8引擎进行封装的JS解释器，一种服务器端的JS运行环境

**npm**是NodeJS的包管理器

**nodist**是Windows下NodeJS版本管理器

**remix**是

Windows本地安装remix时需要安装依赖

```bash
npm install --global --production windows-build-tools
```

 

### **合约文件**

版本

Import

合约结构

​       状态变量

​       函数

​       结构类型

​       枚举类型

​       事件

​       函数修改器

代码注释

 

### **值类型**

Solidity语言是静态类型，必须声明时指定类型（除赋值时的类型推断外）

值类型的变量将始终进行值拷贝来传递

- **布尔类型**

​              bool a = true;

- **整型**

​              int/uint（从8-256以8步进，如int8，默认256）

​              负数右移向0取整

- **定长浮点型**

​              fixed/ufixed    fixMxN（默认128x19）

- **定长字节数组**

​              byte（从1-32以1步进，如byte8，默认1）

​              .length

​              byte[x]，浪费存储空间，尽量使用变长字节数组

- **有理数和整型字面常数**

​              没有八进制

​              支持任意精度，即在常数表达式中不会溢出也不会被截断

​              每个有理数有自己的字面常数类型

- **字符串字面常数**

​              没有结尾符

​              可隐式转成byte bytes string等

- **十六进制字面常数**

​              hex开头，如hex“001122ff”

​              byte1 a = hex“aa”;              //**两位十六进制字符可以表示一字节**

- **枚举**

​              enum Enumexample {A, B, C}

​              枚举类型不属于 |ABI| 的一部分

- **函数类型**

  ```
    function (<parameter types>) = 
          {internal | external | public | private} 
          [pure | constant | view | payable] 
          [modifierfunc(<parameter types>)] 
          [returns (<return types>)]
  ```

- **地址类型**

​              20字节值

​              成员变量：balance、transfer()、send()（有风险，尽量使用transfer）、

- **地址字面常数**

​              通过了地址校验和测试的十六进制字面常数



### **引用类型（复杂类型）**

- **数据的位置**

  复杂类型（数组、结构体）才有数据位置

  - Memory（内存）：内存中

    函数参数（包括返回的参数）值默认memory

  - Storage（存储）：永久存储在区块链中

    局部变量值默认storage，状态变量值强制storage

  - calldata：跟memory差不多

    外部函数的参数（非返回参数）强制calldata

- **值传递方式**
  
  状态变量 = 函数参数                 //拷贝
  
  状态变量 = 状态变量                 //拷贝
  
  状态变量 = 内存局部变量         //拷贝
  
  内存局部变量 = 函数参数          //拷贝
  
  内存局部变量 = 状态变量          //拷贝
  
  内存局部变量 = 内存局部变量  //拷贝
  
  存储局部变量 = 状态变量          //引用
  
- **数组（Arrays）**

  T[k]、T[]、bytes、string

  示例：

  ```solidity
    int[] memory a = new int[](3);      //内存变量可使用new创建数组，存储变量不行
  
    int[][5] memory a = [1,2,3,4,5];    //声明时与其他语言不同，长度从右向左读
  
    int b = a[5][0] ;                   //访问时和其他语言一样，索引从左向右读
  ```

  new创建的数组和数组字面常数[1,2,3]都是**定长的内存类型**

  **内存类型**数组只能赋给状态变量或声明了memory的局部变量

  数组的成员变量：**length，push()**

  .length获取长度，但只有存储数组才能给length赋值来改变数组长度

  push()只有变长数组或bytes才能使用

  数组的删除：

  ```
    delete a[0];
    delete a;     //只有非引用变量才可删除
  ```

- **结构体**

  ```solidity
    struct Funder {
           address a;
           uint b;
    }
  
    Funder f = Funder({a:msg.sender, b:10});
  
    Funder f = Funder(msg.sender, 10);
  ```
  结构体不能包含本身

 

### **映射类型（Mappings）**

- 可视为哈希表，但实际上不存储 key，而是存储它的 keccak256 哈希值

- 所以没有key集合和value集合

- 只有状态变量能使用映射

- 键值不能是映射、变长数组、合约、枚举、结构体

  ```solidity
  mapping(address => uint) public balances;
  ```

  

### **LValues**

LValues是一个变量或者其它可以被赋值的东西

非引用类型的Lvalue变量可以使用**delete**初始化值（会递归delete）

初始化结果：

- 静态数组：重置所有元素
- 动态数组：将长度设为0
- 映射：遇到映射则无效跳过，但可以使用delete a[xxx]删除单个键值

 

### 基本类型转化

- **隐式转化**

  一般来说，只要值类型之间的转换在语义上行得通，而且转换的过程中没有信息丢失，那么隐式转换基本都是可以实现的

  ```solidity
  uint16 a = 0;
  ```

- **显式转化**

  会发生一些无法预料的后果，因此一定要进行测试，确保结果是你想要的

  ```solidity
  int8 y = -3;
  uint x = uint(y);
  ```

- **类型推断**

  编译器会根据分配该变量的第一个表达式的类型自动推断该变量的类型

  ```solidity
  var a = 0;       //a为uint8
  
  uint24 x = 0x123;
  var y = x;       //y为uint24
  ```

 

### **以太币单位**

wei（默认）、finney、szabo、ether

### **时间单位**

seconds、minutes、hours、days、weeks、years（不再支持）

单位后缀不能直接用在变量后面

 

### **控制结构**

与C语言和Javascript类似的关键词

if、else、while、do、for、break、continue、return、? : 

没有switch和goto

​       Solidity中非布尔类型数值不能转为布尔类型，如不能 if(1) {...}



### **函数详解**

```
  function (<parameter types>) = 
        {internal | external | public | private} 
        [pure | constant | view | payable] 
        [modifierfunc(<parameter types>)] 
        [returns (<return types>)]
```

#### **函数参数**

- **输入参数**

  输入参数的声明方式与变量相同。但是有一个例外，未使用的参数可以省略参数名

- **输出参数**

  输出参数的声明方式在关键词 returns 之后，与输入参数的声明方式相同，**可命名，可多个**

  ```
    function a(int x, int y) public returns (int m, int n){
        m = x;
        n = y;
        //或 return (x, y) ;
        //或 (m, n) = (x, y) ;
    }
  ```

- **元组**

  元组使用小括号，可用于解构赋值和返回多个结果

  ```
    var (x, y, z) = f();
    (int x, int y, int z) = f();
    ( , y, z) = f();
    (x, ) = (1, 2);
    (x, z) = (z, x);
  ```

#### **可见性**

- **public（默认）**

  声明了public的函数是合约接口的一部分，即可内部调用，也可外部调用，比external消耗的gas更多

- **external**

  是合约接口的一部分，只能使用外部调用

- **internal**

  只能是内部访问（即当前合约或继承的合约里调用），不使用 `this` 调用

- **private**

  仅在当前定义它们的合约中使用，并且不能被继承的合约使用

#### 声明类型

- **view/constant 视图函数**

  将函数声明为 `view` 类型，这种情况下要保证不修改状态

  - `constant` 是 `view` 的别名
  - Getter 方法被标记为 `view`
  - 编译器**没有强制** `view` 方法不能修改状态

  下面的语句被认为是修改状态：

  1. 修改状态变量
  2. 产生事件
  3. 创建其它合约
  4. 使用 `selfdestruct`
  5. 通过调用发送以太币
  6. 调用任何没有标记为 `view` 或者 `pure` 的函数
  7. 使用低级调用
  8. 使用包含特定操作码的内联汇编

- **pure 纯函数**

  函数可以声明为 `pure` ，在这种情况下，承诺不读取或修改状态

  以下被认为是从状态中读取：

  1. 读取状态变量。
  2. 访问 `this.balance` 或者 `<address>.balance`。
  3. 访问 `block`，`tx`， `msg` 中任意成员 （除 `msg.sig` 和 `msg.data` 之外）。
  4. 调用任何未标记为 `pure` 的函数。
  5. 使用包含某些操作码的内联汇编。

- **payable 可支付函数**

  如果一个函数需要进行货币操作，必须要带上`payable`关键字

  调用该函数时，可使用.value(x).gas(y)来设置与函数调用一起发送的 Wei 值和 gas 的数量

  ```solidity
  contract InfoFeed {
      function info() public payable returns (uint ret) { return 42; }
  }
  
  contract Consumer {
      InfoFeed feed;
      function setFeed(address addr) public { feed = InfoFeed(addr); }
      function callFeed() public { feed.info.value(10).gas(800)(); }
  }
  ```

#### 修饰器

- 使用修饰器modifier可以轻松改变函数的行为。 例如，它们可以在执行函数之前自动检查某个条件。 修饰器是合约的可继承属性， 并可能被派生合约覆盖。

- 如果同一个函数有多个修饰器，它们之间以空格隔开，修饰器会依次检查执行。

- 修饰器或函数体中显式的 return 语句仅仅跳出当前的修饰器和函数体。 返回变量会被赋值，但整个执行逻辑会从前一个 修饰器中的定义的 “_” 之后继续执行。

- 修饰器的参数可以是任意表达式，在此上下文中，所有在函数中可见的符号，在 修饰器中均可见。 在 修饰器中引入的符号在函数中不可见（可能被重载改变）。

  ```
      // 修改器可以接收参数：
      modifier costs(uint price) {
          if (msg.value >= price) {
              _;
          }
      }
      function register() public payable costs(price) {
          registeredAddresses[msg.sender] = true;
      }
  ```

#### 函数调用类型

- **内部函数调用**

  当前合约中的函数可以直接（“从内部”）调用，也可以递归调用

- **外部函数调用**

  会产生一个EVM 调用或称为“消息调用”

  ```
    (contact).funcname.value(10).gas(800)();    //使用指定合约变量调用
  
    this.funcname.value(10).gas(800)();         //使用当前合约调用
  ```

#### 函数重载

- 合约可以具有多个不同参数的同名函数。这也适用于继承函数。

- 重载函数也存在于外部接口中。如果两个外部可见函数仅区别于 Solidity 内的类型而不是它们的外部类型则会导致错误。
- 通过将当前范围内的函数声明与函数调用中提供的参数相匹配，可以选择重载函数。 如果所有参数都可以隐式地转换为预期类型，则选择函数作为重载候选项。如果一个候选都没有，解析失败。
- 若参数通过隐式转换能同时匹配多个函数，调用时会导致类型错误。



### **区块和交易属性函数**

- **block.blockhash(uint blockNumber) returns (bytes32)**

  指定区块的区块哈希——仅可用于最新的 256 个区块且不包括当前区块；而 blocks 从 0.4.22 版本开始已经不推荐使用，由 blockhash(uint blockNumber) 代替

- **block.coinbase (address)**

  挖出当前区块的矿工地址

- **block.difficulty (uint)**

  当前区块难度

- **block.gaslimit (uint)**

  当前区块 gas 限额

- **block.number (uint)**

  当前区块号

- **block.timestamp (uint)**

  自 unix epoch 起始当前区块以秒计的时间戳

- **gasleft() returns (uint256)**

  剩余的 gas

- **msg.data (bytes)**

  完整的 calldata

- **msg.gas (uint)**

  剩余 gas - 自 0.4.21 版本开始已经不推荐使用，由 gesleft() 代替

- **msg.sender (address)**

  消息发送者（当前调用）

- **msg.sig (bytes4)**

  calldata 的前 4 字节（也就是函数标识符）

- **msg.value (uint)**

  随消息发送的 wei 的数量

- **now (uint)**

  目前区块时间戳（block.timestamp）

- **tx.gasprice (uint)**

  交易的 gas 价格

- **tx.origin (address)**

  交易发起者（完全的调用链）

 

### **ABI** **编码函数**

**Application Binary Interface(ABI)：**应用二进制接口

- **abi.encode(...) returns (bytes)**： 

  ABI - 对给定参数进行编码

- **abi.encodePacked(...) returns (bytes)**

  对给定参数执行 紧打包编码

- **abi.encodeWithSelector(bytes4 selector, ...) returns (bytes)**

  ABI - 对给定参数进行编码，并以给定的函数选择器作为起始的 4 字节数据一起返回

- **abi.encodeWithSignature(string signature, ...) returns (bytes)**

  等价于 abi.encodeWithSelector(bytes4(keccak256(signature), ...)

 

### **错误处理函数**

错误处理方式：**回退所有状态**

- **assert(bool condition)**      会消耗gas

  如果条件不满足，则使当前交易没有效果 — 用于检查内部错误。

- **require(bool condition)**:    不会消耗gas

  如果条件不满足则撤销状态更改 - 用于检查由输入或者外部组件引起的错误。

- **require(bool condition, string message)**

  如果条件不满足则撤销状态更改 - 用于检查由输入或者外部组件引起的错误，可以同时提供一个错误消息。

- **revert()**

  终止运行并撤销状态更改。

- **revert(string reason)**

  终止运行并撤销状态更改，可以同时提供一个解释性的字符串。

 

### 数学和密码学函数

- **addmod(uint x, uint y, uint k) returns (uint)**

  计算 `(x + y) % k`，加法会在任意精度下执行，并且加法的结果即使超过 `2**256` 也不会被截取。从 0.5.0 版本的编译器开始会加入对 `k != 0` 的校验（assert）。

- **mulmod(uint x, uint y, uint k) returns (uint)**

  计算 `(x * y) % k`，乘法会在任意精度下执行，并且乘法的结果即使超过 `2**256` 也不会被截取。从 0.5.0 版本的编译器开始会加入对 `k != 0` 的校验（assert）。

- **keccak256(...) returns (bytes32)**

  计算 [(tightly packed) arguments](https://solidity-cn.readthedocs.io/zh/develop/abi-spec.html#abi-packed-mode) 的 Ethereum-SHA-3 （Keccak-256）哈希。

- **sha256(...) returns (bytes32)**

  计算 [(tightly packed) arguments](https://solidity-cn.readthedocs.io/zh/develop/abi-spec.html#abi-packed-mode) 的 SHA-256 哈希。

- **sha3(...) returns (bytes32)**

  等价于 keccak256。

- **ripemd160(...) returns (bytes20)**

  计算 [(tightly packed) arguments](https://solidity-cn.readthedocs.io/zh/develop/abi-spec.html#abi-packed-mode) 的 RIPEMD-160 哈希。

- **ecrecover(bytes32 hash, uint8 v, bytes32 r, bytes32 s) returns (address)**

  利用椭圆曲线签名恢复与公钥相关的地址，错误返回零值。 ([example usage](https://ethereum.stackexchange.com/q/1777/222))



### 地址相关函数

- `<address>`.balance (uint256)

  以 Wei 为单位的 地址类型的余额。

- `<address>`.transfer(uint256 amount)

  向 地址类型发送数量为 amount 的 Wei，失败时抛出异常，发送 2300 gas 的矿工费，不可调节。

- `<address>`.send(uint256 amount) returns (bool)

  向地址类型发送数量为 amount 的 Wei，失败时返回 `false`，发送 2300 gas 的矿工费用，不可调节。

- `<address>`.call(...) returns (bool)

  发出低级函数 `CALL`，失败时返回 `false`，发送所有可用 gas，可调节。

- `<address>`.callcode(...) returns (bool)

  发出低级函数 `CALLCODE`，失败时返回 `false`，发送所有可用 gas，可调节。

- `<address>`.delegatecall(...) returns (bool)

  发出低级函数 `DELEGATECALL`，失败时返回 `false`，发送所有可用 gas，可调节。



### 状态变量

#### **可见性**

- **public**

  会自动创建一个访问器（Getter函数）

- **private**

  仅在当前定义它们的合约中使用，并且不能被继承的合约使用

- **internal（默认）**

  只能是内部访问（即当前合约或继承的合约里调用），不使用 `this` 调用

#### Getter函数

编译器自动为所有 **public** 状态变量创建 getter 函数

```
pragma solidity ^0.4.0;

contract C {
    uint public data = 42;
}

contract Caller {
    C c = new C();
    function f() public {
        uint local = c.data();
    }
}
```

getter 函数具有外部可见性。如果在内部访问 getter（即没有 `this.` ），它被认为一个状态变量。 如果它是外部访问的（即用 `this.` ），它被认为为一个函数。

```
pragma solidity ^0.4.0;

contract C {
    uint public data;
    function x() public {
        data = 3; // 内部访问
        uint val = this.data(); // 外部访问
    }
}
```

下一个例子稍微复杂一些：

```
pragma solidity ^0.4.0;

contract Complex {
    struct Data {
        uint a;
        bytes3 b;
        mapping (uint => uint) map;
    }
    mapping (uint => mapping(bool => Data[])) public data;
}
```

这将会生成以下形式的函数：

```
function data(uint arg1, bool arg2, uint arg3) public returns (uint a, bytes3 b) {
    a = data[arg1][arg2][arg3].a;
    b = data[arg1][arg2][arg3].b;
}
```

因为没有好的方法来提供映射的键，所以结构中的映射被省略。



### 合约详解

#### 创建合约



#### **构造函数**

constructor (uint a) public {

}

#### **回退函数（Fallback）**

合约可以有一个**未命名**的函数。这个函数**不能有参数也不能有返回值**。 如果在一个到合约的调用中，没有其他函数与给定的函数标识符匹配（或没有提供调用数据），那么这个函数（fallback 函数）会被执行。

每当合约收到以太币（没有任何数据），这个函数就会执行。此外，为了接收以太币，fallback 函数必须标记为 `payable`。 如果不存在这样的函数，则合约不能通过常规交易接收以太币。

```solidity
   function() public payable { }
```

- 一个合约只能有一个
- 若合约的调用找不到匹配函数则会调用回退函数
- 若合约要接收以太币，则必须有声明了payable的回退函数
- 每当合约接收以太币时，会自动调用回退函数
- 回退函数应该尽量减少gas消耗（低于 2300 个 gas）

#### 事件

#### 继承

#### 抽象合约

#### 接口

#### 库



### 代币实现标准ERC-20

EIP是提议，ERC是结果

https://eips.ethereum.org/EIPS/eip-20

https://github.com/ethereum/EIPs/blob/master/EIPS/eip-20.md

```go
contract ERC20Interface {
	uint256 public totalSupply;
    // //获取token名字，比如：“jiarenminbi”
    // function name() view returns (string name);
    // //获取token简称，比如："JRMB"
    // function symbol() view returns (string symbol);
    // //获取小数位，比如以太坊的decimals为18：10^18
    // function decimals() view returns (uint8 decimals);
    // //获取token发布的总量,比如：EOS 10亿
    // function totalSupply() view returns (uint256 totalSupply);
    //获取_owner地址的余额
    function balanceOf(address _owner) view returns (uint256 balance);
    //转帐：调用方向_to转_value个token
    function transfer(address _to, uint256 _value) returns (bool success);
    //转帐：从_from向_to转_value个token
    function transferFrom(address _from, address _to, uint256 _value) returns (bool success);
    //允许_spender从自己（调用方）帐户转走_value个token
    function approve(address _spender, uint256 _value) returns (bool success);
    //自己（_owner）查询_spender地址一共可以转走自己多少个token
    function allowance(address _owner, address _spender) view returns (uint256 remaining);
    //转帐的时候必须要调用的事件，如transfer,transferFrom方法
    event Transfer(address indexed _from, address indexed _to, uint256 _value);
    //成功执行approve后调用的事件
    event Approval(address indexed _owner, address indexed _spender, uint256 _value);
}
```

 