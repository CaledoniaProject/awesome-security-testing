# 说明

本章节收录文件操作、配置相关的漏洞

* [1.1 普通文件操作](#11-普通文件操作)
   * [常见利用手法和案例](#常见利用手法和案例)
   * [最佳实践](#最佳实践)
* [1.2 文件上传下载](#12-文件上传下载)
   * [常见利用手法和案例](#常见利用手法和案例-1)
   * [最佳实践](#最佳实践-1)
* [1.3 文件包含](#13-文件包含)
   * [常见利用手法和案例](#常见利用手法和案例-2)
   * [最佳实践](#最佳实践-2)

# 1.1 普通文件操作

## 常见利用手法和案例

文件目录遍历

* 使用 `/`、`\` 等字符跳出目标目录
  * [案例1:【web】zimbra mail会调用unrar解压缩文件，unrar在unix系统下只会检查文件名是否以`../`开头，因此名为`..\..\..\tmp/shell`的文件可以绕过检查。由于后面会调用DosSlashToUnix()函数，将`\`转换为`/`，导致文件可以写入任意目录](https://blog.sonarsource.com/zimbra-pre-auth-rce-via-unrar-0day/)
  * [案例2:【iot】RV340设备内置nginx服务器，并通过`-f /tmp/websession/token/$cookie_sessionid`来判断是否认证通过，通过传入`Cookie: sessionid =../../../etc/passwd`头可以绕过认证](https://blog.security.sea.com/posts/pwn2own-2021-rv340/)
  * [案例3:【android】用户在app里下载附件时，由于附件文件名包含 `../`，导致任意文件覆盖](https://bugs.chromium.org/p/project-zero/issues/detail?id=1342)
  * [案例4: Android 邮件客户端: ContentProvider 未对URI解码，可使用 ..%2F 遍历文件系统](https://www.anquanke.com/post/id/84731)
* 使用软连接跳出目标目录
  * [案例1:【iot】构造一个带有软连接的zip，在Pulse Secure后台上传，解压后可以通过web访问敏感文件](https://research.nccgroup.com/2020/10/26/technical-advisory-pulse-connect-secure-arbitrary-file-read-via-logon-message-cve-2020-8255/)
  * [案例2:【iot】TP-Link允许插入USB并共享文件，并会解析硬盘的软连接；默认只有音频文件会跟随软连接，使用xxx.wav作为文件名即可](https://medium.com/tenable-techblog/tp-link-takeover-with-a-flash-drive-d493666f6b39)
  * [案例3:【iot】zyxel存在一样的问题；另外，zyxel的密码是可逆加密的，直接可以还原](https://th0mas.nl/2020/03/26/getting-root-on-a-zyxel-vmg8825-t50-router/)
* 尝试截断字符串实现任意文件读取
  * [案例1:【iot】内容在PPT的50-51页。SSL VPN存在一处任意文件下载，但是文件必须以`.json`结尾。通过填充`/`，让snprintf buffer打满，实现字符串截断，绕过文件后缀名限制](https://gsec.hitb.org/materials/sg2019/D1%20-%20Infiltrating%20Corporate%20Intranets%20Like%20The%20NSA%20-%20A%20Pre-Auth%20Remote%20Code%20Execution%20on%20Leading%20SSL%20VPNs%20-%20Orange%20Tsai%20&%20Tingyi%20Chan.pdf)
    * snprintf(s, 0x40, "/migadmin/lang/%s.json", lang);
  * [案例2:【iot】内容在PPT的25页。KYOCERA打印机存在一处任意文件下载，接口会先检查是否以特定文件后缀结尾(比如`.js`)，然后再做URL解码获取文件名，读取后作为响应返回。由于是先判断后缀再解码，通过`%00`截断可以实现任意文件读取(比如`shadow%00.js`)](https://conference.hitb.org/hitbsecconf2022sin/materials/D2%20COMMSEC%20-%20Cracking%20Kyocera%20Printers%20-%20Yue%20Liu,%20Minghang%20Shen%20&%20Juyang%20Gao.pdf)
  * [案例3:【iot】netgear存在一个任意文件读取漏洞，并使用strstr检查是否包含`.xml`等字样，通过在U盘上构建一个`aaa.xml`目录，再使用../跳出即可实现任意文件读取，e.g `../../mnt/shares/U/evil.xml/../../../../../etc/passwd`](https://flattsecurity.medium.com/finding-bugs-to-trigger-unauthenticated-command-injection-in-a-netgear-router-psv-2022-0044-2b394fb9edc)
* 利用IIS短文件名爆破路径，比如`/otcmso~1`
* 自研Web服务器，URI处置有问题
  * hxxp://a.com/../../../../../.../../../../../../../etc/hosts
  * [hxxp://XXX.XXX.XXX.XXX/.../.../windows/system32/cmd.exe?/c+net+user+admin+admin+/add](https://wy.zone.ci/bug_detail.php?wybug_id=wooyun-2014-058654)

配置文件利用

* 写配置文件利用
  * [案例1:【iot】Grandstream设备启动时候会执行`nvram get gdb_debug_server`获取tftp服务器地址，然后从这个地址拉取gdb并执行，且没有任何校验。设置此参数并重启，可以实现命令执行](https://www.secforce.com/blog/exploiting-grandstream-ht801-ata-cve-2021-37748-cve-2021-37915/)
  * [案例2:【iot】ZTE路由器允许插入U盘，并将内容通过samba共享出来。samba以root启动，开启软连接解析，并允许编辑任意文件。通过在U盘里增加到smb.conf的软连接，可以实现对smb.conf的编辑，在smb.conf里可以增加preexec配置来执行任意命令。作者增加了busybox telnetd命令，重启路由器后就可以用telnet登录了。](https://niemand.com.ar/2018/08/01/rooting-your-router-zte-f670e-abusing-of-an-old-samba/)
  * [案例3:【iot】Pulse Secure 可以通过上传压缩包覆盖配置文件，利用思路是覆盖 watchdog.conf 或者 postgresql.conf。后者可以修改 shared_preload_libraries 实现任意so加载](https://research.nccgroup.com/2020/10/26/technical-advisory-pulse-connect-secure-rce-via-uncontrolled-gzip-extraction-cve-2020-8260/)  
  * [案例4:【iot】netgear路由器允许使用硬盘上的文件作为banner，在执行`banner motd file /etc/passwd`命后令，可以在下次登录时看到/etc/passwd的内容](https://versprite.com/blog/security-research/netgear-nighthawk-router-security-bug/)
* 读配置文件利用
  * [案例1:【iot】Hikvision DS-7604NI-E1重置出厂密码时候需要提供一个security code。这个code是通过modelNumber、serialNumber计算出来的，而且可以从`/upnpdevicedesc.xml`接口获取，导致密码重置漏洞](https://neonsea.uk/blog/2018/08/01/hikvision-keygen.html)

CRLF漏洞

* [案例1:【linux】/proc/XX/status Name字段存在CRLF问题，通过构造 $'sleep\rfake\rline' 这样的名字可以插入多余的行，导致apport降权失败，带来提权问题](https://alephsecurity.com/2021/02/16/apport-lpe/)
* [案例2:【android】bluedroid 在配对时，会记录设备名称到配置文件里，由于没有过滤换行符，导致一些视觉欺骗问题](http://sploit3r.xyz/cve-2017-13284-injection-in-configuration-file/)
* [案例3:【java】FTP URI没有删除换行符，可以实现FTP命令注入](http://blog.blindspotsecurity.com/2017/02/advisory-javapython-ftp-injections.html)
  * ftp://u:p@evil.example.com/foodir%0APORT%2010,1,1,1,5,57/z.txt
* [案例4:【python】CVE-2016-5699案例，urllib可以注入header，结合SSRF漏洞可以写redis，通过`append`函数实现分段写入绕过长度限制，最后通过pickle反序列化实现RCE(py2、py3的格式有区别)](https://www.leavesongs.com/PENETRATION/getshell-via-ssrf-and-redis.html)  

其他利用方法

* 检查压缩包处理是否得当
  * [案例1:【windows】explorer zip解压缩时，如果原始文件是只读的，生成的文件也会是只读的，导致MOTW(Zone Identifier)写入失败，就绕过了安全提示弹窗](https://breakdev.org/zip-motw-bug-analysis/)

## 最佳实践

待补充

# 1.2 文件上传下载

## 常见利用手法和案例

文件上传

* 文件目录遍历
  * [案例1:【web】使用类似filename=../bd.php的方法，跳出原始上传目录](https://ssd-disclosure.com/index.php/archives/3814)
  * [案例2:【web】vmware vRealize使用python开发，上传的文件名使用os.path.join拼接，当文件名为绝对路径时候会忽略前缀参数，可以往任意路径写入python webshell；另一个漏洞是应用更新流程可以自定义repo地址，并注入preInstall脚本，并通过将validateSignature设置为false来绕过脚本合法性检查，最终实现RCE](https://swarm.ptsecurity.com/hunting-for-bugs-in-vmware-view-planner-and-vrealize-business-for-cloud/)  
* 绕过文件名称检查
  * 尝试使用其他支持的扩展名、切换大小写或者增加关键词，比如 `a.phP`, `a.cer`，`a.jpg.jsp`
  * 尝试截断文件名，通用的有`a.asp%00.jpg`；Windows下可以是`a.asp.`，`a.asp%20`，或者以`%80-%99`结尾
  * 尝试写入NTFS流，比如`xx.php::$data`，`xx.php::$DATA`
* 绕过文件内容检查
  * 增加文件头，比如`GIF89a`
  * 使用短标签，比如`<?php`被过滤，可以改用`<?`
  * 去掉或清空请求参数，比如`?type=image`，检查是否可以成功上传webshell
* 尝试解析漏洞，可以根据`Content-Type`判断是否存在漏洞，利用上传的文件去执行代码
  * IIS 6 `a.asp;.jpg`，`a.jpg/x.php`
  * fastcgi `http://example.com/robots.txt/x.php`
* 利用zip解压缩功能写木马文件，可以用软连接、`../`跳出目录，也可以在三级以上目录存放文件绕过检测，或者利用条件竞争执行任意代码
  * [案例1: mozilla网站在解压缩zip文件时有一个条件竞争问题，可以暂时访问到上传后的PHP文件，造成任意代码执行](https://blog.assetnote.io/bug-bounty/2019/03/19/rce-on-mozilla-zero-day-webpagetest/)  
  * [案例2: CVE-2016-6321案例，构造带有`../`的文件名，实现路径穿越](https://sintonen.fi/advisories/tar-extract-pathname-bypass.proper.txt)
* 绕过上传后缀限制
  * 上传.htaccess绕过脚本限制，参考[Utilizing .htaccess for exploitation purposes — PART #1](https://medium.com/@insecurity_92477/utilizing-htaccess-for-exploitation-purposes-part-1-5733dd7fc8eb)
    * RewriteRule 实现内网代理
    * AddType 让非脚本文件可执行
    * SetHandler server-status 查看服务器信息
    * rewrite 到 file:/// 实现任意文件下载
  * 上传jar包绕过jsp/jspx限制，将jar包上传到`WEB-INF/lib`目录，修改或者覆盖`WEB-INF/web.xml`触发变更，之后jar包`META-INF/resources`目录下的JSP文件可以直接解析和访问，具体参考[记一次过滤jsp的文件上传](https://mp.weixin.qq.com/s/AhzA37S7ENCPUK5shDuDqg)

文件下载

* 修改下载的文件名实现恶意附件投递
  * [案例1: Spring MVC/WebFlux生成的Content-Disposition头，文件名可以用双引号屏蔽掉原有扩展名，比如将`"a.txt"`变成`"a.cmd".txt`，这样就可以让用户下载到cmd后缀的文件](https://securitylab.github.com/research/rfd-spring-mvc-CVE-2020-5398/)

## 最佳实践

待补充

# 1.3 文件包含

## 常见利用手法和案例

PHP利用

* 在能控制SESSION参数的情况下，可以尝试包含session文件，默认文件名为`/tmp/sess_XXXXX`
  * [案例1: mozilla靶场测试环境](https://www.rcesecurity.com/2017/08/upgrade-from-lfi-to-rce-via-php-sessions/)
* POST一个很大的文件到`phpinfo()`页面，可以看到临时文件名，在一定条件下可以尝试包含这个文件
* 如果能上传zip，可以用`zip:///a/b/c.zip#fileName`来包含文件，这种方式不需要截断即可包含文件

## 最佳实践

待补充
