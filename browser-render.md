# 说明

本章节收录浏览器漏洞利用案例

# 1.1 浏览器扩展

## 常见利用手法和案例

* 特权域利用
  * [案例1:【浏览器】Avast浏览器内置的扩展权限很高，而且CSP策略是unsafe-eval，当URL包含screen.yahoo.com字样时会以浏览器扩展的身份执行JS。Avast浏览器接口增加了SWITCH_TO_SAFEZONE命令，可以通过执行命令打开新的网页，通过附加`--utility-cmd-prefix=calc.exe`参数可以实现命令注入](https://palant.info/2020/01/13/pwning-avast-secure-browser-for-fun-and-profit/#going-beyond-the-browser)
  * [案例2: zenmate chrome扩展允许在信任的域名下面获取用户信息，购买这些过期域名可泄露匿名用户信息](https://thehackerblog.com/zenmate-vpn-browser-extension-deanonymization-hijacking-vulnerability-3-5-million-affected-users/index.html)
  * [案例3: 小米浏览器给JS提供了多个接口，其中miui.share可以下载文件到本地，market.install可以用于安装app，最后利用intent打开目标app，实现全自动下载安装和执行](https://labs.withsecure.com/advisories/xiaomi)

## 最佳实践

待补充

# 1.2 HTML渲染

渲染问题

* 利用自定义标签读取任意文件
  * [案例1:【java】pd4ml库可以转换HTML为PDF，可以通过 `<pd4ml:attachment src="file:///etc/passwd"><pd4ml:attachment>` 实现任意文件读取，并导出到PDF里](https://medium.com/bugbountywriteup/how-i-hacked-redbus-an-online-bus-ticketing-application-24ef5bb083cd)
* 利用浏览器渲染特性，比如插入`<frame>/<iframe>`标签、`通过JS执行代码`等方式获取敏感信息
  * [案例1: 某个HTML转PDF的业务，通过在HTML里插入`<script>window.location="http://169.254.169.254/latest/meta-data/iam/security-credentials/"</script>`可以读取到敏感文件](https://www.ionize.com.au/post/stealing-amazon-ec2-keys-via-an-xss-vulnerability)
  * [案例2: phantomjs渲染案例，通过XSS读取`file:///`的内容](https://buer.haus/2017/06/29/escalating-xss-in-phantomjs-image-rendering-to-ssrflocal-file-read/)
  * [案例3: wkhtmltopdf渲染案例，利用方式同上](http://www.noob.ninja/2017/11/local-file-read-via-xss-in-dynamically.html)

# 1.3 无头浏览器利用

## 常见利用手法和案例

* [案例1: 先遍历端口探测无头浏览器监听端口，然后利用无头浏览器修改chrome默认下载目录，再用JS生成blob去生成文件下载，相同路径只能执行一次。作者攻击burp的方法是写一个user.vmoptions文件+javaagent实现RCE](https://hackerone.com/reports/1274695)

## 最佳实践

待补充