

# Mi Router 3G 刷固件

先刷breed。

```bash
mtd -r write /tmp/breed-mt7621-xiaomi-r3g-r3p.bin Bootloader
```

然后断电后按住reset，接电源，当蓝色灯闪烁时即进入breed。
进入breed：192.168.1.1。
删除normal_firmware_md5。
添加环境变量xiaomi.r3g.bootfw, 值为2
更新固件，选择小米r3g第二个。先刷initramfs。然后进入openwrt后，更新固件，再刷squahfs。

**root密码password**



# 配置opkg源

可以直接在luci：系统->软件包进行配置。

常用的那个固件，对应源配置为：

```bash
#版本号改19.07.6成你自己的,在luci状态界面查看版本或者命令: cat /etc/openwrt_*

src/gz openwrt_core https://mirrors.tuna.tsinghua.edu.cn/openwrt/releases/18.06.8/targets/ramips/mt7621/packages
src/gz openwrt_base https://mirrors.tuna.tsinghua.edu.cn/openwrt/releases/18.06.8/packages/mipsel_24kc/base
src/gz openwrt_luci https://mirrors.tuna.tsinghua.edu.cn/openwrt/releases/18.06.8/packages/mipsel_24kc/luci
src/gz openwrt_packages https://mirrors.tuna.tsinghua.edu.cn/openwrt/releases/18.06.8/packages/mipsel_24kc/packages
src/gz openwrt_routing https://mirrors.tuna.tsinghua.edu.cn/openwrt/releases/18.06.8/packages/mipsel_24kc/routing
src/gz openwrt_telephony https://mirrors.tuna.tsinghua.edu.cn/openwrt/releases/18.06.8/packages/mipsel_24kc/telephony

```





# openvpn-client

```bash
opkg install openvpn-openssl luci-app-openvpn openssl-util
```







# 修改主题

```bash
#
opkg install luci-theme-infinityfreedom_1.5.20201213-1_all.ipk
#猫羽雫
opkg install luci-theme-ifit_2021-5-29.ipk
#
opkg install luci-theme-neobird_1.99-202201272020_all.ipk
```







# 修改wan和lan的mac

mac地址查询：[oui](oui.txt)

