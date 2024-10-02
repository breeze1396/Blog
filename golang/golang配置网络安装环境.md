安装Go语言环境经常遇到Connection refused错误。例如：

```text
dial tcp 142.251.43.17:443: connect: connection refused
```

通过设置Go的环境变量可以解决这个问题：

GO111MODULE=on，GOPROXY=[https://goproxy.cn](https://link.zhihu.com/?target=https%3A//goproxy.cn),direct

```bash
go env #显示go环境变量
go env -w GO111MODULE=on #设置变量
go env -w GOPROXY=https://goproxy.cn,direct  #设置变量
go env #检查变量设置生效
go get -v golang.org/x/tools/cmd/goimports #安装goimports测试是否能够正常安装
```