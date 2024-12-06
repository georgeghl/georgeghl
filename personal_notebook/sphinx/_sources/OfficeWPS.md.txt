

# kms激活OFFICE

首先你的OFFICE必须是VOL版本，否则无法激活。 找到你的office安装目录，比如

```powershell
"C:\Program Files (x86)\Microsoft Office\Office16"
```

64位的就是

```powershell
"C:\Program Files\Microsoft Office\Office16"
```

office16是office2016，office15就是2013，office14就是2010.

然后目录对的话，该目录下面应该有个OSPP.VBS。

接下来我们就cd到这个目录下面，例如：

```powershell
#32位执行：
cd "C:\Program Files (x86)\Microsoft Office\Office16"
#64位执行：
cd "C:\Program Files\Microsoft Office\Office16"
```

然后执行注册kms服务器地址：

```powershell
cscript ospp.vbs /sethst:101.34.236.228
```

/sethst参数就是指定kms服务器地址。

一般ospp.vbs可以拖进去cmd窗口，所以也可以这么弄：

```powershell
#32位执行：
cscript "C:\Program Files (x86)\Microsoft Office\Office16\OSPP.VBS" /sethst:101.34.236.228
#64位执行：
cscript "C:\Program Files\Microsoft Office\Office16\OSPP.VBS" /sethst:101.34.236.228
```

一般来说，“一句命令已经完成了”，但一般office不会马上连接kms服务器进行激活，所以我们额外补充一条手动激活命令：

```powershell
cscript ospp.vbs /act
```

如果提示看到successful的字样，那么就是激活成功了，重新打开office就好。

\##　如果遇到报错，请检查：

> 1. 你的系统/OFFICE是否是批量VL版本
> 2. 是否以管理员权限运行CMD
> 3. 你的系统/OFFICE是否修改过KEY/未安装GVLK KEY
> 4. 检查你的网络连接
> 5. 服务器繁忙，多试试（点击检查KMS服务是否可用）
> 6. 根据出错代码自己搜索出错原因



## Office KMS 激活密钥（GVLK Key）

### Office 2010

https://learn.microsoft.com/zh-cn/previous-versions/office/office-2010/ee624355(v=office.14)?redirectedfrom=MSDN

| **套件**     |                               |                               |
| ------------ | ----------------------------- | ----------------------------- |
|              | Office Professional Plus 2010 | VYBBJ-TRJPB-QFQRF-QFT4D-H3GVB |
|              | Office Standard 2010          | V7QKV-4XVVR-XYV4D-F7DFM-8R6BM |
|              | Office Home and Business 2010 | D6QFG-VBYP2-XQHM7-J97RH-VVRCK |
| **独立产品** |                               |                               |
|              | Access 2010                   | V7Y44-9T38C-R2VJK-666HK-T7DDX |
|              | Excel 2010                    | H62QG-HXVKF-PP4HP-66KMR-CW9BM |
|              | SharePoint Workspace 2010     | QYYW6-QP4CB-MBV6G-HYMCJ-4T3J4 |
|              | InfoPath 2010                 | K96W8-67RPQ-62T9Y-J8FQJ-BT37T |
|              | OneNote 2010                  | Q4Y4M-RHWJM-PY37F-MTKWH-D3XHX |
|              | Outlook 2010                  | 7YDC2-CWM8M-RRTJC-8MDVC-X3DWQ |
|              | PowerPoint 2010               | RC8FX-88JRY-3PF7C-X8P67-P4VTT |
|              | Project Professional 2010     | YGX6F-PGV49-PGW3J-9BTGG-VHKC6 |
|              | Project Standard 2010         | 4HP3K-88W3F-W2K3D-6677X-F9PGB |
|              | Publisher 2010                | BFK7F-9MYHM-V68C7-DRQ66-83YTP |
|              | Word 2010                     | HVHB3-C6FV7-KQX9W-YQG79-CRY7T |
| **Visio**    |                               |                               |
|              | Visio Premium 2010            | D9DWC-HPYVV-JGF4P-BTWQB-WX8BJ |
|              | Visio Professional 2010       | 7MCW8-VRQVK-G677T-PDJCM-Q8TCP |
|              | Visio Standard 2010           | 767HD-QGMWX-8QTDB-9G3R2-KHFGJ |

### Office 2013

https://learn.microsoft.com/zh-cn/previous-versions/office/dn385360(v=office.15)?redirectedfrom=MSDN

