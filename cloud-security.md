# 说明

本章节收录云环境和云服务商常见漏洞

* [1.1 VPC安全](#11-vpc安全)
  * [常见利用手法和案例](#常见利用手法和案例)
  * [最佳实践](#最佳实践)
* [1.2 容器环境](#12-容器环境)
  * [常见利用手法和案例](#常见利用手法和案例-1)
  * [最佳实践](#最佳实践-1)
* [1.3 数据库](#13-数据库)
  * [常见利用手法和案例](#常见利用手法和案例-2)
  * [最佳实践](#最佳实践-2)

# 1.1 VPC安全

## 常见利用手法和案例

* [【aws】攻击者在获取云上最高权限后，通过创建VPC网络，可以伪造请求来源，让敏感操作的来源合法](https://www.hunters.security/en/blog/hunters-research-detecting-obfuscated-attacker-ip-in-aws)

## 最佳实践

待补充

# 1.2 容器环境

## 常见利用手法和案例

* [【k8s】azure实现了一个bridge node来转发az exec请求，每次请求都会带上service token。利用runc漏洞逃逸到node节点后，通过mitmproxy劫持到node的请求，即可获取这个token。检查token后，发现是cluster-admin角色，至此整个集群沦陷](https://unit42.paloaltonetworks.com/azure-container-instances/)

## 最佳实践

待补充

# 1.3 数据库

## 常见利用手法和案例

* [【cloud-pgsql】自定义index函数实现命令执行，替换root会执行的文件实现提权，最后利用CAP_ADMIN逃逸](https://www.wiz.io/blog/the-cloud-has-an-isolation-problem-postgresql-vulnerabilities)

## 最佳实践

待补充
