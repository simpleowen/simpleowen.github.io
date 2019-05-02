# 将 Golang 程序部署到 Heroku

- 环境
  - macOS
  - git
  - Golang 代码

```go
// main.go

package main

import (
	"log"
	"net/http"
	"os"
)

func hello(w http.ResponseWriter, r *http.Request) {
	w.Write([]byte("hello.world"))
}

func main() {
	http.HandleFunc("/hello", hello)
  //err := http.ListenAndServe(":8080", nil) 
	err := http.ListenAndServe(":" + os.Getenv("PORT"), nil) 
	if err != nil {
		log.Fatal(err)
	}
}
```



- 前提
  - 创建一个 heroku 账户
  - 安装 heroku 命令行工具
  - 安装 heroku 命令行
    - 运行 `brew install heroku/brew/heroku` 命令
  - 验证安装
    - 运行 `heroku` 命令



- 步骤

  - 命令行登录

    - 运行 `heroku login` 命令

      ```
      heroku: Press any key to open up the browser to login or q to exit:
      Opening browser to https://cli-auth.heroku.com/auth/browser/865fec7b-8b18-4080-82c8-1fe695b1322c
      Logging in... done
      Logged in as xxx@yyyy.com
      ```

  - 将 golang 代码所在目录初始化为 git 仓库，并提交内容

    - git init
    - git add
    - git commit

  - 运行 `heroku create` 命令，创建 heroku app ,并与远程 heroku 仓库建立链接

  - 推送本地内容到远程 heroku 仓库

    - git push heroku master

  - 部署完成，[https://evening-tundra-19792.herokuapp.com/hello](https://evening-tundra-19792.herokuapp.com/hello)

    

- 坑

  ```
  State changed from starting to crashed
  State changed from crashed to starting
  Error R10 (Boot timeout) -> Web process failed to bind to $PORT within 60 seconds of launch
  ```

  - heroku 不需要你绑定固定的端口号, 它会自动绑定 $PORT 环境变量
  - 可查看本文开头的代码



- refs
  - [Getting Started on Heroku with Go | Heroku Dev Center](https://devcenter.heroku.com/articles/getting-started-with-go)
  - [Preparing a Codebase for Heroku Deployment | Heroku Dev Center](https://devcenter.heroku.com/articles/preparing-a-codebase-for-heroku-deployment)
  - [Deploying with Git | Heroku Dev Center](https://devcenter.heroku.com/articles/git#creating-a-heroku-remote)
  - [Heroku Error Codes | Heroku Dev Center](https://devcenter.heroku.com/articles/error-codes)
  - [Heroku Go app crashing - Stack Overflow](https://stackoverflow.com/questions/32463004/heroku-go-app-crashing)