| 产品                          | GVLK                          |
| :---------------------------- | :---------------------------- |
| Office 2013 Professional Plus | YC7DK-G2NP3-2QQC3-J6H88-GVGXT |
| Office 2013 Standard          | KBKQT-2NMXY-JJWGP-M62JB-92CD4 |
| Project 2013 Professional     | FN8TT-7WMH6-2D4X9-M337T-2342K |
| Project 2013 Standard         | 6NTH3-CW976-3G3Y2-JK3TX-8QHTT |
| Visio 2013 Professional       | C2FG9-N6J68-H8BTJ-BW3QX-RM3B3 |
| Visio 2013 Standard           | J484Y-4NKBF-W2HMG-DBMJC-PGWR7 |
| Access 2013                   | NG2JY-H4JBT-HQXYP-78QH9-4JM2D |
| Excel 2013                    | VGPNG-Y7HQW-9RHP7-TKPV3-BG7GB |
| InfoPath 2013                 | DKT8B-N7VXH-D963P-Q4PHY-F8894 |
| Lync 2013                     | 2MG3G-3BNTT-3MFW9-KDQW3-TCK7R |
| OneNote 2013                  | TGN6P-8MMBC-37P2F-XHXXK-P34VW |
| Outlook 2013                  | QPN8Q-BJBTJ-334K3-93TGY-2PMBT |
| PowerPoint 2013               | 4NT99-8RJFH-Q2VDH-KYG2C-4RD4F |
| Publisher 2013                | PN2WF-29XG2-T9HJ7-JQPJR-FCXK4 |
| Word 2013                     | 6Q7VD-NX8JD-WJ2VH-88V73-4GBJ7 |

### Office 2016

https://learn.microsoft.com/zh-cn/deployoffice/vlactivation/gvlks

| **Product**               | **GVLK**                        |
| :------------------------ | :------------------------------ |
| Office 专业增强版 2016    | XQNVK-8JYDB-WJ9W3-YJ8YR-WFG99   |
| Office Standard 2016      | JNRGM-WHDWX-FJJG3-K47QV-DRTFM   |
| Project Professional 2016 | YG9NW-3K39V-2T3HJ-93F3Q-G83KT   |
| Project Standard 2016     | GNFHQ-F6YQM-KQDGJ-327XX-KQBVC   |
| Visio Professional 2016   | PD3PC-RHNGV-FXJ29-8JK7D-RJRJK   |
| Visio Standard 2016       | 7WHWN-4T7MP-G96JF-G33KR-W8GF4   |
| Access 2016               | GNH9Y-D2J4T-FJHGG-QRVH7-QPFDW   |
| Excel 2016                | 9C2PK-NWTVB-JMPW8-BFT28-7FTBF   |
| OneNote 2016              | DR92N-9HTF2-97XKM-XW2WJ-XW3J6   |
| Outlook 2016              | R69KK-NTPKF-7M3Q4-QYBHW-6MT9B   |
| PowerPoint 2016           | J7MQP-HNJ4Y-WJ7YM-PFYGF-BY6C6   |
| Publisher 2016            | F47MM-N3XJP-TQXJ9-BP99D-8 837 K |
| Skype for Business 2016   | 869NQ-FJ69K-466HW-QYCP2-DDBV6   |
| Word 2016                 | WXY84-JN2Q9-RBCCQ-3Q3J3-3PFJ6   |

### Office 2019

https://learn.microsoft.com/zh-cn/deployoffice/vlactivation/gvlks

| **Product**             | **GVLK**                      |
| :---------------------- | :---------------------------- |
| Office 专业增强版 2019  | NMMKJ-6RK4F-KMJVX-8D9MJ-6MWKP |
| Office 标准版 2019      | 6NWWJ-YQWMR-QKGCB-6TMB3-9D9HK |
| Project 专业版 2019     | B4NPR-3FKK7-T2MBV-FRQ4W-PKD2B |
| Project 标准版 2019     | C4F7P-NCP8C-6CQPT-MQHV9-JXD2M |
| Visio 专业版 2019       | 9BGNQ-K37YR-RQHF2-38RQ3-7VCBB |
| Visio 标准版 2019       | 7TQNQ-K3YQQ-3PFH7-CCPPM-X4VQ2 |
| Access 2019             | 9N9PT-27V4Y-VJ2PD-YXFMF-YTFQT |
| Excel 2019              | TMJWT-YYNMB-3BKTF-644FC-RVXBD |
| Outlook 2019            | 7HD7K-N4PVK-BHBCQ-YWQRW-XW4VK |
| PowerPoint 2019         | RRNCX-C64HY-W2MM7-MCH9G-TJHMQ |
| Publisher 2019          | G2KWX-3NW6P-PY93R-JXK2T-C9Y9V |
| Skype for Business 2019 | NCJ33-JHBBY-HTK98-MYCV8-HMKHJ |
| Word 2019               | PBX3G-NWMT6-Q7XBW-PYJGG-WXD33 |

