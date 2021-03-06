 # CryptoGladiator
 [CryptoGladiator-API文档访问地址](https://funjumping.github.io/CrazyGladiator/neo-cg-api-html/neo-cg-api.html)
#### 一.  合约说明
角斗士项目中总共使用三个合约：[sgas](https://github.com/NewEconoLab/neo-ns/tree/master/dapp_sgas)，NFT和Auction，其中[sgas](https://github.com/NewEconoLab/neo-ns/tree/master/dapp_sgas)合约为NEL开发的通用合约，NFT和Auction 为我们自己开发。
NFT合约管理角斗士资源，以及克隆操作；
Auction合约管理角斗士的拍卖，出租，购买和手续费扣取。
本项目所用合约均使用C#开发。
1.	Sgas.cs
作用：用于gas 和[sgas](https://github.com/NewEconoLab/neo-ns/tree/master/dapp_sgas)之间1：1兑换。
由于合约里直接操作gas比较困难，所以需要玩家先将gas通过[sgas](https://github.com/NewEconoLab/neo-ns/tree/master/dapp_sgas)合约转换为NEP5资产，合约里操控的是[sgas](https://github.com/NewEconoLab/neo-ns/tree/master/dapp_sgas)资产，这样合约写起来会比较方便。
2.	NFT.cs
作用：管理角斗士资源，以及克隆操作。
3.	Auction.cs
作用：管理角斗士的拍卖，出租，购买和手续费扣取。
#### 二.  常见问题：
1.	如何获取我拥有的角斗士资产？
这个应该是由tokensOfOwner接口提供的，但是如果要在链上统计获取这些数据，角斗士比较多的时候，这个接口的手续费肯定是要超过10GAS的，所以该接口暂不支持。
现在我们的做法是在产生角斗士的时候，发出通知（ApplicationLog），后台监听到通知后，入后台数据库进行记录，在前端需要的时候由后台提供接口获取角斗士数据。
2.	如何充值到拍卖行
玩家先将gas通过sgas合约，充值进入合约转换成sgas，再将sgas转账给拍卖合约所代表的钱包地址，最后将该转账的交易txid，传给Auction拍卖合约，Auction合约会去sgas合约里查询交易记录，看这笔记录是否转账成功以及转账金额，将该金额记录在拍卖合约的storage中，即充值成功。为避免重复充值，拍卖合约中会标记哪些txid已经处理过。
3.	玩家在拍卖场里买东西为什么要提前充值到拍卖行
因为如果不提前充值到拍卖行，买东西的时候，合约没有办法主动扣除玩家的gas或者sgas。
4. 关于结构体的序列化
	2.7以后的版本支持使用Helper.Deserialize进行序列化，2.6的版本不支持该方法，目前测试网络的版本为2.6，主网的版本为2.7
#### 三.  注意事项：
1. 关于手续费
官方文档上说：“每个智能合约在每次执行过程中有10 GAS 的免费额度，无论是开发者部署还是用户调用，因此，单次执行费用在 10 GAS 以下的智能合约是不需要支付手续费的。当单次执行费用超过10 GAS，会减免10 GAS 的手续费。”。
我们在写智能合约时，应该控制合约单个执行函数的复杂度，尽量将手续费控制在10GAS以内，这样系统减免后就可以免费调用。
为了将手续费控制在10GAS以内， Storage.Put调用的次数在一个调用内尽量不要超过6个；尽量避免遍历读取storage里的数据；尽量避免在循环里多次调用一个复杂的私有函数。
2. 关于数据类型
智能合约中没有string类型，使用Byte字节数组来表示string，不要将int等类型转换为string，不支持该操作；
智能合约中数值类型只有BigInteger，最终也被存储为ByteArray。C#中的byte，int，long等都会被自动编译成BigInteger类型。
	智能合约中不支持C#的浮点小数，可以使用BigInteger表示的定点小数，需要约定小数位数。
	C#合约中的结构体，不支持自定义构造函数，不支持在其中自定义成员函数。
3. 其他常见错误：
不要在智能合约中定义类变量，不支持；可以定义类常量。

