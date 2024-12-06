# 重庆大学虎溪校区openwrt自动校园网登录认证

1.修改以下代码中的用户名和密码。

```shell
# Copyright (C) 2023 cn.edu.cqu.cs.www
# A simple script for CQU-Huxi Campus Internet access auto login.
# Modify your IP, MAC Address, user_account, password in the wget link.


echo "CQU - Huxi Campus Internet access auto Login"
user_account="修改这里、填写用户名"
user_password="修改这里、填写密码"
while true
do
        wanip=$(ifconfig "eth0.2" | grep "inet addr:" | awk '{print $2}' | cut -c 6-)
        wanmac=$(ifconfig "eth0.2" | grep "HWaddr" | awk '{print $5}')
        wanmacmod=$(ifconfig -a | grep "eth0.2" | awk -F ' ' '{print $5}' | awk '{print tolower($0)}' | sed 's/://g')
        echo "WAN IP: "$wanip
        echo "WAN Mac: "$wanmac
        echo "WAN Mac without colon: "$wanmacmod
        wgeturl="http://10.254.7.4:801/eportal/portal/login?callback=dr1004&login_method=1&user_account=,0,"$user_account"&user_password="$user_password"&wlan_user_ip="$wanip"&wlan_user_ipv6=&wlan_user_mac="$wanmacmod"&wlan_ac_ip=&wlan_ac_name=&jsVersion=4.2&terminal_type=1&lang=zh-cn&v=1891&lang=zh"
        echo "Access url: "$wgeturl

        if ping -c 1 -w 50 www.baidu.com >/dev/null;
        then     
                echo "Ping OK, no need to login."
                
        else                   
                echo "Ping FAIL, start to send login request......"
                
                wget $wgeturl

                echo "Login Success!"
                sleep 5
        fi
sleep 360                                                                  
done

```

2.复制以上代码，打开Openwrt设置中`系统-启动项`的`本地启动脚本`（在网页最下方），添加该段脚本到'exit 0' 前，添加完成后，该项应该如下所示：

```shell
# Put your custom commands here that should be executed once
# the system init finished. By default this file does nothing.


# Copyright (C) 2023 cn.edu.cqu.cs.www
# A simple script for CQU-Huxi Campus Internet access auto login.
# Modify your IP, MAC Address, user_account, password in the wget link.


echo "CQU - Huxi Campus Internet access auto Login"
user_account="修改这里、填写用户名"
user_password="修改这里、填写密码"
while true
do
        wanip=$(ifconfig "eth0.2" | grep "inet addr:" | awk '{print $2}' | cut -c 6-)
        wanmac=$(ifconfig "eth0.2" | grep "HWaddr" | awk '{print $5}')
        wanmacmod=$(ifconfig -a | grep "eth0.2" | awk -F ' ' '{print $5}' | awk '{print tolower($0)}' | sed 's/://g')
        echo "WAN IP: "$wanip
        echo "WAN Mac: "$wanmac
        echo "WAN Mac without colon: "$wanmacmod
        wgeturl="http://10.254.7.4:801/eportal/portal/login?callback=dr1004&login_method=1&user_account=,0,"$user_account"&user_password="$user_password"&wlan_user_ip="$wanip"&wlan_user_ipv6=&wlan_user_mac="$wanmacmod"&wlan_ac_ip=&wlan_ac_name=&jsVersion=4.2&terminal_type=1&lang=zh-cn&v=1891&lang=zh"
        echo "Access url: "$wgeturl

        if ping -c 1 -w 50 www.baidu.com >/dev/null;
        then     
                echo "Ping OK, no need to login."
                
        else                   
                echo "Ping FAIL, start to send login request......"
                
                wget $wgeturl

                echo "Login Success!"
                sleep 5
        fi
sleep 360                                                                  
done

exit 0

```



3.重新启动路由器即可。