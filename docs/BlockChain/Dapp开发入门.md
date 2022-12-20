# Dapp开发入门

## 相关资源

web3.js相关：

- 官网：<https://web3js.org/>
- 官方文档：<https://web3js.readthedocs.io/en/v1.8.1/>
- 中文文档：<https://learnblockchain.cn/docs/web3.js/>
- Github: <https://github.com/web3/web3.js>

ethers相关：

- 官网：<https://ethers.org/>
- 中文文档：<https://learnblockchain.cn/docs/ethers.js/>

智能合约相关：

- Remix IDE：一款基于浏览器的智能合约开发IDE,地址 <https://remix.ethereum.org/>
- Solidity中文文档地址: <https://learnblockchain.cn/docs/solidity/>
- 崔棉大师solidity-example： <https://web3dao-cn.github.io/solidity-example/>

节点服务：

- infura：<https://www.infura.io/zh>

## Dapp介绍

Dapp前端的表现上是一样的， 还是H5页面、 小程序、APP，DAPP和传统App关键是后端部分不同，是后端不再是一个中心化的服务器，而是分布式网络上任意节点，注意可以是任意一个节点。

在应用中给节点发送的请求通常称为 **交易**，交易和中心化下的请求有几个很大的不同是：交易的数据经过用户个人签名之后发送到节点，节点收到交易请求之后，会把 请求广播到整个网络，交易在网络达成共识之后，才算是真正的执行（真正起作用的执行不一定是连接的后端节点，尽管该后端也会执行）。以及中心化下的请求大多数都是同步的（及时拿到结果）， 而交易大多数是异步的，这也是在开发去中心应用时需要注意的地方，从节点上获得数据状态（比如交易的结果），一般是通过**事件**回调来获得。

在开发中心化应用最重要两部分是 客户端UI和 后端服务程序， UI表现通过HTTP请求连接到后端服务程序，后端服务程序运行在服务器上，比如Nginx Apached等等。

开发一个去中心化应用最重要也是两部分： 客户端UI及 智能合约，智能合约的作用就像后端服务程序，智能合约是运行在节点的EVM上， 客户端调用智能合约，是通过向节点发起RPC请求完成。

下面是一个对比：客户端UI  <=> 客户端UI 、HTTP  <=> RPC、后端服务程序 <=> 智能合约、Nginx/Apache  <=> 节点

因此对于去中心化应用来说，程序员可以从两个方面切入:

一个是 去中心化应用的客户端开发， 熟悉已经熟悉客户端软件（如Web\APP等）开发的同学，只需要了解一下客户端跟区块链节点通信的API接口，如果是在当前应用最广泛的区块链平台以太坊上开发去中心化应用，那么需要了解Web3这个库，Web3对节点暴露出来的JSON-RPC接口进行了封装，比如Web3提供的功能有：获取节点状态，获取账号信息，调用合约、监听合约事件等等。

目前的主流语言都有Web3的实现是web3.js,它是以太坊官方的Javascript API，可以使用HTTP或者RPC与本地的或者远程的以太坊节点交互，从而调用智能合约。
另外还一个实现是ethers.js，代码量更少，接口也更简洁

另一个切入点是智能合约的开发，在以太坊现在推荐的语言是Solidity，有一些同学对新学一门语言有一些畏惧，Solidity的语法其实很简洁，有过一两门其他语言基础（开发经验）的同学三五天就可以学会。

## 开发入门(web3.js+solidity)

### 搭建测试链

在开发初期，我们并没有必要使用真实的公链，为了开发效率，一般选择在本地搭建测试链。在本文我们选择的Ganache，一个图形化测试软件（也有命令行版本），可以一键在本地搭建以太坊区块链测试环境，并且将区块链的状态通过图形界面显示出来。

安装参考：<https://trufflesuite.com/ganache/>

### 创建智能合约

目前以太坊官方全力支持的智能合约开发环境是Remix IDE，一款基于浏览器的智能合约开发IDE，地址 <https://remix.ethereum.org/>

我们在合约编辑页面编写如下代码：

```solidity
pragma solidity ^0.4.21;

contract InfoContract {

   string fName;
   uint age;

   function setInfo(string _fName, uint _age) public {
       fName = _fName;
       age = _age;
   }

   function getInfo() public constant returns (string, uint) {
       return (fName, age);
   }
}
```

代码很简单，就是简单的给name和age变量赋值与读取，接下来切换到Remix 的 run 的 tab 下，将Environment切换Web3 Provider为Ganache。

