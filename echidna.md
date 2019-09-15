[toc]
# Echidna
### 一、Echidna是什么
Echidna是使用haskell语言，基于属性的针对evm代码的模糊测试工具。它支持相对复杂的基于愈发的模糊活动，以伪造各种判断。
### 二、特性
#### 语言特性
- 根据您的实际代码生成输入
- 可选的覆盖指南，以发现更深层的错误
- 快速
- 强大的API，用于高级用法
- 自动测试用例最小化，可快速分类（不用写测试步骤，只写需要达到的测试目的）
- 不用复杂设置，使用solidity书写propertis
- 直接集成echidna测试合约到开发项目中
### 三、用法

#### 1. 下载docker镜像
```
docker pull docker.io/trailofbits/echidna
```
*或者用dockerfile创建docker镜像，但经测试创建不成功*
```
$ cd /echidna
$ docker build -t echidna .

```

#### 2. 运行docker容器
```
$ cd /echidna
$ docker run -t -v `pwd`:/src echidna echidna-test /src/examples/solidity/basic/flags.sol
```
#### 3. 书写测试代码
虽然可以直接把测试代码写到目标合约里，但一般都是创建一个新合约继承自要测试的合约来进行测试，以将目标合约与测试合约分离

##### 2.1 构造函数
测试合约可以有构造函数，但不能有参数；构造函数做一些初始化的工作
###### 2.1.1 两个特殊地址

教程中没有写明白这两个地址到底什么时候用，但是根据例子的观察，应该是测试erc20或eth转账相关的时候使用

1. 当被测合约没有构造函数时使用地址：0x00a329c0648769a73afac7f9381e08fb43dbea70
2. 当被测合约有构造函数时使用地址：0x00a329c0648769a73afac7f9381e08fb43dbea72

##### 2.2 如何书写属性
Echidna属性是一个solidity方法，该方法有如下特征：
1. 没有参数
2. 不会改变合约状态变量（如view修饰的方法）
3. 返回true表示测试成功
4. 方法名以echidna_开头

Echidna会设法发出交易使得该属性返回false或抛出异常。这就是Echidna核心做的工作。

##### 2.3 示例

目标合约
```
   contract Token{
      mapping(address => uint) public balances;
      function airdrop() public{
          balances[msg.sender] = 1000;
     }
     function consume() public{
          require(balances[msg.sender]>0);
          balances[msg.sender] -= 1;
     }
     function backdoor() public{
          balances[msg.sender] += 1;
     }
   }   
  
```
测试合约
```
    contract TestToken is Token{
         function echidna_balance_under_1000() public view returns(bool){
               return balances[msg.sender] <= 1000;
          }
     }
```

#### 4. 测试
**语法：**
```
echidna-test FILE [CONTRACT] [--config ARG]
```
**示例**
```
$ echidna-test myContract.sol TestToken --config="config.yaml"
```



### 四、配置项

配置文件中的各项意义

* `testLimit`
  * Type: Int
  * Default: `10000`
  * Description: Number of sequences of transactions to generate during testing.  

* `seqLen` 
  * Type: Int
  * Default: `100`
  * Description: Number of transactions to generate during testing.

* `shrinkLimit` 
  * Type: Int
  * Default: `5000`
  * Description: Number of tries to attempt to shrink a failing sequence of transactions.

* `contractAddr`
  * Type:  Address
  * Default: `"0x00a329c0648769a73afac7f9381e08fb43dbea72"`
  * Description: Address to deploy the contract to test.

* `deployer`
  * Type:  Address
  * Default: `"0x00a329c0648769a73afac7f9381e08fb43dbea70"`
  * Description: Address of the deployer of the contract to test.

* `sender`
  * Type: [Address]
  * Default: `["0x00a329c0648769a73afac7f9381e08fb43dbea70"]`
  * Description: List of addresses to (randomly) use during for the transactions sent during testing.

* `psender`
  * Type: Address
  * Default: `"0x00a329c0648769a73afac7f9381e08fb43dbea70"`
  * Description: Address of the sender of the property to test.

* `prefix`
  * Type: String 
  * Default: `"echidna_"`
  * Description: Prefix of the function names used as properties in the contract to test. 

* `solcArgs`
  * Type: String 
  * Default: `""`
  * Description: Additional arguments to use in `solc` for the compilation of the contract to test. 

* `quiet`
  * Type: Bool
  * Default: `False`
  * Description: Hide `solc` stderr output and additional information during the testing.

* `dashboard`
  * Type: Bool
  * Default: `True`
  * Description: Show the ncurses dashboard with real-time information on the properties to test during the fuzzing campaign.

* `style`
  * Type: String
  * Default: `"text"`
  * Description: Select an UI to show the results of each test. 
      * `"text"`: simple textual interface.
      * `"json"`: JSON output.
      * `"none"`: no output.

* `initialBalance`
  * Type: Int
  * Default: `0xffffffff`
  * Description: Initial Ether balance of `deployer` and each of the `sender` accounts.

### 五、相关文档

1. [Tutorial中的三个Exercise](https://github.com/trailofbits/publications/blob/master/workshops/Automated%20Smart%20Contracts%20Audit%20-%20TruffleCon%202018/echidna/exercises)
2. [Echidna](https://github.com/crytic/echidna)
3. [wiki](https://github.com/crytic/echidna.wiki.git)



