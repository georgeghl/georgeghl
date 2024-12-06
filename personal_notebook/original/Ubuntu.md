https://cloud-atlas.readthedocs.io/zh-cn/latest/linux/server/index.html



# Ubuntu修改Grub默认启动Windows

**1.开机的时候，在GRUB选择页面查看Windows boot处于第几行(项)，记住，后续步骤要用！**

2.进入Ubuntu，打开终端。注意以下步骤，有sudo的一定要加sudo！

```bash
cd /etc/default
sudo gedit /etc/default/grub
```

出现以下界面

![在这里插入图片描述](./img/ubuntu111.jpg)

3.假设我的Windows boot处于步骤1中第三行（项），所以对应的GRUB_DEFAULT应改为2。（以0开始算）
GRUB_TIMEOUT为在页面停留时间，根据自己的需要更改。

4.最后在终端执行：

```bash
sudo update-grub
```



# 添加新sudo用户

```bash
sudo adduser newusername
sudo usermod -a -G sudo newusername
```



# 运维面板——Webadmin

先到官网下载deb，然后

```bash
sudo apt-get install --install-recommends ./webmin-current.deb
```

修改端口号：到设置中设置。默认端口：10000。

# 运维面板——Cockpit

```bash
sudo apt install cockpit

# 如果您计划管理 带 Cockpit 的 KVM 虚拟机 ，需要安装以下软件包
# sudo apt install cockpit-machines
# 为 Cockpit 安装 Podman 支持
# sudo apt install cockpit-podman y

#启动 Cockpit Web 控制台服务
sudo systemctl start cockpit
```

## Cockpit监听端口和地址

https://cloud-atlas.readthedocs.io/zh-cn/latest/linux/server/cockpit/cockpit_port_address.html#cockpit

