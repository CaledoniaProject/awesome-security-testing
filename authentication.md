# 说明

本章节包含认证相关和错误的业务逻辑实现

* [1.1 认证绕过](#11-认证绕过)
  * [常见利用手法和案例](#常见利用手法和案例)
  * [最佳实践](#最佳实践)
* [1.2 路径校验缺陷](#12-路径校验缺陷)
  * [常见利用手法和案例](#常见利用手法和案例-1)
  * [最佳实践](#最佳实践-1)
* [1.3 内存安全校验缺陷](#13-内存安全校验缺陷)
  * [常见利用手法和案例](#常见利用手法和案例-2)
  * [最佳实践](#最佳实践-2)

# 1.1 认证绕过

## 常见利用手法和案例

未授权漏洞

* [案例1:【web】Dynamicweb安装完毕后，依然可以访问setup页面去增加管理员账号，登陆后台后，可以上传webshell实现RCE](https://blog.assetnote.io/2022/02/20/logicflaw-dynamicweb-rce/)
* [案例2:【android】ES文件管理器监听了0.0.0.0:59777端口，接口格式为JSON，支持文件上传、下载、列目录等等。由于没有做权限管理，而且可以在本机以外访问，导致认证失效问题](https://github.com/fs0c131y/ESFileExplorerOpenPortVuln)
* [案例3:【iot】Trustwave SWG 允许用户上传SSH key，而且接口没有认证](https://www.exploit-db.com/exploits/44047)
* [案例4:【android】FMMX1软件从/mnt/sdcard获取配置文件、解析服务器地址；更新服务器地址、执行指令的intent都没有保护，可以被其他APK调用；服务端返回的数据也没有校验。结合这几个漏洞可以实现mitm，控制手机](https://char49.com/tech-reports/fmmx1-report.pdf)
* [案例5:【web】对于开启缓存配置的路径，Cloudflare不会进行URL标准化，比如用户配置了`/share/`开启缓存，则`/share/%2F..%2Fapi/auth/session?cachebuster=123`会被CF全局缓存。当用户访问后，任何人再次访问就会看到一样的页面，实现敏感信息窃取](https://nokline.github.io/bugbounty/2024/02/04/ChatGPT-ATO.html)

源IP访问限制绕过

* 增加referer绕过访问限制
* 增加`X-forwarded-for`、`X-remote-IP`、`X-originating-IP`、`x-remote-addr`、`x-client-ip`等可能的请求头，并将源地址设置为127.0.0.1或者其他白名单IP来绕过认证
  * [案例1: Google API网关ESPV2允许通过`x-http-method-override`来修改请求头，由于修改后不会再执行JWT校验等认证逻辑，通过这个方式将PUT请求转换为POST可以实现认证绕过](https://securingbits.com/bypassing-google-cloud-api-gateway)

伪造认证信息

* [案例1: intune MDM准入信息可以伪造，这是个通用问题](https://o365blog.com/post/mdm/)
* [案例2: Moodle的PHP SESSION存储格式是在PHP序列化基础上用`|`做了分割，只要能控制一个变量，就可以篡改$_SESSION、实现会话控制。由于用户登出逻辑实现有问题，执行登出后最后一个$_SESSION对象内容没有销毁，此时带上cookie请求就可以以最后那个用户的身份登录](https://haxolot.com/posts/2022/moodle_pre_auth_shibboleth_rce_part2/)

越权漏洞

* [案例1:【web】yahoo notepad的笔记列表接口，传入用户名返回笔记列表。通过抓包发现用户名是加密的，e.g`fziy4wzxr41k4qwsgumu2v2qymynzat6kclqpwmc`，但实际上传入明文用户名也能返回结果，进一步测试发现这个接口可以越权](https://www.freebuf.com/vuls/200317.html)
* [案例2:【web】DeskPro可以未授权开启JWT认证，接口还会直接返回JWT secret，通过这个接口可以获取管理员身份，并进一步通过反序列化实现后台RCE](https://blog.redforce.io/attacking-helpdesks-part-1-rce-chain-on-deskpro-with-bitdefender-as-case-study/)

## 最佳实践

待补充

# 1.2 路径校验缺陷

## 常见利用手法和案例

错误的校验逻辑

* 更换请求方法绕过
  * [案例1:【iot】GE Healthcare只对GET请求进行身份验证，对于POST请求直接放行，导致认证绕过](https://www.atredis.com/blog/2018/5/14/ge-healthcare-mac-5500-vulnerabilities)
  * [案例2:【web】某公司JBoss只对GET进行basic认证，切换为HEAD方法可以绕过认证、部署war包](http://wy.zone.ci/bug_detail.php?wybug_id=wooyun-2015-0113808)
* 伪造URL参数绕过认证  
  * [案例1:【iot】GPON路由器在进行URL认证校验时，判断条件是包含`style/`、`script/`、`images/`之一就放行，在请求里增加`a=style/`即可绕过认证检查](https://paper.seebug.org/593/)
  * [案例2:【iot】arcadyan路由器不对`/images/`执行basic认证，通过传入`/images/..%2finfo.html`就可以绕过认证](https://medium.com/tenable-techblog/bypassing-authentication-on-arcadyan-routers-with-cve-2021-20090-and-rooting-some-buffalo-ea1dd30980c2)
* 绕过JWT校验
  * [案例1:【web】Satisfyer Toys服务端基于JWT认证，通过frida hook篡改客户端生成的JWT Token，可以登录其他账号，猜测是服务端只校验了JWT是否合法](https://bananamafia.dev/post/satisfyer/)
  * [案例2:【web】某些系统允许JWT算法设置为None，这样就不校验签名了](https://medium.com/@sajan.dhakate/exploiting-json-web-token-jwt-73d172b5bc02)
* Android构造包名绕过认证
  * [案例1:【android】Keeper/LastPass/Dashlane/1Password等应用可以自动填充应用密码，填充内容是根据当前包名`前缀`获取的，比如使用`com.facebook.evil`这样的包名就可以获取到存储的密码](http://www.s3.eurecom.fr/projects/modern-android-phishing/)
* 直接指定路径绕过，比如`/admin`禁止访问，`/admin/xxx.php`可能直接能访问

## 最佳实践

待补充

# 1.3 内存安全校验缺陷

## 常见利用手法和案例

不安全的共享内存

* [案例1: Synaptics创建另一个内存映射，通过winobj查看未设置ACL。检查内存的内容后，发现有一个地方会执行插件注册命令，通过构造内存结构可以实现任意命令执行](https://www.anquanke.com/post/id/90172)
* [案例2: Cylance防护程序通过IsEnabled()函数判断是否开启防护，是否开启存储在两个内存变量里。由于Cylance没有对内存修改做防护，启动另一个SYSTEM进行曲修改.NET managed heap就可以禁用防护](https://www.xorrior.com/You-Have-The-Right-to-Remain-Cylance/)

## 最佳实践

待补充
