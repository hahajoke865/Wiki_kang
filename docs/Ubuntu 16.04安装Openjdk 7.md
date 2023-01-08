Ubuntu 14是可以正常安装openjdk 7的，Ubuntu 16则只能安装openjdk 8以上

ubuntu 16安装openjdk，网上比较常见的，给出了两种方法，均无法实施

网上方法一(过时)：
```shell
sudo add-apt-repository ppa:openjdk-r/ppa
sudo apt-cache search openjdk
sudo apt update
sudo apt install openjdk-7-jdk
```

这种方法已过时，目前已经无法使用。

网上方法二：
离线安装openjdk 7，此方法颇为繁琐，完成很不容易，并且很容易出错，把系统UI挂掉(重启无法进入UI系统，可用tty)。

原因1：openjdk 7依赖特别多(稍后贴出依赖日志)，并且还递归，一个依赖又依赖另一个或多个依赖，相当繁琐。

原因2：openjdk 7版本较旧，一些依赖需要低版本，而在线方式可能会安装依赖高版本，解决相当繁琐

## Ubuntu 16.04 Server

我测试发现ubuntu 14是可以正常安装openjdk 7,然后把ubuntu 16的源改成ubuntu 14的源，更新之后，就可以正常通过在线方式安装openjdk 7了。

下面是Ubuntu 14的源文件: sources.list
```markdown
#deb cdrom:[Ubuntu 14.04.6 LTS _Trusty Tahr_ - Release amd64 (20190304.5)]/ trusty main restricted

# See http://help.ubuntu.com/community/UpgradeNotes for how to upgrade to
# newer versions of the distribution.
deb http://cn.archive.ubuntu.com/ubuntu/ trusty main restricted
deb-src http://cn.archive.ubuntu.com/ubuntu/ trusty main restricted

## Major bug fix updates produced after the final release of the
## distribution.
deb http://cn.archive.ubuntu.com/ubuntu/ trusty-updates main restricted
deb-src http://cn.archive.ubuntu.com/ubuntu/ trusty-updates main restricted

## N.B. software from this repository is ENTIRELY UNSUPPORTED by the Ubuntu
## team. Also, please note that software in universe WILL NOT receive any
## review or updates from the Ubuntu security team.
deb http://cn.archive.ubuntu.com/ubuntu/ trusty universe
deb-src http://cn.archive.ubuntu.com/ubuntu/ trusty universe
deb http://cn.archive.ubuntu.com/ubuntu/ trusty-updates universe
deb-src http://cn.archive.ubuntu.com/ubuntu/ trusty-updates universe

## N.B. software from this repository is ENTIRELY UNSUPPORTED by the Ubuntu
## team, and may not be under a free licence. Please satisfy yourself as to
## your rights to use the software. Also, please note that software in
## multiverse WILL NOT receive any review or updates from the Ubuntu
## security team.
deb http://cn.archive.ubuntu.com/ubuntu/ trusty multiverse
deb-src http://cn.archive.ubuntu.com/ubuntu/ trusty multiverse
deb http://cn.archive.ubuntu.com/ubuntu/ trusty-updates multiverse
deb-src http://cn.archive.ubuntu.com/ubuntu/ trusty-updates multiverse

## N.B. software from this repository may not have been tested as
## extensively as that contained in the main release, although it includes
## newer versions of some applications which may provide useful features.
## Also, please note that software in backports WILL NOT receive any review
## or updates from the Ubuntu security team.
deb http://cn.archive.ubuntu.com/ubuntu/ trusty-backports main restricted universe multiverse
deb-src http://cn.archive.ubuntu.com/ubuntu/ trusty-backports main restricted universe multiverse

deb http://security.ubuntu.com/ubuntu trusty-security main restricted
deb-src http://security.ubuntu.com/ubuntu trusty-security main restricted
deb http://security.ubuntu.com/ubuntu trusty-security universe
deb-src http://security.ubuntu.com/ubuntu trusty-security universe
deb http://security.ubuntu.com/ubuntu trusty-security multiverse
deb-src http://security.ubuntu.com/ubuntu trusty-security multiverse

## Uncomment the following two lines to add software from Canonical's
## 'partner' repository.
## This software is not part of Ubuntu, but is offered by Canonical and the
## respective vendors as a service to Ubuntu users.
# deb http://archive.canonical.com/ubuntu trusty partner
# deb-src http://archive.canonical.com/ubuntu trusty partner

## This software is not part of Ubuntu, but is offered by third-party
## developers who want to ship their latest software.
deb http://extras.ubuntu.com/ubuntu trusty main
deb-src http://extras.ubuntu.com/ubuntu trusty main
```

把这个源替换ubuntu 16的源：
```markdown
sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak
sudo vim /etc/apt/sources.list
# 删除原有内容，替换为ubuntu 14的源内容.
# apt-get已过时，使用apt.
sudo apt update
sudo apt install openjdk-7-jdk
```

这样Ubuntu Server就可以正常安装openjdk 7了。

上面是ubuntu 自带的中国源，你也可以使用清华，中科大等提供的源，清华提供的源可以访问这里获取:

[https://mirrors.tuna.tsinghua.edu.cn/help/ubuntu/].


##  Ubuntu 16.04.7 desktop

1, 首先备份apt源，然后替换为Ubuntu 14.04的源：
```markdown
sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak
sudo vim /etc/apt/sources.list
# 替换为ubuntu 14.04的源.
sudo apt update
```