准备：

- Ubuntu16.04
- QEMU
- arm 交叉编译工具链（如果是想调试 arm 架构的处理器）


## 1.使用 Eclipse 调试 linux 内核的原因

当我们调试 `linux` 内核的时候，我们往往希望可以单步去调试，因为这样可以让我们了解内核的整个执行流程，对于自己感兴趣的模块，自己可以看看内核到底是怎么执行的，追寻内核的执行序列。

一般采用的方案都是自己手头上有一个开发板，然后搭配 `jlink+openpcd` 这样一个组合来进行单步调试，但是往往我们去采购硬件开发板，成本是非常大的，并且我们很多时候并不能采购到自己想要的硬件，比如说你想在硬件上单步调试 `risc-v` 架构，那么目前市面上还没有很好的硬件解决方案供我们去调试学习。因此，我们一般会采用 `QEMU` 来去模拟我们想要的开发环境。

那么当我们使用 `QEMU` 的时候，我们就脱离了硬件环境，一般的调试方案是选择 `QEMU+gdb` 的组合，但是，我们会发现，我们使用 `gdb` 的时候，需要输入命令去打断点，去全速运行或者单步执行，这是一个痛点，还有一个痛点是，我们使用 `gdb` 是很难很好的动态观测某一变量的实时值，每观测一次则需要输入一次命令，那么这就会使得我们调试非常的麻烦。

因此，我们引入 `Eclipse+QEMU+gdb` 的方案，那么我们就可以像使用图像化界面一样，使用图形化界面去打断点，运行，并且可以实时观测到我们想要观察的变量。


## 2.在 Ubuntu 下安装 Eclipse

### 2.1 JDK 的安装

因为 Eclipse 需要依赖 JDK，因此我们需要现在我们的 Ubuntu 中安装 JDK。JDK的安装分成两种办法，一种是使用 `apt` 工具进行安装，一种是从网络上下载 JDK 压缩包进行安装。

#### 2.1.1 使用 apt 安装

我们可以输入以下命令安装 JDK11
```shell
sudo apt install openjdk-11-jdk
```

#### 2.1.2 从网络上下载 JDK 压缩包

可以从 [*JDK 官网*](https://www.oracle.com/java/technologies/downloads/#java11) 中下载 JDK11 的压缩包

![p1](/搭建Eclipse图形化调试linux内核/p1.jpg)

将下载好的 JDK11 压缩包，放入到 Ubuntu 中的任意目录，然后使用以下命令进行操作
```markdown
//1.将 JDK11 解压到 /usr/local/ 目录中
sudo tar -zxvf jdk-11.0.15.1_linux-x64_bin.tar.gz -C /usr/local/
mv jdk-11.0.15.1 jdk11
sudo vi /etc/profile

//2.修改环境变量
按 i 键，进入 vim 编辑模式
在 /etc/profile 文件最后添加以下语句
export JAVA_HOME=/usr/local/jdk11 #修改成自己jdk的安装目录
export CLASSPATH="$JAVA_HOME/lib"
export PATH="$JAVA_HOME/bin:$PATH"

//3.退出编辑模式并保存
按esc键
:wq

//4.重新加载配置文件
source /etc/profile
```

那么我们可以使用如下命令查看 JDK 是否安装成功，如果安装成功会有下图所示
```shell
java -version
```
![p2](/搭建Eclipse图形化调试linux内核/p2.jpg)


### 2.2 下载 Eclipse

那么 Eclipse 的下载我们可以去 [*Eclipse官网*](https://www.eclipse.org/downloads/packages/) 进行下载，需要注意，我们要下载 c/c++ 版本的 Eclipse，如果官网下载太慢了，可以到 [*大连东软 镜像*](https://mirrors.neusoft.edu.cn/eclipse/technology/epp/downloads/release/?C=M&O=D) 下载，特别强调，是需要下载 c/c++ 版本！！！那么下载的文件名字应该类似 `eclipse-cpp-2021-12-R-linux-gtk-aarch64.tar`

![p3](/搭建Eclipse图形化调试linux内核/p3.jpg)

将下载好的 Eclipse 压缩包，放入到 Ubuntu 中的任意目录，然后使用以下命令进行操作，并且打开我们 Eclipse
```shell
sudo tar -xzvf eclipse-cpp-2021-12-R-linux-gtk-x86_64.tar.gz
cd eclipse
./eclipse
```


## 3.配置 Eclipse 调试环境

首先，我们打开 Eclipse 之后，会进入下面这个界面，为了方便管理，我们可以新创建一个工作目录，作为我们 Eclipse 的工作空间，然后点击 `launch` 按键运行。

![p4](/搭建Eclipse图形化调试linux内核/p4.jpg)

当进入 Eclipse 之后，我们可以选择左上角的 `File --> new ---> Makefile project from existing code` 然后会出现下列界面，填写上自己的工程名字，并且选中我们要调试的 linux 内核代码路径。

![p5](/搭建Eclipse图形化调试linux内核/p5.jpg)

然后点击 Eclipse 上方的 `Run --> Debug Configurations`，就会弹出下列界面，我们需要去配置我们的调试选项，在对应栏目填上项目名字，还有内核代码路径，需要注意，这个路径是 `内核代码路径/vmlinux`，这里需要定位到 vmlinux。

![p6](/搭建Eclipse图形化调试linux内核/p6.jpg)

然后我们点击 Debugger 栏目项，并且选择我们调试方式为 gdbserver，选择 gdb 的调试器为交叉编译工具链的 gdb 调试器，这里的交叉编译工具链要根据你实际的交叉编译工具链进行调整。

![p7](/搭建Eclipse图形化调试linux内核/p7.jpg)

接下来我们要调整我们的 gdb 调试端口号，将其设置为 1234，并且 Apply 应用，需要注意，当我们启动了 QEMU 之后，我们再点击 Debug 按键开始调试。

![p8](/搭建Eclipse图形化调试linux内核/p8.jpg)


## 4.调试

我们在要调试的内核目录下，使用下列命令启动 QEMU，需要注意，下列命令只是一个参考，实际的 QEMU 命令要根据自己实际的情况启动。
```shell
qemu-system-arm -nographic -M vexpress-a9 -m 1024M -kernel arch/arm/boot/zImage -append "rdinit=/linuxrc console=ttyAMA0 loglevel=8" -dtb arch/arm/boot/dts/vexpress-v2p-ca9.dtb -S -s
```

启动我们的 Eclipse 和 QEMU 链接，进入调试模式

![p9](/搭建Eclipse图形化调试linux内核/p9.jpg)

输入以下命令即可开始调试，一定需要注意，要先使用 file 命令，加载我们的 vmlinux！！！
```markdown
file xxx/vmlinux    //需要调试的内核的 vmlinux 路径
b do_fork           //可以在自己感兴趣的函数打断点
c                   //全速运行到断点处

```

那么当运行成功的时候，就会出现如下界面，我们可以图形化单步调试 linux 内核，并且可以实时观测变量的值

![p10](/搭建Eclipse图形化调试linux内核/p10.jpg)