### Office 2021

https://learn.microsoft.com/zh-cn/deployoffice/vlactivation/gvlks

| **Product**                  | **GVLK**                      |
| :--------------------------- | :---------------------------- |
| Office LTSC 专业增强版 2021  | FXYTK-NJJ8C-GB6DW-3DYQT-6F7TH |
| Office LTSC 标准版 2021      | KDX7X-BNVR8-TXXGX-4Q7Y8-78VT3 |
| Project Professional 2021    | FTNWT-C6WBT-8HMGF-K9PRX-QV9H8 |
| Project Standard 2021        | J2JDC-NJCYY-9RGQ4-YXWMH-T3D4T |
| Visio LTSC Professional 2021 | KNH8D-FGHT4-T8RK3-CTDYJ-K2HT4 |
| Visio LTSC Standard 2021     | MJVNY-BYWPY-CWV6J-2RKRT-4M8QG |
| Access LTSC 2021             | WM8YG-YNGDD-4JHDC-PG3F4-FC4T4 |
| Excel LTSC 2021              | NWG3X-87C9K-TC7YY-BC2G7-G6RVC |
| Outlook LTSC 2021            | C9FM6-3N72F-HFJXB-TM3V9-T86R9 |
| PowerPoint LTSC 2021         | TY7XF-NFRBR-KJ44C-G83KF-GX27K |
| Publisher LTSC 2021          | 2MW9D-N4BXM-9VBPG-Q7W6M-KFBGQ |
| Skype for Business LTSC 2021 | HWCXN-K3WBT-WJBKY-R8BD9-XK29P |
| Word LTSC 2021               | TN8H9-M34D3-Y64V9-TR72V-X79KV |

### 汇总

| Office版本                    | KMS激活序列号                 |
| ----------------------------- | ----------------------------- |
| Office Professional Plus 2021 | FXYTK-NJJ8C-GB6DW-3DYQT-6F7TH |
| Office Professional Plus 2019 | NMMKJ-6RK4F-KMJVX-8D9MJ-6MWKP |
| Office Professional Plus 2016 | XQNVK-8JYDB-WJ9W3-YJ8YR-WFG99 |
| Office Professional Plus 2013 | YC7DK-G2NP3-2QQC3-J6H88-GVGXT |
| Office Professional Plus 2010 | VYBBJ-TRJPB-QFQRF-QFT4D-H3GVB |











# Word

## 保存文件时，提示需要登陆上载

关闭：OneDrive设置=》Office=》文件协助：通过与Office同步文件，与其他用户一起协作处理Office文件。

<img src="./img/1.webp" alt="v2-977f70b2f35b2a1536d575ff359fa7a0_720w" style="zoom: 67%;" />





# **在 Word 中更改默认字体**

1. 转到" 开始"， 然后选择"字体"对话框启动器 。
2. 选择想要使用的字体和大小。
3. 单击“ 设为默认值 ”。
4. 选择下列选项之一： 仅此文档 基于 Normal 模板的所有文档 。
5. 选择" 确定" 两次。





# 【WORD+PPT】提取PPT中的文字内容到WORD

老师发的课件有时候图片太多，打印时如果一张一张删除比较费时，因而选择将PPT中的文字全部提取，输出为Word。

**操作[[1\]](https://zhuanlan.zhihu.com/p/573434179#ref_1)：**

1、打开PPT文件，按【Fn+Alt + F11】键打开VBA编辑器。

2、点击【工具】-【引用】，找到【Microsoft Word 16.0 Object Library】勾选添加。

3、点击【插入】-【模块】，复制以下代码进编辑器。

4、最后点击【运行】代码，或者按【F5】键，PPT就成功转换成Word了。

```vb
Sub Main()
On Error Resume Next
Dim temp As New Word.Document, tmpShape As Shape, tmpSlide As Slide
For Each tmpSlide In ActivePresentation.Slides
For Each tmpShape In tmpSlide.Shapes
temp.Range().Text = temp.Range() + tmpShape.TextFrame.TextRange.Text
Next tmpShape
Next tmpSlide
temp.Application.Visible = True
End Sub
```