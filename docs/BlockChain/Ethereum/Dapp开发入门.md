# Dapp开发入门

## Dapp介绍

Dapp前端的表现上是一样的， 还是H5页面、 小程序、APP，DAPP和传统App关键是后端部分不同，是后端不再是一个中心化的服务器，而是分布式网络上任意节点，注意可以是任意一个节点。

在应用中给节点发送的请求通常称为 **交易**，交易和中心化下的请求有几个很大的不同是：交易的数据经过用户个人签名之后发送到节点，节点收到交易请求之后，会把 请求广播到整个网络，交易在网络达成共识之后，才算是真正的执行（真正起作用的执行不一定是连接的后端节点，尽管该后端也会执行）。以及中心化下的请求大多数都是同步的（及时拿到结果）， 而交易大多数是异步的，这也是在开发去中心应用时需要注意的地方，从节点上获得数据状态（比如交易的结果），一般是通过**事件**回调来获得。

在开发中心化应用最重要两部分是 客户端UI和 后端服务程序， UI表现通过HTTP请求连接到后端服务程序，后端服务程序运行在服务器上，比如Nginx Apached等等。

开发一个去中心化应用最重要也是两部分： 客户端UI及 智能合约，智能合约的作用就像后端服务程序，智能合约是运行在节点的EVM上， 客户端调用智能合约，是通过向节点发起RPC请求完成。

下面是一个对比：客户端UI  <=> 客户端UI 、HTTP  <=> RPC、后端服务程序 <=> 智能合约、Nginx/Apache  <=> 节点

因此对于去中心化应用来说，程序员可以从两个方面切入:

一个是 去中心化应用的客户端开发， 熟悉已经熟悉客户端软件（如Web\APP等）开发的同学，只需要了解一下客户端跟区块链节点通信的API接口。

另一个切入点是智能合约的开发，在以太坊现在推荐的语言是Solidity，有一些同学对新学一门语言有一些畏惧，Solidity的语法其实很简洁，有过一两门其他语言基础（开发经验）的同学三五天就可以学会。

## Dapp与合约交互方式

为了让Dapp与以太坊区块链交互（通过读取区块链数据或向网络发送交易），它必须连接到以太坊节点。为此目的，每个以太坊客户端都实现了一项 JSON-RPC 规范，因此有一套统一的方法可供应用程序依赖，无论具体的节点或客户端实现如何。详细介绍: <https://ethereum.org/zh/developers/docs/apis/json-rpc/>

web3.js：对节点暴露出来的JSON-RPC接口进行了封装，可以使用HTTP或者RPC与本地的或者远程的以太坊节点交互，从而调用智能合约。比如Web3提供的功能有：获取节点状态，获取账号信息，调用合约、监听合约事件等等。web3.js官网：<https://web3js.org/>

另外还一个实现是ethers.js，代码量更少，接口也更简洁，与web3.js相比，ethers.js有很多优点，一个特性是ethers.js提供的状态和密钥管理。web3的设计场景是DApp应该连接到一个本地节点，由这个节点负责保存密钥、签名交易并与以太坊区块链交互。现实并不是这样的，绝大多数用户不会在本地运行一个geth节点。Metamask在浏览器应用中有效地模拟了这种节点环境，因此绝大多数web3应用需要使用Metamask来保存密钥、签名交易并完成与以太坊的交互。ethers.js官网：<https://ethers.org/>

Ethers.js采取了不同的设计思路，它提供给开发者更多的灵活性。Ethers.js将“节点”拆分为两个不同的角色：

- 钱包：负责密钥保存和交易签名
- 提供器：负责以太坊网络的匿名连接、状态检查和交易发送

## 工程化开发(Hardhat)

- 官网：<https://hardhat.org/>
- 中文文档：<https://learnblockchain.cn/docs/hardhat/getting-started/>

## 工程化开发(Truffle)

- 官网：<https://trufflesuite.com/>
- 中文文档：<https://learnblockchain.cn/docs/truffle/>

## TheGraph入门

官网：<https://thegraph.com/>
