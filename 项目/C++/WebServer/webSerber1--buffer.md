这段代码定义了一个 `Buffer` 类，用于处理数据的读写操作。下面是对每一部分的详细解释：

### 1. 防止重复包含（Header Guard）
```cpp
#ifndef buffer_H
#define buffer_H
```
这部分是头文件保护，防止该头文件在同一编译单元中被多次包含。

### 2. 包含必要的头文件
```cpp
#include <iostream>
#include <unistd.h>		// write
#include <sys/uio.h>	// readv
#include <vector>
#include <string>
```
这里包含了程序运行所需的库文件，比如标准输入输出流 `<iostream>`，Unix标准IO函数 `<unistd.h>` 和 `<sys/uio.h>`，以及 `<vector>` 和 `<string>` 用于容器管理。

### 3. 命名空间定义
```cpp
namespace bre {
```
定义了一个名为 `bre` 的命名空间，以避免全局命名冲突。

### 4. Buffer 类定义
#### 构造函数与析构函数
```cpp
class Buffer {
public:
    explicit Buffer(int initBuffSize = 1024)
        : buffer(initBuffSize), readPos(0), writePos(0) 
    { }
    ~Buffer() = default;
```
构造函数初始化了 `buffer` 的大小，并将读写指针初始化为0。析构函数使用默认定义。

#### 拷贝控制
```cpp
    Buffer(const Buffer&) = delete;              
    Buffer& operator=(const Buffer&) = delete;  
```
这里禁止了类的拷贝构造和赋值操作，因为这个类管理着动态分配的资源。

#### 移动语义支持
```cpp
    Buffer(Buffer&& other) noexcept
        : buffer(std::move(other.buffer)),
        readPos(other.readPos),
        writePos(other.writePos) {
        other.readPos = 0;
        other.writePos = 0;
    }
    Buffer& operator=(Buffer&& other) noexcept {
        if (this != &other) {
            buffer = std::.move(other.buffer);
            readPos = other.readPos;
            writePos = other.writePos;
            other.readPos = 0;
            other.writePos = 0;
        }
        return *this;
    }
```
提供了移动构造函数和移动赋值运算符，使得对象在转移所有权时更加高效。

#### 数据读写状态查询
```cpp
    size_t WritableBytes() const {
        return buffer.size() - writePos;
    }
    size_t ReadableBytes() const {
        return writePos - readPos;
    }
```
返回可写和可读的字节数。

#### 获取数据视图
```cpp
    const char* Peek() const {
        return buffer.data() + readPos;
    }
```
提供一个指向当前可读数据起始位置的指针。

#### 数据读取
```cpp
    std::string Retrieve(size_t len);
    std::string RetrieveUntil(const std::string end);
    void Clear();
    std::string RetrieveAll();
```
这些方法用于从缓冲区中读取数据，或者清除所有数据。

#### 数据追加
```cpp
    void Append(const std::string& str);
    void Append(const void* data, size_t len);
    void Append(const Buffer& buff);
    void Append(const char* str, size_t len);
```
这些方法用于向缓冲区追加数据。

#### 扩展缓冲区大小
```cpp
private:
    void expandBuffer(size_t len);
```
当需要追加的数据长度超过剩余空间时，此方法会扩展缓冲区大小。

#### 文件描述符读写
```cpp
    ssize_t ReadFd(int fd, int* Errno);
    ssize_t WriteFd(int fd, int* Errno);
```
提供通过文件描述符进行读写的方法。

### 5. 私有成员变量

```cpp
private:
    std::vector<char> buffer;
    size_t readPos;
    size_t writePos;
```
定义了缓冲区和读写指针。





