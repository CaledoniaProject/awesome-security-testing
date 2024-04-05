# 说明

本章节收录提权相关案例

* [1. Windows系统提权](#1-windows系统提权)
  * [常见利用手法和案例](#常见利用手法和案例)
  * [最佳实践](#最佳实践)
* [2. Linux系统提权](#2-linux系统提权)
  * [常见利用手法和案例](#常见利用手法和案例-1)
  * [最佳实践](#最佳实践-1)
* [3. Mac系统提权](#3-mac系统提权)
  * [常见利用手法和案例](#常见利用手法和案例-2)
  * [最佳实践](#最佳实践-2)
* [4. Android系统提权](#4-android系统提权)
  * [常见利用手法和案例](#常见利用手法和案例-3)
  * [最佳实践](#最佳实践-3)
* [5. 其他类型](#5-其他类型)
  * [常见利用手法和案例](#常见利用手法和案例-4)

# 1. Windows系统提权

## 常见利用手法和案例

* 条件竞争利用
  * [案例1: 杀毒软件检测病毒和清理通常是两个步骤，通过软连接和条件竞争可能导致任意文件删除，破坏系统](https://www.rack911labs.com/research/exploiting-almost-every-antivirus-software/)
* 配置文件利用
  * [案例1: SaferVPN以system启动openvpn，并加载用户自定义的配置文件。通过指定openvpn DLL插件可以实现提权](https://github.com/VerSprite/research/blob/master/advisories/VS-2018-024.md)
* 权限配置错误
  * [案例1: Mcafee McAWFwk COM接口允许NT AUTHORITY\SELF访问，可以调用RunProgram接口，以SYSTEM权限执行命令](https://the-deniss.github.io/posts/2021/05/17/discovering-and-exploiting-mcafee-com-objects.html)
  * 案例2:【linux】某云主机HIDS使用root执行nginx -V，可伪造进程实现提权
* 软链接利用
  * [案例1: DmEnrollemnt 导出注册表时，没有检查是否为软连接，可读取没有权限的内容](https://docs.google.com/document/d/120J4YG5FoycAsOhMe0SRYt_8sgEYY8A23tQBRwR5zSU/edit)
  * [案例2: Cylance 允许 Users 用户组修改日志文件，通过软连接可以实现任意文件修改](https://www.atredis.com/blog/cylance-privilege-escalation-vulnerability)
    * [此类漏洞利用技巧: Windows Exploitation Tricks: Exploiting Arbitrary File Writes for Local Elevation of Privilege](https://googleprojectzero.blogspot.com/2018/04/windows-exploitation-tricks-exploiting.html)
* MITM配合利用
  * [案例1: 某网站ActiveX安装时会将icbc.com加入IE信任列表并允许执行高权限vbscript。在有中间人劫持的情况下（如WIFI热点），可以实现RCE](http://wy.zone.ci/bug_detail.php?wybug_id=wooyun-2015-096339)
* 业务逻辑错误
  * [案例1: 如果远程管道已经存在，PsExec将直接使用这个管道，导致可以提权到SYSTEM](https://medium.com/tenable-techblog/psexec-local-privilege-escalation-2e8069adc9c8)
  * [案例2: McAfee根据文件路径实现进程保护，只需要将主程序和DLL复制到其他路径即可绕过](https://dmaasland.github.io/posts/mcafee.html)
  * [案例3: PPL进程是受保护的，正常情况下无法注入无签名DLL(待确认)。通过修改计划任务的COM配置，使用TreatAs注入DotNetToJScript脚本，可以实现任意.NET代码执行。SERVICE_CONFIG_LAUNCH_PROTECTED标志位只能对有特殊证书签名的进程开启，所以选择了clipup.exe。微软的修复方案是屏蔽jscript等DLL](https://bugs.chromium.org/p/project-zero/issues/detail?id=1336)
  * [案例4: Palo Alto minifilter可以使用fltmc unload命令卸载，导致防护失效](https://www.c0d3xpl0it.com/2019/01/bypassing-paloalto-traps-edr-solution.html)

## 最佳实践

待补充

# 2. Linux系统提权

## 常见利用手法和案例

* 环境变量污染
  * [案例1: keybase-redirector是一个suid过的程序。这个程序会在启动阶段执行fusermount，由于没有过滤$PATH，导致本地问题提权](https://hackerone.com/reports/426944)
  * [案例2: root_trace是一个suid过的程序。通过设置PYTHONPATH可以加载非预期的python模块，实现本地提权](https://bugs.chromium.org/p/project-zero/issues/detail?id=912)
* 参数特性利用
  * [案例1: juniper telentd有SUID标志且可以通过--debug -p参数自定义login shell，指定/bin/sh可实现本地提权](https://starlabs.sg/advisories/21-0223/)
  * [案例2: AMANDA的runtar是一个suid过的程序。程序会执行tar命令，且支持自定义tar参数。由于没有过滤--rsh-command参数，导致提权问题](https://github.com/BaRRaKudaRain/ExploitDB/blob/master/Hacker%20House/amanda-backup.txt)
* 配置文件利用
  * [案例1: Broadband网关允许修改SMB配置，但是没有过滤转义斜线，可覆盖SMB配置、允许跟随软链接。通过创建到“/”的链接，可以修改passwd文件，实现空密码telnet登录，实现低权限账号提权](https://www.sec-consult.com/en/blog/advisories/local-root-jailbreak-via-network-file-sharing-flaw-in-all-adb-broadband-gateways-routers/)
* 条件竞争利用
  * [案例1: MySQL先打开日志文件，再降低自身权限，通过构造软连接可以实现任意文件读写](http://legalhackers.com/advisories/MySQL-Maria-Percona-RootPrivEsc-CVE-2016-6664-5617-Exploit.html)

## 最佳实践

待补充

# 3. Mac系统提权

## 常见利用手法和案例

* 审计XPC接口
  * [案例1: keybase提供了XPC接口，可以执行特权操作。接口没有校验调用程序的签名，导致提权漏洞](https://hackerone.com/reports/397478)
  * [案例2: TeamViewer提供了增加打印机的XPC接口，没有调用来源校验，可以安装任意PKG文件，实现提权](https://theevilbit.github.io/posts/teamviewer_lpe/)

## 最佳实践

待补充

# 4. Android系统提权

## 常见利用手法和案例

* 审计Intent权限
  * [案例1: 小米分身工具，切换身份intent开启了导出，且接口支持关闭密码校验，可以使用adb低权限直接调用，导致鉴权失效](https://labs.f-secure.com/advisories/xiaomi-second-space)
* 审计广播处理逻辑
  * [案例1: SamsungSMT动态导出了一个receiver，会直接加载SMT_ENGINE_PATH指定的库，由于没有校验调用方导致提权问题](https://paper.seebug.org/1070/)

## 最佳实践

待补充

# 5. 其他类型

## 常见利用手法和案例

* 业务逻辑问题
  * [案例1: Google允许ping sitemap，同时还允许302跳转；结合两个漏洞，可控制其他站点的 sitemap](http://www.tomanthony.co.uk/blog/google-xml-sitemap-auth-bypass-black-hat-seo-bug-bounty/)




