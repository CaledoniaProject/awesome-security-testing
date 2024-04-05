# 说明

# 1.1 HTTP请求

## 常见利用手法和案例

* 普通SSRF案例
  * [案例1:【web】Escalating SSRF to RCE - 利用 aws meta 获取 accesskey，然后S3上传webshell获取权限](https://generaleg0x01.com/2019/03/10/escalating-ssrf-to-rce/)
  * [案例2:【web】Google Data Publishing Language允许从远程HTTP/FTP获取数据，存在SSRF](https://s1gnalcha0s.github.io/dspl/2018/03/07/Stored-XSS-and-SSRF-Google.html)
  * [案例3:【web】Chrome插件Data Saver允许访问Google内部服务，安装插件就可以实现SSRF](http://www.freebuf.com/vuls/164831.html)
  * [案例4:【web】fastmail这个SSRF可以填写`\r\n`，注入自定义header，测试SSRF需要同时看看是否有CRLF问题](https://infosecwriteups.com/bug-bounty-fastmail-feeda67905f5?gi=c70db9c1f831)
* 高级跳转利用
  * [案例1:【web】vimeo某个接口存在SSRF，但是利用参数为URL PATH，结合同域名的任意重定向漏洞实现了SSRF。最后通过metadata.google.internal实现RCE](https://medium.com/@rootxharsh_90844/vimeo-ssrf-with-code-execution-potential-68c774ba7c1e)
  * [案例2:【web】接口有169.254.169.254、localhost过滤，使用303跳转绕过，注意不是301、302](https://medium.com/techfenix/ssrf-server-side-request-forgery-worth-4913-my-highest-bounty-ever-7d733bb368cb)
* 利用PDF渲染功能，比如office插入OLE对象、HTML插入iframe标签等方式实现SSRF
  * [案例1:【web】某web程序会基于用户信息生成PDF。程序是先生成HTML再转换PDF，由于没有过滤iframe标签，也没有限制网络请求，可以实现有回显的 SSRF。结合aws metadata可以实现RCE](https://blog.appsecco.com/finding-ssrf-via-html-injection-inside-a-pdf-file-on-aws-ec2-214cc5ec5d90)
* 请求走私利用
  * [案例1:【web】通过Keep-Alive发送第二个请求，使得POST类型的SSRF接口可以发送PUT请求](http://www.kernelpicnic.net/2017/05/29/Pivoting-from-blind-SSRF-to-RCE-with-Hashicorp-Consul.html)
* 跨域利用
  * [案例1:【web】semrush服务器存在CORS配置错误问题。在请求接口时同时传入Origin头，服务器则会根据Origin生成Access-Control-Allow-Origin，相当于配置了`Access-Control-Allow-Origin: *`](https://hackerone.com/reports/235200)

## 最佳实践

待补充

# 1.2 CDN利用

## 常见利用手法和案例

域前置

1. [案例1: azure租户可以使用msn.com的CDN服务器 + 自己的hostname实现域前置](https://theobsidiantower.com/2017/07/24/d0a7cfceedc42bdf3a36f2926bd52863ef28befc.html)
2. [案例2: AppEngine domain fronting: 使用Google的IP，访问GAE的服务](https://www.cyberark.com/threat-research-blog/red-team-insights-https-domain-fronting-google-hosts-using-cobalt-strike/)

## 最佳实践

待补充