```bash
cd /etc/config
cp network network.BAK
vi network
#修改所有lan的mac为(Apple Inc)：
#60:8B:0E:DB:C5:03
#修改所有wan的mac为(Apple Inc)：
#60:8B:0E:DB:C5:02
#ESC
#:wq
cat network
sync
#重启

#注：各公司MAC前缀
#Apple Inc
60:8B:0E
90:2C:09
A0:B4:0F
EC:0D:51
AC:DF:A1
D8:E5:93
CC:68:E0
#Fujitsu Limited/FUJITSU LIMITED
34:FE:9E
68:84:7E
A8:B2:DA
B0:AC:FA
B0:99:28
EC:79:49
00:E0:00
38:AF:D7
00:23:26
74:2B:62
#Fujitsu General Limited
00:02:DC
#FUJITSU DENSO LTD.
00:A0:CA
#LEXMARK INTERNATIONAL, INC.
00:21:B7
#CHONGQING FUGUI ELECTRONICS CO.,LTD.
40:23:43
#Xiaomi Communications Co Ltd
68:C4:4C
E4:AA:E4
A8:6A:86
FC:A9:F5
#New H3C Technologies Co., Ltd
6C:87:20
#Google, Inc.
FC:91:5D
#Dell Inc.
10:98:19
#LG Electronics
74:C1:7E
#Cisco Systems, Inc
10:E3:76
C4:AB:4D
48:74:10
#Palo Alto Networks
7C:C0:25
#Huawei Device Co., Ltd.
70:66:B9
C4:A1:AE
38:FC:34
3C:F6:92
C0:BF:AC
44:27:2E
E4:54:E5
D8:54:F2
8C:C9:E9
D8:9E:61
34:7E:00
#HUAWEI TECHNOLOGIES CO.,LTD
00:E0:FC #Use this one
34:83:D5
7C:0C:FA
54:44:3B
5C:70:75
#LCFC(HeFei) Electronics Technology co., ltd
FC:5C:EE
74:5D:22
E8:80:88
#Hangzhou Hikvision Digital Technology Co.,Ltd.
0C:75:D2
54:8C:81
24:48:45
24:32:AE
#Zhejiang Dahua Technology Co., Ltd.
74:C9:29
6C:1C:71
08:ED:ED
#Samsung Electronics Co.,Ltd
C0:48:E6
CC:6E:A4
A8:16:D0
A4:6C:F1
08:AE:D6
#Sony Corporation
D4:38:9C
00:19:63
00:1F:E4
00:23:45
6C:0E:0D
#Sony Interactive Entertainment Inc.
00:1F:A7
#Hewlett Packard
9C:7B:EF
10:E7:C6
#Intel Corporate
7C:CC:B8
F4:06:69
00:1D:E1
00:1F:3B
E0:94:67
#Advanced Micro Devices, Inc./ADVANCED MICRO DEVICES
74:27:2C
00:00:1A
#Xilinx, Inc/Xilinx
00:5D:03
00:0A:35
#Luxshare Precision Industry CO., LTD./Luxshare Precision Industry Co.,Ltd/Luxshare Precision Industry Company Limited
E8:95:26
60:7D:09
10:82:86
60:6D:3C
3C:18:A0
#Nintendo Co.,Ltd
60:1A:C7
BC:9E:BB
CC:5B:31
1C:45:86
E8:A0:CD
#Hitachi, Ltd.
68:69:CA
#WINBOND ELECTRONICS CORP.
00:10:1D
#SK hynix
AC:E4:2E
#Sumitomo Electric Industries, Ltd
00:08:F6
00:0B:A2
00:00:5F
08:4E:BF
#Panasonic Corporation
00:12:67
#NIKON CORPORATION
3C:BE:E1
00:90:B5
#CANON INC.
00:00:85
2C:9E:FC
18:0C:AC
74:BF:C0
#Brother industries, LTD.
00:80:77
B4:22:00
94:DD:F8
30:05:5C
#FUJIFILM Business Innovation Corp.
08:00:37
1C:7D:22
#NEC Corporation
D4:92:34
```





# CUPS

下载ipk文件：

[cups_mipsel.zip]: (./cups_mipsel.zip)	"CUPS"

[ipk包,点此下载](./cups_mipsel.zip)

**一、下载安装CUPS**