Cockpit的 `cockpit-ws` 组件默认配置的端口是 `9090` ，这个端口是和 [Prometheus监控](https://cloud-atlas.readthedocs.io/zh-cn/latest/kubernetes/monitor/prometheus/index.html#prometheus) 默认端口是重合的，这会导致 [Prometheus快速起步](https://cloud-atlas.readthedocs.io/zh-cn/latest/kubernetes/monitor/prometheus/prometheus_startup.html#prometheus-startup) 部署在主机上的 Prometheus 启动失败。

比较简单的方法是调整 `cockpit` 的TCP监听端口

- 备份后修改 `/etc/systemd/system/cockpit.socket.d/listen.conf` (在 [Ubuntu Linux](https://cloud-atlas.readthedocs.io/zh-cn/latest/linux/ubuntu_linux/index.html#ubuntu-linux) 发行版默认你配置 `/etc/systemd/system/sockets.target.wants/cockpit.socket` ):

  ```conf
  [Socket]
  ListenStream=
  ListenStream=9091
  ```

这里的 `ListenStream=` 是有意配置的，因为 [Systemd进程管理器](https://cloud-atlas.readthedocs.io/zh-cn/latest/linux/redhat_linux/systemd/index.html#systemd) 允许多端口监听，所以这里配置一个空的配置表示关闭掉原始配置默认的 `9090` 端口

- 重启服务:

  ```bash
  sudo systemctl daemon-reload
  sudo systemctl restart cockpit.socket
  ```





# Ubuntu使用netplan，修改设置静态IP

## 第1步： 查看当前主机的网卡名，当前ip, 子网掩码，网关地址

```shell
ifconfig
route -n
```

如果ifconfig命令无法使用, 请运行以下命令安装net-tools

```shell
#sudo apt update -y 
sudo apt install net-tools -y
```

查看当前主机的网卡名，当前ip, 子网掩码，网关地址

如上图所示：网卡名为 **enp0s5**, 当前ip **10.211.55.6**, 子网掩码 **255.255.255.0**， 网关地址 **10.211.55.1**

## 第2步： 修改配置文件

mac地址查询：[oui](oui.txt)   或   [Openwrt.html#%E4%BF%AE%E6%94%B9wan%E5%92%8Clan%E7%9A%84mac](Openwrt.html#%E4%BF%AE%E6%94%B9wan%E5%92%8Clan%E7%9A%84mac)

进入配置文件夹

```shell
cd /etc/netplan
```

备份旧配置文件内容为 00-installer-config.yaml_before

```shell
sudo cp 00-installer-config.yaml  00-installer-config.yaml_before
```

**旧** 00-installer-config.yaml 的内容为:

```shell
# This is the network config written by 'subiquity'
network:
  ethernets:
    enp0s5:
      dhcp4: true
  version: 2
```

修改配置文件

```shell
sudo vim 00-installer-config.yaml
```

**更新后的**00-installer-config.yaml内容为:

```shell
network:
  version: 2
  renderer: NetworkManager
  ethernets:
    enp0s5:   # 网卡名称
      dhcp4: no     # 关闭dhcp
      dhcp6: no
      addresses: [10.211.55.10/24]  # 静态ip
      gateway4: 10.211.55.1     # 网关
      nameservers:
        addresses: [8.8.8.8, 114.114.114.114] #dns
```

## 第3步： 使配置生效

```shell
sudo netplan apply
```

## thinServer的yaml配置文件内容

```yaml
# Let NetworkManager manage all devices on this system
network:
  version: 2
  renderer: NetworkManager
  ethernets:
    eno1:
      dhcp4: no
      addresses: [192.168.0.125/24]
      optional: true
      gateway4: 192.168.0.1
      macaddress: 00:E0:FC:DA:C6:58
      nameservers:
              addresses: [114.114.114.114,8.8.8.8]
```

## 修改MAC地址

方法一：使用`ifconfig`（推荐）

1. **关闭网络接口**：
   首先，需要关闭你想要修改MAC地址的网络接口，例如`eth0`。

   ```bash
   # sudo ifconfig eth0 down
   ```

2. **更改MAC地址**：
   使用`ifconfig`命令设置新的MAC地址。

   ```bash
   sudo ifconfig eth0 hw ether XX:XX:XX:XX:XX:XX
   #sudo ifconfig eth0 hw ether 00:E0:FC:DA:C6:58
   ```

   将`XX:XX:XX:XX:XX:XX`替换为你想要设置的新MAC地址。

3. **重新启用网络接口**：

   ```bash
   sudo ifconfig eth0 up
   ```

方法二：使用`ip`

1. **关闭网络接口**：
   首先，需要关闭网络接口。

   ```bash
   sudo ip link set dev eth0 down
   ```

2. **更改MAC地址**：
   使用`ip`命令设置新的MAC地址。

   ```bash
   sudo ip link set dev eth0 address XX:XX:XX:XX:XX:XX
   1
   ```

3. **重新启用网络接口**：

   ```bash
   sudo ip link set dev eth0 up
   ```





# Zerotier

**这里 ZeroTier 的注册、管理后台、创建网路不在此赘述；**

**后面代码中 \**\*\*###########\*\**\* 需要更换成管理页面创建的 \**\*\*Network ID\*\**\***

提示 ：Unix 系统上，可能需要在 zerotier-cli 命令前加上sudo，而在 Windows 上，将需要使用管理员模式命令提示符。

###  1. ZeroTier 的 Linux 安装脚本

在 **ubuntu** 中安装 **zerotier** 使用 **SSH** 两种安装方法，安装过程会有进度显示：

● 如果您愿意依靠 SSL 来验证站点，则可以通过以下方式完成单行安装：

```bash
curl -s https://install.zerotier.com | sudo bash
```

● //如果您安装了 GPG，则可以使用更安全的选项：

```bash
curl -s 'https://raw.githubusercontent.com/zerotier/ZeroTierOne/master/doc/contact%40zerotier.com.gpg' | gpg --import && \
if z=$(curl -s 'https://install.zerotier.com/' | gpg); then echo "$z" | sudo bash; fi
```

 ZeroTier 的 Linux 更新脚本

如果您在[下载页面](https://www.zerotier.com/download/)上使用 ZeroTier 的 linux 安装脚本，它会将 repo 添加到您的系统。使用 apt 或 yum 更新 zerotier-one。

例如，在基于 debian/ubuntu 的系统上（ centos 系统需要将 apt 替换为 yum ）：

查看可更新软件：

```bash
sudo apt update
```

更新 ZeroTier

```bash
sudo apt install zerotier-one
```

● 额外提示：您可以指定一个版本，比如更新到今天要更新的 1.10.6 版本：

```bash
sudo apt install zerotier-one=1.10.6
```

提示 ：各服务客户端的版本经尽量保持一致能解决一些玄学问题！！

2.加入一个网络

●加入网络

首先在要连接的Linux设备上输入如下命令加入网络，如果连接成功就会出现 200 join OK 的状态码提示：

```bash
# 加入网络命令，操作成功则返回 “200 join OK” 
sudo zerotier-cli join  ###########
```

●查看当前连接

```bash
# 查看当前连接的网络，如果列表中出现 Network ID、Name 说明连接成功，后台分配好IP后再查看IP地址也会出现。
sudo zerotier-cli listnetworks
```

3. 常用操作

● [zerotier](https://so.csdn.net/so/search?q=zerotier&spm=1001.2101.3001.7020) 服务 手动启动：

```bash
# 返回信息 “200 info ########### 1.10.6 ONLINE”
zerotier-cli status
```

● zerotier 服务 手动启动：

```sql
sudo systemctl start zerotier-one.service
```

● zerotier 服务 开机自启动：

```bash
# 开机自启动，成功怎返回值的最后会有“enable zerotier-one”字样
sudo systemctl enable zerotier-one.service
```

● zerotier 服务 查看本机连接的网络：上面有提过---查看当前连接

```bash
# 查看当前连接的网络，如果列表中出现 Network ID、Name 说明连接成功，后台分配好IP后再查看IP地址也会出现。
sudo zerotier-cli listnetworks
```

● zerotier 服务 加入网络：

```bash
# 加入网络命令，操作成功则返回 “200 join OK”
sudo zerotier-cli join ###########
```

● zerotier 服务 断开当前加入的网络：

```bash
# 断开网络命令，操作成功则返回 “200 leave OK”
sudo zerotier-cli leave ###########
```

● zerotier 服务 停止

```bash
sudo systemctl stop zerotier-one
```

或者

```bash
sudo service zerotier-one stop
```

● zerotier 服务 重启服务

```bash
sudo systemctl start zerotier-one
```

或者

```bash
sudo service zerotier-one start
```

4.卸载命令

在 **ubuntu、Debian** 中卸载 **zerotier** 

使用 **SSH** 依次输入下面两条代码：

```bash
sudo dpkg -P zerotier-one
sudo rm -rf /var/lib/zerotier-one/
```

如果是 **CentOS、Redhat** 中卸载 **zerotier** 

使用 **SSH** 依次输入下面两条代码：

```bash
sudo rpm -e zerotier-one
sudo rm -rf /var/lib/zerotier-one/
```





# Ubuntu开机默认进入图形界面/字符界面

```shell
sudo systemctl set-default multi-user.target  "默认进入字符界面"
sudo systemctl set-default graphical.target   "默认进入图形界面"
```

字符界面下，可以输入以下命令启动图形化界面：

```shell
sudo init 5
#或(不推荐)
#startx
```





# Ubuntu开机grub/grubRescue手动启动

```shell
#查看所有磁盘
ls
#依次查看每个磁盘分区内容，找到linux主分区/boot所在分区
ls (hd0,gpt8)/
#设置root和prefix
set root=(hd0,gpt8)/
set prefix=(hd0,gpt8)/boot/grub
insmod normal
normal
```



# apt走代理

为 `apt` 命令配置代理的最简单方法是创建一个 `proxy.conf` 文件，如下：

```bash
sudo vi /etc/apt/apt.conf.d/proxy.conf
```

对于没有用户名和密码的代理服务器，添加以下条目，如下：

对于 HTTP 代理，添加以下条目：

```bash
Acquire::http::Proxy "http://proxy-IP-address:proxyport/";
```

对 HTTPS 代理执行相同的操作：

```bash
Acquire::https::Proxy "http://proxy-IP-address:proxyport/";
```

例子：

```bash
sudo vi /etc/apt/apt.conf.d/proxy.conf
Acquire::http::Proxy "http://127.0.0.1:7890/";
Acquire::https::Proxy "http://127.0.0.1:7890";
```

如果你的代理服务器需要用户名和密码详细信息，请按以下方式添加：

```bash
Acquire::http::Proxy "http://username:password@proxy-IP-address:proxyport";
Acquire::https::Proxy "http://username:password@proxy-IP-address:proxyport";
```

例子：

```bash
sudo vi /etc/apt/apt.conf.d/proxy.conf
Acquire::http::Proxy "http://init@PassW0rd321#@192.168.56.102:3128/";
Acquire::https::Proxy "http://init@PassW0rd321#@192.168.56.102:3128/";
```



# Ubuntu代理服务器

## Clash for Windows

直接下载解压即可。记得sudo chmod +x cfw。

## √Clash for Linux

```bash
wget https://git.opclash.com/kehuduan/clash/clash-linux-amd64-v1.18.0.gz
sudo gzip -d clash-linux-amd64-v1.18.0.gz
sudo chmod +x clash-linux-amd64-v1.18.0
sudo mv clash-linux-amd64-v1.18.0 /usr/bin/clash

clash -v
```

需要注意：

1：clash 默认会在 ~/.config/clash 目录下生成两个配置文件 config.yaml 和 Country.mmdb。
2：如果提示 Country.mmdb invalid (即Country.mmdb配置无效)，可下载示例文件替换：[Country.mmdb示例文件下载](https://cdn.hftech.ltd/download/clash/Country.mmdb)。
3：在其他平台获取可用的 config.yaml 配置文件后，可替换原来 ~/.config/clash 目录下的配置文件，也可以在运行clash时，使用 -f 指定配置文件，示例：*clash -f config.yaml* (运行clash并指定配置文件为./config.yaml)，或者直接在服务中映射目录。
4: clash成功运行后，使用浏览器访问 [http://127.0.0.1:9090](http://127.0.0.1:9090/) 会出现 “hello clash”, 7890: http代理端口, 7891: socket代理端口。

先创建文件夹，并复制配置文件：

```bash
sudo mkdir /etc/clash
#按需修改路径复制，
#也可以直接 sudo touch /etc/clash/config.yaml
#sudo vi /etc/clash/config.yaml
#然后复制粘贴Windows下的配置文件
sudo cp config.yaml /etc/clash/
sudo cp Country.mmdb /etc/clash/
```

需要修改Windows下的配置文件，修改监听端口：

```yaml
external-controller: '127.0.0.1:9090'
#修改为
external-controller: '0.0.0.0:9090'
```

- 将clash命令设置为系统命令

```bash
# 创建clash.service文件，并编辑该文件
sudo vi /etc/systemd/system/clash.service
# 按i进入编辑模式，粘贴以下内容
[Unit]
Description=clash
[Service]
Type=simple
ExecStart=/usr/bin/clash  -d /etc/clash
# 按ESC 然后：wq保存退出。
```

在/目录下的.config/clash/下，创建config.yml和Country.mmdb。可以直接复制以上下载的mmdb。yml配置文件内容从windows下的文件中直接复制。

```bash
# 依次执行如下命令(reload: 刷新守护进程, enable: 开启自启动, start: 启动, status: 查看状态)
sudo systemctl daemon-reload
sudo systemctl enable clash
sudo systemctl start clash
sudo systemctl status clash
# 访问localhost:9090查看clash是否正确运行。
```

- 设置clash界面

  下载yacd，解压到httpd-default。

  配置完毕后，即可使用。

  浏览器访问：http://127.0.0.1:8234/yacd，即可看到界面。

  （或者直接使用网络上的：https://clash.razord.top/#/proxies。）

5、为确保万无一失，打开 Ubuntu 的“设置”管理，在“网络”内将“网络代理”的“自动”改为“手动”，根据以下内容进行填写 HTTP、HTTPS 和 Socks 地址，或：参考以下内容

### Clash for Linux 完整参考内容：

#### Clash 下载

在 Clash [release](https://github.com/Dreamacro/clash/releases) 页面下载相应的版本，对于 Ubuntu 一般使用 [clash-linux-amd64](https://github.com/Dreamacro/clash/releases/download/v1.14.0/clash-linux-amd64-v1.14.0.gz) 版本：

```bash
wget https://github.com/Dreamacro/clash/releases/download/v1.14.0/clash-linux-amd64-v1.14.0.gz
```

> 如果直接 wget 速度较慢的话，可以本地下载完成后，使用 SFTP 上传到 Linux 服务器。

然后使用 `gunzip` 命令解压，并重命名为 `clash`：

```bash
gunzip clash-linux-amd64-v1.14.0.gz
mv clash-linux-amd64-v1.14.0 clash
```

为 clash 添加可执行权限：

```bash
chmod u+x clash
```

Clash 运行时需要 `Country.mmdb` 文件，当第一次启动 Clash 时（使用 `./clash` 命令） 会自动下载（会下载至 `/home/XXX/.config/clash` 文件夹下）。自动下载可能会因网络原因较慢，可以访问该[链接](https://github.com/Dreamacro/maxmind-geoip/releases)手动下载。

> `Country.mmdb` 文件利用 GeoIP2 服务能识别互联网用户的地点位置，以供规则分流时使用。

##### 配置文件

一般的网络服务提供了 Clash 订阅链接，可以直接下载链接指向的文件内容，保存到 `config.yaml` 中。或者使用订阅转换服务（如该[链接](https://converter.niallapi.top/)。也可以自行搭建，可参考[该文章](https://codeswift.top/posts/docker-subscription-converter/)），将其它订阅转换为 Clash 订阅。

这里推荐使用订阅转换服务，转换后的配置文件已添加更为强大的分流规则。就可以将 Clash 一直保持后台运行，自动分流，且会自动选择最优节点。

> Clash 配置文件的完整参数介绍见[官方文档](https://github.com/Dreamacro/clash/wiki/configuration)。

> 如果使用订阅转换服务，对于转换后的订阅链接，可以使用以下命令来下载配置文件：
>
> ```bash
> curl -o config.yaml 'longURL'
> ```
>
> 对于 `suo.yt` 短链接，需要重定向，因此使用以下命令来下载配置文件：
>
> ```bash
> curl -L -o config.yaml 'shortURL'
> ```

#### Clash as a daemon

将 Clash 转变为系统服务，从而使得 Clash 实现常驻后台运行、开机自启动等。

> 普通用户需要 `sudo` 权限。

##### 配置 systemd 服务

Linux 系统使用 systemd 作为启动服务器管理机制，首先把 Clash 可执行文件拷贝到 `/usr/local/bin` 目录，相关配置拷贝到 `/etc/clash` 目录。

```bash
sudo mkdir /etc/clash
sudo cp clash /usr/local/bin
sudo cp config.yaml /etc/clash/
sudo cp Country.mmdb /etc/clash/
```

创建 systemd 服务配置文件 `sudo vim /etc/systemd/system/clash.service`：

```ini
[Unit]
Description=Clash daemon, A rule-based proxy in Go.
After=network.target

[Service]
Type=simple
Restart=always
ExecStart=/usr/local/bin/clash -d /etc/clash

[Install]
WantedBy=multi-user.target
```

### 使用 systemctl

使用以下命令，让 Clash 开机自启动：

```bash
sudo systemctl enable clash
```

然后开启 Clash：

```bash
sudo systemctl start clash
```

查看 Clash 日志：

```bash
sudo systemctl status clash
sudo journalctl -xe
```

#### 使用代理

##### 利用 `Export` 命令使用代理

Clash 运行后，其在后台监听某一端口。Ubuntu 下使用代理，需要 `export` 命令。根据 config 配置文可以查看到件Clash 代理端口（订阅转换后，端口为7890），设置系统代理命令为：

```bash
export https_proxy=http://127.0.0.1:7890 http_proxy=http://127.0.0.1:7890 all_proxy=socks5://127.0.0.1:7890
```

可以将该命令添加到 `.bashrc` 中，登陆后该用户自动开启代理。

取消系统代理：

```bash
unset  http_proxy  https_proxy  all_proxy
```

> 一般下载数据集时，记得取消代理。

##### DashBoard 外部控制

外部控制端口为 9090，因此也可以访问该[链接](http://clash.razord.top/)，输入 IP 地址（需本机可以访问的 IP）以及端口号 9090，来进入 Clash Dashboard 进行节点的选择。也可以在服务器自行搭建 Clash Dashboard，请参见[该项目](https://github.com/Dreamacro/clash-dashboard)。不过 Clash Dashboard 用处不大，使用**订阅转换**后的配置文件包含了自动选择的功能，Clash 会自动选择延迟最低的节点。

##### 设置密码

`export` 命令其他用户执行后也可以使用该代理，此时通过可以更换代理端口、添加密码等措施加以限制。修改 `/etc/clash/config.yaml` 文件部分配置：

```yaml
mixed-port: 12345
authentication:
  - "用户名1:密码1"
  - "用户名2:密码2"
allow-lan: true
mode: Rule
log-level: info
external-controller: :9090
```

`mixed-port: 12345` 就是混合代理端口，即使用代理时所指定的端口。然后需要重启 Clash，命令为：

```bash
sudo systemctl restart clash
```

根据上述配置文件，`export` 命令变为

```bash
export https_proxy=http://用户名1:密码1@127.0.0.1:12345 http_proxy=http://用户名1:密码1@127.0.0.1:12345 all_proxy=socks5://用户名1:密码1@127.0.0.1:12345
```

##### TUN 模式

新版的 Clash Premium 内核支持 TUN 模式，且目前已支持 Linux 系统下的 `auto-route` 和 `auto-detect-interface`，无需手动设置转发表，可以方便快捷的实现 **透明网关（旁路由）** 的功能。

首先需要下载 [Clash Premium](https://github.com/Dreamacro/clash/releases/tag/premium) 版本，替换上面的 `clash` 文件。接着需要设置 Linux 系统，开启转发功能。编辑文件 `/etc/sysctl.conf`，添加以下内容：

```ini
net.ipv4.ip_forward=1
```

保存退出后，执行以下命令使修改生效：

```
1sudo sysctl -p
```

然后接着需要关闭系统的 DNS 服务，使用以下命令：

```bash
sudo systemctl stop systemd-resolved
sudo systemctl disable systemd-resolved
```

> 关于代理环境下 DNS 解析行为的深入探讨，可以参见[浅谈在代理环境中的 DNS 解析行为](https://blog.skk.moe/post/what-happend-to-dns-in-proxy/)以及[我有特别的 DNS 配置和使用技巧](https://blog.skk.moe/post/i-have-my-unique-dns-setup/)。

接着需要设置 Clash 的配置文件，添加以下内容：

```yaml
dns:
  enable: true
  listen: 0.0.0.0:53
  enhanced-mode: fake-ip
  nameserver:
    - 114.114.114.114
  fallback:
    - 8.8.8.8
tun:
  enable: true
  stack: system # or gvisor
  dns-hijack:
    - 8.8.8.8:53
    - tcp://8.8.8.8:53
    - any:53
    - tcp://any:53
  auto-route: true # auto set global route
  auto-detect-interface: true # conflict with interface-name
```

最后重启 Clash 服务即可，这样流量就会通过 TUN 接口转发，同时利用强大的分流规则，实现按需代理。也可以设置局域网内的网关地址和 DNS 服务器地址，实现透明网关。





## docker-clash

- 下载 [docker-clash](https://hub.docker.com/r/dreamacro/clash) 镜像

- ```bash
  sudo docker pull dreamacro/clash
  # 启动 docker-clash (-p: 设置端口映射; -v: 设置目录/文件映射)
  sudo docker run -d --name clash -p 7890:7890 -p 7891:7891 -p 9090:9090 -v ~/.config/clash/config.yaml:/root/.config/clash/config.yaml -v ~/.config/clash/clash-ui:/opt/clash/ui dreamacro/clash
  ```



## 在 WSL2 中使用 Clash for Windows 代理连接

https://eastmonster.github.io/2022/10/05/clash-config-in-wsl/#

因为网络问题，在 WSL2 中配环境时通常会非常缓慢甚至失败。在整合了各方资料及个人实践后，我总结了在 WSL2 中通过 Clash for Windows 连接代理的方法，供大家参考。

1.在 Clash 客户端内的操作

打开客户端的允许局域网连接 (Allow LAN) 开关，如图 1 所示。
记录下客户端内的端口 (Port)，如这里是 `16514`. 一般默认端口为 `7890`.
[![图 1](https://s2.loli.net/2024/01/22/sla3HO9UVu4wozn.png)](https://s2.loli.net/2024/01/22/sla3HO9UVu4wozn.png)

2.配置

方法一

这个方法需要 Windows 10/11 22H2 和 WSL 2.0.5 及以上版本，如果你不满足版本要求，请参考[方法二](https://eastmonster.github.io/2022/10/05/clash-config-in-wsl/#方法二)。
你可以使用 `winver` 和 `wsl -v` 查看你的 Windows 和 WSL 版本。

如果你使用这个方法，你需要确保 Clash 客户端中设置的系统代理类型为 HTTP 而不是 PAC，否则无法代理 HTTPS 请求:
[![img](https://s2.loli.net/2024/01/24/Y9DhxRw47tzlWIH.png)](https://s2.loli.net/2024/01/24/Y9DhxRw47tzlWIH.png)
如果你确实需要使用 PAC 模式，请参考方法二，在 shell 的配置文件中添加:

```bash
export https_proxy=$http_proxy
```

在 WSL [2.0.5 版本](https://github.com/microsoft/WSL/releases/tag/2.0.5)后，一些特性得以稳定，这里要用到的是[镜像模式网络](https://learn.microsoft.com/zh-cn/windows/wsl/networking#mirrored-mode-networking)。

在 `C:\Users\<UserName>\.wslconfig` 文件中 (如果不存在就手动创建一个) 加入以下内容:

```config
[wsl2]
networkingMode=mirrored
```

不过需要注意，shell 的配置文件中**不能**再 `export http_proxy=...`.

方法二

WSL [2.2.1 版本](https://github.com/microsoft/WSL/releases/tag/2.2.1)默认启用了 DNS 隧道, 会导致**该方法**失效。如果你正在使用**该版本**可以参照[官方文档](https://learn.microsoft.com/zh-cn/windows/wsl/wsl-config#wslconfig)配置，在 `C:\Users\<UserName>\.wslconfig` 文件中 (如果不存在就手动创建一个) 加入以下内容以关闭 DNS 隧道:

```config
[wsl2]
dnsTunneling=false
```

你可以执行 `wsl -v` 查看你的 WSL 版本。

这里默认你使用 bash, 如果你使用其他的 shell, 我相信你知道该改哪里。

编辑 `.bashrc`，在文件末尾根据第 1 步记录的数据，加入三行内容：

```bash
host_ip=$(cat /etc/resolv.conf |grep "nameserver" |cut -f 2 -d " ")
export http_proxy="http://$host_ip:[端口]"
export https_proxy="http://$host_ip:[端口]"
```

如图 2 所示，修改完后保存即可。
[![图 2](https://s2.loli.net/2022/10/05/qWgwrFU7OMupiH2.png)](https://s2.loli.net/2022/10/05/qWgwrFU7OMupiH2.png)

输入 `source ~/.bashrc` 使文件修改生效，可以通过输入 `echo $http_proxy` 验证修改是否成功。

3.使 root 用户保留先前设置的环境变量

在使用 `sudo` 执行命令时，之前设置的环境变量并不会生效，需要进一步设置。
输入 `sudo visudo`, 加入以下内容并保存，如图 3-1 所示。

```bash
Defaults env_keep="http_proxy https_proxy"
```

[![图 3-1](https://s2.loli.net/2022/10/05/z4Q9PwhFmgRLG8d.png)](https://s2.loli.net/2022/10/05/z4Q9PwhFmgRLG8d.png)

**如果你使用 Ubuntu 22.04**, 则你看到的内容应与图 3-1 不同。如下图所示，取消相应行的注释 (删除行首的 `#`) 即可.
[![图 3-2](https://s2.loli.net/2023/10/13/Dp628PahyF1jOVl.png)](https://s2.loli.net/2023/10/13/Dp628PahyF1jOVl.png)

4. 测试

输入 `wget www.google.com`, 可以发现连接成功，至此大功告成，如图 4 所示。
[![图 4](https://s2.loli.net/2022/10/05/UIQ8LhuROaXq6S2.png)](https://s2.loli.net/2022/10/05/UIQ8LhuROaXq6S2.png)





# Ubuntu自动挂载磁盘分区(NTFS)

## 方式一

安装ubuntu后才发现分配给ubuntu系统的磁盘容量根本不够，打算将windows的磁盘空间挤出来给ubuntu用，或者后续添加硬盘，挂载到ubuntu.

1. 在windows系统中，控制面板进入磁盘管理，选择要挤出空间的分区，右键压缩磁盘得到适当的空闲磁盘空间，结束后会得到free space（空闲未分配的分区）。
2. 进入ubuntu系统，执行以下命令安装gparted分区管理工具：

```bash
sudo apt install gparted
```

安装完成，运行之：

```bash
sudo gparted
```

1. 进入gparted软件界面，选择挤出来的那个分区，右键创建新分区，类型和/目录一样，都是主分区(primary)，大小就是全部，格式为ext4，然后，点击上面的淡绿色的对勾，应用(apply)分区更改。
   结束后，分区表会自动更新，退出软件，最好重启。
2. 在你想要挂载的路径下创建挂载文件夹：

```bash
mkdir /home/xc/space1
BASH 复制 全屏
```

1. 找到新分区的名称，可以采用如下命令查看要挂载的分区（通过大小区分）：

```bash
df -l
```

输出类似如下：

```plain
Filesystem     1K-blocks     Used Available Use% Mounted on
udev             7992024        0   7992024   0% /dev
tmpfs            1604316     2416   1601900   1% /run
/dev/nvme0n1p7  75664204 52652344  19122608  74% /
tmpfs            8021580        0   8021580   0% /dev/shm
tmpfs               5120        4      5116   1% /run/lock
tmpfs            8021580        0   8021580   0% /sys/fs/cgroup
/dev/loop1        136576   136576         0 100% /snap/chromium/2036
/dev/loop3         56832    56832         0 100% /snap/core18/2128
/dev/loop2         56960    56960         0 100% /snap/core18/2538
/dev/loop4        224256   224256         0 100% /snap/gnome-3-34-1804/77
/dev/loop5          2688     2688         0 100% /snap/gnome-calculator/920
/dev/loop6          2560     2560         0 100% /snap/gnome-calculator/884
/dev/loop7        247168   247168         0 100% /snap/gnome-3-38-2004/70
/dev/loop0           128      128         0 100% /snap/bare/5
/dev/loop8           768      768         0 100% /snap/gnome-characters/741
/dev/nvme0n1p6   9740796   140816   9083580   2% /boot
/dev/nvme0n1p8  81988552       12  77777852   1% /home/xc/space1
/dev/nvme0n1p1    262144    36848    225296  15% /boot/efi
/dev/loop9         63488    63488         0 100% /snap/core20/1587
/dev/loop10         2688     2688         0 100% /snap/gnome-system-monitor/178
/dev/loop11          768      768         0 100% /snap/gnome-characters/726
/dev/loop12        48128    48128         0 100% /snap/snapd/16292
/dev/loop13         2560     2560         0 100% /snap/gnome-system-monitor/163
/dev/loop14          640      640         0 100% /snap/gnome-logs/112
/dev/loop15       224256   224256         0 100% /snap/gnome-3-34-1804/72
/dev/loop16       410496   410496         0 100% /snap/gnome-3-38-2004/112
/dev/loop17        63360    63360         0 100% /snap/core20/1081
/dev/loop18        66688    66688         0 100% /snap/gtk-common-themes/1515
/dev/loop19        93952    93952         0 100% /snap/gtk-common-themes/1535
/dev/loop20          640      640         0 100% /snap/gnome-logs/106
tmpfs            1604316       16   1604300   1% /run/user/121
tmpfs            1604316       20   1604296   1% /run/user/1000
```

`/dev/nvme0n1p8 81988552 12 77777852 1% /home/xc/space1`是我已经挂载了的，一般会默认挂载到/media目录下。

1. 查看要挂载分区的块标志符block id(blkid)，如目标分区是'/dev/nvme0n1p8'，则查询：

```bash
sudo blkid /dev/nvme0n1p8
```

输出类似如下：

```plain
/dev/nvme0n1p8: UUID="df97552f-4af8-4469-a65a-7098fd5733ac" TYPE="ext4" PARTUUID="13dbfc2f-c5bd-4b9c-ab6d-6af72d4df077"
```

记录下UUID： UUID="df97552f-4af8-4469-a65a-7098fd5733ac"

1. 修改`/etc/fstab`持久化挂载映射表：

```bash
sudo gedit /etc/fstab
```

添加如下内容：

```bash
UUID=df97552f-4af8-4469-a65a-7098fd5733ac /home/xc/space1 ext4   defaults         0       2
```

其中uuid就是上面blkid查询得到的分区块标志符，而第二列是要挂载的目录，最好挂载到`/home/xc/`下，这样没有权限问题。
第三列是文件格式，选择ext4。第四列是挂载参数，如自动挂载等，保持defaults就好。
第五列的0表示不备份分区，1表示备份，通常不备份。
第六列的2表示系统磁盘fsck自检查的等级，0表示不检查磁盘，1表示最先检查，可以同时检查也可靠后，通常填2即可。
具体的fstab的字段解释参考网址：https://blog.csdn.net/imliuqun123/article/details/109289716 .

1. 保存好fstab文件后重启系统，此时可以看到自动挂载的文件夹。但是文件夹有个🔓，表示需要root权限才能访问，因此，我们需要更改其权限为本用户所有：

```shell
sudo nautilus /home/xc/space1
```

设置文件夹权限为所有者为本用户xc。权限为创建与删除。
退出，就OK了。

## 方式二(推荐)

ubuntu 20.04挂载ntfs磁盘步骤如下：

- 查看当前用户id和组id，记下uid和gid

```shell
id
```

显示结果如下：
uid=1000(scue) gid=1000(scue) 组=1000(scue),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),109(lpadmin),124(sambashare),125(vboxusers),1001(usbfs)

- 查看当前磁盘的UUID和分区表信息

```shell
sudo blkid
```

显示结果如下：
/dev/sda1: UUID=“DA18EBFA09C1B27D” TYPE=“ntfs” PTTYPE=“atari” PARTLABEL=“Basic data partition” PARTUUID=“7fd2a2dd-d633-4740-bf96-6db2540b0f3a”

- 创建挂载目录

```shell
sudo mkdir -p /home/xxx/data2
```

此步骤不创建目录的话会导致挂载失败。

- 打开/etc/fstab

```shell
sudo gedit /etc/fstab
```

添加如下内容：

```shell
UUID="DA18EBFA09C1B27D"     /home/xx/data2    ntfs defaults,locale=zh_CN.UTF-8 0   0
```

- 验证挂载

```shell
sudo mount -a
```



# 解决ubuntu20.04系统不发出声音、显示伪输出（dummy output）的问题(Intel板载声卡)

**一、切换到root管理员权限vim 编辑**
/etc/modprobe.d/alsa-base.conf文件
在文件中加入
options snd-hda-intel dmic_detect=0
或直接输入以下命令回车
echo options snd-hda-intel dmic_detect=0 | sudo [tee](https://so.csdn.net/so/search?q=tee&spm=1001.2101.3001.7020) -a /etc/modprobe.d/alsa-base.conf
**二、再编辑**
/etc/modprobe.d/blacklist.conf文件
加入
blacklist snd_soc_skl
或直接输入以下命令回车
echo blacklist snd_soc_skl | sudo tee -a /etc/modprobe.d/blacklist.conf
重启系统
reboot

参考以下脚本

```bash
cd /etc/modprobe.d
sudo cp ./alsa-base.conf ./alsa-base.conf.BAK
sudo cp ./blacklist.conf ./blacklist.conf.BAK

sudo vi ./alsa-base.conf
# 在文件末尾插入 options snd-hda-intel dmic_detect=0
sudo vi ./blacklist.conf
# 在文件末尾插入 blacklist snd_soc_skl

sudo reboot
```







# 安装和配置 Fail2ban 以保护 SSH

Fail2ban 介绍

Fail2Ban 是一款开源的防护软件，旨在保护计算机系统免受恶意攻击和暴力破解。它通过监视系统日志文件，检测到多次失败的登录尝试、恶意行为或其他不寻常活动，然后采取预先配置的措施来应对这些威胁，比如暂时封禁攻击者的 IP 地址。

以下是 Fail2Ban 的一些关键特点和功能：

1. **监视日志文件**：Fail2Ban 主要通过监视系统的日志文件来检测潜在的安全问题。它可以监视 SSH、FTP、SMTP 等服务的日志文件，并分析其中的登录尝试、错误信息等。
2. **基于规则的匹配**：Fail2Ban 使用基于正则表达式的规则来匹配日志文件中的事件。通过配置适当的规则，可以识别恶意行为，如暴力破解密码、拒绝服务攻击等。
3. **自动封禁**：一旦 Fail2Ban 检测到恶意行为，例如多次失败的登录尝试，它会自动封禁攻击者的 IP 地址。封禁可以是暂时的，也可以是永久的，具体取决于配置。
4. **灵活的配置**：Fail2Ban 的配置非常灵活，可以根据特定的需求和威胁模式进行定制。用户可以配置规则、封禁时长、白名单等参数，以适应不同的环境和安全策略。
5. **日志和报告**：Fail2Ban 提供了详细的日志和报告，记录了检测到的事件、封禁的 IP 地址以及采取的措施。这些日志和报告对于审计和调查安全事件非常有用。

Fail2Ban 是一款功能强大的工具，可以帮助系统管理员保护计算机系统免受各种网络攻击的威胁，提高系统的安全性和稳定性。它已经被广泛应用于各种网络环境中，包括个人计算机、企业服务器和云平台。

工作原理

Fail2ban 是一款功能强大的工具，它可以通过监控日志文件（例如 /var/log/auth.log 和 /var/log/apache/access.log）来帮助保护您的服务器免受暴力攻击。该软件可以检测到多次登录系统的失败尝试，然后在一定时间内自动禁止违规 IP 地址。Fail2ban 可以执行各种操作，包括更新 Iptable 防火墙规则、将 IP 地址添加到 TCP Wrapper 的 hosts.deny 表、发送电子邮件通知以及执行任何用户定义的操作。

在这篇文章中，我们将向您展示如何安装和配置 Fail2ban 以保护您的服务器免受 SSH 和 apache 基本身份验证的暴力登录攻击。

安装

要在您的服务器上安装 Fail2ban，您首先需要确保您具有管理权限。确认这一点后，您可以继续使用您选择的包管理器安装软件。例如，在 Ubuntu 上，您可以使用以下命令：

```bash
sudo apt-get install fail2ban -y    
```

检查状态

```bash
sudo systemctl status fail2ban
```

配置

所有配置文件都位于 /etc/fail2ban/ 目录中。您应该能够看到目录中的文件，如下所示：

![img](https://nas.yanghong.dev:8200/wp-content/uploads/2024/05/image-11-1024x262.png)

jail.conf 是主配置文件，包含所有可用选项。它包含 HTTP、SSH、FTP、Webmail、WebApplications 等服务的配置文件。一些最常用的微调参数是：

- **bantime**：主机被禁止的秒数。
- **findtime**：如果主机在“findtime”期间生成了“maxretry”，则会被禁止。
- **maxretry**：主机被禁止之前的失败次数。
- **ignoreip**：fail2ban 将忽略的 IP 地址。
- **port**：服务名称或端口。
- **logpath**：fail2ban 检查的日志文件路径。

一般不建议直接更改 jail.conf 文件。可以拷贝一个新的 jail.local 文件来管理 SSH 的安全，这样可以轻松跟踪 jail，并最大限度地减少错误配置的机会。

```bash
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
```

**为 SSH 配置 Fail2Ban**

在 ubuntu 上，ssh 的 fail2ban 已默认配置。您可以使用以下命令验证 jail 是否已启用：

```bash
sudo fail2ban-client status
```

![img](https://nas.yanghong.dev:8200/wp-content/uploads/2024/05/image-13-1024x66.png)

但是需要手动配置 jail.local 文件。在该文件中找到 [sshd]并进行配置。如下是示范，可以按照自己的实际情况更改。

```bash
[sshd]

# To use more aggressive sshd modes set filter parameter "mode" in jail.local:
# normal (default), ddos, extra or aggressive (combines all).
# See "tests/files/logs/sshd" or "filter.d/sshd.conf" for usage example and details.
#mode   = normal
enable  = true # 打开sshd保护
port    = ssh # ssh端口。也可以是端口号
filter  = sshd
logpath = %(sshd_log)s
backend = %(sshd_backend)s
bantime = 7d # block 7 days, 604800s
findtime        = 1d # fail in 1 hour

maxretry = 5 # max retry 5 times
```

更改完配置文件后需要重新启动服务以生效。可以使用以下命令完成此操作：

```bash
sudo systemctl restart fail2ban.service
```

检查监狱状态和禁止/取消禁止 IP

我们可以使用以下命令检查监狱的状态：

```bash
sudo fail2ban-client status
```

![img](https://nas.yanghong.dev:8200/wp-content/uploads/2024/05/image-14-1024x58.png)

我们可以通过故意输入错误的 ssh 身份验证参数来测试这一点。达到配置的 maxretry 后，该 ip 将被列入配置 bantime 的黑名单。那些偷偷摸摸干坏事的 IP 也列出来亮个相。

![img](https://nas.yanghong.dev:8200/wp-content/uploads/2024/05/image-15-1024x152.png)

检查位于 /var/log/fail2ban.log 的 fail2ban 日志后，我们可以看到 fail2ban 采取的措施

```bash
2024-05-04 13:15:38,352 fail2ban.actions        [138750]: NOTICE  [sshd] Restore Ban 120.26.123.230
2024-05-04 13:15:38,373 fail2ban.actions        [138750]: NOTICE  [sshd] Restore Ban 182.92.180.6
2024-05-04 13:15:38,376 fail2ban.actions        [138750]: NOTICE  [sshd] Restore Ban 193.201.9.156
2024-05-04 13:15:38,380 fail2ban.actions        [138750]: NOTICE  [sshd] Restore Ban 193.32.162.36
2024-05-04 13:15:38,387 fail2ban.actions        [138750]: NOTICE  [sshd] Restore Ban 64.227.140.112
2024-05-04 13:15:38,398 fail2ban.actions        [138750]: NOTICE  [sshd] Restore Ban 85.209.11.227
```

为了删除 IP 地址，我们可以发出以下命令：

```bash
sudo fail2ban-client set sshd unbanip <REMOTE-IP-ADDRESS> 
```

我们还可以使用以下命令禁止特定的 IP：

```bash
sudo fail2ban-client set sshd banip <REMOTE-IP-ADDRESS>
```



# ubuntu 如何放开端口

 在 Ubuntu 上放开端口，可以通过以下步骤进行操作：

1. 使用 root 用户或具有管理员权限的用户登录到 Ubuntu 系统。
2. 使用防火墙工具（如ufw）放开特定的端口。ufw 是 Ubuntu 上的一个简化的防火墙配置工具，可以轻松管理端口和防火墙规则。
   - 检查 ufw 是否已安装：在终端中执行 `sudo ufw version`，如果显示版本信息，则已安装。
   - 如果 ufw 未安装，可以使用以下命令进行安装：`sudo apt install ufw`
3. 开放指定的端口：执行以下命令来放开特定的端口，例如放开 80 端口：

```bash
sudo ufw allow 80
```

1. 如果要放开其他端口，请将 `80` 替换为相应的端口号。
2. 验证端口是否已放开：可以执行以下命令来查看 ufw 的状态和已开放的端口：

```bash
sudo ufw status
```

这将显示当前的防火墙规则和已启用的端口。

请注意，放开端口也意味着允许外部访问该端口，因此在放开端口之前，请确保你了解安全风险，并只放开需要的端口。另外，如果你的系统上运行了其他防火墙工具（如iptables），你需要根据具体工具的使用方式进行相应配置。



# springboot 服务制作成 systemd 服务

**注意！防火墙开放8900端口**

2、如何制作 systemd 服务

首先，我们需要创建一个名为 `my-spring-boot-app.service` 的 systemd 服务文件。在 Linux 中，通常将此文件存储在 `/etc/systemd/system/` 目录下。

my-spring-boot-app.service 内容为：

```ini
[Unit]
Description=My Spring Boot Application  
#名称描述
After=syslog.target 
#告诉 systemd 在哪个主要系统服务后启动本服务

[Service]
User=myuser 
#应用程序运行的用户
WorkingDirectory=/path/to/my/springboot/app 
#应用程序的工作目录
Group=mygroup 
#组信息
ExecStart=/usr/bin/java -Xms256m -Xmx512m -jar /path/to/my/springboot/app/my-spring-boot-app.jar --server.port=8080
#或直接使用脚本sh文件：/mnt/toshiba/server/run/systemd_v1_0_1_localhost.sh
SuccessExitStatus=143 
#应用程序退出时的退出状态码，设置为 143 表示systemctl stop 命令下服务可以快速关闭
TimeoutStopSec=10 
#表示在关闭服务前等待的时间
Restart=always 
#设置服务崩溃时自动重启服务
RestartSec=5 
#在崩溃后五秒重试
LimitNOFILE=65536 
#表示服务最多可以打开 65536 个文件
ExecStop=/usr/bin/pkill -f my-spring-boot-app.jar 
#执行 systemctl 时执行的命令
#ExecStop=/bin/kill -15 $MAINPID

[Install]
WantedBy=multi-user.target
#服务会在 default.target 启动后启动，也可修改为multi-user.target
```

3、重新加载 systemd 配置

添加服务文件后，需要重新加载 systemd 配置以更新系统。使用以下命令重新加载 systemd 配置：

```bash
sudo systemctl daemon-reload
```

4、启动 systemd 服务

```bash
sudo systemctl start my-spring-boot-app
```

5、查看服务状态

```bash
sudo systemctl status my-spring-boot-app
```

6、停止服务

```bash
sudo systemctl stop my-spring-boot-app
```

7、设置服务自启动和禁止自启动

```bash
sudo systemctl enable my-spring-boot-app   #设置服务自启动
sudo systemctl disable my-spring-boot-app  #取消服务自启动
```





# 开机自动运行命令

## 第0种添加系统启动脚本方式：

Ubuntu 20.04的服务管理是基于[systemd](https://github.com/systemd/systemd)的，因此设置服务自启动最推荐的方法是创建一个systemd服务文件，配置好要执行的服务。过程如下:

1. 创建我们需要开机自启动的脚本，例如/mnt/toshiba/server/run/run_v1_0_1_localhost.sh，其内容如下：

```bash
#!/bin/bash

Your command here
```

**注意，开头一定要加上：**

```bash
#!/bin/bash
```

2. 在`/etc/systemd/system/`目录下创建一个systemd服务文件, 命名为`video_station.service`, 内容如下：

```bash
[Unit]
After=network.service

[Service]
User=root
Group=root
WorkingDirectory=/mnt/toshiba/server/run/
Environment="PATH=/mnt/toshiba/server/run/"
ExecStart=/mnt/toshiba/server/run/run_v1_0_1_localhost.sh
ExecReload=/bin/kill -s HUP $MAINPID
RestartSec=5

[Install]
WantedBy=default.target
```

其中，After表示服务何时启动，After=network.service 表示网络连接完成后，启动我们的服务；ExecStart表示我们的脚本（步骤1中的test.sh)的路径；WantedBy默认填default.target。

注意，ExecStart=/home/ubuntu/video_station.sh 这里一定不能用 ~/ 来代替/home/$USER

3. 将systemd服务文件和我们的脚本更改权限，使其可执行。

```bash
sudo chmod 744 /mnt/toshiba/server/run/run_v1_0_1_localhost.sh
sudo chmod 664 /etc/systemd/user/video_station.service
```

4. 重新加载系统的systemd服务文件，并启用我们自己写的video_station.service文件。

```bash
sudo systemctl daemon-reload
sudo systemctl --user enable video_station.service
```

[参考文章](https://linuxconfig.org/how-to-run-script-on-startup-on-ubuntu-20-04-focal-fossa-server-desktop)

## 第一种shell脚本方式

1、进入 /etc/init.d/ 目录

```bash
cd /etc/init.d/
```

2、新建一个自定义名称的sh脚本，这里以 xxx 名称为例建立一个 xxx.sh 的脚本

```bash
vi video_station.sh        # xxx为你的脚本文件名
```

video_station.sh 脚本内容，编辑好以后保存退出 :wq

```bash
### BEGIN INIT INFO
# Provides:          svnd.sh
# Required-start:    $local_fs $remote_fs $network $syslog
# Required-Stop:     $local_fs $remote_fs $network $syslog
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: starts the svnd.sh daemon
# Description:       starts svnd.sh using start-stop-daemon
### END INIT INFO
 
 
bt restart        # 这里为需要执行的命令
```

注：脚本内容必须包含 ### BEGIN INIT INFO  ......  ### END INIT INFO ,不然会报错误：***\*missing LSB tags and overrides\****

3、赋予脚本权限

```bash
sudo chmod 775 ./video_station.sh        
# xxx为你的脚本文件名 
```

4、然后执行以下命令

```bash
sudo update-rc.d video_station.sh defaults 90        
 # xxx为你的脚本文件名
```

5、最后重启系统即可：reboot

附：取消开机启动脚本

```bash
cd /etc/init.d
# 进入/etc/init.d目录
sudo update-rc.d -f video_station.sh remove
# 移除脚本，XXX为你的脚本文件名
```

## 第二种添加系统启动脚本方式：

0x01创建rc.local

ubuntu-18.04以后 默认是没有 /etc/rc.local 这个文件的，需要自己创建

```bash
sudo touch /etc/rc.local 
```

0x02写入开机自启的脚本

```bash
#!/bin/sh

echo "看到这行字，说明添加自启动脚本成功。" > /usr/local/test.log
exit 0
```

0x03给rc.local加执行权限

```bash
sudo chmod +x /etc/rc.local
```

0x04链接配置文件

systemd 默认读取 /etc/systemd/system 下的配置文件, 所以还需要在 /etc/systemd/system 目录下创建软链接

```bash
ln -s /lib/systemd/system/rc-local.service /etc/systemd/system/ 
```



# WSL Ubuntu systemctl报错

WSL 中暂时不支持 systemctl，所以使用下面的命令会报错。

```bash
sudo systemctl
```

报错信息为

> System has not been booted with systemd as init system (PID 1). Can't operate.
> Failed to connect to bus: Host is down

即系统没有通过 systemd 启动，所以不能操作。

WSL 的官方文档上也提到了这一点。

> 目前 WSL 不支持 `systemd`（Linux 中的服务管理系统）。

但是有折中的方法。

可以把 systemctl 命令换成 /etc/init.d/

例如

```bash
sudo systemctl status docker
```

换成下面这样

```bash
sudo /etc/init.d/docker status
```

也可以使用 service 命令

```bash
sudo service docker status
```







# Ubuntu关闭盖子/按电源键/按休眠键功能，关闭盖子不休眠

`HandlePowerKey`: 按下电源键后的行为，默认power off

`HandleSleepKey`: 按下挂起键后的行为，默认suspend

`HandleHibernateKey`: 按下休眠键后的行为，默认hibernate

`HandleLidSwitch`: 合上笔记本盖后的行为，默认suspend（改为ignore；即合盖不休眠）

```shell
sudo vi /etc/systemd/logind.conf
#然后将其中的：
#HandleLidSwitch=suspend
#改成下面，记得去掉“#”号：
HandleLidSwitch=ignore
#最后重启服务
sudo service systemd-logind restart
```

配置文件解析：

```shell
#  This file is part of systemd.
# 
#  systemd is free software; you can redistribute it and/or modify it
#  under the terms of the GNU Lesser General Public License as published by
#  the Free Software Foundation; either version 2.1 of the License, or
#  (at your option) any later version.
#
# Entries in this file show the compile time defaults.
# You can change settings by editing this file.
# Defaults can be restored by simply deleting this file.
#
# See logind.conf(5) for details.

[Login]
#配置分配的虚拟终端(VTs)数量，默认6个
#NAutoVTs=6

#标识一个将无条件预留给autovt@.的虚拟终端服务激活
#ReserveVT=6

#配置用户退出时是否停止该用户的进程
#KillUserProcesses=no

#设置进程被杀死的用户
#KillOnlyUsers=

#设置不被杀死进程的用户
#KillExcludeUsers=root

#指定系统关闭或睡眠请求延迟的最长时间，默认为5秒
#InhibitDelayMaxSec=5

#为电源操作，默认关机
#HandlePowerKey=poweroff

#为电源操作，默认休眠到内存
#HandleSuspendKey=suspend

#为电源操作，默认休眠到硬盘
#HandleHibernateKey=hibernate

#关盖操作，默认休眠
#HandleLidSwitch=ignore

#使用外部电源关盖操作，默认休眠
#HandleLidSwitchExternalPower=ignore

#插了扩展坞或者连接多个显示器关盖子操作，默认无操作
#HandleLidSwitchDocked=ignore

#电源事件所触发的systemd-logind操作是否应该忽略高级inhibitor锁
#PowerKeyIgnoreInhibited=no

#休眠(到内存)事件所触发的systemd-logind操作是否应该忽略高级inhibitor锁
#SuspendKeyIgnoreInhibited=no

#休眠(到磁盘)事件所触发的systemd-logind操作是否应该忽略高级inhibitor锁
#HibernateKeyIgnoreInhibited=no

#关盖子事件所触发的systemd-logind操作是否应该忽略高级inhibitor锁
#LidSwitchIgnoreInhibited=yes

#当开机/唤醒时间 尚未超出此处设置的时间长度时， systemd将延迟响应合上盖子(lid)硬件事件，默认30秒
#HoldoffTimeoutSec=30s

#当系统持续处于空闲状态超过 IdleActionSec= 所设置的时间后，应该执行何种操作，默认无操作。
#IdleAction=ignore

#当系统持续处于空闲状态超过此处设置的时长之后将会自动执行IdleAction=设定的动作
#IdleActionSec=30min

#为每个已登录用户设置运行时目录 $XDG_RUNTIME_DIR 的最大尺寸
#RuntimeDirectorySize=10%

#当用户完全退出登录时(也就是退出最后一个登录会话时)， 是否销毁所有属于该用户的 SysV 与 POSIX IPC 对象
#RemoveIPC=yes

#控制最多允许同时存在多少个inhibitor锁，默认值是8192(8K)
#InhibitorsMax=8192

#控制最多允许同时管理多少个用户会话，默认值是 8192 
#SessionsMax=8192
```



# Linux资源监控

1、nmon

```bash
sudo apt install nmon
```

2、glances(suggested)

```bash
sudo apt install glances
```

3、htop

```bash
sudo apt install htop
```



# Linux美化输出

```bash
#一个在 Linux 终端中输出彩虹特效的命令行工具
sudo apt install lolcat

#一款能够将文字转换为放大艺术字的命令
sudo apt install figlet
```







# Linux命令行多终端方案

tmux 由会话（session），窗口（window），面板（pane）组织起每个 shell 的输入框。会话用于区分不同的工作；窗口是会话中以显示屏为单位的不同的页；而面板则是一个窗口上被白线分割的不同区域。熟练掌握会话，窗口，面板之间的切换，可以极大提高使用效率。

下面先行讲解这一工具的用法：

```bash
sudo apt install tmux
tmux
```

首先，Ctrl + B 是 tmux 的全局前缀命令，按下该快捷键表示让 tmux 接收命令。

一些 tmux 中的常见功能

| 快捷键（需先按下 Ctrl + B） | 功能                                                         |
| :-------------------------- | :----------------------------------------------------------- |
| %                           | 左右分屏                                                     |
| "                           | 上下分屏                                                     |
| ↑ ↓ ← →                     | 焦点切换为上、下、左、右侧 pane，正在交互的 pane 被绿色框选中。 |
| d (detach)                  | 从 tmux 中脱离，回到命令行界面                               |
| z (zoom)                    | 将 pane 暂时全屏，再按一次恢复原状                           |
| c                           | 新建窗口                                                     |
| ,                           | 为窗口命名                                                   |
| s                           | 列出所有 session                                             |

刚才提到如果不幸掉线，会话仍然被保存在后台，如果再次登录，可使用 `tmux attach [-t 窗口名称]` 重新连接窗口，不加 `-t` 参数将默认连接最后一次打开的窗口。









# Ubuntu vim上下左右键乱码

新装的ubuntu系统，在使用vi命令操作文件时，按上下左右键时不正常工作，而是出现ABCD现象，以下列举三种方式解决：

## 方式一

普通用户下，执行 `cp /etc/vim/vimrc ~/.vimrc` ，然后关闭终端，重新打开终端就可以正常使用 `vi` 命令

但是这种方式有一个缺陷：如果编辑一个只能 `sudo` （管理员）权限的文件时，不能发挥作用

比如执行 `sudo vi /etc/apt/source.list` 命令时，无法正常使用上下左右键

## 方式二

安装[vim](https://so.csdn.net/so/search?q=vim&spm=1001.2101.3001.7020)编辑器，执行 `sudo apt-get install vim`，如果没有出现错误，再次进入vi 尝试一下，看看有没有修改过来，如果出现以下错误：
`E: Package 'vim' has no installation candidat`

则依次执行

```bash
sudo apt-get update
sudo apt-get remove vim-common
sudo apt-get install vim
```

再次使用 `vi` 编辑文件，就能正常使用上下左右键啦！

## 方式三

列举一个笨方法:如果方式一无法解决，使用方式二时系统源又无法安装 `vim` ，可以使用方式三，保准解决

1.换系统源

使用 `vi` 命令，在普通模式下，按 `shift+:` 进入命令模式，之后输入 `%d` 清空文件内容，从其他地方复制国内源到此文件中，保存文件退出

2.安装vim

```bash
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install vim
```

### 正常使用vi或vim编辑文件

再次使用 `vi` 就可以任意操作啦





# 常用命令

## 打包/解压

速查:

```sh
#压缩文件:压缩 a.c文件为test.tar.gz
tar -czvf test.tar.gz a.c
#列出压缩文件内容
tar -tzvf test.tar.gz 
#解压文件
tar -xzvf test.tar.gz 


#将 /home/html/ 这个目录下所有文件和文件夹打包为当前目录下的 html.zip：
zip -q -r html.zip /home/html
#如果在我们在 /home/html 目录下，可以执行以下命令：
zip -q -r html.zip *
#从压缩文件 cp.zip 中删除文件 a.c
zip -dv cp.zip a.c


#查看压缩文件中包含的文件
unzip -l abc.zip
#-v 参数用于查看压缩文件目录信息，但是不解压该文件
unzip -v abc.zip 
#解压文件到当前目录
unzip ./abc.zip
```

完整:

```shell
tar [-ABcdgGhiklmMoOpPrRsStuUvwWxzZ][-b <区块数目>][-C <目的目录>][-f <备份文件>][-F <Script文件>][-K <文件>][-L <媒体容量>][-N <日期时间>][-T <范本文件>][-V <卷册名称>][-X <范本文件>][-<设备编号><存储密度>][--after-date=<日期时间>][--atime-preserve][--backuup=<备份方式>][--checkpoint][--concatenate][--confirmation][--delete][--exclude=<范本样式>][--force-local][--group=<群组名称>][--help][--ignore-failed-read][--new-volume-script=<Script文件>][--newer-mtime][--no-recursion][--null][--numeric-owner][--owner=<用户名称>][--posix][--erve][--preserve-order][--preserve-permissions][--record-size=<区块数目>][--recursive-unlink][--remove-files][--rsh-command=<执行指令>][--same-owner][--suffix=<备份字尾字符串>][--totals][--use-compress-program=<执行指令>][--version][--volno-file=<编号文件>][文件或目录...]
#-A或--catenate 新增文件到已存在的备份文件。
#-b<区块数目>或--blocking-factor=<区块数目> 设置每笔记录的区块数目，每个区块大小为12Bytes。
#-B或--read-full-records 读取数据时重设区块大小。
#-c或--create 建立新的备份文件。
#-C<目的目录>或--directory=<目的目录> 切换到指定的目录。
#-d或--diff或--compare 对比备份文件内和文件系统上的文件的差异。
#-f<备份文件>或--file=<备份文件> 指定备份文件。
#-F<Script文件>或--info-script=<Script文件> 每次更换磁带时，就执行指定的Script文件。
#-g或--listed-incremental 处理GNU格式的大量备份。
#-G或--incremental 处理旧的GNU格式的大量备份。
#-h或--dereference 不建立符号连接，直接复制该连接所指向的原始文件。
#-i或--ignore-zeros 忽略备份文件中的0 Byte区块，也就是EOF。
#-k或--keep-old-files 解开备份文件时，不覆盖已有的文件。
#-K<文件>或--starting-file=<文件> 从指定的文件开始还原。
#-l或--one-file-system 复制的文件或目录存放的文件系统，必须与tar指令执行时所处的文件系统相同，否则不予复制。
#-L<媒体容量>或-tape-length=<媒体容量> 设置存放每体的容量，单位以1024 Bytes计算。
#-m或--modification-time 还原文件时，不变更文件的更改时间。
#-M或--multi-volume 在建立，还原备份文件或列出其中的内容时，采用多卷册模式。
#-N<日期格式>或--newer=<日期时间> 只将较指定日期更新的文件保存到备份文件里。
#-o或--old-archive或--portability 将资料写入备份文件时使用V7格式。
#-O或--stdout 把从备份文件里还原的文件输出到标准输出设备。
#-p或--same-permissions 用原来的文件权限还原文件。
#-P或--absolute-names 文件名使用绝对名称，不移除文件名称前的"/"号。
#-r或--append 新增文件到已存在的备份文件的结尾部分。
#-R或--block-number 列出每个信息在备份文件中的区块编号。
#-s或--same-order 还原文件的顺序和备份文件内的存放顺序相同。
#-S或--sparse 倘若一个文件内含大量的连续0字节，则将此文件存成稀疏文件。
#-t或--list 列出备份文件的内容。
#-T<范本文件>或--files-from=<范本文件> 指定范本文件，其内含有一个或多个范本样式，让tar解开或建立符合设置条件的文件。
#-u或--update 仅置换较备份文件内的文件更新的文件。
#-U或--unlink-first 解开压缩文件还原文件之前，先解除文件的连接。
#-v或--verbose 显示指令执行过程。
#-V<卷册名称>或--label=<卷册名称> 建立使用指定的卷册名称的备份文件。
#-w或--interactive 遭遇问题时先询问用户。
#-W或--verify 写入备份文件后，确认文件正确无误。
#-x或--extract或--get 从备份文件中还原文件。
#-X<范本文件>或--exclude-from=<范本文件> 指定范本文件，其内含有一个或多个范本样式，让ar排除符合设置条件的文件。
#-z或--gzip或--ungzip 通过gzip指令处理备份文件。
#-Z或--compress或--uncompress 通过compress指令处理备份文件。
#-<设备编号><存储密度> 设置备份用的外围设备编号及存放数据的密度。
#--after-date=<日期时间> 此参数的效果和指定"-N"参数相同。
#--atime-preserve 不变更文件的存取时间。
#--backup=<备份方式>或--backup 移除文件前先进行备份。
#--checkpoint 读取备份文件时列出目录名称。
#--concatenate 此参数的效果和指定"-A"参数相同。
#--confirmation 此参数的效果和指定"-w"参数相同。
#--delete 从备份文件中删除指定的文件。
#--exclude=<范本样式> 排除符合范本样式的文件。
#--group=<群组名称> 把加入设备文件中的文件的所属群组设成指定的群组。
#--help 在线帮助。
#--ignore-failed-read 忽略数据读取错误，不中断程序的执行。
#--new-volume-script=<Script文件> 此参数的效果和指定"-F"参数相同。
#--newer-mtime 只保存更改过的文件。
#--no-recursion 不做递归处理，也就是指定目录下的所有文件及子目录不予处理。
#--null 从null设备读取文件名称。
#--numeric-owner 以用户识别码及群组识别码取代用户名称和群组名称。
#--owner=<用户名称> 把加入备份文件中的文件的拥有者设成指定的用户。
#--posix 将数据写入备份文件时使用POSIX格式。
#--preserve 此参数的效果和指定"-ps"参数相同。
#--preserve-order 此参数的效果和指定"-A"参数相同。
#--preserve-permissions 此参数的效果和指定"-p"参数相同。
#--record-size=<区块数目> 此参数的效果和指定"-b"参数相同。
#--recursive-unlink 解开压缩文件还原目录之前，先解除整个目录下所有文件的连接。
#--remove-files 文件加入备份文件后，就将其删除。
#--rsh-command=<执行指令> 设置要在远端主机上执行的指令，以取代rsh指令。
#--same-owner 尝试以相同的文件拥有者还原文件。
#--suffix=<备份字尾字符串> 移除文件前先行备份。
#--totals 备份文件建立后，列出文件大小。
#--use-compress-program=<执行指令> 通过指定的指令处理备份文件。
#--version 显示版本信息。
#--volno-file=<编号文件> 使用指定文件内的编号取代预设的卷册编号。

#e.g.
#压缩文件
touch a.c       
tar -czvf test.tar.gz a.c   //压缩 a.c文件为test.tar.gz
#列出压缩文件内容
tar -tzvf test.tar.gz 
#解压文件
tar -xzvf test.tar.gz 



zip [-AcdDfFghjJKlLmoqrSTuvVwXyz$][-b <工作目录>][-ll][-n <字尾字符串>][-t <日期时间>][-<压缩效率>][压缩文件][文件...][-i <范本样式>][-x <范本样式>]
#-A 调整可执行的自动解压缩文件。
#-b<工作目录> 指定暂时存放文件的目录。
#-c 替每个被压缩的文件加上注释。
#-d 从压缩文件内删除指定的文件。
#-D 压缩文件内不建立目录名称。
#-f 更新现有的文件。
#-F 尝试修复已损坏的压缩文件。
#-g 将文件压缩后附加在既有的压缩文件之后，而非另行建立新的压缩文件。
#-h 在线帮助。
#-i<范本样式> 只压缩符合条件的文件。
#-j 只保存文件名称及其内容，而不存放任何目录名称。
#-J 删除压缩文件前面不必要的数据。
#-k 使用MS-DOS兼容格式的文件名称。
#-l 压缩文件时，把LF字符置换成LF+CR字符。
#-ll 压缩文件时，把LF+CR字符置换成LF字符。
#-L 显示版权信息。
#-m 将文件压缩并加入压缩文件后，删除原始文件，即把文件移到压缩文件中。
#-n<字尾字符串> 不压缩具有特定字尾字符串的文件。
#-o 以压缩文件内拥有最新更改时间的文件为准，将压缩文件的更改时间设成和该文件相同。
#-q 不显示指令执行过程。
#-r 递归处理，将指定目录下的所有文件和子目录一并处理。
#-S 包含系统和隐藏文件。
#-t<日期时间> 把压缩文件的日期设成指定的日期。
#-T 检查备份文件内的每个文件是否正确无误。
#-u 与 -f 参数类似，但是除了更新现有的文件外，也会将压缩文件中的其他文件解压缩到目录中。
#-v 显示指令执行过程或显示版本信息。
#-V 保存VMS操作系统的文件属性。
#-w 在文件名称里假如版本编号，本参数仅在VMS操作系统下有效。
#-x<范本样式> 压缩时排除符合条件的文件。
#-X 不保存额外的文件属性。
#-y 直接保存符号连接，而非该连接所指向的文件，本参数仅在UNIX之类的系统下有效。
#-z 替压缩文件加上注释。
#-$ 保存第一个被压缩文件所在磁盘的卷册名称。
#-<压缩效率> 压缩效率是一个介于1-9的数值。

#e.g.
#将 /home/html/ 这个目录下所有文件和文件夹打包为当前目录下的 html.zip：
zip -q -r html.zip /home/html
#如果在我们在 /home/html 目录下，可以执行以下命令：
zip -q -r html.zip *
#从压缩文件 cp.zip 中删除文件 a.c
zip -dv cp.zip a.c




unzip [-cflptuvz][-agCjLMnoqsVX][-P <密码>][.zip文件][文件][-d <目录>][-x <文件>] 或 unzip [-Z]
#-c 将解压缩的结果显示到屏幕上，并对字符做适当的转换。
#-f 更新现有的文件。
#-l 显示压缩文件内所包含的文件。
#-p 与-c参数类似，会将解压缩的结果显示到屏幕上，但不会执行任何的转换。
#-t 检查压缩文件是否正确。
#-u 与-f参数类似，但是除了更新现有的文件外，也会将压缩文件中的其他文件解压缩到目录中。
#-v 执行时显示详细的信息。
#-z 仅显示压缩文件的备注文字。
#-a 对文本文件进行必要的字符转换。
#-b 不要对文本文件进行字符转换。
#-C 压缩文件中的文件名称区分大小写。
#-j 不处理压缩文件中原有的目录路径。
#-L 将压缩文件中的全部文件名改为小写。
#-M 将输出结果送到more程序处理。
#-n 解压缩时不要覆盖原有的文件。
#-o 不必先询问用户，unzip执行后覆盖原有文件。
#-P<密码> 使用zip的密码选项。
#-q 执行时不显示任何信息。
#-s 将文件名中的空白字符转换为底线字符。
#-V 保留VMS的文件版本信息。
#-X 解压缩时同时回存文件原来的UID/GID。
#[.zip文件] 指定.zip压缩文件。
#[文件] 指定要处理.zip压缩文件中的哪些文件。
#-d<目录> 指定文件解压缩后所要存储的目录。
#-x<文件> 指定不要处理.zip压缩文件中的哪些文件。
#-Z unzip -Z等于执行zipinfo指令。

#e.g.
#查看压缩文件中包含的文件
unzip -l abc.zip
#-v 参数用于查看压缩文件目录信息，但是不解压该文件
unzip -v abc.zip 
#解压文件到当前目录
unzip ./abc.zip
```



## 复制/移动&重命名/删除

```shell
cp [options] source dest
cp [options] source... directory
#-a：此选项通常在复制目录时使用，它保留链接、文件属性，并复制目录下的所有内容。其作用等于dpR参数组合。
#-d：复制时保留链接。这里所说的链接相当于 Windows 系统中的快捷方式。
#-f：覆盖已经存在的目标文件而不给出提示。
#-i：与 -f 选项相反，在覆盖目标文件之前给出提示，要求用户确认是否覆盖，回答 y 时目标文件将被覆盖。
#-p：除复制文件的内容外，还把修改时间和访问权限也复制到新文件中。
#-r：若给出的源文件是一个目录文件，此时将复制该目录下所有的子目录和文件。
#-l：不复制文件，只是生成链接文件。

mv [options] source dest
mv [options] source... directory
#-b: 当目标文件或目录存在时，在执行覆盖前，会为其创建一个备份。
#-i: 如果指定移动的源目录或文件与目标的目录或文件同名，则会先询问是否覆盖旧文件，输入 y 表示直接覆盖，输入 n 表示取消该操作。
#-f: 如果指定移动的源目录或文件与目标的目录或文件同名，不会询问，直接覆盖旧文件。
#-n: 不要覆盖任何已存在的文件或目录。
#-u：当源文件比目标文件新或者目标文件不存在时，才执行移动操作。

rm [options] name...
#-i 删除前逐一询问确认。
#-f 即使原档案属性设为唯读，亦直接删除，无需逐一确认。
#-r 将目录及以下之档案亦逐一删除。

```

## 后台运行/显示进程/kill进程

```shell
nohup Command [ Arg … ] [　& ]
#Command：要执行的命令。
#Arg：一些参数，可以指定输出文件。
#&：让命令在后台执行，终端退出后命令仍旧执行。


#实时显示 process 的动态
top [-] [d delay] [q] [c] [S] [s] [i] [n] [b]
#d : 改变显示的更新速度，或是在交谈式指令列( interactive command)按 s
#q : 没有任何延迟的显示速度，如果使用者是有 superuser 的权限，则 top 将会以最高的优先序执行
#c : 切换显示模式，共有两种模式，一是只显示执行档的名称，另一种是显示完整的路径与名称
#S : 累积模式，会将己完成或消失的子进程 ( dead child process ) 的 CPU time 累积起来
#s : 安全模式，将交谈式指令取消, 避免潜在的危机
#i : 不显示任何闲置 (idle) 或无用 (zombie) 的进程
#n : 更新的次数，完成后将会退出 top
#b : 批次档模式，搭配 "n" 参数一起使用，可以用来将 top 的结果输出到档案内


#将所有进程以树状图显示，树状图将会以 pid (如果有指定) 或是以 init 这个基本进程为根 (root)，如果有指定使用者 id，则树状图会只显示该使用者所拥有的进程。
pstree [-a] [-c] [-h|-Hpid] [-l] [-n] [-p] [-u] [-G|-U] [pid|user]
#-a 显示该进程的完整指令及参数, 如果是被记忆体置换出去的进程则会加上括号
#-c 如果有重覆的进程名, 则分开列出（预设值是会在前面加上 *）


#删除执行中的程序或工作。kill 可将指定的信息送至程序。预设的信息为 SIGTERM(15)，可将指定程序终止。若仍无法终止该程序，可使用 SIGKILL(9) 信息尝试强制删除程序。程序或工作的编号可利用 ps 指令或 jobs 指令查看。
kill [-s <信息名称或编号>][程序]　或　kill [-l <信息编号>]
#-l <信息编号> 　若不加<信息编号>选项，则 -l 参数会列出全部的信息名称。
#-s <信息名称或编号> 　指定要送出的信息。
#[程序] 　[程序]可以是程序的PID或是PGID，也可以是工作编号。
```

