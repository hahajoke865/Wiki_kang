## 1.WHY？

我们在写应用时可能会需要调用系统隐藏的 API，这些 API 是我们无法直接访问的，需要通过 classes.jar 来进行这些 API 的访问，采用这种方式的时候，Android Studio 中是无法使用虚拟机来进行调试的，只能通过直接连接硬件来进行调试

## 2.HOW

### 2.1

进入安卓编译输出目录 `out/target/common/obj/JAVA_LIBRARY/framework_intermediates`，找到此目录下编译出来的 `classes.jar` 文件，并且拷贝到 Android Studio 工程的 app\libs 目录下

右键 `classes.jar`，选择 `【add as Library...】`

### 2.2

在 Module 的 build.gradle 中，dependencies 自动增加依赖配置，注释掉下面一行
```markdown
implementation fileTree(dir: 'libs', include: ['*.jar'])
```

将 implementation files('libs/classes.jar') 改为 compileOnly files('libs/classes.jar')

![p1](/AndroidStudio导入classesJar/p1.jpg)
