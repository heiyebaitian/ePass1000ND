# ePass1000ND
前几天在做淘宝看广告任务的时候发现，飞天诚信ePass1000ND提供标准安全中间件CSP接口，提供标准安全中间件PKCS#11接口，支持多个密钥的存储，最高可存储6对密钥；硬件实现数字签名，支持X.509 v3标准证书格式，能与多种PKI应用无缝集成等等功能。并且这东西几十元一枚，应用需求没那么多的情况下比Yubikey实惠多了。

在网上搜了一下，使用这个作为个人应用的例子还是挺少见的，只找到一篇[uKey双向认证https](https://juejin.cn/post/6885609293891141645)可以作为参考的，配套工具只有某收费站点有下载，而且也不知道是不是完整，于是找了一个贵一点但是提供SDK的商家买了一枚。

**PS:** Yubikey相关使用记录看[这里](https://github.com/awardat/YubiKey-Guide_CHS)

## 选购

我选择的是飞天诚信 ePass1000ND 3.0版本，应该也是现在的最新版本。理论上说更早的版本也可以用，但是对于这种无法升级固件的设备来说还是尽量使用新版本为上。

理论上 ePass系列的其他产品也可以实现相同效果。

## 安装软件

ePass1000ND为免驱动版本，无需专门安装驱动，插入系统后会被识别为USB输入设备。但是此时是不能使用的，还需要安装中间件提供标准接口和状态显示。

在将设备插入USB接口后操作系统会自动作为HID设备安装驱动，在设备管理器中可见位于人机接口设备\USB输入设备，使用AIDA64可见HID Token M32.

![USB设备](C:\src\ePass1000ND\img\hid_in_usb.png)

此时是不能对设备进行操作的，还需要安装中间件实现与设备的通信，运行eps1knd_stdSimpChinese.exe安装中间件，一路下一步完成就可以了。

![安装中间件](C:\src\ePass1000ND\img\midware_inst.png)

同时在系统通知栏会多出一个ePassNG证书管理工具的图标，双击弹出主窗口

![ePassNG证书管理工具](C:\src\ePass1000ND\img\ePass_CertMgr.png)

证书管理工具中显示了当前令牌和其中安装的证书，可以显示证书信息，如果是新的或者初始化过的令牌则证书栏为空。

**首次初始化**

如果插入令牌后，证书管理工具提示”KEY已插入“，但是在选择令牌的地方却是空白的，表示令牌尚未进行过初始化，需要运行PKIInit_M32.exe对令牌进行首次初始化。

![初始化令牌](C:\src\ePass1000ND\img\new_ukey_init.png)

根据提示按回车初始化令牌，在初始化时证书管理工具会快速的弹出令牌初始化过程的提示，直到提示初始化完成后按ESC退出初始化程序，初始化完成会创建默认SOPIN和USERPIN，重试次数为15次。

| 账号    | 口令   |
| ------- | ------ |
| SOPIN   | rockey |
| USERPIN | 1234   |

**注：**此工具仅可对首次使用的新令牌进行初始化（我将其称为首次初始化），已经进行过首次初始化的令牌需要使用下述管理工具进行初始化进行复位。

进行过初始化后即可使用ePassNG PKI 管理工具（管理员）查看和管理令牌

![ePassNG PKI 管理工具](C:\src\ePass1000ND\img\epass_mgr_adm.png)

在此界面上可以看到令牌信息和管理功能，还有一个用户版管理软件，主要少了最下行三个按钮的功能。

点击初始化进行令牌初始化界面，输入SOPIN，新的USERPIN进行初始化，如果不知道SOPIN则无法进行初始化。

![初始化令牌](C:\src\ePass1000ND\img\ukey_init.png)

点击登录输入USERPIN后即可查看和管理令牌中保存的数据。

![数据管理](C:\src\ePass1000ND\img\data_mgr.png)

可以在此界面导入PFX、P12、P7B、X509格式证书，查看已有的证书信息，导出公钥和证书，删除证书。

## 获取GDCA免费邮件签名证书

选择这个证书主要有几个个原因，一是免费；二是相比自签证书，CA签发的证书在使用的时候不会有报错；三是虽然该证书在CA公布的证书类别是`基础邮件证书`，但是该证书密钥用法包括`数字签名、密钥加密、数据加密`，增强型密钥用法包括`客户端身份验证、电子邮件保护`，也可以直接用于签名、加密等用途。

打开[SSL证书,HTTPS证书,代码签名证书-数安时代-数字证书电子商城 (trustauth.cn)](https://certmall.trustauth.cn/Home/Login/login.html)

点击右上角注册/登录，进入个人中心

![个人中心](C:\src\ePass1000ND\img\GDCA_member_home.png)

找到免费证书-GDCA安全邮件证书，点击免费申请

![申请免费邮件证书](C:\src\ePass1000ND\img\GDCA_freeEmail.png)

输入邮箱，点击发送验证邮件后会收到一封来自**autovalidation@trustauth.cn**的标题为`GDCA电子邮件验证码, YYYY-MM-DD HH:MM:SS`的邮件，将其中的验证码填入以上表单，设置一个强壮的口令，填入手机号和随机码，勾选用户协议后提交申请即可。

> 请选择一个**可靠**的方式保存口令，在导入证书时需要使用到。
>
> 如果是没有验证过的手机号需要发送手机验证码进行验证，手机号应该是实名认证用的，不会体现在证书中。
>
> 似乎Gmail收不到验证邮件。

跳转到下个页面就可以下载证书了，下载得到一个zip文件，其中包含了一个pfx文件和一个[安装说明页面](https://www.trustauth.cn/gdca-ccpb)的快捷方式，导入windows和用作邮件签名的话直接按说明操作即可。

>证书只能下载一次，如果下载的证书文件丢了就只能重新申请了。

证书有效期自申请之时刻起**一年**，到期直接再申请一个即可。

该证书的持有人信息**只包括申请时使用的邮箱地址**，如果需要包含更多信息则只能购买收费产品了。

## 向令牌中导入证书

打开ePassNG PKI 管理工具（管理员）并进行用户登录

![ePassNG PKI 管理工具](C:\src\ePass1000ND\img\epass_mgr_adm.png)

登录后会自动跳转到数据管理界面

![数据管理](C:\src\ePass1000ND\img\data_mgr.png)

点击导入，在弹出窗口中选择数字证书并输入证书密码

![导入证书](C:\src\ePass1000ND\img\Cert_import.png)

点击确定后开始执行导入过程，驻留的证书管理工具会在状态栏弹出“正在生成私钥…”、”生成私钥成功“、“正在生成公钥…”、“生成公钥成功”、“正在产生X509证书…”、“产生X509证书”等提示。

同时操作系统会提示导入GDCA根证书的确认信息，点击是确认。

![GDCA根证书](C:\src\ePass1000ND\img\cert_root.png)

此时在数据管理界面中即可看到刚才安装了一个用户证书、一个机构根证书和一个机构颁发者证书，同时这些证书也可以在Windows的证书管理器中查看。

![证书列表](C:\src\ePass1000ND\img\Cert_list.png)

如果有多个证书重复以上过程导入，因为机构证书已经导入，在提示有重复证书的时候点两次否即可。