```c++
#ifndef buffer_H
#define buffer_H

#include <iostream>
#include <unistd.h>
#include <sys/uio.h>
#include <vector>
#include <string>

namespace bre {


class Buffer {
public:
    explicit Buffer(int initBuffSize = 1024)
        : buffer(initBuffSize), readPos(0), writePos(0) 
    { }

    ~Buffer() = default;

    Buffer(const Buffer&) = delete;              // 禁止拷贝构造
    Buffer& operator=(const Buffer&) = delete;   // 禁止拷贝赋值

    // 移动构造
    Buffer(Buffer&& other) noexcept
        : buffer(std::move(other.buffer)),
        readPos(other.readPos),
        writePos(other.writePos) {
        other.readPos = 0;
        other.writePos = 0;
    }

    // 移动赋值
    Buffer& operator=(Buffer&& other) noexcept {
        if (this != &other) {
            buffer = std::move(other.buffer);
            readPos = other.readPos;
            writePos = other.writePos;
            other.readPos = 0;
            other.writePos = 0;
        }
        return *this;
    }

    // 可写字节数
    size_t WritableBytes() const {
        return buffer.size() - writePos;
    }
    // 可读字节数
    size_t ReadableBytes() const {
        return writePos - readPos;
    }

    const char* Peek() const {
        return buffer.data() + readPos;
    }

    std::string Retrieve(size_t len) {
        if (len > ReadableBytes()) {
            throw std::out_of_range("Buffer::Retrieve: len is too large");
        }
        auto ret = std::string(Peek(), len);
        readPos += len;
        return ret;
    }

    std::string RetrieveUntil(const std::string end) {
        const std::size_t pos = std::string(Peek(), ReadableBytes()).find(end);
        if(pos == std::string::npos) {
            return "";
        }
        return Retrieve(pos + end.size());
    }

    void Clear() {
        readPos = writePos = 0;
    }

    std::string RetrieveAll() {
        std::string ret = std::string(Peek(), ReadableBytes());
        Clear();
        return ret;
    }

    void Append(const std::string& str) {
        Append(str.data(), str.size());
    }
    void Append(const void* data, size_t len) {
        Append(static_cast<const char*>(data), len);
    }
    void Append(const Buffer& buff) {
        Append(buff.Peek(), buff.ReadableBytes());
    }
    // 最终添加数据
    void Append(const char* str, size_t len) {
        if (WritableBytes() < len) {
            expandBuffer(len);
        }
        std::copy(str, str + len, buffer.data() + writePos);
        writePos += len;
    }

    ssize_t ReadFd(int fd, int* Errno) {
        char buff[65535];
        struct iovec iov[2];
        const size_t writable = WritableBytes();
        /* 分散读， 保证数据全部读完 */
        iov[0].iov_base = buffer.data() + writePos;
        iov[0].iov_len = writable;
        iov[1].iov_base = buff;
        iov[1].iov_len = sizeof(buff);

        const ssize_t len = readv(fd, iov, 2);
        if(len < 0) {
            *Errno = errno;
        }
        else if(static_cast<size_t>(len) <= writable) {
            writePos += len;
        }
        else {
            writePos = buffer.size();
            Append(buff, len - writable);
        }
        return len;
    }

    ssize_t WriteFd(int fd, int* Errno) {
        size_t readSize = ReadableBytes();
        ssize_t len = write(fd, Peek(), readSize);
        if(len < 0) {
            *Errno = errno;
            return len;
        } 
        readPos += len;
        return len;
    }

private:
    void expandBuffer(size_t len) {
        if (WritableBytes() + readPos < len) {
            // 重置缓冲区
            buffer.resize(writePos + len);
        } else {
            // 移动数据
            size_t readable = ReadableBytes();
            std::copy(buffer.data() + readPos, buffer.data() + writePos, buffer.data());
            readPos = 0;
            writePos = readPos + readable;
        }
    }
    
private:
    std::vector<char> buffer;
    size_t readPos;     // 读取偏移量
    size_t writePos;    // 写入偏移量
};


} // namespace bre
#endif // buffer_H
```

测试数据

```c++
#include <iostream>
#include <chrono>
#include <functional>
#include "Buffer.hpp"

class Test {
    std::chrono::time_point<std::chrono::system_clock> start;
    std::function<void()> callable;
public:
    Test(std::function<void()> func) : callable(func) {
        start = std::chrono::system_clock::now();
        if (callable) {
            callable();
        }
    }
    ~Test() {
        auto end = std::chrono::system_clock::now(); // 获取当前时间点
        std::chrono::duration<double, std::milli> d = end - start; // 计算时间差，并转换为毫秒
        std::cout << "Spend time: " << d.count() << "ms\n"; 
    }
};

void TestBuffer() {
    bre::Buffer buffer;

    // 测试初始状态
    std::cout << "Initial Writable Bytes: " << buffer.WritableBytes() << std::endl;
    std::cout << "Initial Readable Bytes: " << buffer.ReadableBytes() << std::endl;

    // 添加
    buffer.Append("Hello, ");
    char str[] = "World!";
    buffer.Append(str, 6);  // Buffer: Hello,World!

    // 获取
    std::cout << "获取: " << buffer.Retrieve(6) << std::endl;   // 获取“Hello,”  Buffer: World!
    // 寻找
    std::cout << "寻找: " << buffer.RetrieveUntil("o") << std::endl; // “Wo”    Buffer: rld!

    // 可读字符数和可写字符数
    std::cout << "Writable Bytes: " << buffer.WritableBytes() << std::endl;
    std::cout << "Readable Bytes: " << buffer.ReadableBytes() << std::endl;
    
    buffer.Append(str, 6);
    buffer.Retrieve(6);

    // 写入超过1024个字符
    for(int i = 0; i < 1000; ++i) {
        buffer.Append(str, 6);
    }
    buffer.Retrieve(5990);      // d!World!World! 从后往前14个字符剩下

    // 获取所有字符
    std::cout << "buffer.RetrieveAll(): " << buffer.RetrieveAll() <<"\n";

    // 清空
    buffer.Clear();

    // 测试清空后状态
    std::cout << "After Clear Writable Bytes: " << buffer.WritableBytes() << std::endl;
    std::cout << "After Clear Readable Bytes: " << buffer.ReadableBytes() << std::endl;
}

void TestPerformance() {
    bre::Buffer buffer;
    char str[] = "world!";
    for(int i = 0; i < 10000; ++i) {
        buffer.Append(str, 6);
        buffer.Retrieve(6);
    }
}
int main() {
    Test([]{TestBuffer();});
    Test([]{TestPerformance();});
    return 0;
}

```

