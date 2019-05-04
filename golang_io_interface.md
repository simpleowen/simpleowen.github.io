## 原文链接

- [Streaming IO in Go – Learning the Go Programming Language – Medium](https://link.zhihu.com/?target=https%3A//medium.com/learning-the-go-programming-language/streaming-io-in-go-d93507931185)

## 以下为译文

在 Go 中，输入输出操作是通过能读能写的字节流数据模型来实现的。为此，io 包提供了 io.Reader 和 io.Writer 接口来进行输入输出操作，如下所示：

![img](https://pic2.zhimg.com/80/v2-da53523e772285dd79ee560b64023025_hd.png)

Go 附带了许多 API，这些 API 支持来自内存结构，文件，网络连接等资源的流式 IO。本文重点介绍如何自定义实现以及使用标准库中的 io.Reader 和 io.Writer接口创建能够传输流式数据的 Go 程序

## io.Reader

由 io.Reader 接口表示的读取器将数据从某些源读取到缓冲区，可以像用水管输送水流一样来传送它，如下所示

![img](https://pic2.zhimg.com/80/v2-fca7cc836b6dbfcb9f037c90c91266c9_hd.jpg)

对于要用作读取器的类型，它必须从接口 io.Reader 实现 Read(p [] byte)方法，如下所示：

```go
type Reader interface {
    Read(p []byte) (n int, err error)
}
```

Read() 方法的实现应返回读取的字节数或发生的错误。如果数据源已输出全部内容，则 Read 应返回 io.EOF

### 读取规则(补充)

在 Reddit 反馈之后，我决定添加有关读取规则的这一部分。读取器的行为取决于它的实现，但是你应该知道从读取器读取数据时， io.Reader 中的一些规则：

> 译者注：p 为缓冲区，n 为字节数

1. 如果可能，Read() 将读取 len(p) 到 p
2. 调用 Read() 后，返回的字节数 n 可能小于 len(p)
3. 出错时，Read() 仍可在缓冲区 p 中返回 n 个字节。例如，从突然关闭的 TCP 套接字读取。取决于您的程序设计，您可以选择将字节保存在 p 中或重新尝试从 TCP 套接字中读取
4. 当 Read() 读完所有可用数据时，读取器可能返回非零 n 和 err = io.EOF。尽管如此，您可以自己实现返回规则，如可以选择在流的末尾返回非零 n 和 err = nil。在这种情况下，任何后续读取必须返回 n = 0，err = io.EOF
5. 最后，调用 Read() 返回 n = 0 和 err = nil 并不意味着 EOF，因为下一次调用 Read() 可能会返回更多数据

如您所见，直接从读取器读取流数据可能会非常棘手。幸运的是，标准库中的读取器使用的一些方法使其易于流式传输。不过，在使用读取器之前，请查阅其文档

### 从读取器中流式传输数据

直接从读取器流式传输数据很容易。Read 方法被设计为在循环内调用，每次迭代时，它从源读取一大块数据并将其放入缓冲区 p 中。直到 Read 方法返回io.EOF 错误

以下是一个简单的[示例](https://link.zhihu.com/?target=https%3A//github.com/vladimirvivien/learning-go/blob/master/tutorial/io/simple_reader.go)，它使用 string.NewReader(string) 创建的字符串读取器来从字符串源中流式传输字节值:

```go
func main() {
	reader := strings.NewReader("Clear is better than clever")
	p := make([]byte, 4)
	for {
		n, err := reader.Read(p)
		if err == io.EOF {
			break
		}
		fmt.Println(string(p[:n]))
	}
}
```

上面的源代码用 make([] byte，4) 创建一个 4 字节长的传输缓冲区 p。缓冲区故意保持小于字符串源的长度, 这是为了演示如何从大于缓冲区的源正确传输数据块

**更新**: Reddit 上有人指出上面的代码中有 bug, 它永远不会捕获非零错误 err != io.EOF . 以下修复了代码:

```go
func main() {
	reader := strings.NewReader("Clear is better than clever")
	p := make([]byte, 4)
	
	for {
		n, err := reader.Read(p)
		if err != nil{
		    if err == io.EOF {
			fmt.Println(string(p[:n])) //should handle any remainding bytes.
			break
		    }
		    fmt.Println(err)
		    os.Exit(1)
		}
		fmt.Println(string(p[:n]))
	}
}
```

### 自定义一个 io.Reader

上一节使用标准库中的现有 IO 读取器实现。现在，让我们看看如何编写自己的读取器。以下是 io.Reader 的[简单实现](https://link.zhihu.com/?target=https%3A//github.com/vladimirvivien/learning-go/blob/master/tutorial/io/alpha_reader.go)，它从流中过滤掉非字母字符。

```go
package main

import (
    "fmt"
    "io"
)

// alphaReader is a simple implementation of an io.Reader
// that streams only alpha chars from its string source.
type alphaReader struct {
    src string
    cur int
}

func newAlphaReader(src string) *alphaReader {
    return &alphaReader{src: src}
}

func alpha(r byte) byte {
    if (r >= 'A' && r <= 'Z') || (r >= 'a' && r <= 'z') {
        return r
    }
    return 0
}

func (a *alphaReader) Read(p []byte) (int, error) {
    if a.cur >= len(a.src) {
        return 0, io.EOF
    }

    x := len(a.src) - a.cur
    n, bound := 0, 0
    if x >= len(p) {
        bound = len(p)
    } else if x <= len(p) {
        bound = x
    }

    buf := make([]byte, bound)
    for n < bound {
        if char := alpha(a.src[a.cur]); char != 0 {
            buf[n] = char
        }
        n++
        a.cur++
    }
    copy(p, buf)
    return n, nil
}

func main() {
    reader := newAlphaReader("Hello! It's 9am, where is the sun?")
    p := make([]byte, 4)
    for {
        n, err := reader.Read(p)
        if err == io.EOF {
            break
        }
        fmt.Print(string(p[:n]))
    }
    // or use io.Copy
    // io.Copy(os.Stdout, reader)
    fmt.Println()
}
```

程序执行时，输出:

```text
$> go run alpha_reader.go
HelloItsamwhereisthesun
```

### 链式读取器

标准库已经实现了许多读取器。使用读取器作为另一个读取器的源是一种常见的习语。读取器的这种链接允许一个读取器重用另一个读取器的逻辑，就像在下面的[源代码](https://link.zhihu.com/?target=https%3A//github.com/vladimirvivien/learning-go/blob/master/tutorial/io/alpha_reader2.go)片段中所做的那样，更新 alphaReader 以接受 io.Reader 作为其源。这通过将流管理问题推向根读取器来降低代码的复杂性。

```go
package main

import (
    "fmt"
    "io"
    "strings"
)

// alphaReader is a simple implementation of an io.Reader
// that streams only alpha chars from its string source.
// This example uses another reader as data source.
type alphaReader struct {
    reader io.Reader
}

func newAlphaReader(reader io.Reader) *alphaReader {
    return &alphaReader{reader: reader}
}

func alpha(r byte) byte {
    if (r >= 'A' && r <= 'Z') || (r >= 'a' && r <= 'z') {
        return r
    }
    return 0
}

func (a *alphaReader) Read(p []byte) (int, error) {
    n, err := a.reader.Read(p)
    if err != nil {
        return n, err
    }
    buf := make([]byte, n)
    for i := 0; i < n; i++ {
        if char := alpha(p[i]); char != 0 {
            buf[i] = char
        }
    }

    copy(p, buf)
    return n, nil
}

func main() {
    // use an io.Reader as source for alphaReader
    reader := newAlphaReader(strings.NewReader("Hello! It's 9am, where is the sun?"))
    p := make([]byte, 4)
    for {
        n, err := reader.Read(p)
        if err == io.EOF {
            break
        }
        fmt.Print(string(p[:n]))
    }
    fmt.Println()
}
```

这种方法的另一个优点是 alphaReader 现在能够从任何读取器实现中读取。例如，[以下代码段](https://link.zhihu.com/?target=https%3A//github.com/vladimirvivien/learning-go/blob/master/tutorial/io/alpha_reader3.go)显示了如何将 alphaReader 与 os.File 源结合以过滤掉文件中的非字母字符：

```go
package main

import (
    "fmt"
    "io"
    "os"
)

// alphaReader is a simple implementation of an io.Reader
// that streams only alpha chars from its string source.
// This example uses another reader as data source.
type alphaReader struct {
    reader io.Reader
}

func newAlphaReader(reader io.Reader) *alphaReader {
    return &alphaReader{reader: reader}
}

func alpha(r byte) byte {
    if (r >= 'A' && r <= 'Z') || (r >= 'a' && r <= 'z') {
        return r
    }
    return 0
}

func (a *alphaReader) Read(p []byte) (int, error) {
    n, err := a.reader.Read(p)
    if err != nil {
        return n, err
    }
    buf := make([]byte, n)
    for i := 0; i < n; i++ {
        if char := alpha(p[i]); char != 0 {
            buf[i] = char
        }
    }

    copy(p, buf)
    return n, nil
}

func main() {
    // use an io.Reader as source for alphaReader
    file, err := os.Open("./alpha_reader2.go")
    if err != nil {
        fmt.Println(err)
        os.Exit(1)
    }
    defer file.Close()

    reader := newAlphaReader(file)
    p := make([]byte, 4)
    for {
        n, err := reader.Read(p)
        if err == io.EOF {
            break
        }
        fmt.Print(string(p[:n]))
    }
    fmt.Println()
}
```

## io.Writer

由接口 io.Writer 表示的写入器从缓冲区流式传输数据并将其写入目标资源，如下所示

![img](https://pic2.zhimg.com/80/v2-01afe72161975293a1e34803be4029e1_hd.jpg)

所有流写入器必须从接口 io.Writer 实现方法 Write(p [] byte)。该方法旨在从缓冲区 p 读取数据并将其写入指定的目标资源

```go
type Writer interface {
  Write(p []byte) (n int, err error)
}
```

Write() 方法的实现应返回写入的字节数或发生的错误

### 使用写入器

标准库附带了许多预先实现的 io.Writer 类型。直接使用写入器很简单，如下面的[代码片段](https://link.zhihu.com/?target=https%3A//github.com/vladimirvivien/learning-go/blob/master/tutorial/io/using_writer.go)所示，它使用 bytes.Buffer 作为 io.Writer 将数据写入内存缓冲区

```go
package main

import (
    "bytes"
    "fmt"
    "os"
)

func main() {
    proverbs := []string{
        "Channels orchestrate mutexes serialize",
        "Cgo is not Go",
        "Errors are values",
        "Don't panic",
    }
    var writer bytes.Buffer

    for _, p := range proverbs {
        n, err := writer.Write([]byte(p))
        if err != nil {
            fmt.Println(err)
            os.Exit(1)
        }
        if n != len(p) {
            fmt.Println("failed to write data")
            os.Exit(1)
        }
    }

    fmt.Println(writer.String())
}
```

### 自定义一个 io.Writer

本节中的[代码](https://link.zhihu.com/?target=https%3A//github.com/vladimirvivien/learning-go/blob/master/tutorial/io/chan_writer.go)显示了如何实现一个名为 chanWriter 的自定义 io.Writer，它将其内容作为字节序列写入 Go 通道。

```go
package main

import "fmt"

type chanWriter struct {
    ch chan byte
}

func newChanWriter() *chanWriter {
    return &chanWriter{make(chan byte, 1024)}
}

func (w *chanWriter) Chan() <-chan byte {
    return w.ch
}

func (w *chanWriter) Write(p []byte) (int, error) {
    n := 0
    for _, b := range p {
        w.ch <- b
        n++
    }
    return n, nil
}

func (w *chanWriter) Close() error {
    close(w.ch)
    return nil
}

func main() {
    writer := newChanWriter()
    go func() {
        defer writer.Close()
        writer.Write([]byte("Stream "))
        writer.Write([]byte("me!"))
    }()
    for c := range writer.Chan() {
        fmt.Printf("%c", c)
    }
    fmt.Println()
}
```

要使用写入器，代码只需在函数 main() 中调用方法 writer.Write() (在单独的goroutine 中）。因为 chanWriter 还实现了接口 io.Closer，所以调用方法writer.Close() 来正确关闭通道，以避免在访问通道时出现死锁

## Useful types and packages for IO

如前所述，Go 标准库附带了许多有用的功能和其他类型，可以轻松使用流式IO

### os.File

os.File 类型表示本地系统上的文件。它实现了 io.Reader 和 io.Writer，因此可以在任何流 IO 上下文中使用。例如，以下[示例](https://link.zhihu.com/?target=https%3A//github.com/vladimirvivien/learning-go/blob/master/tutorial/io/file_write.go)显示如何将连续的字符串切片直接写入文件

```go
package main

import (
    "fmt"
    "os"
)

func main() {
    proverbs := []string{
        "Channels orchestrate mutexes serialize\n",
        "Cgo is not Go\n",
        "Errors are values\n",
        "Don't panic\n",
    }
    file, err := os.Create("./proverbs.txt")
    if err != nil {
        fmt.Println(err)
        os.Exit(1)
    }
    defer file.Close()

    for _, p := range proverbs {
        n, err := file.Write([]byte(p))
        if err != nil {
            fmt.Println(err)
            os.Exit(1)
        }
        if n != len(p) {
            fmt.Println("failed to write data")
            os.Exit(1)
        }
    }
    fmt.Println("file write done")
}
```

相反，io.File 类型可以用作读取器来从本地文件系统流式传输文件的内容。例如，以下[源代码段](https://link.zhihu.com/?target=https%3A//github.com/vladimirvivien/learning-go/blob/master/tutorial/io/file_read.go)读取文件并打印其内容：

```go
package main

import (
    "fmt"
    "io"
    "os"
)

func main() {
    file, err := os.Open("./proverbs.txt")
    if err != nil {
        fmt.Println(err)
        os.Exit(1)
    }
    defer file.Close()

    p := make([]byte, 4)
    for {
        n, err := file.Read(p)
        if err == io.EOF {
            break
        }
        fmt.Print(string(p[:n]))
    }
}
```

### Standard output, input, and error

os 包公开三个变量，os.Stdout，os.Stdin 和 os.Stderr，它们的类型为* os.File，分别表示操作系统标准输出\输入\错误的文件句柄。例如，以下[源代码段](https://link.zhihu.com/?target=https%3A//github.com/vladimirvivien/learning-go/blob/master/tutorial/io/stdout_write.go)直接打印到标准输出：

```go
package main

import (
    "fmt"
    "os"
)

func main() {
    proverbs := []string{
        "Channels orchestrate mutexes serialize\n",
        "Cgo is not Go\n",
        "Errors are values\n",
        "Don't panic\n",
    }

    for _, p := range proverbs {
        n, err := os.Stdout.Write([]byte(p))
        if err != nil {
            fmt.Println(err)
            os.Exit(1)
        }
        if n != len(p) {
            fmt.Println("failed to write data")
            os.Exit(1)
        }
    }
}
```

### io.Copy()

io.Copy() 方法可以轻松地将数据从源读取器传输到目标写入器。它抽象出 for 循环模式（我们到目前为止已经看到）并正确处理 io.EOF 和字节计数。

以下显示了以前程序的[简化版本](https://link.zhihu.com/?target=https%3A//github.com/vladimirvivien/learning-go/blob/master/tutorial/io/io_copy.go)，该程序复制内存读取器 proberbs 的内容并将其复制到 writer 文件：

```go
package main

import (
    "bytes"
    "fmt"
    "io"
    "os"
)

func main() {
    proverbs := new(bytes.Buffer)
    proverbs.WriteString("Channels orchestrate mutexes serialize\n")
    proverbs.WriteString("Cgo is not Go\n")
    proverbs.WriteString("Errors are values\n")
    proverbs.WriteString("Don't panic\n")

    file, err := os.Create("./proverbs.txt")
    if err != nil {
        fmt.Println(err)
        os.Exit(1)
    }
    defer file.Close()

    // copy from reader data into writer file
    if _, err := io.Copy(file, proverbs); err != nil {
        fmt.Println(err)
        os.Exit(1)
    }
    fmt.Println("file created")
}
```

同样，我们可以使用 io.Copy() 函数重写以前从文件读取并打印到标准输出的程序，[如下](https://link.zhihu.com/?target=https%3A//github.com/vladimirvivien/learning-go/blob/master/tutorial/io/io_copy2.go)所示

```go
package main

import (
    "fmt"
    "io"
    "os"
)

func main() {
    file, err := os.Open("./proverbs.txt")
    if err != nil {
        fmt.Println(err)
        os.Exit(1)
    }
    defer file.Close()

    if _, err := io.Copy(os.Stdout, file); err != nil {
        fmt.Println(err)
        os.Exit(1)
    }
}
```

### io.WriterString()

此函数提供了将字符串值写入指定写入器的便利，[如下](https://link.zhihu.com/?target=https%3A//github.com/vladimirvivien/learning-go/blob/master/tutorial/io/write_str.go)所示

```go
package main

import (
    "fmt"
    "io"
    "os"
)

func main() {
    file, err := os.Create("./magic_msg.txt")
    if err != nil {
        fmt.Println(err)
        os.Exit(1)
    }
    defer file.Close()
    if _, err := io.WriteString(file, "Go is fun!"); err != nil {
        fmt.Println(err)
        os.Exit(1)
    }
}
```

### Pipe writers and readers

io.PipeWriter 类型和 io.PipeReader 模型 IO 操作在内存管道中。数据被写入管道的 writer-end，并使用单独的 go 例程在管道的 reader-end 上读取。下面使用 io.Pipe() 创建管道读取器/写入器对，然后使用 io.Pipe() 将数据从缓冲区 proverbs 复制到 io.Stdout, [如下](https://link.zhihu.com/?target=https%3A//github.com/vladimirvivien/learning-go/blob/master/tutorial/io/io_pipe.go)所示

```go
package main

import (
    "bytes"
    "io"
    "os"
)

func main() {
    proverbs := new(bytes.Buffer)
    proverbs.WriteString("Channels orchestrate mutexes serialize\n")
    proverbs.WriteString("Cgo is not Go\n")
    proverbs.WriteString("Errors are values\n")
    proverbs.WriteString("Don't panic\n")

    piper, pipew := io.Pipe()

    // write in writer end of pipe
    go func() {
        defer pipew.Close()
        io.Copy(pipew, proverbs)
    }()

    // read from reader end of pipe.
    io.Copy(os.Stdout, piper)
    piper.Close()
}
```

### Buffered IO

Go 通过 bufio 包支持缓冲 IO，可以轻松处理文本内容。例如，[以下](https://link.zhihu.com/?target=https%3A//github.com/vladimirvivien/learning-go/blob/master/tutorial/io/bufread.go)程序逐行读取文件以值 '\ n' 分隔的内容

```go
package main

import (
    "bufio"
    "fmt"
    "io"
    "os"
)

func main() {
    file, err := os.Open("./planets.txt")
    if err != nil {
        fmt.Println(err)
        os.Exit(1)
    }
    defer file.Close()
    reader := bufio.NewReader(file)

    for {
        line, err := reader.ReadString('\n')
        if err != nil {
            if err == io.EOF {
                break
            } else {
                fmt.Println(err)
                os.Exit(1)
            }
        }
        fmt.Print(line)
    }

}
```

### Util package

ioutil 包为 IO 提供了几个便利功能。例如，[以下](https://link.zhihu.com/?target=https%3A//github.com/vladimirvivien/learning-go/blob/master/tutorial/io/io_util.go)使用函数 ReadFile 将文件内容加载到[]字节中

```go
package main

import (
    "fmt"
    "io/ioutil"
    "os"
)

func main() {
    bytes, err := ioutil.ReadFile("./planets.txt")
    if err != nil {
        fmt.Println(err)
        os.Exit(1)
    }
    fmt.Printf("%s", bytes)
}
```

## 结论

本文介绍如何使用 io.Reader 和 io.Writer 接口在程序中实现流式 IO。阅读本文后，您应该能够了解如何创建使用 io 包流式传输 IO 数据的程序，有很多示例向您展示了如何为自定义功能创建自己的 io.Reader 和 io.Writer 类型。

这是一个介绍性的讨论，几乎没有涉及支持流 IO 的 Go 包的范围。例如，我们没有进入文件 IO，缓冲 IO，网络 IO或格式化 IO（为将来的写作而保留）。我希望这能让你了解 Go 的流式 IO 惯用语是什么