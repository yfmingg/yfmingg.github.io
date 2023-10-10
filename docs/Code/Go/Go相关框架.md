# GoWeb常用工具

## Protocol Buffers

- https://protobuf.dev/

Protocol Buffers 是一种与语言无关、与平台无关的可扩展机制，用于序列化结构化数据。

Protocol Buffers 是一种定义语言（.proto文件），通过编译器生成的用于与数据、特定于语言的运行时库交互的代码，以及写入文件（或通过网络连接发送）的数据的序列化格式，的组合。

它最常用于定义通信协议（与 gRPC 一起）和数据存储。


## Wire

- Github: https://github.com/google/wire

Wire 是 Go 的编译时依赖注入，有两个基本概念：提供者和注入者。

- 提供者是普通的 Go 函数，函数的参数是它的依赖项，经常一起使用的提供者可以形成ProviderSet，如 dbConfig一般只会与 repo 一起使用  
- 注入器是生成的函数，它们按依赖顺序调用提供者。您编写注入器的签名，包括任何所需的输入作为参数，并插入wire.Build对构建最终结果所需的提供程序或提供程序集列表的调用

通过 go generate 可以生成注入器代码，运行时不依赖 Wire，所生成的代码都是普通的 Go 代码，故编译时依赖注入

## gorm

- https://gorm.io/

## gRPC

## gopls

微软在开发 VS Code 过程中, 定义一种协议, 语言服务器协议 Language Server Protocol, 这可是个好东西, 如果你需要开发编辑器或 IDE, 就不需要再为每种语言实现诸如自动完成, 代码提示等功能了, 直接利用, gopls 就是GO官方的语言服务器.

https://microsoft.github.io/language-server-protocol/