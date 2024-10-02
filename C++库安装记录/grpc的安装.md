

## 安装编译依赖软件

在 Windows 系统上编译 gRPC 需要首先准备下述软件：

- Visual Studio，将使用到 Visual C++ compiler
- Git
- CMake
- nasm



### nasm

gRPC 的第三方依赖 `boringssl` 需要此软件。

在 [nasm 官网](https://www.nasm.us/)下载，可以选择下载 `.exe` 文件直接安装。

例如对于 64 位的 Windows 电脑安装 `nasm 2.15.05`，可以进入 `/pub/nasm/releasebuilds/2.15.05/win64` 目录下载 `nasm-2.15.05-installer-x64.exe` 文件并执行安装操作。

nasm 默认安装目录为 `C:\Users\${您的工号}\AppData\Local\bin\NASM`（若非此目录，请在安装界面确认安装的路径），将该目录添加到环境变量中即可。

```
PS C:\Users\breeze> nasm --version
NASM version 2.16.03 compiled on Apr 17 2024
```



## 拉取 gRPC 库

建议在能够连接 Github 的机器环境利用 Git 克隆 gRPC 库并获取第三方依赖，再打包出来给 windows 系统编译使用。

```
# 克隆 gRPC 仓库
# 对于特定的分支，例如 gRPC 1.28.1 版本，可以使用此命令：
# git clone https://github.com/grpc/grpc.git -b v1.28.1
git clone https://github.com/grpc/grpc.git
# 获取 gRPC 第三方依赖
cd grpc
git submodule update --init  // 这是最难的地方，因为连接外网络下载极其的慢
```

## 编译安装 gRPC

特别的，如果您使用的 CMake 版本低于 3.13，或编译的 gRPC 版本低于 1.27，在执行 `cmake` 命令之前，需要自行手动编译安装 gRPC 的依赖库，且在执行 `cmake` 命令时指定这些库的路径，这里是[官方的说明](https://github.com/grpc/grpc/blob/master/BUILDING.md#install-after-build)。

下面的内容基于的 CMake 版本不低于 3.13，且编译的 gRPC 版本不低于 1.27。

首先创建文件夹存储编译结果，并执行 `cmake` 命令生成 Makefile 文件。使用**管理员权限**打开命令行界面，执行下面的命令：

```
# 在 grpc 目录下创建 .build 目录并进入
md .build
cd .build
# 生成 Makefile 文件
# 其中 Visual Studio 15 2017 为当前的 VS 版本
cmake .. -DgRPC_INSTALL=ON -G "Visual Studio 15 2017"
```

> 尽管[不推荐](https://github.com/grpc/grpc/blob/master/BUILDING.md#windows-a-note-on-building-shared-libs-dlls)，在上面执行 `cmake` 命令时，您可以指定 `-DBUILD_SHARED_LIBS=ON` 以编译生成 gRPC C++ 的 DLL 文件。

接下来对 gRPC 进行编译安装操作，包括两种方式，这里更建议使用 VS。

### 使用 VS 编译并安装 gRPC

首先，使用**管理员权限**打开 VS，否则在安装 gRPC 时会报错。

接着使用 VS 打开此目录下的 `grpc.sln` 解决方案，找到**解决方案资源管理器**（默认情况下在 VS 的右侧）中的 `ALL_BUILD` 项，右键并选择**生成**按钮，开始执行编译操作。

编译结束后，在**解决方案资源管理器**中找到 `INSTALL` 项，右键并选择**生成**按钮，开始执行安装操作。

gRPC 默认安装在 `C:\Program Files (x86)\grpc` 目录。

如果报错，请确保您已使用管理员权限打开 VS。使用管理员权限重新打开 VS 后，右键点击 `ALL_BUILD` 并选择**重新生成**按钮即可。

### 使用命令行编译并安装 gRPC

您也可以直接使用命令行界面，执行 `cmake` 命令来编译安装 gRPC。

```
# 编译并安装 gRPC
cmake --build . --target install --config Release
```

编译结束后，会自动进行安装操作。

gRPC 默认安装在 `C:\Program Files (x86)\grpc` 目录。

如果没有使用管理员权限打开命令行界面，安装时会发生报错。请重新使用管理员权限打开命令行界面，并执行上面的命令。

## 测试 gRPC 编译安装结果

接下来的步骤基于 VS 编译安装 `gRPC 1.28.1` 的结果，测试 Windows 系统下的 gRPC 环境是否安装成功。

移动到 git 克隆的 gRPC 源码目录下的 `grpc\examples\cpp\helloworld` 目录，创建存放编译结果的文件夹 `cmake\build`，进入到该目录，执行下面的命令：

```
# 生成 Makefile 文件
# 其中 C:\Program Files (x86)\grpc 是 gRPC 默认的安装目录
cmake -DCMAKE_PREFIX_PATH='C:\Program Files (x86)\grpc' ../..
```

使用管理员权限打开 VS，并打开当前目录下的 `HelloWorld.sln` 解决方案，右键分别选择解决方案资源管理器中的 `greeter_client.cc` 和 `greeter_server.cc` 并点击生成按钮。

编译完成后，会在 `cmake\build\Debug` 目录下生成我们需要的可执行文件 `greeter_server.exe` 和 `greeter_client.exe`。

使用 Powershell 移动到该目录，启动服务端 `./greeter_server.exe`：

```
PS grpc\examples\cpp\helloworld\cmake\build\Debug> ./greeter_server.exe
Server listening on 0.0.0.0:50051
```

服务端默认监听 50051 端口。

再启动一个 Powershell 移动到该目录，启动客户端 `./greeter_client.exe`：

```
PS grpc\examples\cpp\helloworld\cmake\build\Debug> ./greeter_client.exe
Greeter received: Hello world
```

成功打印出 `Greeter received: Hello world` 字段，测试成功！

作为对照组，您可以关闭掉服务端，再执行客户端，观察打印的结果。

Hello, gRPC world!