- 看了别人的资料后，原本以为可以直接在opkg中安装使用。但我的源里却没有这个包。无奈之下，一番搜索，发现有大佬提供[ipk包。点此下载](https://gitee.com/ytwl333/openwrt-musl-cups/tree/master)，有能力的大佬也可以自己编译，[源码地址](https://github.com/wgedu/cups-for-openwrt)。

- 根据自己平台下载好相应的包（有十多个都是）后，上传到LEDE的tmp目录下，（其中cups-filters这个包好像和cups不太兼容，我选择删去cups-filters这个包，不安装它），在ssh下输入命令opkg install /tmp/*.ipk --force-depends来安装这些包。
  注：如果安装了kmod-usb-printer，该模块可能与cups存在兼容问题，请使用下面命令卸载opkg remove kmod-usb-printer

- 安装完成后，打开/etc/cups/cupsd.conf文件。对应更改下方配置。

- ```conf
  这个配置打开网页配置，和配置中文
  WebInterface Yes +DefaultLanguage zh
  
  下方是设置允许访问的IP地址范围，根据自己情况配置，如你LAN_IP为192.168.1.1，则改成如下方所示
  <Location />
    Order Allow,Deny
    Allow From 127.0.0.1
    Allow From 192.168.1.0/24 
  </Location>
  
  下方是设置允许访问使用admin账户的IP地址范围，如下设置即可
  <Location /admin>
    AuthType Basic
    AuthClass Anonymous
    Order Allow,Deny
    Allow From 127.0.0.1
    Allow From 192.168.1.0/24
  </Location>
  ```

然后，输入ssh命令/etc/init.d/cupsd restart重启CUPS服务。

**二、在CUPS上添加打印机**

- 使用http://lan_ip:631/admin登陆CUPS管理页面。
  如果需要登录，则用户名：root 密码：你配置的路由器的登陆密码。
- 添加打印机
- 一般来说，接上打印机后应当能在此看到，选中后继续。
- 写一个简单的名字，选择上共享这台打印机
- 这里默认没有驱动，选择Raw，然后点击Add Printer添加打印机即可。
- 添加成功后，应该会跳转到打印机信息界面。这里，我们复制这个网页链接。应如：http://lan_ip:631/printers/你前面设定的打印机名

**三、WIN系统上添加该打印机**

- 点击微软图标，点击设置后再进入设备
- 进入打印机和扫描仪，点击添加打印机或和扫描仪，等一段时间后会出现下图选项，点它
- 然后点按名称选择选择打印机，输入你刚复制的地址 ，点击下一步
- 如果连接成功会让你选择打印机驱动。
- 然后下一步，完成即可
- 此时，打印机应当能正常工作了。



# 安装ipk包

opkg install *.ipk







# x86安装(pve)

新建虚拟机，不需要添加磁盘、光盘。

上传openwrt的img镜像到指定存储区，如data（/mnt/data/）

新建完毕后，使用pve的终端运行：

```shell
qm importdisk 100 /mnt/data/template/iso/openwrt-xx-xx-xx.img data
```

然后在虚拟机的设置-硬件中启用该新增的磁盘，在选项-引导顺序中启用并将该磁盘放在首位。

开机。等一会。

按Enter，出现shell。

## 添加/指定/配置wan口，配置br-lan的ip

```shell
cd /etc/config/
cp network network.BAK
vi network

```

修改/添加以下内容：

```yml
# 配置lan

# (1)以下是192.168.x.x类型的私有局域网配置：
config interface 'lan'
	option type 'bridge'
	option ifname 'eth0'
	option proto 'static'
	option ipaddr '192.168.123.1'
	option netmask '255.255.255.0'
	option delegate '0'





# 配置wan
config interface 'wan'
	option ifname 'eth1'
	option proto 'dhcp'
	option metric '64'
	option delegate '0'
	
config interface 'wan6'
	option proto 'dhcp6'
	option ifname 'eth1'
	option delegate '0'
	option reqaddress 'none'
	option reqprefix 'no'
	
```

## 允许wan口访问/管理

```shell
cd /etc/config/
cp firewall firewall.BAK
vi firewall
```

然后修改(添加)：

```yml
config rule
    option src 'wan'
    option dest_port '80'
    option target 'ACCEPT'
    option proto 'tcp'
    option name 'Allow-Remote-HTTP'
```

最后：

```shell
/etc/init.d/firewall restart
```

# [在PVE下安装OpenWrt并配置旁路由](https://neverup.cn/p/pve-openwrt-1/)

**下载OpenWrt固件**

因为我用的是X86小主机，所以本教程以X86为例，其他的架构也大同小异。

首先，打开[OpenWrt官网](https://downloads.openwrt.org/)（有动手能力的小伙伴也可以尝试自己编译固件）。我这里选择了OpenWrt 24.10.0。点击`targets`。往下滑，找到`X86`。继续点击`64`。下载最上面的`generic-ext4-combined-efi.img.gz`。

**在PVE中安装OpenWrt**

登录PVE后台，然后点击右上角的`创建虚拟机`。随便起一个名称。`VM ID`这里需要记一下，等会要用。然后点击下一步。这里选择`不使用任何介质`。然后点击下一步,这里的设置保持默认即可。点击下一步，这里直接`删除磁盘`。点击下一步，这里的类别改成`host`（这样运行效率更高），插槽和核心按需修改。点击下一步，这里的内存按需分配，一般512MB-1024MB就够了。（这里可以点`高级`，然后把`Ballooning设备`（动态分配内存）取消勾选）。点击下一步，这里保持默认。最后检查一下所有信息是否有误，点击完成。不过这时候还不能直接运行（硬盘还没配置呢）

**img格式转化为gcow2**

为了适配PVE，这里需要把img格式转化为qcow2格式。首先使用SSH工具登录PVE，这里我使用的SSH工具是MobaXterm。为方便后续操作，把下载的文件重命名为`openwrt-24.10.0.img`。把`openwrt-24.10.0.img`上传：

```
qm importdisk 100 /mnt/data/template/iso/openwrt-xx-xx-xx.img data
```

提示successfully 说明转化成功。

**OpenWrt的初次开机安装**

回到PVE中，找到`硬件`这里的`未使用的磁盘`，双击一下。点击`添加`。然后点击左侧的`选项`，双击`引导顺序`。勾选`scsi0(刚才添加到硬盘)`，然后`移动到第一项`，最后点击OK。在控制台中点击`Start Now`开机。修改OpenWrt的IP。在控制台中，输入`vi /etc/config/network`。然后需要把192.168.1.1改成你需要的ip（按i即可编辑）改完之后，按Esc（图片所示），输入`:wq`，回车。然后输入`reboot`，重启系统。

**OpenWrt的初步配置**

在浏览器输入IP（你刚才修改的）访问OpenWrt。点`Log in`登录（密码为空）确定可以访问后，回到PVE，找到`选项`，把`开机自启动`勾选上。OpenWrt已经安装好了，下面开始OpenWrt的初步配置。修改密码、配置网络。

首先，点击Network，然后点击子菜单Interfaces。点击Edit，填写主路由的IP，设置DNS，因为是旁路由，先把DHCP关了。最后一定要点击`Save Apply`（保存并应用）。

更换opkg为国内源，点击导航栏上的`System`，然后点击子菜单`Software`。点击`Configure opkg`。然后用下面的内容覆盖原有的/etc/opkg/distfeeds.conf，然后点击OK。

```
src/gz openwrt_core https://mirrors.tuna.tsinghua.edu.cn/openwrt/releases/24.10.0/targets/x86/64/packages
src/gz openwrt_base https://mirrors.tuna.tsinghua.edu.cn/openwrt/releases/24.10.0/packages/x86_64/base
src/gz openwrt_luci https://mirrors.tuna.tsinghua.edu.cn/openwrt/releases/24.10.0/packages/x86_64/luci
src/gz openwrt_packages https://mirrors.tuna.tsinghua.edu.cn/openwrt/releases/24.10.0/packages/x86_64/packages
src/gz openwrt_routing https://mirrors.tuna.tsinghua.edu.cn/openwrt/releases/24.10.0/packages/x86_64/routing
src/gz openwrt_telephony https://mirrors.tuna.tsinghua.edu.cn/openwrt/releases/24.10.0/packages/x86_64/telephony
```

然后点击`Update lists…`。然后自己刷新一下页面，如果下面出现了软件包，说明配置好了。

安装argo主题：在Software中搜索框依次搜索并安装`luci-compat`、`luci-lib-ipkg`、`luci-i18n-base-zh-cn`，

然后下载文件https://github.com/jerrykuku/luci-theme-argon/releases/download/v2.3.2/luci-theme-argon_2.3.2-r20250207_all.ipk，再上传文件（Upload Package），最后刷新一下就OK了。

**旁路由配置**

来到`防火墙`，取消勾选`启用SYN-flood防御`，然后把`入站数据，出站数据，转发`都改成`接受`。往下滑，找到`区域`。把`wan`口`删除`，`lan`口勾选`IP动态伪装`。最后点击`保存并应用`。







































































































