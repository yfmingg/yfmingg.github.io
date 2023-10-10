# Go环境配置

## 常用命令

- go install // 下载可执行文件
- go mod init <模块名> // 初始化模块文件
- go get xxx // 添加依赖到当前模块并安装它们
- go mod download // 下载依赖到缓存

## GOROOT

GOROOT就是go的安装路径

## GOPATH和GO111MODULE

在Go1.11之前，使用go get下载第三方包,会下载到$GOPATH/src下,会产生如下问题：

- 在不使用额外的工具的情况下，Go的依赖包需要手工下载，
- 第三方包没有版本的概念，如果第三方包的作者做了不兼容升级，会让开发者很难受
- 协作开发时，需要统一各个开发成员本地$GOPATH/src下的依赖包
- 引用的包引用了已经转移的包，而作者没改的话，需要自己修改引用。
- 第三方包和自己的包的源码都在src下，很混乱。对于混合技术栈的项目来说，目录的存放会有一些问题

新的包管理模式使用go mod，解决了以上问题：

- 自动下载依赖包
- 项目不必放在GOPATH/src内了
- 项目内会生成一个go.mod文件，列出包依赖
- 所以来的第三方包会准确的指定版本号
- 对于已经转移的包，可以用replace 申明替换，不需要改代码

go mode使用流程:

1. 初始化模块： go mod init <模块名>
2. 下载 go.mod 文件中指明的所有依赖：go mod download
3. 更多命令使用：go help mod

运行完后，会在当前项目目录下生成一个go.mod 文件，这是一个关键文件，之后的包的管理都是通过这个文件管理。

除了go.mod之外，go命令还维护一个名为go.sum的文件，其中包含特定模块版本内容的预期加密哈希go命令使用go.sum文件确保这些模块的未来下载检索与第一次下载相同的位，以确保项目所依赖的模块不会出现意外更改，无论是出于恶意、意外还是其他原因。 go.mod和go.sum都应检入版本控制。go.sum 不需要手工维护，所以可以不用太关注。

生成出来的文件包含模块名称和当前的go版本号。子目录里是不需要init的，所有的子目录里的依赖都会组织在根目录的go.mod文件里。

问题一：依赖的包下载到哪里了？还在GOPATH里吗？不在。
使用Go的包管理方式，依赖的第三方包被下载到了$GOPATH/pkg/mod路径下。如果包的作者还没有标记版本，默认为 v0.0.0

问题二： 可以把项目放在$GOPATH/src下吗？可以。
但是go会根据GO111MODULE的值而采取不同的处理方式
默认情况下，GO111MODULE=auto 自动模式

- auto 自动模式下，项目在$GOPATH/src里会使用$GOPATH/src的依赖包，在$GOPATH/src外，就使用go.mod 里 require的包
- on 开启模式，1.12后，无论在$GOPATH/src里还是在外面，都会使用go.mod 里 require的包
- off 关闭模式，就是老规矩。

问题三： 依赖包中的地址失效了怎么办？比如 http://golang.org/x/… 下的包都无法下载怎么办？
在go快速发展的过程中，有一些依赖包地址变更了。
在go.mod文件里用 replace 替换包

## GOPROXY

这个环境变量主要是用于设置 Go 模块代理（Go module proxy）,其作用是用于使 Go 在后续拉取模块版本时直接通过镜像站点来快速拉取。

GOPROXY 的默认值是：https://proxy.golang.org,direct

proxy.golang.org国内访问不了,需要设置国内的代理.

- 阿里云：https://mirrors.aliyun.com/goproxy/
- 七牛云：https://goproxy.cn,direct

设置命令：go env -w GOPROXY=https://goproxy.cn,direct

GOPROXY 的值是一个以英文逗号 “,” 分割的 Go 模块代理列表，允许设置多个模块代理，假设你不想使用，也可以将其设置为 “off” ，这将会禁止 Go 在后续操作中使用任何 Go 模块代理。

如:go env -w GOPROXY=https://goproxy.cn,https://mirrors.aliyun.com/goproxy/,direct

