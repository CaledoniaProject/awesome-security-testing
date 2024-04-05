# 说明

本章节收录代码和命令执行漏洞

* [1.1 命令执行](#11-命令执行)
  * [常见利用手法和案例](#常见利用手法和案例)
  * [最佳实践](#最佳实践)
* [1.2 DLL加载](#12-dll加载)
  * [常见利用手法和案例](#常见利用手法和案例-1)
  * [最佳实践](#最佳实践-1)

# 1.1 命令执行

## 常见利用手法和案例

通用漏洞

* open函数利用
  * [案例1:【ruby】NET::Ftp模块在打开文件时用的`open`函数，而不是`File.open`，通过传入`| os command`可执行任意命令](https://hackerone.com/reports/294462)
    * open(localfile, "w")
  * [案例2:【perl】Sophos UTM是perl实现的，调用`open`函数时，文件名可被用户控制，也有`| os command`的问题](https://www.atredis.com/blog/2021/8/18/sophos-utm-cve-2020-25223)
    * open my $fh, $ARGV[0] or die $!;
* 参数注入
  * [案例1:【iot】Moxa AWK-3131A在执行登陆操作时，会调用命令行程序验证账号密码。这里没有使用execve，而是直接使用system执行，造成命令注入](https://talosintelligence.com/vulnerability_reports/TALOS-2017-0507)
    * sprintf(buf, "/usr/sbin/iw_event_user %s %s %s", IW_LOG_AUTH_FAIL); 
    * system(buf)
  * [案例2: TexLive支持`repstopdf [options] [epsfile [pdffile.pdf]]`指令来生成PDF。repstopdf是一个perl脚本，通过执行ghostscript命令来生成PDF](https://paper.seebug.org/1596/)
    * repstopdf在执行命令时为了提升安全性，增加了`--safer`参数，并过滤了`--gsopt/--gsopts/--gscmd`等危险参数
    * 但由于PDF文件名参数epsfile会被放到`-sOutputFile=XXX`里，且这里支持使用管道，e.g `repstopdf '|id #'`，最终导致命令注入漏洞
  * [案例3:【iot】GoAhead set_ftp.cgi接口先生成一个shell脚本再执行。用户的输入在`/system/system/bin/ftp -n<<EOF`与`EOF`之间，由于没有使用`'EOF'`，所以会解析变量，存在命令注入问题](https://paper.seebug.org/252/)
  * [案例4:【iot】SSL VPN调用tcpdump时存在命令注入，最后注入`-r`参数让tcpdump报错，并重定向stderr生成perl webshell，实现漏洞利用。内容在PPT 91-95页](https://gsec.hitb.org/materials/sg2019/D1%20-%20Infiltrating%20Corporate%20Intranets%20Like%20The%20NSA%20-%20A%20Pre-Auth%20Remote%20Code%20Execution%20on%20Leading%20SSL%20VPNs%20-%20Orange%20Tsai%20&%20Tingyi%20Chan.pdf)
    * 利用限制
      * 过滤了 `&|*()..` 等各种特殊字符
      * tcpdump版本太低，不支持`-z`参数
    * 技巧  
      * perl支持标签语法，e.g `error: print 0x41`
      * 最终payload是`-r 'print 123#' 2>/xxxx/setcookie.html.ttc`
  * [案例5:【linux】kibana nodejs原型链污染 + 通用的 child_process.spawn env污染实现RCE](https://slides.com/securitymb/prototype-pollution-in-kibana/)
    1. timelion页面可以控制Object的原型链，并污染env的值
    2. canvas页面会调用child_process.spawn()启动新的node进程
    3. spawn方法会将options.env透传给子进程，由于前面通过原型链控制了NODE_OPTIONS和AAA两个环境变量，可以实现RCE
  * [案例6:【mac】hdiutil可以生成带有换行符的磁盘名，timemachinehelper会拼接这个名字，并传递给awk执行。通过换行符+$()可以实现命令注入](https://objective-see.org/blog/blog_0x40.html)
  * [案例7:【iot】IOT设备扫描SSID、或者连接时，会调用底层命令，SSID、加密方式等字段都有可能造成命令注入 - 详情在第55页](http://files.brucon.org/2018/18-Lilith-Wyatt-IoT-RCE.ppt)
  * [案例8:【android】EMUI保存主题时候会先用`renameTo()`，失败了再执行`mv`命令。命令注入时不能出现`/`，用`${ANDROID_DATA%data}`构造一个就可以了](https://zhuanlan.zhihu.com/p/24983092)
  * [案例9: Evinceh会拼接文件名到tar解压缩命令里，在文件名里加上`--checkpoint-action`字样可以实现命令注入](https://bugzilla.gnome.org/show_bug.cgi?id=784630)
  * [案例10: NagiosXI错误的使用了escapeshellcmd去转义命令行参数，可以注入不带引号的参数，通过注入`-o XXX`到curl命令，可以实现getshell](https://medium.com/tenable-techblog/rooting-nagios-via-outdated-libraries-bb79427172)
* 下游命令执行
  * [案例1: Centreon V20.04后台getshell，PHP程序上传mib本身没有漏洞，但是centFillTrapDB会调用snmptranslate，把每一行拿出来进行翻译，这里可以用反引号实现命令执行，获取后台权限](https://paper.seebug.org/1313/)
* 配置校验不足
  * [案例1:【windows】BitDefender安装包从自身文件名里获取base64编码后的XML URL地址（e.g `setupdownloader_dGVzdA==.exe`），然后根据配置里的run_command配置执行任意命令。由于安装包没有校验XML的合法性，导致任意命令执行](https://labs.nettitude.com/blog/cve-2018-8955-bitdefender-gravityzone-arbitrary-code-execution/)
* 基于chrome的浏览器利用
  * [案例1:【windows】amazon workspace注册了自定义URL处理器，可以通过构造URL参数可以实现命令注入，e.g`workspaces://anything%20--gpu-launcher=%22calc.exe%22@REGISTRATION_CODE`，最终参数会传递给chrome执行](https://rhinosecuritylabs.com/aws/cve-2021-38112-aws-workspaces-rce/)

git利用

* [案例1: 使用`--output=/XXX`写文件，比如github企业版可以在`env.d`创建一个新的启动脚本](https://devcraft.io/2020/10/18/github-rce-git-inject.html)
* [案例2: 使用`--no-index`跟源码目录以外的文件进行对比，通过读取diff结果，可以实现任意文本文件读取](https://mp.weixin.qq.com/s/8OSdYVTkv0J12ZKbLacITw)
* [案例3: git archive会调用`sh -c`，比如通过`--prefix xd --exec='echo pew#' --remote=file:///tmp/ -- blah`执行命令](https://blog.assetnote.io/2022/09/14/rce-in-bitbucket-server/)
* 案例4: git --upload-pack 注入
  * [git ls-remote --upload-pack='$(cp /etc/passwd def)' abc https://github.com/threedr3am/ZhouYu](https://justi.cz/security/2021/04/20/cocoapods-rce.html)
  * [git clone '-u$({open,-a,calculator}):x' './qwe/-u$({open,-a,calculator}):x'](https://blog.sonarsource.com/securing-developer-tools-argument-injection-in-vscode/)
* [案例5: git clone注入ssh`-oProxyCommand`参数实现RCE，可以在LFS或者submodule上触发漏洞，目前已经修复](http://blog.recurity-labs.com/2017-08-10/scm-vulns)  

Electron利用

* [案例1: VSCode 1.19.0~1.19.2默认带有`--inspect`启动，导致调试模式开启，可执行任意代码](https://codecolor.ist/2018/03/16/visual-studio-code-silently-fixed-a-remote-code-execution-vulnerability/)
* [案例2: CVE-2018-1000006案例，程序注册URL协议，比如`c:\a.exe "%1"`。explorer会调用ShellExecuteW执行，而且`%1`里面的双引号可以被闭合掉，然后实现参数注入、命令执行；ShellExecuteW使用域名+`..\`跳出目录的问题，在Windows 2022已经无法复现](https://paper.seebug.org/515/)
  * [类似案例: Line 闭合双引号，通过浏览器就能触发](https://blogs.securiteam.com/index.php/archives/3724)
  * [类似案例: mIRC 无需闭合双引号即可触发](https://github.com/proofofcalc/cve-2019-6453-poc)  

Mac案例

* [案例1: GitHub桌面应用支持clone+自动打开目录，如果目录名字是abc.app就相当于打开了任意的应用，造成RCE](https://pwning.re/2018/12/04/github-desktop-rce/)

## 最佳实践

待补充

# 1.2 DLL加载

## 常见利用手法和案例

* 根据配置加载DLL
  * [案例1:【通用】QT5 编译的程序默认支持一些列命令行参数，比如`-platformpluginpath`可以加载远程DLL](https://www.thezdi.com/blog/2019/4/3/loading-up-a-pair-of-qt-bugs-detailing-cve-2019-1636-and-cve-2019-6739)
  * [案例2:【linux】Ubuntu snap错误的将LD_LIBRARY_PATH设置为 ...:XXX::XXX:...，即多了一个空的路径。这导致程序会从CWD加载依赖的so，带来潜在的RCE问题](https://blog.ret2.io/2021/08/04/snapcraft-injection/)

## 最佳实践

待补充

