# 说明

本章节收录流水线安全案例

# 1.1 通用

## 常见利用手法和案例

* 利用CI权限读取代码
  * [PaaS平台允许在maven打包的validate阶段执行代码，由于平台一般会用一个大账号去clone代码，这里可以用env获取变量，或者直接反弹shell等等都是可以的](https://mp.weixin.qq.com/s/g7V4zo1RIKD0QPMzrOEodg)

# 1.2 github workflow

## 常见利用手法和案例

* [Pull Request通常审批后才能合入。作者在提交PR时同时提交workflow脚本，并通过workflow调用API接口自动approve，实现审批绕过。修复方法是将审批人数量改为2个以上](https://medium.com/cider-sec/bypassing-required-reviews-using-github-actions-6e1b29135cc7)
* [github workflow 允许读取仓库文件，但是没有过滤软连接。通过上传/proc/self/environ的软连接，spell-check会提示这个文件语法错误，并打印文件内容，导致GITHUB_TOKEN泄露](https://github.com/justinsteven/advisories/blob/master/2021_github_actions_checkspelling_token_leak_via_advice_symlink.md)
* [github workflow的${{}}语法会调用bash执行命令。由于branch名字用户可控，当workflow写法有问题时，可以在CI运行环境执行任意命令](https://blog.ryotak.me/post/pypi-potential-remote-code-execution-en/)
  ```
  有漏洞的写法
  run: |
    echo "${{steps.fetch-branch-names.outputs.result}}"
  
  利用时传入的branch名字
  dependabot;cat$IFS$(echo$IFS'LmdpdA=='|base64$IFS'-d')/config|base64;sleep$IFS'10000';#
  ```

## 最佳实践

待补充

