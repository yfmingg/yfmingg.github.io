# AwesomeBlockChain

## APR/APY

- APR代表年度百分比率（Annual Percentage Rate）。它是实际的年回报率，不考虑复利的影响。
- APY代表年溢率百分比（Annual Percentage Yield）。它是实际的年收益率，考虑了复利的影响。
- APY更适合计算你的投资回报，而APR在贷款中更常见。

## Web3 Slang/俚语

In the web3 space, there is a unique vocabulary and set of terminologies, often referred to as "slang," that are commonly used. 

Understanding these terms can help you navigate the world of blockchain and decentralized technologies. 

Here are some important slang terms to know in web3:

- HODL: A misspelling of "hold," it means to keep and not sell your cryptocurrency investments, even during market downturns.
- FUD: Fear, Uncertainty, and Doubt. It refers to negative information or rumors spread to create fear and uncertainty in the cryptocurrency market.
- DYOR: Do Your Own Research. Encourages individuals to research and verify information before making investment decisions.
- DeFi: Decentralized Finance. It refers to financial applications and services built on blockchain, offering traditional financial services without intermediaries.
- NFT: Non-Fungible Token. Unique digital assets that represent ownership of a specific item or piece of content.
- DAO: Decentralized Autonomous Organization. It's an organization governed by smart contracts and the votes of its token holders, operating without a central authority.
- Gas: The fee paid in cryptocurrency to execute transactions or smart contracts on a blockchain platform.
- Rug Pull: A fraudulent action in which developers or project creators drain the funds invested in a token by unsuspecting users.
- Yield Farming: The practice of staking or providing liquidity to DeFi protocols to earn rewards or interest.
- Metaverse: A virtual shared space, often built on blockchain technology, where users can interact with digital assets and each other.
- Whale: A term used to describe individuals or entities that hold a significant amount of cryptocurrency.
- FOMO: Fear Of Missing Out. The feeling of anxiety or urgency to buy or invest due to the fear of missing out on potential gains.
- ATH: All-Time High. Refers to the highest price a cryptocurrency has ever reached.
- Bear Market: A market characterized by declining prices and a pessimistic outlook.
- Bull Market: A market characterized by rising prices and a positive outlook.

## 比特币

在北京时间2008年11月1日，一个化名为中本聪（Satoshi Nakamoto）的神秘密码学极客或者黑客组织发布了比特币白皮书《比特币白皮书：一种点对点的电子现金系统》。

在这比特币白皮书中，中本聪提出来这一新的点对点的电子现金系统，我们也把这种系统称为比特币系统。与此同时，比特币白皮书的诞生标志着区块链的诞生。

在2009年1月3日，在这个伟大的日子里，白皮书的作者中本聪在位于芬兰赫尔辛基的一个小型服务器上，亲手创建了第一个区块——即比特币的创世区块（Genesis Block），并获得了系统自动产生的第一笔50枚比特币的奖励，第一个比特币就此问世。

由于当时正处于08年金融危机，为了纪念比特币的诞生，中本聪将当天的《泰晤士报》头版标题——“The Times 03/Jan/2009 ，Chancellor on brink of second bailout for banks”刻在了第一个区块上。

这句话是泰晤士报当天的头版文章标题，引用这句话，不仅仅是对该区块产生时间的说明，也可以看做是提醒人们一个独立的货币制度的重要性。

创始区块地址：<https://explorer.btc.com/btc/block/0>

- 官网：<https://bitcoin.org/>
- 闪电网络：<http://lightning.network/>
- 比特币浏览器: <https://btc.com/>
- 区块链浏览器: <https://tokenview.io/>

## 以太坊

2015 年 7 月 30 日标志着以太坊区块链的诞生。（前沿阶段）

- 官网：<https://ethereum.org/>
- 区块浏览器: <https://etherscan.io/>
- 信标链浏览器：<https://beaconcha.in/>

## TheGraph

官网：<https://thegraph.com/>

编写智能的合约时，通常状态的变化是通过触发一个事件来表达，The Graph则是捕捉区块链事件并提供一个查询事件的GraphQL接口，让我们可以方便的跟踪数据的变化。 实际上很多 DEFI 协议及都是The Graph来基于查询数据。

因为需要借助 TheGraph 的节点来完成数据的索引，因此我们需要在 thegraph.com 上创建一个Subgraph。

TheGraph中定义如何为数据建立索引，称为Subgraph，它包含三个组件：

- Manifest 清单(subgraph.yaml) - 定义配置项
- Schema 模式(schema.graphql) - 定义数据
- Mapping 映射(mapping.ts) - 定义事件到数据的转换

subgraph.yaml 配置文件通常会定义这些内容：

要索引哪些智能合约(地址，网络，ABI...)、监听哪些事件、其他要监听的内容，例如函数调用或块、被调用的映射函数(mapping.ts)
