---
layout: post
title: 腾讯云安装 Rstudio Server 与 Shiny Server
---

最近有一个数据展示的项目，使用 Shiny 进行展示，为了方便小组成员交流，就部署到腾讯云上了。不难，但是坑不少，因此记录一下。参考了知乎专栏文章 [在 Google 云计算平台上使用 Rstudio Server](https://zhuanlan.zhihu.com/p/23142231)[^1]。

#### 申请腾讯云

不一定非得是 [腾讯云](https://www.qcloud.com)，哪家都可以，只是鹅厂的『云 + 校园计划』正好符合我。

服务器操作系统选择 Linux，千万别选 Windows，建议 Ubuntu，Centos 也行，但一定要是 64 位，因为现在很多软件都不支持 32 位。安全组策略选择全部放行，不然后面配置起来很麻烦。

按照腾讯云的文档登陆服务器，尽量不要选择浏览器登陆。如果本机是 Linux，直接：

```bash
ssh -q -l [云服务器登录账号] -p 22 [云服务器的公网IP]
```

#### 安装最新版本 R

坑爹的地方在于，默认不是最新的 R，而 Shiny 不支持老版本，因此要更新到最新版本。参照 [Installing R latest version](http://blogs.helsinki.fi/bioinformatics-viikki/documentation/getting-started-with-r-programming/installingrlatest/) 这个教程[^2]升级 R，并安装中文字体，代码如下：

```bash
sudo sh -c "echo 'deb https://mirrors.tuna.tsinghua.edu.cn/CRAN/bin/linux/ubuntu trusty/' >>/etc/apt/sources.list"
sudo apt-get update
sudo apt-get install r-base
sudo apt-get install language-pack-zh-hans
sudo apt-get install xfonts-wqy
```

#### Rstudio Server 的安装与使用

**安装**

必须先安装 gdebi，然后用 wget 下载 deb 包，但是非常坑爹的一点是腾讯云从 Rstudio 官网下载 Rstudio Server 安装包非常慢，我把包上传到坚果云了，可以从坚果云下载。

官网[^3]下载：

```bash
sudo apt-get install gdebi-core
wget "https://download2.rstudio.org/rstudio-server-1.0.44-amd64.deb"
sudo gdebi rstudio-server-1.0.44-amd64.deb
```

从坚果云下载：

在此 [网页](https://www.jianguoyun.com/p/DbGRH84Q2JzwBRivhB8) 点击下载，然后取消，点击右键复制下载地址，替换掉上述 `wget` 后面的网址。

**使用**

Rstudio Server 默认不能使用管理员账户，因此，必须新建账户，这里创建账户 test。

```bash
sudo adduser test
```

然后就可以用浏览器打开 `http://<server-ip>:8787`，server-ip 是你的腾讯云公网 ip。之后输入账户和密码就可以使用 Rstudio Server 了，可以使用我的测试帐号，不过好像每次只能单人登陆：

网址：<http://123.207.156.73:8787/>

帐号：test

密码：test2016

进去之后，最好首先把软件源改为国内的，否则下载软件包的速度感人。修改方法很简单，菜单栏“Tools --> Global option --> Packages --> CRAN mirror”，选择 “China (Beijing) [https] - TUNA Team, Tsinghua University”。

Rstudio Server 与桌面版相比，可以在 Files 窗口下直接上传数据和文件。新建项目 test，然后上传文件，在 Rstudio 里测试之后就可以进行 Shiny 展示了。

#### 安装 Shiny Server

**安装**

必须安装最新版 R，否则不能正常运行。登陆腾讯云服务器，安装 shiny，之后下载 Shiny Server 并安装[^4]。

```bash
sudo su - \
-c "R -e \"install.packages('shiny', repos='https://cran.rstudio.com/')\""
wget "https://download3.rstudio.org/ubuntu-12.04/x86_64/shiny-server-1.5.1.834-amd64.deb"
sudo gdebi shiny-server-1.5.1.834-amd64.deb
```

同理，如果速度太慢，可以从坚果云下载：

在此 [网页](https://www.jianguoyun.com/p/DaN0kvAQ2JzwBRjrkh8#) 点击下载，然后取消，点击右键复制下载地址，替换掉上述 `wget` 后面的网址。

**使用**

最后一步，将 test 用户的 test 项目部署到服务器：

```bash
sudo cp -R /home/test/test/ /srv/shiny-server/
```

然后访问 `http://<hostname>:3838/APP_NAME/` 就可以了，比如，我这里就是 <http://123.207.156.73:3838/test/>。

#### 参考资料

[^1]: [在 Google 云计算平台上使用 Rstudio Server](https://zhuanlan.zhihu.com/p/23142231)
[^2]: [Installing R latest version](http://blogs.helsinki.fi/bioinformatics-viikki/documentation/getting-started-with-r-programming/installingrlatest/)
[^3]: [RStudio Server: Getting Started](https://support.rstudio.com/hc/en-us/articles/200552306-Getting-Started)
[^4]: [GitHub: Shiny Server](https://github.com/rstudio/shiny-server)
