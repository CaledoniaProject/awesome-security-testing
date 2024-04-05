# 说明

本文档收录数据库安全案例

# 1.1 未分类

* [案例1: MySQL不会执行`/*!99999 id = XXXX*/`里面的内容，但是MaxScale会处理，导致过滤器绕过](https://www.4hou.com/system/16886.html)
* [案例2: cockpit使用了PHP MongoLite库。在操作MongoDB时候没有强制转换string类型，导致NoSQL注入。这个库还支持传入`$func、$fn、$f`来执行var_dump这样的函数，可以打印数据库里的内容](https://swarm.ptsecurity.com/rce-cockpit-cms/)
* [案例3: mysqljs库的prepare查询，是自己实现的escape，在处理object类型的时候会转换为\`key\`='value'`的形式，导致SQL注入问题。所以，prepare查询并不是绝对安全的](https://flattsecurity.medium.com/finding-an-unseen-sql-injection-by-bypassing-escape-functions-in-mysqljs-mysql-90b27f6542b4)
* [案例4: typo3不允许设置orderByAllowed的值，但是因为后面实际是转换成setOrderByAllowed函数去调用，所以可以用大写的OrderByAllowed绕过，实现SQL注入](https://www.ambionics.io/blog/typo3-news-module-sqli)
