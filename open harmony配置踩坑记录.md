基于open Harmony的开发需要Linux与Windows两种操作系统环境，其中，Linux主要用于源码的下载和编译，Windows用于代码编写与固件烧录等（小白也不太清楚其他的:(）。

Linux我采用了Windows11中wsl功能（也就是Windows子系统（也算是一个虚拟机））中的Ubuntu22.04LTS(亲测好用：可以跳过用于传输文件的过程，可以直接在Windows里打开Linux里的文件)。Windows就是win11。不过需要熟悉一些Linux的基本命令行命令如ls等。

Ubuntu虚拟机在终端中了可以打开

配置完基本的操作系统环境，就开始配置软件环境：

Windows：直接下载安装DevEco Device Tool IDE然后再安装（是一个插件，但是可以直接帮你安装上vscode）

ubuntu:因为不懂，就也安装了一个DevEco Device Tool IDE后来发现没有图形化界面用不了，也可能我不会用。之后安装Linux基础使用软件

sudo apt-get update && sudo apt-get install -y vim net-tools
--sudo apt-get update 更新软件列表
--vim linux 编辑器
--net-tools linux网络工具（ifconfig查看网络工具）
下面这个虽然是远程访问工具，但是发现后面还是用到Windows上的vscode远程连接，所以推荐安装。

sudo apt-get install -y openssh-server
安装编译依赖软件

sudo apt-get install -y build-essential gcc g++ make zlib* libffi-dev
--build-essential 提供编译程序必须软件包的列表信息
--gcc gcc编译器
--g++ g++编译器
--make 编译工具
--zlib* 数据压缩函式库
--libffi-dev libffi的开发文件
安装并升级Python包管理工具（pip3）

sudo apt-get install python3-setuptools python3-pip -y
sudo pip3 install --upgrade pip
验证是否安装成功 pip3 -v

建立软连接，将python指向python3.10

sudo ln -s /usr/bin/python3.10 /usr/bin/python
配置pip包下载源，加速国内安装pip包： 下面命令可直接拷贝到终端窗口

mkdir ~/.pip/
cat <<EOF > ~/.pip/pip.conf
[global]
index-url = https://mirrors.huaweicloud.com/repository/pypi/simple
trusted-host = mirrors.huaweicloud.com
timeout = 120
EOF
安装命令(用scons -v验证是否成功要求版本在3.0.4以上)

sudo apt-get install -y scons
--scons 编译构建脚本
--setuptools 是Python的 distutils增强工具，它可以帮助我们更简单的创建和分发Python包， 尤其是拥有依赖关系的。 --kconfiglib 是 Python 2/3 中的 Kconfig 实现。它最初是一个帮助程序库，但现在具有足够 的功能，也可以作为独立的 Kconfig 实现（包括终端和 GUI 菜单配置接口以及 Kconfig 扩展）。 --pycryptodome 是Python一个强大的加密算法库 --ecdsa 主要用于对数据（比如一个文件）创建数字签名，以便于在不破坏其安全性的前提下对其 真实性进行验证。 --six 是一个专门用来兼容 Python 2 和 Python 3 的库

sudo pip3 install setuptools kconfiglib pycryptodome ecdsa --upgrade --ignore-installed six
接着是编译工具链：

# 下载gn/ninja包
URL_PREFIX=https://repo.huaweicloud.com/harmonyos/compiler
DOWNLOAD_DIR=~/Downloads # 下载目录，可自行修改
TOOLCHAIN_DIR=~/harmonyos/toolchain # 工具链存放目录，可自行修改
[ -e $DOWNLOAD_DIR ] || mkdir $DOWNLOAD_DIR
[ -e $TOOLCHAIN_DIR ] || mkdir -p $TOOLCHAIN_DIR
wget -P $DOWNLOAD_DIR $URL_PREFIX/gn/latest/linux/gn-linux-x86-1717.tar.gz
wget -P $DOWNLOAD_DIR $URL_PREFIX/ninja/1.9.0/linux/ninja.1.9.0.tar
# 编译 hi3861 需要 riscv 编译工具链
wget -P $DOWNLOAD_DIR $URL_PREFIX/gcc_riscv32/7.3.0/linux/gcc_riscv32-linux-7.3.0.tar.gz
接着一些配置：

# 解压gn/ninja包：
tar -C $TOOLCHAIN_DIR/ -xvf $DOWNLOAD_DIR/gn-linux-x86-1717.tar.gz
tar -C $TOOLCHAIN_DIR/ -xvf $DOWNLOAD_DIR/ninja.1.9.0.tar
tar -C $TOOLCHAIN_DIR/ -xvf $DOWNLOAD_DIR/gcc_riscv32-linux-7.3.0.tar.gz
# 向 ~/.bashrc 中追加gn/ninja路径配置：
cat <<EOF >> ~/.bashrc
TOOLCHAIN_DIR=$TOOLCHAIN_DIR
export PATH=\$TOOLCHAIN_DIR/gn:\$PATH
export PATH=\$TOOLCHAIN_DIR/ninja:\$PATH
export PATH=\$TOOLCHAIN_DIR/gcc_riscv32/bin:\$PATH
export PATH=~/.local/bin:\$PATH # 用户pip二进制工具目录
EOF
# 生效环境变量
source ~/.bashrc
之后就到了坑最多的安装hb阶段：python3 -m pip install --user ohos-build

如果你安装后输入hb直接没有报错，那么你很成功，我是报了错，报了错也别慌，可以先下载open Harmony原码，然后进入类似于\home\user\harmony\openharmony\之类的源码目录下执行pip install build\lite 进行hb安装。

如果还是报错可以尝试进入openharmony目录下build目录下找到一个build.py文件执行 python build.py （你的要编译的开发的源码名）我的是wifi_iot

如果和我一样背，还是错，如果报错信息是

[OHOS ERROR] Reason:      cannot import name 'Mapping' from 'collections' (/usr/lib/python3.10/collections/init.py)
那么就更新代码： 如果你有权修改脚本代码，尝试将 Mapping 替换为 collections.abc.Mapping。在 Python 3.3 及更高版本中，collections.abc 模块提供了 Mapping 抽象基类

from collections.abc import Mapping
之后我的hb就可以顺利运行了，附一张运行成功截图：

hb set 命令运行后截图
 作者：街飞雨 https://www.bilibili.com/read/cv29268419/ 出处：bilibili