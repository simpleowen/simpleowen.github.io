# 常用 Golang 镜像包

由于众所周知的原因，国内无法下载位于谷歌服务器上的包，如 golang.org/x/net, golang.org/x/text , golang.org/x/sys，甚至使用 VPN 的情况下，也无法下载



但这些包基本都有在 github.com 的镜像，go.mod 替换整理如下：

```go
module packageName

go 1.12

replace (
	cloud.google.com/go => github.com/googleapis/google-cloud-go latest
	golang.org/x/crypto => github.com/golang/crypto latest
	golang.org/x/exp => github.com/golang/exp latest
	golang.org/x/image => github.com/golang/image latest
	golang.org/x/lint => github.com/golang/lint latest
	golang.org/x/mobile => github.com/golang/mobile latest
	golang.org/x/net => github.com/golang/net latest
	golang.org/x/oauth2 => github.com/golang/oauth2 latest
	golang.org/x/sync => github.com/golang/sync latest
	golang.org/x/sys => github.com/golang/sys latest
	golang.org/x/text => github.com/golang/text latest
	golang.org/x/time => github.com/golang/time latest
	golang.org/x/tools => github.com/golang/tools latest
	google.golang.org/api => github.com/googleapis/google-api-go-client latest
	google.golang.org/appengine => github.com/golang/appengine latest
	google.golang.org/genproto => github.com/google/go-genproto latest
	google.golang.org/grpc => github.com/grpc/grpc-go latest
)
```

