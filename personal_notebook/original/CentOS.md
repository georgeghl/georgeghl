# AlmaLinux 9 上配置静态 IP 地址

https://blog.csdn.net/li_c_yang/article/details/142056357

在 AlmaLinux 9 中，密钥文件的新默认存储位置在 /etc/NetworkManager/system-connections 中

```shell
cd /etc/NetworkManager/system-connections
```

##### 默认dhcp配置

```shell
[connection]
id=ens18
uuid=77c1e2f2-fe9d-3b54-a1cc-ea306b4261e4
type=ethernet
autoconnect-priority=-999
interface-name=ens33
timestamp=1725853338

[ethernet]

[ipv4]
method=auto

[ipv6]
addr-gen-mode=eui64
method=auto

[proxy]

```

##### 修改配置

```shell
[connection]
id=ens18
uuid=77c1e2f2-fe9d-3b54-a1cc-ea306b4261e4
type=ethernet
autoconnect-priority=-999
interface-name=ens33
timestamp=1725853338

[ethernet]

[ipv4]
#手动设置
method=manual
#IP/网关地址
address1=192.168.50.4/24,192.168.50.1
# DNS服务器
dns=114.114.114.114


[ipv6]
addr-gen-mode=eui64
method=auto

[proxy]

```

修改

```dart
[ipv4]
method=manual
# IP/子网掩码,网关
address1=192.168.112.22/24,192.168.112.1
# DNS服务器
dns=172.16.8.220
```

##### 重新加载配置文件，注意我的网卡id是id=ens33，所以重载ens33

```dart
nmcli connection reload ens18.nmconnection
nmcli connection up ens18
```

























































