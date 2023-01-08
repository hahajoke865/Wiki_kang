## 1.WHY？

一般如果不安装双系统，但是又想在一个系统上模拟另外一个系统，那么我们可能就需要安装 vmware 进行虚拟机的安装并且使用，但是往往我们一直往 vmware 中添加文件，就会发现，vmware 的磁盘空间只增不减？？？即使你的虚拟机系统中删除了某些文件，并且在虚拟机系统中观测磁盘使用情况是有很多盈余的，但是往往我们 windows 主机端的磁盘却没有变化？只增不减，因此这里提出几种解决方案。

并且，在 vmware 中，如果主机和虚拟机的文件传输不是使用 ftp服务器等方式进行互传，一般会采用 `VM tools` 工具，也就是可以通过拖拽的方式把文件拉入虚拟机之中。但每一次拖拽，其实都是现在 cache 文件夹里面生成一个同样的文件，并使用 cp 拷贝的方式将其拷贝到拖拽放置的目录中。

因此，如果不进行清理的话，cache文件夹中产生的文件，并不会自动删除或者释放。

该文件夹位于用户目录下/home/xxxx/.cache/vmware/drag_and_drop

![p1](/vmware磁盘释放/p1.jpg)

## 2.DO

### 2.1 使用 vmware 的压缩方案

使用虚拟机->设置->硬盘->碎片整理/压缩，这是使用 vmware 自带的压缩方案进行磁盘的释放，虽然可以释放出一部分的磁盘空间，但是释放效果甚微，亲测一般只能释放几个 G 的空间，并且一次释放后则几乎无法再次成功释放出更多的空间。

![p2](/vmware磁盘释放/p2.jpg)

### 2.2 使用虚拟机压缩+主机压缩方案

此方案可以真正大量释放出主机的磁盘空间，首先，如果你是使用上述提及到的采用 `VM tools` 工具的方案，在主机端和虚拟机端互传文件，该文件夹是宿主机与虚拟机间交互后产生缓存的地方，那么，我们可以使用以下命令把 `/home/xxxx/.cache/vmware/drag_and_drop` 这个缓存文件给删掉
```markdown
# 查找文件夹的完整路径
$ find / -name 'drag_and_drop'
# 删除文件夹
$ rm -rf /home/xxx/.cache/vmware/drag_and_drop
```

接下来是填充虚拟机的空间，猜测虚拟机在进行磁盘读写的时候，可能只操作扇区，不考虑磁盘中文件系统的类型和内容，只有当数据全为0时空间才能被释放
```markdown
#填充空间
$dd if=/dev/zero of=/zero.file bs=2M
#将文件同步到磁盘
sudo sync
#删除填充文件
rm -rf /zero.file
```

接下来是收缩虚拟机根目录，但是可能会报出虚拟磁盘空间错误不足的情况，如果出现这种情况，无需理会
```shell
$ /usr/bin/vmware-toolbox-cmd disk shrink /
```

在主机端（windows）使用 vmware-vdiskmanager.exe 虚拟机磁盘管理工具进行压缩操作
```markdown
VMware Virtual Disk Manager - build 3272444.
Usage: vmware-vdiskmanager.exe OPTIONS <disk-name> | <mount-point>
Offline disk manipulation utility
  Operations, only one may be specified at a time:
     -c                   : create disk.  Additional creation options must be specified.  Only local virtual disks can be created.
     -d                   : defragment the specified virtual disk. Only local virtual disks may be defragmented.
     -k                   : shrink the specified virtual disk. Only local virtual disks may be shrunk.
     -n <source-disk>     : rename the specified virtual disk; need to specify destination disk-name. Only local virtual disks may be renamed.
     -p                   : prepare the mounted virtual disk specified by the mount point for shrinking.
     -r <source-disk>     : convert the specified disk; need to specify destination disk-type.  For local destination disks the disk type must be specified.
     -x <new-capacity>    : expand the disk to the specified capacity. Only local virtual disks may be expanded.
     -R                   : check a sparse virtual disk for consistency and attempt to repair any errors.
     -e                   : check for disk chain consistency.
     -D                   : make disk deletable.  This should only be used on disks that have been copied from another product.

  Other Options:
     -q                   : do not log messages

  Additional options for create and convert:
     -a <adapter>         : (for use with -c only) adapter type (ide, buslogic, lsilogic). Pass lsilogic for other adapter types.
     -s <size>            : capacity of the virtual disk
     -t <disk-type>       : disk type id

  Disk types:
      0                   : single growable virtual disk
      1                   : growable virtual disk split in 2GB files
      2                   : preallocated virtual disk
      3                   : preallocated virtual disk split in 2GB files
      4                   : preallocated ESX-type virtual disk
      5                   : compressed disk optimized for streaming
      6                   : thin provisioned virtual disk - ESX 3.x and above

     The capacity can be specified in sectors, KB, MB or GB.
     The acceptable ranges:
                           ide/scsi adapter : [1MB, 8192.0GB]
                           buslogic adapter : [1MB, 2040.0GB]
        ex 1: vmware-vdiskmanager.exe -c -s 850MB -a ide -t 0 myIdeDisk.vmdk
        ex 2: vmware-vdiskmanager.exe -d myDisk.vmdk
        ex 3: vmware-vdiskmanager.exe -r sourceDisk.vmdk -t 0 destinationDisk.vm
dk
        ex 4: vmware-vdiskmanager.exe -x 36GB myDisk.vmdk
        ex 5: vmware-vdiskmanager.exe -n sourceName.vmdk destinationName.vmdk
        ex 6: vmware-vdiskmanager.exe -r sourceDisk.vmdk -t 4 -h esx-name.mycompany.com \
              -u username -f passwordfile "[storage1]/path/to/targetDisk.vmdk"
        ex 7: vmware-vdiskmanager.exe -k myDisk.vmdk
        ex 8: vmware-vdiskmanager.exe -p <mount-point>
              (A virtual disk first needs to be mounted at <mount-point>)

```

我们可以看到 -k 是用于压缩指定的本地虚拟磁盘的参数因此，我们使用 vmware-vdiskmanager 进行压缩空间,打开主机端 cmd 命令窗口，使用以下命令，xxx 为主机端 vmware 的安装目录，yyy 为vmware 虚拟机端真正存放的目录，myDisk.vmdk 为你要压缩的虚拟磁盘，一般为 `Ubuntu-xxx.vmdk`
```shell
xxx/vmware-vdiskmanager.exe -k "yyy/myDisk.vmdk"
```