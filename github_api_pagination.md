# Golang 获取 Github API 分页总页数

Github API 会自动对所请求的项目进行分页

不同的 API 会有不同的每页数目，有的 API 默认每页 100 项，有的 API 默认每页 30 项，可以通过 per_page=xxx 参数改变每页展示项目数，但最大值为 100。但是也有例外，比如，[Events](https://developer.github.com/v3/activity/events/) 就不会让你设置最大每页数



调用 Github API 时会返回在一个响应头` Link header` ， 开发者可通过解析 `Link header` 来获取分页信息



## Link header 样式

默认请求，github api 返回的 Link header 如下：

```
Link: <https://api.github.com/search/code?q=addClass+user%3Amozilla&page=2>; rel="next",
  <https://api.github.com/search/code?q=addClass+user%3Amozilla&page=34>; rel="last"
```

请求传入 page=xxx 参数，返回的 Link header 如下：

```
Link: <https://api.github.com/search/code?q=addClass+user%3Amozilla&page=15>; rel="next",
  <https://api.github.com/search/code?q=addClass+user%3Amozilla&page=34>; rel="last",
  <https://api.github.com/search/code?q=addClass+user%3Amozilla&page=1>; rel="first",
  <https://api.github.com/search/code?q=addClass+user%3Amozilla&page=13>; rel="prev"
```





## 解析 Link Header 总页数

- 方案一
  - 分解字符串
    - 以 `逗号` 为分隔符，分解出每条 link
    - 以 `分号` 为分割符，分解带 `rel="last"` 的链接为链接部分和链接类型部分
    - 以 `page=` 为分割符，分解出链接前部和 `页数>` 部分 
    - 去除 `>` 后得总页数

- 方案二
  - 正则匹配
    - 设置正则表达式 `page=(\d+)>; rel="last"`
    - 匹配整个 Link header 字符串
    - 找出总页数

```go
package main

import (
	"fmt"
	"log"
	"regexp"
	"strconv"
	"strings"
)

var linkHeader = `<https://api.github.com/search/code?q=addClass+user%3Amozilla&page=15>; rel="next",
			<https://api.github.com/search/code?q=addClass+user%3Amozilla&page=34>; rel="last",
			<https://api.github.com/search/code?q=addClass+user%3Amozilla&page=1>; rel="first",
			<https://api.github.com/search/code?q=addClass+user%3Amozilla&page=13>; rel="prev"`

func split(linkHeader string) (int, error) {
	var link string
	links := strings.Split(linkHeader, ",")
	for _, v := range links {
		v = strings.TrimSpace(v)
		if strings.HasSuffix(v, `rel="last"`) {
			link = strings.Split(v, ";")[0]
		}
	}
	// fmt.Println(link)
	totalPageNum, err := strconv.Atoi(strings.TrimSuffix(strings.Split(link, "page=")[1], ">"))
	if err != nil {
		log.Println(err)
		return 0, err
	}
	return totalPageNum, nil

}

func regex(linkHeader string) (int, error) {

	rege := regexp.MustCompile(`page=(\d+)>; rel="last"`)
	// fmt.Println(rege.MatchString(linkHeader))
	totalPages := rege.FindStringSubmatch(linkHeader)[1]

	totalPageNum, err := strconv.Atoi(totalPages)
	if err != nil {
		log.Println("Error: ", err)
		return 0, err
	}
	// fmt.Println(totalPageNum)
	return totalPageNum, nil
}

func main() {
  //方案一
	numberOfPages, err := regex(linkHeader)
	if err != nil {
		log.Println(err)
		return
	}
	fmt.Println(numberOfPages)
  
  //方案二
	numberOfPages, err = split(linkHeader)
	if err != nil {
		log.Println(err)
		return
	}
	fmt.Println(numberOfPages)
}

```





## Refs

- [Traversing with Pagination | GitHub Developer Guide](https://developer.github.com/v3/guides/traversing-with-pagination/)
- [platform-samples/api/ruby/traversing-with-pagination at master · github/platform-samples](https://github.com/github/platform-samples/tree/master/api/ruby/traversing-with-pagination)
- [RFC 5988 - Web Linking](https://tools.ietf.org/html/rfc5988#section-5)
- [Regular expression - Wikipedia](https://en.wikipedia.org/wiki/Regular_expression)
- [Go语言正则表达式提取网页文本 - 蝈蝈俊 - 博客园](https://www.cnblogs.com/ghj1976/archive/2013/03/21/2972522.html)