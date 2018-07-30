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

### Development environment configuration
The tools included in the experience image are: vim, make, git, go and other basic tools. And the source code of pouch is located at /root/gopath/src/github.com/alibaba/pouch.

- For users who are familiar with Vim development, you can develop in the virtual machine (replace the files in the pouch directory with the project files they folk to your own repository).

- For users who are familiar with IDE development, you can pull the code to the host machine and then mount the directory to the source directory of the pouch in the virtual machine.

### GitHub configuration
Use HTTPS or SSH to clone the remote code to local. 

SSH is recommended here, which can save the trouble of entering the password every time you submit the code to the remote end. But the SSH key need to be added under your personal github account first. Details of generating and adding SSH key can refer to the [github official document](https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/)

On host machine, you still need to configure your github account as follows:

``` git
git config --global user.name YOUR_USERNAME
git config --global user.email YOUR_EMAIL
```
### multiple github accounts configuration on host machines
There may be a need to configure multiple github accounts on the host machine, which can be resolved through the configuration file:

``` bash
# Generate a public key by ssh-keygen, for example, generate under ~/.ssh
ssh-keygen -t rsa -b 4096 -C "your.email@address.com"

# Add the public key file to the corresponding site

# Add the private key to the SSH agent (replace the file with the private key just generated)
ssh-add -K ~/.ssh/id_rsa 
```
After completing the steps above, create a config file under ~/.ssh and write it in the following format: 
- Host: can be customized
- HostName: the site address
- User: git
- IdentityFile: the private key address
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

### File mounting on host machine 
When mounting, enhanced plug-in for VirtualBox are needed, which can be installed as follows:
``` bash
apt install virtualbox-guest-x11
```
In VirtualBox，Set the project directory folder of the host machine to `shared folder`, and choose `automatic mount` and `fixed assignment`。
![](https://i.loli.net/2018/07/30/5b5eee8226510.png)
go to the directory /root/gopath/src/github.com/alibaba/, remove the files from the original pouch directory(retain the pouch folder)，execute command to mount in the directory. (where SHARE_FOLDER_NAME is the name of the shared folder)
``` bash
sudo mount -t vboxsf SHARE_FOLDER_NAME pouch/
```
The pouch service can be started to detect if the configuration is correct.
