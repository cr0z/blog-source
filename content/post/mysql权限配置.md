+++
date = "2017-09-07T15:55:24+08:00"
draft = false
title = "mysql权限配置"
tags = ["mysql"]

+++

查看用户root的权限:   
```SHOW GRANTS FOR root;```   

刷新用户权限:   
```FLUSH PRIVILEGES;```   

创建用户:   
```CREATE USER 'croz'@'localhost' IDENTIFIED BY '[YOURPASSWORD]'; ```   

赋予'croz'@'%'用户数据库'shzl'的所有权限:   
```GRANT ALL PRIVILEGES ON shzl.* TO 'croz'@'%';```   

赋予'croz'@'%'用户数据库'shzl'的SELECT权限:   
```GRANT SELECT ON shzl.* TO 'croz'@'%';```   
(其他权限替换```SLELECT```为```UPDATE```、```DELETE```等)    

移除'croz'@'%'用户对数据库'shzl'的所有权限:   
```REVOKE ALL PRIVILEGES ON shzl.* FROM 'croz'@'%';```   

移除'croz'@'%'用户对数据库'shzl'的SELECT权限:   
```REVOKE SELECT ON shzl.* FROM 'croz'@'%';```   

修改权限后刷新权限:   
```FLUSH PRIVILEGES```


host修改为 %后还是不能远程访问，错误码 111:     
错误原因:mysql启动时只绑定了本地地址,修改```/etc/mysql/my.cnf```,注释掉以下内容:   
```bind-address            = 127.0.0.1```   
(也可能my.cnf里只有两个include，就到include的目录下找相关配置，如 /etc/mysql/mysql.conf.d/mysqld.cnf)   
最后重启mysql   
```sudo /etc/init.d/mysql restart```