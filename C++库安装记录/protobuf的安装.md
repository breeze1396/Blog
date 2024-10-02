

#  1.解压缩

- 首先，从[官方github](https://github.com/protocolbuffers/protobuf/releases)下载适用于Windows的protobuf编译器。
- 将下载的文件解压到一个特定的目录下。

# 2. 配置环境变量

- 在系统的环境变量中，找到`PATH`变量。
- 将protobuf编译器的bin目录添加到`PATH`变量中。
- 保存更改并关闭环境变量窗口。

# 3. 验证安装

打开命令提示符或PowerShell，输入以下命令：

```bash
protoc --version
```

如果显示protobuf的版本信息，则表示安装成功。