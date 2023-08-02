---
title: ONNXRuntime入门 - 如何编译ONNXRuntime?
tags:
  - Ascend
categories: []
number: 3
date: 2023-07-20 20:09:13
---

作者：张思博

本文介绍如何从0到1完成ONNXRuntime项目的编译。

<!--more-->

华为云 ECS 服务器配置如下

```
实例类型：AI加速型ai1s
规格名称：ai1s.large.4
操作系统：Ubuntu 18.04
```

#### 1.安装驱动

若驱动版本太老，卸载旧驱动

```shell
cd /usr/local/Ascend/opp/aicpu/script
./uninstall.sh

cd /usr/local/Ascend/ascend-toolkit/latest/toolkit/script
./uninstall.sh

cd /usr/local/Ascend/driver/script
./uninstall.sh
```

安装最新驱动，NPU 型号 [Atlas 300I 推理卡（型号：3010）](https://www.hiascend.com/zh/hardware/firmware-drivers/community?product=2&model=3&cann=6.3.RC2.alpha002&driver=1.0.18.alpha)

```shell
# 添加执行权限
chmod u+x A300-3010-npu-driver_6.0.0_linux-x86_64.run

# 安装驱动
./A300-3010-npu-driver_6.0.0_linux-x86_64.run --full

# 若能正常显示两块 Ascend 310，则安装成功
npu-smi info
```

#### 2.安装开发套件

安装 Anaconda

```shell
# 下载安装包
wget https://repo.anaconda.com/archive/Anaconda3-2023.03-1-Linux-x86_64.sh

# 增加可执行权限
chmod u+x Anaconda3-2023.03-1-Linux-x86_64.sh

# 执行安装，选择执行 conda init
./Anaconda3-2023.03-1-Linux-x86_64.sh

# 更新 conda
conda update -n base -c defaults conda

# 重新打开终端进入base环境，并创建虚拟环境
conda create -n onnxruntime python=3.8

# 激活环境
conda activate onnxruntime
```

安装依赖

```shell
# 安装依赖依赖软件
apt-get install -y gcc g++ make cmake zlib1g zlib1g-dev openssl libsqlite3-dev libssl-dev libffi-dev unzip pciutils net-tools libblas-dev gfortran libblas3 libopenblas-dev

# 安装 python 库
conda install  attrs numpy decorator sympy cffi pyyaml pathlib2 psutil protobuf scipy requests
```

安装开发套件，在[官网](https://www.hiascend.com/zh/software/cann/community)选择合适版本

```shell
# 下载安装包
wget https://ascend-repo.obs.cn-east-2.myhuaweicloud.com/Florence-ASL/Florence-ASL%20V100R001C30SPC702/Ascend-cann-toolkit_6.3.RC2.alpha002_linux-x86_64.run

# 修改权限
chmod u+x Ascend-cann-toolkit_6.3.RC2.alpha002_linux-x86_64.run

# 安装
./Ascend-cann-toolkit_6.3.RC2.alpha002_linux-x86_64.run  --install
```

#### 3.项目编译

安装 cmake-3.26 或更高版本

```shell
python3 -m pip install cmake
```

升级 gcc 和 g++ 版本

```shell
# 添加 PPA 源
sudo add-apt-repository ppa:ubuntu-toolchain-r/test
sudo apt-get update

# 安装高版本
sudo apt-get install gcc-11       #(大版本号即可)
sudo apt-get install g++-11

# 删除原链接
cd /usr/bin
rm /usr/bin/gcc
rm /usr/bin/g++

# 添加软连接
ln -s gcc-11 gcc
ln -s g++-11 g++
```

添加环境变量

```shell
source /usr/local/Ascend/ascend-toolkit/set_env.sh
```

编译

```shell
# 克隆项目
 git clone --recursive https://github.com/Microsoft/onnxruntime.git
 cd onnxruntime

# 编译
./build.sh --allow_running_as_root --config RelWithDebInfo --build_shared_lib --parallel --use_cann --build_wheel
```

若想在后台运行编译过程

```shell
setsid ./build.sh --allow_running_as_root --config RelWithDebInfo --build_shared_lib --parallel --use_cann --build_wheel &>/root/Projects/build.log &

# 实时监控日志
tail -f ../build.log
```

若需要部署应用，可以将编译后的动态链接库和头文件进行安装

```shell
# 安装
make -C build/Linux/RelWithDebInfo/ install

# 卸载（不删除文件夹）
cat install_manifest.txt | sudo xargs rm
```
