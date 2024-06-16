# Jetson Xavier NX 安装 CUDA 支持的 PyTorch 指南

本指南将帮助开发者完成在 Jetson Xavier NX 上安装 CUDA 支持的 PyTorch。

## 安装方法

在 Jetson 上安装 Pytorch 只有两种方法。

- 一种是直接安装他人已经编译好的 PyTorch 轮子；
- 一种是自己从头开始开始构建 PyTorch 轮子并且安装。

## 使用轮子安装

可以从我的 [GitHub 仓库](https://github.com/Cassius0924/JetsonPytorch/) 直接下载我编译好的 PyTorch 轮子（torch-1.13.0-cuda-11.4-python-3.8-aarch64）。

安装前先确保 python 版本为 PyTorch 轮子对应的 Python 3.8。

下载完毕后使用`pip install`安装：

```bash
sudo -H pip install torch-1.13.0a0+git7c98e70-cp38-cp38-linux_aarch64.whl
```

如果你以前安装过 torch 的其他版本，则需要添加`--force-reinstall`覆盖安装。

其他低于 CUDA 11 的 PyTorch 轮子可以在 [Qengineering 的 GitHub 仓库](https://github.com/Qengineering/PyTorch-Jetson-Nano) 寻找。

## 编译源码安装

接下来说一下重点，自己从源码构建轮子并安装。自行构建需要花费数个小时的时间。

PyTroch 1.11.0 以及以上版本只能在 Ubuntu 20.04 上安装。

### 安装依赖

```bash
sudo apt-get update
sudo apt-get upgrade
```

```bash
sudo apt-get install ninja-build git cmake 
sudo apt-get install libjpeg-dev libopenmpi-dev libomp-dev ccache libopenblas-dev libblas-dev libeigen3-dev
```

```bash
sudo pip3 install -U --user wheel mock pillow
```

```bash
sudo -H pip3 install testresources setuptools==58.3.0 scikit-build
```

### 下载源码

请克隆自己需要的 PyTorch 版本，修改`-b`后的版本参数。

```bash
git clone -b v1.13.0 --depth=1 --recursive https://github.com/pytorch/pytorch.git
```

```bash
cd pytorch
```

### 扩大交换内存

构建 PyTorch 需要大于 4GB 的 RAM 和 2GB 的交换内存。

先使用`free -m`查看当前的交换内存。

安装`dphys-swapfiel`：

```bash
sudo apt-get install dphys-swapfile
```

需要修改两个文件`/sbin/dphys-swapfile`和`/etc/dphys-swapfile`。

先修改`/sbin/dphys-swapfile`的第30行，将`CONF_MAXSWAP=2048`修改成`4096`：

```bash
sudo vim /sbin/dphys-swapfile
```

![/sbin/dphys-swapfile](https://s2.loli.net/2023/10/13/omvFEQWBjHRyIKO.png)

再修改`/etc/dphys-swapfile`的第26行，将`#`去掉，后面加上`4096`：

```bash
sudo vim /etc/dphys-swapfile
```

![/etc/dphys-swapfile](https://s2.loli.net/2023/10/13/UVB2AKEsSbHTuWt.png)

保存修改后重启系统。

```bash
sudo reboot
```

重启完毕后可以使用 `free -m` 查看交换内存是否扩大成功。

## 使用 GCC

请使用 GCC 和 G++ 编译器，如果使用 GNU 编译器会导致浮点数出现错误的问题，使用 Clang 编译器在构建 CUDA 11.4 的 PyTorch 时候会报错。

## 设置 CUDA 版本

### 修改环境变量

修改系统环境变量，如果你使用的是 bash 则修改 `~/.bashrc`，如果使用 zsh 则修改 `~/.zshrc`。

```bash
vim ~/.zshrc #或 ~/.bashrc
```

在文件末尾添加三行：

```bash
export CUDA_HOME=/usr/local/cuda
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/cuda/lib64:/usr/local/cuda/extras/CUPTI/lib64
export PATH=$PATH:$CUDA_HOME/bin
```

如果你之前添加过 CUDA 的环境变量，则直接在对应行修改。

保存并退出，应用修改：

```bash
source ~/.zshrc #或 ~/.bashrc
```

### 切换 CUDA 版本

先切换系统 CUDA 版本到即将编译的 CUDA 版本，这里以 CUDA 11.4 为例。

```bash
sudo update-alternatives --config cuda
```

选择`2`，回车。

![update-alternatives cuda](https://s2.loli.net/2023/10/13/hqS3wJzZbT7GtKo.png)

### 修改 CMakeLists.txt

需要在 CMakeLists.txt 文件里指定 CUDA 编译器的路径。

```bash
cd pytorch
vim CMakeLists.txt
```

在 `project(Torch CXX C)`上方添加一行 `set(CMAKE_CUDA_COMPILER /usr/local/cuda/bin/nvcc)`

![CMAKE_CUDA_COMPILER](https://s2.loli.net/2023/10/13/4dqANWEFkuxYcei.png)

保存并退出。

## 构建 PyTorch

### 设置 Ninja 构建参数

需要添加一些环境变量。

```bash
export BUILD_CAFFE2_OPS=OFF
export USE_FBGEMM=OFF
export USE_FAKELOWP=OFF
export BUILD_TEST=OFF
export USE_MKLDNN=OFF
export USE_NNPACK=OFF
export USE_XNNPACK=OFF
export USE_QNNPACK=OFF
export USE_PYTORCH_QNNPACK=OFF
export USE_CUDA=ON
export USE_CUDNN=ON
export TORCH_CUDA_ARCH_LIST="5.3;6.2;7.2"
export USE_NCCL=OFF
export USE_SYSTEM_NCCL=OFF
export USE_OPENCV=OFF
export MAX_JOBS=4
export PATH=/usr/lib/ccache:$PATH
export CC=gcc
export CXX=g++
export CUDACXX=/usr/local/cuda/bin/nvcc
export CUDA_TOOLKIT_ROOT_DIR=/usr/local/cuda

sudo ln -s /usr/lib/aarch64-linux-gnu/libcublas.so /usr/local/cuda/lib64/libcublas.so
```

### 开始构建 PyTorch

```bash
python3 setup.py bdist_wheel
```

如果构建过程中报错，需要重新构建，请先用`python3 setup.py clean`重置构建。

## 安装 PyTorch

经过漫长的编译，编译好的`.whl`轮子文件会放在`dist`文件夹里。

```bash
cd dist
sudo -H pip3 install <torch文件>.whl
```

如果你以前安装过 torch 的其他版本，则需要添加`--force-reinstall`覆盖安装：

```bash
sudo -H pip3 install <torch文件>.whl --force-reinstall
```

## 验证安装

导入验证 torch 不能在 PyTorch 源代码目录，否则会报错，先用`cd ../..`退出源代码文件夹。

```bash
python3 -c "import torch as t; print(t.__version__); print(t.version.cuda); print(t.cuda.is_available());"
```

![CleanShot 2023-10-13 at 11.05.01@2x](https://s2.loli.net/2023/10/13/FybLW9Ad5sVOfGR.png)

大功告成！

## 清理空间

先停 dphys-swapfile：

```bash
sudo /etc/init.d/dphys-swapfile stop
```

如果磁盘空间紧缺，可以将 PyTorch 源代码删除，不过删除前建议先将 Jetson 辛辛苦苦构建 PyTorch 轮子备份保存一下。

```bash
mv pytorch/dist/*.whl ~
```

删除源代码：

```bash
rm -r pytorch
```

