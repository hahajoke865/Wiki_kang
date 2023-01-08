准备：
- 虚拟机 Ubuntu Linux 16.04（其他版本也可）
- Android5.0.2 源码

## 1.安装 JDK

### 1.1 安装 JDK8 的方式
如果编译比较高版本的 Android 源码，那么可以使用以下命令安装 JDK8
```shell
sudo apt-get update
sudo apt-get install openjdk-8-jdk
```

### 1.2 安装 JDK7 的方式
如果编译的是 Android5.0.2 版本的源码，那么 Android5.0.2 是不支持 JDK8 的，他只支持 JDK7，但是很不幸，我们没有办法在 Ubuntu16.04 通过 apt 安装 JDK7，因为在 apt 树里面已经把 JDK7 除掉了，因此，需要我们手动安装，访问 [*openjdk*](https://jdk.java.net/java-se-ri/7) 的官网可以找到 openjdk7 的安装包，点击下图 GNU 版本进行下载

![p1](/Android系统编译/p1.jpg)

#### 1.2.1 配置JAVA环境变量

下载完成后解压这个压缩包到自己 Ubuntu 16.04 的目录
```shell
sudo tar zxvf openjdk-7u75-b13-linux-x64-18_dec_2014.tar.gz -C /opt
```

然后配置环境变量
```markdown
//修改profile文件
sudo gedit /etc/profile

//加入下面的配置
export JAVA_HOME=/opt/java-se-7u75-ri
export JRE_HOME=${JAVA_HOME}/jre 
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib 
export PATH=${JAVA_HOME}/bin:$PATH

//更新profile文件
source /etc/profile
```

可以使用 `java -version` 命令查看是否安装成功，下图为安装成功示例

![p2](/Android系统编译/p2.jpg)


## 2.编译 Android5.0.2 源码

### 2.1 下载依赖包

使用 ubuntu 14+，需要安装以下依赖包
```shell
sudo apt-get install git-core gnupg flex bison gperf build-essential zip curl zlib1g-dev gcc-multilib g++-multilib libc6-dev-i386 lib32ncurses5-dev x11proto-core-dev libx11-dev lib32z-dev ccache libgl1-mesa-dev libxml2-utils xsltproc unzip
```

### 2.2 初始化环境

进入 Android5.0.2 源码根目录，使用以下命令初始化环境
```shell
. setenv
```

### 2.3 选择编译目标

lunch命令是 envsetup.sh 里定义的一个命令，用来让用户选择编译目标，会有以下信息输出，然后选择编译目录，输入对应编译目录前面的数字进行选择，比如，选择 `full_tiny4412-eng`，则输入 `14`
```markdown
You're building on Linux
Lunch menu... pick a combo:
 1. aosp_arm-eng
 2. aosp_arm64-eng
 3. aosp_mips-eng
 4. aosp_mips64-eng
 5. aosp_x86-eng
 6. aosp_x86_64-eng
 7. full_tiny4412-userdebug
 8. full_tiny4412-eng
 9. aosp_mako-userdebug
 10. aosp_hammerhead-userdebug
 11. aosp_tilapia-userdebug
 12. aosp_grouper-userdebug
 13. aosp_deb-userdebug
 14. full_fugu-userdebug
 15. aosp_fugu-userdebug
 16. aosp_flo-userdebug
 17. aosp_shamu-userdebug
 18. aosp_manta-userdebug
 19. mini_emulator_mips-userdebug
 20. mini_emulator_x86_64-userdebug
 21. mini_emulator_arm64-userdebug
 22. m_e_arm-userdebug
 23. mini_emulator_x86-userdebug

Which would you like? [aosp_arm-eng] 
```

### 2.4 编译

使用以下命令进行编译，也可以在 make 后加上“-j”参数利用 CPU 的多核加快编译速度，比如在 4 核 CPU 上可以执行 `make –j4`。注意，编译过程可能持续 4、5 个小时。
```shell
make
```

### 2.5 生成映象文件

执行以下命令生成 system.img 映象文件
```shell
./gen-img.sh
```

## 3.FAQ

### 3.1

按照上面 1.2.1 配置后会出现两个问题
```markdown
Q1、使用 java 版本选择工具可能无法识别到 jdk1.7
Q2、进行 make 编译 Android5.0.2 源码会提示 jdk1.7 是不能识别的 openJDK1.7，这里主要是 makefile 里的正则表达式判断失误，所以需要修改 makefile
```

修改安卓源码目录下的 /build/envsetup.sh
```shell
sudo gedit build/envsetup.sh
# 搜索  ANDROID_SET_JAVA_HOME
# 将刚才 1.2.1 步骤解压的 jdk 路径粘贴到下方的 JAVA_HOME 变量中
```

![p3](/Android系统编译/p3.jpg)

注释掉版本检测的终止语句
```shell
# 编辑主make文件
sudo gedit build/core/main.mk
# 注释掉终止命令
# $(error stop)
```

![p4](/Android系统编译/p4.jpg)

保存后按照正常的编译命令进行 `make` 编译 Android5.0.2 源码就可以了。虽然终端还会打印 jdk 不匹配的信息，但是 `make` 编译 Android5.0.2 源码过程中不会再产生 openjdk7 产生的问题了。


### 3.2
出现如下错误
```markdown
Install: out/host/linux-x86/bin/apicheck
Checking API: checkapi-last
Checking API: checkapi-current
out/target/common/obj/PACKAGING/public_api.txt:20: error 5: Added public field android.Manifest.permission.BACKUP
out/target/common/obj/PACKAGING/public_api.txt:81: error 5: Added public field android.Manifest.permission.INVOKE_CARRIER_SETUP
out/target/common/obj/PACKAGING/public_api.txt:105: error 5: Added public field android.Manifest.permission.READ_PRIVILEGED_PHONE_STATE
out/target/common/obj/PACKAGING/public_api.txt:115: error 5: Added public field android.Manifest.permission.RECEIVE_EMERGENCY_BROADCAST

******************************
You have tried to change the API from what has been previously approved.

To make these errors go away, you have two choices:
   1) You can add "@hide" javadoc comments to the methods, etc. listed in the
      errors above.

   2) You can update current.txt by executing the following command:
         make update-api

      To submit the revised current.txt to the main Android repository,
      you will need approval.
******************************



build/core/tasks/apicheck.mk:57: recipe for target 'out/target/common/obj/PACKAGING/checkapi-current-timestamp' failed
make: *** [out/target/common/obj/PACKAGING/checkapi-current-timestamp] Error 38

 make failed to build some targets (11:21 (mm:ss)) 
```

按照错误提示，执行以下命令
```shell
make update-api
```

出现如下语句，并不能说明整个工程编译成功。仅是 make update-api 成功，然后要重新执行 `make` 命令重新编译
```markdown
#### make completed successfully (10:45 (mm:ss)) ####
```

### 3.3

发现如下错误：
```markdown
clang: error: linker command failed with exit code 1 (use -v to see invocation)
build/core/host_shared_library_internal.mk:44: recipe for target 'out/host/linux-x86/obj32/lib/libc++.so' failed
make: *** [out/host/linux-x86/obj32/lib/libc++.so] Error 1

#### make failed to build some targets (09:25 (mm:ss)) ####

```

执行如下命令,然后使用 `make` 命令继续编译
```shell
cp /usr/bin/ld.gold prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.11-4.6/x86_64-linux/bin/ld
```

### 3.4

发现如下错误
```markdown
prebuilts/misc/linux-x86/bison/bison -d  -o out/host/linux-x86/obj32/EXECUTABLES/aidl_intermediates/aidl_language_y.cpp frameworks/base/tools/aidl/aidl_language_y.y
prebuilts/misc/linux-x86/bison/bison: error while loading shared libraries: libstdc++.so.6: cannot open shared object file: No such file or directory
build/core/binary.mk:537: recipe for target 'out/host/linux-x86/obj32/EXECUTABLES/aidl_intermediates/aidl_language_y.cpp' failed
make: *** [out/host/linux-x86/obj32/EXECUTABLES/aidl_intermediates/aidl_language_y.cpp] Error 127
make: *** 正在等待未完成的任务....
注: 某些输入文件使用了未经检查或不安全的操作。
注: 有关详细信息, 请使用 -Xlint:unchecked 重新编译。
注: 某些输入文件使用了未经检查或不安全的操作。
注: 有关详细信息, 请使用 -Xlint:unchecked 重新编译。
注: 某些输入文件使用或覆盖了已过时的 API。
注: 有关详细信息, 请使用 -Xlint:deprecation 重新编译。
注: 某些输入文件使用了未经检查或不安全的操作。
注: 有关详细信息, 请使用 -Xlint:unchecked 重新编译。

#### make failed to build some targets (04:01 (mm:ss)) ####

```

原因：ubuntu 64位系统运行32位程序的问题，需要安装运行32位程序的兼容库，安装之后继续 `make` 编译
```shell
sudo apt-get install lib32ncurses5
sudo apt-get install lib32stdc++6
```

### 3.5

报如下错误
```markdown
/bin/bash: gperf: 未找到命令
external/chromium_org/third_party/WebKit/Source/platform/make_platform_generated.target.linux-arm.mk:48: recipe for target 'out/target/product/tiny4412/obj/GYP/shared_intermediates/blink/platform/ColorData.cpp' failed
make: *** [out/target/product/tiny4412/obj/GYP/shared_intermediates/blink/platform/ColorData.cpp] Error 127

#### make failed to build some targets (03:29:10 (hh:mm:ss)) ####

```

安装 gperf，安装之后继续 `make` 编译
```shell
sudo apt-get install gperf
```

### 3.6

发现如下错误
```markdown
/bin/bash: xmllint: 未找到命令
build/core/Makefile:34: recipe for target 'out/target/product/tiny4412/system/etc/apns-conf.xml' failed
make: *** [out/target/product/tiny4412/system/etc/apns-conf.xml] Error 127
make: *** 正在等待未完成的任务....
warning: no entries written for drawable/ab_solid_shadow_mtrl (0x010800bb)
warning: no entries written for drawable/ab_transparent_mtrl_alpha (0x010800c4)
warning: no entries written for drawable/btn_check_off_mtrl_alpha (0x010800f8)
warning: no entries written for drawable/btn_check_on_mtrl_alpha (0x0108010e)
warning: no entries written for drawable/btn_radio_off_mtrl_alpha (0x010801b0)
warning: no entries written for drawable/btn_radio_off_pressed_mtrl_alpha (0x010801b4)
warning: no entries written for drawable/switch_off_mtrl_alpha (0x010806b1)
warning: no entries written for drawable/switch_on_mtrl_alpha (0x010806b2)
warning: no entries written for drawable/sym_keyboard_delete_holo (0x010806c5)

#### make failed to build some targets (11:25 (mm:ss)) ####

```

安装如下软件，安装之后继续 `make` 编译
```shell
sudo apt-get  install libxml2-utils
```

### 3.7 

发现如下错误
```markdown
  File "../build/scripts/rule_bison.py", line 82, in <module>
    returnCode = subprocess.call([bisonExe, '-d', '-p', prefix, inputFile, '-o', outputCpp])
  File "/usr/lib/python2.7/subprocess.py", line 523, in call
    return Popen(*popenargs, **kwargs).wait()
  File "/usr/lib/python2.7/subprocess.py", line 711, in __init__
    errread, errwrite)
  File "/usr/lib/python2.7/subprocess.py", line 1343, in _execute_child
    raise child_exception
OSError: [Errno 2] No such file or directory
external/chromium_org/third_party/WebKit/Source/core/make_core_generated.target.linux-arm.mk:414: recipe for target 'out/target/product/tiny4412/obj/GYP/shared_intermediates/blink/core/CSSGrammar.cpp' failed
make: *** [out/target/product/tiny4412/obj/GYP/shared_intermediates/blink/core/CSSGrammar.cpp] Error 1

```

安装如下软件，安装之后继续 `make` 编译
```shell
sudo apt-get install bison 
```

### 3.7 安装了 JDK7，但是依然版本报错

打开 /etc/profile 文件，配置 OpenJDK
```shell
sudo gedit /etc/profile
```

在末尾追加下面代码
```bash
export JAVA_HOME=/usr/lib/jvm/java-7-openjdk-amd64
export JRE_HOME=${JAVA_HOME}/jre 
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib 
export PATH=${JAVA_HOME}/bin:$PATH
```

修改了/etc/profile 文件需要重启才能生效，但使用下面命令可以在不重启的情况下在当前 bash 环境生效，如果不行，重启 ubuntu 即可
```shell
source /etc/profile
```

### 3.8 com.sun.javadoc does not exist

配置 ANDROID_JAVA_HOME 环境变量，配置成和我们的 JDK7 一样的路径，打开 /etc/profile 文件，在末尾追加下面代码
```markdown
 export ANDROID_JAVA_HOME=$JAVA_HOME
```

![p5](/Android系统编译/p5.jpg)