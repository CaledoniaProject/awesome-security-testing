# 1.1 账号风控

* [案例1: 某网站通过短信发送优惠券，同一个手机号有多种表示形式，可以绕过是否已领优惠券的判断](https://wooyun.x10sec.org/static/bugs/wooyun-2015-0125060.html)
  1. 区号，e.g `+8613010002000`、`8613010002000`
  2. 末尾特殊字符，e.g `13010002000!`
  3. 分号分割，e.g `130XXXXXXXX;13010002000` 或者 `13010002000;130XXXXXXX`
  4. 逗号分割，e.g `13010002000,13010002001`

# 1.2 邮件安全

* [案例1: Google公开的bug管理工具会自动生成`buganizer-system+componentID+issueID@google.com`的邮件组。由于任何人都可以向这个地址发邮件，所以可以用来接收验证码，相当于拥有了一个@google.com的邮箱。另外，在gmail可以将添加这个邮箱为"发送邮箱"。](https://medium.freecodecamp.org/i-bypassed-how-i-hacked-googles-bug-tracking-system-itself-for-15-600-in-bounties-here-s-how-3355c8c63955)

# 1.3 终端安全

* [案例1: 某EDR的钩子通过Tls来判断是否为重入，在调用被hook的函数前设置Tls标记位可以绕过EDR钩子](https://www.deepinstinct.com/blog/evading-antivirus-detection-with-inline-hooks)
* [案例2: slack支持slack://setting?update={JSON}来修改设置，可以将文件默认下载路径改为SMB路径，以窃取用户文件。通过attachment接口还可以自定义链接标题，避免用户察觉](https://medium.com/tenable-techblog/stealing-downloads-from-slack-users-be6829a55f63)

# 1.4 移动安全

* [案例1: 通过逆向APK分析出隐藏的组合键，然后用遥控器打开工程模式，可以安装apk等等](hhttps://delikely.eu.org/2021/05/15/%E6%99%BA%E8%83%BD%E7%94%B5%E8%A7%86%E6%BC%8F%E6%B4%9E%E6%8C%96%E6%8E%98%E5%88%9D%E6%8E%A2%E4%B9%8B%E5%AF%BB%E6%89%BE%E5%B7%A5%E7%A8%8B%E6%A8%A1%E5%BC%8F%E7%83%AD%E9%94%AE)

# 1.5 支付业务

## 常见利用手法和案例

* 篡改支付参数
  * [案例1: 后端校验了支付金额，但可以修改实际购买的套餐信息](https://wy.zone.ci/bug_detail.php?wybug_id=wooyun-2015-089074)
  * [案例2: 后端没有校验支付金额，产品个数设置为负数实现减免](http://www.freebuf.com/vuls/149251.html)
  * [案例3: 某些信用卡发行商，只提交有效期就可以支付，不需要CVV](http://127.0.0.1/tools/wooyun/view.php?id=wooyun-2016-0188155)
* 尝试条件竞争漏洞，比如同时发出多个提现请求，检查数据库是否锁表，能否重复提现
* 尝试突破HMAC签名计算，比如通过调整字段内容，实现不同的支付金额、相同的HMAC
* 其他总结
  * [支付宝微信Web支付漏洞总结 ](https://www.sohu.com/a/222198699_354899)

## 最佳实践

待补充
