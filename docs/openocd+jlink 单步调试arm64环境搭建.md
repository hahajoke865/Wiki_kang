准备：
- 树莓派 4B 开发板（树莓派 4B 才是兼容 arm64 的）
- armv8 体系结构
- 虚拟机 Ubuntu Linux 16.04（其他版本也可）
- MicroSD 卡以及读卡器（建议8GB~32GB）
- 杜邦线若干
- 串口转 TTL 模块
- J-Link 仿真器（最好是 v9.0 版本以上）

![p9](/openocd+jlink单步调试arm64环境搭建/p9.jpg)

## 0.树莓派与串口 TTL 接线
接线方式：

    usb         ---->           树莓派
    GND         ---->           Ground 06
    RXD         ---->          (TXD0)GPIO14 08
    TXD         ---->          (RXD0)GPIO15 10 

![p4](/openocd+jlink单步调试arm64环境搭建/p4.jpg)

## 1.树莓派镜像安装

对于树莓派的镜像，我们可以登录 [*Raspberry镜像网站*](https://www.raspberrypi.com/software/operating-systems/) 进行镜像的下载，那么因为我们主要用于 arm64 的嵌入式调试，因此我们下载要下载 64 bit 的树莓派镜像，并且是没有桌面的。

![p1](/openocd+jlink单步调试arm64环境搭建/p1.jpg)

当我们下载好树莓派镜像之后，我们可以使用 `Win32DiskImager` 工具，将我们的树莓派镜像烧写到我们的 MicroSD 卡中（Linux 主机可以使用 dd 命令进行烧写），但在烧写之前，建议先使用 `SDFormatter` 工具格式化我们的 MicroSD 卡，防止卡中其他文件对镜像进行干扰。

![p2](/openocd+jlink单步调试arm64环境搭建/p2.jpg)

当烧写完毕之后，我们的 MicroSD 卡中就会生成一个 boot 分区，在 boot 分区中我们可以找到一个名为 config.txt 的配置文件，我们需要在此配置文件的末尾添加以下内容。
```markdown
uart_2ndstage=1  //打开固件的调试日志
enable_uart=1    //使用串口1
```

有了以上操作，我们的树莓派就可以正常的上电运行了，但是我们为了可以查看树莓派打印出来的调试信息，并且在树莓派启动后我们可以使用命令行中断去操作它，调试它，因此我们还需要将树莓派和我们的串口转 TTL 模块进行硬件上的连接，之后将串口转 TTL 模块插入到我们的 pc/windows 机中，然后使用类似 `SecureCRT 7.3` 软件来进行调试信息的打印和交互。
```markdown
当树莓派运行起来之后，他会要求我们输入用户名和密码
默认用户名：pi
默认密码：raspberry
```


## 2.树莓派联网与更新软件包

对于树莓派联网，我们可以输入以下命令进入配置页面，然后再在配置页面选择 wifi 联网操作，然后输入 ssid 和密码，即可连上 wifi 网络。
```markdown
1.sudo raspi-config           //打开配置界面
2.Sytem Option                //配置界面选项
    --->Wireless LAN
        --->输入ssid
            --->输入密码
3.sudo reboot                 //重启
```

![p3](/openocd+jlink单步调试arm64环境搭建/p3.png)

当连接上网络之后，我们就可以输入以下命令为我们的树莓派更新软件包了，更新的过程会比较久，请耐心等待
```shell
sudo apt update
sudo apt full-upgrade
sudo reboot
```

## 3.J-Link 测试

如果使用非正版的 J-Link 的话，有可能会出现 J-Link 无法识别的情况，这很有可能是厂家硬件或者固件的问题，那么为了排除 J-Link 的硬件问题，防止下面 OpenOCD 出错时，我们还以为是 OpenOCD 安装出错了，我们可以先来验证一下 J-Link 是否可以正常调试。

那么我们可以在 Ubuntu Linux 16.04 虚拟机上，先安装 J-Link，我们可以登录 [*SEGGER JLINK 下载*](https://www.segger.com/downloads/jlink) 进行 J-Link 程序的 deb 版本的下载，下载好了之后，我们可以使用下面命令来进行安装。
```shell
sudo dpkg -i 文件名.deb
```

那么接下来就是 J-Link 接线，J-Link 可以先不连接我们的树莓派，仅仅将 J-Link 通过它自身的 usb 线连接到我们的虚拟机即可。

接线和安装完成之后，我们可以新建一个终端，在终端中输入 `lsusb` ，看看我们的 J-Link 是否已经在 Ubuntu Linux 16.04 虚拟机上识别出了 usb 设备，需要注意，可能每个人输出的 usb 设备的名字是不一样的，然后在终端输入 `JLinkExe` 来运行我们的 J-Link 程序，如果执行结果如下所示，说明我们的 J-Link 仿真器是没有问题的。

![p7](/openocd+jlink单步调试arm64环境搭建/p7.jpg)


## 4.J-Link 和树莓派硬件连接

连接如下图所示，需要注意，若 J-Link 需要给树莓派供电，那么请接上 J-Link 上的 VTref 和 GND 引脚，若树莓派是单独另外供电，那么无需接上这两个引脚。

![p5](/openocd+jlink单步调试arm64环境搭建/p5.jpg)

![p6](/openocd+jlink单步调试arm64环境搭建/p6.jpg)


## 5.OpenOCD 安装

OpenOCD（Open On-Chip Debugger，开源片上调试器），它是一款开源的调试软件，OpenOCD 提供针对嵌入式设备的调试，系统编程和边界扫描功能。OpenOCD 需要使用硬件仿真器来配合完成调试，例如 J-Link 仿真器等。OpenOCD 内置了 GDB server 模块，可以通过 GDB 命令来调试硬件。

安装 OpenOCD 之前，我们先使用以下命令，安装 OpenOCD 所需的依赖
```shell
sudo apt-get install qemu-system-arm libncurses5-dev gcc-aarch64-linux-gnu build-essential git bison flex libssl-dev

sudo apt install make libtool pkg-config autoconf automake texinfo
```

对于 OpenOCD 的安装，在网上有很多通过 git 的方式来下载 OpenOCD 源码，但是经过我的尝试，git 下载的 OpenOCD 会出现插入 J-Link 后无法识别出 J-Link 的情况，目前暂时没有办法排查出原因所在，因此，在这里推荐大家在 `sourceforge.net` 上进行下载，也可以直接在 [*OpenOCD 0.12.0-rc1*]  下载，下载完成后传到我们的 Ubuntu Linux 16.04 虚拟机上，在用户目录下使用以下命令进行安装。
```markdown
mkdir OpenOCD_Install                //创建一个目录，把下载的 OpenOCD 压缩包存放到里面
tar -vxzf openocd-0.12.0-rc1.tar.gz  //解压下载的 OpenOCD
cd openocd-0.12.0-rc1/               //进入解压后的目录
mkdir tmp                            //创建安装目录
./configure --prefix=/your_path/OpenOCD_Install/openocd-0.12.0-rc1/tmp  //指定安装路径，需要注意这里的 your_path 为你存放 OpenOCD_Install 文件夹的目录
sudo make                            //编译
sudo make install                    //安装
```

安装完成之后，就会在 tmp 目录下得到下面的目录架构，需要注意，下面不重要的目录使用 xxx 表示
```markdown
tmp
├── bin
│   └── openocd
│   
├── share
    ├── info
    ├── man
    └── openocd
        ├── contrib
        ├── OpenULINK
        └── script
            ├── xxx
            ├── xxx
            └── interface
                ├── xxx
                ├── xxx
                └── jlink.cfg
```

为了使用 openocd 命令连接 J-Link 仿真器，需要指定配置文件。OpenOCD 的安装包里内置了 jlink。cfg 文件，该文件保存在我们安装目录的 tmp/share/openocd/script/interface 目录下，jlink.cfg 文件比较简单，可以通过 "adapter" 命令连接 J-Link 仿真器。
```markdown
<jlink.conf> 配置文件
#SEGGER J-Link

adapter driver jlink
```

在 `/your_path/OpenOCD_Install/openocd-0.12.0-rc1/tmp/bin` 目录里，我们可以通过 `openocd -f ../share/openocd/scripts/interface/jlink.cfg -f ../share/openocd/scripts/interface/raspberry4.cfg` 命令来启动我们的调试。

![p8](/openocd+jlink单步调试arm64环境搭建/p8.jpg)


### 待更新
1.登录 Telnet 服务

2.使用 GDB 调试

