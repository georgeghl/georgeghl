# 遥遥领先®鸿蒙™2.0(手机)瘦身

1.打开手机开发者模式（**android**）

2.usb可调式

3.下载ADB

4.ADB 常用命令：

| 命令                                      | 含义                                                         |
| ----------------------------------------- | ------------------------------------------------------------ |
| `adb kill-server`                         | 关闭服务                                                     |
| `adb start-server`                        | 开启服务                                                     |
| `adb devices`                             | 查看当前连接的设备, 连接到计算机的android设备或者模拟器将会列出显示 |
| `adb install apk路径`                     | 安装                                                         |
| `adb shell pm uninstall -k --user 0 包名` | 卸载                                                         |
| `adb tcpip 5555`                          | ADB 无线配置+ Vysor(Win App or Chrom plugin)                 |
| `adb connect ip`                          | 远程操控android                                              |
| `adb shell pm disable-user 包名`          | 禁用APP                                                      |
| `adb shell pm enable 包名`                | 解禁APP                                                      |

5.内置应用常用包名：

```
com.android.cts.priv.ctsshim
com.huawei.camera           相机
com.huawei.android.tips           玩机技巧
com.google.android.ext.services
com.huawei.synergy
com.huawei.android.launcher         华为桌面
com.android.gallery3d.overlay
com.android.mediacenter              音乐
com.android.providers.telephony         通话/信息存储
com.unionpay.tsmservice             银联可信服务安全组件
com.huawei.androidx
com.huawei.android.UEInfoCheck
com.android.phone.recorder         通话录音
com.android.providers.calendar       日历存储
com.huawei.featurelayer.featureframework
com.huawei.health         运动健康
com.huawei.hicard
com.huawei.hidisk         文件管理
com.huawei.hiview
com.huawei.iaware
com.huawei.bluetooth       通过蓝牙导入
com.android.providers.media         媒体存储
com.huawei.android.thememanager         主题
com.huawei.android.chr
com.huawei.android.hsf         华为服务框架
com.google.android.onetimeinitializer
com.google.android.ext.shared
com.android.wallpapercropper
com.huawei.imedia.dolby          杜比全景声
com.huawei.nearby
com.huawei.android.FloatTasks            悬浮导航
com.huawei.desktop.systemui        系统界面
com.huawei.intelligent         智慧助手
com.huawei.motionservice          手势服务
com.huawei.appmarket        华为应用市场
com.huawei.recsys
com.xy.smartmmsplugin.remote        智能短信插件
com.huawei.parentcontrol           健康使用手机
com.huawei.search          指挥搜索
com.huawei.secime
com.android.documentsui           文件
com.huawei.numberidentity             号码识别
com.android.externalstorage         外部存储设备
com.android.htmlviewer        HTML查看器
com.android.companiondevicemanager           跟随设备管理器
com.huawei.regservice
com.iflytek.speechsuite         讯飞语音引擎
com.android.mms.service
com.huawei.wallet           华为钱包
com.huawei.android.totemweather          天气
com.android.providers.downloads       下载管理器
com.huawei.gameassistant          应用助手
com.huawei.android.findmyphone      查找我的手机
com.huawei.trustcircle        设备认证服务
com.huawei.fastapp             快应用中心
com.huawei.phoneservice           服务
com.huawei.desktop.explorer          我的文件
com.huawei.screenrecorder          屏幕录制
com.huawei.arengine.service          AREngineServer
com.huawei.hwpolicyservice
com.huawei.spaceservice     地理围栏服务
com.huawei.videoeditor         视频编辑
com.huawei.securitymgr         隐私空间
com.google.android.configupdater
com.android.soundrecorder         录音机
com.huawei.iconnect            设备连接服务
com.huawei.android.AutoRegSms          AutoRegSms
com.google.android.overlay.settingsProvider
com.huawei.KoBackup          备份
com.android.defcontainer          软件包访问帮助程序
com.huawei.hiviewtunnel
com.google.ar.core       谷歌AR
com.android.providers.downloads.ui        下载内容
com.android.vending      谷歌play服务更新程序
com.android.pacprocessor
com.android.simappdialog
com.android.systemui.overlay
com.huawei.smarthome         智慧生活
androidhwext                   androidhwext
com.huawei.pengine           华为智能建议
com.baidu.input_huawei
com.android.frameworkhwext.honor
com.android.certinstaller      证书安装器
com.android.carrierconfig
com.google.android.marvin.talkback
android              Android系统
com.huawei.fans         花粉俱乐部
com.huawei.hiai           华为智慧引擎
com.huawei.hwid            华为移动服务
com.huawei.msdp           综合传感信息处理平台
com.huawei.tips          智能提醒
com.android.contacts       联系人
com.huawei.contentsensor         取词
com.huawei.systemserver           系统服务
com.huawei.mycenter           会员中心
com.android.frameworkhwext.dark
com.android.mms          信息
com.android.mtp
com.android.nfc          NFC服务
com.android.stk        SIM卡应用
com.android.backupconfirm
com.huawei.android.instantshare
com.huawei.trustagent         智能解锁
com.huawei.trustspace         支付保护中心
com.android.statementservice
com.huawei.printservice           默认打印服务
com.huawei.android.internal.app
com.huawei.gamebox             华为游戏中心
com.huawei.cloud              华为云电脑
com.huawei.hwasm
com.huawei.lives              华为生活服务
com.android.calendar       日历
com.huawei.wifiprobqeservice
com.huawei.livewallpaper.matetwenty        华山动态壁纸
com.huawei.smartlocation          室内定位服务
com.android.providers.settings       设置存储
com.android.sharedstoragebackup
com.android.printspooler          打印处理服务
com.android.frameworkres.overlay
com.huawei.featurelayer.sharedfeature.map          华为地图服务
com.android.dreams.basic           基本互动屏保
com.android.incallui          电话
com.huawei.systemmanager     手机管家
com.huawei.nb.service       数据管理服务
com.android.se
com.android.inputdevices         输入设备
com.huawei.browser            浏览器
com.hicloud.android.clone       手机克隆
com.google.android.overlay.gmsconfig
com.huawei.contacts.sync         联系人同步
com.huawei.hilink.framework
com.google.android.webview
com.huawei.HwMultiScreenShot         滚动截屏
com.huawei.vassistant             语音助手
com.android.server.telecom             通话管理
com.google.android.syncadapters.contacts       谷歌通讯录同步
com.example.android.notepad         备忘录
com.android.keychain         密钥链
com.android.keyguard        华为杂志锁屏
com.microsoft.translator          微软翻译
com.huawei.languagedownloader         下载语言
com.android.gallery3d            图库
com.google.android.gms       谷歌play服务
com.google.android.gsf        谷歌服务框架
com.huawei.hwstartupguide
com.android.calllogbackup
com.google.android.partnersetup    谷歌合作伙伴设置
com.huawei.fido.uafclient
com.android.packageinstaller          打包安装程序
com.huawei.hwireader         阅读
com.android.proxyhandler
com.android.systemui.feature_cover
com.android.frameworkhwext.overlay.dark
com.huawei.contactscamcard         扫名片
com.google.android.printservice.recommendation
com.android.managedprovisioning          工作资料设置
com.huawei.himovie             华为视频
com.huawei.hitouch           智慧识屏
com.huawei.compass         指南针
com.android.dreams.phototable        图片屏保程序
com.huawei.vrservice        华为VR服务
com.huawei.android.dsdscardmanager          双卡管理
com.huawei.android.hwaps
com.huawei.android.hwouc         软件更新
com.huawei.android.wfdft         WLAN直连
com.android.wallpaper.livepicker     动态壁纸
com.huawei.mmitest
com.android.apps.tag        标记
com.huawei.powergenie
com.google.android.gms.policy_sidecar_aps
com.qeexo.smartshot           智能截屏
com.google.android.backuptransport       谷歌备份传输
com.huawei.android.instantonline
com.android.storagemanager      存储空间管理器
com.android.settings      设置
com.vmall.client            华为商城
com.android.calculator2        计算器
com.huawei.pcassistant
com.huawei.android.projectmenu       工程菜单
com.android.cts.ctsshim
com.huawei.android.remotecontroller        智能遥控
com.huawei.android.pushagent         推送服务
com.android.vpndialogs
com.huawei.android.hwupgradeguide       升级向导
com.android.email      电子邮件
com.android.phone      拨号服务
com.android.shell
com.android.wallpaperbackup
com.android.providers.blockednumber        存储已屏蔽的号码
com.huawei.hwapplet
com.android.providers.userdictionary          用户词典
com.android.emergency           个人紧急信息
com.huawei.scanner        智慧视觉
com.android.huawei.HiMediaEngine
com.huawei.suggestion         情景智能
com.android.location.fused        融合定位
com.android.deskclock       时钟
com.android.systemui        系统用户界面
com.android.bluetoothmidiservice
com.huawei.hwdetectrepair           智能检测
com.huawei.hwvoipservice            智能电话
com.huawei.android.karaoke       K歌特效
com.huawei.hbs.framework      华为智服务引擎
com.huawei.bd           用户体验改进计划
com.huawei.ca            CA
com.huawei.skytone        天际通数据服务
com.huawei.aod         灭屏显示
com.huawei.ims
com.huawei.lbs         融合定位
com.android.bluetooth           蓝牙
com.android.providers.contacts              联系人存储
com.android.captiveportallogin
com.huawei.hiaction
com.android.hwmirror            镜子
com.huawei.android.airsharing            无线投屏
com.huawei.rcsserviceapplication          华为RCS服务
```