而在刚刚设置的值中，我们可以发现值列表中有 “direct” 标识，它又有什么作用呢？

实际上 “direct” 是一个特殊指示符，用于指示 Go 回源到模块版本的源地址去抓取（比如 GitHub 等），场景如下：当值列表中上一个 Go 模块代理返回 404 或 410 错误时，Go 自动尝试列表中的下一个，遇见 “direct” 时回源，也就是回到源地址去抓取，而遇见 EOF 时终止并抛出类似 “invalid version: unknown revision...” 的错误。

## GOSUMDB

它的值是一个 Go checksum database，用于在拉取模块版本时（无论是从源站拉取还是通过 Go module proxy 拉取）保证拉取到的模块版本数据未经过篡改，若发现不一致，也就是可能存在篡改，将会立即中止。

GOSUMDB 的默认值为：sum.golang.org，在国内也是无法访问的，但是 GOSUMDB 可以被 Go 模块代理所代理（详见：Proxying a Checksum Database）。

因此我们可以通过设置 GOPROXY 来解决，而先前我们所设置的模块代理 goproxy.cn 就能支持代理 sum.golang.org，所以这一个问题在设置 GOPROXY 后，你可以不需要过度关心。

## GONOPROXY/GONOSUMDB/GOPRIVATE

这三个环境变量都是用在当前项目依赖了私有模块，例如像是你公司的私有 git 仓库，又或是 github 中的私有库，都是属于私有模块，都是要进行设置的，否则会拉取失败。

更细致来讲，就是依赖了由 GOPROXY 指定的 Go 模块代理或由 GOSUMDB 指定 Go checksum database 都无法访问到的模块时的场景。

而一般建议直接设置 GOPRIVATE，它的值将作为 GONOPROXY 和 GONOSUMDB 的默认值，所以建议的最佳姿势是直接使用 GOPRIVATE。

并且它们的值都是一个以英文逗号 “,” 分割的模块路径前缀，也就是可以设置多个，例如：

$ go env -w GOPRIVATE="git.example.com,github.com/eddycjy/mquote"

设置后，前缀为 git.xxx.com 和 github.com/eddycjy/mquote 的模块都会被认为是私有模块。

如果不想每次都重新设置，我们也可以利用通配符，例如：

$ go env -w GOPRIVATE="*.example.com"

这样子设置的话，所有模块路径为 example.com 的子域名（例如：git.example.com）都将不经过 Go module proxy 和 Go checksum database，需要注意的是不包括 example.com 本身。

## go.mod文件

- module: 用于定义当前项目的模块路径
- go:标识当前Go版本.即初始化版本
- require: 当前项目依赖的一个特定的必须版本

- // indirect: 示该模块为间接依赖，也就是在当前应用程序中的 import 语句中，并没有发现这个模块的明确引用，有可能是你先手动 go get 拉取下来的，也有可能是你所依赖的模块所依赖的.我们的代码很明显是依赖的"github.com/aceld/zinx/znet"和"github.com/aceld/zinx/ziface",所以就间接的依赖了github.com/aceld/zinx

## go.sum文件

查看go.sum文件在第一次拉取模块依赖后，会发现多出了一个 go.sum 文件，其详细罗列了当前项目直接或间接依赖的所有模块版本，并写明了那些模块版本的 SHA-256 哈希值以备 Go 在今后的操作中保证项目所依赖的那些模块版本不会被篡改。

h1 hash 是 Go modules 将目标模块版本的 zip 文件开包后，针对所有包内文件依次进行 hash，然后再把它们的 hash 结果按照固定格式和算法组成总的 hash 值。

而 h1 hash 和 go.mod hash 两者，要不就是同时存在，要不就是只存在 go.mod hash。那什么情况下会不存在 h1 hash 呢，就是当 Go 认为肯定用不到某个模块版本的时候就会省略它的 h1 hash，就会出现不存在 h1 hash，只存在 go.mod hash 的情况。


## 参考
- https://www.jianshu.com/p/2d4d0bd7d2e4