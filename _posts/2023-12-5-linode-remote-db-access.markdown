---
layout: post
title: (筆記) liode-remote-db-access
date: 2023-12-5 15:10:00 +0800
image: 02.jpg
tags: Resources
---

紀錄一下 linode 安裝 mysql 直到在本機用 sql editor 連線進去的過程

1. 安裝 mysql-server

```bash
sudo apt update && sudo apt install mysql-server
```

2. 初始化 mysql-server

```bash
sudo mysql_secure_installation
# VALIDATE PASSWORD component :  N
# root remote access :  N
# 其他看自己
```

3. 改 mysql 外部連線設定

```bash
vim /etc/mysql/mysql.conf.d/mysqld.cnf
# bind-address 127.0.0.1  -> 0.0.0.0
```

4. 新增遠端存取的使用者

```bash
CREATE USER 'remote'@'%' IDENTIFIED BY '12345';
GRANT ALL PRIVILEGES ON *.* TO 'remote'@'%' WITH GRANT OPTION;
FLUSH PRIVILEGES;
```

5. 開防火牆

```bash
sudo ufw allow 3306
```

6. 重開 mysql

```bash
sudo systemctl restart mysql
```

7. 刪除 mysql-server

```bash
sudo apt purge --auto-remove mysql-common mysql-server
sudo rm -rf /etc/mysql rm -rf /var/lib/mysql
sudo killall -9 mysql userdel mysql
```
