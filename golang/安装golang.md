### 1 在自己的工作目录下建立一个goproject文件夹

/home/go

### 2 在文件夹下建立如下三个文件

bin pkg src
bin 保存执行go install 源码目录后生成的可执行文件
pkg 文件夹是存在go编译生成的文件
src存放的是我们的go源代码，不同工程项目的代码以包名区分

### 3 安装go

去国内的网站下载https://studygolang.com/dl 根据不同的平台选择对应的安装包
然后将压缩包解压到 /usr/local/
sudo tar -C /usr/local/ -xzvf goxxxxx.linux-amd64.tar.gz

### 4 配置环境变量

可以配置`/etc/profile`对所有用户起作用 ，也可以配置`~/.profile`只对当前用户起作用
打开后，添加如下代码

```c++
export GOROOT=/usr/local/go
export GOPATH=$HOME/go
export PATH=$PATH:$GOROOT/bin:$GOPATH/bin
```

保存后执行

```bash
 source ~/.profile #刷新下配置
```

远程可能需要重新打开，执行go version 可以看到配置好了go版本

简单说下上述配置
GOROOT表示你golang的安装目录
GOPATH 表示 你工程的路径
GOBIN目录是执行 go install 后生成可执行文件的目录
然后将两个路径导入到PATH中。

### 5 编写代码测试

在go/tmp下src文件夹里，创建两个文件夹，分别是calcsrc和simplemath，在calcsrc中创建calc.go文件，源码如下

```
package main
import "os"// 用于获得命令行参数os.Args
import "fmt"
import "tmp/simplemath"
import "strconv"
var Usage = func() {
    fmt.Println("USAGE: calc command [arguments] ...")
    fmt.Println("\nThe commands are:\n\tadd\tAddition of two values.\n\tsqrt\tSquare root of a non-negative value.")
}
func main() {
    args := os.Args
    if args == nil || len(args) < 2 {
        Usage()
        return
    }
    switch args[1] {
        case "add":
            if len(args) != 4 {
                fmt.Println("USAGE: calc add <integer1><integer2>")
                return
            }
            v1, err1 := strconv.Atoi(args[2])
            v2, err2 := strconv.Atoi(args[3])
            if err1 != nil || err2 != nil {
                fmt.Println("USAGE: calc add <integer1><integer2>")
                return
            }
            ret := simplemath.Add(v1, v2)
            fmt.Println("Result: ", ret)
        case "sqrt":
            if len(args) != 3 {
                fmt.Println("USAGE: calc sqrt <integer>")
                return
            }
            v, err := strconv.Atoi(args[2])
            if err != nil {
                fmt.Println("USAGE: calc sqrt <integer>")
                return
            }
            ret := simplemath.Sqrt(v)
            fmt.Println("Result: ", ret)
        default:
            Usage()
    }
}
```

在simplemath文件加下分别创建add.go和sqrt.go
add.go

```
package simplemath
func Add(a int, b int) int {
    return a + b
}
```

sqrt.go

```go
package simplemath
import "math"
func Sqrt(i int) int {
    v := math.Sqrt(float64(i))
    return int(v)
}
```

### 6 编译并运行程序

可以在/home/go/src/calcsrc文件夹下执行go build calc.go，该文件夹下生成calc可执行文件，然后运行`./calc add 2 3` 可以看到结果
也可以在/home/go/src 下执行go install calcsrc,在/home/go/bin下生成 calc可执行程序
然后运行`./calc add 3 2` 也可以看到结果