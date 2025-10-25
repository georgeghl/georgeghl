# ESXi7 VMFSL虚拟内存占用120g空间

1、开始安装ESXI7.0，第一次进入安装时，抢在倒计时5秒之内 按shift + O 【是字母不是数字0】如下图：

2、在**`cdromBoot runweasel`**后输入autoPartitionOSDataSize=5120 【注意大小写】【注意有可能开头默认非`cdromBoot runweasel`，如果不是的话需要自己修改】
5120表示指定OSDataSzie为5GB，再按enter回车进行正常安装即可；当然了，如果觉得5G小了点，那就分8G 10G都可以。



# PVEtools

https://github.com/ivanhao/pvetools







# PVE调整/mnt/data、root分区大小

## 1.直接删除data，只使用root

```bash
#先删除/dev/mapper/pve-data
lvremove /dev/mapper/pve-data

#然后扩充/dev/mapper/pve-root
lvextend -L +100G /dev/mapper/pve-root
resize2fs /dev/mapper/pve-root
```





## 2.缩小data，扩大root

```bash
#先缩小/dev/mapper/pve-data
#e2fsck -f /dev/mapper/pve-data
resize2fs 600G /dev/mapper/pve-data
lvreduce -L 600G /dev/mapper/pve-data

#然后扩充/dev/mapper/pve-root
lvextend -L +100G /dev/mapper/pve-root
resize2fs /dev/mapper/pve-root
```







# **PVE 禁用Swap分区、分给root** 、删除分区并分配给root（请结合下面local-lvm改文件存储部分使用）

https://waylee.net/thread-2430-1-1.html

**1.临时禁用 Swap**
你可以通过以下命令来临时禁用 Swap：

```undefined
swapoff -a
```

此命令会禁用所有的 Swap 分区和 Swap 文件。禁用后，它会立即生效，但系统重启后 Swap 会再次启用。
**2. 永久禁用 Swap**
为了永久禁用 Swap，可以编辑 /etc/fstab 文件，将 Swap 分区的挂载项注释掉。
步骤如下：
**2.1. 编辑 /etc/fstab**
使用你喜欢的文本编辑器（例如 vim 或 nano）打开 /etc/fstab 文件：

```undefined
vim /etc/fstab
```

找到类似于以下内容的行：

```undefined
/dev/sdX     swap      swap    defaults    0   0
```

或者：

```undefined
UUID=xxxxxx  swap      swap    defaults    0   0
```

将这行前面加上 #，注释掉这行，使它看起来像这样：

```python
# /dev/sdX     swap      swap    defaults    0   0
```

**2.2. 保存并退出编辑器。**
**2.3. 验证 Swap 是否已禁用**

```undefined
free -h
```

或者：

```lua
swapon --show
```

如果没有 Swap 信息，则表示 Swap 已被成功禁用。
**2.4 删除PVE的Swap分区**

```undefined
fdisk -l
```

显示为

```python
Disk /dev/mapper/pve-swap: 8 GiB, 8589934592 bytes, 16777216 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 16384 bytes / 131072 bytes
Disk /dev/mapper/pve-root: 223.88 GiB, 240392339456 bytes, 469516288 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
```

**2.5 移除掉交换分区**

```undefined
lvremove /dev/mapper/pve-swap
```


**2.6 执行pvdisplay查看已经分配的容量**

```lua
# 查看 Allocated PE 确认可以分配的容量
➜  ~ pvdisplay
  --- Physical volume ---
  PV Name               /dev/nvme0n1p3
  VG Name               pve
  PV Size               231.88 GiB / not usable 2.16 MiB
  Allocatable           yes 
  PE Size               4.00 MiB
  Total PE              59362
  Free PE               2048
  Allocated PE          57314
  PV UUID               OzlpbK-sbRD-QnrV-q0oD-3IJ7-2sxB-R6Xulb
```

**2.7 给根目录分配多8G磁盘**

```undefined
lvextend -L +8G /dev/mapper/pve-root
```


**2.8 resize2fs 后才能使用**

```undefined
resize2fs /dev/mapper/pve-root
```







# [Proxmox的local-lvm改文件存储，提升运行速度](https://www.cnblogs.com/doracloud/p/16874171.html)

https://www.cnblogs.com/doracloud/p/16874171.html

Proxmox的缺省安装会创建 local 和 local-lvm 两个存储。其中local大约磁盘容量的10%，存储类别为目录。 local-lvm的存储类别为 lvm-thin。

实际使用中，发现大批量创建虚拟机时，lvm-thin的效率比目录的效率低，影响系统的IO性能。 主要的原因应该是 lvm-thin 模式下，缺少了文件层的缓存机制。影响了IO效率。