如果连接成功，那么在下面的Account的选项会默认选择 Ganache 创建的第一个账户地址。接下来我们点击Create就会将我们的智能合约部署到我们的测试网中。接下来 Remix 的页面不要关闭，在后面编写前端代码时还要用到合约的地址以及ABI信息。

### 创建 UI

在这之前，先在终端创建我们的项目：

```x
> mkdir info
> cd info
> npm init
> npm install web3 // 安装web3.js
```

在项目目录下创建index.html，在这里我们将创建基础的 UI，功能包括name和age的输入框，以及一个按钮，这些将通过 jQuery 实现：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
permalink: web3-html

    <link rel="stylesheet" type="text/css" href="main.css">

    <script src="./node_modules/web3/dist/web3.min.js"></script>

</head>
<body>
    <div class="container">

        <h1>Info Contract</h1>

        <h2 id="info"></h2>

        <label for="name" class="col-lg-2 control-label">Name</label>
        <input id="name" type="text">

        <label for="name" class="col-lg-2 control-label">Age</label>
        <input id="age" type="text">

        <button id="button">Update Info</button>


    </div>

    <script src="https://code.jquery.com/jquery-3.2.1.slim.min.js"></script>

    <script>
       // Our future code here..
    </script>

</body>
</html>
```

接下来需要编写main.css文件设定基本的样式：

```css
body {
    background-color:#F0F0F0;
    padding: 2em;
    font-family: 'Raleway','Source Sans Pro', 'Arial';
}
.container {
    width: 50%;
    margin: 0 auto;
}
label {
    display:block;
    margin-bottom:10px;
}
input {
    padding:10px;
    width: 50%;
    margin-bottom: 1em;
}
button {
    margin: 2em 0;
    padding: 1em 4em;
    display:block;
}

#info {
    padding:1em;
    background-color:#fff;
    margin: 1em 0;
}
```

### 使用Web3与智能合约交互

UI 创建好之后，在index.html中的 script 标签中间编写web.js的代码与智能合约交互。

首先创建web3实例，并与我们的测试环境连接：

```Javascript
// set a provider
const web3 = new Web3(Web3.givenProvider || "http://localhost:7545");

// abi json
const myAbi = [
    {
        "constant": true,
        "inputs": [],
        "name": "getInfo",
        "outputs": [
            {
                "name": "",
                "type": "string"
            },
            {
                "name": "",
                "type": "uint256"
            }
        ],
        "payable": false,
        "stateMutability": "view",
        "type": "function"
    },
    {
        "constant": false,
        "inputs": [
            {
                "name": "_fName",
                "type": "string"
            },
            {
                "name": "_age",
                "type": "uint256"
            }
        ],
        "name": "setInfo",
        "outputs": [],
        "payable": false,
        "stateMutability": "nonpayable",
        "type": "function"
    }
];

const myContractAddress = "0xc694836ff32a421De673b4A2fA4184728d32e470";
const myAccount = "0xc038c9C7C61820f111460E7b637992fB283d21AA";

// create a new contract object, providing the ABI and address
const myContract = new web3.eth.Contract(myAbi, myContractAddress);

// using contract.methods to get value
myContract.methods.getInfo().call({from: myAccount}, function(error, result){
    console.log(result)
    $("#name").val(result[0]);
    $("#age").val(result[1]);
});

$("#button").click(function() {
    var name = $("#name").val();
    var age = $("#age").val();
    if (name == null || age == null) {
        alert("姓名或年龄不能为空");
        return;
    }
    
    myContract.methods.setInfo(name, age).send({from: myAccount})
    .then(function(receipt){
        console.log(receipt);
        alert("更新成功")
    });
    
    myContract.methods.getInfo().call({from: myAccount}, function(error, result){
        console.log(result)
        $("#name").val(result[0]);
        $("#age").val(result[1]);
    });
});    
```

以上的代码就简单地实现了对合约中两个函数的调用，分别读取和显示name和age变量。

在浏览器中打开index.html测试效果如下图（输入名字和年龄后刷新）。

## 开发入门(ethers.js+solidity)

## 工程化开发(Truffle)

官网：<https://trufflesuite.com/>
中文文档：<https://learnblockchain.cn/docs/truffle/>

## 工程化开发(Hardhat)

官网：<https://hardhat.org/>
中文文档：<https://learnblockchain.cn/docs/hardhat/getting-started/>
