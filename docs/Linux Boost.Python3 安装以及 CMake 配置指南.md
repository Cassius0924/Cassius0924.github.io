# Linux Boost.Python3 安装以及 CMake 配置指南

## 下载Boost

前往[Boost官网](https://www.boost.org/)下载源码压缩包。

或通过 `wget` 下载 1.83 版本：

```bash
wget https://boostorg.jfrog.io/artifactory/main/release/1.83.0/source/boost_1_83_0.7z
```

下载完毕后，解压 7z 压缩包

```bash
7z x boost_1_83_0.7z
```

## 编译安装

```bash
cd boost_1_83_0
./bootstrap.sh --with-python=/root/.virtualenvs/ETRS/bin/python3 --with-python-version=3.8 --with-libraries=all 
```

`--with-python` 的路径可以通过 `which python3` 查看。

`--with-python-version` 的版本号可以通过 `python3 --version` 查看，注意这里需要填成 `3.x` 格式 ，而不是 `3.y.z`，省略最后的版本号。

运行完以上命令后，需要修改 `project-config` 配置文件。

```bash
vim project-config.jam
```

修改第 21行，在双引号里添加两个路径，分别是当前 Python 版本的 include 路径和 lib 路径。

![CleanShot 2023-11-11 at 20.44.10@2x](https://s2.loli.net/2023/11/11/gKp5WhZYdrC1XAN.png)

Python 的 include 路径和 lib 路径可以通过下面的 Python 代码查看：

```python
import sysconfig
sysconfig.get_path('include')		# 查看 include 路径
sysconfig.get_path('stdlib')		# 查看 lib 路径
```

开始编译：

```bash
./b2 
```

开始安装：

```bash
sudo ./b2 install --with-python include="/usr/include/python3.8"
```

## CMakeList 配置

```cmake
find_package(Boost 1.83 REQUIRED COMPONENTS python38)
include_directories(${Boost_INCLUDE_DIRS})

set(PYTHON_DOT_VERSION 3.8)
set(PYTHON_INCLUDE /usr/include/python3.8)
set(PYTHON_LIBRARY /usr/lib/python3.8/config-aarch64-linux-gnu)

add_executable(BoostTest Boostt.cpp)

target_link_libraries(BoostTest
        ${Boost_LIBRARIES}
        -lpython3.8
        -lpython2.7
        )
```

- `3.8`对应自己的 Python 的代码，例如`2.7`，`3.4`。
- `/usr/lib/python3.8/config-aarch64-linux-gnu` 需要对应自己电脑的路径，`python3.8` 需要改成自己系统的 Python 环境版本，若为 ARM64 架构的系统则为`config-aarch64-linux-gnu`，若是64位的 x86 架构的系统则为`config-x86_64-linux-gnu`。
- `-lpython3.8` 同样对应自己的 Python 环境版本。