比如双路服务器，使用960G SSD企业硬盘做为存储，如果采用lvm-thin机制，使用 DoraCloud桌面云创建50个桌面，需要超过10分钟。如果采用目录存储，创建速度会提升不少。

有些Proxmox 用户喜欢删除 local-lvm，把所有的磁盘空间都分配给 local 。对于这种操作，我们不推荐。 因为 local 存储上有proxmox的系统文件。如果虚拟机的磁盘也保存在local上，如果遇到磁盘空间耗尽的情况，可能导致 proxmox 的系统也无法启动。 影响系统稳定性，给生产环境的长期运行带来隐患。

本文主要介绍如何将缺省安装的 local-lvm 存储改成 目录机制。

**第一步：从Proxmox Web界面，删除 local-lvm**

登录 PVE web界面，在 数据中心->存储中，选择 local-lvm，然后点击删除。

```
pvesh delete  /storage/local-lvm
```

**第二步：通过命令删除 lvm，新建lvm，并创建文件系统。**

1、登录 Proxmox的 ssh。

2、卸载并删除 lvm-thin

```
umount /dev/pve/data
lvremove /dev/pve/data
```

按Y 确认删除。

3、检查磁盘剩余空间

```
vgdisplay pve | grep Free
```

4、创建新的lvm

注意，请使用上一步的空闲空间的数字92482替换下面的数字。

```
lvcreate -l 92482 -n data pve
```

5、格式化，并挂载文件系统。

```
mkfs.ext4 /dev/pve/data
mkdir /mnt/data
mount /dev/pve/data /mnt/data
```

6、最后配置 fstab，确保重启时，可以挂载文件系统

```
nano /etc/fstab
/dev/pve/data /mnt/data ext4 defaults 0 0
```











# Proxmox Virtual Environment（PVE）安装、docker配置等

## 1.镜像下载

https://www.proxmox.com/en/downloads

https://enterprise.proxmox.com/iso/proxmox-ve_8.3-1.iso

https://www.proxmox.com/images/download/pve/iso/proxmox-ve_8.3-1.torrent

清华tuna：

https://mirrors.tuna.tsinghua.edu.cn/proxmox/iso/

## 2.安装

略。

## 3.proxmox软件仓库、apt软件仓库配置

### 3.1.使用清华源配置proxmox软件仓库

https://mirrors.tuna.tsinghua.edu.cn/help/proxmox/

新建 `/etc/apt/sources.list.d/pve-no-subscription.list`（如果选择的不是 pve，需要使用另一个文件名），内容为：

Debian 版本Debian 12 (bookworm)，服务Proxmox Virtual Environment (PVE)

```
deb https://mirrors.tuna.tsinghua.edu.cn/proxmox/debian/pve bookworm pve-no-subscription
```

注意快速配置中的文件名需要根据需求而修改。

### 3.2.使用清华源/阿里云源配置apt软件仓库

先备份原有的：

```bash
cd /etc/apt/
cp sources.list sources.list.BAK
cp sources.list sources.list.original
```

##### **阿里云：**

```bash
rm sources.list
touch sources.list
nano sources.list
```

复制粘贴以下内容（Ctrl+Shift+V），下面给出两种写法：

（1）cnblogs，实测有效：

https://www.cnblogs.com/smlile-you-me/p/17727308.html

```sh
deb https://mirrors.aliyun.com/debian/ bookworm main non-free non-free-firmware contrib
deb-src https://mirrors.aliyun.com/debian/ bookworm main non-free non-free-firmware contrib
deb https://mirrors.aliyun.com/debian-security/ bookworm-security main
deb-src https://mirrors.aliyun.com/debian-security/ bookworm-security main
deb https://mirrors.aliyun.com/debian/ bookworm-updates main non-free non-free-firmware contrib
deb-src https://mirrors.aliyun.com/debian/ bookworm-updates main non-free non-free-firmware contrib
deb https://mirrors.aliyun.com/debian/ bookworm-backports main non-free non-free-firmware contrib
deb-src https://mirrors.aliyun.com/debian/ bookworm-backports main non-free non-free-firmware contrib
```

（2）aliyun官方，只给了**debian 11.x (bullseye)**：

https://developer.aliyun.com/mirror/debian