# 华为手机、EMUI、应用包名，内置app卸载参考

https://rescenter.top/newsinfoview-105.html

```
package:com.huawei.hifolder //华为精品应用文件夹，   
package:com.android.mediacenter //华为音乐，一堆广告，还要登录华为ID...
package:com.huawei.hidisk //文件管理,功能挺全，但是竟然强行绑定云空间...
package:com.huawei.android.thememanager //主题，里面一堆巨丑&要付费的主题
package:com.huawei.intelligent //手机桌面滑到最左侧的智能情景模式...全是广告，唉
package:com.huawei.appmarket //华为应用市场，连skype、google都搜不出来的市场有鸟用？
package:com.huawei.wallet //华为钱包，用不着，刷卡有云闪付，公交卡有大都会。
package:com.huawei.android.findmyphone //查找手机，1000块的手机查找什么手机？
package:com.huawei.phoneservice //会员服务，不好意思，不是会员
package:com.android.browser //（华为）浏览器，看到我的文章上方推送的新闻了吧    
package:com.android.soundrecorder //录音机，卸载防止系统悄咪咪的监听我
package:com.baidu.input_huawei //百度输入法-华为版，起到1+1>2的效果，笑死我了
package:com.android.contacts //联系人，注意EMUI系统的拨号是属于联系人的子功能，
因此你会发现最终手机主页上的拨号+联系人都不见了
package:com.android.stk //SIM卡应用，已经过了2G时代，永别了
package:com.huawei.trustspace //支付保护中心，类似360沙箱，太麻烦不需要
package:com.android.calendar //日历，土味太浓，删了
package:com.huawei.vassistant //语音助手，防窃听，不多说
package:com.android.gallery3d //图库（系统相册），功能挺好，强绑华为云空间令人不爽，卸了
package:com.huawei.himovie //华为视频
adb shell pm uninstall --user 0 com.android.browser    浏览器
adb shell pm uninstall --user 0 com.android.cts.priv.ctsshim    旅行助手
adb shell pm uninstall --user 0 com.android.dreams.basic    基本互动屏保
adb shell pm uninstall --user 0 com.android.keyguard    华为杂志锁屏
adb shell pm uninstall --user 0 com.android.stk    SIM卡应用
adb shell pm uninstall --user 0 com.baidu.input_huawei    百度输入法华为版
adb shell pm uninstall --user 0 com.huawei.android.hwpay    安全支付
adb shell pm uninstall --user 0 com.huawei.android.karaoke    卡拉ok
adb shell pm uninstall --user 0 com.huawei.appmarket    应用市场
adb shell pm uninstall --user 0 com.huawei.bd    用户体验计划
首先列出我个人已经卸载掉的项目：
1.com.huawei.hwstartupguide安装初始向导 ，没什么用就卸了2.com.huawei.hifolder精品文件夹，没什么用就卸了
3.com.huawei.himovie.overseas自带(华为)播放器（推荐替换成MX PLAYER）
4.com.android.mediacenter 自带(华为)音乐库 （推荐替换成QQ或者呢netease）
5.com.huawei.weather 华为天气（小挂件）（还挺好用的，我是卸了换墨迹天气了）
6.com.huawei.browser国内浏览器&com.huawei.browserhomepage浏览器首页（需要浏览器的别卸载，因为国际版已经内置Chrome所以这里卸了）
7.com.huawei.hicard华为卡片服务，没什么用就卸了
8.com.huawei.health华为健康服务，没什么用就卸了
9.com.huawei.securityserver华为隐私伺服、隐私空间，没什么用就卸了
securitymgr(设置里)
securitypluginbase(插件里)
10.com.huawei.powergenie 华为超级省电服务，没什么用就卸了
11.com.huawei.contacts.sync 联系人同步服务，没什么用就卸了
然后是 卸载或者禁止显示（仍然保留安装包）的指令，可以自行斟酌，推荐不想看到的disable就行，如项目不正常 可使用adb shell pm enable +包名称 启用；举个栗子：
adb shell pm uninstall --user 0 com.google.android.projection.gearhead谷歌车载驾驶
adb shell pm disable com.google.android.marvin.talkback谷歌回聊服务

./ 以下内容需要再添加前缀com.huawei. 得到完整的包名，这里为了方便排序省略了 /.
./ ~!掉的是个人以及disable的未见“不良反应” /.
android.chr HwChrService充电识别管理服务
android.FloatTasks 侧边浮动窗
android.hsf 华为全家桶框架基础(service frames)
android.hwaps 华为APS服务（非关键服务）
android.hwouc 华为提示系统更新弹窗
android.instantonline 华为一键热点应用（共享热点）
android.instantshare 华为机对机共享（下拉菜单里有）
android.internal.app 内置APP，不知道干啥的
~!android.karaoke 音乐K歌功能
~!android.calendar 华为日历
android.launcher 华为启动器
android.mirrorshare 无线镜像分享功能
android.overlay.modules.modulemetadata 覆盖模组
android.projectmenu 系统关键进程
android.pushagent 华为推送代理进程
android.thememanager 华为主题
android.wfdft WLAN连接器
androidx 系统关键进程
aod 息屏指纹图标
appmarket 系统关键进程华为应用市场
arengine.service 扫描AR引擎
~!assetsync 华为资产同步
~!assetsyncservice 华为资产同步服务项
autoinstallapkfrommcc 从MCC里自动安装
~!bd 华为用户体验改进（设置菜单靠后位置一览）
behaviorauth 行为授权功能
bluetooth 系统关键进程蓝牙
camera 系统关键进程相机
camerakit.impl 系统关键进程附件
coauthservice 联合认证服务
desktop.explorer 系统关键进程系统桌面
desktop.systemui 系统关键进程系统桌面
deviceauth 系统关键进程设备验证
dmsdp Hicar相关服务
dsdscardmanager 系统关键进程双卡管理器
featurelayer.featureframework 系统关键进程
featurelayer.sharedfeature.map 系统关键进程
fido.uafclient 快速用户身份认证
hff Hff服务
hiai 华为智慧AI
~!hicloud 华为云服务基础（系统设置第一栏显示）
~!com.huawei.hwid 华为用户账户ID服务（登录云服务）
~!com.huawei.fans 花粉俱乐部（国内系统有）
hidisk 系统关键进程 存储管理器（文档管理）
~!hitouch 华为HiTouch
华为CARD服务相关：HiTouch、HiVision、HiVoice、iconnect智能家居、场景包
hiview 图片属性获取
hiviewtunnel 图片属性获取组件
hwasm 华为ASM，可能是系统进程
~!hwblockchain 区块链服务
hwddmp 系统日志
hwdetectrepair 系统检测和修复
hwdockbar EMUI10.1侧边栏格式
hwMultiScreenShot 截屏服务
hwpanpayservice 华为pan支付
hwusbearphoneupdate USB耳机
iaware 性能自动调度（大概打游戏用得上）
~!iconnect 智能家居链接
imedia.dolby dolby音效
ims 华为Image Management Service
~!intelligent 华为购物情景智能
~!KoBackup KO备份
languagedownloader 语言下载工具
lbs 系统关键进程HwLBSService基站定位服务
livewallpaper.paradise 动态桌面背景图
~!magazine 华为锁屏杂志（或者叫keyguard）
~!mmitest 进入工程调试模式入口
motionservice 划屏手势支持
msdp 多播路由
multimedia.audioengine 多媒体音频引擎
nb.service 数据管理服务
~!nearby 查看附近
numberidentity 电话号码识别
omacp 邮件设置功能
~!parentcontrol 家长控制
pcassistant Hisuite连接电脑助手
permissioncontroller.overlay 系统关键进程权限管理器涵盖
phoneservice 系统关键进程通话服务
printservice 打印服务
~!recsys recommended system服务
~!remotepassword 远程密码
scanner 相机扫码功能
screenrecorder 录屏功能
search 系统设置搜索条功能
smartshot 双击关节识别服务
synergy 多屏协同驱动
systemmanager 手机管家（防病毒清理垃圾等等），据说删了有BUG
systemserver 系统伺服
tmecustomize TME这个东西的用户自定义
trustagent 华为智能锁屏解锁
vassistant 华为智慧voice语音服务
videoeditor 视频编辑器
~!wallet.sdk.walletsdk 华为钱包kit工具
wifieapsimplmn WiFi相关功能
wifiprobqeservice WiFi相关功能
```

