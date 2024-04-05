# 说明

本章节包含登陆、认证、鉴权、验证码、风控等多个方面的内容。

* [1.1 密码校验](#11-密码校验)
  * [常见利用手法和案例](#常见利用手法和案例)
  * [最佳实践](#最佳实践)
* [1.2 密码重置](#12-密码重置)
  * [常见利用手法和案例](#常见利用手法和案例-1)
* [1.3 OAuth实现错误](#13-oauth实现错误)
  * [常见利用手法和案例](#常见利用手法和案例-2)
* [2. 用户注册](#2-用户注册)
  * [常见利用手法和案例](#常见利用手法和案例-3)
* [3. 验证码安全](#3-验证码安全)
  * [常见利用手法和案例](#常见利用手法和案例-4)

# 1.1 密码校验

## 常见利用手法和案例

暴力破解

* 使用系统初始化密码登陆
* 使用常见用户名+TOP弱口令喷洒
* 使用万能密码登陆(SQL注入、HASH传递)
  * [案例1:【web】Zipato SmartHubs登陆接口通过token认证，token计算公式为 SHA1(nonce + SHA1(password))，因此只需要知道SHA1(password)就可以登陆，存在PTH问题](https://blackmarble.sh/zipato-smart-hub/)
* 错误的密码校验逻辑
  * [案例1:【iot】TotalFlow认证密码是4位，而实际认证通信只传递CRC-16校验和。校验和只有2位，且服务端也没有请求限速。如果直接穷举密码需要65536次请求，而穷举校验和只需要256次请求](https://claroty.com/team82/research/an-oil-and-gas-weak-spot-flow-computers)
  * [案例2:【iot】dlink telnet实现了防止密码爆破的功能，当密码正确时立即返回Welcome，错误时等待3秒再返回Incorrect字样。然而攻击者可以只等待0.05s，如果没有收到Welcome字样就认为密码是错误的，导致防止爆破功能失效](https://blog.whtaguy.com/2021/05/d-link-router-cve-2021-27342.html)
* 账号密码错误时，明确返回账号不存在字样，可以枚举账号
  * [案例1: 某企业邮箱登陆失败时候，会返回用户不存在或者密码错误，可以根据错误来判断账号是否存在](http://wy.zone.ci/bug_detail.php?wybug_id=wooyun-2015-0145012)

认证绕过

* 如果只是前端校验，修改登录接口的返回结果，看是否可以进入下一界面
* 服务端只判断了认证cookie是否存在
* 服务端使用cookie里的字段判断权限，且可以篡改，如用户ID、用户角色
* 服务端认证信息没有绑定用户，比如A用户的短信验证码可以登录B用户
* 未初始化的参数导致校验失败
  * [案例1:【iot】Wyze摄像头认证时，客户端先发出指令A，之后服务端会初始化随机字符串，加密后发给客户端。客户端若能解密，则调用指令B，提交解密后的内容来完成认证。然而，如果直接发送指令B，此时服务端随机字符串还没有初始化，因此服务端是对比NULL是否等于NULL，就直接通过了认证](https://www.bitdefender.com/files/News/CaseStudies/study/413/Bitdefender-PR-Whitepaper-WCam-creat5991-en-EN.pdf)
  * [案例2:【web】checkmk认证秘钥通过secret和其他已知参数计算，但是当secret文件不存在，PHP程序不会报错，只会返回空字符串。通过任意文件删除漏洞不断删除secret文件，利用条件竞争可以绕过认证](https://blog.sonarsource.com/checkmk-rce-chain-2/)
  * [案例3:【iot】dlink 在判断密码时使用了strncmp()。当要比较的第二个字符串为空，strcmp也会返回0，导致认证绕过](https://www.thezdi.com/blog/2020/9/30/the-anatomy-of-a-bug-door-dissecting-two-d-link-router-authentication-bypasses)
* 尝试绕过2FA验证
  * [案例1: 通过接口调用获取backup codes，绕过TOTP验证](http://c0d3g33k.blogspot.com.au/2018/02/how-i-bypassed-2-factor-authentication.html)
* 第三方认证利用
  * [案例1: 某邮箱可直接修改其他用户密码，手机号绑定界面可以修改UID参数，实现为任意用户绑定自己的手机号，并通过密码重置接管账号](https://wy.zone.ci/bug_detail.php?wybug_id=wooyun-2012-08307)
  * [案例2: 某网站检测到登录来源是twitter，就暂时允许用户在未订阅服务的情况下查看文章。将User-Agent改为Twitter App的、Referer头改成t.co，就可以实现未订阅访问文章](https://elaineou.com/2017/01/19/how-the-twitter-app-bypasses-paywalls/)
* 快捷登录，比如某些场景下通过IMSI、手机号就可以完成认证(可以用xposed修改手机号)

其他参考

* [我的通行你的证 - 呆子不开口](https://wooyun.js.org/drops/%E6%88%91%E7%9A%84%E9%80%9A%E8%A1%8C%E4%BD%A0%E7%9A%84%E8%AF%81.html)

## 最佳实践

待补充

# 1.2 密码重置

## 常见利用手法和案例

伪造凭据

* 输入手机号等内容，看页面是否返回邮箱等其他信息
* 检查找回密码凭据是否在前端可见，比如在响应里返回、前端生成、或者存放到Cookie里
* 检查找回密码凭据是否为简单的base64编码、MD5加密，尝试伪造凭据
* 篡改接收邮件的地址
  * [案例1: 某网站重置密码接口存在漏洞，Cookie里返回了PHP序列化数据，且服务端从序列化数据里读取邮箱，可设置为自己的收件地址](https://wy.zone.ci/bug_detail.php?wybug_id=wooyun-2015-0130081)
  * [案例2: wordpress发送重置密码邮件，会使用当前的HOST头来设置From头和退信地址，对于单IP无虚拟主机场景，可以绑定HOST头请求，可以收到这个重置密码的邮件](https://exploitbox.io/vuln/WordPress-Exploit-4-7-Unauth-Password-Reset-0day-CVE-2017-8295.html)
* 攻击发送邮件的服务器，比如弱口令、或者登录密码在github存放
  * [案例1: 某网站发送找回密码的邮箱是个外部邮箱，而且密码是弱口令，登录后就可以看到重置密码链接](https://wy.2k8.org/bug_detail.php?wybug_id=wooyun-2013-029118)
* 对于有秘钥的场景，检查秘钥是否彻底更换
  * [案例1: Bitwarden密码管理器修改主密码后，只是重新加密私钥，而不是重新生成私钥](https://cdn.bitwarden.net/misc/Bitwarden%20Security%20Assessment%20Report%20-%20v2.pdf)  
* 找回密码分很多步骤，可以直接进入下一步
* 在找回密码的最后一步，如果有同时提交用户名和密码，是否可以修改其他人的密码？
* 服务器支持发邮件的方式修改密码，可以伪造来源地址

其他参考

* [10 Password Reset Flaws](https://anugrahsr.github.io/posts/10-Password-reset-flaws/)

# 1.3 OAuth实现错误

## 常见利用手法和案例

* 尝试窃取access token，接管用户账号
  * [案例1:【web】用户已经登录状态下，Airbnb OAuth接口会根据Referer跳转回原始页面，让攻击者可以获取到token](https://www.arneswinnen.net/2017/06/authentication-bypass-on-airbnb-via-oauth-tokens-theft/)
  * 使用有302跳转漏洞的页面，绕过微信OAuth认证的域名白名单
  * [案例2:【web】Google Cloud Workstations没有用redirect参数，而是把跳转地址放到state这个JSON参数里了，可以伪造后发送给用户点击，然后获取到token](https://blog.stazot.com/auth-bypass-in-google-cloud-workstations/)
  * [案例3:【web】Periscope TV 在调用twitter登录时，如果将Host头设置为`attacker.com/www.periscope.tv`，也可以获取到合法的twitter OAuth跳转链接。发送给用户点击后，token会发送到`attacker.com`，实现token窃取](https://hackerone.com/reports/317476)
  * [案例4:【web】uber facebook登录接口允许跳转到`https://auth.uber.com/login?`开头的URL，这个接口支持一个next_url参数，可以继续跳转到任意地址。由于`login.uber.com/logout`是按照referer跳转，因此只要从攻击者页面发起facebook登录，最终就会带着token跳回攻击者的页面，就可以获取到token](https://hackerone.com/reports/202781)
* 检查oauth callback的实现是否有错误
  * [案例1:【web】Tinder使用Facebook AccountKit进行登录，但是未验证clientid和token的对应关系，可以登录任意绑定了facebook登录的账号](https://medium.com/@appsecure/hacking-tinder-accounts-using-facebook-accountkit-d5cc813340d1)
  * [案例2:【web】gcloud IAP网关支持OAuth认证，攻击者构建了一个钓鱼网站，使用相同的client_id + 错误的client_secret依然可以获取到正确的redirect_token，导致凭证泄露问题](https://www.seblu.de/2021/12/iap-bypass.html)
* 检查oauth权限检查是否足够
  * [案例1:【web】instagram oauth返回的access token权限过大，恶意的第三方程序可以用这个token执行非预期的操作](https://ysamm.com/?p=684)

# 2. 用户注册

## 常见利用手法和案例

逻辑漏洞

* 当前端不允许提交(比如按钮是灰色)，尝试直接POST表单数据，看是否可以成功
* 通过接口注册高权限账号
* 系统不允许注册账号，通过接口开启注册功能
  * [案例1:【web】wordpress gpdr插件可以通过接口开启注册功能，且可以注册管理员](https://www.wordfence.com/blog/2018/11/trends-following-vulnerability-in-wp-gdpr-compliance-plugin/)
* 在注册步骤进行到一半时，打开登录页面（带有账号ID的那种），可能会直接登录成功
* 邮箱校验绕过
  * [案例1: 注册@google.com 邮箱，在最后一步邮箱验证时填写其他邮箱导致filter绕过](https://medium.com/@alex.birsan/messing-with-the-google-buganizer-system-for-15-600-in-bounties-58f86cc9f9a5)
  * [案例2: fs0c131y@protonmail.com@elysee.fr 无法注册，但是 fs0c131y@protonmail.com@presidence@elysee.fr 成功了，最后发现是python内置的email.utils.parseaddr发邮件时会忽略第二个@之后的内容，导致认证通过](https://medium.com/@fs0c131y/tchap-the-super-not-secure-app-of-the-french-government-84b31517d144?sk=59e15e44ba75dd78d7248262a4c8f0b7)

Web漏洞

* 尝试在用户名里插入特殊字符，比如 `\` 和 `'`，看是否可以造成二次注入
* 尝试利用IIS 6.X解析漏洞，比如注册用户名设置为`xx.asp`，头像上传路径是`/UploadFiles/用户名/文件名`，则或许可以利用

# 3. 验证码安全

## 常见利用手法和案例

* 如果不传递验证码这个参数，或者验证码留空，检查验证码校验逻辑是否失效
* 检查验证码是否可以爆破，比如SESSION不变，验证码是否就不变
  * [案例1: 多种姿势秒伪造任意号码拨打电话，只需要爆破4位验证码](https://wy.zone.ci/bug_detail.php?wybug_id=wooyun-2015-0131650)
* 检查验证码是否前端可见，比如嵌入到HTML里、或者在响应里、Cookie里面
* 修改请求参数让验证码强度降级
  * [案例1: 某网站新版选择图片验证码存在逻辑漏洞，可以降级为老的4位验证码](https://wy.zone.ci/bug_detail.php?wybug_id=wooyun-2015-0103427)
