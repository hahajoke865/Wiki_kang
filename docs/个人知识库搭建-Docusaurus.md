
准备：

- 英语基础（一点点即可）
- 基于百度/google的随机应变能力
- 科学上网的条件  

## 1.Docusaurus搭建

### 1.1安装Node.js

访问[Node.js官网](https://nodejs.org/zh-cn/)，下载并安装Node.js


```markdown
使用 Docusaurus2 需要注意下面两个注意事项：
1、Node.js 16.14 或更高版本（可以通过执行 node -v 命令来查看当前所用的 Node。js 版本）。你可以使用 nvm 管理同一台计算机上安装的多个 Node 版本。

2、当安装 Node.js 时，建议选中与依赖项相关的所有复选框。
```

### 1.2安装vscode  

安装 vs code 可以使得我们更好编写本地的笔记文章，并可作为本地编辑器使用

访问[vs code官网](https://code.visualstudio.com/)，下载并安装 vs code

为了使用 vs code 可以更加的方便，我们可以选择安装以下 vs code 的插件

- [*Chinese (Simplified) Language Pack*](https://marketplace.visualstudio.com/items?itemName=MS-CEINTL.vscode-language-pack-zh-hans): 用于汉化我们的 vs code
- [*Markdown All in One*](https://marketplace.visualstudio.com/items?itemName=yzhang.markdown-all-in-one): 让我们的 vs code 提供更多的 vs code 语法支持


### 1.3安装 Docusaurus2 脚手架

创建我们本地的网站项目目录

例如：如果我们想在电脑的 F 盘下创建一个名为 `mysite` 的文件夹作为我们知识库的本地部署，那么我们可以在 vs code 内选择 `文件`->`打开文件夹`，选中我们 F 盘下的 `mysite`

使用 npx 初始化网站：该工具可帮助您搭建 Docusaurus 网站的雏形。你可以在新的空仓库中或已有的仓库中的运行此命令，它将创建一个包含脚手架文件的新目录。
```shell
npx create-docusaurus@latest [name] [template]
```

例如：如果我的知识库项目文件夹的名字为 `mysite`，那么我们就使用 `mysite` 去替换 `[name]`，`[template]` 指的是知识库项目的模板主题，根据[*Docusaurus 官方文档*](https://www.docusaurus.cn/docs/installation)，这里我们将 `[template]` 替换为 `classic `，即可。所以此命令变成了：
```shell
npx create-docusaurus@latest mysite classic
```

接着，我们在 vs code `F:` 目录内打开对应的终端，把上面的代码复制粘贴进来，敲击回车执行，耐心等待执行结果完成
```markdown
注意事项：
        若执行过程中有警告出现，但是并没有出现报错停止，我们可以暂时不用理会，不会影响后续的操作，并且注意，安装 Docusaurus2，Node.js 需使用 16.14 或更高版本
```

构造完成之后，你会在 `[name]`，在本例子里面就是 `F:` 目录下，你会得到如下项目架构
```markdown
my-website
├── blog
│   ├── 2019-05-28-hola.md
│   ├── 2019-05-29-hello-world.md
│   └── 2020-05-30-welcome.md
├── docs
│   ├── doc1.md
│   ├── doc2.md
│   ├── doc3.md
│   └── mdx.md
├── src
│   ├── css
│   │   └── custom.css
│   └── pages
│       ├── styles.module.css
│       └── index.js
├── static
│   └── img
├── docusaurus.config.js
├── package.json
├── README.md
├── sidebars.js
└── yarn.lock
```

当加载完成后，我们在 vs code 终端内使用命令切换到 Docusaurus 生成的网站文件夹目录
```shell
cd [name]（在本例子中即为 "cd mysite"）
```

接着，执行以下命令，运行针对开发环境的服务器，默认情况下，浏览器将打开 http://localhost:3000 地址，如果一切步骤完成顺利，那么则可以预览生成的 Docusaurus 网站，至此，Docusaurus 本地部署完成
```shell
npm run start
```

## 2.将 Docusaurus 部署到云端  

上述步骤 `1` 只是将我们的 Docusaurus 在本地部署了，即只有本地可以访问创建的 Docusaurus 知识库项目，而无法在互联网上访问到此站点，因此，我们需要将我们的 Docusaurus 知识库项目部署到云端，这里采用的是 Vercel+GitHub 的方式来进行云端部署，若部署顺利，则可让其他用户从互联网上进行访问。


### 2.1注册 GitHub 账户

在 [*GitHub官网*](https://github.com/join) 注册自己的 GitHub 账户


### 2.2创建 GitHub 仓库

此仓库用于在云端保存我们的 Docusaurus 知识库项目

![p3](/Efficiency_and_Miscellaneous/p3.jpg)


### 2.3安装 Git

从 [*Git 官网*](https://git-scm.com/downloads) 下载 Git 软件，并完成安装

重启 vs code，打开终端，使用以下命令初始化 Git，并进行验证，其实就是将本地 Git 与云端 GitHub 绑定
```shell
git config --global user.name "username"
git config --global user.email "email@example.com"
```


### 2.4将本地 Docusaurus 知识库项目推送到 Github

可使用下列命令进行推送，推送到 `2.2` 步骤中创建的GitHub 仓库

```shell
git init
git config core.autocrlf false  //解决 warning: LF will be replaced by CRLF in ** 警告
git add .
git remote add origin git@github.com:system/system.git  //git@github.com:system/system.git为你的仓库地址
git commit -m "xxx"
git branch -M main
git push -u origin main  //一定要注意，是 push 到 main 分支!

```


### 2.5注册并登录 [*Vercel*](https://vercel.com/) 账户

我们可以根据以下图片所示，点击注册并且登录我们的 Vercel 账户

![p1](/Efficiency_and_Miscellaneous/p1.png)

将 Vercel 跟我们的 GitHub 知识库仓库绑定

![p2](/Efficiency_and_Miscellaneous/p2.jpg)


### 2.6使用 Vercel 生成我们的博客页面

点击 Deploy 即可生成我们的博客页面

![p4](/Efficiency_and_Miscellaneous/p4.jpg)


## 3.FAQ

### 3.1 Vercel 分支问题

一般我们都会习惯性把我们的本地仓库推送到 GitHub 的 master 分支，但是 Vercel 默认是使用 main 分支来生成我们的页面的，所以这里千万要注意！


### 3.2 warning: LF will be replaced by CRLF in ** 的原因及解决办法

当我们使用 `git add .` 命令之前，我们先使用 `git config core.autocrlf false` 命令即可


### 3.3解决 Failed to connect to github.com port 443:connection timed out

在推送时我们可能会遇到这个错误，那么可以使用以下命令解决
```shell
取消代理：
        git config --global --unset http.proxy
        git config --global --unset https.proxy
```


### 3.4后续推送
```shell
git add .
git config core.autocrlf false
git commit -m "xxxx"
git push origin main

```



## 参考致谢
- [Doc：Docusaurus](https://www.docusaurus.cn/docs/installation)



