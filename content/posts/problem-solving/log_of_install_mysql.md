---
title: Homebrew启动Mysql踩坑记录
date: 2024-10-07
categories:
  - Technology
tags:
  - ProblemSloving
nolastmod: true
cover: /img/pic-6.jpg
draft: true
---
### 起因
原来用Homebrew安装过mysql@8.0和@5.7均可使用，一段时间后当我运行`brew services run mysql`时仍然会显示成功，但是`brew services list`显示的是状态却是`stopped`。尝试了重新启动和卸载均无效。
### 解决
1. 查看日志，在`/usr/local/var/mysql/xxx.err`的错误日志文件报错信息为：Failed to find valid data directory.
2. 以此为线索查看解决方案：`mysqld --initialize`，This shall initialse the data directory in the path where you have installed MySql server.
3. `command not found: mysqld`，查看mysql安装路径为`/usr/local/opt/mysql/bin/`，注意mysql需要加上具体的版本号，比如`/usr/local/opt/mysql@8.0/bin/`，发现是有`mysqld`的，需要在`~/.zshrc`中添加对应的路径。
4. 此时重新运行`mysqld --initialize`，依然失败，报错信息提示`/usr/local/var/mysql`已经存在，使用`sudo rm -rf /usr/local/var/mysql`删除后重新执行，成功。
5. 记住保存好临时的密码，重新启动mysql服务，成功。
6. `mysql -u root -p`,然后输入刚刚的临时密码，执行`ALTER USER 'root'@'localhost' IDENTIFIED BY 'newpassword';`，执行`FLUSH PRIVILEGES;`使密码修改生效。