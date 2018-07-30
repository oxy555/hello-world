# PouchContainer "Have a try" with VirtualBox + Ubuntu16.04 env.

## Introduction
PouchContainer是一款由阿里巴巴开源的高效、轻量的企业级富容器引擎技术，具有隔离性强、可移植性高、资源占用少的特性。

PouchContainer 是企业级的容器方案，故只支持 Linux 操作系统，故需要使用虚拟机才能做到本地运行和测试。
## VirtualBox 的安装及镜像导入
- VirtualBox 安装包可通过[官网](https://www.virtualbox.org/wiki/Downloads)或【阿里郎】中的**办公软件管理**进行下载
![](https://i.loli.net/2018/07/30/5b5ed5e30c753.png)

- Linux系统推荐使用一下版本
    - Ubuntu 16.04
    - CentOS 7

> 在本文中的使用的体验版镜像为 Ubuntu 16.04 (64-bit)

### 具体镜像导入步骤
1. 打开 VirtualBox，【新建】一个实例，选择【Linux】-【Ubuntu(64-bit)】，名称可自定义
![](https://i.loli.net/2018/07/30/5b5ed82dc96e8.png)
> 如遇到 Linux 版本选择只能选择 32位 时，可进入宿主机器的 BIOS 中，开启【Virtualization】，然后重启机器
2. 选择【继续】，内存大小选择【1024M】，进入下一步
3. 选择【使用已有的虚拟硬盘文件】，导入体验版镜像，点击【创建】
![](https://i.loli.net/2018/07/30/5b5ed8d04536f.png)

### 服务启动步骤
1. 启动创建好的实例，在登录阶段使用【账号：pouch / 密码：123456】进行登录
![](https://i.loli.net/2018/07/30/5b5eda998850b.png)
2. 切换至 root 用户
``` bash
sudo su
```
3. 检查网络是否正常（示例）
``` bash
ping www.alibaba-inc.com
```
4. 启动 pouch 服务
``` bash
systemctl start pouch
```
5. 启动一个 busybox 基础容器，会生成一个key
``` bash
pouch run -t -d busybox sh
```
6. 登入启动的容器
``` bash
pouch exec -it {ID}
```
> {ID} 为上一步生成的 key 的前六位

完成以上步骤后，新建的基础容器应该已经启动并且能成功登入容器了。
![](https://i.loli.net/2018/07/30/5b5edd8c3f50f.png)

### 开发环境的配置
体验版镜像中已经包含的工具有：vim、make、git、go等基本工具，用户无需在配置。其中 pouch 的源码路径位于 /root/gopath/src/github.com/alibaba/pouch 。

- 对于习惯 Vim 开发的同学，可直接在虚拟机中进行开发（需将 pouch 目录下的文件替换为自己 repo 中 fork 的项目文件）。

- 对于习惯 IDE 开发的同学，可以将代码拉取到宿主机器，而后将宿主机器对应目录挂载到虚拟机中 pouch 的源码目录下。

### GitHub配置
将远端代码 clone 到本地可使用 HTTPS 或 SSH 的方式。

本文推荐使用 SSH 的连接方式，可省去后期向远端提交代码时每次输入密码的繁琐，但需先在 github.com 的个人账号下添加 SSH key，[github 官方文档](https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/)已有很详细的生成及添加方式，本文不予赘述。

在本机中仍需配置为 git 配置 github 账号的相关信息，具体配置如下：
``` git
git config --global user.name YOUR_USERNAME
git config --global user.email YOUR_EMAIL
```
### 宿主机多 github 账号配置
在宿主机器中可能存在配置多 github 账号的需求，可通过配置文件解决：

``` bash
# 通过 ssh-keygen 生成公钥密钥，如生成在 ~/.ssh下
ssh-keygen -t rsa -b 4096 -C "your.email@address.com"

# 将公钥文件添加到对应的站点

# 将私钥添加到 SSH agent 上（文件替换成刚才生成的私钥）
ssh-add -K ~/.ssh/id_rsa 
```
完成以上步骤后，在 ~/.ssh 下创建 config 文件，按照格式写入即可。
- Host 可自定
- HostName 为站点地址
- User 填写为 git
- IdentityFile 为私钥地址
``` bash
# first.github (first@gmail.com)
Host github.com
HostName github.com
User git
IdentityFile ~/.ssh/id_rsa

# second (second@gmail.com)
Host github-second
HostName github.com
User git
IdentityFile ~/.ssh/id_rsa_second
```

### 宿主机器文件夹的挂载
在挂载时，需要用到 VirtualBox 的增强插件，在虚拟机中的安装方式如下：
``` bash
apt install virtualbox-guest-x11
```
在 VirtualBox 中，将宿主机器的工程目录文件夹设置为【共享文件夹】。需选择【自动挂载】和【固定分配】。
![](https://i.loli.net/2018/07/30/5b5eee8226510.png)
进入到 /root/gopath/src/github.com/alibaba/ 目录下，将原有的 pouch 目录下的文件移除（保留 pouch 文件夹），在目录下执行命令进行挂载。（其中 SHARE_FOLDER_NAME 为共享文件夹的名称）
``` bash
sudo mount -t vboxsf SHARE_FOLDER_NAME pouch/
```
可启动 pouch 服务检测配置是否正确。
