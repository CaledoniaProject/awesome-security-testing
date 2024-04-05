# 说明

本章节收录虚拟终端的案例

# 1.1 字符流安全

## 常见利用手法和案例

* 查看文件产生特殊字符流导致RCE
  * [案例1: xterm before patch 375 can enable an RCE under certain conditions](https://www.openwall.com/lists/oss-security/2022/11/10/1)
  * [案例2: From Terminal Output to Arbitrary Remote Code Execution](https://blog.solidsnail.com/posts/2023-08-28-iterm2-rce)
  * [案例3: Issue 2154: Terminal escape injection in AWS CloudShell](https://bugs.chromium.org/p/project-zero/issues/detail?id=2154)
* 其他  
  * [案例1: 利用escape sequence移动光标，让cat看到的内容被覆盖](https://twitter.com/0xasm0d3us/status/1774534241084445020)

## 最佳实践

待补充
