# MySQL配置的一些坑

## 安装

使用apt安装

```bash
sudo apt install mysql-server
```

初始化配置。根据自己需要的安全等级选择就好

```bash
sudo mysql_secure_installation
```

## 新建用户

我们在使用工具连接时不应该使用root用户登录

```mysql
CREATE DATABASE DB;
GRANT ALL PRIVILEGES ON DB.* TO yb@% IDENTIFIED BY "PASSWD";
```

这样用户yb就可以通过任意IP访问DB数据库的任何表

## 配置字符集

MySQL默认字符集为Latin，不支持中文显示，我们需要修改`/etc/mysql/mysqld.cnf`这个文件

```bash
[mysqld]
default-storage-engine = INNODB
character-set-server = utf8
collation-server = utf8_general_ci
```

但是这对于已经创建的数据库并不适用

```mysql
alter database DB character set utf8;
show VARIABLES like 'character%';
```