# 说明

本章节收录密码相关案例

* [1.1 密码算法](#11-密码算法)
  * [常见利用手法和案例](#常见利用手法和案例)
  * [最佳实践](#最佳实践)
* [1.2 密码存储](#12-密码存储)
  * [常见利用手法和案例](#常见利用手法和案例-1)
  * [最佳实践](#最佳实践-1)

# 1.1 密码算法

## 常见利用手法和案例

随机数

* 随机范围太小
  * [案例1:【web】pearweb在生成密码重置token时候，使用`md5(mt_rand(4, 13) + time() + 新密码 + 用户名)`来作为种子，穷举范围很小可以爆破](https://blog.sonarsource.com/php-supply-chain-attack-on-pear)

## 最佳实践

待补充

# 1.2 密码存储

## 常见利用手法和案例

硬编码问题

* 提取程序里的秘钥
  * [案例1:【windows】FortifyClient 使用写死的解密秘钥，导致密码可被解密](http://seclists.org/fulldisclosure/2017/Dec/43)
  * [案例2:【windows】某客户端在登陆时，直接连接远程MSSQL查询所有账号密码，然后本地判断密码是否正确。导致MSSQL账密泄漏、SQL注入、明文流量等多个问题](http://www.freebuf.com/articles/system/143513.html)
  * [案例3:【iot】Ichano摄像头设备在JS登陆函数里硬编码用户名和密码，在telnet启动程序里使用写死的密码](https://blogs.securiteam.com/index.php/archives/3576)

## 最佳实践

待补充

# 1.3 证书安全

## 常见利用手法和案例

* [案例1: thunderbird会自动导入S/MIME证书，通过替换证书可以实现中间人攻击](https://bugzilla.mozilla.org/show_bug.cgi?id=1438949)
* [案例2: CVE-2021-3162 Docker XPC服务提权案例，SMJobBless根据info.plist里配置的签名要求去校验客户端，而docker程序没有配置`anchor apple generic`字样，只是检查CN字段。因此伪造签名证书就可以调用XPC，实现Mac下面的提权](https://wojciechregula.blog/post/when-vulnerable-library-is-actually-your-physical-book/)

## 最佳实践

待补充