```sh
deb https://mirrors.aliyun.com/debian/ bullseye main non-free contrib
deb-src https://mirrors.aliyun.com/debian/ bullseye main non-free contrib
deb https://mirrors.aliyun.com/debian-security/ bullseye-security main
deb-src https://mirrors.aliyun.com/debian-security/ bullseye-security main
deb https://mirrors.aliyun.com/debian/ bullseye-updates main non-free contrib
deb-src https://mirrors.aliyun.com/debian/ bullseye-updates main non-free contrib
deb https://mirrors.aliyun.com/debian/ bullseye-backports main non-free contrib
deb-src https://mirrors.aliyun.com/debian/ bullseye-backports main non-free contrib
```

按Ctrl+X保存，输入大写Y回车保存。

备份配置文件：

```bash
cp sources.list sources.list.aliyun
```

##### **清华tuna：**

https://mirrors.tuna.tsinghua.edu.cn/help/debian/

```bash
rm sources.list
touch sources.list
nano sources.list
```

复制粘贴以下内容（Ctrl+Shift+V）：

```sh
# 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释
deb https://mirrors.tuna.tsinghua.edu.cn/debian/ bookworm main contrib non-free non-free-firmware
# deb-src https://mirrors.tuna.tsinghua.edu.cn/debian/ bookworm main contrib non-free non-free-firmware

deb https://mirrors.tuna.tsinghua.edu.cn/debian/ bookworm-updates main contrib non-free non-free-firmware
# deb-src https://mirrors.tuna.tsinghua.edu.cn/debian/ bookworm-updates main contrib non-free non-free-firmware

deb https://mirrors.tuna.tsinghua.edu.cn/debian/ bookworm-backports main contrib non-free non-free-firmware
# deb-src https://mirrors.tuna.tsinghua.edu.cn/debian/ bookworm-backports main contrib non-free non-free-firmware

# 以下安全更新软件源包含了官方源与镜像站配置，如有需要可自行修改注释切换
deb https://mirrors.tuna.tsinghua.edu.cn/debian-security bookworm-security main contrib non-free non-free-firmware
# deb-src https://mirrors.tuna.tsinghua.edu.cn/debian-security bookworm-security main contrib non-free non-free-firmware
```

按Ctrl+X保存，输入大写Y回车保存。

备份配置文件：

```bash
cp sources.list sources.list.tuna
```

### 3.3.完成后，更新apt源

```sh
apt-get update
```

### 3.4.完成后，安装vim

如果不使用apt重新安装vim，自带的vim会有上下左右键失效的问题。

```bash
apt install vim
```

## 4.docker安装

### 4.1.使用清华tuna源

https://mirrors.tuna.tsinghua.edu.cn/help/docker-ce/

#### 手动安装—Debian用户

