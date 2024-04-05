# 说明

# 1. 敏感信息泄露

## 常见利用手法和案例

流量泄露

* [案例1:【android】广告SDK使用HTTP协议上报用户地理位置、设备指纹等信息，可被截获](https://securelist.com/leaking-ads/85239/)

配置泄露

* 如果配置了redis session handler，phpinfo里可以看到save_path配置，可能会有redis认证密码

日志泄露

* [案例1:【iot】Victure VD300提供了3个web调试接口，开启日志、下载日志、关闭日志。由于日志里包含了Wifi密码，导致敏感信息泄漏](https://research.nccgroup.com/2020/12/18/domestic-iot-nightmares-smart-doorbells/)
* [案例2:【iot】Swann重置出厂状态时，虽然删除了wpa_supplicant.conf，但是日志里依然有SSID和PSK，导致信息泄露](https://www.pentestpartners.com/security-blog/hacking-swann-home-security-camera-video/)

用户隐私泄露

* [案例1:【web】某网站泄露用户发帖IP](https://wy.zone.ci/bug_detail.php?wybug_id=wooyun-2011-02705)

内存泄露

* [案例1:【iot】Citrix Bleed案例，openid接口根据主机名等信息，调用snprintf构造JSON数据然后返回，发送的长度由snprintf返回值决定。然而snprintf返回值是“如果buffer足够大时，当前字符串长度是多少”，因此可以越界读取内存，比如读取管理员会话cookie等等](https://www.assetnote.io/resources/research/citrix-bleed-leaking-session-tokens-with-cve-2023-4966)

## 最佳实践

待补充
