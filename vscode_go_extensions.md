# VScode 安装 go 插件

在 VScode 使用过程中，会提示安装 各种 go 插件，点击安装后，很多插件安装都会失败

其中，安装失败的包主要是 golang.org 上的包(由于众所周知的原因)，而很多其他包又依赖这些包，所以 VScode 的 golang 环境用起来就不顺畅了，如没有自动提示，没有自动格式化

vscode 官方提供了一个 go extension 依赖列表 -> [Go tools that the Go extension depends on · Microsoft/vscode-go Wiki](https://github.com/Microsoft/vscode-go/wiki/Go-tools-that-the-Go-extension-depends-on)

```shell
go get -u -v github.com/ramya-rao-a/go-outline
go get -u -v github.com/acroca/go-symbols
go get -u -v github.com/mdempsky/gocode
go get -u -v github.com/rogpeppe/godef
go get -u -v golang.org/x/tools/cmd/godoc
go get -u -v github.com/zmb3/gogetdoc
go get -u -v golang.org/x/lint/golint
go get -u -v github.com/fatih/gomodifytags
go get -u -v golang.org/x/tools/cmd/gorename
go get -u -v sourcegraph.com/sqs/goreturns
go get -u -v golang.org/x/tools/cmd/goimports
go get -u -v github.com/cweill/gotests/...
go get -u -v golang.org/x/tools/cmd/guru
go get -u -v github.com/josharian/impl
go get -u -v github.com/haya14busa/goplay/cmd/goplay
go get -u -v github.com/uudashr/gopkgs/cmd/gopkgs
go get -u -v github.com/davidrjenni/reftools/cmd/fillstruct
go get -u -v github.com/alecthomas/gometalinter
gometalinter --install
```

将这些脚本拷贝到命令行执行，部分 golang.org 下的包是无法成功 get 的

将上述依赖列表分成两部分，一部分是  `golang.org` 下的包，一部分是 `非  golang.org` 下的包

对于 `非  golang.org` 下的包，可通过以下脚本处理

```
go get -u -v github.com/ramya-rao-a/go-outline
go get -u -v github.com/acroca/go-symbols
go get -u -v github.com/mdempsky/gocode
go get -u -v github.com/rogpeppe/godef
go get -u -v github.com/zmb3/gogetdoc
go get -u -v github.com/fatih/gomodifytags
go get -u -v sourcegraph.com/sqs/goreturns
go get -u -v github.com/cweill/gotests/...
go get -u -v github.com/josharian/impl
go get -u -v github.com/haya14busa/goplay/cmd/goplay
go get -u -v github.com/uudashr/gopkgs/cmd/gopkgs
go get -u -v github.com/davidrjenni/reftools/cmd/fillstruct
go get -u -v github.com/alecthomas/gometalinter
```



对于 ` golang.org` 下的包，则需要进行特殊处理：克隆 github.com 上的镜像包到固定目录

进入 `$GOPATH/src/golang.org/x` , 若无此目录，新建即可。将以下脚本内拷贝到命令行执行

```shell
git clone git@github.com:golang/lint.git
git clone git@github.com:golang/tools.git
git clone git@github.com:golang/appengine.git
git clone git@github.com:golang/time.git
git clone git@github.com:golang/net.git
git clone git@github.com:golang/text.git
git clone git@github.com:golang/image.git
git clone git@github.com:golang/sys.git
git clone git@github.com:golang/sync.git
git clone git@github.com:golang/mobile.git
git clone git@github.com:golang/exp.git
git clone git@github.com:golang/crypto.git
git clone git@github.com:golang/oauth2.git
```



经过上述两类处理后，VScode 的 golang 环境就算比较好用了。