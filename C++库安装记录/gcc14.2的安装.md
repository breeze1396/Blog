# 安装 gcc-14.2  Ubuntu 22.04



```bash
sudo apt install build-essential
sudo apt install libmpfr-dev libgmp3-dev libmpc-dev -y
wget http://ftp.gnu.org/gnu/gcc/gcc-14.2.0/gcc-14.2.0.tar.gz
tar -xf gcc-14.2.0.tar.gz
cd gcc-14.2.0
./configure -v --build=x86_64-linux-gnu --host=x86_64-linux-gnu --target=x86_64-linux-gnu --prefix=/usr/local/gcc-14.2.0 --enable-checking=release --enable-languages=c,c++ --disable-multilib --program-suffix=-14.2.0
make
sudo make install
```

```
sudo update-alternatives --install /usr/bin/g++ g++ /usr/local/gcc-14.2.0/bin/g++-14.2.0 14
sudo update-alternatives --install /usr/bin/gcc gcc /usr/local/gcc-14.2.0/bin/gcc-14.2.0 14
```

检查 gcc 和 g++ version:

```c++
g++ --version
g++ (GCC) 14.2.0
Copyright © 2024 Free Software Foundation, Inc.
 
gcc --version
gcc (GCC) 14.2.0
Copyright © 2024 Free Software Foundation, Inc.
```



解决报错：/lib/x86_64-linux-gnu/libstdc++.so.6: version `GLIBCXX_3.xxx‘ not found

第1步，移除现有软连接

```bash
sudo rm /usr/lib/x86_64-linux-gnu/libstdc++.so.6
```

第2步，找到缺失的libstdc++.so.6.0.33，cd到下载路径下再执行

```bash
cd /usr/local/gcc-14.2.0/lib64/ #使用find
sudo mv libstdc++.so.6.0.33 /usr/lib/x86_64-linux-gnu/
```

第四步，重新建立软连接

```bash
sudo ln -s /usr/lib/x86_64-linux-gnu/libstdc++.so.6.0.29 /usr/lib/x86_64-linux-gnu/libstdc++.so.6
```



最后测试，C++23输出

```c++
g++ -std=c++23 -fmodules-ts cpp20test.cpp
```



```c++
// cpp20test.cpp

#include <iostream>
#include <concepts>
#include <ranges>
#include <cassert>
#include <vector>  // 包含 vector 头文件
#include <compare> // 包含用于三态比较的头文件
#include <functional>

template<typename T>
concept SignedInteger = std::integral<T> && T(-1) < T(0);

// 模式匹配
void patternMatching() {
    auto data = std::vector<std::pair<int, std::string>>{
        {1, "one"},
        {2, "two"},
        {3, "three"}
    };

    for (auto [number, text] : data) {
        std::cout << "Number: " << number << ", Text: " << text << std::endl;
    }
}

// 三态比较
void threeWayComparison() {
    auto compareInts = [](int x, int y) {
        if constexpr (SignedInteger<decltype(x - y)>) {
            return std::strong_ordering(x <=> y); // 使用 <=> 运算符
        } else {
            if (x < y) return std::strong_ordering::less;
            if (x > y) return std::strong_ordering::greater;
            return std::strong_ordering::equal;
        }
    };

    assert(compareInts(1, 2) == std::strong_ordering::less);
    assert(compareInts(2, 1) == std::strong_ordering::greater);
    assert(compareInts(2, 2) == std::strong_ordering::equal);
}

int main() {
    patternMatching();
    threeWayComparison();

    return 0;
}
```

