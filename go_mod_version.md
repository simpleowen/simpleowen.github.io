



golang 1.11 以上版本引入了依赖包管理



- 如何使用此新功能
- 原来的 go get 还有吗
- gopath 目录呢
- go111module 环境变量到底需不需要



## 激活模块支持

安装成功后，可通过以下两种方式激活模块支持

- 想在 `$GOPATH/src` 目录内使用模块支持，需要设置 `GO111MODULE=on` 环境变量
- 而在 `$GOPATH/src` 目录外则不一定需要设置 `GO111MODULE=on` 环境变量，在项目目录中使用 `git mod init packageName` 命令来生成 go.mod 文件即可





## 版本号管理







golang 项目不需要放在 $GOPATH 目录下了，可以在任意位置



使用 命令 go mod init 就可以将此目录纳入到 go modules 管理下

新手最头痛的是 go.mod 文件中版本号，特别是墙内开发者，有些包 golang.org/x/text, golang.org/x/net 等包无法下载，即使使用 VPN，这样势必需要手工在 go.mod 文件下添加替代依赖

go module 使用 replace 关键字，应对此情况

replace 可将某些无法下载的依赖替换为国内镜像

然，镜像包的版本号也需要手工输入

新手对此很头痛

go module 使用语义化版本标准来描述依赖包的版本号











- [Modules · golang/go Wiki](https://github.com/golang/go/wiki/Modules#modules)

- [go module 来了 | wudaijun's blog](http://wudaijun.com/2018/10/go-mod-intro/)
- [语义化版本 2.0.0 | Semantic Versioning](https://semver.org/lang/zh-CN/)