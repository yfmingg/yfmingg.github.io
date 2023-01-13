# TheGraph概览

官网：<https://thegraph.com/>

编写智能的合约时，通常状态的变化是通过触发一个事件来表达，The Graph则是捕捉区块链事件并提供一个查询事件的GraphQL接口，让我们可以方便的跟踪数据的变化。 实际上很多 DEFI 协议及都是The Graph来基于查询数据。

因为需要借助 TheGraph 的节点来完成数据的索引，因此我们需要在 thegraph.com 上创建一个Subgraph。

TheGraph中定义如何为数据建立索引，称为Subgraph，它包含三个组件：

- Manifest 清单(subgraph.yaml) - 定义配置项
- Schema 模式(schema.graphql) - 定义数据
- Mapping 映射(mapping.ts) - 定义事件到数据的转换

subgraph.yaml 配置文件通常会定义这些内容：

要索引哪些智能合约(地址，网络，ABI...)
监听哪些事件
其他要监听的内容，例如函数调用或块
被调用的映射函数(mapping.ts)
