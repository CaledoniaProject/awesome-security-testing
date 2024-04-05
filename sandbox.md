# 说明

本章节包含沙箱相关案例

# 1.1 逃逸问题

## 常见利用手法和案例

* 尝试逃逸
  * [案例1: play-with-docker网站挂载了`/dev`目录。虽然不能调用`mount`，但是`debugfs`可以读写磁盘，也能用`insmod`加载内核模块，导致RCE](https://www.cyberark.com/threat-research-blog/how-i-hacked-play-with-docker-and-remotely-ran-code-on-the-host/)
  * [案例2: 某开发平台会根据用户输入动态设置`open_basedir`，导致PHP沙箱突破](https://wy.zone.ci/bug_detail.php?wybug_id=wooyun-2011-03687)
* 利用软链接突破沙箱
  * [案例1: git里提交一个软连接，发布到某开发平台的时候会被解析，可以访问沙盒外的文件](https://wy.zone.ci/bug_detail.php?wybug_id=wooyun-2015-0131911)
  * [案例2: 某开发平台可以上传zip代码包部署，可以在zip里放一个软连接，解压缩后就被替换成了对应的文件](https://wy.zone.ci/bug_detail.php?wybug_id=wooyun-2014-086433)
* 利用通信协议突破沙箱
  * [案例1: 利用x11协议，使用xdotool模拟键盘输入，执行命令并绕过沙箱](https://bugs.chromium.org/p/project-zero/issues/detail?id=1293&desc=2)
* Office沙箱突破
  * [案例1: Mac Office的沙箱配置文件里，允许写名字为`~$XXXX`的文件，可以在LaunchAgents目录里创建plist结尾的文件，实现沙箱逃逸](https://www.mdsec.co.uk/2018/08/escaping-the-sandbox-microsoft-office-on-macos/)
  * [案例2: HWP office在调用ghostscript的时候没有指定`-dSAFER`参数，可以实现沙箱绕过](https://mp.weixin.qq.com/s/X8Iz26L1k5ibdqv3lKFHKg)

## 最佳实践

待补充