以下内容根据 [官方文档](https://docs.docker.com/engine/install/debian/) 修改而来。

如果你过去安装过 docker，先删掉：

```bash
for pkg in docker.io docker-doc docker-compose podman-docker containerd runc; do apt-get remove $pkg; done
```

首先安装依赖：

```bash
apt-get update
apt-get install ca-certificates curl gnupg
```

信任 Docker 的 GPG 公钥并添加仓库：

```bash
install -m 0755 -d /etc/apt/keyrings

curl -fsSL https://download.docker.com/linux/debian/gpg | gpg --dearmor -o /etc/apt/keyrings/docker.gpg

chmod a+r /etc/apt/keyrings/docker.gpg

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/debian \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  tee /etc/apt/sources.list.d/docker.list > /dev/null
```

最后安装

```bash
apt-get update

apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

#### 自动安装方式（需要使用代理，不推荐）

Docker 提供了一个自动配置与安装的脚本，支持 Debian、RHEL、SUSE 系列及衍生系统的安装。请注意，Docker 官方不建议在生产环境使用此脚本安装 Docker CE。

以下内容假定

- 您为 root 用户，或有 sudo 权限，或知道 root 密码；
- 您系统上有 curl 或 wget

```bash
export DOWNLOAD_URL="https://mirrors.tuna.tsinghua.edu.cn/docker-ce"

# 如您使用 curl
curl -fsSL https://raw.githubusercontent.com/docker/docker-install/master/install.sh | sh
# 如您使用 wget
wget -O- https://raw.githubusercontent.com/docker/docker-install/master/install.sh | sh
```

### 4.2.使用阿里云源

阿里云官方教程只给出了ubuntu的安装教程：https://developer.aliyun.com/mirror/docker-ce

### 4.3.安装后查看状态

```bash
systemctl status docker
```

## 5.cockpit安装

参考ubuntu中cockpit安装和配置的教程，过程一样。

```bash
apt install cockpit
systemctl start cockpit
systemctl status cockpit
```

修改监听端口号：

```bash
cd /etc/systemd/system/cockpit.socket.d/
cp listen.conf listen.conf.original
cp listen.conf listen.conf.bak
vi listen.conf
```

修改文件内容并保存：

```bash
[Socket]
ListenStream=
ListenStream=9091
```

完成后，重启cockpit服务：

```bash
systemctl restart cockpit
```

## 6.ssh配置、安装ufw、安装net-tools、安装neofetch等

默认自带有ssh，如果未默认启动，则设置自启：

```bash
systemctl enable sshd
systemctl start sshd
```

安装ufw：

```
apt install ufw
```

安装net-tools（ifconfig）：

```bash
apt install net-tools
```

安装neofetch、screenfetch、s-tui：

```bash
apt install neofetch
apt install screenfetch
apt install s-tui
```





# 2024年PVE8最新安装使用指南|新手入门|安装|优化|Proxmox VE 8.1

https://post.smzdm.com/p/akle62mk/

### **PVE镜像下载**

1、在浏览器输入以下网址下载pve的iso安装镜像。

https://www.proxmox.com/en/downloads

2、点击download进行下载。

[![2024年PVE8最新安装使用指南|新手入门|安装|优化|Proxmox VE 8.1](./img/65f268ad44ffe6177.png_e1080.jpg)](https://post.smzdm.com/p/akle62mk/pic_2/)

3、最新的pve8.1-2镜像大小为1.2GB，接下来制作引导U盘。

[![2024年PVE8最新安装使用指南|新手入门|安装|优化|Proxmox VE 8.1](./img/65f268ad54ed46177.png_e1080.jpg)](https://post.smzdm.com/p/akle62mk/pic_3/)

### **引导U盘制作**

1、首先下载写盘工具，本文使用的写盘工具为Rufus，下载地址为https://rufus.ie/downloads/[，选择最新版本的下载。](https://rufus.ie/downloads/)

[![2024年PVE8最新安装使用指南|新手入门|安装|优化|Proxmox VE 8.1](./img/65f268ad108836177.png_e1080.jpg)](https://post.smzdm.com/p/akle62mk/pic_4/)

2、下载完成后直接双击运行，无需安装，设备中选择U盘，引导类型选择PVE镜像，然后点击开始，注意U盘中的原有数据会被格式化，重要数据记得备份。

[![2024年PVE8最新安装使用指南|新手入门|安装|优化|Proxmox VE 8.1](./img/65f268ad03f996177.png_e1080.jpg)](https://post.smzdm.com/p/akle62mk/pic_5/)

3、当状态为准备就绪时，表示U盘写入成功，拔下U盘插入需要安装的机器的USB接口中（[台式机](https://www.smzdm.com/fenlei/taishiji/)推荐后置USB接口）

[![2024年PVE8最新安装使用指南|新手入门|安装|优化|Proxmox VE 8.1](./img/65f268ad8bf806177.png_e1080.jpg)](https://post.smzdm.com/p/akle62mk/pic_6/)

### **PVE安装**

1、打开主机电源，不断按键盘F12按键（不同主机的U启动按键不同，请自行百度或者询问卖家）进入PVE安装页面，选择第一个Install Proxmox VE （Graphical)，按回车。

[![2024年PVE8最新安装使用指南|新手入门|安装|优化|Proxmox VE 8.1](./img/65f268ade00826177.png_e1080.jpg)](https://post.smzdm.com/p/akle62mk/pic_7/)

2、 页面会进入如下的跑代码阶段，跑码时间看机器配置，跑码期间不要动键盘鼠标。

[![2024年PVE8最新安装使用指南|新手入门|安装|优化|Proxmox VE 8.1](./img/65f268ad2c8bf6177.png_e1080.jpg)](https://post.smzdm.com/p/akle62mk/pic_8/)

3、进入下面页面安装正式开始，首先使用协议页面点击 I agree（我同意）。

[![2024年PVE8最新安装使用指南|新手入门|安装|优化|Proxmox VE 8.1](./img/65f268add0af86177.png_e1080.jpg)](https://post.smzdm.com/p/akle62mk/pic_9/)

4、选择pve安装的[硬盘](https://www.smzdm.com/fenlei/yingpan/)，然后继续点击Next(下一步）。

[![2024年PVE8最新安装使用指南|新手入门|安装|优化|Proxmox VE 8.1](./img/65f268ad45af46177.png_e1080.jpg)](https://post.smzdm.com/p/akle62mk/pic_10/)

5、设置国家、时区、键盘布局，然后点击Next(下一步）。

[![2024年PVE8最新安装使用指南|新手入门|安装|优化|Proxmox VE 8.1](./img/65f268ada340b6177.png_e1080.jpg)](https://post.smzdm.com/p/akle62mk/pic_11/)

6、设置密码、和邮箱，继续点击Next(下一步）。

[![2024年PVE8最新安装使用指南|新手入门|安装|优化|Proxmox VE 8.1](./img/65f268adabb426177.png_e1080.jpg)](https://post.smzdm.com/p/akle62mk/pic_12/)

7、设置网卡、域（保持默认即可）、PVE管理地址、网关、DNS等，继续点击Next(下一步）。

[![2024年PVE8最新安装使用指南|新手入门|安装|优化|Proxmox VE 8.1](./img/65f268ad60d686177.png_e1080.jpg)](https://post.smzdm.com/p/akle62mk/pic_13/)

8、在配置详情中检查配置信息是否有误，没有问题点击Install（安装）。

[![2024年PVE8最新安装使用指南|新手入门|安装|优化|Proxmox VE 8.1](./img/65f268ad8fb786177.png_e1080.jpg)](https://post.smzdm.com/p/akle62mk/pic_14/)

9、移除U盘，然后点击Reboot重启机器。

[![2024年PVE8最新安装使用指南|新手入门|安装|优化|Proxmox VE 8.1](./img/65f268adb54cd6177.png_e1080.jpg)](https://post.smzdm.com/p/akle62mk/pic_15/)

### **初始化**

1、重启完成之后出现如下页面，表示安装成功，记录下屏幕中出现的ip地址，接下来的配置无需使用PVE主机显示，单个[显示器](https://www.smzdm.com/fenlei/xianshiqi/)的朋友可以将显示器切换到主机了。

[![2024年PVE8最新安装使用指南|新手入门|安装|优化|Proxmox VE 8.1](./img/65f268ad6a91e6177.png_e1080.jpg)](https://post.smzdm.com/p/akle62mk/pic_16/)

2、在浏览器输入[https://192.168.2.167:8006](https://192.168.2.167:8006/)[进入PVE配置页面，用户名为root，密码为在安装过程中设置的，语言为了方便设置为简体中文，然后点击登录。](https://192.168.2.167:8006/)

[![2024年PVE8最新安装使用指南|新手入门|安装|优化|Proxmox VE 8.1](./img/65f268ad614866177.png_e1080.jpg)](https://post.smzdm.com/p/akle62mk/pic_17/)

3、第一次登录会出现无效订阅不用管，直接点击确定，后续我们将通过设置去掉这个选项。

[![2024年PVE8最新安装使用指南|新手入门|安装|优化|Proxmox VE 8.1](./img/65f268adca0e26177.png_e1080.jpg)](https://post.smzdm.com/p/akle62mk/pic_18/)

4、首先我们需要配置一下pve的dns来保证pve可以正确访问到网络，这里我使用了阿里云公共的DNS，223.5.5.5，223.6.6.6，然后点击ok。

[![2024年PVE8最新安装使用指南|新手入门|安装|优化|Proxmox VE 8.1](./img/65f268adb183b6177.png_e1080.jpg)](https://post.smzdm.com/p/akle62mk/pic_19/)

5、使用pve自带的shell工具来ping百度，测试网络是否正常，出现正确的ping值表示外网访问通畅。

[![2024年PVE8最新安装使用指南|新手入门|安装|优化|Proxmox VE 8.1](./img/65f268adac08b6177.png_e1080.jpg)](https://post.smzdm.com/p/akle62mk/pic_20/)

6、接下来还涉及到pve的换源、系统更新、直通等操作，为了方便小白和第一次接触到pve的朋友们，这里我们将使用第三方工具来实现。

### **优化（PVE_Source）**

pve_source是老虎大佬开发的pve脚本（[首发于X86派论坛](https://bbs.x86pi.cn/thread?topicId=20)），可以更方便的初始化PVE，无需使用复杂的命令，使用方法很简单，登录PVE，打开Shell粘贴如下命令。

`wget -q -O /root/pve_source.tar.gz 'https://bbs.x86pi.cn/file/topic/2023-11-28/file/01ac88d7d2b840cb88c15cb5e19d4305b2.gz' && tar zxvf /root/pve_source.tar.gz && /root/./pve_source`[![2024年PVE8最新安装使用指南|新手入门|安装|优化|Proxmox VE 8.1](./img/65f268ad42eef6177.png_e1080.jpg)](https://post.smzdm.com/p/akle62mk/pic_21/)

1、脚本加载成功出现使用协议，输入Y，然后继续按回车。

[![2024年PVE8最新安装使用指南|新手入门|安装|优化|Proxmox VE 8.1](./img/65f268ada44a96177.png_e1080.jpg)](https://post.smzdm.com/p/akle62mk/pic_22/)

2、进入脚本优化选项，脚本覆盖了PVE的绝大部分必备优化选项，需要使用那个直接输入数字然后按回车即可。

[![2024年PVE8最新安装使用指南|新手入门|安装|优化|Proxmox VE 8.1](./img/65f268ad983436177.png_e1080.jpg)](https://post.smzdm.com/p/akle62mk/pic_23/)

### **更换 Proxmox VE 源**

3、为了更方便了解脚本都干了什么，建议分步执行23456，输入数字2,然后再输入1更换 PVE 软件源 + Debian 源，分别输入列表前面的数字选择PVE版本和软件源镜像站（不清楚的直接都选择清华大学源就行）。

[![2024年PVE8最新安装使用指南|新手入门|安装|优化|Proxmox VE 8.1](./img/65f268ad97bdb6177.png_e1080.jpg)](https://post.smzdm.com/p/akle62mk/pic_24/)

4、选择软件源以后继续按回车执行，脚本将自动执行更换源成功后会给出提示。

[![2024年PVE8最新安装使用指南|新手入门|安装|优化|Proxmox VE 8.1](./img/65f268ade06866177.png_e1080.jpg)](https://post.smzdm.com/p/akle62mk/pic_25/)

5、按照以上操作继续完成PVE Ceph 源 、PVE LXC 仓库源 的更新操作（在Shell页面输入./pve_source即可再次调出pve工具小技巧直接输入./pve然后按tab按键可自动补全后面）。

[![2024年PVE8最新安装使用指南|新手入门|安装|优化|Proxmox VE 8.1](./img/65f268adc82926177.png_e1080.jpg)](https://post.smzdm.com/p/akle62mk/pic_26/)

### **更新软件包系统**

更新软件包有两种方法，一种是通过pve工具箱更新，另外一种因为在前面我们已经通过pve工具箱更新了软件源，所以直接在web页面更新也可以

1、点击pve节点→更新→刷新→升级，既可以完成最新软件包和组件的升级。

[![2024年PVE8最新安装使用指南|新手入门|安装|优化|Proxmox VE 8.1](./img/65f268adcfc7c6177.png_e1080.jpg)](https://post.smzdm.com/p/akle62mk/pic_27/)

2、通过pve工具升级，在Shell中输入./pve_source，然后使用脚本3、4来更新软件包和系统。

[![2024年PVE8最新安装使用指南|新手入门|安装|优化|Proxmox VE 8.1](./img/65f268ad597dd6177.png_e1080.jpg)](https://post.smzdm.com/p/akle62mk/pic_28/)

### **设置系统DNS**

1、通过web页面修改，点击节点→DNS→编辑。

[![2024年PVE8最新安装使用指南|新手入门|安装|优化|Proxmox VE 8.1](./img/65f268ad4c3966177.png_e1080.jpg)](https://post.smzdm.com/p/akle62mk/pic_29/)

2、通过pve工具箱修改。

[![2024年PVE8最新安装使用指南|新手入门|安装|优化|Proxmox VE 8.1](./img/65f268adb13e46177.png_e1080.jpg)](https://post.smzdm.com/p/akle62mk/pic_30/)

### **去除无效订阅源提示**

每次在登录pve的时候会弹出企业订阅提示，对于普通用户我们直接去除掉弹窗即可，登录PVE，在Shell窗口中输入./pve_source,进入pve工具箱，输入6，系统会自动执行去除订阅脚本，脚本运行完成后请强制刷新浏览器缓存。

[![2024年PVE8最新安装使用指南|新手入门|安装|优化|Proxmox VE 8.1](./img/65f268adedc636177.png_e1080.jpg)](https://post.smzdm.com/p/akle62mk/pic_31/)

### **修改PVE概要信息**

1、pve默认概要中不会显示cpu温度、硬盘信息等，通过pve工具箱可以修改信息概要页面，使其信息显示更加完善。

[![2024年PVE8最新安装使用指南|新手入门|安装|优化|Proxmox VE 8.1](./img/65f268ada68286177.png_e1080.jpg)](https://post.smzdm.com/p/akle62mk/pic_32/)

2、登录PVE，在Shell窗口中输入./pve_source,进入pve工具箱，按7进入pve概要信息定制向导，根据自身喜好选择合适的版本，这里我使用推荐方案1：高大全版本。

[![2024年PVE8最新安装使用指南|新手入门|安装|优化|Proxmox VE 8.1](./img/65f268adfdf576177.png_e1080.jpg)](https://post.smzdm.com/p/akle62mk/pic_33/)

3、修改改完成后可以看到概要信息中显示的非常全面，包括CPU温度核心占用情况，以及硬盘的温度写入量等都可以很直观的显示。

[![2024年PVE8最新安装使用指南|新手入门|安装|优化|Proxmox VE 8.1](./img/65f268ad573756177.png_e1080.jpg)](https://post.smzdm.com/p/akle62mk/pic_34/)

### **虚拟机安装**

pve安装虚拟机还是非常简单的，本篇我们将使用pve安装win10来演示一下安装过程，其他系统安装大同小异。

1、首先需要上传安装镜像，通过local(pve)→ISO镜像→上传，将win10镜像上传到pve中。

[![2024年PVE8最新安装使用指南|新手入门|安装|优化|Proxmox VE 8.1](./img/65f268ad9a3cf6177.png_e1080.jpg)](https://post.smzdm.com/p/akle62mk/pic_35/)

2、回到pve首页，点击右上角的创建虚拟机，常规中自定义虚拟机的名称

[![2024年PVE8最新安装使用指南|新手入门|安装|优化|Proxmox VE 8.1](./img/65f268ad587816177.png_e1080.jpg)](https://post.smzdm.com/p/akle62mk/pic_36/)

3、操作系统中选择windows10镜像，然后在类型中选择安装操作系统的类型，注意版本也要选择。

[![2024年PVE8最新安装使用指南|新手入门|安装|优化|Proxmox VE 8.1](./img/65f268adda64c6177.png_e1080.jpg)](https://post.smzdm.com/p/akle62mk/pic_37/)

4、系统保持默认即可。

[![2024年PVE8最新安装使用指南|新手入门|安装|优化|Proxmox VE 8.1](./img/65f268acb10ea8851.png_e1080.jpg)](https://post.smzdm.com/p/akle62mk/pic_38/)

5、硬盘中总线选择sata，其他保持默认。

[![2024年PVE8最新安装使用指南|新手入门|安装|优化|Proxmox VE 8.1](./img/65f268ad1b0046177.png_e1080.jpg)](https://post.smzdm.com/p/akle62mk/pic_39/)

6、CPU根据机器配置选择，插槽表示cpu个数单CPU机器输入1，核心根据CPU选择，类别使用HOST系统性能最大化。

[![2024年PVE8最新安装使用指南|新手入门|安装|优化|Proxmox VE 8.1](./img/65f268ad5b8536177.png_e1080.jpg)](https://post.smzdm.com/p/akle62mk/pic_40/)

7、[内存](https://www.smzdm.com/fenlei/neicun/)根据机器配置设置。

[![2024年PVE8最新安装使用指南|新手入门|安装|优化|Proxmox VE 8.1](./img/65f268ad420c56177.png_e1080.jpg)](https://post.smzdm.com/p/akle62mk/pic_41/)

8、网络设置中选择兼容性最高的e1000即可。

[![2024年PVE8最新安装使用指南|新手入门|安装|优化|Proxmox VE 8.1](./img/65f268addaca66177.png_e1080.jpg)](https://post.smzdm.com/p/akle62mk/pic_42/)

9、最后检查配置没有，确认无误后点击完成。

[![2024年PVE8最新安装使用指南|新手入门|安装|优化|Proxmox VE 8.1](./img/65f268adc58706177.png_e1080.jpg)](https://post.smzdm.com/p/akle62mk/pic_43/)

10、稍等几秒，机器创建成功。

[![2024年PVE8最新安装使用指南|新手入门|安装|优化|Proxmox VE 8.1](./img/65f268adcbe226177.png_e1080.jpg)](https://post.smzdm.com/p/akle62mk/pic_44/)

11、点击控制台启动，按照正常windows系统安装流程安装即可。

[![2024年PVE8最新安装使用指南|新手入门|安装|优化|Proxmox VE 8.1](./img/65f268ad6cfa06177.png_e1080.jpg)](https://post.smzdm.com/p/akle62mk/pic_45/)













# r3g刷breed

看有人在问R3G刷Breed，所以分享出自己在用的教程。
一、刷开发版(目前最新2018.10.30，版本2.25.124)
1、下载固件(http://www.miwifi.com/miwifi_download.html)文件，miwifi_r3g_firmware_12f97_2.25.124.bin
2、备份原厂稳定版固件
3、进路由器设置，系统状态-系统版本-手动升级，上传.bin 文件，升级重启
二、开SSH权限
1、（如果app已经绑定路由器，跳过）通过手机app【小米wifi】绑定路由器
2、找到开SSH的工具包：https://d.miwifi.com/rom/ssh下载对应的工具包miwifi_ssh.bin，并保存 root 密码
3、把工具包miwifi_ssh.bin(文件名只能是这个)放入U盘(FAT/FAT32格式,U盘不能是启动盘)根目录
4、断开小米路由器电源，插入U盘到路由器
5、按住reset键开机，指示灯变黄色闪烁状态即松开 reset 键
6、待3-5秒后蓝灯亮起安装完成，路由器自动重启
7、终端ssh登录：ssh [root@192.168.31.1](mailto:root@192.168.31.1)，输入ssh权限密码
三、刷不死平台breed：(注意，原版OpenWrt不要通过 breed刷，因为，OpenWrt 原版需要上传2个文件，而breed，只能上传一个文件，如果要用原版 OpenWrt，可以scp 复制文件到/tmp 下，然后 ssh 登录后命令行安装)
1、https://breed.hackpascal.net/下载breed-mt7621-xiaomi-r3g.bin
2、用ssh工具上传.bin文件到路由器/tmp目录下：
scp -P 22 文件名路径 [root@192.168.31.1](mailto:root@192.168.31.1):/tmp
3、ssh进入小米路由器，写入 breed：
mtd -r write /tmp/breed-mt7621-xiaomi-r3g.bin Bootloader
4、刷入后，机器会重新启动。
四、进入 breed方法：
1、按住reset键开机，灯闪烁后松开reset键
2、等获取到ip，浏览器进入192.168.1.1
3、非常重要的备份：
在 Breed 页面中，先备份所有的数据，固件备份选项里能点的都点一下，备份的eeprom.bin(EEPROM)和full.bin(编程器固件)要保存好，以防止丢失了SN以后又想刷回官方版本的麻烦。

已知小米R3G需要
1、删除normal_firmware_md5=8945bfb64aa2cdc1a35dd63333b6f28f(R4是8a5ea648853b89e0936f3e2a18b31099)这个键值，否则可能会导致无法开机
2、添加环境变量xiaomi.r3g.bootfw:2







# [Proxmox VE（PVE）添加硬盘做存储](https://www.cnblogs.com/txqdm/p/17946432)

首先需要先先看下需添加硬盘的设备名称

如下图的/dev/sda(当然也可以在命令里用fdisk -l去确认)

确定好名称后打开Shell命令终端，输入

```
fdisk /dev/sda
```

这里的操作就跟Linux没什么区别了，分区格式化

先分区，输入n(分区) - 输入p(主分区) - 输入w(写入分区)

格式化(格式化为ext4格式)

```
mkfs -t ext4 /dev/sda1
```

 创建/mnt/sda1目录

```
mkdir -p /mnt/sda1
```

 将/dev/sda1分区挂载在/mnt/sda1上

```
mount -t ext4 /dev/sda1 /mnt/sda1
```

 将/mnt/sda1挂载写入到系统启动项

```
echo /dev/sda1 /mnt/sda1 ext4 defaults 1 2 >> /etc/fstab
```

选择“数据中心”(这块一定要注意不是选择PVE)存储 - 添加 - 目录

ID可以随便填，目录就是刚才创建好的/mnt/sda1，内容**全选**，其它默认点击添加。







# pve dhcp

1. **登录到PVE服务器的命令行界面**：可以通过直接连接键盘和显示器或通过SSH进行。
2. **备份原始网络配置文件**：以防万一。 *sudo cp /etc/network/interfaces /etc/network/interfaces.bak*
3. **使用文本编辑器打开网络配置文件**： *sudo nano /etc/network/interfaces*
4. **找到主网络接口并修改配置**：将*static*更改为*dhcp*，并删除或注释掉*address*、*netmask*和*gateway*行。修改后的配置应如下所示： *auto lo iface lo inet loopback iface enp1s0 inet manual auto vmbr0 iface vmbr0 inet dhcp # address 192.168.124.17/24 # gateway 192.168.124.1 bridge-ports enp1s0 bridge-stp off bridge-fd 0*
5. **保存更改并重启网络服务**： *sudo systemctl restart networking*











# pve sudo(Debian12) 安装sudo权限

https://www.mulingyuer.com/archives/912/

登录root账号，安装依赖：

```bash
apt-get install sudo
```

安装完毕后我们还需要配置文件，输入命令：

```bash
visudo
```

进入到编辑状态，找到root的地址，再下面新增一个你允许提权的账号.

```ini
your_user_name  ALL=(ALL:ALL) ALL
```

改好后 `CTRL + O`保存文件，`CTRL + X`退出编辑模式。

此时我们通过ABC ssh连接服务器，输入命令：`sudo -i`，然后输入ABC账号的密码，即可提权成root账号。



